---
title: "Kafka 기본개념 및 생태계"
date: 2021-09-11T01:23:17+09:00
categories:
  - "kafka"
tags:
  - "kafka"
---




## Before Kafka

![](/image/kafka1.jpg)

- 엔드투엔드(end-to-end) 연결 방식의 아키텍처
- 데이터 연동의 복잡성 증가(하드웨어, 운영체제, 장애 등)
- 각기 다른 데이터 파이프라인 연결 구조
- 확장에 엄청난 노력 필요

모든 시스템으로 데이터를 전송 실시간 처리도 가능한 것
데이터가 갑자기 많아지더라도 확장이 용이한 시스템이 필요함

## After Kafka

![](/image/kafka2.jpg)

- 프로듀서/컨슈머 분리
- 메시지 데이터를 여러 컨슈머에게 허용
- 높은 처리량을 위한 메시지 최적화
- 스케일 아웃 가능
- 관련 생태계 제공

## Kafka broker

![](/image/kafka3.jpg)

- 실행된 카프카 애플리케이견 서버 중 1대
- 3대 이상의 브로커로 클러스터 구성
- 주키퍼와 연동
  - 주키퍼의 역할 : 메타데이터(브로커id, 컨트롤러id 등) 저장
- n개 브로커 중 1대는 컨트롤러(Controller)기능 수행
  - 컨트롤러 : 각 브로커에게 담당파티션 할당 수행. 브로커 정상 동작 모니터링 관리. 누가 컨트롤러 인지는 주키퍼에 저장.



            new ProducerRecord<String,String>("topic","key","message");


            ConsumerRecords<String,String> records = consumer.poll(1000);
            for (ConsumerRecord<String,String> record : records) {
            ...


- 객체를 프로듀서에서 컨슈머로 전달하기 위해 Kafka 내부에 byte 형태로 저장할 수 있도록
  직렬화/역직렬화하여 사용.

- 기본 제공 직렬화 class : StringSerializer, ShortSerializer 등
- 커스텀 직렬화 class를 통해 Custom Object 직렬화/역직렬화 가능



## Topic & Partition

![](/image/kafka4.jpg)

- 메세지 분류 단위
- n개의 파티션 할당 가능
- 각 파티션마다 고유한 오프셋(offset)을 가짐 old -> new
- 메시지 처리순서는 파티션 별로 유지 관리됨


![](/image/kafka5.jpg)


1. 프로듀서는 레코드를 생성하여 브로커로 전송
2. 전송된 레코드는 파티션에 신규 오프셋과 함께 기록됨
3. 컨슈머는 브로커로부터 레코드를 요청하여 가져감(polling)

## Kafka log and segment

- 실제로 메시지가 저장되는 파일시스템 단위
- 메시지가 저장될때는 세그먼트 파일이 열려있음
- 세그먼트는 시간 또는 크기 기주으로 닫힘
- 세그먼트가 닫힌 이후 일정 시간(또는 용량)에 따라 삭제(delete) 또는 압축(compact)

## 파티션 3개인 토픽과 컨슈머 1대

![](/image/kafka6.jpg)

- 파티션이 3개인 토픽
- 1개의 프로듀서가 토픽에 레코드를 보내는 중
- 1개의 컨슈머가 3개의 partition으로부터 polling 중

## 파티션 3개인 토픽과 컨슈머 3대

![](/image/kafka7.jpg)

- 파티션이 3개인 토픽
- 1개의 프로듀서가 토픽에 레코드를 보내는 중
- 3개의 컨슈머로 이루어진 1개의 컨슈머 그룹이 토픽으로부터 polling 중

컨슈머 한 대당 process하는 시간이 한정적이다. 컨슈머를 여러개를 해서 병렬처리 하도록 하면
각 파티션의 데이터를 빠른 속도로 처리가능하다

## 파티션이 3개인 토픽과 컨슈머 4대

- 가능한 경우 : 파티션 개수 >= 컨슈머 개수
- 불가능 : 파티션개수 < 컨슈머 개수
  - 남은 컨슈머는 파티션을 할당받지 못하고 대기 중

## 커슈머 3대 중 1대 장애 발생

![](/image/kafka8.jpg)

- 컨슈머 중 한개가 장애가 난 경우에 대한 대비 가능
- 리밸런스 발생 : 파티션 컨슈머 할당 재조정
- 나머지 컨슈머가 파티션으로부터 polling 수행

## 2개 이상의 컨슈머 그룹

![](/image/kafka9.jpg)

- 목적에 따른 컨슈머 그룹을 분리할 수 있음

장애에 대응하기 위해 재입수(또는 재처리) 목적으로
임시 신규 컨슈머 그룹을 생성하여 사용하기도 함

## 애플리케이션 로그 적재용 컨슈머 그룹 2개

![](/image/kafka10.jpg)

- Application log 적재 상황 Example
  - 엘라스틱서치 : 로그 실시간 확인용. 시간순 정렬
  - 하둡 : 대용량 데이터 적재. 이전 데이터 확인용

## 컨슈머 그룹 장애에 격리되는 다른 컨슈머 그룹

![](/image/kafka11.jpg)

- 컨슈머 그룹간 간섭(coupling) 줄임

- 하둡에 이슈가 발생하여 컨슈머의 적재지연이 발생하더라도 
  엘라스틱서치에 적재하는 컨슈머의 동작에는 이슈가 없음

## Broker partition replication

![](/image/kafka12.jpg)

$ bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic topic_name --partition 3 <- 파티션 개수

아래의 명령어는 카프카의 토픽을 생성하는 것이다. kafka-topics.sh 카프카 쉘 스크립트이다. 이것을 사용해서
카프카의 토픽을 생성하거나 list를 보거나 수정할 수 있다.
다음 옵션은 bootstrap-server인데 bootstrap.servers 설정은 클라이언트가 접근하는 토픽 파티션의 메타데이터를 요청하기 위한 설정입니다. 그렇기 때문에 브로커의 호스트/포트 정보의 리스트 형태의 값을 가집니다
개별 브로커는 클러스터 전체 데이터를 가지고 있지 않습니다. 그렇기 때문에 클라이언트는 브로커와 연결하여 브로커 내부의 자원에 접근하기 위해, 클라이언트가 접근하고자 하는 자원의 위치를 알아야 함.
클라이언트는 브로커 호스트 localhost:9092 부터 메타데이터를 요청

## Broker partition replication

![](/image/kafka13.jpg)

Broker #1이 복구되기 전까지 partition #1 사용 불가

Question : Kafka broker 이슈에 대응하기 위해 사용할 수 있는 방법은?

Answer : Partition을 다른 Broker에 복제하여 이슈에 대응한다.
            1번 Broker에 이슈가 생기면 다른 Broker에 복제된 데이터를 사용한다.

## Broker partition replication 설정 in kafka-topics

![](/image/kafka14.jpg)


        $ bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic topic_name --partition 3 --replication-factor 3

        고 가용성을 위한 파티션 복제 기능으로 데이터 유실 방지

## 리더 파티션, 팔로워 파티션

![](/image/kafka15.jpg)

- 리더 파티션 : Kafka 클라이언트(Producer, Consumer)와 데이터를 주고 받는 역할
- 팔로워 파티션 : 리더 파티션으로부터 레코드를 지속 복제(복제하는 시간이 걸림).
                      리더 파티션의 동작이 불가능할 경우 나머지 팔로워 중 1개가 리더로 선출됨

## ISR, 리더와 팔로워의 싱크

![](/image/kafka16.jpg)

- 파티션 3개, 레플리케이션이 3개로 이루어진 토픽이 브로커에 할당된 모습
- 특정 파티션의 리더, 팔로워가 레코드가 모두 복제되어 sync가 맞는 상태
  -> ISR(In-Sync Replica)
- ISR이 아닌 상태에서 장애가 나면 -> unclean.leader.election.enable

## Broker partition replication

![](/image/kafka17.jpg)

- Broker #1 장애 발생 시
  - partition #1의 리더가 브로커 1 또는 2 중에 새로 할당
  - Kafka 클라이언트는 새로운 파티션 리더와 연동

## Kafka rack-awareness

![](/image/kafka18.jpg)

- 1개의 Rack에 다수의 브로커를 몰아 넣는 것은 위험

- 다수의 Rack에 분산하여 브로커 옵션(broker.rack) 설정 및 배치
  - 파티션 할당 및 레플리케이션 동작시 특정 브로커에 몰리는 현상 방지

## 왜 카프카 클러스터는 서버 장애에 대응한 로직이 많나요?
- 서비스 운영에 있어 장애 허용(Fault-tolerant)은 아주 중요
  - 서버의 중단(이슈발생, 재시작) 언제든 발생할 수 있음
     - Example) 30대 브로커로 이루어진 카프카 클러스터가 있을 때 1대의 서버가
       365일 중 1일 중단이 발생할 가능성이 있다고 가정하면 12.1일(약 2주)에 한번씩 브로커 이슈 발생!!

- 일부 서버가 중단되더라도 데이터가 유시되면 안됨
  - 안정성이 보장되지 않으면 신뢰도가 하락(사용 중단)

## Kafka의 핵심요소 중간정리

- Broker : 카프카 애플리케이션 서버 단위
- Topic : 데이터 분리 단위, 다수 파티션 보유
- Partition : 레코드를 담고 있음. 컨슈머 요청 시 레코드 전달
- Offset : 각 레코드당 파티션에 할당도니 고유 번호
- Consumer : 레코드를 polling 하는 애플리케이션
  - Consumer group : 다수 컨슈머 묶음
  - Consumer offset : 특정 컨슈머가 가져간 레코드의 번호
- Producer : 레코드를 브로커로 전송하는 애플리케이션
- Replication : 파티션 복제 기능
  - ISR : 리더 + 팔로워 파티션의 sync가 된 묶음
- Rack-awareness : Server rack 이슈에 대응

## Kafka Client

![](/image/kafka19.jpg)

- Kafka와 데이터를 주고받기 위해 사용하는 Java Library
- Producer, Consumer, Admin, Stream 등 Kafka 관련 api 제공
- 다양한 3rd party library 존재 : C/C++, Node.js, Pythom, .Net 등
- Kafka broker 버전과 client 버젼 하위호환 확인 필요

## Kafka Streams

![](/image/kafka20.jpg)

- 데이터를 변환(Transformation)하기 위한 목적으로 사용하는 API
- 스트림 프로세싱을 지원하기 위한 다양한 기능을 제공
  - Stateful 또는 Stateless 와 같이 상태기반 스트림 처리 가능
  - Stream api와 DSL(Domain Specific Language)를 동시 지원
  - Exactly-once 처리, 고 가용성 특징
  - Kafka security(acl, sasl 등) 완벽 지원
  - 스트림 처리를 위한 별도 클러스터(ex, yam 등) 불필요

## Kafka Connect

![](/image/kafka21.jpg)


- 많은 경우 Kafka client로 Kafka로 데이터를 넣는 코드를 작성할때도 있지만,
  Kafka connect를 통해 data를 Import/Export 할 수 있음
- 코드 없이 configuration으로 데이터를 이동시키는 것이 목적
  - Standalone mode, distribution mode 지원
  - REST api interface를 통해 제어
  - Stream 또는 Batch 형태로 데이터 전송 가능
  - 커스텀 connector을 통한 다양한 plugin 제공(File, S3, Hive, Mysql etc..)

## Kafka Mirror maker

![](/image/kafka22.jpg)

- 특정 카프카 클러스터에서 다른 카프카 클러스터로 Topic 및 Record를 복제하는 Stanalone tool
- 2019년 11월, 기존 MirrorMaker를 개선한 MirrorMaker2.0 release
- 클러스터간 토픽에 대한 모든 것을 복제하는 것이 목적
  - 신규 토픽, 파티션 감지기능 및 토픽 설정 자동 Sync 기능
  - 양방향 클러스터 토픽 복제
  - 미러링 모니터링을 위한 다양한 metric(latency, count 등) 제공

## 그 외 Kafka 생태계를 지탱하는 application들

- confluent/ksqlDB : sql 구문을 토한 stream data processing 지원
- confluent/Schema Registry : avro 기반의 스키마 저장소
- confluent/REST Proxy : REST api를 통한 consumer/producer
- linkedin/Kafka burrow : consumer lag 수집 및 분석
- yahoo/CMAK : 카프카 클러스터 매니저
- uber/uReplicator : 카프카 클러스터 간 토픽 복제(전달)
- Spark stream : 다양한 소스(카프카 포함)로 부터 실시간 데이터 처리



### 출처 : SKplanet Tacademy, https://www.youtube.com/watch?v=VJKZvOASvUA