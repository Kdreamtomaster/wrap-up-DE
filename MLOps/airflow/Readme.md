# MLOps 관점에서 Airflow를 사용하는 이유

Apache Airflow는 데이터 파이프라인 관리뿐만 아니라 MLOps 워크플로우에서 중요한 역할을 수행할 수 있는 도구입니다. 이 문서에서는 **MLOps 관점에서** Airflow를 사용하는 이유를 정리합니다.

---

## 1. MLOps 관점에서 Airflow를 사용하는 이유

### 1.1 모델 라이프사이클 관리
Airflow는 머신러닝 모델의 전체 라이프사이클을 관리하는 데 효과적입니다.
- **데이터 수집부터 배포까지의 단계 정의**: 데이터 전처리, 모델 학습, 모델 평가, 배포 단계를 연결하여 전체 워크플로우 자동화 가능.
- **재현 가능한 모델 실행**: 동일한 DAG(Directed Acyclic Graph)를 통해 동일한 워크플로우를 반복 실행 가능.
- **의존성 관리**: 모델 학습 단계와 배포 단계 간의 의존성을 명확히 정의하여, 각 단계가 독립적이면서도 순차적으로 실행되도록 보장.

### 1.2 작업 자동화
Airflow는 모델 학습과 관련된 반복적인 작업을 자동화합니다.
- **모델 학습 주기 설정**: 스케줄링 기능을 통해 정기적으로 모델 학습 파이프라인 실행 가능.
- **모니터링 및 재실행**: 실패한 작업을 재시도하거나 특정 단계부터 재실행 가능.

### 1.3 다양한 툴 및 라이브러리 통합
Airflow는 다양한 머신러닝 프레임워크 및 툴과 쉽게 통합됩니다.
- **데이터 준비**: Pandas, Spark 등으로 데이터를 전처리하는 작업을 통합.
- **모델 학습**: TensorFlow, PyTorch 등 ML 프레임워크로 학습 작업 실행.
- **모델 평가 및 배포**: Docker, Kubernetes, MLflow 등과 연동하여 평가 및 배포 작업 자동화.

### 1.4 스케줄링과 이벤트 기반 실행
Airflow는 머신러닝 모델의 스케줄링 및 이벤트 기반 실행을 지원합니다.
- **정기적인 모델 재학습**: 예를 들어, 매일 또는 매주 최신 데이터를 기반으로 모델 재학습.
- **이벤트 트리거**: 데이터 업데이트나 로그 수집 등의 이벤트를 기반으로 파이프라인 실행 가능.

### 1.5 확장성과 관리 용이성
Airflow는 대규모 MLOps 환경에서도 확장성과 관리 용이성을 제공합니다.
- **확장 가능한 클러스터 아키텍처**: Airflow는 Celery Executor를 통해 클러스터 확장이 가능하여 대규모 작업을 처리 가능.
- **작업 상태 시각화**: DAG 실행 상태를 시각적으로 모니터링하여 MLOps 워크플로우의 상태를 한눈에 확인.

---

## 2. 데이터 파이프라인 관점과의 차이점

### 데이터 파이프라인 관점
- 주로 **데이터 이동, 변환, 적재(ETL)**에 초점을 맞춤.
- 데이터 소스와 데이터 웨어하우스 간의 워크플로우 정의.
- 데이터 품질 검사 및 로그 추적과 같은 작업에 중점.

### MLOps 관점
- **모델 학습, 평가, 배포** 등 머신러닝 워크플로우 관리에 초점.
- 모델 라이프사이클을 중심으로 데이터 파이프라인을 포함한 종합적인 관리.
- 데이터 전처리, 하이퍼파라미터 튜닝, 모델 배포 등의 작업 자동화.

---

## 3. 적합한 사용 사례

### 3.1 MLOps 관점에서의 사용 사례
1. **정기적인 모델 재학습**  
   - 새로운 데이터가 추가될 때 모델을 자동으로 재학습하고, 성능 평가 후 배포.

2. **머신러닝 파이프라인 관리**  
   - 데이터 전처리 → 모델 학습 → 하이퍼파라미터 튜닝 → 모델 평가 → 배포 단계를 자동화.

3. **모델 모니터링 및 재배포**  
   - 프로덕션 환경의 모델 성능을 모니터링하고, 성능 저하 시 새로운 모델로 교체.

4. **ML 실험 추적 연동**  
   - MLflow와 통합하여 실험 결과를 저장하고, 최적 모델을 배포 파이프라인으로 연결.

---

## 4. 장점과 단점

### 장점
- **유연성**: 다양한 ML 툴과 통합 가능.
- **워크플로우 자동화**: 복잡한 ML 파이프라인을 DAG로 간단히 정의 가능.
- **스케줄링**: 모델 재학습 및 평가를 정기적으로 실행 가능.
- **확장성**: 대규모 MLOps 환경에서도 안정적으로 작동.

### 단점
- **초기 설정 복잡성**: Airflow를 설정하고 Kubernetes나 클라우드 서비스와 통합하는 데 시간이 걸릴 수 있음.
- **ML 전용 기능 부족**: Kubeflow와 같은 ML 전용 플랫폼에 비해 일부 ML 특화 기능이 부족.
- **실시간 처리 한계**: 주로 배치 기반으로 설계되어 실시간 처리에는 적합하지 않음.

---

## 5. 결론

MLOps 관점에서 Airflow는 모델 학습부터 배포까지의 워크플로우를 자동화하고 관리하는 데 적합합니다. Airflow는 강력한 스케줄링 및 작업 의존성 관리 기능을 제공하며, 다양한 도구와 통합되어 유연한 MLOps 환경을 지원합니다.  

특히, 데이터 준비와 모델 학습 워크플로우를 연결하여 실험과 배포 간의 프로세스를 간소화할 수 있는 점이 큰 장점입니다.