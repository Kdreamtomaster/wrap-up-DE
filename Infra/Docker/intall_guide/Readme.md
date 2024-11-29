
# Docker 설정 가이드 (한국어)

## Docker의 APT 저장소 설정

### 1. Docker의 공식 GPG 키 추가
Docker를 설치하기 전에 필요한 인증 키를 추가합니다.  
먼저 APT 패키지 목록을 업데이트하고 필요한 패키지를 설치합니다:
```shell
sudo apt-get update
sudo apt-get install ca-certificates curl
```

다음으로, GPG 키를 저장할 디렉터리를 생성합니다:
```shell
sudo install -m 0755 -d /etc/apt/keyrings
```

그 후 Docker의 GPG 키를 다운로드하여 저장소의 서명을 검증할 수 있도록 설정합니다:
```shell
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### 2. APT 소스에 Docker 저장소 추가
Docker 패키지를 다운로드할 수 있도록 Docker 저장소를 APT 소스에 추가합니다:
```shell
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

---

## Docker 패키지 설치 (최신 버전)
Docker 엔진, CLI, 컨테이너 런타임, 그리고 관련 플러그인을 설치합니다:
```shell
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## 현재 사용자를 Docker 그룹에 추가
Docker 명령어를 사용할 때 매번 `sudo`를 입력하지 않도록 현재 사용자를 Docker 그룹에 추가합니다:
```shell
sudo usermod -aG docker $USER
```

---

## +테스트
Docker 그룹에 추가된 후 적용을 바로 테스트하기 위해 그룹을 새로고침합니다:
```shell
newgrp docker
```

그리고 Docker 컨테이너를 확인하는 명령어를 실행합니다:
```shell
docker container ls
```

---

## Docker를 시스템 부팅 시 자동 시작하도록 설정
시스템이 재부팅될 때 Docker가 자동으로 실행되도록 설정합니다:
```shell
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

---

## 시스템 재부팅 및 확인
시스템을 재부팅한 후, Docker가 제대로 동작하는지 확인합니다:
```shell
sudo reboot
```

재부팅 후 다음 명령어로 컨테이너 리스트를 확인합니다:
```shell
docker container ls
```

이 가이드를 따라 하면 Docker를 성공적으로 설치하고 사용할 준비가 완료됩니다! 😊
