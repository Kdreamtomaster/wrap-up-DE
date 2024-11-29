
# Kubeflow DAG 작성법

Kubeflow Pipelines는 머신러닝 워크플로우를 정의하고 실행할 수 있는 플랫폼으로, 워크플로우는 DAG(Directed Acyclic Graph)로 구성됩니다.  
이 문서에서는 Kubeflow Pipelines에서 DAG를 작성하는 방법과 기본 구조를 설명합니다.

---

## 1. Kubeflow DAG의 기본 구조

Kubeflow DAG는 Python 코드를 사용하여 정의되며, 각 단계는 독립적인 작업(Component)으로 구성됩니다.

### 주요 구성 요소:
1. **Component**: 각 DAG의 노드에 해당하며, 특정 작업(예: 데이터 처리, 모델 학습)을 수행합니다.
2. **Pipeline**: DAG 전체를 정의하는 함수로, 각 Component 간의 관계를 설정합니다.
3. **Input/Output**: Component 간 데이터 전달을 정의합니다.

---

## 2. Kubeflow DAG 작성 단계

### 2.1 Kubeflow Pipelines SDK 설치
```bash
pip install kfp
```

### 2.2 Component 작성
Component는 Python 함수 또는 Docker 컨테이너로 정의됩니다.

#### Python 함수로 Component 정의:
```python
from kfp.v2.dsl import component

@component
def preprocess_op(data_path: str) -> str:
    # 데이터 전처리 로직
    processed_data_path = f"{data_path}/processed_data.csv"
    return processed_data_path
```

#### Docker 컨테이너로 Component 정의:
Component를 Docker 컨테이너로 정의하려면 YAML 파일로 설정합니다.
```yaml
name: preprocess
inputs:
  - {name: data_path, type: String}
outputs:
  - {name: processed_data_path, type: String}
implementation:
  container:
    image: my-docker-image
    command:
      - python
      - preprocess.py
```

### 2.3 Pipeline 작성
Pipeline은 Component 간의 관계를 정의하는 함수입니다.

```python
from kfp.v2.dsl import pipeline

@pipeline(name="my_pipeline")
def my_pipeline(data_path: str):
    preprocess_task = preprocess_op(data_path=data_path)
    # 추가 작업 연결 가능
```

---

## 3. Kubeflow DAG 작성 예제

### 데이터 전처리 → 모델 학습 → 평가 파이프라인 예제

```python
from kfp.v2.dsl import pipeline, component

@component
def preprocess_op(data_path: str) -> str:
    processed_data_path = f"{data_path}/processed_data.csv"
    return processed_data_path

@component
def train_op(processed_data_path: str) -> str:
    model_path = "/path/to/model.pkl"
    return model_path

@component
def evaluate_op(model_path: str, test_data_path: str) -> float:
    accuracy = 0.95  # 평가 로직
    return accuracy

@pipeline(name="ml_pipeline")
def ml_pipeline(data_path: str, test_data_path: str):
    preprocess_task = preprocess_op(data_path=data_path)
    train_task = train_op(processed_data_path=preprocess_task.output)
    evaluate_task = evaluate_op(
        model_path=train_task.output, test_data_path=test_data_path
    )
```

---

## 4. Kubeflow Pipeline 실행

### 4.1 파이프라인 컴파일
Kubeflow 파이프라인을 실행하기 전에 컴파일해야 합니다.
```python
from kfp.v2 import compiler

compiler.Compiler().compile(
    pipeline_func=ml_pipeline,
    package_path="ml_pipeline.json"
)
```

### 4.2 Kubeflow Pipelines UI에 업로드
1. Kubeflow Pipelines 대시보드에 접속.
2. **Upload Pipeline** 버튼 클릭.
3. `ml_pipeline.json` 파일을 업로드하여 실행.

---

## 5. 추가 설정

### 5.1 입력 매개변수
Pipeline 함수에 매개변수를 추가하여 사용자 입력을 받을 수 있습니다.
```python
@pipeline(name="parameterized_pipeline")
def parameterized_pipeline(data_path: str, num_epochs: int):
    preprocess_task = preprocess_op(data_path=data_path)
    train_task = train_op(processed_data_path=preprocess_task.output, num_epochs=num_epochs)
```

### 5.2 환경 변수 설정
Component 실행 시 필요한 환경 변수를 설정하려면 `container_op`에서 설정합니다.
```python
from kfp.dsl import ContainerOp

op = ContainerOp(
    name="example",
    image="my-image",
    command=["python", "script.py"],
)
op.container.set_env_variable("ENV_VAR", "value")
```

---

## 6. 결론

Kubeflow Pipelines를 사용하면 DAG 형태의 머신러닝 워크플로우를 손쉽게 정의하고 관리할 수 있습니다.  
Python SDK를 활용하여 Component와 Pipeline을 정의하고, Kubeflow UI를 통해 실행 및 모니터링할 수 있습니다.
