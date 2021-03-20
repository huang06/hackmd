---
tags: MQTT
---

# MQTT 3.1.1 Essentials 翻譯+重點整理

---

<https://www.hivemq.com/mqtt-essentials/>

September 2020

---

[TOC]

## Part 1: Introducing MQTT

<https://www.hivemq.com/blog/mqtt-essentials-part-1-introducing-mqtt/> Published: January 12, 2015

MQTT並非是**message queue**的解決方案，即使它能做到。

> Many sources label MQTT incorrectly as a message queue protocol. That is simply not true. MQTT is not a traditional message queuing solution (although it is possible to queue messages in certain cases, a fact that we discuss in detail in an upcoming post).

2019年，MQTT 5 版本釋出。

> In March 2019, OASIS ratified the new MQTT 5 specification.

## Part 2: Publish & Subscribe Basics

<https://www.hivemq.com/blog/mqtt-essentials-part2-publish-subscribe/> Published: January 19, 2015

### The publish/subscribe pattern

![](https://i.imgur.com/K7W5npu.png)

從3個維度探討decoupling的好處:

- 空間解耦: pub/sub不需要知道對方(e.g. 不需交換對方的ip,port)
- 時間解耦: pub/sub不需要同時執行
- 同步性解耦: pub/sub不會因為發送或接收訊息時中斷工作。

> The most important aspect of pub/sub is the decoupling of the publisher of the message from the recipient (subscriber). This decoupling has several dimensions:
> - Space decoupling: Publisher and subscriber do not need to know each other (for example, no exchange of IP address and port).
> - Time decoupling: Publisher and subscriber do not need to run at the same time.
> - Synchronization decoupling: Operations on both components do not need to be interrupted during publishing or receiving.

### Scalability

比起傳統client-server,pub/sub的擴充性更好。原因包含broker可以被高度平行化以及訊息能被event-driven方式處理。關鍵字：**message caching**, **intelligent routing of messages**。

> Pub/Sub scales better than the traditional client-server approach. This is because operations on the broker can be highly parallelized and messages can be processed in an event-driven way. Message caching and intelligent routing of messages are often a decisive factors for improving scalability.

### Message filtering

SUBJECT-BASED FILTERING: 根據message的主題過濾。

CONTENT-BASED FILTERING: 根據message的關鍵字過濾，重點是message不能被加密以及不能輕易被更改。

TYPE-BASED FILTERING: 當使用OOP語言，透過type/class進行過濾。

> The broker has several filtering options:
> - OPTION 1: SUBJECT-BASED FILTERING
  This filtering is based on the subject or topic that is part of each message. The receiving client subscribes to the broker for topics of interest. From that point on, the broker ensures that the receiving client gets all message published to the subscribed topics. In general, topics are strings with a hierarchical structure that allow filtering based on a limited number of expressions.
> - OPTION 2: CONTENT-BASED FILTERING
  In content-based filtering, the broker filters the message based on a specific content filter-language. The receiving clients subscribe to filter queries of messages for which they are interested. A significant downside to this method is that the content of the message must be known beforehand and cannot be encrypted or easily changed.
> - OPTION 3: TYPE-BASED FILTERING
  When object-oriented languages are used, filtering based on the type/class of a message (event) is a common practice. For example,, a subscriber can listen to all messages, which are of type Exception or any sub-type.

pub/sub並非適用於所有use case。而decoupling面臨的挑戰包含:

1. 必須要事先知道來自pub data的資料結構。
2. subject-based filtering, pub和sub都需要知道使用哪個topic。
3. message delivery, pub無法假設會有人接收它正傳出的message, 所以可能發生沒有sub讀取message的情形。

> **Of course, publish/subscribe is not the answer for every use case. There are a few things you need to consider before you use this model.** The decoupling of publisher and subscriber, which is the key in pub/sub, presents a few challenges of its own. For example, you need to be aware of how the published data is structured beforehand. For subject-based filtering, both publisher and subscriber need to know which topics to use. Another thing to keep in mind is message delivery. The publisher can’t assume that somebody is listening to the messages that are sent. In some instances, it is possible that no subscriber reads a particular message.

### MQTT

這段主要說明MQTT實現哪些pub/sub的功能，以及透過**QoS levels**解決上述的pub/sub的挑戰，但看不太懂**QoS**的例子，待後續章節是否會詳細說明**QoS**。

### Distinction from message queues

**message queue**會一直儲存message直到consumer來拿資料。但是**MQTT**會把沒有sub的message處理掉。

> **A message queue stores message until they are consumed** When you use a message queue, each incoming message is stored in the queue until it is picked up by a client (often called a consumer). If no client picks up the message, the message remains stuck in the queue and waits to be consumed. In a message queue, it is not possible for a message not to be processed by any client, as it is in MQTT if nobody subscribes to a topic.

**message queue**的message只能被一個consumer進行處理。但**MQTT**的多個sub可以接收同一個topic的message。

> **A message is only consumed by one client** Another big difference is that in a traditional message queue a message can be processed by one consumer only. The load is distributed between all consumers for a queue. In MQTT the behavior is quite the opposite: every subscriber that subscribes to the topic gets the message.

**queue**必須被明確地命名且建立後才能對message進行publish/consume，但**MQTT**可以即時建立topic。

> **Queues are named and must be created explicitly** A queue is far more rigid than a topic. Before a queue can be used, the queue must be created explicitly with a separate command. Only after the queue is named and created is it possible to publish or consume messages. In contrast, MQTT topics are extremely flexible and can be created on the fly.

## Part 3: Client, Broker and Connection Establishment

<https://www.hivemq.com/blog/mqtt-essentials-part-3-client-broker-connection-establishment/> Published: July 17, 2019

### Introduction-Client

pub/sub都被視為MQTT clients，而單一client可以同時扮演pub和sub的角色。

> Both publishers and subscribers are MQTT clients. The publisher and subscriber labels refer to whether the client is currently publishing messages or subscribing to messages (publish and subscribe functionality can also be implemented in the same MQTT client).

### Introduction-Broker

broker的工作包含: 接收訊息,過濾訊息,確認誰訂閱訊息,傳送訊息給訂閱者,保留persistent session的session data, authentication, authorization of clients

### MQTT Connection

MQTT protocol是基於TCP/IP stack。

> The MQTT protocol is based on TCP/IP. Both the client and the broker need to have a TCP/IP stack.

![](https://i.imgur.com/76WpFM7.png)

初始化連線：

1. client 傳送 **CONNECT** message 給 broker.
2. broker 回傳 **CONNACK** message 以及 status code.

broker會一直持續開啟connection, 直到client傳送disconnect command或是連線中斷。

> The MQTT connection is always between one client and the broker. Clients never connect to each other directly. To initiate a connection, the client sends a CONNECT message to the broker. The broker responds with a CONNACK message and a status code. Once the connection is established, the broker keeps it open until the client sends a disconnect command or the connection breaks.

![](https://i.imgur.com/P4aCcLx.png)

### MQTT connection through a NAT

MQTT不會受到NAT影響。詳細原因看不太明白...

### Client initiates connection with the CONNECT message

這段介紹 **CONNECT** message 包含哪些資訊。

![CONNECT message](https://i.imgur.com/KTDkaAd.png)

#### ClientId

> The broker uses the ClientID to identify the client and the current state of the client.

> In MQTT 3.1.1 (the current standard), you can send an empty ClientId, if you don’t need a state to be held by the broker. The empty ClientID results in a connection without any state. In this case, the **clean session** flag must be set to true or the broker will reject the connection.

#### Clean Session

> The clean session flag tells the broker whether the client wants to establish a persistent session or not.

#### Username/Password

> MQTT can send a user name and password for client authentication and authorization.

#### Will Message

> The last will message **is part of** the Last Will and Testament (LWT) feature of MQTT. This message notifies other clients when a client disconnects ungracefully. 

#### KeepAlive

> The keep alive is **a time interval in seconds** that the client specifies and communicates to the broker when the connection established. This interval defines the longest period of time that the broker and client can endure without sending a message.

> The client commits to sending regular **PING Request** messages to the broker. The broker responds with a **PING response**. This method allows both sides to determine if the other one is still available.

### Broker response with a CONNACK message

當broker收到 **CONNECT** message,會回傳 **CONNACK** message。

> When a broker receives a **CONNECT** message, it is obligated to respond with a **CONNACK** message. The **CONNACK** message contains two data entries:
> 1. The session present flag
> 2. A connect acknowledge flag

![CONNACK message](https://i.imgur.com/1xdKzu5.png)

#### Session Present flag

回傳告訴client上一次的連線是否已經有 persistent session。

> The session present flag tells the client whether the broker already has a persistent session available from previous interactions with the client.

如果client的 **Clean Session** 是True,回傳的 **session present flag** 一定會是 False,  
但是client的 **Clean Session** 是False,回傳的 **session present flag** 有2種可能:  
1. 如果broker有保存 session information, 回傳 True
2. 如果broker沒保存 session information, 回傳 False

這個flag幫助client確認是否需要subscribe topics, 或是topic是否仍存在 persistent session。

> This flag was added in MQTT 3.1.1 to help clients determine whether they need to subscribe to topics or if the topics are still stored in a persistent session.

#### Connect acknowledge flag

> This flag contains a return code that tells the client whether the connection attempt was successful.

Return Code 只有 0 才代表成功。

![](https://i.imgur.com/TMkHynW.png)

### Loose ends

看後續章節,會說明如何讓connection保持open,或是如何得知connection is lost。

## Part 4: Publish, Subscribe & Unsubscribe

<https://www.hivemq.com/blog/mqtt-essentials-part-4-mqtt-publish-subscribe-unsubscribe/> Published: February 2, 2015

### Publish

> Each message must contain a **topic** that the broker can use to forward the message to interested clients. Typically, each message has a **payload** which contains the data to transmit in byte format.

data-agnostic, 由client決定layload的結構。

> MQTT is data-agnostic. The use case of the client determines how the payload is structured. The sending client (publisher) decides whether it wants to send binary data, text data, or even full-fledged XML or JSON.

![PUBLISH message](https://i.imgur.com/F5NG5mu.png)

#### Topic Name

#### QoS

詳見 **part 6 of MQTT Essentials.**

> This number indicates the Quality of Service Level (QoS) of the message.

> The service level determines what kind of guarantee a message has for reaching the intended recipient (client or broker).

#### Retain Flag

詳見 **part 8 of MQTT Essentials.**

Broker 收到这样的 PUBLISH 包以后，将保存这个消息，当有一个新的订阅者订阅相应主题的时候，Broker 会马上将这个消息发送给订阅者。

> This flag defines whether the message is saved by the broker as the last known good value for a specified topic.

> When a new client subscribes to a topic, they receive the last message that is retained on that topic.

#### Payload

#### Packet Identifier

> The packet identifier uniquely identifies a message as it flows between the client and broker.

**QoS** > 0 

> The packet identifier is only relevant for QoS levels greater than zero.

#### DUP flag

是否是重複的message。參考QoS章節會比較容易懂。

> The flag indicates that the message is a duplicate and was resent because the intended recipient (client or broker) did not acknowledge the original message.

**QoS** > 0

---

當client傳送message給broker,broker讀message,根據QoS進行ack,處理message(確認對應的topic,並傳送給對應的subscribers)。

![](https://i.imgur.com/aJM9ran.png)

### Subscribe

#### Packet Identifier

#### List of Subscriptions

A subscription = (topic name, QoS level)

wildcards: 可以用pattern指定多個topics

> A **SUBSCRIBE** message can contain multiple subscriptions for a client. Each subscription is made up of a topic and a QoS level. The topic in the subscribe message can contain **wildcards** that make it possible to subscribe to a topic pattern rather than a specific topic.

### Suback

基於訂閱數量會回傳多個 return codes。

> To confirm each subscription, the broker sends a SUBACK acknowledgement message to the client.

![](https://i.imgur.com/Wrmk9vT.png)

#### Packet Identifier

#### Return Code

![](https://i.imgur.com/VfXN6xc.png)

![](https://i.imgur.com/GVFwjz9.png)

### Unsubscribe

直接看圖

![](https://i.imgur.com/3ANf9Fd.png)

### Unsuback

直接看圖

![](https://i.imgur.com/xD2M4Fy.png)

![](https://i.imgur.com/JPPjT5B.png)

## Part 5: Topics & Best Practices

<https://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices/> Published: August 20, 2019

### Topics

![](https://i.imgur.com/oWr4wsy.png)

1. In MQTT, the word topic refers to an **UTF-8** string that the broker uses to filter messages for each connected client. The topic consists of one or more topic levels. 
2. Each topic level is separated by a **forward slash** (topic level separator).
3. In comparison to a **message queue**, MQTT topics are very lightweight. The client does not need to create the desired topic before they publish or subscribe to it. The broker accepts each valid topic without any prior initialization.
4. Note that each topic must contain at least 1 character and that the topic string permits empty spaces.
5. Topics are case-sensitive.
6. the forward slash alone is a valid topic.

### Wildcards

#### Single Level: +

直接看圖

![](https://i.imgur.com/kSuSH5E.png)

![](https://i.imgur.com/plQWOgS.png)

#### Multi Level: \#

直接看圖

![](https://i.imgur.com/iSmO85h.png)

![](https://i.imgur.com/ItsFjhX.png)

### Topics beginning with $

> The $-symbol topics are reserved for internal statistics of the MQTT broker.

> Commonly, **$SYS/** is used for all the following information, but broker implementations varies.

<https://github.com/mqtt/mqtt.github.io/wiki/SYS-Topics>

```vim
$SYS/broker/clients/connected
$SYS/broker/clients/disconnected
$SYS/broker/clients/total
$SYS/broker/messages/sent
$SYS/broker/uptime
```

### Best practices

#### Never use a leading forward slash

``/myhome/groundfloor/livingroom`` 代表最上層有個0字元的topic

#### Never use spaces in a topic

1. 不容易閱讀和debug
2. UTF-8有很多不同 white space types

#### Keep the topic short and concise

#### Use only ASCII characters, avoid non printable characters

non-ASCII UTF-8 會有顯示問題

#### Embed a unique identifier or the Client Id into the topic

1. 可以幫助識別誰傳送message
2. 可以作為授權管理,client只能能傳送message至對應的topic

#### Don’t subscribe to #

透過在broker寫插件進行將message寫入db,而不要直接透過subscribe將全部資料讀出。

> Frequently, the subscribing client is not able to process the load of messages that results from this method (especially if you have a massive throughput).

> Our recommendation is to implement an **extension** in the MQTT broker. For example, with the plugin system of HiveMQ you can hook into the behavior of HiveMQ and add an asynchronous routine to process each incoming message and persist it to a database.

#### Don’t forget extensibility

建立topic tree時先想好未來的擴展性,避免未來將topic tree砍掉重建。

#### Use specific topics, not general ones

盡量將topic區分開,方便使用其他MQTT features(e.g. retained messages)

## Part 6: Quality of Service Levels

<https://www.hivemq.com/blog/mqtt-essentials-part-6-mqtt-quality-of-service-levels/> February 16, 2015

### What is Quality of Service?

QoS是雙方一起遵守的協議。

> The Quality of Service (QoS) level is an agreement between the sender of a message and the receiver of a message that defines the guarantee of delivery for a specific message. There are 3 QoS levels in MQTT:
> 1. At most once (0)
> 2. At least once (1)
> 3. Exactly once (2).

當提及MQTT QoS，必須考量到以下2個情境

> When you talk about QoS in MQTT, you need to consider the two sides of message delivery:
> 1. Message delivery form the publishing client to the broker.
> 2. Message delivery from the broker to the subscribing client.

### Why is Quality of Service important?

> QoS gives the client the power to choose a level of service that matches its network reliability and application logic.

### QoS 0 - at most once

盡力而為

> This service level guarantees a best-effort delivery.

射後不理

> QoS level 0 is often called “fire and forget” and provides the same guarantee as the underlying TCP protocol.

![](https://i.imgur.com/Z3LhFIG.png)

### QoS 1 - at least once

sender會保留message直到收到 **PUBACK** packet，否則會一直重傳。

收到**PUBACK** packet也就代表至少收到一次message。

> QoS level 1 guarantees that a message is delivered at least one time to the receiver. The sender stores the message until it gets a  PUBACK packet from the receiver that acknowledges receipt of the message. It is possible for a message to be sent or delivered multiple times.

![](https://i.imgur.com/px47YFf.png)

![](https://i.imgur.com/jGVemdj.png)

呼應前面章節提到的 DUP flag

> If the publishing client sends the message again it sets a duplicate (DUP) flag. In QoS 1, this DUP flag is only used for internal purposes and is not processed by broker or client. The receiver of the message sends a PUBACK, regardless of the DUP flag.

### QoS 2 - exactly once

最安全但也是速度最慢的QoS level

必須產生2次request/response

> QoS 2 is the safest and slowest quality of service level. The guarantee is provided by at least two request/response flows (a four-part handshake) between the sender and the receiver.

![QoS2](https://i.imgur.com/0j8xWev.png)

![PUBREC](https://i.imgur.com/5lEyi4M.png)

sender收到PUBREC packet之後,丟棄PUBLISH packet,並保存PUBREC packet,並回傳 PUBREL packet

> Once the sender receives a PUBREC packet from the receiver, the sender can safely discard the initial PUBLISH packet. The sender stores the PUBREC packet from the receiver and responds with a  PUBREL packet.

![PUBREL](https://i.imgur.com/3zY68N0.png)

> After the receiver gets the PUBREL packet, it can discard all stored states and answer with a PUBCOMP packet (the same is true when the sender receives the PUBCOMP). Until the receiver completes processing and sends the PUBCOMP packet back to the sender, the receiver stores a reference to the packet identifier of the original PUBLISH packet. This step is important to avoid processing the message a second time. After the sender receives the PUBCOMP packet, the packet identifier of the published message becomes available for reuse.

![PUBCOMP](https://i.imgur.com/Cjq9SF6.png)

> If a packet gets lost along the way, the sender is responsible to retransmit the message within a reasonable amount of time. This is equally true if the sender is an MQTT client or an MQTT broker. The recipient has the responsibility to respond to each command message accordingly.

### Best Practice

#### Use QoS 0 when …

1. 穩定的網路
2. 不在意訊息遺失
3. 不需要message queue

#### Use QoS 1 when …

> QoS level 1 is the most frequently used service level because it guarantees the message arrives at least once but allows for multiple deliveries.

#### Use QoS 2 when …

如果重複的message讓sub收到會造成影響。

### Queuing of QoS 1 and 2 messages

> All messages sent with QoS 1 and 2 are queued for offline clients until the client is available again. However, this queuing is only possible if the client has a persistent session.

## Part 7: Persistent Session and Queuing Messages

## Part 8: Retained Messages

## Part 9: Last Will and Testament

## Part 10: Keep Alive & Client Take-Over
