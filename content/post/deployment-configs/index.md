---
title: '服务器服务管理'
slug: 'deployment-configs'
description: 介绍服务器软件如何管理
date: 2025-02-04T13:00:32+08:00
categories:
  - Ops
tags:
  - Serve
  - Methodology
---

介绍一下服务器中服务管理的方法。

<!--more-->

## 服务器服务管理

### 管理技术选型

在服务器服务管理中，选择合适的管理技术至关重要。当前，虚拟化技术因其高效性和灵活性，成为管理容器的首选方案。基于此，Docker 被选为运行各项服务的基础平台。Docker 不仅提供了轻量级的虚拟化环境，还能确保应用在不同环境中的一致性，极大地简化了部署和迁移过程。

然而，直接使用 `docker run` 命令来运行各个服务存在一定的局限性，尤其是在服务数量增多时，难以进行有效的归类整理和统一管理。为了解决这一问题，`docker-compose` 被选为服务管理工具。`docker-compose` 允许通过一个简单的 YAML 文件定义和管理多个容器，极大地提高了服务的可维护性和可扩展性。

关于为什么不使用 Kubernetes (k8s) 或 k3s 等更复杂的容器编排系统，主要原因在于它们的体量较大，适合大规模、高可用性的生产环境。在企业生产实践中，为了保证服务的高可用性，通常会采用多实例部署，并依赖滚动更新、自动重启等功能。然而，对于当前场景，服务的体量较小，且对高可用性的要求相对较低。更倾向于在服务出现问题时，通过及时通知和手动干预来恢复服务，而不是依赖复杂的自动化机制。因此，选择 `docker-compose` 不仅简化了管理流程，还降低了系统的复杂性和维护成本。

综上所述，基于当前的需求和技术特点，使用 Docker 结合 `docker-compose` 是一种既高效又实用的服务器服务管理方案。

### 文件夹结构

```txt
- deployment-configs
|-- scripts           # 用于管理服务的实用程序脚本
|  |-- start-all.sh     # 启动所有服务
|  |-- stop-all.sh      # 关闭所有服务
|-- services          # 包含各个服务的配置
|  |-- bark               # bark 通知服务
|  |  |-- data               # bark 数据存醋
|  |  |-- docker-compose.yml # bark 启动配置
|  |-- mysql              # mysql 数据库
|  |  |-- data               # mysql 数据存醋
|  |  |-- docker-compose.yml # mysql 启动配置
|  |  |-- logs               # mysql 日志文件位置
|  |  |-- my.cnf             # mysql 相关配置
|-- shared            # 包括环境变量和网络设置等共享配置
|  |-- common.env       # 通用环境变量
|-- README.md         # 介绍各个服务和相关命令
```

> 提供一下脚本，可直接执行，创建响应的目录结构

```shell
# 创建根目录
mkdir -p deployment-configs

# 进入根目录
cd deployment-configs

# 创建 scripts 目录及其文件
mkdir -p scripts

# 创建 start-all.sh 文件
touch scripts/start-all.sh
# 写入以下内容到 start-all.sh 文件
cat << 'EOF' > scripts/start-all.sh
#!/usr/bin/env bash

# Start all services defined in the 'services' directory

echo "Starting all services..."

# Loop through each service directory and bring up the service
for service in services/*; do
  if [ -d "$service" ]; then
    echo "Starting service: $service"
    cd "$service" || continue
    docker-compose up -d
    cd - || continue
  fi
done

echo "All services started."
EOF

# 创建 stop-all.sh 文件
touch scripts/stop-all.sh
# 写入 stop-all.sh 内容
cat << 'EOF' > scripts/stop-all.sh
#!/usr/bin/env bash

# Stop all services defined in the 'services' directory

echo "Stopping all services..."

# Loop through each service directory and bring down the service
for service in services/*; do
  if [ -d "$service" ]; then
    echo "Stopping service: $service"
    cd "$service" || continue
    docker-compose down
    cd - || continue
  fi
done

echo "All services stopped."
EOF


# 创建 services 目录
mkdir -p services/

# 创建 shared 目录及其文件
mkdir -p shared
touch shared/common.env

# 创建 README.md 文件
touch README.md
cat << 'EOF' > README.md
# Deployment Configs

## Overview

This repository contains configuration files and setups for deploying services using Docker Compose. It is organized into modular directories for individual services and shared resources.

## Structure

- `services/`: Contains configurations for individual services.
- `shared/`: Includes shared configurations such as environment variables and network settings.
- `scripts/`: Utility scripts for managing services.

EOF


# 设置文件权限（可选）
chmod +x scripts/start-all.sh
chmod +x scripts/stop-all.sh
```

### 如何保留空文件夹

在使用 Git 进行服务管理和同步时，通常不希望将 `data`、`logs` 等文件夹中的文件同步到远程仓库，因为这些文件可能包含临时数据或敏感信息。为了避免这些文件被同步，可以通过配置 `.gitignore` 文件来忽略它们，例如：

```gitignore
**/data/*
**/logs/*
**/cert/*
**/nginx/files/**
```

然而，配置忽略规则后，在推送代码到远程仓库或从远程仓库拉取代码时，这些文件夹将不会被创建。这可能导致在运行 docker-compose up -d 启动服务时出现问题，因为某些服务可能依赖这些文件夹的存在。

为了解决这一问题，既需要避免将文件夹中的文件同步到远程仓库，又需要确保这些空文件夹能够被传输到远程并在拉取时保留，可以在每个需要保留的文件夹中添加一个 `.gitignore` 文件，并在其中配置规则，使其不忽略自身。具体方法如下：

```shell
echo -e '*\n!.gitignore' >> .gitignore
```

上述命令会在 `.gitignore` 文件中添加两行内容：第一行 `*` 表示忽略该文件夹中的所有文件，第二行 `!.gitignore` 表示不忽略 `.gitignore` 文件本身。这样，Git 会保留空文件夹及其中的 `.gitignore` 文件，同时忽略其他文件，从而满足服务启动时的依赖需求。

通过这种方式，既避免了不必要文件的同步，又确保了文件夹结构的完整性。
