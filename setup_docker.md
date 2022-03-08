# Docker 설치

서버 설치 후 부터 도커 설치 후 컨테이너에서 gpu를 사용할 수 있게 하기까지의 과정입니다.

- 참고 자료

  [docker + GPU](https://wolfzone.tistory.com/31)
  
  
## SSH 설치

```bash
$ sudo apt-get update
$ sudo apt upgrade
$ sudo apt-get install ssh
$ sudo service ssh start


# SSH 제대로 실행 중인지 확인
$ sudo service ssh status


# 서버 부팅시 SSH 자동실행
$ sudo systemctl enable ssh.service
```


## NVIDIA driver 설치
RTX30** 시리즈는 450+ driver만 지원한다는 썰이 있습니다.
```bash
# recommand된 드라이버 확인
$ sudo ubuntu-drivers devices


# 드라이버 설치
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt-cache search nvidia | grep [드라이버 이름]
$ sudo apt-get install [드라이버 이름]


# 재부팅
$ sudo reboot


# 드라이브 설치된 것 확인
$ nvidia-smi


# 드라이버 제거 (드라이버 설치 잘못 됐거나 버전 맞지 않아 재설치 할 경우)
$ sudo apt-get --purge -y remove 'nvidia*'
```


## CUDA Toolkit 설치

```bash
# network 설치 
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
$ sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
$ sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
$ sudo add-apt-repository "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/ /"
$ sudo apt-get update
$ sudo apt-get -y install cuda


# 다운 후 런파일로 설치
$ wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda_10.2.89_440.33.01_linux.run
$ sudo sh cuda_10.2.89_440.33.01_linux.run


# 설치 확인
$ cat /usr/local/cuda/version.txt
# CUDA 설치 이후 nvidia-smi가 실행되지 않을 경우, 서버 GUI에서 update-manager를 실행하여 프로그램 업그레이드


# CUDA 제거 (CUDA 설치 잘못 됐거나 버전 맞지 않아 재설치 할 경우)
$ cd /usr/local/cuda/
$ sudo ./bin/cuda-uninstaller
$ sudo apt-get --purge -y remove 'cuda*'
$ sudo apt-get autoremove —purge cuda
$ cd /usr/local/
$ sudo rm -rf cuda*
```


## Docker 설치

- curl 설치 (안 되어 있을 시)

```bash
$ sudo apt-get update
$ sudo apt-get install curl
```

- Docker 설치

```bash
$ curl -fsSL https://get.docker.com/ | sudo sh
```

- TroubleShooting (위 방법으로 설치가 되지 않을 때)

```bash
# 이전 버전 삭제
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

```bash
# prepare
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
$ sudo apt-get update

# Docker official GPG key 추가
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 결과: OK

# docker repository 추가
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# 설치
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

# 설치 확인
$ sudo docker run hello-world
```

## DockerGroup에 사용자 추가

sudo를 붙이지 않고 docker를 사용하는 방법에는 root 권한을 부여하는 것과 DockerGroup에 사용자 추가하는 것, 두가지가 있다. 아래는 $USER 사용자를 DockerGroup에 추가하는 명령어이다.

```bash
$ sudo usermod -aG docker $USER 

# 재부팅해야 적용된다 한다.
$ sudo reboot
```


## nvidia-docker2 설치

container 내부에서 nvidia-driver 사용하기 위함.

```bash
# 레포지토리 추가
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# 설치 후 docker 데몬 로드
$ sudo apt-get update
$ sudo apt-get install -y nvidia-docker2
$ sudo pkill -SIGHUP dockerd

# 재부팅
$ sudo reboot now

# 잘 설치되었는지 확인
$ docker run --runtime=nvidia --rm nvidia/cuda:10.1-base nvidia-smi
```

--runtime 옵션을 주지 않아도 GPU를 사용할 수 있도록 설정 추가

```bash
$ vi /etc/docker/daemon.json
```
아래의 맨 윗 줄 ("default-runtime": "nvidia",) 추가
```json
{
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia":{
      "path": "/usr/bin/nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}
```
도커 재시작후 잘 되는지 확인
```bash
$ sudo service docker restart
$ docker run --rm nvidia/cuda:10.1-base nvidia-smi
```

## docker-compose 설치

```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```


## ppa로 설치하느 과정에서 오류가 생겼을 때

```bash
$ sudo apt-get install ppa-purge
$ ppa-purge ppa:[whatever]/ppa
$ sudo apt-get purge [package_name]
```


