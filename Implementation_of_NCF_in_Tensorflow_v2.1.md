---
tags: Paper, Tensorflow, Deep Learning
---

# TF2如何實現NCF

[TOC]

## Tensorflow 官方實作版本 (r2.1.0)

建立ncf網路結構: <https://github.com/tensorflow/models/blob/r2.1.0/official/recommendation/neumf_model.py#L152>

啟動方式: <https://github.com/tensorflow/models/blob/r2.1.0/official/recommendation/ncf_keras_main.py#L200>

必要前置作業:

```bash
export PYTHONPATH=$PYTHONPATH:/home/tom/tesorflow-models
pip3 install --user -r official/requirements.txt
```

啟動方式

```bash
python ncf_keras_main.py --eval_batch_size 1000 --num_gpus 0 --data_dir /tmp/movielens-data/ --dataset ml-1m
```

### 紀錄ncf用到了哪些flags

- run_ncf
  - FLAGS.enable_xla
  - FLAGS.seed
  - FLAGS.model_dir
  - FLAGS.dtype
  - FLAGS.fp16_implementation
  - FLAGS.loss_scale
  - FLAGS.distribution_strategy
  - FLAGS.num_gpus
  - FLAGS.tpu
- ncf
  - FLAGS.batch_size
  - flags_obj.eval_batch_size
  - flags_obj.batch_size
  - flags_obj.train_epochs
  - learning_rate
  - num_factors
  - layers
  - mf_regularization
  - mlp_regularization
  - num_neg
  - distribution_strategy
  - tpu_zone
  - tpu_gcp_project
  - beta1
  - beta2
  - epsilon
  - ml_perf
  - keras_use_ctl
  - hr_threshold
  - train_dataset_path
  - eval_dataset_path
  - input_meta_data_path
  - early_stopping

### NCF 進入點

official/recommendation/ncf_keras_main.py#L200

```python
def run_ncf(_):
    """Run NCF training and eval with Keras."""

    # google考量到tf版本差異,另外 function 處理 session config.
    # 參考 official/utils/misc/keras_utils.py#L161
    keras_utils.set_session_config(enable_xla=FLAGS.enable_xla)   

    # 設定 random seed
    if FLAGS.seed is not None:
        print("Setting tf seed")
        tf.random.set_seed(FLAGS.seed)  

    # 刪除目錄 FLAGS.model_dir
    # 參考 official/utils/misc/model_helpers.py#L89
    model_helpers.apply_clean(FLAGS)

    # 設定全域 policy
    # get_loss_scale 用到了 FLAGS.loss_scale 和 FLAGS.dtype
    # 參考 official/utils/flags/_performance.py#L45
    if FLAGS.dtype == "fp16" and FLAGS.fp16_implementation == "keras":
      policy = tf.keras.mixed_precision.experimental.Policy(
          "mixed_float16",
          loss_scale=flags_core.get_loss_scale(FLAGS, default_for_fp16="dynamic"))
      tf.keras.mixed_precision.experimental.set_policy(policy)

    # 設定分散式策略
    # 參考 official/utils/misc/distribution_utils.py#L84
    strategy = distribution_utils.get_distribution_strategy(
        distribution_strategy=FLAGS.distribution_strategy,
        num_gpus=FLAGS.num_gpus,
        tpu_address=FLAGS.tpu)

    # 抽取ncf需要的參數,以dict呈現
    # 參考 official/recommendation/ncf_common.py#L68
    params = ncf_common.parse_flags(FLAGS)
    params["distribute_strategy"] = strategy

    # 限定tf2才能設定分散式策略
    if not keras_utils.is_v2_0() and strategy is not None:
      logging.error("NCF Keras only works with distribution strategy in TF 2.0")
      return
    # custom training loop 只能用在tf2以及分散式策略不能是None
    if (params["keras_use_ctl"] and (
        not keras_utils.is_v2_0() or strategy is None)):
      logging.error(
          "Custom training loop only works with tensorflow 2.0 and dist strat.")
      return
    # 採用 TPU 必須使用 custom training loop !?
    if params["use_tpu"] and not params["keras_use_ctl"]:
      logging.error("Custom training loop must be used when using TPUStrategy.")
      return

    # TimeHistory: keras callback 紀錄時間用
    # TimeHistory 參考 official/utils/misc/keras_utils.py#L44
    batch_size = params["batch_size"]
    time_callback = keras_utils.TimeHistory(batch_size, FLAGS.log_steps)
    callbacks = [time_callback]
    producer, input_meta_data = None, None
    generate_input_online = params["train_dataset_path"] is None  # 預設是 None

    # 如果沒有 train_dataset_path, 預設使用movielens資料集
    # 參考 official/recommendation/ncf_common.py#L43
    if generate_input_online:
        # Start data producing thread.
        num_users, num_items, _, _, producer = ncf_common.get_inputs(params)
        producer.start()
        # 增加 keras callback: IncrementEpochCallback
        per_epoch_callback = IncrementEpochCallback(producer)
        callbacks.append(per_epoch_callback)
    else:
        assert params["eval_dataset_path"] and params["input_meta_data_path"]
        with tf.io.gfile.GFile(params["input_meta_data_path"], "rb") as reader:
            input_meta_data = json.loads(reader.read().decode("utf-8"))
            num_users = input_meta_data["num_users"]
            num_items = input_meta_data["num_items"]

    # user數量和item數量這時候才會知道
    params["num_users"], params["num_items"] = num_users, num_items

    # 設定 early stopping 條件
    # 增加 keras callback: CustomEarlyStopping
    if FLAGS.early_stopping:
        early_stopping_callback = CustomEarlyStopping(
            "val_HR_METRIC", desired_value=FLAGS.hr_threshold)
        callbacks.append(early_stopping_callback)

    # 還無法理解 train_input_dataset 和 eval_input_dataset 是什麼物件??
    # 參考: official/recommendation/ncf_input_pipeline.py
    train_input_dataset, eval_input_dataset, num_train_steps, num_eval_steps = (
        ncf_input_pipeline.create_ncf_input_data(
            params, producer, input_meta_data, strategy
        )
    )
    steps_per_epoch = None if generate_input_online else num_train_steps

    with distribution_utils.get_strategy_scope(strategy):

        # 建立NCF網路結構的關鍵!!
        keras_model = _get_keras_model(params)
        
        # optimizer 會因為參數而影響建立方式.
        optimizer = tf.keras.optimizers.Adam(
            learning_rate=params["learning_rate"],
            beta_1=params["beta1"],
            beta_2=params["beta2"],
            epsilon=params["epsilon"]
        )
        if FLAGS.fp16_implementation == "graph_rewrite":
            optimizer = tf.compat.v1.train.experimental.enable_mixed_precision_graph_rewrite(
                optimizer,
                loss_scale=flags_core.get_loss_scale(FLAGS, default_for_fp16="dynamic")
            )
        elif FLAGS.dtype == "fp16" and params["keras_use_ctl"]:
        # When keras_use_ctl is False, instead Model.fit() automatically applies
        # loss scaling so we don't need to create a LossScaleOptimizer.
        optimizer = tf.keras.mixed_precision.experimental.LossScaleOptimizer(
            optimizer,
            tf.keras.mixed_precision.experimental.global_policy().loss_scale
        )

        # 有 custom training loop 的訓練方式
        if params["keras_use_ctl"]:
            train_loss, eval_results = run_ncf_custom_training(
                params,
                strategy,
                keras_model,
                optimizer,
                callbacks,
                train_input_dataset,
                eval_input_dataset,
                num_train_steps,
                num_eval_steps,
                generate_input_online=generate_input_online
            )

        # 沒有 custom training loop 的訓練方式
        else:
            # TODO(b/138957587): Remove when force_v2_in_keras_compile is on longer
            # a valid arg for this model. Also remove as a valid flag.
            
            # 參數 `run_eagerly` 不在源碼的參數列表內,看源碼才知道!
            if FLAGS.force_v2_in_keras_compile is not None:
                keras_model.compile(
                    optimizer=optimizer,
                    run_eagerly=FLAGS.run_eagerly,
                    experimental_run_tf_function=FLAGS.force_v2_in_keras_compile
                )
            else:
                keras_model.compile(optimizer=optimizer, run_eagerly=FLAGS.run_eagerly)

            # 增加 keras callback: TensorBoard, ModelCheckpoint
            if not FLAGS.ml_perf:
                # Create Tensorboard summary and checkpoint callbacks.
                summary_dir = os.path.join(FLAGS.model_dir, "summaries")
                summary_callback = tf.keras.callbacks.TensorBoard(summary_dir)
                checkpoint_path = os.path.join(FLAGS.model_dir, "checkpoint")
                checkpoint_callback = tf.keras.callbacks.ModelCheckpoint(
                    checkpoint_path, save_weights_only=True
                )
                callbacks += [summary_callback, checkpoint_callback]

            # 開始訓練
            history = keras_model.fit(
                train_input_dataset,
                epochs=FLAGS.train_epochs,
                steps_per_epoch=steps_per_epoch,
                callbacks=callbacks,
                validation_data=eval_input_dataset,
                validation_steps=num_eval_steps,
                verbose=2
            )

            logging.info("Training done. Start evaluating")

            # 拿 eval_data 評估表現
            eval_loss_and_metrics = keras_model.evaluate(
                eval_input_dataset, steps=num_eval_steps, verbose=2
            )

            logging.info("Keras evaluation is done.")

            # Keras evaluate() API returns scalar loss and metric values from
            # evaluation as a list. Here, the returned list would contain
            # [evaluation loss, hr sum, hr count].
            eval_hit_rate = eval_loss_and_metrics[1] / eval_loss_and_metrics[2]

            # Format evaluation result into [eval loss, eval hit accuracy].
            eval_results = [eval_loss_and_metrics[0], eval_hit_rate]

            if history and history.history:
              train_history = history.history
              train_loss = train_history["loss"][-1]

    # 總結指標
    stats = build_stats(train_loss, eval_results, time_callback)
    return stats
```

### NCF base_model

official/recommendation/neumf_model.py#L152

```python
def construct_model(user_input, item_input, params):
  # type: (tf.Tensor, tf.Tensor, dict) -> tf.keras.Model
  """Initialize NeuMF model.

  Args:
    user_input: keras input layer for users
    item_input: keras input layer for items
    params: Dict of hyperparameters.
  Raises:
    ValueError: if the first model layer is not even.
  Returns:
    model:  a keras Model for computing the logits
  """
  num_users = params["num_users"]
  num_items = params["num_items"]

  model_layers = params["model_layers"]

  mf_regularization = params["mf_regularization"]
  mlp_reg_layers = params["mlp_reg_layers"]

  mf_dim = params["mf_dim"]

  mlperf_helper.ncf_print(key=mlperf_helper.TAGS.MODEL_HP_MF_DIM, value=mf_dim)
  mlperf_helper.ncf_print(key=mlperf_helper.TAGS.MODEL_HP_MLP_LAYER_SIZES,
                          value=model_layers)

  if model_layers[0] % 2 != 0:
    raise ValueError("The first layer size should be multiple of 2!")

  # Initializer for embedding layers
  embedding_initializer = "glorot_uniform"

  def mf_slice_fn(x):
    x = tf.squeeze(x, [1])
    return x[:, :mf_dim]

  def mlp_slice_fn(x):
    x = tf.squeeze(x, [1])
    return x[:, mf_dim:]

  # It turns out to be significantly more effecient to store the MF and MLP
  # embedding portions in the same table, and then slice as needed.
  embedding_user = tf.keras.layers.Embedding(
      num_users,
      mf_dim + model_layers[0] // 2,
      embeddings_initializer=embedding_initializer,
      embeddings_regularizer=tf.keras.regularizers.l2(mf_regularization),
      input_length=1,
      name="embedding_user")(
          user_input)

  embedding_item = tf.keras.layers.Embedding(
      num_items,
      mf_dim + model_layers[0] // 2,
      embeddings_initializer=embedding_initializer,
      embeddings_regularizer=tf.keras.regularizers.l2(mf_regularization),
      input_length=1,
      name="embedding_item")(
          item_input)

  # GMF part
  mf_user_latent = tf.keras.layers.Lambda(
      mf_slice_fn, name="embedding_user_mf")(embedding_user)
  mf_item_latent = tf.keras.layers.Lambda(
      mf_slice_fn, name="embedding_item_mf")(embedding_item)

  # MLP part
  mlp_user_latent = tf.keras.layers.Lambda(
      mlp_slice_fn, name="embedding_user_mlp")(embedding_user)
  mlp_item_latent = tf.keras.layers.Lambda(
      mlp_slice_fn, name="embedding_item_mlp")(embedding_item)

  # Element-wise multiply
  mf_vector = tf.keras.layers.multiply([mf_user_latent, mf_item_latent])

  # Concatenation of two latent features
  mlp_vector = tf.keras.layers.concatenate([mlp_user_latent, mlp_item_latent])

  num_layer = len(model_layers)  # Number of layers in the MLP
  for layer in xrange(1, num_layer):
    model_layer = tf.keras.layers.Dense(
        model_layers[layer],
        kernel_regularizer=tf.keras.regularizers.l2(mlp_reg_layers[layer]),
        activation="relu"
    )
    mlp_vector = model_layer(mlp_vector)

  # Concatenate GMF and MLP parts
  predict_vector = tf.keras.layers.concatenate([mf_vector, mlp_vector])

  # Final prediction layer
  logits = tf.keras.layers.Dense(
      1, activation=None, kernel_initializer="lecun_uniform",
      name=movielens.RATING_COLUMN)(predict_vector)

  # Print model topology.
  model = tf.keras.models.Model([user_input, item_input], logits)
  model.summary()
  sys.stdout.flush()

  return model
```

### 對 base_model 繼續疊加 Layers

official/recommendation/ncf_keras_main.py#L148

```python
def _get_keras_model(params):
  """Constructs and returns the model."""
  batch_size = params["batch_size"]

  user_input = tf.keras.layers.Input(
      shape=(1,), name=movielens.USER_COLUMN, dtype=tf.int32)

  item_input = tf.keras.layers.Input(
      shape=(1,), name=movielens.ITEM_COLUMN, dtype=tf.int32)

  valid_pt_mask_input = tf.keras.layers.Input(
      shape=(1,), name=rconst.VALID_POINT_MASK, dtype=tf.bool)

  dup_mask_input = tf.keras.layers.Input(
      shape=(1,), name=rconst.DUPLICATE_MASK, dtype=tf.int32)

  label_input = tf.keras.layers.Input(
      shape=(1,), name=rconst.TRAIN_LABEL_KEY, dtype=tf.bool)

  base_model = neumf_model.construct_model(user_input, item_input, params)

  # 繼續對 base model 疊加 layers

  logits = base_model.output

  zeros = tf.keras.layers.Lambda(lambda x: x * 0)(logits)

  softmax_logits = tf.keras.layers.concatenate([zeros, logits], axis=-1)

  # Custom training loop calculates loss and metric as a part of
  # training/evaluation step function.
  if not params["keras_use_ctl"]:
    softmax_logits = MetricLayer(params)([softmax_logits, dup_mask_input])
    # TODO(b/134744680): Use model.add_loss() instead once the API is well
    # supported.
    softmax_logits = LossLayer(batch_size)(
        [softmax_logits, label_input, valid_pt_mask_input])

  keras_model = tf.keras.Model(
      inputs={
          movielens.USER_COLUMN: user_input,
          movielens.ITEM_COLUMN: item_input,
          rconst.VALID_POINT_MASK: valid_pt_mask_input,
          rconst.DUPLICATE_MASK: dup_mask_input,
          rconst.TRAIN_LABEL_KEY: label_input},
      outputs=softmax_logits)

  keras_model.summary()
  return keras_model
```

## Tensorflow對資料集MovieLens做哪些前處理

搞了近半天時間,才找到處理movielens的流程,在這之前一直搞不明白dataset如何得知NCF的input需求.

### num_users, num_items, _, _, producer = ncf_common.get_inputs(params)

```python
def get_inputs(params):
  """Returns some parameters used by the model."""
  
  # FLAGS.dataset 和 FLAGS.data_dir 預設是 movielens 下載位置
  # FLAGS.use_synthetic_data 預設是 False
  
  if FLAGS.download_if_missing and not FLAGS.use_synthetic_data:
    movielens.download(FLAGS.dataset, FLAGS.data_dir)  

  if FLAGS.seed is not None:
    np.random.seed(FLAGS.seed)

  if FLAGS.use_synthetic_data:
    producer = data_pipeline.DummyConstructor()
    num_users, num_items = data_preprocessing.DATASET_TO_NUM_USERS_AND_ITEMS[
        FLAGS.dataset]
    num_train_steps = rconst.SYNTHETIC_BATCHES_PER_EPOCH
    num_eval_steps = rconst.SYNTHETIC_BATCHES_PER_EPOCH
  else:
    # 對 movielens 處理看這段
    num_users, num_items, producer = data_preprocessing.instantiate_pipeline(
        dataset=FLAGS.dataset, data_dir=FLAGS.data_dir, params=params,
        constructor_type=FLAGS.constructor_type,
        deterministic=FLAGS.seed is not None
    )
    num_train_steps = producer.train_batches_per_epoch
    num_eval_steps = producer.eval_batches_per_epoch

  return num_users, num_items, num_train_steps, num_eval_steps, producer
```

### num_users, num_items, producer = data_preprocessing.instantiate_pipeline

參考 official/recommendation/data_preprocessing.py#L180

```python
def instantiate_pipeline(dataset,
                         data_dir,
                         params,
                         constructor_type=None,
                         deterministic=False,
                         epoch_dir=None,
                         generate_data_offline=False):
  # type: (str, str, dict, typing.Optional[str], bool, typing.Optional[str], bool) -> (int, int, data_pipeline.BaseDataConstructor)
  """Load and digest data CSV into a usable form.

  Args:
    dataset: The name of the dataset to be used.
    data_dir: The root directory of the dataset.
    params: dict of parameters for the run.
    constructor_type: The name of the constructor subclass that should be used
      for the input pipeline.
    deterministic: Tell the data constructor to produce deterministically.
    epoch_dir: Directory in which to store the training epochs.
    generate_data_offline: Boolean, whether current pipeline is done offline
      or while training.
  """
  logging.info("Beginning data preprocessing.")
  st = timeit.default_timer()
  raw_rating_path = os.path.join(data_dir, dataset, movielens.RATINGS_FILE)
  cache_path = os.path.join(data_dir, dataset, rconst.RAW_CACHE_FILE)

  # raw_data 是 dict, dataframe包在裡面
  # 參考: official/recommendation/data_preprocessing.py#L158
  raw_data, _ = _filter_index_sort(raw_rating_path, cache_path)  # 整理dataframe過程,值得參考
  user_map, item_map = raw_data["user_map"], raw_data["item_map"]
  num_users, num_items = DATASET_TO_NUM_USERS_AND_ITEMS[dataset]

  if num_users != len(user_map):
    raise ValueError("Expected to find {} users, but found {}".format(
        num_users, len(user_map)))
  if num_items != len(item_map):
    raise ValueError("Expected to find {} items, but found {}".format(
        num_items, len(item_map)))

  producer = data_pipeline.get_constructor(constructor_type or "materialized")(
      maximum_number_epochs=params["train_epochs"],
      num_users=num_users,
      num_items=num_items,
      user_map=user_map,
      item_map=item_map,
      train_pos_users=raw_data[rconst.TRAIN_USER_KEY],
      train_pos_items=raw_data[rconst.TRAIN_ITEM_KEY],
      train_batch_size=params["batch_size"],
      batches_per_train_step=params["batches_per_step"],
      num_train_negatives=params["num_neg"],
      eval_pos_users=raw_data[rconst.EVAL_USER_KEY],
      eval_pos_items=raw_data[rconst.EVAL_ITEM_KEY],
      eval_batch_size=params["eval_batch_size"],
      batches_per_eval_step=params["batches_per_step"],
      stream_files=params["stream_files"],
      deterministic=deterministic,
      epoch_dir=epoch_dir,
      create_data_offline=generate_data_offline)

  run_time = timeit.default_timer() - st
  logging.info("Data preprocessing complete. Time: {:.1f} sec."
               .format(run_time))

  print(producer)
  return num_users, num_items, producer
```

### producer.start() ?

### per_epoch_callback = IncrementEpochCallback(producer)

### ncf_input_pipeline.create_ncf_input_data(params, producer, input_meta_data, strategy)
