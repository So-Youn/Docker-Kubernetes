# 도커 엔진

## Docker?

```markdown
기존의 `가상화 기술`은 하이퍼바이저를 이용해 여러 개의 운영체제를 하나의 호스트에서 생성해 사용하는 방식
이러한 여러 개의 운영체제는 가상 머신이라는 단위로 구별되고, 각 가상머신에는 운영체제가 설치되어 사용된다.
> 하이퍼바이저에 의해 생성되고 관리되는 운영체제는 게스트 운영체제`(Guest OS)`라 하고, 각 게스트 운영체제는 다른 게스트와는 완전히 독립된 공간과 시스템 자원을 할당받아 사용한다.
```



```markdown
도커 컨테이너는 가상화된 공간을 생성하기 위해 리눅스 자체기능인 chroot, 네임스페이스, cgroup을 사용함으로써 프로세스 단위의 격리 환경을 만들기 때문에 성능 손실이 거의 없다.
컨테이너에 필요한 커널은 `호스트의 커널을 공유`해 사용하고,컨테이너 안에는 애플리케이션을 구동하는 데 필요한 라이브러리 및 실행 파일만 존재하기 때문에 컨테이너를 이미지로 만들었을 때 이미지의 용량 또한 가상머신에 비해 대폭 줄어든다. 
```

* 도커 엔진에서 사용하는 기본 단위는 `이미지`와 `컨테이너`이며, 이 두가지가 **도커엔진의 핵심**이다.
* **이미지**는 컨테이너를 생성할 때 필요한 요소이며, 가상 머신을 생성할 때 사용하는 `iso` 파일과 비슷한 개념이다.

> 도커에서 사용하는 이미지의 이름은 **[저장소 이름]/[이미지 이름]:[태크]**의 형태로 구성된다.

## Docker Install

>  AWS EC2 사용하기 [Mac m1은 VirtualBox 사용이 불가하므로...]

* `.pem` 키 권한 변경

  ```shell
  soyun@yunsoyun-ui-MacBookPro docker % chmod 600 ubuntu-docker.pem
  ```

* 접속하기

  ```shell
  soyun@yunsoyun-ui-MacBookPro docker % ssh -i ubuntu-docker.pem ubuntu@13.209.18.250
  Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 5.4.0-1045-aws x86_64)
  
   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage
  ```

* 리눅스 도커 엔진 설치

  ```shell
  $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  OK
  
  $ add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  ''
  
  $ apt-get update
  $ apt-get install docker-ce
  ```

  * 설치 완료 후 도커 엔진 정보 출력

  ```shell
  $ docker info
  ...
  Server:
   Containers: 0
    Running: 0
    Paused: 0
    Stopped: 0
   Images: 0
   Server Version: 20.10.7
  ```

  * 도커 엔진 버전 확인

  ```shell
  $ docker -v
  Docker version 20.10.7, build f0df350
  ```

  

## 컨테이너 생성

* `docker run` : 컨테이너 생성 및 실행

  * `-i -t` 옵션은 컨테이너와 상호 입출력이 가능하게 한다.
  * `-i` : 상호 입출력
  * `-t` : `tty` 활성화해서 bash 셸 사용
  * `run` 명령어는 `pull`, `create`, `start` 명령어를 일괄적으로 실행한 후 컨테이너 내부로 들어간다.

  ```shell
  root@ip-172-31-5-197:~# docker run -i -t ubuntu:14.04
  Unable to find image 'ubuntu:14.04' locally
  # ubuntu:14.04 이미지가 로컬 도커 엔진에 존재하지 않으므로 도커 중앙 이미지 저장소인 도커 허브에서 자동으로 이미지 내려받음
  Status: Downloaded newer image for ubuntu:14.04
  ```

* `docker pull` : 이미지를 내려받을 때 사용

  ```shell
  root@ip-172-31-5-197:~# docker pull centos:7
  7: Pulling from library/centos
  2d473b07cdd5: Pull complete 
  Status: Downloaded newer image for centos:7
  docker.io/library/centos:7
  ```

* `docker images` : 도커 엔진에 존재하는 이미지의 목록 출력

  ```shell
  root@ip-172-31-5-197:~# docker images
  REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
  ubuntu       14.04     13b66b487594   3 months ago   197MB
  centos       7         8652b9f0cb4c   8 months ago   204MB
  ```

  * 컨테이너를 대상으로 하는 모든 명령어는 컨테이너 이름 대신 ID를 대신 쓸 수 있다. ( 너무 길 때는 앞의 2~3글자만 써도 OK. )

* `docker ps` : 생성한 컨테이너 목록 + 정지되지 않은 컨테이너만 출력한다.

  * 정지된 컨테이너를 포함한 모든 컨테이너 출력하려면 `-a` 옵션 추가
  * `exit` 사용해 빠져나온 컨테이너는 정지 상태
  * `Ctrl + P, Q` 입력해 빠져나온 컨테이너는 실행중

  ```shell
  root@ip-172-31-5-197:~# docker ps
  CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
  root@ip-172-31-5-197:~# docker ps -a
  CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                     PORTS     NAMES
  608717f340f5   ubuntu:14.04   "/bin/bash"   12 minutes ago   Exited (0) 8 minutes ago             beautiful_fermi
  ```

* `docker rm` : 컨테이너 삭제

  * `docker rm -f ID` : 실행 중인 컨테이너 삭제
  * `docker container prune` : 모든 컨테이너 삭제

* `docker stop` : 컨테이너 정지

```shell
# 모든 컨테이너 중지 및 삭제 
$ docker stop $(docker ps -a -q)
$ docker rm $(docker ps -a -q)
```



## 컨테이너 외부에 노출

> 컨테이너는 가상 머신과 마찬가지로 <u>가상 IP 주소</u>를 할당받음
>
> 도커는 컨테이너에 `172.17.0.x` 의 IP를 순차적으로 할당한다.

```shell
root@9ca443c7e215:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02  
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:9 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:806 (806.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
 # NAT IP 인 172.17.0.2를 할당받은 eth0 인터페이스
 # 로컬 호스트인 lo 인터페이스
```



* 외부에 컨테이너의 애플리케이션을 노출하기 위해서는 **eth0의 IP와 포트**를 **호스트의 IP와 포트**에 바인딩 해야한다.

   <u>[호스트의 포트]:[컨테이너의 포트]</u>

```shell
root@ip-172-31-5-197:~# docker run -i -t --name mywebserver -p 80:80 ubuntu:14.04
```

* 호스트의 특정 IP를 사용하려면 바인딩할 `IP:Port` 를 명시한다.

```shell
root@ip-172-31-5-197:~# docker run -i -t -p 192.168.0.100:7777:80 ubuntu:14.04
```

* 아파치 웹 서버 설치

```shell
$ apt-get update
$ apt-get install apache2 -y
$ service apache2 start
```

