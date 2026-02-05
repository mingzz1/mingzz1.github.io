---
title: "[Docker] Docker 명령어"
date: 2020-01-19
draft: false
category: [development]
subcategories: [docker]
tags: [Docker]
---

예전에는 문제 출제때문에라도 `Docker`를 자주 썼었는데, 이제는 사용 할 일이 별로 없다.
그러다 보니 명령어를 자꾸 까먹어서 정리한다.  

<!--more-->

## 1. Build  
```sh
$ docker build -t [Name] .
```

## 2. Run  
```sh
$ docker run -d -it -p [hostPort]:[ContainerPort] --name [Name] [Name]
```

옵션 설명은 다음과 같다.  

* -d : Detached mode. run the container in the background
* -i : Interactive mode
* -t : Allocate a pseudo-TTY
  * -it 혹은 -i -t 등으로 사용 할 경우 Docker Container 내에서 명령어 실행 가능
* -p : Port. ex) -p 80:80, -p 80:3123
* --name : Container name. 아래의 3가지 방법으로 사용할 수 있으며, docker images 명령어를 통해 Container Name을 확인할 수 있음  

| Identifier type | Example value |  
| --- | --- |  
| UUID long identifier | "f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778" |  
| UUID short identifier | "f78375b1c487" |  
| Name | "Container Name" |  

## 3. Manange  

* Execute Command  
```sh
$ docker exec -i -t [ContainerName] [Command]
```

ex)
```sh
$ docker exec -i -t [ContainerName] /bin/bash
```

* Stop Running 
```sh
$ docker stop [ContainerName]
```

* Remove Container  
```sh
$ docker rm [ContainerName]
```

* Restart Container
```sh
$ docker restart [ContainerName]
```

* Check Docker Images  
```sh
$ docker images
```

* Check Docker Process  
```sh
$ docker ps -a
```

* Remove Cache  
```sh
$ docker system prune -a
```
