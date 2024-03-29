## 0. 배우게 된 계기

bob에서도 배웠었지만 사용할 계기가 없어서 잘 정리하지 못 했다.  
이번에는 특별히 개념보다는 명령어 위주의 기록이 많다.  
아마 자세히 배우자면 끝도 없을 것이고 k8s까지 추가하면 턱도 없는 지식일테니 여기에서는 딱 도커에 대해서만 다룰 예정이다.

## 1. 컨테이너

### 정의

서버 가상화 기술의 일종으로 호스트 OS 상에 논리적인 구획을 만들고, 어플리케이션을 작동시키기 위해 필요한 라이브러리나 어플리케이션 등을 하나로 모아, 마치 별도의 서버인 것처럼 사용할 수 있게 만드는 것이다.  
기존에 사용하던 가상머신(VMWare, Virtual Box) 등에 비해 오버헤드가 적기 때문에 가볍고 빠르게 작동한다.  
기존 monolithic(하나의 구조)에서 MSA(Micro Service Architecture, 하나의 서비스를 잘게 분해해서 관리)로 변화하면서 용량을 작게 관리할 수 기술이 필요하면서 중요성이 증가했다.  
대표적인 제품으로 Docker가 있다.

cf) _hypervisor type_

- type1(bare-metal, native): 호스트 하드웨어에 직접 설치하여 구동. HW - Hyper - OS
- type2(hosted): 일반적인 application과 마찬가지로 호스트 OS 위에 설치. HW - Host OS - Hyper - Guest OS. VMWare, Virtual Box 등이 여기에 속함.

### 이미지

필요한 프로그램과 라이브러리, 소스를 설치한 뒤 만든 하나의 파일이다.

### Docker 장단점

- 장점
  - **type2 hypervisor보다 실행 속도가 빠름.** OS를 실행하지 않고도 모든 프로세스에 대한 컨테이너를 실행할 수 있기 때문. HW - Host OS - 컨테이너 환경
  - **안전함.** container들끼리 분리되어 있기 때문. 이는 리눅스 커널의 namespace의 cgroup을 이용했기 때문에 가능.
  - **실행환경 구성이 빠르고 간단함.**
  - **비용절감.** 하나의 물리적 서버에 다수의 컨테이너를 가동할 수 있음.
- 단점
  - **플랫폼 의존적.** 도커는 Linux에서만 실행가능. Windows나 MacOS는 별도의 프로그램이 동작해야 함.
  - **bare-metal 방식 보다는 느림.**
  - **docker를 이용하여 GUI 앱을 돌리기에는 불편.**
  - 불필요한 리소스 사용. 이 때문에 이미지, 네트워크, 볼륨 등 사용하지 않는 부분은 적절히 삭제 필요.

_cf) 불필요한 리소스 사용_  
[마음을 전해요 사이드 프로젝트](https://github.com/meopin-top/convey-your-mind-FE) 진행 시 해당 단점 때문에 고민했던 부분이 있다.  
~정답이 아닐 수도 있다. 만약 이후에 운영하다가 새로운 해결법을 알거나 잘못된 부분이 있으면 수정하겠다.~  
프로젝트 진행 시 매번

- ssh 연결
- 기존 서버 종료
- `git pull`
- `yarn build`
- `yarn start`

를 하기 싫어 github actions를 이용해 배포를 자동화했다.  
그래서 [github actions](https://github.com/meopin-top/convey-your-mind-FE/blob/main/.github/workflows/push_ci-cd.yaml)에서

- `docker build`
- `docker push 새로운 태그의 이미지`
- ssh 연결
- `docker pull 새로운 태그의 이미지`
- `docker stop 기존 태그의 컨테이너`
- `docker rm 기존 태그의 컨테이너`
- `docker run 새로운 태그의 컨테이너`
- `docker image rune`
- `docker rmi 기존 태그의 이미지`

와 같은 순서로 배포를 진행했다.  
참고로 오케스트레이션 도구를 쓰거나 여러 개의 서버를 구동하고 있던 것이 아니었기 때문에, 기존 태그의 컨테이너와 새로운 태그의 컨테이너가 같은 포트를 사용해야 됐기 때문에 기존 태그의 컨테이너를 종료하기 전까지 새로운 태그의 컨테이너를 실행할 수 없었다.  
즉, rolling update 같은 방식을 통한 zero-downtime은 불가능했다.  
또한 어플리케이션(서비스) 버전이 올라갈 때마다 docker hub에 태그를 바꿔서 이미지를 새로 업로드했는데, 이는 도커 이미지를 서비스와 같은 버전으로 관리하여 변경 사항 추적을 용이하게 하기 위해서였다.

명령어를 위와 같은 순서로 진행하는 데에는 다음과 같은 3가지 목적이 있었다.

1. **downtime 최소화**
2. **네트워크 통신 최소화**
3. **디스크 용량 낭비 최소화**

이를 이해하기 위해서 [docker layer](https://github.com/mochang2/development-diary/blob/main/013-docker.md#3-docker-image-layer)을 한 번 보고 오자.  
docker layer에 대해 안다면 `docker rmi` 동작 방식도 한 번 짚고 넘어가자.

<img src="https://github.com/mochang2/development-diary/assets/63287638/3ea8ec39-d92d-4efc-aded-e9148f21be03" alt="docker rmi" width="700" height="auto" />

출처: ChatGPT(나도 직접 ubuntu 서버에서 여러 tag를 가지고 실험했는데 이 말이 맞는 거 같다)

이 개념을 활용하여 **downtime 최소화**와 **네트워크 통신 최소화**를 달성하기 위해 `docker stop` -> `docker rm` -> `docker rmi` -> `docker pull` -> `docker run`을 진행하지 않고 `docker pull`과 `docker run` 사이에 `docker stop`과 `docker rm`을 실행했다.  
`docker pull`을 실행함으로써 새로운 layer(기존 태그와 새로운 태그의 이미지 간 다른 layer)를 다운로드하는 시간만큼 늘어나는 downtime을 줄이기 위해서이다(하지만 위에서 말한대로 여전히 downtime은 존재한다).  
또한 `docker rmi`를 가장 마지막에 실행함으로써 기존 태그의 이미지와 새로운 태그의 이미지는 공유하는 docker layer는 `docker pull` 명령 실행 시 다운로드하지 않아도 됐다.  
(새로운 태그의 컨테이너를 실행해도 `docker rmi`는 사용하지 않는 layer만 삭제해주므로 서비스 실행에는 문제가 없었다)  
**디스크 용량 낭비 최소화**를 위해 `docker rmi` 이외에도 `docker image prune`을 진행했다.  
이 명령어는 dangling된 이미지를 삭제해주는 명령어이다.  
이외에도 network, volume 등도 prune이 가능했지만, convey-your-mind-fe 컨테이너에서는 사용하지 않았기 때문에 따로 진행하지 않았다(이 부분은 API의 github actions에서 진행하는 게 맞는 것 같다).

## 2. 도커 라이프 사이클

<img src="https://github.com/mochang2/development-diary/assets/63287638/a308716a-0051-4e3f-87ce-6ab90bf3ddd7" alt=" " width="500" height="auto" />

출처: https://jaimemin.tistory.com/1829

<br/>위 그림을 설명하자면 이미지는 레지스트리에서 pull을 받아서 사용할 수 있고, 반대로 원하는 이미지가 있다면 도커 허브에 이미지를 push할 수 있다.  
깃과 비슷한 개념이라고 생각할 수 있으며, push를 하기 위해서는 인증 정보(로그인 정보)가 필요하다.  
이미지는 아직 메모리에 올라가서 실행 가능한 상태가 아니며 이미지를 create할 경우 컨테이너가 된다.  
컨테이너를 start할 경우 컨테이너가 메모리에 올라가게 되고 stop을 할 경우 메모리에서 내려오게 된다.  
run은 pull + create + start를 합친 명령어로 이미 받은 이미지의 경우 pull을 하지 않고 create + start만 진행한다.  
commit은 컨테이너에서 변경된 사항을 이미지로 저장할 수 있게 만들어준다.  
rmi는 이미지를 지우며, rm은 컨테이너를 지운다.

## 3. docker image layer

도커는 이미지의 계층구조로 이루어져 있다.  
그 각각의 계층을 레이어(layer)라고 표현하며, 레이어는 도커 이미지가 빌드할 때 dockerfile에 정의된 명령문을 순서대로 실행하면서 만들어진다.  
레이어들은 각각 독립적으로 저장되며 읽기 전용이기 때문에 임의로 수정할 수 없다.  
docker run을 통해 이미지로 컨테이너를 생성할 경우 기존의 이미지 레이어들 위에 container layer가 생성된다.  
아래 사진은 위에 말한 내용을 잘 표현한다.  
![ff](https://user-images.githubusercontent.com/63287638/161035813-9db90b72-d762-496e-b1b9-b3fde0e63084.png)

또한 이런 레이어 구조 덕분에 용량을 크게 줄일 수 있다.  
python:3.8와 python:3.7의 이미지가 동시에 필요하다고 하면, (아마) 최상위 레이어를 제외하고 그 아래 레이어들은 동일할 것이다.  
ubuntu 이미지를 활용하고, apt 명령어로 python을 설치하며, ... 해당 과정에서 동일한 부분은 하나의 이미지로 표현할 수 있기 때문이다.

## 4. docker 명령어

~docker 설치는 구글링해보면 금방 나오므로 사용하는 OS 별로 검색하면 될 것 같다.~  
https://captcha.tistory.com/49 를 참조했다.

```sh
docker search <이미지> # docker hub로부터 사용 가능한 image를 찾는 명령어

docker pull <이미지[:버전]> # docker hub로부터 image를 다운받는 명령어
# 버전을 명시하지 않는 경우 최신 버전을 기본으로 함

docker images # 현재 PC에 받아져 있는 image들을 출력하는 명령어

docker inspect [옵션] <image> # 이미지에 대한 정보들을 출력
# 대표적으로 이미지 레이어의 해시값, ID, Cmd 등을 볼 수 있음

docker run [옵션] <이미지> [실행할 파일] # pull + create + start를 포괄하는 명령어
# 자주 사용하는 옵션으로는
## -d: 백그라운드에서 실행
## -i: interactive. 사용자가 입출력을 할 수 있는 상태로 실행
## -t: 가상 터미널 환경을 애뮬레이트
## -p: 포트 포워딩. host port:container port로 사용
## -v: 볼륨 마운드. host location:container location으로 사용
## -e: 환경 설정

docker exec <컨테이너> <명령어> # 실행중인 container에 명령어 실행 가능
## 예시 docker exec QW341C5 ls

docker ps # process를 볼 수 있는 명령어. 실행 중인 container만 볼 수 있음
## -a: 종료된 container까지 볼 수 있음
## -q: container id만 출력
### 응용: docker -rm `docker ps -aq`

docker attach <컨테이너> # 현재 실행중인 container에 접속

docker create <이미지> # 이미지를 통해 컨테이너 생성
docker start <컨테이너> # 종료된 컨테이너 시작
docker stop <컨테이너> # 동작중인 컨테이너 종료
docker rm <컨테이너> # 컨테이너 삭제
```

## 5. `dockerfile`

docker image를 생성하기 위한 스크립트이다.  
나열된 명령어를 차례대로 수행한 뒤 docker image를 생성해준다.  
이미지가 어떻게 만들어졌는지 기록하여 어떠한 이미지를 배포할 때 이미지 파일 자체를 배포할 필요없이 `dockerfile`만을 배포하면 되므로 배포가 간단해진다.  
또한 이를 통해 컨테이너가 특정 행동을 수행하도록 할 수 있다.  
기본적으로 파일의 이름은 `dockerfile`이어야 하며 확장자가 존재하지 않는다.  
아래는 사용 방법을 정리해둔 것이다.

```plaintext
FROM : Docker Base Image (기반이 되는 이미지, <이미지 이름>:<태그> 형식으로 설정)
MAINTAINER : 메인테이너 정보 (작성자 정보)
RUN : Shell Script 또는 명령을 실행
CMD : 컨테이너가 실행되었을 때 명령이 실행
LABEL : 라벨 작성 (docker inspect 명령으로 label 확인 가능)
EXPOSE : 호스트와 연결할 포트 번호를 설정
ENV : 환경변수 설정
# 사용 예시. ENV LC_ALL=C.UTF-8
# 베이스 이미지로 node:10를 사용해야 됐는데 (os는 gnu linux) 컨테이너 내부에서 한글 인식이 되지 않았다.
# 해당 문제를 해결하기 위해 위와 같이 사용해서 해결했다.
ADD : 파일 / 디렉터리 추가
COPY : 파일 복사
ENTRYPOINT : 컨테이너가 시작되었을 때 스크립트 실행
VOLUME : 볼륨 마운트
USER : 명령 실행할 사용자 권한 지정
WORKDIR : "RUN", "CMD", "ENTRYPOINT" 명령이 실행될 작업 디렉터리
ARG : Dockerfile 내부 변수
ONBUILD : 다른 이미지의 Base Image로 쓰이는 경우 실행될 명령 수행
SHELL : Default Shell 지정
```

### RUN / CMD / ENTRYPOINT

https://blog.leocat.kr/notes/2017/01/08/docker-run-vs-cmd-vs-entrypoint 를 참고했다.  
셋 모두 Shell 방식처럼 `RUN? apt update && apt install apache2 -y`와 같이 사용할 수 있으며, Exec 방식처럼 RUN? \["/bin/bash", "-c", "apt-get update"\] 처럼 사용할 수 있다.

- RUN.
  - 새로운 레이어에서 명령어를 실행하고, 새로운 이미지를 생성한다.
  - 보통 패키지 설치 등에 사용된다.
  - 주의할 점은 `RUN apt update` 한 뒤에 `RUN apt install`을 하면 update한 레이어와 install한 레이어는 다른 레이어이다.
- CMD.
  - default 명령이나 파라미터를 설정한다.
  - `docker run` 실행 시 실행할 커맨드를 주지 않으면 이 default 명령이 실행된다.
  - 그리고 ENTRYPOINT의 파라미터를 설정할 수도 있다.
  - CMD의 주용도는 컨테이너를 실행할 때 사용할 default를 설정하는 것이다.
  - CMD는 여러 번 사용할 수 있지만 가장 마지막에 있는 CMD만 남게 된다(override).
- ENTRYPOINT.
  - 컨테이너를 실행할 수 있게 설정한다.
  - dockerfile 내에 1번만 정의 가능하다.
  - CMD와 다른 점으로, `docker run`으로 실행시 command를 입력하면 ENTRYPOINT의 파라미터로 인식한다.

## 6. NW

https://jonnung.dev/docker/2020/02/16/docker_network/ 와 https://captcha.tistory.com/70 를 참조했다.<br/>
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcvqvBQ%2Fbtq1jminJ40%2F47SuSitCt4UJ5vUSlrLTyk%2Fimg.png" alt=" " width="550" height="auto" />  
<br/> 위 사진은 도커 네트워크에 대한 개괄이다.

### 도커 설치 시

docker를 설치한 이후 host의 네트워크 인터페이스를 살펴보면 _docker0_ 라는 가상 인터페이스가 생긴다.  
_docker0_ 는 도커가 자체적으로 제공하는 네트워크 드라이버 중 브리지에 해당한다.  
도커에서 사용할 수 있는 네트워크 종류는 브리지, 호스트, 논(none) 등이 있다.

- bridge: Docker default networking mode which will enable the connectivity to the other interfaces of the host machine as well as among containers.
- host: Container will share the host’s network stack and all interfaces from the host will be available to the container.
- none: This mode will not configure any IP for the container and doesn’t have any access to the external network as well as for other containers. It does have the loopback address and can be used for running batch jobs.

참고: https://stackoverflow.com/questions/41083328/what-is-the-use-of-docker-host-and-none-networks  
<br/>
_docker0_ 브리지는 컨테이너가 통신하기 위해 사용된다.  
도커 컨테이너를 생성하면 자동으로 이 브리지를 활용하도록 설정되어 있다.  
_docker0_ 인터페이스는 172.17.0.0/16 서브넷을 갖기 때문에 컨테이너가 생성되면 이 대역 안에서 IP를 할당받게 된다.

```sh
docker network inspect bridge
```

위 명령어를 통해 브리지 네트워크의 자세한 정보를 알 수 있다.

### 도커 컨테이너 생성 시

도커는 컨테이너에 내부 IP를 순차적으로 할당하며 해당 IP는 컨테이너를 재시작할 때마다 변경될 수 있다.  
컨테이너를 시작할 때마다 외부와 통신하기 위해 2개의 NW 인터페이스를 함께 생성한다.  
하나는 컨테이너 내부 Namespace에 할당되는 eth0 이름의 인터페이스이고, 다른 하나는 호스트 네트워크 브리지 _docker0_ 에 바인딩되는 veth~ 라는 이름의 veth(virtual eth) 인터페이스이다.  
각 veth 인터페이스는 _docker0_ 브리지에 바인딩돼 호스트의 eth0와 같은 인터페이스와 이어진다.

```sh
docker network ls
```

위 명령어를 통해 현재 컨테이너 네트워크 목록을 확인할 수 있다.

## 7. `docker-compose`

https://sohyun-lee.tistory.com/12 를 참고했다.  
여러 개의 컨테이너가 하나의 애플리케이션으로 동작할 때, 이를 테스트하려면 각 컨테이너를 하나씩 생성해야 한다.  
여러 개의 컨테이너로 구성된 애플리케이션을 구축하기 위해 `run` 명령어를 여러 번 사용할 수 있지만, 테스트 단계에서는 매번 `run` 명령어에 대한 옵션을 설정해서 진행하기 번거롭다.  
이를 해결하기 위해 yaml 파일 설정한 뒤 `docker-compose` 명령어를 수행함으로써 여러 개의 컨테이너 실행을 하나의 프로젝트처럼 다룰 수 있다.  
기본적으로 현재 또는 상위 디렉토리에서 `docker-compose.yml` 파일을 찾아 컨테이너를 생성하지만, _-f_ 옵션을 통해 위치와 이름을 지정할 수 있다.  
또한 `docker-compose config` 명령어를 통해 오타나 파일 포맷이 적절한지 등에 대한 검사를 할 수 있다.  
참고로 docker와 별도로 추가적인 설치가 필요하다.

### `docker-compose.yml` 구조

버전, 서비스, 볼륨, 네트워크 이렇게 4가지 항목으로 구성된다.

```
version: '3.0'
services:
  web:
    image: [저장소이름]/[이미지이름]:[태그]
    ports:
      - "80:80"
    links:
      - mysql:db
    command:
networks:
  driver: ....
```

- 버전
  - docker-compose 버전은 도커 엔진 버전에 의존성이 있으므로 가능하다면 최신 버전을 사용하는 것이 좋음.
- 서비스
  - 생성될 컨테이너들을 묶어놓은 단위로 서비스 항목 아래에는 각 컨테이너에 적용될 생성 옵션을 지정할 수 있음.
  - 서비스의 이름은 services의 하위 항목으로 정의함.
  - 옵션
    - image: 서비스의 컨테이너를 생성할 때 쓰일 이미지의 이름을 설정. 이미지 이름 포맷은 `docker run`의 [저장소이름]/[이미지이름]:[태그] 형태와 같으며, 존재하지 않을 경우 저장소에서 자동으로 내려받음.
    - links: —link와 동일하며, 다른 서비스에 서비스명만으로 접근할 수 있도록 설정.
    - environment: —env, -e 옵션과 동일하며, 컨테이너 내부에서 사용할 환경변수 설정.
    - command: `docker run` 명령어의 마지막에 붙는 커맨드와 같으며, 컨테이너가 실행될 때 수행할 명령어를 설정.
    - depends_on: 특정 컨테이너에 대한 의존 관계를 나타내며, 이 항목에 명시된 컨테이너가 먼저 실행되고 해당 컨테이너가 실행.
    - ports: -p와 같으며, 서비스의 컨테이너를 개방할 포트를 설정. 단일 호스트 환경에서 80:80과 같이 호스트의 특정 포트를 서비스의 컨테이너에 연결하면 `docker-compose scale` 명령어로 서비스의 컨테이너의 수를 늘릴 수 없음.
    - build: 도커파일에서 이미지를 빌드해 서비스의 컨테이너를 생성하도록 설정.
    - extends: 다른 YAML 파일이나 현재 YAML 파일에서 서비스 속성을 상속받음.
- 네트워크
  - 옵션
    - driver: 서비스의 컨테이너가 브리지 네트워크가 아닌 다른 네트워크를 사용하도록 설정할 수 있음.
    - ipam: IPAM(IP Address Manager)를 위해 사용할 수 있는 옵션으로 subnet, ip 범위 등을 설정할 수 있음.
    - external: YAML 파일을 통해 프로젝트를 생성할 때마다 네트워크를 생성하는 것이 아닌 기존의 네트워크를 사용하도록 설정할 수 있음.
- 볼륨
  - 옵션
    - driver: 볼륨을 생성할 때 사용될 드라이버를 설정. 어떠한 설정도 하지 않으면 local로 설정되며 사용하는 드라이버에 따라 변경해야 함.
    - external: 프로젝트를 생성할 때마다 볼륨을 매번 생성하지 않고 기존 볼륨을 사용할 수 있도록 설정.

### 명령어

```sh
docker-compose up [옵션] # 컨테이너들 띄우기

docker-compose down [옵션] # 컨테이너들 내리기

docker-compose restart

docker-compose logs

docker-compose ps

docker-compose config

docker-compose -f <경로> # 다른 경로에 있는 도커 컴포즈 파일을 사용.
# 또한 여러 개의 docker-compose 설정 파일을 사용할 수도 있음.
```
