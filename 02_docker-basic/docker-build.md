## 1. 이미지 생성하는 법

1. 아무것도 존재하지 않는 이미지 컨테이너를 생성
2. 애플리케이션을 위한 환경을 설치하고 소스코드 등을 복사해 잘 동작하는 것 확인
3. 컨테이너를 이미지로 commit

>  이러한 작업을 기록한 파일의 이름을 **dockerfile**이라고 하며, 빌드 명령어는 Dockerfile을 읽어 이미지를 생성한다.
>
> Git을 통해 자동화가 가능하다

## 2. Dockerfile 작성

> Dockerfile에는 컨테이너에서 수행해야 할 작업을 명시한다.

```shell
$ mkdir dockerfile && cd dockerfile
$ echo test >> test.html

$ ls
test.html
```

```shell
$ vi Dockerfile
FROM fluent/fluentd:v1.12 # 이미지의 베이스
LABEL maintainer=alice_k106@naver.com # 이미지에 메타데이터 추가
USER root

RUN apk update && \
  apk add musl-dev gcc make ruby-dev && \
  fluent-gem install fluent-plugin-mongo

EXPOSE 24284 # 이미지에서 노출할 포트
USER fluent
CMD exec fluentd -c /fluentd/etc/$FLUENTD_CONF -p /fluentd/plugins $FLUENTD_OPT
# CMD : 컨테이너가 시작될 때마다 실행할 명령어를 설정한다
```

## 3. 이미지 생성

* Docker build
  * `-t` : 생성될 이미지의 이름 설정
  * 명령어 끝에는 dockerfile이 저장된 경로 입력 (`./` : 현재 디렉토리)

```shell
$ docker build -t mybuild:0.0 ./
```

```shell
# 호스트의 포트 확인
$ docker port 
$ docker ps

# ex)
$ docker port myserver
```

* 도커 캐시

  * 한 번 이미지 빌드를 마치고 난 뒤 다시  같은 빌드를 진행하면 이전의 이미지 빌드에서 사용했던 캐시를 사용한다.

  ```shell
  $ vi Dockerfile2
  FROM ubuntu:14.04
  MAINTAINER alicek106
  LABEL "purpose"="practice"
  RUN apt-get update
  ```

  * `-f` or `--file`로 docker file 이름 지정 가능

  ```shell
  $ docker build -f Dockerfile2 -t mycache:0.0 ./
  ```

  * 캐시가 필요하지 않을 때는 `--no-cache` 옵션을 추가한다.

  ```shell
  $ docker build --no-cache -t mybuild:0.0 .
  ```

  

# Dockerfile 기타 명령어

* `ENV` : 환경변수 지정
* `VOLUME` : 빌드된 이미지로 컨테이너 생성했을 때 호스트와 공유할 컨테이너 내부의 디렉터리 설정
* ARG` : build 명령어를 실행할 때 추가로 입력받아 Dockerfile 내에서 사용될 변수의 값을 설정한다.
