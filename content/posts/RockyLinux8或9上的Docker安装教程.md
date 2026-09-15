---
title: RockyLinux8或9上的Docker安装教程
date: 2026-07-16T04:22:15+08:00
draft: false
categories:
  - 技术分享
tags:
  - Rocky Linux
  - Docker
---

# Rocky Linux 8/9 上的 Docker 安装教程

下面给你 **Rocky Linux 8 / 9 通用、最标准、一步不踩坑** 的 Docker 安装教程，包含 Docker Compose，直接复制命令即可。

---

# 1. 卸载旧版本（如果有）

```Bash

sudo dnf remove -y docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

# 2. 安装依赖

```Bash

sudo dnf install -y yum-utils device-mapper-persistent-data lvm2
```

# 3. 添加 Docker 官方源

```Bash

sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

# 4. 安装 Docker 引擎

```Bash

sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

# 5. 启动并设置开机自启

```Bash

sudo systemctl enable --now docker
```

# 6. 验证安装成功

```Bash

docker --version
docker compose version
```

出现版本号就 OK。

# 7.（可选）让普通用户不用 sudo 执行 docker

```Bash
# 将当前用户加入 docker 用户组
sudo usermod -aG docker $USER

# 刷新组权限（无需重启）
newgrp docker
```

没有执行`newgrp docker`需要**重新登录终端**生效。



# 8.（可选）配置国内 Docker 镜像源（解决拉取慢 / 失败的核心）

## 步骤 1：验证网络连通性（先排查基础）

```bash
# 1. 测试 Docker 官方仓库的域名解析
nslookup registry-1.docker.io
# 2. 测试 HTTPS 访问（关键）
curl -I https://registry-1.docker.io/v2/
```

+ 如果 `nslookup` 超时 / 返回异常 IP，说明 DNS 有问题

+ 如果 `curl` 超时 / 连接失败，说明网络 / 防火墙拦截了 HTTPS 请求



## 步骤 2：编辑 Docker 配置文件，添加阿里云 / 网易等国内镜像源：

```bash
# 1. 创建/编辑 daemon.json 配置文件
sudo vim /etc/docker/daemon.json
```

写入以下内容（如果文件已有内容，只添加 `registry-mirrors` 部分）：

```bash
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ],
  "dns": ["223.5.5.5", "223.6.6.6"]
}
```

保存后重启 Docker 生效：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 步骤 3：验证配置是否生效：

```bash
# 1. 查看 Docker 状态（确认重启后正常）
systemctl status docker
# 2. 测试镜像拉取
docker pull hello-world
# 4. 查看 容器状态
docker compose ps
```

如果出现下面内容，**说明配置成功、网络正常、Docker 完全没问题**：

```tex
Using default tag: latest
latest: Pulling from library/hello-world
...
Status: Downloaded newer image for hello-world:latest
```

再运行一下，确认完全正常

```
# 运行
docker run hello-world
```

出现 Hello from Docker! 就代表 **100% 正常**



# 9.（可选）开放端口与临时关闭防火墙

```bash
# 永久放行 Docker 相关端口
sudo firewall-cmd --add-port=9621/tcp --permanent
sudo firewall-cmd --add-port=2375/tcp --permanent
# 重启防火墙
sudo systemctl start firewalld
sudo firewall-cmd --reload
```

临时测试 - 关闭防火墙（最快验证）

```bash
# 临时关闭防火墙 (测试用)
sudo systemctl stop firewalld

# 测试 curl 是否连通
curl -I https://registry-1.docker.io/v2/

# 如果测试成功，再永久开放端口
sudo firewall-cmd --add-port=443/tcp --permanent
sudo firewall-cmd --reload
```

# 10.（可选）环境加固

**恢复防火墙**：测试完成后，建议重新启动 `firewalld`，避免系统暴露在无防护状态。

**验证镜像源**：执行 `docker info`，查看 `Registry Mirrors` 列表，确认国内镜像源已生效。

# 10.异常处理（若拉取中断）

如果拉取过程中网络中断，执行以下命令重试：

```bash
# 停止容器
docker compose down
# 重新拉取并启动
docker compose up -d
```

若拉取速度慢，可在 `daemon.json` 中补充更多国内镜像源（如阿里云个人加速器）。





---

# 常用 Docker 命令

```Bash

# 查看运行中的容器
docker ps

# 查看镜像
docker images

# 一键启动 docker-compose
docker compose up -d

# 查看日志
docker compose logs -f

# 停止
docker compose down

# 查看容器运行状态
docker compose ps
```

---

如果你是**低配虚拟机/服务器**，我可以再给你：

- Docker 低内存优化配置

- 限制 CPU/内存的配置文件

- 一套直接跑的轻量 RAG `docker-compose.yml`（支持硅基流动千问8B）

要吗？
