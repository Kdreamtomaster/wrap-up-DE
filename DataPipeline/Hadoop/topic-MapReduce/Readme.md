
# 데이터 파이프라인 관점에서 Hadoop MapReduce

Hadoop MapReduce는 대규모 데이터 처리에 사용되는 프로그래밍 모델이자 분산 처리 프레임워크입니다. 데이터 파이프라인에서 MapReduce는 데이터를 병렬로 처리하여 효율성을 극대화합니다. 이 문서에서는 데이터 파이프라인 관점에서 Hadoop MapReduce의 역할과 이를 사용하는 이유를 설명합니다.

---

## 1. Hadoop MapReduce란 무엇인가?

Hadoop MapReduce는 분산 환경에서 대량의 데이터를 처리하기 위한 **맵(Map)과 리듀스(Reduce)** 작업으로 구성된 프로그래밍 모델입니다.
- **Map** 단계: 데이터를 키-값 쌍으로 변환하고, 같은 키를 가진 데이터를 그룹화.
- **Reduce** 단계: 그룹화된 데이터를 합산, 집계, 필터링 등의 방식으로 처리.

---

## 2. 데이터 파이프라인에서 MapReduce의 역할

### 2.1 데이터 변환
- 데이터 파이프라인에서 원시 데이터를 구조화된 데이터로 변환하는 데 사용됩니다.
- 예: 로그 데이터를 분석 가능한 형태로 변환.

### 2.2 데이터 집계
- 대규모 데이터셋에서 합계, 평균, 최대값/최소값 등의 통계적 정보를 계산.
- 예: 사용자 활동 로그에서 월간 사용자 수를 계산.

### 2.3 데이터 필터링
- 특정 조건에 맞는 데이터만 필터링하여 하위 파이프라인으로 전달.
- 예: 로그 데이터에서 오류 메시지만 추출.

### 2.4 분산 데이터 처리
- 데이터를 여러 노드에서 병렬로 처리하여 처리 속도를 크게 향상.
- 데이터 파이프라인의 대량 데이터 처리 요구 사항에 적합.

---

## 3. MapReduce의 작동 방식

### 3.1 데이터 흐름
1. **Input Split**: 데이터를 작은 조각으로 분할.
2. **Map**: 각 데이터 조각에 대해 키-값 쌍을 생성.
3. **Shuffle and Sort**: 같은 키를 가진 데이터를 그룹화하고 정렬.
4. **Reduce**: 그룹화된 데이터를 기반으로 집계, 변환 등의 작업 수행.
5. **Output**: 최종 결과를 파일 시스템(HDFS)에 저장.

### 3.2 예제: 단어 빈도 계산
- 입력 데이터: 텍스트 파일
- Map 단계: 각 단어와 `1`의 쌍 생성. 예: `("word", 1)`
- Reduce 단계: 같은 단어를 그룹화하고 빈도를 합산. 예: `("word", 5)`

---

## 4. 데이터 파이프라인에서 MapReduce를 사용하는 이유

### 4.1 대규모 데이터 처리
- 수십 테라바이트(TB) 이상의 데이터를 병렬로 처리 가능.
- 수평 확장성을 제공하여 데이터 크기 증가에 유연하게 대응.

### 4.2 내결함성
- 작업 실패 시 자동으로 재시도하며, 중간 결과를 저장하여 데이터 손실 방지.

### 4.3 비용 효율성
- 상용 하드웨어를 사용하여 클러스터를 구성하므로 경제적.
- 오픈소스 프레임워크로 라이선스 비용 없음.

### 4.4 HDFS와 통합
- HDFS에 저장된 데이터를 효율적으로 처리하도록 최적화.
- 데이터가 저장된 노드에서 직접 작업을 수행하여 네트워크 부하를 줄임.

### 4.5 유연한 데이터 처리
- 구조적, 반구조적, 비구조적 데이터를 모두 처리 가능.
- JSON, CSV, 텍스트 로그 등 다양한 데이터 형식을 지원.

---

## 5. 데이터 파이프라인에서의 주요 사용 사례

### 5.1 ETL(추출, 변환, 적재)
- 대규모 데이터셋을 처리하여 데이터 웨어하우스로 적재.
- 예: 원시 로그 데이터를 분석 가능한 데이터로 변환.

### 5.2 로그 분석
- 웹 로그 데이터를 분석하여 트래픽, 오류 빈도 등의 정보를 추출.
- 예: 특정 시간대별 사용자 요청 수 계산.

### 5.3 사용자 행동 분석
- 전자상거래 데이터에서 사용자 행동 패턴 분석.
- 예: 가장 많이 조회된 상품 카테고리 식별.

### 5.4 금융 데이터 처리
- 대규모 트랜잭션 데이터를 분석하여 통계 및 위험 관리 수행.

---

## 6. MapReduce의 장점과 단점

### 6.1 장점
1. **대규모 데이터 처리**: 병렬 처리와 분산 환경에서 높은 성능 제공.
2. **내결함성**: 작업 실패 시 재시도 및 데이터 복구 가능.
3. **확장성**: 노드를 추가하여 처리 용량을 쉽게 확장 가능.
4. **유연성**: 다양한 데이터 형식을 처리할 수 있음.

### 6.2 단점
1. **실시간 처리 한계**: 배치 처리 모델로 설계되어 실시간 작업에는 부적합.
2. **복잡한 프로그래밍**: 맵과 리듀스 작업을 정의하기 위한 프로그래밍이 복잡할 수 있음.
3. **I/O 과다**: 중간 데이터를 디스크에 저장하므로 처리 속도가 느릴 수 있음.

---

## 7. 결론

Hadoop MapReduce는 데이터 파이프라인에서 대규모 데이터를 효율적으로 처리하고 변환하는 데 중요한 역할을 합니다.  
특히 ETL, 로그 분석, 사용자 행동 분석과 같은 작업에서 병렬 처리를 통해 데이터 파이프라인의 성능을 크게 향상시킬 수 있습니다.  
다만, 실시간 처리가 필요한 경우에는 Spark와 같은 대안과 병행하여 사용하는 것이 효과적일 수 있습니다.