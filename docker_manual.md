# Docker 사용법

도커에 대한 개념은 [생활코딩](https://opentutorials.org/course/128/8657)에 잘 나와 있습니다.

## Image 관련

사용하려는 이미지를 [Docker Hub](https://hub.docker.com/)에 검색할 수 있다.

e.g) pytorch 검색 → pytorch/pytorch by Pytorch → Tags → 다양한 조합의 pytorch 버전 + CUDA 버전 + cudnn 버전 을 포함하는 이미지들을 찾을 수 있다.

```bash
# 이미지 받아오기
# 태그는 필수가 아니며, 기본 태그는 :latest이다.
# 이미지 식별자는 이미지 이름, ID 모두 가능
$ docker pull <이미지>:<태그>

# 이미지 목록 보기
$ docker images

# 이미지 삭제 (해당 이미지로 생성한 컨테이너가 존재하지 않을 때)
$ docker rmi <이미지>:<태그>
# 이미지와 그 이미지로 생성한 컨테이너 모두 삭제
$ docker rmi -f <이미지>
# 모든 이미지 삭제 (복구 불가)
$ docker rmi $(docker images -q)

# 빌드 오류 발생 시 생성되는 <none> images 찾기
$ docker images -f "dangling=true" -q
# <none> images 삭제
$ docker rmi $(docker images -f "dangling=true" -q)
# <none> images와 그 이미지로 생성한 컨테이너 모두 삭제
$ docker rmi -f $(docker images -f "dangling=true" -q)

# 이미지의 히스토리
$ docker history <이미지>
```

- example

```bash
# 파이토치 버전 - 쿠다 버전 - cudnn 버전 - base / runtime / devel
# 개발 환경에서는 devel을 많이 사용하지만 용량이 큼.
$ docker pull pytorch/pytorch:1.9.0-cuda11.1-cudnn8-devel
```
RTX30**의 경우 450번대 이상의 driver을 사용해야된다는 썰이 있습니다.

450번대 이상의 driver를 사용할 경우 11.0 버전 이상의 CUDA를 사용해야 한다고 합니다.

11.0 버전 이상의 CUDA를 사용할 경우 8.0 버전 이상의 cuDnn을 사용해야 한다고 합니다.

다른 버전으로 실험해보지는 않았는데 참고하면 좋을 것 같습니다.


## Container 관련

[컨테이너 생성 옵션]
|option|Description|
|------|-----------|
|--detach / -d|detached mode.  컨테이너가 백그라운드로 실행|
|--entrypoint|Dockerfile의 ENTRYPOINT 덮어쓰기|
|--env / -e|환경변수 설정. 보통 설정 값 / 비밀번호 전달|
|--gpus|컨테이너 내부에서 사용할 수 있는 GPU|
|--hostname|컨테이너의 호스트 이름 설정|
|--ipc|리소스 접근에 대한 관리|
|--iteractive / -it|표준 입력(stdin)을 활성화. 컨테이너와 연결(attach)되어 있지 않더라도 표준 입력을 유지. 보통 이 옵션을 사용하여 Bash에 명령을 입력.|
|--name / -n|컨테이너 이름 설정|
|--privileged|컨테이너 안에서 호스트의 리눅스 커널 기능(Capability) 모두 사용|
|--publish / -p|포트 포워딩. <호스트 포트>:<컨테이너 포트>. 컨테이너의 포트만 설정히면 호스트의 포트 번호가 무작위로 설정됨.|
|--rm|컨테이너 안의 프로세스가 종료되면 컨테이너 삭제. -d 옵션과 함께 사용 불가.|
|--tty / -t|TTY 모드(pseudo-TTY)를 사용. 이 옵션을 사용해야 셸 사용 가능|
|--volume / -v|볼륨 마운트. <호스트 경로>:<컨테이너 경로>|

```bash
# 이미지로 컨테이너 생성
$ docker run <옵션> <이미지>:<태그> <실행할 커맨드>

# 실행중인 컨테이너 목록
$ docker ps
# 모든 컨테이너 목록
$ docker ps -a
# 컨테이너 검색
# e.g) ancestor=pytorch/pytorch:latest로 최신 pytorch 이미지를 생성한 컨테이너 검색
$ docker ps -a --filter <옵션>

# 컨테이너 이름 변경
# 컨테이너 생성 과정에서 까먹고 --name 옵션 안 주었을 때 쓰면 좋다.
docker rename <old-name> <new-name>

# 정지된 컨테이너 시작
# 마찬가지로 컨테이너 식별자는 이미지 이름, ID 모두 가능
$ docker start <컨테이너>
# 실행중인 컨테이너 재시작
$ docker restart <컨테이너>
# 실행중인 컨테이너 접속
$ docker attach <컨테이너>
# 실행중인 컨테이너에 접속하면서 커맨드 실행
# -it 옵션 필수 (STDIN 표준 입출력을 열고 가상 tty를 통해 접속한다는 뜻) 
$ docker exec -it <컨테이너> <커맨드>

# 컨테이너 내부에서 컨테이너를 정지하지 않고 탈출
# 컨테이너 생성 과정에서 -it 옵션을 주고 생성했을 때만 가능
ctrl + P -> ctrl + Q
# 컨테이너 내부에서 컨테이너 정지하고 탈출
$ exit
or ctrl + D
# 컨테이너 외부에서 컨테이너 정지
$ docker stop <컨테이너>

# 컨테이너 삭제
$ docker rm <컨테이너>
# 모든 컨테이너 삭제
# $ 뒤의 검색 조건을 변경하여 원하는 조건의 컨테이너를 모두 삭제할 수 있다.
$ docker rm -f $(docker ps -aq)

# 컨테이너내부에서 외부로 파일 복사
$ docker cp <컨테이너>:<호경로>/<파일명> <호스트 경로>/
# 컨테이너 외부에서 컨테이너 내부로 파일 복사
docker cp <호스트 경로>/<파일명> <컨테이너>:<경로>/
# 파일명을 바꾸면서 복사
docker cp <호스트 경로>/<파일명> <컨테이너>:<경로>/<파일명>

# 도커의 로그 확인
# 컨테이너 생성 과정에서 에러 났을 때 확인하기 좋다.
$ docker logs <컨테이너>
# 컨테이너 커밋 (컨테이너를 이미지 파일로 생성)
$ docker commit <옵션> <컨테이너> <이미지>:<태그>
```

- example

```bash
# 첫줄은 보통 다 하는 듯
# 두번째 줄로 볼륨 마운트, 워킹 디렉토리 설정.
# 세번째 줄로 사용하고자하는 gpu 할당, 이름 설정, 이미지 지정.
$ docker run -d -it --ipc="host" \
-v /home/chkmook/:/home/chkmook/ -v /datasets/:/datasets/ -w /home/chkmook \
--gpus="device=0" --name chkmook pytorch/pytorch:1.9.0-cuda11.1-cudnn8-devel

# gpu여러개 쓸 경우 --gpus='"device=0,1"'

# run 했을 경우 container가 생성되고, 접속되어있지는 않다.

# 접속
$ docker attach chkmook
# 접속 후 bash 실행
$ docker exec -it chkmook /bin/bash
```

## 도커 허브

이미지 이름이 <Docker Hub 계정>/<이미지> 형식으로 지어진 경우에만 업로드 가능.

자신의 계정 이름과 일치하는 경우에만 이미지 업로드 가능.

```bash
# 도커 허브 로그인
$ docker login
Username: 
Password:
Email: 

# 이미지 업로드
# 태그 지정하지 않을 시 :latest
$ sudo docker push <Docker Hub 계정>/<이미지>:<태그>

# 다운로드 가능
$ docker pull <Docker Hub 계정>/<이미지>:<태그>
```

## Dockerfile

Dockerfile을 통해 베이스 이미지에 자주 사용하는 프로그램, 설정들을 포함시켜 새로운 이미지를 만들 수 있다.

많이 추가할 경우 이미지의 크기가 커진다. (devel 버전 pytorch+cuda+cudnn이 7~8GB, 이것저것 넣으면 20GB 가까이 나옵니다.)

서버를 옮길 때 docker push + pull을 통해 자주 사용하는 프로그램, 설정들을 한번에 옮길 수 있다는 장점이 있다.

- Dockerfile

```docker
# 베이스 이미지 지정
# 이미지 생성 과정에서 제일 먼저 실행. 한번만 써야 함
FROM <베이스 이미지>:<태그>

# 라벨 설정. 굳이 안해도 됨
LABEL name = <내 이름>
LABEL email = <내 이메일 주소>

# 환경 변수 설정
ENV <변수명> <값>

# 이미지 생성 중에 실행되는 명령어
# \ 이용해서 여러 줄로 쓸 수 있다.
# RUN은 여러번 써도 됨.
RUN <명령어>
RUN ["executable", "param1", "param2"]
# -y를 통해 프로그램 생성과정에서 나오는 모든 물음에 yes로 대답 
RUN apt-get install -y <프로그램명>

# 볼륨 마운트
# docker run 할 때 -v 옵션으로 설정 가능
VOLUME ["<경로1>", "<경로2>"]

# CMD에서 설정한 실행파일이 실행되는 위치
WORKDIR <경로>

# 컨테이너가 시작되었을 때 실행되는 명령어
# 하나만 실행 가능하며, 여러개 있을 경우 가장 아래에 있는 명령어만 실행
CMD <명령어>
CMD ["param1","param2"]

# 컨테이너가 실행중에 열리는 포트
EXPOSE 80
```

- Dockerfile → image

```bash
# dockerfile이 저장된 디렉터리에서 dockerfile 기반으로 이미지 생성
# 폴더 안에 Dockerfile으로 된 이름의 파일만 존재한다면 . 만 써서 가능
$ docker build --tag <이미지>:<태그> <Dockerfile 경로>

# 생성된 이미지로 컨테이너 생성 가능
# Dockerfile에서 컨테이너 생성시 실행되는 명령어를 적어두었기 때문에 쓸 필요 X
$ docker run <옵션> <이미지>:<태그>
```

- Dockerfile example

```docker
# 베이스 이미지 지정
FROM pytorch/pytorch:1.9.0-cuda11.1-cudnn8-devel
# 태그 설정
LABEL name = "chkmook"
LABEL email = "chkmook@yonsei.ac.kr"

# Update and basic install
RUN apt-get update && apt-get install -y \
      git \
      zsh \
      byobu \
      htop \
      curl \
      wget \
      locales \
      zip \
      nano

# 필요한 패키지 설치
RUN pip install jupyter numpy scipy ipython pandas opencv-python matplotlib scikit-image scikit-learn wandb ipykernel==5.5.5

# 작업 디렉터리
WORKDIR /

# 실행 프로그램
CMD ["/bin/bash"]
```

```bash
# chkmook.pytorch, 태그 1의 이미지 빌드
$ docker build --tag chkmook/pytorch:1 .
# 빌드한 chkmook/pytorch:1 이미지로 chkmooook 컨테이너 생성
$ docker run -it -d --ipc=host --name chkmooook \
-v /home/chkmook:/home/chkmook -v /datasets:/datasets -w /home/chkmook \
 chkmook/pytorch:1
```

## docker-compose

docker-compose를 통해 다중 컨테이너 애플리케이션(서비스)을 정의할 수 있고, 단일 명령을 사용하여 모두 실행 또는 종료할 수 있다.

e.g) jupyter, tensorboard, workspace를 한번에 열고 닫을 수 있다.

docker run 시 입력해주어야하는 옵션들을 보기 쉽게 관리할 수 있다는 장점도 있는 것 같다.

- docker-compose.yaml

```yaml
version:<docker-compose 버전명>

services:

  <서비스1>: # <계정 이름>_<서비스 이름>_<number> 형식으로 컨테이너 생성
    image: <이미지> # 베이스로 사용할 이미지
    working_dir: <경로> # 시작 경로
    tty: true # -t 옵션
    stdin_open: true # -it 옵션인 것 같다.
    ipc: "host" # 리소스 접근 권한
    volumes:
      - <호스트 경로1>:<컨테이너 경로1>
      - <호스트 경로2>:<컨테이너 경로2>
    ports:
      - "<호스트 포트>:<컨테이너 포트>"
    command: <명렁어> <인자>

  <서비스2>:
    image: <이미지>
    working_dir: <경로>
    tty: true
    stdin_open: true
    environment:
      - <환경 변수 정의. 아래는 예시>
      - NVIDIA_VISIBLE_DEVICES=0 (사용할 GPU)
      - TERM: xterm-256color
    ipc: "host"
    volumes:
      - <호스트 경로1>:<컨테이너 경로1>
      - <호스트 경로2>:<컨테이너 경로2>
    ports:
      - "<호스트 포트>:<컨테이너 포트>"
    command: <명렁어> <인자>
```

- 위 파일을 바탕으로 컨테이너 생성

docker-compose는 기본적으로 커맨드가 실행되는 디렉토리의 docker-compose.yml (.yaml)을 설정 파일로 사용한다.

임의의 이름을 가진 설정 파일을 사용하기 위해 -f 옵션을 사용한다.

-d 옵션을 통해 백그라운드에 컨테이너를 띄운다.

```bash
# docker-compose.yaml 설정 파일 이용하여 컨테이너(들) 생성
# 경로에 이름이 docker-compose.yaml으로 된 파일이 하나밖에 없다면 -f 옵션은 안줘도 됨.
$ docker-compose -f docker-compose.yaml up -d
$ docker-compose up -d
# docker-compose.yaml 설정 파일 이용하여 생성된 컨테이너(들) 정지 후 삭제 (stop+rm)
$ docker-compose down

# 내려간 특정 서비스의 컨테이너를 올린다.
$ docker-compose start <서비스>
# 올라가 있는 특정 서비스의 컨테이너를 내린다.
$ docker-compose stop <서비스>
```

- docker-compose.yaml example
```yaml
version: "3.8"

services:
  
  notebook:
    image: chkmook/pytorch:1.9.0-cuda11.1-cudnn8-devel
    working_dir: /home/chkmook
    tty: true
    stdin_open: true
    ipc: "host"
    volumes:
      - /home/chkmook:/home/chkmook
      - /datasets:/datasets
    ports:
      - "8888:8888"
    command: jupyter notebook --allow-root --ip="0.0.0.0" --port="8888" --no-browser --NotebookApp.password='sha1:46a9318d3a9d:69f2d46a50573c10545e943c4f286fad304ae6f5'

  lab: 
    image: chkmook/pytorch:1.9.0-cuda11.1-cudnn8-devel
    working_dir: /home/chkmook
    tty: true
    stdin_open: true
    ipc: "host"
    volumes:
      - /home/chkmook:/home/chkmook
      - /datasets:/datasets
    ports:
      - "9999:9999"
    command: jupyter lab --allow-root --ip="0.0.0.0" --port="9999" --no-browser --NotebookApp.password='sha1:46a9318d3a9d:69f2d46a50573c10545e943c4f286fad304ae6f5'

  workspace:
    image: chkmook/pytorch:1.9.0-cuda11.1-cudnn8-devel
    working_dir: /home/chkmook
    tty: true
    stdin_open: true
    ipc: "host"
    volumes:
      - /home/chkmook:/home/chkmook
      - /datasets:/datasets
```


## 포트로 주피터 연결

1. vpn + 포트 포워딩 이용

정보보안팀 공문에 따르면 2021-2학기 중으로 필요성이 인정되는 웹서비스(80(http), 443(https). 추후 별도 안내 예정)를 제외한 기존 보안정책을 삭제하고, 외부에 개방되어있는 중요 서비스포트(FTP(21), SSH(22) 등)를 차단한다고 합니다. (※ 교내에서 이루어지는 통신은 관계없음)

따라서, 아래 링크를 따라 VPN 접속으로만 포트 접속이 가능합니다.

[https://ibook.yonsei.ac.kr/Viewer/ysvpn_user_manual](https://ibook.yonsei.ac.kr/Viewer/ysvpn_user_manual) 이용하여 VPN을 통해 접속 

- 주피터 패스워드 / 토큰 받기

```bash
# jupyter config 생성 
$ jupyter notebook --generate-config

$ ipython
In [1]: from notebook.auth import passwd
# or from IPython.lib import passwd

# 입력한 패스워드가 주피터 노트북에 접속할 때 사용하는 패스워드
In [2]: passwd()
Enter password:
Verify password:
out [2] u<이러면 여기에 토큰이 나옵니다>
# u를 제외한 위 문자열을 복사해서 keep.


# jupyter notebook --generate-config 했으면 자동으로 생성됩니다.
$ vi .jupyter/jupyter_notebook_config.py

# 아래 코드 아무데나 넣고 저장
c = get_config()
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.open_browser = False
c.NotebookApp.port = <사용하고자 하는 포트번호를 넣는게 맞는데 대충 넣어도 되는 듯>
c.NotebookApp.password = <여기에 위에서 적은 토큰 붙여넣기>
```

- run과 함께 jupyter notebook 실행

```bash
# 당연한 얘기지만 이미지로 만들어진 컨테이너에 jupyter가 설치되어 있어야 jupyter notebook을 실행할 수 있다.
# 위에서 생성한 jupyter config파일을 마운트 시에 같이 갖고 와야 위에서 설정해준 비밀번호를 사용할 수 있다.
# 비밀번호를 사용하지 않고 jupyter를 사용하려면 아래 커맨드에서 <--NotebookApp.password= --NotebookApp.tokem=> 으로 변경 (비추)
$ docker run -d -it --ipc="host" -v <경로>:<경로> -w <시작 디렉터리> -p <호스트 포트>:<컨테이너 포트> --name <컨테이너 이름> <주피터가 설치된 이미지> jupyter notebook --allow-root --ip="0.0.0.0" --port="<사용할 포트 번호>" --no-browser --NotebookApp.password=<아까 복사해놓은 그 토큰>
```

위 코드 실행하여 컨테이너를 생성한 후 웹에서 <서버 ip>:<포트번호>로 jupyter에 접속할 수 있다. 비밀번호는 jupyter config 설정에서 입력한 것 사용



2. 80번 포트 + nginx + 포트 포워딩 이용

공문에 따르면 80번 포트는 열 수 있을 것 같습니다. 80번 포트에서 nginx를 열고 멀티 도메인으로 여러개의 jupyter를 열면 가능할 포트 접속이 가능할 것 같습니다.

위 방법과 비교했을 때의 장점으로는 교외에서 접속할 때 VPN을 키지 않아도 되지만, 단점으로는 설정 과정이 복잡합니다. (일단 80번 포트도 지금은 닫혀있습니다.)

```bash
# To be written later
```