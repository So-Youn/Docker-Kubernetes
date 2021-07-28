## 자원 할당

* 컨테이너에 설정된 자원 제한을 확인하는 방법 `docker inspect` 

  ```shell
  $ docker inspect rsyslog
  ```

* 컨테이너의 자원 제한 변경하려면 `update` 명령어 사용

  ```shell
  $ docker update (변경할 자원 제한) (컨테이너 이름)
  $ docker update --cpuset-cpus=1 centos ubuntu
  ```



## 메모리 제한

> `docker run` 명령어에 `--memory` 지정해 컨테이너의 메모리를 제한할 수 있다.
>
> * m(megabyte)
> * g(gigabyte)
> * 제한할 수 있는 최소 메모리는 4MB

```shell
# 컨테이너 메모리 사용량 1GB로 제한
$ docker run -d \
--memory="1g" \ 
--name memory_1g \
nginx

# 메모리 값 확인
$ docker inspect memory_1g | grep \"Memory\"
```

* 컨테이너 내에서 동작하는 프로세스가 컨테이너에 할당된 메모리를 초과하면 컨테이너는 자동으로 종료되므로 애플리케이션에 따라 메모리를 적절하게 할당하는 것이 좋다.
* 기본적으로 컨테이너의 swap 메모리는 메모리의 2배로 설정된다.

```shell
$ docker run -it --name swap_500m \
--memory=200m \
--memory-swap=500m \
ubuntu:14.04
```

## CPU 제한

* `--cpu-shares` 옵션은 컨테이너에 가중치를 설정해 해당 컨테이너가 CPU를 상대적으로 얼마나 사용할 수 있는지를 나타낸다.
  * 시스템에 존재하는 CPU를 어느 비중만큼 나눠 쓸 것인지 명시하는 옵션
  * 아무런 설정을 하지 않았을 때 컨테이너가 가지는 값은 `1024`, 이는 CPU 할당에서 1의 비중을 뜻한다.

```shell
$ docker run -it --name cpu_share \
--cpu-shares 1024 \
ubuntu:14.04

$ docker run -d --name cpu_1024 \
--cpu-shares 1024 \
alicek106/stress \
stress --cpu 1 # 1개의 프로세스로 cpu에 부하를 주는 명령어
```

```shell
# cpu 사용률 확인
$ ps aux | grep stress

# --cpu-shares 값이 512로 설정된다면
$ docker run -d --name cpu_512 \
--cpu-shares 512 \
alicek106/stress \
stress --cpu 1 

# 시간 흐른 뒤 다시 cpu 사용률 확인
$ ps aux | grep stress

# 1024:512=2:1의 비율대로 cpu 사용함 (66%, 33%)
```

* `stress` 명령어는 CPU와 메모리에 과부하를 줘서 성능을 테스트한다.

  * `stress`를 `-d` 옵션으로 생성하면 백그라운드에서 계속 cpu에 부하를 줄 수 있으므로 테스트가 끝나면 바로 컨테이너를 삭제해야 한다.

  ```shell
  $ apt-get install stress
  ```

* `--cpuset-cpu` : 호스트에 cpu가 여러 개 있을 때 `--cpuset-cpus` 옵션을 지정해 컨테이너가 특정 cpu만 사용하도록 설정

  * cpu 집중적인 작업이 필요하다면 여러 개의 cpu를 사용하도록 설정해 작업 적절히 분배 

