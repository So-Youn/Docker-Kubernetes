# 도커 데몬

> 도커 엔진은 외부에서 API 입력을 받아 도커 엔진의 기능을 수행하는데, 도커 프로세스가 실행되어 서버로서 입력을 받을 준비가 된 상태를 **도커 데몬**  이라고 한다.

## 1. 도커의 구조

```markdown
* 클라이언트로서의 도커
도커 데몬이 API를 사용할 수 있도록 CLI를 제공하는 것
 > docker로 시작하는 명령어
 
* 서버로서의 도커 
실제로 컨테이너를 생성하고 실행하며 이미지를 관리하는 주체,
이는 dockerd 프로세스로서 동작한다.
```



* 도커의 위치 확인

```shell
$ which docker
/usr/bin/docker
```

* 실행 중인 도커 프로세스 확인

```shell
$ ps aux | grep docker
```

## 2. 도커 데몬 실행

* 우분투는 도커가 자동으로 설정되므로 바로 사용 가능

```shell
$ service docker start
$ service docker stop
```

* 레드헷은 명령어를 통해 자동으로 실행되도록 설정하려면 서비스 활성화 필요

```shell
$ systemctl enable docker

# 도커 서비스 정지 후 dockerd 명령어 실행
$ service docker stop
$ dockerd
```

* 사용 가능한 옵션 확인

```shell
$ dockerd --help
```

* `COPY` 는 로컬 디렉터리에서 읽어 들인 컨텍스트로부터 이미지에 파일을 복사하는 역할
  *  COPY를 사용하는 형식은 ADD와 같습니다.