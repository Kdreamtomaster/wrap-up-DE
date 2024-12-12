
# Kubernetes 클러스터 구축 가이드 (Control-plane 1개 + Worker Node 2개)

이 가이드는 **Ubuntu 22.04 LTS** 환경에서 Kubernetes 클러스터를 구축하는 과정을 설명합니다. Control-plane(마스터) 노드 1대와 Worker 노드 2대로 구성된 클러스터를 구축하며, Docker 대신 Containerd를 컨테이너 런타임으로 사용합니다.

---

## 1. 환경 준비

### 1.1 요구사항
1. **서버 구성**:
   - Control-plane: 1대
   - Worker 노드: 2대
2. **고정 IP 설정**:
   - 모든 노드에 고정 IP를 할당.
3. **방화벽 규칙**:
   - Kubernetes 통신을 위해 아래 포트를 열어야 합니다:
     - Control-plane: 6443, 2379-2380, 10250-10259
     - Worker 노드: 10250, 30000-32767

---

## 2. 모든 노드에서 사전 작업

### 2.1 Swap 비활성화
Kubernetes는 Swap 활성화를 지원하지 않습니다:
\`\`\`bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
\`\`\`

### 2.2 필요한 패키지 설치
\`\`\`bash
sudo apt-get update
sudo apt-get install -y apt-transport-https curl ca-certificates gnupg
\`\`\`

### 2.3 Containerd 설치 및 설정
1. **Containerd 설치**:
   \`\`\`bash
   sudo apt-get install -y containerd
   sudo mkdir -p /etc/containerd
   containerd config default | sudo tee /etc/containerd/config.toml
   \`\`\`

2. **Containerd Systemd 설정**:
   `/etc/containerd/config.toml` 파일의 `SystemdCgroup` 설정을 활성화:
   \`\`\`bash
   sudo vi /etc/containerd/config.toml
   \`\`\`
   아래 내용을 수정:
   \`\`\`toml
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
     SystemdCgroup = true
   \`\`\`

3. **서비스 재시작**:
   \`\`\`bash
   sudo systemctl restart containerd
   sudo systemctl enable containerd
   \`\`\`

### 2.4 Kubernetes 설치
1. **GPG 키 추가 및 저장소 설정**:
   \`\`\`bash
   sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
   sudo apt-get update
   \`\`\`

2. **Kubeadm, Kubelet, Kubectl 설치**:
   \`\`\`bash
   sudo apt-get install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   \`\`\`

---

## 3. Control-plane 노드 설정

### 3.1 클러스터 초기화
Control-plane의 IP 주소를 지정하여 클러스터를 초기화:
\`\`\`bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<CONTROL_PLANE_IP> --cri-socket unix:///run/containerd/containerd.sock
\`\`\`

### 3.2 kubectl 설정
1. 관리자를 위해 `admin.conf` 파일 설정:
   \`\`\`bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   \`\`\`

2. 초기화 확인:
   \`\`\`bash
   kubectl get nodes
   \`\`\`

### 3.3 네트워크 플러그인 설치
Calico를 설치하여 Pod 네트워크를 구성:
\`\`\`bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
\`\`\`

---

## 4. Worker 노드 추가

### 4.1 Join 명령어 생성
Control-plane에서 Worker 노드 연결을 위한 명령어를 생성:
\`\`\`bash
kubeadm token create --print-join-command
\`\`\`

### 4.2 Worker 노드에서 Join 명령어 실행
각 Worker 노드에서 Control-plane에서 복사한 명령어 실행:
\`\`\`bash
sudo kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> \
    --discovery-token-ca-cert-hash sha256:<HASH> \
    --cri-socket unix:///run/containerd/containerd.sock
\`\`\`

---

## 5. 클러스터 상태 확인

Control-plane에서 모든 노드가 Ready 상태인지 확인:
\`\`\`bash
kubectl get nodes
\`\`\`

**출력 예시**:
\`\`\`
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   5m    v1.31.3
worker-01      Ready    <none>          3m    v1.31.3
worker-02      Ready    <none>          3m    v1.31.3
\`\`\`

---

## 6. 클러스터 테스트

### 6.1 샘플 애플리케이션 배포
Nginx를 배포하여 클러스터 상태를 확인합니다:
\`\`\`bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --type=NodePort --port=80
\`\`\`

### 6.2 서비스 확인
NodePort로 노출된 Nginx 서비스의 상태를 확인:
\`\`\`bash
kubectl get svc
\`\`\`
출력된 `NodePort`를 통해 클러스터 외부에서 접근:
\`\`\`bash
http://<노드IP>:<NodePort>
\`\`\`

---

## 7. 추가 설정 (선택 사항)

### 7.1 Kubernetes 대시보드 설치
\`\`\`bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
\`\`\`

### 7.2 대시보드 토큰 생성
\`\`\`bash
kubectl -n kubernetes-dashboard create token admin-user
\`\`\`

### 7.3 대시보드 접속
\`\`\`bash
kubectl proxy
\`\`\`
브라우저에서 [http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/)로 접속.

---

## 8. 유지 관리 팁

1. **노드 상태 확인**:
   \`\`\`bash
   kubectl get nodes
   \`\`\`

2. **Pod 상태 확인**:
   \`\`\`bash
   kubectl get pods -A
   \`\`\`

3. **Pod 로그 확인**:
   \`\`\`bash
   kubectl logs <POD_NAME> -n <NAMESPACE>
   \`\`\`

---

이 가이드는 Kubernetes 클러스터를 성공적으로 설정하고 관리하는 데 필요한 모든 단계를 포함합니다. 질문이 있다면 언제든 문의하세요! 🚀
