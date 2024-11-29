# 아래 내용은 아직 초안입니다. 


# Kubernetes 클러스터 구축 가이드 (Control-plane 1개 + Worker Node 2개)

이 가이드는 Ubuntu 환경에서 Kubernetes 클러스터를 구축하는 과정을 설명합니다. Control-plane(마스터) 노드 1개와 Worker 노드 2개로 구성된 클러스터를 구축합니다.

---

## 1. 준비 사항

1. Ubuntu 기반 서버 3대:
   - Control-plane 노드 1대
   - Worker 노드 2대
2. 각 서버에서 `sudo` 권한 필요.
3. 각 노드의 고정 IP 설정.

---

## 2. 모든 노드에서 사전 설정

### 2.1 Swap 비활성화
```shell
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### 2.2 필요한 패키지 설치
```shell
sudo apt-get update
sudo apt-get install -y apt-transport-https curl
```

### 2.3 Docker 설치 및 설정
1. Docker 설치:
   ```shell
   sudo apt-get install -y docker.io
   ```
2. Docker가 Kubernetes에서 제대로 동작하도록 설정:
   ```shell
   cat <<EOF | sudo tee /etc/docker/daemon.json
   {
     "exec-opts": ["native.cgroupdriver=systemd"],
     "cgroup-driver": "systemd"
   }
   EOF
   sudo systemctl restart docker
   ```

### 2.4 Kubernetes 패키지 설치
1. Kubernetes의 GPG 키 추가:
   ```shell
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   ```
2. Kubernetes 패키지 저장소 추가:
   ```shell
   cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
   deb https://apt.kubernetes.io/ kubernetes-xenial main
   EOF
   sudo apt-get update
   ```
3. Kubernetes 패키지 설치:
   ```shell
   sudo apt-get install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

---

## 3. Control-plane 노드 설정

### 3.1 클러스터 초기화
```shell
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

### 3.2 kubectl 설정
1. 관리자 사용자에게 kubectl 설정 복사:
   ```shell
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```
2. 설정 확인:
   ```shell
   kubectl get nodes
   ```

### 3.3 Pod 네트워크 추가
Calico 네트워크 플러그인 설치:
```shell
kubectl apply -f https://docs.projectcalico.org/v3.25/manifests/calico.yaml
```

---

## 4. Worker 노드 추가

### 4.1 Control-plane에서 Join 명령어 가져오기
Control-plane에서 다음 명령어를 실행하여 Worker 노드에 사용할 Join 명령어를 확인:
```shell
kubeadm token create --print-join-command
```

### 4.2 Worker 노드에서 Join 명령어 실행
각 Worker 노드에서 Control-plane에서 복사한 Join 명령어 실행:
```shell
sudo kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN>     --discovery-token-ca-cert-hash sha256:<HASH>
```

---

## 5. 클러스터 상태 확인

### 5.1 모든 노드가 준비 상태인지 확인
Control-plane에서 다음 명령어를 실행:
```shell
kubectl get nodes
```

출력 예시:
```
NAME           STATUS   ROLES           AGE   VERSION
control-plane  Ready    control-plane   5m    v1.28.0
worker-node-1  Ready    <none>          3m    v1.28.0
worker-node-2  Ready    <none>          3m    v1.28.0
```

---

## 6. 추가 설정 (선택 사항)

### 6.1 Kubernetes 대시보드 설치
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

### 6.2 대시보드 액세스 토큰 생성
```shell
kubectl -n kubernetes-dashboard create token admin-user
```

---

## 7. 클러스터 유지 관리 팁

1. 노드 상태 확인:
   ```shell
   kubectl get nodes
   ```
2. Pod 상태 확인:
   ```shell
   kubectl get pods -A
   ```
3. 특정 리소스 로그 확인:
   ```shell
   kubectl logs <POD_NAME> -n <NAMESPACE>
   ```

---

이 가이드를 따라 Control-plane 1개와 Worker 노드 2개로 구성된 Kubernetes 클러스터를 성공적으로 구축할 수 있습니다.
