# Docker Image

## 컨테이너 애플리케이션 구축

* `mysql` 이미지를 이용해 DB 컨테이너 생성

```shell
$ docker run -d --name wordpressdb \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
mysql:5.7
```

* 미리 준비된 워드프레스 이미지 사용해 워드프레스 웹 서버 컨테이너 생성

```shell
root@ip-172-31-11-242:~# docker run -d \
> -e WORDPRESS_DB_HOST=mysql \
> -e WORDPRESS_DB_USER=root \
> -e WORDPRESS_DB_PASSWORD=password \
> --name wordpress \
> --link wordpressdb:mysql \     # wordpressdb 컨테이너를 mysql라는 이름으로 설정.(alias)
> -p 80 \     # 호스트 포트 중 하나와 컨테이너의 80번 포트 연결시킴
> wordpress
```

```shell
root@ip-172-31-11-242:~# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                                     NAMES
46cedc39d01b   wordpress   "docker-entrypoint.s…"   48 seconds ago   Up 46 seconds   0.0.0.0:49153->80/tcp, :::49153->80/tcp   wordpress
2dd71681bbab   mysql:5.7   "docker-entrypoint.s…"   6 minutes ago    Up 6 minutes    3306/tcp, 33060/tcp                       wordpressdb
# 바인딩된 포트만 확인
root@ip-172-31-11-242:~# docker port wordpress
80/tcp -> 0.0.0.0:49153
80/tcp -> :::49153
# host의 49153 포트와 연결됨
```



`--link` : 내부 IP 알 필요 없이 항상 컨테이너에 별명으로 접근하도록 설정.

* `-e` : 컨테이너 내부의 환경변수 설정.

  ```shell
  ... -e MYSQL_ROOT_PASSWORD=password
  
  # 컨테이너 내부에서 환경변수 확인 : 배시 셸 사용하는 환경 필요함
  $ echo ${ENVIRONMENT_NAME}
  ```

* `exec` : `컨테이너 내부 셸` 바로 사용

  ```shell
  root@ip-172-31-11-242:~# docker exec -i -t wordpressdb /bin/bash
  root@2dd71681bbab:/# echo $MYSQL_ROOT_PASSWORD
  password
  ```

  * 옵션 추가하지 않으면 내부에서 실행한 명령어에 대한 결과만 반환.

  ```shell
  root@ip-172-31-11-242:~# docker exec wordpressdb ls
  bin
  boot
  dev
  docker-entrypoint-initdb.d
  ...
  ```

* 설정된 환경변수가 MySQL에 적용됐는지 확인

  ```shell
  root@2dd71681bbab:/# mysql -u root -p
  Enter password: 
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 4
  Server version: 5.7.35 MySQL Community Server (GPL)
  ...
  mysql> 
  ```

  

## 이미지 에러

> requested access to the resource is denied. 

* [Docker hub](https://hub.docker.com/signup) 회원가입

* Docker 로그인

  ```shell
  root@ip-172-31-11-242:~# docker login
  Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
  Username: soyun3963
  Password: 
  WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
  Configure a credential helper to remove this warning. See
  https://docs.docker.com/engine/reference/commandline/login/#credentials-store
  
  Login Succeeded
  ```



# Docker Volume

```markdown
도커 이미지로 컨테이너 생성하면 이미지는 읽기 전용이 되며, 컨테이너의 변경 사항만 별도로 저장해서 각 컨테이너의 정보를 보존한다.

도커 컨테이너 레이어(쓰기 가능)
-----------------------
도커 이미지 (읽기 전용)

> 이렇게 생성된 이미지는 어떠한 경우로도 변경되지 않으며, 컨테이너 계층에 이미지에서 변경된 파일시스템을 저장한다.
```

```markdown
But, 컨테이너 삭제하면 컨테이너 계층에 저장된 DB정보도 삭제된다는 치명적인 단점이 존재함.
> 데이터 복구 불가능

이를 방지하고자 컨테이너의 데이터를 영속적으로 활용할 수 있는 방법 : **볼륨**
```

- `-v` : 디렉터리 공유

  **[host 공유 디렉토리]:[컨테이너 공유 디렉토리]**

```shell
$ docker run -d --name wordpressdb \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
-v /home/wordpress_db:/var/lib/mysql \
mysql:5.7
# 컨테이너의 /var/lib/mysql 디렉터리는 MySQL이 데이터베이스의 데이터를 저장하는 기본 디렉터리이다.
# /home/wordpress_db == /var/lib/mysql
```

```shell
$ docker run -i -t \
--name volume_overide \
-v /home/wordpress_db:/home/testdir_2 \
alicek106/volume_test
```

* `--volumes-from` : `-v` or `--volume` 을 적용한 컨테이너의 볼륨 디렉토리 공유 가능 ( `-v`  옵션을 적용한 컨테이너를 통해 공유)

  > Ex) 위에서 생성한 volume_overide컨테이너는 /home/testdir_2 디렉터리를 호스트와 공유하고 있다. 
  >
  > 이 컨테이너를 볼륨 컨테이너로서 volumes_from_container 컨테이너에 다시 공유하는 것임

```shell
$ docker run -i -t \
--name volumes_from_container \
--volumes-from volume_overide  \
ubuntu:14.04

root@282a7a1c20ed:/# ls /home/testdir_2/
auto.cnf         client-key.pem  ibdata1             private_key.pem  sys
ca-key.pem       ib_buffer_pool  ibtmp1              public_key.pem   wordpress
ca.pem           ib_logfile0     mysql               server-cert.pem
client-cert.pem  ib_logfile1     performance_schema  server-key.pem
```

* `docker volume` 명령어

```shell
# 볼륨 생성
$ docker volume create --name myvolume
myvolume

# 생성된 볼륨 확인
$ docker volume ls
DRIVER    VOLUME NAME
local     c84c33b757bd0999555d1f1bd39df480032db5f9ed548f5609083f2e592d7ecd
local     myvolume
```

```shell
$ docker run -i -t --name myvolume_1 \
-v myvolume:/root/ \
ubuntu:14.04

$ echo hello,volume! >> /root/volume # root 디렉터리에 volume 파일 생성
```

```shell
$ docker run -i -t --name myvolume_2 \
-v myvolume:/root/ \
ubuntu:14.04

# 확인해보면 volume 파일 존재함
$ cat /root/volume
hello,volume!
```

* 호스트에 저장함으로써 데이터를 보존하지만 파일이 실제로 어디에 저장되는 지 사용자는 알 필요 없음.
* `docker inspect` 명령어를 사용해 볼륨이 실제 어디에 저장되는 지 알 수 있음
  * 컨테이너, 이미지, 볼륨 등 도커의 모든 구성 단위의 정보를 확인할 때 사용한다.
  * 정보를 확인할 종류를 명시하기 위해 `--type` 옵션에 image, volume등을 입력하는 것이 좋다.

```shell
$ docker inspect --type volume myvolume
[
    {
        "CreatedAt": "2021-07-23T07:55:23Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/myvolume/_data",
        "Name": "myvolume",
        "Options": {},
        "Scope": "local"
    }
]
# 특정 구성 단위를 제어하는 명령어 사용...
$ docker container inspect
$ docker volume inspect
```

* `Mountpoint` : 해당 볼륨이 실제로 호스트의 어디에 저장됐는 지를 의미한다.

* 해당 디렉터리의 파일 살펴보면 컨테이너에서 사용했던 파일이 남아있음을 알 수 있다.

  ```shell
  $ ls /var/lib/docker/volumes/myvolume/_data
  volume
  
  $ ls /var/lib/docker/volumes/myvolume/_data/volume
  /var/lib/docker/volumes/myvolume/_data/volume
  ```

* `docker volume create` 명령어를 사용하지 않아도 `-v` 옵션 입력할 때 이를 수행하도록 설정할 수 있다.

```shell
$ docker run -i -t --name volume_auto \
-v /root \
ubuntu:14.04

$ docker volume ls
local     6bff12dd76e7a1bf25629e959054ef5f4d6fc374704f8d818b3efcef8f3561af
local     c84c33b757bd0999555d1f1bd39df480032db5f9ed548f5609083f2e592d7ecd
local     myvolume
$ docker container inspect volume_auto
# Source 항목에 정의된 디렉터리인 /var/lib/ ... 이 volume_auto 컨테이너에 마운트돼 볼륨으로 쓰고 있다.
```

* 사용되지 않은 볼륨을 한꺼번에 삭제하려면 `docker volume prune` 명령어를 사용한다.



