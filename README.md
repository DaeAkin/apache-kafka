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
- [Stream API](https://kafka.apache.org/documentation/streams) 는 





## 참고자료

[카프카 공식문서](https://kafka.apache.org/documentation/#introduction)