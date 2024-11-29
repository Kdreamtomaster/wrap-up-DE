
# Kubernetes Pod 개념, 메니페스토 예시, 로그 확인 및 트러블슈팅 가이드

Pod는 Kubernetes의 기본 배포 단위로, 컨테이너화된 애플리케이션의 실행 환경을 제공합니다. 이 문서에서는 Pod의 개념, 예시 메니페스토 파일, 로그 확인 방법, 그리고 자주 발생하는 문제와 해결 방법을 설명합니다.

---

## 1. Pod란 무엇인가?

- Pod는 하나 이상의 컨테이너와 스토리지, 네트워크, 컨테이너 실행 명세를 포함하는 Kubernetes의 최소 배포 단위입니다.
- Pod는 동일한 네트워크 네임스페이스와 IP를 공유하므로 컨테이너 간 통신이 용이합니다.
- Pod는 일반적으로 단일 컨테이너를 실행하지만, 긴밀히 협력하는 다중 컨테이너를 포함할 수도 있습니다.

---

## 2. Pod 예시 메니페스토 파일

### 2.1 단일 컨테이너 Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: single-container-pod
  namespace: default
spec:
  containers:
    - name: nginx-container
      image: nginx:1.23
      ports:
        - containerPort: 80
```

> **각주**:
> - `metadata.name`: Pod의 이름.
> - `spec.containers`: Pod에 포함된 컨테이너 정의.
> - `image`: 실행할 컨테이너 이미지.
> - `ports`: 컨테이너에서 노출하는 포트.

---

### 2.2 다중 컨테이너 Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
  namespace: default
spec:
  containers:
    - name: app-container
      image: my-app:1.0
      ports:
        - containerPort: 8080
    - name: sidecar-container
      image: log-collector:1.0
```

> **각주**:
> - Pod 내 모든 컨테이너는 동일한 네트워크와 볼륨을 공유.
> - `sidecar-container`: 주 컨테이너의 로그 또는 데이터를 처리하는 보조 컨테이너.

---

### 2.3 볼륨을 사용하는 Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
    - name: app-container
      image: my-app:1.0
      volumeMounts:
        - name: app-storage
          mountPath: /data
  volumes:
    - name: app-storage
      emptyDir: {}
```

> **각주**:
> - `volumeMounts`: 컨테이너에서 볼륨을 마운트하는 경로.
> - `emptyDir`: Pod의 생애 동안 임시 데이터를 저장하는 볼륨.

---

## 3. Pod 로그 확인 방법

Pod의 상태 및 로그는 다음 명령어로 확인할 수 있습니다:

### 3.1 Pod 상태 확인
```bash
kubectl get pods
```

출력 예:
```
NAME                     READY   STATUS    RESTARTS   AGE
single-container-pod     1/1     Running   0          10m
multi-container-pod      2/2     Running   0          8m
```

### 3.2 Pod 상세 정보 확인
```bash
kubectl describe pod <POD_NAME>
```

### 3.3 Pod 로그 확인
- 단일 컨테이너 Pod:
  ```bash
  kubectl logs <POD_NAME>
  ```
- 다중 컨테이너 Pod:
  ```bash
  kubectl logs <POD_NAME> -c <CONTAINER_NAME>
  ```

---

## 4. 자주 발생하는 문제와 트러블슈팅

### 4.1 Pod이 `Pending` 상태에 멈춰 있을 때
- **원인**: 스케줄링 리소스 부족.
- **해결 방법**:
  - `kubectl describe pod <POD_NAME>`로 이벤트 확인.
  - 노드 리소스를 확인하여 부족한 경우 클러스터 크기 확장.

---

### 4.2 Pod이 `CrashLoopBackOff` 상태일 때
- **원인**: 컨테이너가 시작되지 않거나 반복적으로 충돌.
- **해결 방법**:
  1. `kubectl logs <POD_NAME>` 명령어로 오류 확인.
  2. 애플리케이션 오류 수정 후 이미지 다시 빌드 및 배포.

---

### 4.3 Pod이 네트워크에 접근하지 못할 때
- **원인**: 네트워크 정책 또는 DNS 문제.
- **해결 방법**:
  - `kubectl describe pod <POD_NAME>`에서 네트워크 이벤트 확인.
  - 클러스터 DNS 확인:
    ```bash
    kubectl exec -it <POD_NAME> -- nslookup google.com
    ```

---

### 4.4 Pod이 `ImagePullBackOff` 상태일 때
- **원인**: 잘못된 이미지 이름 또는 이미지 레지스트리에 인증 실패.
- **해결 방법**:
  1. 이미지 이름이 올바른지 확인.
  2. 프라이빗 이미지 레지스트리의 경우 `imagePullSecrets` 추가:
     ```yaml
     spec:
       imagePullSecrets:
       - name: my-registry-secret
     ```

---

### 4.5 Pod 종료 또는 삭제 문제
- **원인**: `finalizer` 설정 또는 디펜던시 리소스.
- **해결 방법**:
  - `kubectl describe pod <POD_NAME>`에서 `finalizer` 확인 후 수동 제거:
    ```bash
    kubectl patch pod <POD_NAME> -p '{"metadata":{"finalizers":null}}'
    ```

---

## 5. 결론

Pod는 Kubernetes 워크로드의 기본 단위로, 안정적인 애플리케이션 실행 환경을 제공합니다.  
Pod의 로그 및 상태를 확인하고, 자주 발생하는 문제에 대한 해결 방법을 숙지하면 Kubernetes 클러스터를 효율적으로 관리할 수 있습니다.
