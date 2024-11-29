
# Kubeflow DAG 작성법 및 모델 등록, 추론 엔드포인트 서빙 및 접근 방법

Kubeflow Pipelines는 머신러닝 워크플로우를 정의하고 실행할 수 있는 플랫폼으로, 워크플로우는 DAG(Directed Acyclic Graph)로 구성됩니다.  
또한 학습된 모델을 등록하고 추론을 위한 엔드포인트로 서빙할 수 있으며, HTTP 요청을 통해 추론 엔드포인트에 접근할 수 있습니다. 이 문서에서는 DAG 작성법과 함께 모델 등록, 서빙, 그리고 엔드포인트 접근 방법에 대해 설명합니다.

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

---

## 3. 모델 등록 및 추론 엔드포인트 서빙

### 3.1 모델 등록

Kubeflow는 학습된 모델을 저장하고 관리하기 위해 모델 레지스트리를 제공합니다.

#### 모델 등록을 위한 Component 작성:
```python
@component
def register_model_op(model_path: str, model_name: str):
    import kfp
    # 모델 등록 로직 작성
    print(f"Registering model: {model_name} at {model_path}")
```

---

### 3.2 추론 엔드포인트 서빙

Kubeflow Serving(KServe)을 사용하여 모델을 엔드포인트로 서빙합니다.

#### 서빙을 위한 Component 작성:
```python
@component
def deploy_model_op(model_name: str, version: str):
    import requests
    # KServe 배포 로직 작성
    print(f"Deploying model: {model_name}, version: {version}")
```

---

### 3.3 추론 엔드포인트 접근

#### 엔드포인트 요청을 위한 Component 작성:
```python
@component
def predict_op(endpoint: str, input_data: dict) -> dict:
    import requests
    response = requests.post(endpoint, json=input_data)
    return response.json()
```

#### 예제 엔드포인트 호출:
- 엔드포인트 URL: `http://example-model.default.svc.cluster.local/v1/models/example-model:predict`
- 입력 데이터(JSON 형식):
  ```json
  {
      "instances": [[1.0, 2.0, 3.0]]
  }
  ```

#### Python 코드로 추론 요청:
```python
import requests

endpoint = "http://example-model.default.svc.cluster.local/v1/models/example-model:predict"
input_data = {"instances": [[1.0, 2.0, 3.0]]}

response = requests.post(endpoint, json=input_data)
print("Prediction:", response.json())
```

---

## 4. 데이터 전처리 → 모델 학습 → 등록 → 서빙 → 추론 요청 DAG 예제

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
def register_model_op(model_path: str, model_name: str):
    print(f"Registering model: {model_name} at {model_path}")

@component
def deploy_model_op(model_name: str, version: str):
    print(f"Deploying model: {model_name}, version: {version}")

@component
def predict_op(endpoint: str, input_data: dict) -> dict:
    import requests
    response = requests.post(endpoint, json=input_data)
    return response.json()

@pipeline(name="ml_pipeline_with_serving_and_inference")
def ml_pipeline_with_serving_and_inference(data_path: str, model_name: str, version: str, input_data: dict):
    preprocess_task = preprocess_op(data_path=data_path)
    train_task = train_op(processed_data_path=preprocess_task.output)
    register_task = register_model_op(
        model_path=train_task.output, model_name=model_name
    )
    deploy_task = deploy_model_op(
        model_name=model_name, version=version
    )
    predict_task = predict_op(
        endpoint=f"http://{model_name}.default.svc.cluster.local/v1/models/{model_name}:predict",
        input_data=input_data
    )
```

---

## 5. Kubeflow Pipeline 실행

### 5.1 파이프라인 컴파일
Kubeflow 파이프라인을 실행하기 전에 컴파일해야 합니다.
```python
from kfp.v2 import compiler

compiler.Compiler().compile(
    pipeline_func=ml_pipeline_with_serving_and_inference,
    package_path="ml_pipeline_with_serving_and_inference.json"
)
```

### 5.2 Kubeflow Pipelines UI에 업로드
1. Kubeflow Pipelines 대시보드에 접속.
2. **Upload Pipeline** 버튼 클릭.
3. `ml_pipeline_with_serving_and_inference.json` 파일을 업로드하여 실행.

---

## 6. 결론

Kubeflow Pipelines와 KServe를 활용하면 데이터 전처리, 모델 학습, 등록, 서빙, 추론 요청까지 하나의 통합된 워크플로우를 구현할 수 있습니다.  
HTTP 요청을 통해 추론 엔드포인트에 접근할 수 있으며, 이를 통해 모델을 실시간 서비스로 제공할 수 있습니다.
