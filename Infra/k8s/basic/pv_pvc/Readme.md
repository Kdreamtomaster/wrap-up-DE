
# Kubernetes PV/PVC 개념 및 문제 해결

Kubernetes에서 Persistent Volume(PV)와 Persistent Volume Claim(PVC)은 클러스터 내에서 스토리지를 관리하고 사용하는 방법을 제공합니다.  
이 문서에서는 PV와 PVC의 개념, 메니페스토 예시, 로그 확인 방법, 자주 발생하는 문제와 해결 방법을 설명합니다.

---

## 1. PV와 PVC란?

### 1.1 Persistent Volume (PV)
- 클러스터 관리자가 프로비저닝한 스토리지 리소스.
- Pod에서 직접 참조하지 않고 PVC를 통해 사용.
- NFS, iSCSI, Ceph, AWS EBS 등 다양한 스토리지 제공자와 통합 가능.

### 1.2 Persistent Volume Claim (PVC)
- 개발자가 스토리지를 요청하는 리소스.
- PVC는 특정 크기와 접근 모드를 정의하며, PV와 바인딩됩니다.

---

## 2. PV/PVC 예시 메니페스토 파일

### 2.1 PV 예시
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

> **각주**:
> - `capacity`: PV의 크기.
> - `accessModes`: 접근 모드 (예: `ReadWriteOnce`, `ReadOnlyMany`).
> - `persistentVolumeReclaimPolicy`: PV가 바인딩 해제될 때 동작 (예: `Retain`, `Delete`, `Recycle`).
> - `hostPath`: 로컬 경로를 스토리지로 사용.

---

### 2.2 PVC 예시
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

> **각주**:
> - `accessModes`: PVC가 요청하는 접근 모드.
> - `resources.requests.storage`: PVC가 요청하는 스토리지 크기.

---

### 2.3 PVC를 사용하는 Pod 예시
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-pvc
spec:
  containers:
    - name: app-container
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: storage-volume
  volumes:
    - name: storage-volume
      persistentVolumeClaim:
        claimName: example-pvc
```

> **각주**:
> - `volumeMounts`: Pod의 컨테이너에 볼륨을 마운트.
> - `persistentVolumeClaim`: PVC를 참조하여 볼륨을 연결.

---

## 3. PV/PVC 상태 확인 및 로그 확인

### 3.1 PV 상태 확인
```bash
kubectl get pv
```

출력 예:
```
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   AGE
example-pv   1Gi        RWO            Retain           Bound    default/example-pvc   manual       10m
```

### 3.2 PVC 상태 확인
```bash
kubectl get pvc
```

출력 예:
```
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
example-pvc   Bound    example-pv   1Gi        RWO            manual         10m
```

### 3.3 상세 정보 확인
- PV 상세 정보:
  ```bash
  kubectl describe pv example-pv
  ```
- PVC 상세 정보:
  ```bash
  kubectl describe pvc example-pvc
  ```

---

## 4. 자주 발생하는 문제와 트러블슈팅

### 4.1 PVC가 `Pending` 상태일 때
- **원인**:
  - 요청한 PVC에 맞는 PV가 없음.
  - 스토리지 클래스가 제대로 설정되지 않음.
- **해결 방법**:
  1. PVC의 요청 사항 확인 (`kubectl describe pvc <PVC_NAME>`).
  2. PVC와 일치하는 PV를 생성하거나 동적 프로비저닝 활성화.

---

### 4.2 PV가 `Released` 상태일 때
- **원인**:
  - PV가 이전 PVC와 바인딩 해제되었지만 데이터가 여전히 남아 있음.
- **해결 방법**:
  1. PV의 `persistentVolumeReclaimPolicy`가 `Retain`일 경우 수동으로 삭제:
     ```bash
     kubectl delete pv <PV_NAME>
     ```
  2. PV를 초기화하거나 재사용.

---

### 4.3 Pod이 PVC를 마운트하지 못할 때
- **원인**:
  - PVC가 Pod의 요구사항과 일치하지 않음.
  - 스토리지 공급자의 문제.
- **해결 방법**:
  1. Pod 이벤트 확인:
     ```bash
     kubectl describe pod <POD_NAME>
     ```
  2. PVC 상태 확인:
     ```bash
     kubectl get pvc
     ```
  3. 스토리지 공급자 로그 확인 (예: NFS, AWS EBS).

---

### 4.4 PVC가 동적 프로비저닝에 실패할 때
- **원인**:
  1. PVC와 스토리지 클래스의 요청 조건이 일치하지 않음.
  2. 스토리지 클래스가 존재하지 않거나 잘못 설정됨.
  3. 스토리지 프로비저너가 비활성화 상태이거나 장애 발생.

- **해결 방법**:
  1. PVC와 스토리지 클래스 설정 확인:
     - PVC의 요청 조건과 `storageClassName`이 올바른지 확인:
       ```bash
       kubectl describe pvc <PVC_NAME>
       ```
     - PVC가 지정한 `storageClassName`이 올바른지 확인:
       ```bash
       kubectl get sc
       ```
  2. 스토리지 클래스 정의 점검:
     ```yaml
     apiVersion: storage.k8s.io/v1
     kind: StorageClass
     metadata:
       name: example-storage-class
     provisioner: kubernetes.io/aws-ebs
     parameters:
       type: gp2
     reclaimPolicy: Delete
     ```
  3. 프로비저너 로그 확인:
     - 스토리지 프로비저너 Pod 로그 확인:
       ```bash
       kubectl logs -n kube-system <PROVISIONER_POD>
       ```
  4. PVC 상태를 다시 확인하거나 삭제 후 재생성:
     ```bash
     kubectl delete pvc <PVC_NAME>
     kubectl apply -f pvc.yaml
     ```

---

## 5. 결론

Kubernetes의 PV와 PVC는 스토리지 관리의 핵심 개념으로, Pod에서 안정적으로 스토리지를 사용할 수 있도록 합니다.  
상태 확인, 로그 분석, 자주 발생하는 문제를 해결하는 방법을 익히면 Kubernetes 스토리지를 효율적으로 관리할 수 있습니다.
