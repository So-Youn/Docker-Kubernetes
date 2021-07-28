# Docker Image

> `docker search` 명령어는 도커 허브에서 이미지를 검색하며, 도커 허브 이미지임을 명시하기 위해 `docker.io/ubuntu` 와 같이 `docker.io` 접두어를 사용할 수 있다.

```shell
# stars 는 해당 이미지가 도커 사용자로부터 얼마나 즐겨찾기(star)됐는지를 나타냄
$ docker search ubuntu

```

## 도커 이미지 생성

* docker search를 통해 검색한 이미지를 pull명령어로 내려받아 사용할 수도 있지만, 본인만의 이미지를 직접 생성해야 할 때도 있음

### For example

```shell
$ docker run -it --name commit_test ubuntu:14.04

$ echo test_first! >> first # 컨테이너 내부에 first라는 이름의 파일을 하나 생성해 기존의 이미지로부터 변경사항 만듦
```

* 호스트로 빠져나와 `docker commit` 명령어를 입력해 **컨테이너를 이미지로** 만든다.

```shell
$ docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

* commit_test 라는 컨테이너를 commit_test:first 라는 이름의 이미지로 생성
* `-a` 옵션은 author, 이미지의 작성자
* `-m` : 이미지에 포함될 부가 설명을 입력

```shell
$ docker commit \
-a "alicek106" -m "my first commit" \ 
commit_test \
commit_test:first

# 이미지 생성 확인
$ docker images
```

* `docker history` : 이미지의 레이어 구조 확인
* `docker rmi` : 이미지 삭제

```shell
$ docker history commit_test:first
$ docker rmi commit_test:first
# 이미지를 사용중인 컨테이너가 존재할 때, 에러 발생
# 이럴 경우 사용중인 컨테이너 중지 및 삭제 후 다시 실행
$ docker stop commit_test2 && docker rm commit_test2
$ docker rmi commit_test:first
```

* 만약 컨테이너가 사용중인 이미지를 `docker rmi -f` 로 강제 삭제하면 이미지의 이름이 **NONE** 으로 변경되며, 이러한 이미지들을 **dangling**(댕글링) 이미지라고 한다.
* dangling 이미지는 `$ docker images -f dangling=true` 명령어를 사용해 확인 가능
* 사용 중이지 않은 댕글링 이미지는 `docker image prune` 명령어로 한꺼번에 삭제 가능