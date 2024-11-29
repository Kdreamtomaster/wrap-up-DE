
# Docker 초심자를 위한 기본 개념

Docker는 컨테이너 기술을 활용하여 애플리케이션과 그 환경을 가볍고 이식 가능한 단위로 패키징하는 플랫폼입니다. 아래는 Docker의 주요 개념과 최신 명령어를 초심자의 관점에서 이해하기 쉽게 정리한 내용입니다.

---

## 1. Docker란 무엇인가?
- Docker는 애플리케이션과 필요한 환경(라이브러리, 의존성)을 **컨테이너**라는 단위로 패키징하여 어디서든 실행할 수 있도록 만들어주는 도구입니다.
- "내 로컬에서는 잘 되는데..."  와 같은 문제를 해결하기 위해 등장한 기술입니다.

---

## 2. Docker의 주요 개념

### 2.1 이미지 (Image)
- 컨테이너를 실행하기 위한 **청사진** 또는 **템플릿**입니다.
- 애플리케이션과 필요한 소프트웨어, 설정 등을 포함.
- 예: `python:3.10`, `nginx:latest` 등.
- 이미지는 변경할 수 없으며, 컨테이너를 만들기 위해 사용됩니다.

### 2.2 컨테이너 (Container)
- 이미지를 기반으로 실행되는 **가상 환경**입니다.
- 컨테이너는 실제 애플리케이션이 동작하는 독립적인 환경을 제공합니다.
- 경량화되어 있고, 빠르게 생성 및 삭제할 수 있습니다.

### 2.3 Dockerfile
- **이미지를 생성하기 위한 설정 파일**입니다.
- 애플리케이션 설치 과정, 필요한 라이브러리, 실행 명령어 등을 정의합니다.
- 예:
  ```dockerfile
  FROM python:3.10
  COPY app.py /app/app.py
  CMD ["python", "/app/app.py"]
  ```

### 2.4 레지스트리 (Registry)
- Docker 이미지를 저장하고 공유하는 **저장소**입니다.
- 주요 Docker 레지스트리:
  - **Docker Hub**: 기본 제공되는 공개 레지스트리.
  - **AWS ECR, GCP Artifact Registry**: 클라우드 환경의 프라이빗 레지스트리.

### 2.5 볼륨 (Volume)
- 컨테이너 내부 데이터와 호스트 시스템 데이터를 연결하기 위한 **저장소**입니다.
- 컨테이너 삭제 시 데이터를 유지하려면 볼륨을 사용합니다.
- 예: 데이터베이스 컨테이너의 데이터 유지.

### 2.6 네트워크 (Network)
- 컨테이너 간 또는 컨테이너와 외부 간의 통신을 관리합니다.
- 기본적으로 3가지 유형 제공:
  - **bridge**: 기본 네트워크. 컨테이너 간 통신 지원.
  - **host**: 컨테이너가 호스트의 네트워크를 공유.
  - **none**: 네트워크 연결 없음.

---

## 3. Docker의 최신 명령어 및 기본 동작

### 3.1 이미지 실행
- 이미지를 기반으로 컨테이너를 실행합니다.
  ```bash
  docker container run [이미지 이름]
  ```
  예:
  ```bash
  docker container run hello-world
  ```

### 3.2 컨테이너 실행 및 관리
- 컨테이너 실행:
  ```bash
  docker container start [컨테이너 이름 또는 ID]
  ```
- 실행 중인 컨테이너 확인:
  ```bash
  docker container ps
  ```
- 실행된 모든 컨테이너 확인:
  ```bash
  docker container ps -a
  ```
- 컨테이너 중지:
  ```bash
  docker container stop [컨테이너 이름 또는 ID]
  ```

### 3.3 이미지 생성
- Dockerfile을 사용하여 새로운 이미지를 생성:
  ```bash
  docker image build -t [이미지 이름] [Dockerfile 경로]
  ```
  예:
  ```bash
  docker image build -t my-python-app .
  ```

### 3.4 이미지 관리
- 이미지를 다운로드:
  ```bash
  docker image pull [이미지 이름]
  ```
- 로컬에 저장된 이미지 목록 확인:
  ```bash
  docker image ls
  ```
- 이미지 삭제:
  ```bash
  docker image rm [이미지 이름 또는 ID]
  ```

---

## 4. Docker Compose
- 여러 컨테이너를 하나의 설정 파일(`docker-compose.yml`)로 정의하고 관리합니다.
- 예제 `docker-compose.yml` 파일:
  ```yaml
  version: '3'
  services:
    web:
      image: nginx
      ports:
        - "8080:80"
    app:
      build: .
      volumes:
        - ./app:/app
  ```
- 실행 명령:
  ```bash
  docker compose up
  ```

---

## 5. Docker를 사용하는 이유

1. **재현 가능한 환경 제공**: 어디서든 동일한 환경에서 애플리케이션 실행 가능.
2. **경량화**: 가상 머신보다 가볍고 빠르게 실행 가능.
3. **이식성**: 다양한 플랫폼에서 컨테이너 실행 가능.
4. **확장성**: 여러 컨테이너를 오케스트레이션 도구(Kubernetes 등)와 결합하여 확장 가능.
5. **개발-운영 격차 해소**: 개발 환경과 배포 환경의 차이 문제 해결.

---

## 6. Docker 초심자를 위한 주요 명령어 요약

| 명령어                             | 설명                                                                 |
|------------------------------------|----------------------------------------------------------------------|
| `docker image pull [이미지 이름]`  | Docker Hub에서 이미지를 다운로드.                                     |
| `docker container run [이미지 이름]` | 이미지를 기반으로 컨테이너 실행.                                     |
| `docker container ps`              | 실행 중인 컨테이너 목록 확인.                                        |
| `docker container ps -a`           | 모든 컨테이너 목록 확인 (종료된 컨테이너 포함).                       |
| `docker container stop [컨테이너 ID]` | 실행 중인 컨테이너 중지.                                             |
| `docker container rm [컨테이너 ID]` | 종료된 컨테이너 삭제.                                                |
| `docker image ls`                  | 로컬에 저장된 이미지 목록 확인.                                      |
| `docker image rm [이미지 ID]`      | 이미지 삭제.                                                        |
| `docker image build`               | Dockerfile을 사용해 새 이미지 생성.                                  |
| `docker compose up`                | `docker-compose.yml`에 정의된 서비스 실행.                          |

---

## 7. 결론

Docker는 애플리케이션 실행 환경을 경량화하고 재현성을 보장하며, 초심자에게도 쉽게 사용할 수 있는 강력한 도구입니다. 최신 명령어와 개념을 이해하면 Docker를 활용한 기본 작업을 효과적으로 수행할 수 있습니다.
