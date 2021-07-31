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

$ openssl req -new -key ./certs/domain.key -subj /CN=${DOCKER_HOST_IP} -out ./certs/domain.csr
$ openssl x509 -req -in ./certs/domain.csr -CA ./certs/ca.crt -CAkey ./certs/ca.key -CAcreateserial -out ./certs/domain.crt -days 10000 -extfile extfile.cnf
```

*  레지스트리에 로그인할 때 사용할 계정과 비밀번호를 저장하는 파일을 생성한다.

```shell
# htpasswd 설치
# 우분투
$ apt-get install apache2-utils

# CentOS, 레드헷
$ yum install httpd-tools
```

```shell
$ htpasswd -c htpasswd soyun3963
New password:

$ mv htpasswd certs/
```



