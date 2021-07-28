* 스테이트리스(stateless) : 컨테이너가 아닌 외부에 데이터를 저장하고 컨테이너는 그 데이터로 동작하도록 설계하는 것
  * 컨테이너 자체는 상태가 없고 상태를 결정하는 데이터는 외부로부터 제공받는다.
  * 컨테이너가 삭제돼도 데이터는 보존되므로 권고되는 설계
* 스테이트풀(stateful) : 컨테이너가 데이터를 저장하고 있어 상태가 있음
  * 컨테이너 자체에서 데이터를 보관하므로 지양하는 설계

# 도커 네트워크

```markdown
도커는 컨테이너에 내부 IP를 순차적으로 할당하며, 이 IP는 컨테이너를 재시작할 때마다 변경될 수 있음.
이 내부 IP는 도커가 설치된 호스트, 즉 **내부 망**에서만 쓸 수 있는 IP이므로 외부와 연결될 필요가 있다.
> 이 과정은 컨테이너를 시작할 때마다 호스트에 `veth..`라는 네트워크 인터페이스를 생성함으로써 이뤄진다.
> veth에서 v는 virtual을 의미한다. 즉, virtual eth
```

* `eth0` 는 <u>공인IP</u> 또는 내부 IP가 할당되어 <u>실제로 외부와 통신할 수 있는</u> 호스트의 네트워크 인터페이스.
* `veth` 로 시작되는 인터페이스는 컨테이너를 시작할 때 생성됐으며, 각 컨테이너의 `eth0` 와 연결됨.
* `brctl` 명령어를 통해 `docker0` 브리지에 `veth`이 실제로 바인딩 됐는지 확인 가능함

```shell
$ apt install bridge-utils

$ brctl show docker0
bridge name	bridge id					STP enabled	interfaces
docker0		  8000.0242c346607f	no					vethb462209
																					vethc4ca298
```

* 컨테이너를 생성하면 기본적으로 `docker0` 브리지를 통해 외부와 통신할 수 있는 환경을 사용할 수 있지만 사용자의 선택에 따라 여러 네트워크 드라이버를 쓸 수도 있다. 
* `docker network` : 도커의 네트워크를 다루는 명령어

```shell
# 도커에서 기본적으로 쓸 수 있는 네트워크 확인 - bridge, host, none
root@ip-172-31-11-242:~# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
2aa95336c791   bridge    bridge    local
e8d0cc7f2027   host      host      local
5e14d745b85a   none      null      local
```

```shell
$ docker network inspect bridge
...
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
     
```

## 브리지 네트워크

```markdown
docker0 브리지와 비슷하게 `브리지 네트워크`는 docker0이 아닌 사용자 정의 브리지를 새로 생성해 각 컨테이너에 연결하는 네트워크 구조이다.
> 새로운 브리지 타입의 네트워크 생성
```

```shell
# bridge 타입의 mybridge라는 네트워크 생성
$ docker network create --driver bridge mybridge
8f72ccddd7715e687d47d043ff0858a8645fc9cd951818bca5bea67d7721cb12
```

* `docker run` or `create` 명령어에 `--net` 옵션 값을 설정하면 컨테이너가 이 네트워크를 사용하도록 설정할 수 있다.

```shell
$ docker run -i -t --name mynetwork_container \
--net mybridge \
ubuntu:14.04

# 컨테이너 내부에서 ifconfig 입력하면 새로운 IP대역 생성된 것 확인 가능
root@b4032e9d6088:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:12:00:02  
          inet addr:172.18.0.2  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:12 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1112 (1.1 KB)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
  
```

* 이렇게 생성된 네트워크는 `docker network disconnect` `connect` 를 통해 컨테이너에 유동적으로 붙이고 뗄 수 있다.

```shell
root@ip-172-31-11-242:~# docker network disconnect mybridge mynetwork_container
root@ip-172-31-11-242:~# docker network connect mybridge mynetwork_container
```

* `none` 네트워크

  * 말 그대로 아무런 네트워크를 쓰지 않는 것을 의미한다.
  * none네트워크 생성 시 외부와 연결이 단절된다.

  ```shell
  $ docker run -i -t --name network_none \
  > --net none \
  > ubuntu:14.04
  root@f0fa75672cfb:/$ ifconfig
  lo        Link encap:Local Loopback  
            inet addr:127.0.0.1  Mask:255.0.0.0
            UP LOOPBACK RUNNING  MTU:65536  Metric:1
            RX packets:0 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000 
            RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
            # 로컬호스트를 나타내는 lo외에는 존재하지 않음.
  ```

* dig : DNS로 도메인 이름에 대응하는 IP를 조회할 때 쓰는 도구

  ```shell
  $ apt-get update
  $ apt-get install dnsutils
  ```

## MacVLAN 네트워크

```markdown
`MacVLAN`은 호스트의 네트워크 인터페이스 카드를 가상화해 물리 네트워크 환경을 컨테이너에게 동일하게 제공한다.
따라서 MacVLAN을 사용하면 컨테이너는 물리 네트워크상에서 가상의 MAC주소를 가지며, `해당 네트워크에 연결된 다른 장치`와의 통신이 가능해진다.
MacVLAN에 연결된 컨테이너는 기본적으로 할당되는 IP대역인 172.17.x.x 대신 네트워크 장비의 IP를 할당하기 때문이다.

* MacVLAN을 사용하는 컨테이너는 기본적으로 호스트와 통신이 불가능하다.
* MacVLAN을 사용하려면 적어도 1개의 네트워크 장비와 서버가 필요하다.
```

* `-d` : 네트워크 드라이버로 macvlan을사용한다는 것을 명시함. (--driver)
* `--subnet` : 컨테이너가 사용할 네트워크 정보를 입력한다.



## 컨테이너 로깅

* 도커는 컨테이너의 표준 출력(stdOut)과 에러(StdErr) 로그를 별도의 **메타데이터 파일**로 저장하며 이를 확인하는 명령어를 제공한다.

```shell
# 컨테이너 생성해 간단한 로그 남기기
$ docker run -d --name mysql \
-e MYSQL_ROOT_PASSWORD=1234 \
mysql:5.7
# mysql과 같은 애플리케이션을 구동하는 컨테이너는 포그라운드 모드로 실행되므로 -d 옵션을 써서 백그라운드 모드로 컨테이너를 생성하는 경우가 많다.
```

* `docker logs` : 컨테이너 내부에서 출력을 보여주는 명령어

``` shell
$ docker logs mysql
```

```shell
$ docker run -d --name no_passwd_mysql \
mysql:5.7

# 위의 명령어 실행 후 docker ps로 컨테이너 목록 확인해보면 no_passwd_mysql이 생성은 됐으나 실행이 되지 않는 것 확인 가능함
$ docker ps --fomat "talbe {{.ID}}\t{{.Status}}\t{{Ports}}\t{{.Names}}"

$ docker start no_passwd_mysql
no_passwd_mysql 

$ docker ps --fomat "talbe {{.ID}}\t{{.Status}}\t{{Ports}}\t{{.Names}}"

# 컨테이너 시작 안됨
```

* `docker logs`를 통해 애플리케이션에 무슨 문제가 있는 지 확인해보기

```shell
# mysql 실행에 필요한 환경변수를 지정하지 않아 컨테이너가 시작되지 않은 것 알 수 있음
$ docker logs no_passwd_mysql
$ docker logs --tail 2 mysql # 마지막 2줄만 출력
```

```shell
# 특정 시간 이후의 로그 확인
$ docker logs --since 1474765979 mysql

# -t 옵션 : 타임스탬프
$ docker logs -f -t mysql
```

* 컨테이너 로그는 JSON 형태로 도커 내부에 저장된다.

```shell
$ cat /var/lib/docker/containers/${CONTAINER_ID}/${CONTAINER_ID}-json.log
```

* `--log-opt` 옵션으로 컨테이너 json 로그 파일의 최대 크기 설정 가능
* `max-size` 는 로그 파일의 최대 크기, `max-file` 은 로그 파일의 개수를 의미한다.

### syslog

```shell
$ docker run -d --name syslog_container \
--log-driver=syslog \
ubuntu:14.04 \
echo syslogtest
```

* Syslog 로깅 드라이버는 기본적으로 로컬호스트의 syslog에 저장한다.
  * 우분투 14.04는 `/var/log/syslog`
* CentOS와 RHEL은 `/var/log/messages`

```shell
# 우분투 syslog
$ tail /var/log/syslog
```

* `rsyslog` : 중앙 컨테이너로 로그를 저장

```shell
$ docker run -i -t \
-h rsyslog \
--name rsyslog_server \
-p 514:514 -p 514:514/udp \
ubuntu:14.04
```

```shell
# 컨테이너 내부의 rsyslog.conf 파일을 열어 syslog 서버를 구동시키는 항목의 주석을 해제한 후 변경사항 저장
$ vi /etc/rsyslog.conf

# rsyslog 서비스 재시작
$ service rsyslog restart
```

## 아마존 클라우드워치 로그

1. 클라우드워치에 해당하는 IAM 권한 생성
2. 