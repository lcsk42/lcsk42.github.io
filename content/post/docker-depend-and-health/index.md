---
title: 'Docker Compose 服务依赖和健康检查'
slug: docker-depend-and-health
description: 介绍 Docker Compose 服务依赖和健康检查相关知识
date: 2024-06-01T10:06:34+08:00
categories:
  - Ops
tags:
  - Docker
---

介绍 Docker Compose 服务依赖和健康检查相关知识。

<!--more-->

## 概念介绍

### 什么是服务依赖？

服务启动有先后顺序，B 依赖 A，如部署 Service 服务一般需要依赖 Database, 这时我们需要先启动数据库，再启动我们的应用程序。

### 如何使用服务依赖？

在 docker-compose.yml 中指定 `depends_on` 参数可以指定容器的启动顺序，B 服务要晚于 A 服务启动，则需要在 B 服务配置 `depends_on` A 服务。

### 什么是健康检查？

容器启动后，判断其是否可以正常对外提供服务，不能服务启动了是 `up` 状态，但是里面的服务一直在报错，这样事实上是启动有问题的，这个就是健康检查。

### 如何进行健康检查？

容器本身提供的有健康检查，可以在服务启动时，或者 Dockerfile 中指定相关参数, 不过使用更多的还是直接在 docker-compose.yml 中指定。

服务启动时：

```shell
--health-cmd string              Command to run to check health
--health-interval duration       Time between running the check
                                (ms|s|m|h) (default 0s)
--health-retries int             Consecutive failures needed to
                                report unhealthy
--health-start-period duration   Start period for the container to
                                initialize before starting
                                health-retries countdown
                                (ms|s|m|h) (default 0s)
--health-timeout duration        Maximum time to allow one check to
```

Dockerfile 指定：

```shell
HEALTHCHECK [OPTIONS] CMD command

The options that can appear before CMD are:

--interval=DURATION (default: 30s)
--timeout=DURATION (default: 30s)
--start-period=DURATION (default: 0s)
--start-interval=DURATION (default: 5s)
--retries=N (default: 3)
```

docker-compose 方式指定:

```shell
healthcheck:
      test: [ "CMD", "curl", "http://127.0.0.1:8080" ]
      # healthcheck 间隔时间
      interval: 10s
      # 执行命令超时时间
      timeout: 3s
      # 重试次数
      retries: 5
      # 容器启动多久后开始执行 healthcheck
      start_period: 10s
```

### 两者的结合？

我们实际使用中遇到的情况可能是，B 服务依赖 A 服务，但是 A 服务开始启动之后，实际需要耗时一段时间（初始化数据等操作），此时如果直接启动 B 服务，就会报错，所以我们需要在 A 服务启动成功，能对外提供服务时，才开始启动 B 服务。

```text
异常情况(只使用 depends_on)：B 服务 需要访问 A 服务时，A 服务还在启动中，B 服务报错
A  [-----created-----](up)[-----starting-----][-----running-----]
B                         [-created-][-starting-][-error-]

正确情况: A 服务启动完成后，开始对外提供服务，开始启动 B 服务
A  [-----created-----][-----starting-----](healthy)[-----running-----]
B                     [-created-]                  [-starting-][-error-]
```

### 如何解决？

在 A 服务中配置健康检查，在 B 服务的服务依赖中配置 A 服务的服务状态为健康。

```yml
services:
  serviceA:
    healthcheck:
      test: ['CMD', 'curl', 'http://127.0.0.1:8080']
      interval: 10s
      timeout: 3s
      retries: 5
      start_period: 10s
  serviceB:
    depends_on:
      serviceA:
        condition: service_healthy
```

## 具体示例

> **请注意**：此文件仅作为测试使用，无实际意义。

```shell
docker-compose down
cat > docker-compose.yml << EOF
version: '3.9'
services:
  mysql8:
    image: mysql:8
    environment:
      - MYSQL_ROOT_PASSWORD=lucas
    healthcheck:
      test:
        [
          "CMD",
          "mysql",
          "-u",
          "root",
          "-plucas",
          "-e",
          "select 1"
        ]
      interval: 10s
      timeout: 3s
      retries: 5
      start_period: 10s
  tomcat9:
    image: tomcat:9
    depends_on:
      mysql8:
        condition: service_healthy
    healthcheck:
      test: [ "CMD", "curl", "http://127.0.0.1:8080" ]
      interval: 10s
      timeout: 3s
      retries: 5
      start_period: 10s
EOF

docker-compose up -d

```

### 正常启动日志

```shell
[+] Running 0/3
 ⠇ Network docker_default      Created                                                                                                                     1.9s
 ⠇ Container docker-mysql8-1   Waiting                                                                                                                     1.9s
 ⠇ Container docker-tomcat9-1  Created

[+] Running 2/3
 ⠹ Network docker_default      Created                                                                                                                    11.2s
 ✔ Container docker-mysql8-1   Healthy                                                                                                                    10.9s
 ✔ Container docker-tomcat9-1  Started                                                                                                                    11.1s

❯ docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS                            PORTS                 NAMES
33036c106e99   tomcat:9   "catalina.sh run"        16 seconds ago   Up 5 seconds (health: starting)   8080/tcp              docker-tomcat9-1
5f778d7b5c8d   mysql:8    "docker-entrypoint.s…"   16 seconds ago   Up 15 seconds (healthy)           3306/tcp, 33060/tcp   docker-mysql8-1

❯ docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS                    PORTS                 NAMES
33036c106e99   tomcat:9   "catalina.sh run"        22 seconds ago   Up 11 seconds (healthy)   8080/tcp              docker-tomcat9-1
5f778d7b5c8d   mysql:8    "docker-entrypoint.s…"   22 seconds ago   Up 22 seconds (healthy)   3306/tcp, 33060/tcp   docker-mysql8-1
```

### 启动过程分析

1. 刚开始启动时，docker-mysql8-1 在 `Waiting` 中。等待启动后的服务检查，docker-tomcat9-1 处于 `Created` (容器创建了，再等待启动)
2. docker-mysql8-1 状态为 `Healthy`(启动成功，健康检查正常)，此时 docker-tomcat9-1 开始启动
3. 刚开始查看服务状态时，docker-tomcat9-1 服务的 `STATUS` 为 (`health: starting`)
4. 过了一段时间后，docker-tomcat9-1 服务的 `STATUS` 变为 (`health`)

### 健康检查失败日志

我们修改 mysql 的健康检查语句，使其执行失败，查看如果健康检查失败会是什么结果

```diff
[dependencies.bevy]
docker-compose.yml
-          "-plucas",
+          "-perrorpassword",
```

```shell
[+] Running 1/3
 ⠧ Network docker_default      Created                                                                                                                    50.8s
 ✘ Container docker-mysql8-1   Error                                                                                                                      50.7s
 ⠧ Container docker-tomcat9-1  Created                                                                                                                    50.7s
dependency failed to start: container docker-mysql8-1 is unhealthy
❯ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                      PORTS                 NAMES
868b161494b0   mysql:8   "docker-entrypoint.s…"   57 seconds ago   Up 57 seconds (unhealthy)   3306/tcp, 33060/tcp   docker-mysql8-1
❯ docker ps -a
CONTAINER ID   IMAGE      COMMAND                  CREATED              STATUS                          PORTS                 NAMES
52c90d95d7e9   tomcat:9   "catalina.sh run"        About a minute ago   Created                                               docker-tomcat9-1
868b161494b0   mysql:8    "docker-entrypoint.s…"   About a minute ago   Up About a minute (unhealthy)   3306/tcp, 33060/tcp   docker-mysql8-1
```

### 失败日志分析

1. 首先启动时，docker-tomcat9-1 一直在等待，docker-mysql8-1 最终创建失败，耗时 50s (`interval 10s` \* `retries 5`) 左右
2. 使用 docker ps -a 可以查看所有的容器状态，可以看到 docker-mysql8-1 为 `unhealthy`, docker-tomcat9-1 为 `Created` 并未启动

### 如何查看分析错误

使用 docker 的 `inspect` 命令, 在 `State.Health.Log` 中可以看到相关错误信息。

```shell
❯ docker inspect docker-mysql8-1
[
    {
        ...
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 35046,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2024-05-31T18:13:03.413731054Z",
            "FinishedAt": "0001-01-01T00:00:00Z",
            "Health": {
                "Status": "unhealthy",
                "FailingStreak": 39,
                "Log": [
                    {
                        "Start": "2024-06-01T02:18:54.821003238+08:00",
                        "End": "2024-06-01T02:18:54.860412275+08:00",
                        "ExitCode": 1,
                        "Output": "mysql: [Warning] Using a password on the command line interface can be insecure.\nERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)\n"
                    },
                    {
                        "Start": "2024-06-01T02:19:04.862050738+08:00",
                        "End": "2024-06-01T02:19:04.905490712+08:00",
                        "ExitCode": 1,
                        "Output": "mysql: [Warning] Using a password on the command line interface can be insecure.\nERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)\n"
                    },
                    {
                        "Start": "2024-06-01T02:19:14.907068839+08:00",
                        "End": "2024-06-01T02:19:14.941198241+08:00",
                        "ExitCode": 1,
                        "Output": "mysql: [Warning] Using a password on the command line interface can be insecure.\nERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)\n"
                    },
                    {
                        "Start": "2024-06-01T02:19:24.94265745+08:00",
                        "End": "2024-06-01T02:19:24.980647633+08:00",
                        "ExitCode": 1,
                        "Output": "mysql: [Warning] Using a password on the command line interface can be insecure.\nERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)\n"
                    },
                    {
                        "Start": "2024-06-01T02:19:34.983078462+08:00",
                        "End": "2024-06-01T02:19:35.023463408+08:00",
                        "ExitCode": 1,
                        "Output": "mysql: [Warning] Using a password on the command line interface can be insecure.\nERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)\n"
                    }
                ]
            }
        },
        ...
    }
]
```
