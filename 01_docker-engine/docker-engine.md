# ê¶í ë¶ì¬

```shell
ubuntu@ip-172-31-5-197:~$ docker ps -a
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json?all=1: dial unix /var/run/docker.sock: connect: permission denied
```

ðð» ì¬ì©ìê° `/var/run/docker.sock` ì ê·¼ ìëíì¼ë ê¶íì´ ìì´ ë°ìíë ë¬¸ì 

 :point_right: ì¬ì©ìê° `root:docker` ê¶íì ê°ì§ê³  ìì´ì¼ íë¤.

```shell
ubuntu@ip-172-31-5-197:~$ ls -al /var/run/docker.sock 
srw-rw---- 1 root docker 0 Jul 17 15:54 /var/run/docker.sock
# root ê¶í ì¤íì ê¶ì¥ X -> ì¬ì©ìë¥¼ docker groupì í¬í¨ìì¼ì£¼ê¸°
ubuntu@ip-172-31-5-197:~$ sudo usermod -aG docker $USER
ubuntu@ip-172-31-5-197:~$ newgrp docker
```

