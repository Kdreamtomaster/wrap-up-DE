# Docker Setting

## Set up Docker's apt repository.

### Add Docker's official GPG key:
```shell
sudo apt-get update
sudo apt-get install ca-certificates curl
```

```
sudo install -m 0755 -d /etc/apt/keyrings
```

```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

```
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Add the repository to Apt sources:

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

## Install the Docker packages (Latest Version)

```shell
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


## Add your user to the docker group.
```
sudo usermod -aG docker $USER
```

## +For test

```
newgrp docker
```

```
docker container ls
```

## Configure Docker to start on boot with systemd

```shell
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```
## Reboot and check

```
sudo reboot

```

```
docker container ls
```

