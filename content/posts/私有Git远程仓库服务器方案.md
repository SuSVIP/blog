---
title: 私有Git远程仓库服务器方案
date: 2026-10-08T20:37:12+08:00
draft: false
categories:
  - 技术分享
tags:
  - Linux
  - Git
---

# 私有 Git 远程仓库服务器方案（适配 2 核 2G 虚拟机，RockyLinux / Ubuntu）

> 目标：搭建一套**模拟企业内部私有代码仓库**，支持网页管理、PR/MR、分支保护、多账号，用来练习开发 Git 流程；排除重资源 GitLab。
>
>  硬件：2 核 2G 虚拟机，推荐方案：**Gitea（首选）**；备选：Gogs；极简裸仓库（无 Web）。

# 一、前置条件

## 1.1 硬件资源检查（2 核 2G）

虚拟机内执行下面命令查看：

```bash
# 查看CPU核心
nproc
# 查看内存
free -h
```

✅ 合格标准：

- CPU：≥2 核
- 内存：≥2G

> Gitea+SQLite 空载只占用 100~300MB，2 核 2G 完全够用；只要不同时跑大量其他服务就没问题。

## 1.2 网络检查

执行命令：

```bash
ip a
```

✅ 合格标准：

1. 存在网卡 `ens33`（状态 `UP`），并且有 `inet 192.168.1.x` 的 IP 地址
2. VMware 虚拟机设置 → 网络适配器：
   - ✅ 勾选【已连接】、【启动时连接】
   - ✅ 网络模式：**桥接模式**（推荐，宿主机直接访问虚拟机 IP:3000）

> 合格后：宿主机浏览器访问 `http://虚拟机IP:3000` 打开 Gitea 网页。

### 网络合格自测命令（虚拟机内执行）

```bash
# 测试访问外网（安装docker需要联网）
ping 8.8.8.8
# 测试DNS
ping baidu.com
```

✅ 能 ping 通代表外网正常，可以拉取 docker 镜像。

## 1.3 系统环境检查（Docker 环境）

### 1.3.1. 检查是否安装 docker

```bash
docker -v
docker-compose -v
```

✅合格：输出版本号；如果提示 command not found，需要安装 docker。

### 1.3.2. 检查 docker 是否开机自启并正在运行

```bash
systemctl status docker
```

✅合格：`active (running)`

## 1.4 防火墙检查（放行 3000、2222 端口）

RockyLinux 默认 firewalld，查看规则：

```bash
firewall-cmd --list-ports
```

✅合格：输出包含 `3000/tcp`、`2222/tcp`

> 如果没有，执行放行命令：

```bash
firewall-cmd --add-port=3000/tcp --permanent
firewall-cmd --add-port=2222/tcp --permanent
firewall-cmd --reload
```



# 二、方案 A：Gitea Docker 部署（推荐，私有仓库，带 Web+PR，完整模拟 GitHub）

环境：虚拟机 RockyLinux9 / Ubuntu22.04，2 核 2G

> 数据库选择 SQLite（**不需要独立 MySQL/Postgres，省内存！练习首选**）

### 1. 前置：虚拟机安装 Docker + Docker Compose

#### Ubuntu

```bash
# 更新包
apt update && apt upgrade -y
# 安装docker
apt install docker.io -y
# 开机自启
systemctl enable --now docker
# 安装docker-compose
apt install docker-compose -y
```

#### RockyLinux9

```bash
dnf install docker -y
systemctl enable --now docker
# 安装docker-compose
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### 2. docker-compose.yml 配置文件（新建存放目录）

```
mkdir -p /opt/gitea
cd /opt/gitea
nano docker-compose.yml
```

写入下面内容：

```bash
version: "3"
services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    restart: always
    ports:
      - "3000:3000"  # web页面端口 http://ip:3000
      - "2222:22"    # SSH克隆端口
    volumes:
      - ./data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    environment:
      - TZ=Asia/Shanghai
```

### 3. 启动服务

```bash
docker-compose up -d
```

### 4. 初始化 Gitea

1. 浏览器打开：`http://虚拟机IP:3000`
2. 数据库类型：**SQLite3**（直接选，无需额外数据库）
3. 应用基础设置：
   - 仓库根路径：`/data/gitea-repositories`
   - 网站 URL：`http://虚拟机IP:3000`
4. 管理员账号：自定义用户名密码（记住，超级管理员）
5. 点击【安装 Gitea】等待完成。

#### 修改项

- SSH 服务端口：把 `22` → 改成 `2222`
- 其余全部保持默认不动：
  - 数据库类型：SQLite3 ✔
  - 数据库文件路径：`/data/gitea/gitea.db` ✔
  - 站点名称：可以保留，也可以改成你自己的名字（比如个人代码仓库）
  - Data Path：`/data/gitea` ✔
  - 以用户名运行：git ✔
  - HTTP Server Port：3000 ✔
  - Gitea Website URL：`http://192.168.1.131:3000/` ✔（192.168.1.131应为实际IP）

#### 页面最下方：管理员账号（重点！）

往下滚动页面，找到**管理员账户设置**

- 设置管理员用户名（例如 admin）
- 设置管理员邮箱
- 设置管理员密码（**务必记下来，这是登录后台的账号**）

> ⚠️ SQLite 适合个人自用，不用额外装数据库，非常省事。

全部填完，拉到最底部，点击【安装 Gitea】，等待几秒自动跳转登录页。

#### 安装完成后测试

1. 网页登录管理员账号
2. 创建仓库，测试 http 克隆：`http://192.168.1.131:3000/xxx/xxx.git`
3. SSH 克隆地址会自动变成 `git@192.168.1.131:2222:xxx/xxx.git`（因为我们改了 ssh 端口 2222）

### 5. 基础使用（模拟私有仓库）

1. 登录管理员账号 → 新建仓库（私有仓库）
2. 创建多个测试账号（模拟同事，练习代码评审 PR）
3. 开启分支保护：设置`main`分支禁止直接 push，**只能 PR 合并**（企业常用规范）
4. 克隆地址两种：

```
# http方式
git clone http://192.168.x.x:3000/admin/demo.git
# ssh方式
git clone ssh://git@192.168.x.x:2222/admin/demo.git
```

> 后续就可以完整练习：feature 分支、commit、push、提 PR、代码评审、合并、冲突。

### 防火墙放行端口

RockyLinux

```
# 放行3000网页端口，2222ssh端口
firewall-cmd --add-port=3000/tcp --permanent
firewall-cmd --add-port=2222/tcp --permanent
firewall-cmd --reload
```

Ubuntu ufw

```
ufw allow 3000/tcp
ufw allow 2222/tcp
```

------

# 三、方案 B：Gogs（极简轻量备选，2 核 2G 低配）

和 Gitea 用法几乎一致，UI 类似，占用内存更低，只是不再大更新。 同样 docker 一键部署，适合只想简单跑仓库。

```
# docker-compose.yml
version: "3"
services:
  gogs:
    image: gogs/gogs
    container_name: gogs
    restart: always
    ports:
      - "3000:3000"
      - "2222:22"
    volumes:
      - ./data:/data
    environment:
      - TZ=Asia/Shanghai
```

------

# 四、方案 C：纯 Git 裸仓库【零服务，无 Web 界面，极简】

> 不装任何 web 程序，仅用 git 自带裸仓库作为远程私有仓库，**没有 PR，只能练习 git 基础命令**。 虚拟机操作：

```
# 创建存放所有私有仓库目录
mkdir -p /home/git/repos
# 创建demo裸仓库，xxx.git 约定命名
mkdir /home/git/repos/demo.git
cd /home/git/repos/demo.git
git init --bare
# 权限设置
chown -R git:git /home/git/repos
```

宿主机克隆：

```
git clone git@192.168.x.x:/home/git/repos/demo.git
```

# 网络模式选择（虚拟机）

1. **桥接模式【推荐】**：虚拟机独立局域网 IP，宿主机、同局域网其他电脑都可以访问仓库
2. NAT 模式：仅宿主机能访问，需要 VMware 端口转发 3000、2222 端口，外部设备无法访问

# 2 核 2G 虚拟机优化 & 避坑

1. 使用 SQLite，**不要上 MySQL/Postgres**，额外消耗内存，练习场景完全没必要
2. 虚拟机不要同时跑其他服务（Redis、Java 应用等），防止内存吃满
3. 不要部署 GitLab，2G 内存必 OOM 崩溃
4. Gitea 可以在后台限制并发，减少内存占用；练习场景默认配置足够
5. 虚拟机做快照！配置好 Gitea 后快照，系统崩了直接回滚，不用重装
