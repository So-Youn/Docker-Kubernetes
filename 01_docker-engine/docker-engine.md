# 권한 부여

```shell
ubuntu@ip-172-31-5-197:~$ docker ps -a
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json?all=1: dial unix /var/run/docker.sock: connect: permission denied
```

👉🏻 사용자가 `/var/run/docker.sock` 접근 시도했으나 권한이 없어 발생하는 문제

 :point_right: 사용자가 `root:docker` 권한을 가지고 있어야 한다.

```shell
ubuntu@ip-172-31-5-197:~$ ls -al /var/run/docker.sock 
srw-rw---- 1 root docker 0 Jul 17 15:54 /var/run/docker.sock
# root 권한 실행은 권장 X -> 사용자를 docker group에 포함시켜주기
ubuntu@ip-172-31-5-197:~$ sudo usermod -aG docker $USER
ubuntu@ip-172-31-5-197:~$ newgrp docker
```

