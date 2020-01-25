# Apache Kafka

## 소개

Apache Kafka는 **분산 스트리밍 플랫폼**입니다. 분산 스트리밍 플랫폼을 무엇을 의미하는 걸까요?

분산 스트리밍 플랫폼은 다음의 세가지 능력을 갖고 있습니다.

- 메세지 큐와 비슷하게 Stream의 자료에게 Publish 하거나 Subscribe 합니다.
- 시스템에 장애가 일어나면 자료들을 스트림에 저장하므로 장애내성이 있습니다.
- 이벤트가 일어나면 즉시 처리합니다.

카프가는 일반적으로 두 개의 광범위한 클래스가 사용됩니다.

- 시스템 또는 애플리케이션간에 데이터를 안정적으로 가져 오는 실시간 스트리밍 데이터 파이프라인 구축
- 데이터를 변환하거나, 반응하는 실시간 스트리밍 애플리케이션 구축

이러한 것들을 카프카가 어떻게 하는지 알기 위해서는 카프카 바닥부터 뒤집어 보겠습니다.

- 카프카는 한개 또는 그 이상 클러스터를 구축해서 여러 개의 데이터센터를 가질 수 있습니다.
- 카프카 클러스터는 카테고리처럼 <u>topic</u>이라고 곳에 데이터들을 저장할 수 있습니다.
- 각각의 데이터들은 key와 value 그리고 timestamp로 이루어져 있습니다.

다음은 카프카의 4개의 주요 API들 입니다 

- [Producer API](https://kafka.apache.org/documentation.html#producerapi) 는 한개 또는 여러 개의 topic에 데이터를 publish(발행) 해줍니다.
- [Consumer API](https://kafka.apache.org/documentation.html#consumerapi) 는 한개 또는 여러 개의 topic 을 subscribe(구독)하고 그 데이터를 가져와 가공합니다.
- [Stream API](https://kafka.apache.org/documentation/streams) 는 stream 프로세서인데, 한개 또는 여러개의 topic에서 데이터를 가져와 가공한 다음 다시 stream에 넣습니다. 인풋 스트림에서 아웃풋 스트림으로 어떤걸 변환해야 할 때 효과적 입니다.
- [Connector API](https://kafka.apache.org/documentation.html#connect) 는 이미 있는 애플리케이션이나 DB에 접근하는 Kafka의 topic을 만들거나 실행할 때 재사용 가능한 producer나 consumer를 제공해줍니다. 예를들어 관계형 데이터베이스 커넥터는 테이블의 변화를 감지할 수 있습니다.

카프카에서 클라이언트와 서버는 **간단하며**, **높은 성능**과 **언어에 구애받지 않는 TCP 프로토콜**로 서로 통신 합니다. 

![kafka-apis](https://kafka.apache.org/24/images/kafka-apis.png)

## 토픽과 로그

첫번 째로 핵심적인 토픽을 살펴 보겠습니다.

토픽은 데이터가 발행되는 카테고리를 말합니다. 카프카에서 토픽은 항상 multi-subscribe 입니다. 즉 토픽은 0개나 1개 또는 그 이상의 데이터들을 구독(subscribe)를 하는 소비자(consumer)를 가질 수 있습니다.

각각의 토픽은 카프카 클러스터가 다음과 같이 생긴 파티션된 로그로 유지합니다.

![log_anatomy](https://kafka.apache.org/24/images/log_anatomy.png)

각각의 파티션은 정렬되있으며, 불변적인 데이터이며, 계속해서 데이터가 덧붙여집니다.  파티션에 있는 데이터는 파티션 내부에서 offset이라고 불리는 유니크한 식별자인 연속적인 id를 할당 받습니다.

카프카 클러스터 published 된 데이터들이 소비자에 의해 소비 됐던 말던 데이터들을 안전하게 보관합니다.(보관 기간은 설정 가능) 예를 들어 데이터 보관 기간을 2일로 정했다면, 2일동안 데이터가 살아 남아있고, 그 후에는 데이터가 사라집니다. 

![log_consumer](https://kafka.apache.org/24/images/log_consumer.png)

사실





실제로, 소비자(Consumer)에게 전달되는건 로그에 있는 소비자의 offset이나 위치의 메타데이터들 입니다.(In fact, the only metadata retained on a per-consumer basis is the offset or position of that consumer in the log.) 이 offset은 소비자가 제어하는데, 일반적으로 소비자가 데이터를 읽을 때 offset을 선형으로 진행합니다. 그러나 사실은, ㅇ

In fact, the only metadata retained on a per-consumer basis is the offset or position of that consumer in the log. This offset is controlled by the consumer: normally a consumer will advance its offset linearly as it reads records, but, in fact, since the position is controlled by the consumer it can consume records in any order it likes. For example a consumer can reset to an older offset to reprocess data from the past or skip ahead to the most recent record and start consuming from "now".

This combination of features means that Kafka consumers are very cheap—they can come and go without much impact on the cluster or on other consumers. For example, you can use our command line tools to "tail" the contents of any topic without changing what is consumed by any existing consumers.

The partitions in the log serve several purposes. First, they allow the log to scale beyond a size that will fit on a single server. Each individual partition must fit on the servers that host it, but a topic may have many partitions so it can handle an arbitrary amount of data. Second they act as the unit of parallelism—more on that in a bit.

## 분산

로그의 파티션들은 데이터를 핸들링하고 파티션의 공유를 요청하는 카프카 클러스터에게 전해집니다.  각각의 파티션은 장애내성을 위해 설정 값들을 복사합니다. 

각가의 파티션은 **"leader"** 처럼 행동하는 파티션을 하나씩 갖고 있고, 0개 또는 여러 개의 **followers** 를 갖고 있습니다. **leader**는 파티션에 관한 모든 읽기와 쓰기 요청을 담당하고, 반면에 **follower** 는 수동적으로 leader를 복사합니다. 만약 leader가 실패하면, follwer중 하나가 자동적으로 새로운 leader가 됩니다. 그래서 각각 서버는 클러스터안에 밸런스있게 리더와 팔로워가 공존하고 있습니다.

## Geo-Replication

카프카 MirrorMaker는 클러스터한테 geo-replication을 제공합니다. MirrorMaker를 사용하면 메세지는 여러 개의 데이터센터 또는 클라우드를 넘어 메세지가 복제 됩니다. 백업 또는 리커버리지에 관한 시나리오로 활성화하거나 수동적으로 사용할 수 있습니다. 

## Producers

Producers는 선택된 topic에게 데이터를 주는 역할을 합니다. (RX나 Reactor3에도 동일한 개념이 있습니다.)  생산자는 어떤 데이터를 어떤 파티션에 어떤 토픽에 넣을지에 대한 책임이 있습니다. 이런 것들은 로드밸런싱을 위해 라운드로빈을 사용하거나, 특정 파티션 함수 이용합니다. 

> 주로 저는 Producer를 한글로 생산자라고 말합니다.

## Consumers

Consumer는 그룹 이름으로 구별하고, 토픽으로 들어온 데이터는 알맞은 구독된 소비자 인스턴스에 전달 됩니다. 





## 참고자료

## 참고자료

[카프카 공식문서](https://kafka.apache.org/documentation/#introduction)