
# Kubernetes Finalizer 개념 및 문제 해결

Finalizer는 Kubernetes 리소스가 삭제되기 전에 특정 작업을 완료해야 함을 지정하는 메타데이터 속성입니다.  
리소스를 삭제할 때 Finalizer가 설정되어 있으면, 지정된 작업(예: 리소스 정리 또는 외부 시스템에 대한 호출)이 완료되기 전까지 리소스 삭제가 지연됩니다.

---

## 1. Finalizer의 주요 특징

1. **리소스 삭제 보호**:
   - Finalizer는 리소스 삭제가 완료되기 전에 필요한 작업을 보장합니다.
   - 예: PVC(Persistent Volume Claim) 삭제 시, 관련 PV(Persistent Volume)도 함께 삭제되도록 설정.

2. **삭제 워크플로우 제어**:
   - Finalizer는 외부 리소스를 클린업하거나 동기화 작업을 수행하는 데 활용됩니다.

3. **메타데이터 설정**:
   - Finalizer는 리소스의 `metadata.finalizers` 필드에 문자열 배열로 설정됩니다.

---

## 2. Finalizer 설정 예시

Pod에 Finalizer를 설정한 예:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  finalizers:
    - example.com/my-finalizer
spec:
  containers:
    - name: nginx
      image: nginx
```

---

## 3. Finalizer로 인한 Pod 삭제 문제

### 3.1 문제 상황
- Finalizer가 설정된 리소스가 삭제되지 않고 `Terminating` 상태로 남아 있음.

### 3.2 원인
- Finalizer에서 지정된 작업이 완료되지 않음.
- Finalizer를 제거하지 못한 경우.

### 3.3 해결 방법
- `kubectl patch` 명령어로 Finalizer를 수동으로 제거:
  ```bash
  kubectl patch pod <POD_NAME> -p '{"metadata":{"finalizers":null}}'
  ```
- 삭제 후 Pod가 정상적으로 종료되었는지 확인:
  ```bash
  kubectl get pods
  ```

---

## 4. Finalizer 활용 사례

1. **PVC와 PV**:
   - PVC를 삭제할 때 Finalizer를 사용하여 연결된 PV가 자동으로 삭제되도록 보장.

2. **외부 시스템과 통합**:
   - 리소스 삭제 전에 외부 API 호출이나 로그 정리 작업 수행.

3. **리소스 정리**:
   - 네트워크 정책이나 ConfigMap과 같은 리소스 삭제 시 의존성을 클린업.

---

## 5. 결론

Finalizer는 Kubernetes 리소스의 삭제 과정을 안전하고 체계적으로 관리하기 위한 도구입니다.  
다만, 설정된 작업이 완료되지 않으면 리소스가 삭제되지 않으므로 이를 적절히 관리하는 것이 중요합니다.
