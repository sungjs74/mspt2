# Docker & Kubernetes 노트 (MSP T2 10차)  (✿◡‿◡)
## Q&A

### Docker

- Docker build를 했는데 교재/강사화면과 다르네요. 왜 그런가요? ( FROM docker.io/library/node:10-alpine ... )
  - 빌드하는 시점/환경에 따라 다를 수 있습니다.
  - 뒤에 배우겠지만, 레이어에 해당하는 것이 호스트 머신에 있을 때와 없을 때 차이가 있을 수 있습니다.

- container layer ( R/W layer ) 디렉토리에서 `-init` 이 붙는건 뭔가요?
  - layer 디렉토리에는 파일/디렉토리가 있습니다.  (최 하위 레이어는 link와 diff만 있음.)
    - link (파일) : overlay2/l 디렉토리에 존재하는 링크 정보
    - diff (디렉토리) : 그 레이어의 contents (파일들) 가 존재함.
    - lower (파일) : lower layer에 대한 링크정보
    - merged (디렉토리) : lower layer(의 컨텐츠)와 자신의 레이어(의 컨텐츠)가 병합된 컨텐츠가 존재함. 
    - work (디렉토리) : storage driver에 의해 사용되는 디렉토
   - 컨테이너가 생성되면 같은 이름의 디렉토리가 두 개 생성되고, 그 중 하나는 `-init`이 뒤에 붙어있음.
   - init이 그 하위의 레이어들을 lower에 담고있고, init이 아닌 디렉토리의 link는 init까지를 lower로 담고있음. (아래 예시 참조.) -> init보다 init이 없는 것이 더 상위.
  - [How the overlay2 driver works](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#how-the-overlay2-driver-works)도 참고하세요.  
  - [Docker image layer 정보 확인](https://ikcoo.tistory.com/396) 에도 설명이 잘 되어있네요.
```bash
ubuntu $ tree -L 2
.
|-- 19c812ab930a19683baf95ea2d12c403ef21aba75f5581e0004801d952de2fcf
|   |-- committed
|   |-- diff
|   |-- link
|   |-- lower
|   `-- work
|-- 3f30e4eb9605634a84e913c3d3204b624d32c8e5bb3d4f930079f20fb28d3df5
|   |-- committed
|   |-- diff
|   `-- link
|-- 67e24567f32ee7e5614688f8db47b8b01dc35411fcdccac1f0da6ca18ddd0ef3
|   |-- committed
|   |-- diff
|   |-- link
|   |-- lower
|   `-- work
|-- 95d81d5e704106af63d3e55fa0e5e35e21f84f97b2a328677d23cc63c2289448
|   |-- committed
|   |-- diff
|   |-- link
|   |-- lower
|   `-- work
|-- c605c2d746f17ac17476e1cb88719cba0c9f29d8e2e79dcf0d4beb586f4b4cda
|   |-- committed
|   |-- diff
|   |-- link
|   |-- lower
|   `-- work
|-- e95f12761991634699e80b56e9510162dc7efacaba6ae5f461d12d29295fe154
|   |-- diff
|   |-- link
|   |-- lower
|   |-- merged
|   `-- work
|-- e95f12761991634699e80b56e9510162dc7efacaba6ae5f461d12d29295fe154-init
|   |-- committed
|   |-- diff
|   |-- link
|   |-- lower
|   `-- work
|-- ec45a49fa3c5ad838aa073510debc690dae3c425252eb558414ece53c67fa244
|   |-- committed
|   |-- diff
|   |-- link
|   |-- lower
|   `-- work
`-- l
    |-- 22VBECB4VVVAPE3KULZX6UYBCM -> ../3f30e4eb9605634a84e913c3d3204b624d32c8e5bb3d4f930079f20fb28d3df5/diff
    |-- 5R2VMDDUASJCZGLXNBCX3VKSH4 -> ../e95f12761991634699e80b56e9510162dc7efacaba6ae5f461d12d29295fe154-init/diff
    |-- BIATFAJNMCEW5EZT4V4YZI6V3P -> ../19c812ab930a19683baf95ea2d12c403ef21aba75f5581e0004801d952de2fcf/diff
    |-- CEDMQDL3V5ZU3TU5JCJWDZLK4A -> ../ec45a49fa3c5ad838aa073510debc690dae3c425252eb558414ece53c67fa244/diff
    |-- GWFDJRLMH7UJYSX4DGQEE5RXWZ -> ../67e24567f32ee7e5614688f8db47b8b01dc35411fcdccac1f0da6ca18ddd0ef3/diff
    |-- OJRYLRWZC7HCV3FH6JZYFKSAAR -> ../c605c2d746f17ac17476e1cb88719cba0c9f29d8e2e79dcf0d4beb586f4b4cda/diff
    |-- PG22LLBRRZDUSR5REQDX7B34O2 -> ../95d81d5e704106af63d3e55fa0e5e35e21f84f97b2a328677d23cc63c2289448/diff
    `-- R3UKY4B6QBUYZOW2NU76WMU3QW -> ../e95f12761991634699e80b56e9510162dc7efacaba6ae5f461d12d29295fe154/diff

33 directories, 22 files

ubuntu $ cat ./e95f12761991634699e80b56e9510162dc7efacaba6ae5f461d12d29295fe154-init/lower 
l/BIATFAJNMCEW5EZT4V4YZI6V3P:l/CEDMQDL3V5ZU3TU5JCJWDZLK4A:l/PG22LLBRRZDUSR5REQDX7B34O2:l/GWFDJRLMH7UJYSX4DGQEE5RXWZ:l/OJRYLRWZC7HCV3FH6JZYFKSAAR:l/22VBECB4VVVAPE3KULZX6UYBCMubuntu $ 

ubuntu $ cat ./e95f12761991634699e80b56e9510162dc7efacaba6ae5f461d12d29295fe154/lower 
l/5R2VMDDUASJCZGLXNBCX3VKSH4:l/BIATFAJNMCEW5EZT4V4YZI6V3P:l/CEDMQDL3V5ZU3TU5JCJWDZLK4A:l/PG22LLBRRZDUSR5REQDX7B34O2:l/GWFDJRLMH7UJYSX4DGQEE5RXWZ:l/OJRYLRWZC7HCV3FH6JZYFKSAAR:l/22VBECB4VVVAPE3KULZX6UYBCMubuntu $ 
```
> `-init`이 안붙은 디렉토리의 lower가 link를 하나 더(`-init` 디렉토리까지 포함.) 가지고 있음.

- docker images 해보면 k8s.gcr.id~ 로 시작하는 이미지들이 보입니다.이것들 rmi 해도 되나요?
  - 아니되옵니다. Σ(っ °Д °;)っ ( docker desktop 환경인 경우, kubernetes 구동에 사용되는 것들입니다. )

- /var/lib 아래 docker-desktop이 보이는데 이건 뭔가요?
  - 교재에 말씀드린 `\\wsl.localhost\docker-desktop-data\data\docker` 또는 `\\wsl$\docker-desktop-data\data\docker` 가 docker root 입니다.
  - [Docker 이미지 스토리지 폴더 찾기 문제](https://learn.microsoft.com/ko-kr/windows/wsl/tutorials/wsl-containers#trouble-finding-docker-image-storage-folder) 도 참고하세요.

- run 상태의 이미지는 삭제가 안되나요?
  - 네, 안됩니다. (R/O 레이어는 사용해야 하거든요)
  - 아래 예제 참고하세요.
```bash
ubuntu@ip-10-0-1-205:~/app$ docker run -d nginx
e56ac97ef60a3923b119490978af9785f7c97c9b5d27d4f53e2c85d82dc59789
ubuntu@ip-10-0-1-205:~/app$ docker images
REPOSITORY      TAG         IMAGE ID       CREATED          SIZE
docker-101      latest      de63c73e314b   59 minutes ago   172MB
nginx           latest      76c69feac34e   2 weeks ago      142MB
my-ubuntu       v3          d863c9811ce7   3 weeks ago      231MB
nginx           <none>      51086ed63d8c   5 weeks ago      142MB
todo-test       latest      0bad600f4bd9   7 weeks ago      172MB
busybox         latest      2bd29714875d   2 months ago     1.24MB
nginx           <none>      2d389e545974   2 months ago     142MB
centos          latest      5d0da3dc9764   14 months ago    231MB
node            10-alpine   aa67ba258e18   19 months ago    82.7MB
nginx           1.18.0      c2c45d506085   19 months ago    133MB
nginx           my-tag      c2c45d506085   19 months ago    133MB
rogallo/nginx   1.18.0      c2c45d506085   19 months ago    133MB
ubuntu@ip-10-0-1-205:~/app$ docker rmi nginx
Error response from daemon: conflict: unable to remove repository reference "nginx" (must force) - container e56ac97ef60a is using its referenced image 76c69feac34e
```

---

- Volume은 docker root (/var/lib/docker/)를 사용하니 bind mount가 고성능 I/O에 더 유리하지 않나요?
  - 성능차이는 Storage driver를 거치느냐, 아니냐의 차이여서 volume과 bind mount와는 차이가 없습니다. (둘 다 host machine의 I/O를 사용)
  - R/W layer는 Storage driver를 통해서 I/O를 하기 때문에 성능 차이가 나게 됩니다.
  - [About storage drivers](https://docs.docker.com/storage/storagedriver/)와 [Use volumes](https://docs.docker.com/storage/volumes/) 참고하세요.
  - [Performance best practices](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#performance-best-practices) 도 참고하세요.


---

- Docker의 Bridge가 로드밸런싱 역할을 해주는건가요?
  - 그렇지는 않습니다. 네트워크 구간에서 서브 네트워크를 가지는 G/W역할을 합니다.
  - LB는 쿠버네티스에서 더 자세히 배울겁니다.

- 컨테이너간 통신을 하려면 IP를 알아야 할 것 같은데 직접 할당 가능한가요?
  - IP보다는 Name이나 Alias를 사용하는 것이 일반적입니다.
  - docker run 명령어에 `--ip` 라는 옵션을 쓸 수는 있습니다. (e.g. `docker run --ip 172.30.100.104` )

- CIDR에서 마지막 16은 뭔가요?
  - 앞에서 부터 16 비트라는 의미입니다.
  - [CIDR](https://ko.wikipedia.org/wiki/CIDR) 참고하세요.

- docker run -p 3000:3000의 의미를 다시 설명해주세요.
  - 앞은 host machine의 포트, 뒤는 컨테이너의 포트입니다. 둘을 연결하여 publish 한다는 의미 입니다.

- dockerfile과 docker-compose는 어떻게 다른건가요?
  - dockerfile은 하나의 이미지를 만들기위한 명령어들의 집합이고,
  - docker compose는 여러개의 컨테이너를 구성할 때 사용되는 것입니다.
  - 예를들어 todo app과 mySql을 만들때는 각각의 dockerfile을 사용하고, 실행할 때 docker compose로 한 번에 실행할 수 있습니다.

- docker run 옵션에서 -d(detached)나 -i(interactive)말고 둘 다 없이 하는 경우도 실제 쓰이나요?
  - `-d`를 생략하면 기본적으로는 foreground 모드 입니다.
  - [Detached vs foreground](https://docs.docker.com/engine/reference/run/#detached-vs-foreground) 의 Foreground 부분 참고하세요~

- dockerfile도 선언형으로 생성할 수 있나요? (이미지도 선언형으로 생성할 수 있나요?)
  - 없는 듯 합니다. (도커 자체만으로는...)
  - 다른 방법으로 선언형 구성을 지원하는 다른 툴을 이용해서 컨테이너 이미지를 만드는 방법은 있는 것 같습니다.


- `exit`를 했을 때 컨테이너가 종료되지 않도록 하거나 자동 재시작하도록 하는 방법이 있나요?
  - `exit`는 종료(stop), `ctrl+p,q`는 종료하지 않고 빠져나오기 입니다.
  - 아래 box1은 exit로 나온 경우이고(container가 stop된 상태), box2는 ctrl+p,q로 나온 경우(container가 여전히 running상태)입니다.
```bash
ubuntu@ip-10-0-1-205:~$ docker run -it --name box1 busybox sh
/ # exit
ubuntu@ip-10-0-1-205:~$ docker run -it --name box2 busybox sh
/ # ubuntu@ip-10-0-1-205:~$
ubuntu@ip-10-0-1-205:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                      PORTS     NAMES
99afe3cc6025   busybox   "sh"      8 seconds ago    Up 7 seconds                          box2
508b37c96942   busybox   "sh"      28 seconds ago   Exited (0) 25 seconds ago             box1
```

---

### Kubernetes

- K8s에서 pod가 도커에서 컨테이너의 의미인가요?
  - 엄밀하게는 다르긴 하지만, 거의 같은 의미로 쓰이고 있습니다. ( 하지만, 우리는 이 과정을 듣고나면 `다르다`고 자신있게 말할 수 있습니다. ...(*￣０￣)ノ    )
  - 내일 Workload에서 자세히 다룰 예정입니다.

- kubectl get pods 하면 어떤 노드의 pod인지 어떤 선언의 의미인가요?
  - `--output wide` 옵션을 사용하면 어떤 노드에 실행되고 있는지도 알 수 있습니다.
  - 더 자세한 정보는 `kubectl describe pods OOO --namespace XXX`와 같은 명령어를 사용하시면 됩니다.

- `kubectl get` 이나 `kubectl describe` 명령어에서 뒤에 `pod` `pods` 둘 다 가능한가요?
  - 네. 단/복수형 다 됩니다. (문서나 help에는 복수로 나와있기는 합니다.)

- yaml 파일에서 키 앞에 -(dash)가 있는게 있고, 없는게 있는데 무슨 차이인가요?
  - parent가 복수형인 경우 (e.g. containers:) 그 하위의 정보는 시작 라인 앞에 - 가 붙어있습니다.
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: multipod
spec:
 containers:      
 - name: nginx-container	# 첫번째 컨테이너
   image: nginx:1.14
   ports:
   - containerPort: 80
 - name: centos-container	# 두번째 컨테이너
   image: centos:7
   command:
   - sleep
   - "10000"  
```

- `k`는 alias인가요?
  - 네, 맞습니다.
  - kubectl 설치 후 간편한 사용(alias, auto-completion)을 위한 환경설정은 [리눅스에서 bash 자동 완성 사용하기](https://kubernetes.io/ko/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/)를 참고하세요.


- Readiness probe 는 fail 시 어떻게 동작하나요?
  - Readiness probe는 Service 의 엔드포인트에 Pod를 포함시킬지를 판단하는 것입니다. Fail일 경우(아직 준비가 덜 된 경우)는 Service에서 해당 Pod로의 네트워킹을 하지 않습니다.
  - Service에 대해서는 뒤에 더 자세히 배우게 됩니다.

- Readiness Probe와 Liveness Probe의 차이는 노드와 서비스의 모니터링 차이인가요?
  - 둘 다 Pod의 상태를 체크하는 것인데요, Readiness는 사용자에게 연결할지를(service의 LB 대상으로 할지, endpoint로 할지) 판단하는 것이고, Liveness는 Running중인 pod의 상태를 체크해서 문제가 생기면 재시작을 해주는 것입니다.

- 명령형으로 생성한 오브젝트를 추후 선언형으로 적용할 수 있나요?
  - ...

- `kubectl delete -f ~`로 하면 명령형 오브젝트랑 같은 거 아닌가요?
  - 정확하게 말씀하셨네요. 맞습니다.
  - 생성/수정은 선언형 오브젝트 구성(`kubectl apply -f ~`)로 하고, 삭제는 명령형 오브젝트 구성 (`kubectl delete -f ~`)으로 하는 것입니다.
  - 삭제도 apply 명령으로 할 수는 있지만, 권장하지는 않습니다. [오브젝트 삭제 방법](https://kubernetes.io/ko/docs/tasks/manage-kubernetes-objects/declarative-config/#%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8-%EC%82%AD%EC%A0%9C-%EB%B0%A9%EB%B2%95) 참고하세요.

- 샘플에 app과 name들이 모두 동일해서 헷갈리네요.
  - ...

- pod를 굳이 replicaset, deployment를 사용해서 만드는 이유가 있나요?
  - 뒤에서 배우겠지만, scale이나 update 등을 위해서 상위 오브젝트인 deployment를 활용하는 것이 좋습니다.
  - pod를 직접 만들고 관리할 수도 있지만, scale이나 update등의 작업이 필요한 경우 개별적인 핸들링이 필요합니다. 또한, history관리도 개별적으로 해야하구요.

- deployment를 이용한 업데이트 시 pod들이 삭제되고 새로 생성되는데요, 이럴경우 추적성이 좀 떨어질 것 같습니다. 기존 pod를 업데이트할 수는 없나요?
  - 컨테이너의 개념은 새로 생성하는것이 기본입니다.
  - 수업시간에 자세히 설명드릴게요.

- pod의 개수를 조정할 때 (scale in/out) 수행되고 있는 트랜잭션의 정상적인 종료는 보장하나요?
  - 네, 그런 부분이 고려되어 있습니다.
  - 기존의 Transaction처리를 보장하기 위해서 Graceful shutdown을 하게됩니다. (SIGTERM신호를 먼저 보내고, 이후에 SIGKILL)
  - [파드의 종료](https://kubernetes.io/ko/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination) 부분 참고하세요.

- http request로도 readiness 체크 가능한가요?
  - 네, 가능합니다. tcp, http 다 가능합니다. (liveness probe와 동일)

- 예제에서 deployment에서 replicas를 5로 설정하면 지속적으로 5개가 생성되나요?
  - k8s는 그렇게 하려고 노력합니다.
  - 하지만, 자원이 부족하여 pod를 생성할 수 없는 경우에는 원하는 대로 안만들어지기도 합니다. (자원에 여유가 있다면 원하는 spec대로 생성하는 것을 보장합니다.)

- pod가 생성될 때 readiness probe가 liveness probe로 바뀌는건가요?
  - 아니오, 두 가지를 다 설정하게 되어있습니다.
  - [Define a TCP liveness probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-tcp-liveness-probe)를 보시면 두 가지를 다 설정한 샘플이 있습니다.
  - readiness probe는 시작단계에만 관여하고, 그 이후에는 liveness probe가 체크를 합니다.

- 
