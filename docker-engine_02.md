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

