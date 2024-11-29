
# Docker 네트워크 통신과 관련된 자세한 개념

Docker 네트워크는 컨테이너 간의 통신, 외부 네트워크와의 연결을 가능하게 하며, 네트워크 프로토콜 및 표준을 활용하여 이를 구현합니다. 이 문서는 Docker 네트워크 통신을 다음 네 가지 챕터로 나누어 설명합니다:
1. Docker 컨테이너 내부에서의 통신
2. 한 서버 내부에서 서로 다른 컨테이너 간의 통신
3. 다른 서버에 각각 구성된 컨테이너 간의 통신
4. 컨테이너와 외부 인터넷과의 통신

---

## 1. Docker 컨테이너 내부에서의 통신

### 개요
- 컨테이너 내부에서는 `localhost` 또는 `127.0.0.1` 인터페이스를 사용하여 통신합니다.
- **로컬 통신만 가능**하며, 동일한 컨테이너 내부에서 실행 중인 프로세스들 간의 통신을 지원합니다.

### 네트워크 프로토콜 관점
- **Loopback 인터페이스**: `127.0.0.1`로 알려진 루프백 인터페이스는 데이터가 외부 네트워크로 나가지 않고 자체적으로 처리됩니다.
- TCP/UDP와 같은 표준 프로토콜을 사용하여 동일 컨테이너 내에서 애플리케이션과 서비스가 통신합니다.

### 설정 방법
- 예: 컨테이너 내부에서 실행 중인 웹 서버가 포트 `8080`에서 동작할 경우:
  ```shell
  curl http://localhost:8080
  ```

---

## 2. 한 서버 내부에서 서로 다른 컨테이너 간의 통신

### 개요
- 동일한 Docker 호스트에서 실행 중인 컨테이너는 기본적으로 **bridge 네트워크**를 통해 통신합니다.
- 각 컨테이너는 고유한 IP를 할당받으며, 컨테이너 이름 또는 IP 주소로 통신할 수 있습니다.

### 네트워크 프로토콜 관점
1. **ARP(Address Resolution Protocol)**:
   - 컨테이너 간 통신 시, 컨테이너의 IP 주소를 MAC 주소로 변환하여 데이터 패킷을 전달합니다.
   - 예: 컨테이너 A가 컨테이너 B에 요청 시, ARP를 통해 컨테이너 B의 MAC 주소를 확인합니다.
   
2. **브리지 네트워크**:
   - 기본 Docker 네트워크 드라이버이며, 가상 이더넷 브리지를 통해 컨테이너를 연결합니다.
   - 브리지 네트워크에서 컨테이너 간 데이터 전송은 **OSI 2계층(Layer 2)**에서 이루어집니다.

### 설정 방법
1. 기본 bridge 네트워크 사용:
   ```shell
   docker network inspect bridge
   ```

2. 사용자 정의 네트워크 생성:
   ```shell
   docker network create my-network
   docker run --name container-a --network my-network my-image
   docker run --name container-b --network my-network my-image
   ```

3. 컨테이너 간 통신:
   - 이름 기반 접근:
     ```shell
     curl http://container-b:8080
     ```
   - IP 기반 접근:
     ```shell
     curl http://<컨테이너_B_IP>:8080
     ```

---

## 3. 다른 서버에 각각 구성된 컨테이너 간의 통신

### 개요
- 서로 다른 서버에서 실행 중인 컨테이너는 **서버 간 네트워크 연결**을 통해 통신합니다.
- 포트 포워딩 또는 오케스트레이션 도구(Docker Swarm, Kubernetes)를 사용하여 통신을 설정합니다.

### 네트워크 프로토콜 관점
1. **NAT(Network Address Translation)**:
   - 외부 서버 간 통신 시, Docker는 NAT를 사용하여 컨테이너의 내부 IP를 호스트 서버의 공인 IP로 변환합니다.
   - 예: `docker run -p 8080:80`은 컨테이너의 내부 포트 `80`을 호스트의 `8080` 포트와 매핑합니다.

2. **라우팅**:
   - 서버 간 트래픽은 라우터를 통해 전달되며, 목적지 IP 주소를 기반으로 패킷이 전송됩니다.

### 설정 방법
1. 포트 포워딩을 사용하여 외부 통신 설정:
   ```shell
   docker run -d -p 8080:80 my-image
   curl http://<서버_1_IP>:8080
   ```

2. Docker Swarm 또는 Kubernetes 사용:
   - Swarm:
     ```shell
     docker swarm init
     docker service create --name web -p 8080:80 my-image
     ```
   - Kubernetes:
     ```shell
     kubectl expose deployment my-app --type=LoadBalancer --port=8080
     ```

---

## 4. 컨테이너와 외부 인터넷과의 통신

### 개요
- 컨테이너는 기본적으로 호스트의 네트워크를 통해 인터넷과 통신합니다.
- Docker는 **NAT**와 **포트 포워딩**을 통해 외부 트래픽을 관리합니다.

### 네트워크 프로토콜 관점
1. **DNS(Domain Name System)**:
   - Docker는 컨테이너의 네트워크 설정에서 DNS 서버를 자동으로 할당하여 이름 기반 요청을 처리합니다.
   - `/etc/resolv.conf` 파일을 통해 DNS 설정을 확인할 수 있습니다.

2. **HTTP/HTTPS**:
   - 컨테이너 내부 애플리케이션은 HTTP 또는 HTTPS 프로토콜을 사용하여 외부 API와 통신합니다.

### 설정 방법
1. 인터넷 연결 테스트:
   ```shell
   docker run --rm busybox ping -c 4 google.com
   ```

2. 외부 포트 노출:
   ```shell
   docker run -d -p 8080:80 my-image
   ```

3. 네트워크 문제 해결:
   - Docker 네트워크 설정 재시작:
     ```shell
     sudo systemctl restart docker
     docker network inspect bridge
     ```

---

## 5. 결론

Docker 네트워크는 ARP, NAT, DNS와 같은 네트워크 프로토콜을 활용하여 컨테이너 간 및 외부와의 통신을 효율적으로 처리합니다. 네트워크 개념을 이해하면 Docker 환경에서 더욱 안정적이고 효율적인 통신 구성을 구현할 수 있습니다.
