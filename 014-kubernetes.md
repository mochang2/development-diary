## 0. 공부하게 된 계기
docker 정리하는 김에 k8s도 같이 정리하고자 한다.  
k8s를 배운다는 것은 대부분 k8s API를 배운다는 것이다(API가 무엇인지는 후술).  
유튜브 따배런 채널의 \[따배쿠\]를 참고했다.  
  
## 1. 개념
컨테이너를 도커 플랫폼에 올려서 관리 + 운영 + 클러스터 서비스를 지원해주는 것을 말한다.  
구글에서 만들었으며 그리스어로 조타수라는 의미를 가진다.  
MSA가 되면서 하나의 어플리케이션은 적게는 수 십 개, 많게는 수 백 개의 컨테이너로 구성된다.  

##### 컨테이너 오케스트레이션
control plane이 worker node들을 관리하는 구조를 말한다.  
필요에 그리고 상황에 따라서 컨테이너를 실행하기도 하고, 정지시키기도 한다.  

##### 컨테이너 계층구조
* Layer1: Physical Infastructure(Raw Computer, NW, Storage)
* Layer2: Virutal Infrastructure(vSphere, EC2, GCP, Azure)
* Layer3: OS(Ubuntu, CoreOS, Unikernels)
* Layer4: Container Engine(__Docker__, Rocket, RunC)
* Layer5: Orchestartion/Scheduling Service Model(__Kubernetes__, Docker Swarm, Marathon/Mesos, Nomad)
* Layer6: Development Workflow Opinionated Containers(OpenShift, Cloud Foundary, Docker Cloud)

##### 특징
* 워크로드 분리(컨테이너 간 분리가 가능하며 통신이 원활하게끔 만듦)
* 어디서나 실행 가능(on premise, cloud 환경 모두 가능)
* 선언적 API(DevOps, NoOps가 가능하도록 도와줌)
* 기능
  * horizontal scaling
  * self-healing


## 2. k8s 클러스터 구성
* control plane(master node): 워커 노드들의 상태를 관리하고 제어. 여러 개일 수 있음.
  * etcd: key-value 타입의 저장소로 worker node 및 k8s에 대한 상태 정보를 저장.
  * kube-apiserver: `kubectl` 명령어 등을 통해 k8s API를 사용하도록 요청을 받고 요청(권한, 문법 등)이 유효한지 검사.
  * kube-scheduler: pod을 실행할 노드를 선택.
  * kube-controller-manager: pod를 관찰하며 개수를 보장.
  * ~여기도 kubelet은 있지만 크게 중요하게 보진 않는듯~
* worker node: 도커 플랫폼을 통해 컨테이너를 동작하며 실제 서비스 제공.
  * kubelet: 모든 노드에서 실행되는 k8s 에이전트. 데몬 형태로 동작. `docker` 명령어로 런타임에게 실행해달라고 전달.
  * kube-proxy: k8x의 network 동작을 관리.
  * 컨테이너 런타임: 컨테이너를 실행하는 엔진. docker, containerd, runc 등

`kubectl` 명령어로 master node에 REST call을 발생시키면 scheduler는 가장 적합한 worker node에 할당한다.  
해당 노드의 `kubelet`이 명령을 받아 컨테이너로서 실행하고, k8s 이러한 컨테이너를 pod이라는 단위로 관리한다.  


##### kubeadm, kubelet, kubectl
* kubeadm: k8s 전체를 관리, 운영하는 커맨드
* kubelet: 컨테이너를 조작해주고 마스터와 통신하기 위한 데몬. static pod을 동작시킴.
* kubectl: 클러스터를 조작하기 위한 cli util

```
kubeadm init # master node에서 실행시, API, controller, scheduler, etcd 등의 컴포넌트가 생성됨.
# 해당 결과 이후에 나오는 token을 통해 worker node가 join할 수 있음.
```

##### CNI(Container Network Interface)
Pod network add-on(부가 기능)으로 클러스터 간 네트워크 통신이 가능하도록 한다.  
대표적으로 weave net 등이 있다.  


## 3. pod
컨테이너를 표현하는 k8s API의 최소 단위이다.  
control plane의 API에서 컨테이너 동작을 명령할 수 없다. 대신 pod에 대한 동작을 명령할 수 있다.  
이 pod을 통해서 컨테이너를 실행시키는 것이다.  
기본적으로 모든 pod은 항상 pause container를 포함하고 있다.  
pause container는 pod에 대한 인프라(IP, hostname 등)을 관리하는 컨테이너이다.  
pod에는 하나 또는 여러 개의 컨테이너가 포함될 수 있지만 IP는 pod에게만 부여된다.  

```
// yaml 예시
apiVersion: v1
kind: Pod
metadata:
  name: <Pod 이름>
  namespace: <namespace 이름>
  labels:
    <key>: <value>
spec:
  containers:
  - name: <컨테이너 이름>
    image: <컨테이너 이미지>
    env:
    - name: <환경변수 키>
      value: <환경변수 값>
    resources:
      requests:
        memory: <Mi 단위>
        cpu: <m 단위 또는 개수>
      limits:
        memory: ~~
        cpu: ~~
```

##### 동작 flow
scheduler가 어떤 node가 좋을지 선택할 동안 pod은 pending 상태가 된다.  
node를 배정받으면 running 상태가 되며 그 이후 상태는 succeeded 또는 failed로 구분된다.  

##### liveness probe
k8s의 특징 중 하나는 self-healing이다.  
이 기능은 heartbeat를 측정하는 기능이라고 볼 수 있다.  
probe를 할 때 연속해서 세 번(기본 설정 사용 시) 실패하면 오류라고 판단하고, 오류라고 판단하면 컨테이너를 다시 pull 받은 뒤 시작한다.  
단, pod은 다시 시작하지 않으므로 IP 주소가 변경되지 않는다.  
probe 하는 방법으로는 3가지가 있다.  

1. httpGet probe: 지정한 IP 주소, port, path에 HTTP GET 요청을 보내 200이 아닌 값이 오면 오류라고 판단.
2. tcpSocket probe: 지정된 포트에 TCP 연결을 시도. 연결되지 않으면 오류라고 판단.
3. exec probe: 명령을 전달하고 명령의 종료코드가 0이 아니라면 오류라고 판단.

##### resource(cpu, memory) 할당
pod 실행할 때 여유로운 노드에서 실행할 수 있도록 명령을 내릴 수 있다.  
만약 여유가 없는 노드에서 실행하면 메모리 부족 등의 버그가 일어날 수 있기 때문이다.  
이를 request라고 한다.  
한편, pod이 가질 수 있는 최대 resource를 제한할 수도 있는데, 이를 limit이라고 한다.  


## 4. namespace
k8s API 중 하나로 관리에 도움을 주는 기능이다.  
클러스터 하나를 여러 개의 _논리적인_ 단위로 나눠서 사용하기 위해 사용한다.  
`-n(--namespace)` 옵션을 추가함으로써 또는 yaml 파일에 namespace를 추가함으로써 해당 namespace에 해당하는 pod들에 대해서만 명령을 내릴 수 있다.  
`kubectl create namespace [네임스페이스 이름]` 을 통해 만들 수 있으며 yaml 파일로 만드는 방법도 있다.  
기본적으로 `default` 라는 namespace가 존재한다.  
참고해야할 점은 namespace를 지우면 해당 namespace에 포함되어 있는 모든 object들이 지워진다.


## 5. kubectl 명령어
> kubectl \[command\] <TYPE> <NAME> <flags>

command에는 자원(object)에 실행할 명령인 create, get, delete, edit, run, apply, port-forward 등이 들어간다.  
type에는 자원의 타입인 node, pod, service, replicationcontrollers 등이 들어간다.  
name은 사용자가 만든 자원의 이름이 들어간다.  

```
kubectl get namespaces # namespace들을 볼 수 있음.
  
kubectl api-resource # api resource에 대한 약어 정보 등을 볼 수 있음.

kubectl logs [컨테이너] # 컨테이너의 로그를 봄.
  
kubectl get nodes(no) # node 정보들을 보여줌.
  
kubectl describe node [노드 이름] # 자세한 정보를 보여줌.
  
kubectl run [pod 명명] --image=[컨테이너 이미지] # pod을 만듦.
  
kubectl get pods # 컨테이너 pod 정보들을 보여줌. 기본적으로 default namespace에 해당하는 pod들을 보여줌.
# 모든 namespace에 대해 보고 싶으면 --all-namespaces 옵션을 붙이면 됨.
  
kubectl create deployment [pod 명명] --image=[컨테이너 이미지]:[버전] --replicas=[개수]
# 개수만큼 컨테이너 pod을 만들 수 있음.
  
kubectl get deploy(deployments, deployments.apps)
  
kubectl exec pod(생략 가능) [pod 이름] <-c> <컨테이너 이름> -it -- /bin/bash # 컨테이너 내부로 들어가서 bash를 실행시킴.
# pod이 여러 컨테이너를 포함하고 있을 때 -c 옵션으로 특정한 컨테이너를 지정할 수 있다.
  
kubectl create -f(file) [파일 이름] # 컨테이너 pod을 파일에 적힌 명령어 대로 실행(yaml, json 등).
# run 명령어 시 --dry-run을 사용하면 실행 가능한지를 확인할 수 있으며 -o 옵션으로 출력 형식을 변경함으로써
# 위에서 사용한 [파일 이름]에 해당하는 파일의 내용을 얻을 수 있음.
```


## 6. init container
서비스가 정상적으로 실행하기 전에 즉, main container를 실행하기 전에 전제 조건이 있을 수 있다.  
예를 들면, 로그인을 하기 위해서는 db와 connection이 이루어져 있어야 되고, 또는 특정 서비스가 켜져있어야 될 수도 있다.  
이러한 서비스들을 정의해두는 컨테이너를 init container라고 한다.  


## 7. Controller
controller의 종류에는 daemon set, __replica set__, statefule sets, cronjob, __replication controller__ 등이 있다.  

##### replication controller
API, etcd, Scheduler와 함께 Pod의 개수를 보장한다.  
요구하는 pod의 개수가 부족하면 template을 이용해 pod를 추가하며 요구하는 pod 수 보다 많으면 최근에 생성된 pod을 삭제함으로써 동작한다.  

```
// yaml 
apiVersion: v1
kind: ReplicationController
metadata:
  name: <RC이름>
spec:
  replicas: <배포 개수>  // 개수를. kubectl scale 또는 kubectl edit 등의 명령어로 변경 가능
  selector:  // 이것과 일치하는 pod의 labels를 확인
    <key>: <value>
  template:
    metadata:
      name: <pod 이름>
      labels:
        <key>: <value>
    spec: ~~
```

##### replica set
replication controller와 같은 역할을 하지만 보다 풍부한 selector를 가지고 있다.  
`matchLabels` 는 replication controller의 selector와 같은 역할을 한다.  
`matchExpressions`는 아래와 같이 사용된다.  

```
selector:
  matchExpressions:
  - {key: tier, operator: In, values: [cache]}
  - {key: environment, operator: NotIn, values:[dev]}
```

##### deployment
rolling update / rolling back을 위해 자주 사용되며 replica set을 컨트롤해서 pod을 조절한다.  
즉, replicaset의 상위 개념으로 이해해도 되며, replicaset이 지워져도 deploy를 선언했다면 deploy에 의해 관리된다.  
`kubectl set image deplay <deploy 이름> <container 이름>=<container 이미지>:<이미지 버전> --record` 명령어를 통해 rolling update를 진행할 수 있다.  
`kubelctl rollout undo deployment <deploy 이름>` 명령어를 통해 rolling back을 진행할 수 있다.  

```
// yaml 예시. replica set과 비교할  kind만 제외하고 똑같이 사용할 수 있다.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <deployment 이름>
spec:
  replicas: <원하는 pod 수>
  selector:
    matchLabels:
      <key>: <value>
  template:
    metadata:
      name: <pod 이름>
      labels:
        <key>: <value>
    spec:
      containers:
        - name: <container 이름>
          image: <컨테이너 이미지>
```

##### deamon set
__전체 노드가 pod이 한 개씩 실행되도록 보장__ 하는 것이다.  
따라서 replicaset과 yaml 파일에서 `replicas`만 제외하고 동일하게 사용할 수 있다.  
로그 수입기, 모니터링 에이전트와 같은 프로그램 실행 시 적합하다.  
daemon set 또한 rolling update 기능이 존재한다.  

##### stateful set
__pod의 상태를 유지해주는 컨트롤러__ 이다.  
pod의 상태에는 pod의 이름, pod의 볼륨을 말한다.  
`kubectl run`을 통해서 pod을 실행하면 그 뒤에 이름이 자동으로 random hash값이 생성된다.  
하지만 stateful set 설정을 해주면 0부터 차례대로 붙고, scale out을 하면 순차적으로 숫자가 증가한다.  
반대로 하나씩 지우게 되면 가장 큰 숫자부터 사라지게 된다.  

```
// yaml 예시
apiVersion: apps/v1
kind: StatefulSet
metadata:
  ~
spec:
  replicas: 3
  serviceName: <서비스 이름>
  이하 동일..
```

##### job(cronjob의 하위 controller)
k8s는 기본적으로 pod을 running 중인 상태로 유지하려는 경향이 있다.  
그래서 한 번만 실행되기 원하는 서비스도, 작업이 정상적으로 완료된 서비스도 다시 실행시킨다.  
이를 해결하기 위해, 만약 __해당 작업을 완료한 뒤 pod을 종료시키고 싶다면__ job을 사용하면 된다.  
job은 batch 처리에 적합한 컨트롤러로 pod의 성공적인 완료를 보장한다.  
동작은 다음과 같다. 비정상 종료 시 pod을 다시 실행시키며, 정상 종료 시는 pod을 다시 실행시키지 않는다.  
단 이는 pod을 삭제한다는 의미가 아니라, pod을 "다시 실행시키지 않는다"라는 의미이므로 로그 등은 여전히 볼 수 있다.  


## 8. Service
4가지 종류가 있다.  
각각 cluster IP(default), node port, load balancer, external name이다.  
이와 달리 cluster IP가 없는 서비스로 단일 진입점이 필요 없을 때는 headless service를 사용하면 된다.  

##### cluster IP
동일한 서비스를 제공하는 pod 그룹의 단일 진입점을 제공하는 것을 말한다.  
deployment 등으로 하나의 서비스를 실행할 때 같은 동작을 하는 여러 컨테이너(pod)들이 서로 다른 노드에서 실행될 수 있다.  
이때 각각의 서비스들은 다른 IP를 갖게 되는데, label 설정을 통해 하나의 IP(Virtual IP)로 묶어서 관리할 수 있다.  
이 IP는 LB IP처럼 사용되며, control plane의 etcd에 기록된다.  

```
// yaml 예시
apiVersion: v1
kind: Service
metadata:
  name: <service 이름>
spec:
  clusterIP: x.x.x.x # 생략 가능
  selector:
    <key>: <value>
  ports:
  - protocol: TCP 등
    port: <cluster port>
    targetPort: <pod port>
```

##### node port
cluster IP를 기본적으로 포함하고 있다.  
추가적으로 모든 worker node에 외부에서 접속가능한 포트를 예약할 수 있다.  
이 worker node port를 통해 cluster IP 기능을 수행할 수 있다.  
default port 범위는 30000-32767이다.  

##### load balancer
node port를 기본적으로 포함하고 있다.  
클라우드 플랫폼(public cloud 환경)에서만 사용 가능하다.  
LB처럼 동작하는 것이 아니라 실제 LB를 자동으로 프로비전하는 기능을 지원한다.  

##### external name(DNS와 비슷한 서비스)
클러스터 내부에서 외부에 접속 시 사용할 도메인을 등록해서 사용 가능하다.  
클러스터 도메인이 실제 외부 도메인으로 치환되어 동작한다.  


## 9. ingress controller
HTTP나 HTTPS를 통해 클러스터 내부의 서비스를 외부로 노출시키는 것이다.  
이 앞까지의 방법을 통해서는 외부에서 내가 실행 중인 컨테이너들에 접근할 수 있는 방법이 없었다.  
ingress controller를 이용하면 가능하다.  
단 이를 이용하기 위해서는 k8s 이외에 별도의 설치가 필요하며 어떤 서비스와 매핑시킬지 ingress rule에 대한 설정이 필요하다.  

* 기능
  * Service에 외부 URL을 제공
  * 트래픽을 로드밸런싱
  * SSL 인증서 처리
  * Virtual hosting 지정
* 동작 방식

![화면 캡처 2022-04-02 182334](https://user-images.githubusercontent.com/63287638/161376773-e0a40088-8555-47ca-ad4b-7b7c1b0f5bd1.png)
<br/>
출처: https://www.youtube.com/watch?v=y5-u4jtflic&list=PLApuRlvrZKohaBHvXAOhUD-RxD0uQ3z0c&index=28
<br/>
      
```
// yaml(ingress rule) 예시
apiVersion: extensions/v1
kind: Ingress
metadata:
  name: <ingress 이름>
spec:
  rules:
  - http:
    paths:
    - path: /
      backend:
        serviceName: <서비스 이름>
        servicePort: <서비스 포트>
    - path: /another
      backend:
        serviceName: <서비스 이름>
        servicePort: <서비스 포트>
```


## 10. annotation
label과 동일하게 key-value를 통해 리소스의 특성을 기록하는 것이다.  
k8s에게 특정 정보 전달할 용도로 사용된다.  
예를 들어 Deployment의 rolling update 정보를 기록하고 한다면

```
annotations:
  kubernetes.io/change-cause: version 1.15
```

라고 사용할 수 있다.  

또는 아래와 같이 관리를 위해 필요한 정보를 기록할 용도로 사용할 수도 있다.

```
annotations:
  builder: "누가"
  buildDate: "언제"
  imageRegistry: "docker hub"
```


## 11. config map
일반적으로 개발 프로젝트를 진행할 때 설정 파일을 한 곳에서 모아 관리한 뒤 해당 설정 내용들을 갖다 쓰기도 한다.  
이것을 생각하면 config map의 용도가 쉽게 와닿을 것이다.  
config map은 컨테이너 구성 정보를 한 곳에 모아서 관리하는 것을 말한다.  
`kubectl create configmap <이름> [--from-file=source] [--from-literal=key1=value1]` 과 같은 식으로 사용 가능하며 파일에서 읽어올 수 있다.  
여기서 value는 1Mi를 넘지만 않는 어떠한 형식(json, nginx.conf 등)이든 들어갈 수 있다.  
      
```
// yaml 예시(선언)
apiVersion: v1
data:
  key1: value1
  key2: value2
kind: ConfigMap
metadata:
  ~~
```

이후 yaml 형식으로 config map을 적용할 수 있다.  

```
// yaml 예시(적용)
~~
spec:
  containers:
  -image: <컨테이너 이미지>
  env:
  - name: <환경 설정 이름>
  valueFrom:
    configMapKeyRef:
      name: <config map 이름>
      key: <config map의 key>
```





