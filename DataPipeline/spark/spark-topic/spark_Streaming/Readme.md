
# 데이터 파이프라인 구축 관점에서 Spark Structured Streaming

Apache Spark Structured Streaming은 실시간 데이터 처리 프레임워크로, 데이터 파이프라인 구축 시 실시간 데이터를 효율적으로 처리하는 데 사용됩니다.  
이 문서에서는 Structured Streaming의 개념, 이전 버전인 Spark Streaming과의 차이점, 그리고 배치 데이터 처리에서 사용되는 Spark Submit과의 차이를 설명합니다.

---

## 1. Spark Structured Streaming이란?

- **정의**: Structured Streaming은 Spark SQL API 위에서 동작하며, 실시간 데이터를 **DataFrame** 또는 **Dataset** 형식으로 처리합니다.
- **주요 특징**:
  1. 선언적 API 사용: SQL 쿼리 또는 DataFrame/Dataset 연산으로 스트리밍 작업 정의.
  2. 정확한 처리 보장: **Exactly-Once** 처리 모델 지원.
  3. 배치-스트리밍 통합: 배치 및 스트리밍 작업 간의 API 일관성 제공.
  4. 이벤트 시간 처리: 데이터 도착 시간과 상관없이 이벤트 타임 기반 연산 가능.

---

## 2. Spark Streaming과 Structured Streaming의 차이점

| **특징**                  | **Spark Streaming (Deprecated)**                  | **Spark Structured Streaming**              |
|---------------------------|---------------------------------------------------|---------------------------------------------|
| **API**                  | RDD 기반, DStream 사용                              | DataFrame 및 Dataset 기반                   |
| **처리 모델**            | 마이크로 배치 처리 (Mini Batch)                     | 마이크로 배치와 연속 처리 (Continuous Processing) 지원 |
| **구현 복잡성**          | 낮음 (RDD 수준에서 연산)                           | 높음 (상태 저장 처리, 이벤트 시간 지원)       |
| **Fault Tolerance**       | 기본적으로 지원                                   | Exactly-Once 보장                           |
| **성능**                 | 상대적으로 낮음                                    | 최적화된 Catalyst 엔진 사용, 높은 성능       |
| **Deprecated**           | Spark 3.0 이상에서 사용 중단                      | 최신 Spark 버전에서 권장                   |

---

### Spark Streaming의 한계
1. **DStream의 제한**:
   - RDD 기반으로 구성되어 고급 데이터 연산의 표현력이 부족.
2. **실시간 처리 성능**:
   - Catalyst 엔진과 통합되지 않아 처리 성능이 떨어짐.
3. **상태 저장 작업**:
   - 상태 저장 연산(예: 세션 윈도우)이 제한적.

Structured Streaming은 이러한 한계를 극복하고 배치와 스트리밍 작업 간의 경계를 허물기 위해 설계되었습니다.

---

## 3. 배치 데이터 처리와 Spark Submit의 차이점

### Spark Structured Streaming과 Spark Submit의 사용 비교

| **특징**                  | **Spark Submit (배치 데이터)**                  | **Spark Structured Streaming (실시간 데이터)** |
|---------------------------|-------------------------------------------------|-----------------------------------------------|
| **처리 방식**            | 고정된 데이터셋을 한 번에 처리                     | 지속적으로 들어오는 데이터를 처리             |
| **입력 소스**            | HDFS, S3, RDBMS 등                                | Kafka, Socket, File Stream 등                 |
| **출력 소스**            | HDFS, S3, RDBMS 등                                | Kafka, Console, Memory 등                     |
| **트리거**               | 없음 (정적인 배치 작업)                          | **Trigger**를 사용해 주기적 또는 연속적 처리 |
| **제어 및 실행**         | 단발성 실행 (Spark Submit 명령어)                 | 스트리밍 애플리케이션으로 지속 실행           |

---

### Spark Submit의 배치 처리 예시
```bash
spark-submit   --class com.example.MyBatchJob   --master yarn   --deploy-mode cluster   my-batch-job.jar
```
- **설명**:
  - 고정된 데이터셋(HDFS, S3 등)에 대해 실행.
  - 배치 작업은 한 번 실행되며 종료.

---

### Structured Streaming 처리 예시
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder     .appName("StructuredStreamingExample")     .getOrCreate()

# 실시간 데이터 소스 (Kafka 예시)
df = spark.readStream     .format("kafka")     .option("kafka.bootstrap.servers", "localhost:9092")     .option("subscribe", "topic_name")     .load()

# 데이터 변환
query = df.selectExpr("CAST(value AS STRING)")     .writeStream     .format("console")     .start()

query.awaitTermination()
```
- **설명**:
  - 스트리밍 소스(Kafka 등)에서 데이터를 실시간으로 읽어 처리.
  - 애플리케이션은 종료되지 않고 지속적으로 실행.

---

## 4. Spark Structured Streaming의 주요 활용 사례

1. **실시간 데이터 처리**:
   - Kafka, MQTT, Socket 등에서 데이터 수집 후 실시간 분석 및 처리.
2. **로그 분석**:
   - 로그 데이터를 실시간으로 수집하고 집계 및 이상 탐지.
3. **ETL 파이프라인**:
   - 실시간 ETL(추출, 변환, 적재) 작업 수행.
4. **이벤트 기반 아키텍처**:
   - 사용자 이벤트(클릭, 구매 등)를 실시간으로 분석하여 인사이트 제공.

---

## 5. 결론

Spark Structured Streaming은 실시간 데이터 처리를 위한 강력하고 유연한 도구입니다.  
배치 처리에 최적화된 Spark Submit과 비교해 지속적이고 실시간으로 데이터를 처리하며, 기존의 Spark Streaming보다 향상된 성능과 기능을 제공합니다.  
Structured Streaming은 데이터 파이프라인의 실시간 분석과 처리 요구사항을 만족시키는 데 중요한 역할을 합니다.
