# docker-compose

* 설치 확인
  * Mac은 기본적으로 설치 되어있음

```shell
yunsoyun-ui-MacBookPro:~ root# docker-compose version
docker-compose version 1.29.2, build 5becea4c
docker-py version: 5.0.0
CPython version: 3.9.0
OpenSSL version: OpenSSL 1.1.1h  22 Sep 2020
```



* Linux - `docker-compose` 설치

```markup
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)"
sudo chmod +x /usr/local/bin/docker-compose
```

* Docker-compose.yml

```shell
₩version: '2'
services:
  db:
    image: mariadb:10.5
    volumes:
      - ./mysql:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
  wordpress:
    image: wordpress:latest
    volumes:
      - ./wp:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
```

* `version` : yml파일의 명세 버전
* `services` : 실행할 컨테이너 정의 = `docker run --name ~~`
* `image` : 컨테이너에 사용할 이미지 이름과 태그
  * 태그 생략하면 `latest`
  * 이미지가 없으면 자동으로 Pull
* `ports` : 컨테이너와 연결할 포트
  * [호스트 포트] : [컨테이너 포트]
* `environment`: 컨테이너에서 사용할 환경변수
  * [환경변수 이름] : [값]
* `volumes` : 마운트하려는 디렉터리
  * [호스트 디렉터리] : [컨테이너 디렉터리]
* `restart` : 재시작 정책
  * "no" / always / on-failure / unless-stopped
* `build` : 이미지를 자체 빌드 후 사용
  * image 속성 대신 사용함

**쉘 스크립트**

- 실행: docker-compose up
  - docker-compose up -d
  - docker-compose up --force-recreate : 컨테이너 새로 만들기
  - docker-compose up --build : 도커 이미지를 다시 빌드(build로 선언했을 때만)
- 재개 : docker-compose start
  - docker-compose start wordpress
- 재시작 : docker-compose restart
- 컨테이너 멈춤 : docker-compose stop
- 종료 및 삭제: docker-compose down
- 컨테이너의 로그 : docker-compose logs
  - Docker-compose logs -f