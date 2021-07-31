# 접근권한 생성

* 도커 데몬에 `--insecure-registry` 옵션만 추가하면 별도의 인증 절차 없이 레지스트리 컨테이너에서 이미지를 push, pull 할 수 있다.
* 미리 정의된 계정으로 로그인하도록 설정함으로써 접근을 제한할 수 있다.

* TSL 적용 방법

  > Self-signed ROOT 인증서(CA) 파일 생성

```shell
$ mkdir certs
$ openssl genrsa -out ./certs/ca.key 2048
$ openssl req -x509 -new -key ./cert/ca.key -days 10000 -out ./certs/ca.crt
$ openssl genrsa -out ./cert/domain.key 2048

```

 

