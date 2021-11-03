# Docker-Kubernet

> 도커 
>
> 가상 머신처럼 독립적으로 실행되지만 보다 빠르고, 쉽고 효율적이다
>
> 1. 리눅스 커널의 여러 기술 활용
> 2. 하드웨어 가상화 기술보다 가벼움
> 3. 이미지 단위로 프로세스 실행 환경 구성

* 도커 버전 : 클라이언트 / 서버

  * `docker cli`는 도커 호스트에 명령을 전달하고 결과를 받아서 출력함


```shell
soyun@yunsoyucBookPro ~ % docker -v
Docker version 20.10.7, build f0df350

soyun@yunsoyun-ui-MacBookPro ~ % docker version
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
Client:
 Cloud integration: 1.0.17
 Version:           20.10.7
 API version:       1.41
 Go version:        go1.16.4
 Git commit:        f0df350
 Built:             Wed Jun  2 11:56:23 2021
 OS/Arch:           darwin/arm64
 Context:           default
 Experimental:      true
Server: Docker Engine - Community
 Engine:
  Version:          20.10.7
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       b0f5bc3
  Built:            Wed Jun  2 11:55:36 2021
  OS/Arch:          linux/arm64
  Experimental:     false
 containerd:
  Version:          1.4.6
  GitCommit:        d71fcd7d8303cbf684402823e425e9dd2e99285d
 runc:
  Version:          1.0.0-rc95
  GitCommit:        b9ee9c6314599f1b4a7f497e1f1f856fe433d3b7
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

```

* 커널 버전

```shell
soyun@yunsoyucBookPro ~ % docker info | grep Kernel
 Kernel Version: 5.10.25-linuxkit
```

* VirtulBox [download](https://www.virtualbox.org/wiki/Downloads)
* 가상 머신에 사용될 이미지파일[(.iso)](https://releases.ubuntu.com/16.04/)

* Jenkins : java로 만들어진 CI/CD tool
