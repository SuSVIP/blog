---
title: Hugo博客部署
date: 2026-09-14T11:11:45+08:00
draft: false
categories:
  - 技术分享
tags:
  - 博客
  - Hugo
---

# 一、环境准备

## 1.1、安装Hugo Extended

> ⚠️注意：必须安装 **Hugo Extended（扩展版）**，普通版本无法处理 SCSS 样式，PaperMod 主题会样式错乱。本机版本：**hugo v0.165.0 extended**

### Windows

#### 方式一：Chocolatey包管理

```
choco install hugo-extended
```

#### 方式二：Github Realease下载

去 hugo release 下载`hugo_extended_0.165.0_Windows‑64bit.zip`，解压，把 hugo.exe 加入到系统环境变量**PATH**。

### macOS

```bash
# 使用Homebrew安装
brew install hugo
```

### Linux(Ubuntu/Debian)

```bash
# 下载deb包，以0.165.0版本举例
wget https://github.com/gohugoio/hugo/releases/download/v0.165.0/hugo_extended_0.165.0_linux‑amd64.deb
sudo dpkg -i hugo_extended_0.165.0_linux‑amd64.deb
```

### 检验是否安装成功，终端运行

```bash
hugo version
```

输出包含`extended`字样即为扩展版安装成功。

> 💡提示：本地 Hugo 版本必须和 GitHub Action 中`HUGO_VERSION`保持一致，避免本地正常、线上构建样式错乱。



## 1.2、必备工具

- Git，已经登陆 Github 账号，SSH 密钥推荐配置完成，避免 HTTPS 频繁输入 token
- 文本编辑器：VS Code (推荐)，安装 Markdown All in One、Prettier 插件
- 可选：Typora 用于写文章



## 1.3、创建Github远程仓库

1. 登录 GitHub 网页，点击右上角 `+` → New repository 新建仓库
2. 仓库名：`blog`（仓库名称，自定义）
3. 仓库权限选择 **Public**（GitHub Pages 免费需要公开仓库；私有仓库需要付费）
4. **不要勾选 Add a README file、.gitignore、license**，保持仓库完全空仓库，点击 Create repository 创建。

⚠️ 不要初始化 README，否则本地后续 push 会出现历史提交冲突。 

✅ Pages 设置中**Source 必须选择 GitHub Actions** ，不要选 Deploy from a branch。

> 📌两种仓库命名区分：
>
> 1. 用户主页博客：仓库名 = `用户名.github.io`，访问地址：`https://用户名.github.io`（无二级路径，baseURL 末尾不要加仓库名）
> 2. 普通项目博客：仓库名自定义，访问地址：`https://用户名.github.io/blog/`（二级子路径）





# 二、本地初始化Hugo博客项目

## 2.1、创建站点（2种方式）

**方式一：使用命令创建**

```bash
# 创建博客站点，blog为你的项目文件夹名称
hugo new site blog
cd blog
# 初始化git仓库
git init
```

**方式二：克隆远程仓库(推荐)**

```
git clon git@github.com:username/blog.git
```



## 2.2、安装PaperMod主题（最主流博客主题）

**两种安装方式，推荐Git Submodule方式，方便后续更新主题**

```bash
# 添加PaperMod主题作为git子模块，代码拉取到 themes/PaperMod 目录，方便后续升级主题
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# 👉国内网络慢用镜像代理
# git submodule add https://mirror.ghproxy.com/https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# 初始化并递归拉取所有子模块代码（首次克隆项目后必须执行，把主题完整下载下来）
git submodule update --init --recursive
```

> ❗不要直接复制主题文件到本地，子模块可以拉取主题新版本，方便升级。



备选主题清单（简单备注优缺点）

```bash
# PaperModX 魔改版，注意新版Hugo部分兼容问题
git submodule add https://github.com/reorx/hugo-PaperModX.git themes/PaperModX

# chaos-theme，归档体验好，部分页面卡片异常
git submodule add https://github.com/delphij/chaos-theme.git themes/chaos-theme

# Hugo‑Theme‑Simple，极简轻量，无复杂侧边栏
git submodule add https://github.com/simple-is-awesome/Hugo-Theme-Simple.git themes/Hugo-Theme-Simple

# Hugo‑Theme‑Stack，美观，强依赖Dart Sass
git submodule add https://github.com/CaiJimmy/hugo-theme-stack.git themes/hugo-theme-stack

# FixIt，国内热门，功能丰富，依赖Dart Sass
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt

# blog‑awesome，简洁无侧边
git submodule add https://github.com/hugo-sid/hugo-blog-awesome.git themes/blog-awesome
```

**更新主题命令**

```bash
# 更新单个主题
git submodule update --remote themes/PaperMod
# 更新全部子模块
git submodule update --remote
```

**项目目录结构说明**

```yaml
my-blog/
├── hugo.toml                 # 主配置
├── content/                  # 所有文章/页面
│   ├── posts/                # 博客文章
│   │   └── hello.md
│   └── about/
│       └── _index.md
├── archetypes/
│   └── default.md            # 新建文章模板
├── static/                   # 图片、附件、静态资源
├── assets/                   # css/js资源，一般不用改
├── themes/chaos/             # chaos主题源码
└── public/                   # hugo build生成的静态网站（部署用）

blog/
├── hugo.toml                 # 主配置文件（hugo0.110+默认toml，旧版config.toml）
├── content/                  # 所有文章/页面，写博客核心目录
│   ├── posts/                # 博客文章存放目录
│   │   └── hello.md
│   └── about.md
│   └── archives.md
│   └── search.md
│       
├── archetypes/
│   └── default.md            # 新建文章模板，自定义文章头部参数
├── static/                   # 图片、附件、静态资源；访问直接用 `/xxx.jpg`
├── assets/                   # css/js资源，主题样式，一般不修改
├── themes/PaperMod/          # 主题源码，不要直接修改主题内部文件！
├── layouts/                  # 覆盖主题模板，自定义重写（优先级高于themes）
├── .github/workflows/        # GitHub Actions部署流水线
└── public/                   # hugo build生成的静态网站产物，不要提交git

```

> ✨重要：**不要直接修改 themes 下主题源码**，自定义修改全部放到根目录`layouts`做模板覆盖，升级主题不会丢失改动。



## 2.3、修改站点配置文件`hugo.toml`，启用主题：

初始配置：

```yaml
baseURL = 'https://example.org/'
locale = 'en-us'
title = 'My New Hugo Project'
```

改成如下配置：

```toml
# 站点基础配置
#baseURL = "http://localhost:1313/" #本地预览
baseURL = "https://username.github.io/blog/" #线上部署，末尾必须带 /
languageCode = 'zh‑CN'
title = '我的Hugo博客'
theme = "PaperMod"

# 开启输出html
[outputs]
home = ["HTML", "RSS", "JSON"]

# 分类标签系统
[taxonomies]
category = "categories"
tag = "tags"

# PaperMod主题参数
[params]
defaultTheme = "auto" # auto跟随系统 / light浅色 / dark深色
disableThemeToggle = false # 是否关闭明暗切换按钮
author = "你的名字"
description = "个人技术博客"
ShowToc= true #全局默认开启Toc
UseHugoToc= false #不使用Hugo原生目录生成引擎，而是使用主题的生成引擎
disableImageProcessing = true #关闭 PaperMod 自带图片渲染

# 搜索功能配置
[params.fuseOpts]
isCaseSensitive = false
includeMatches = true
includeScore = true
threshold = 0.3
location = 0
distance = 100
minMatchCharLength = 2

# 主页profile模式（可选，关闭则普通列表首页）
[params.profileMode]
enabled = false
#enabled = true
title = "我的博客"
subtitle = "记录学习与思考"
imageUrl = "/blog/avatar.png"
imageWidth = 120
imageHeight = 120

# 顶部导航菜单 [[menu.main]] weight越小越靠前
[[menu.main]]
identifier = "home"
name = "首页"
url = "/"
weight = 1

[[menu.main]]
identifier = "posts"
name = "文章"
url = "/posts/"
weight = 2

[[menu.main]]
identifier = "categories"
name = "分类"
url = "/categories/"
weight = 3

[[menu.main]]
identifier = "tags"
name = "标签"
url = "/tags/"
weight = 4

[[menu.main]]
identifier = "archives"
name = "归档"
url = "/archives/"
weight = 5

[[menu.main]]
identifier = "search"
name = "🔍搜索"
url = "/search/"
weight = 6

[[menu.main]]
identifier = "about"
name = "关于"
url = "/about/"
weight = 7

#[[menu.main]]
#identifier = "github"
#name = "Github"
#url = "https://github.com/username"
#weight = 8

# Markdown渲染配置（支持代码块、公式等）
[markup.goldmark.renderer]
unsafe = true
```

### 配套说明

1. **baseURL**：后续部署到 Github Pages 后，替换成你的真实地址，格式：`https://用户名.github.io/仓库名/`
2. **languageCode**：改为`zh‑CN`实现全站中文，原来的`en‑us`是英文。
3. `outputs.home = ["HTML", "RSS", "JSON"]`：**JSON 是搜索功能必须**，不配置搜索页面无法工作。
4. `[[menu.main]]`：每一块对应一个导航栏菜单，`weight`控制排序，数字越小越靠左。
5. profileMode：`enabled=true`会开启头像 + 简介的个人名片首页；新手建议保持`false`使用默认文章列表首页。



## 2.4、解决点击「归档 / 搜索 / 关于」404（重要！现在点会报找不到页面）

PaperMod 不会自动生成这几个页面，需要手动创建。在项目执行下面命令：

```bash
# 关于页面
hugo new content/about.md

# 搜索页面
hugo new content/search.md

# 归档页面
hugo new content/archives.md
```



### 1. content/search.md 内容,（搜索页面，**必须写 layout: "search"**）

```markdown
---
title: "搜索"
layout: "search"
summary: "站内搜索"
placeholder: "输入关键词搜索文章..."
draft: false
---
<!-- search页面正文不需要写任何内容，layout会接管渲染 -->
```

### 2. content/archives.md 内容

```markdown
---
title: "文章归档"
layout: "archives"
url: "/archives/"
summary: "全部文章归档"
draft: false
---
<!-- archives页面正文不需要写任何内容，layout接管渲染 -->
```

### 3. content/about.md 自己随便写介绍

```markdown
---
title: "关于我"
date: 2026-09-01T10:00:00+08:00
draft: false
---

这里写关于页面正文，Markdown格式
- 个人简介
- 技术栈
- 联系方式
```

> ⚠️注意：把生成文件头部的 `draft: true` 全部改为 `draft: false`，否则 hugo 不会渲染页面。





## 2.5、本地预览

```bash
hugo server
```

浏览器打开：`http://localhost:1313/blog/`

> 终止预览：`Ctrl + C`



## 2.6、`.gitignore` 文件，忽略生成的public、缓存等。为git上传做准备

在项目根目录新建 / 编辑 `.gitignore` 文件，复制下面全部内容，每一项附带注释解释用途：

```yaml
# Hugo 生成的静态网站产物，不需要提交到git，Github Action会远程构建生成
public/

# Hugo 资源缓存目录，SCSS编译、图片处理缓存，本地生成，无需版本管理
resources/_gen/

# Hugo 运行时生成的临时文件
.hugo_build.lock
.hugo/

# OS系统生成垃圾文件
.DS_Store
Thumbs.db
desktop.ini

# VS Code编辑器配置缓存
.vscode/*.log
*.code-workspace-storage.json

# 编辑器、系统临时文件
*.swp
*.swo
*~

# 本地环境变量密钥（如有自定义密钥配置文件）
.env
.env.local

# npm依赖（主题自定义编译时才会产生）
node_modules/
package-lock.json
yarn.lock

# 临时草稿文件
*.tmp.md
temp-*
```



> `.gitignore`说明：
>
> 1. `public/`：hugo 编译输出的完整网站静态文件，**不要提交仓库**，交由 Github Action 服务器构建生成。
> 2. `resources/_gen/`：Sass、图片处理本地缓存，多人协作会产生冲突，不上传 git。
> 3. `.hugo_build.lock`：hugo 构建锁文件，本地构建生成。
> 4. 其余是操作系统、编辑器产生的垃圾文件，避免混入版本库。



## 2.7、修改模板主题

模板地址:`根目录/archetypes/default.md`

```markdown
+++
date = '{{ .Date }}'
draft = true
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
+++
```

改成

```markdown
---
title: {{ replace .File.ContentBaseName "-" " " | title }}
date: {{ .Date }}
draft: false
categories:
  - 技术分享
tags:
  - 标签
---
```





# 三、Github仓库准备，配置Github Pages

## 3.1 本地仓库建立链接

> git clon 远程仓库的方式可忽略此步骤

进入你的 hugo 项目本地根目录，打开终端。

### **方式 1：HTTPS 连接远程地址**

```bash
# 绑定远程仓库地址。username为实际用户名，blog为实际仓库名
git remote add origin https://github.com/username/blog.git  
```

### **方式 2：SSH 连接远程地址（推荐，避免 token、凭据管理器问题）**

> 前置条件：本机已经生成 SSH 密钥，公钥已添加到 GitHub 账号 SSH and GPG keys

```bash
# 设置远程地址为SSH格式。username为实际用户名，blog为实际仓库名
git remote add origin git@github.com:username/blog.git
```

若以前配置过，要修改可以用以下命令

```bash
# 修改远程地址为SSH格式。username为实际用户名，blog为实际仓库名
git remote set-url origin git@github.com:username/blog.git
```

### 测试远程绑定结果

```bash
git remote -v
```

- 如果输出为空：说明没有 origin，需要`git remote add origin xxx`
- 如果有 origin 地址：才不需要重配。



## 3.2 首次提交推送完整命令

git 提交基础文件

```bash
git add .
git commit -m "init hugo blog"
git branch -M main
git push -u origin main
```



# 四、配置Github Action自动部署（核心配置）

## 4.1 创建 workflow 文件

路径：`.github/workflows/hugo.yml`

> ⚠️`HUGO_VERSION`版本号和本地 hugo extended 版本保持一致！
>
> hugo version可查看本地hugo版本

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.165.0 #版本号需与本地一致
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5

```



## 4.2、提交流水线配置

```bash
git add .
git commit -m "add github action deploy"
git push origin main
```



## 4.3 仓库网页开启 Pages

1. 仓库页面 → Settings → Pages
2. Build and deployment → Source 选择 **GitHub Actions**
3. 推送代码后，进入仓库`Actions`标签，查看流水线执行状态；绿色✅代表部署成功。

> 首次部署大约等待 1‑3 分钟，访问页面：`https://用户名.github.io/仓库名/`

## ✅部署前检查清单（必看）

-  ✅ `hugo.toml`线上 baseURL 末尾带斜杠`/`；本地预览切换`http://localhost:1313/`
-  ✅ `.github/workflows/hugo.yml`文件已经提交 git
-  ✅ about.md/search.md/archives.md 全部 `draft: false`
-  ✅ 所有文章 post 下 md 文件`draft: false`，草稿不会线上渲染
-  ✅ git 子模块完整：`git submodule update --init --recursive`
-  ✅ Action 配置的`HUGO_VERSION`与本地 hugo version 版本号对齐

> 本地手动构建测试（模拟线上构建产物）

```
hugo --minify
```

生成 public 文件夹，就是最终网站静态文件。

## 常见坑

1. **submodules 一定要开启**，否则 PaperMod 主题缺失，页面空白。配置里`submodules: true`已经写好。
2. baseURL 末尾**必须带斜杠 `/`**，否则 css、静态资源 404。
3. 本地 hugo 版本和 yml 内 hugo‑version 尽量保持接近，避免构建差异，你本地是 0.140+，配置写 0.140.2。
4. 推完代码看仓库 `Actions` 标签，查看构建日志，报错可以定位问题。



# 五、撰写博客文章&本地预览

新建第一篇文章

```bash
hugo new content/posts/hello-hugo.md
```

文件路径：`content/posts/hello‑hugo.md`

编辑`content/posts/hello‑hugo.md`，把`draft:true`改为`draft:false`，写正文。

```
---
title: "第一篇 Hugo 博客文章"
date: 2026-08-31T20:00:00+08:00
draft: false
categories:
  - 技术笔记
  - Hugo
tags:
  - Hugo
  - PaperMod
  - 博客搭建
---

## 开篇
这是我的第一篇 Hugo + PaperMod 博客文章。

### 小标题
测试代码块：
```java
public static void main(String[] args){
    System.out.println("Hello Hugo!");
}
```



重启服务：`Ctrl+C` → `hugo server`，访问 `http://127.0.0.1:1313`。





# 六、日常写博客完整流程

1. 写新文章：`hugo new content/posts/文章名.md`

2. 修改文章内容，设置`draft: false`

3. 本地运行`hugo server`预览效果

4. 提交代码：

   ```bash
   git add .
   git commit -m "新增：xxx文章"
   git push
   ```

5. 等待 GitHub Actions 自动构建部署，访问网站查看线上效果。

> 访问地址：`https://用户名.github.io/仓库名/`





# 七、常见功能配置

## 7.1 图片处理（适配 Typora 导入图片）

根目录处创建文件：：`layouts/_markup/render-image.html`

```html
{{- $u := urls.Parse .Destination -}}
{{- $src := $u.String -}}

{{/* --------新增：转换 Typora 的 static 相对路径-------- */}}
{{- if hasPrefix $src "../../static/" -}}
  {{- $rest := strings.TrimPrefix "../../static/" $src -}}
  {{- $src = printf "/blog/%s" $rest -}}
{{- else if hasPrefix $src "../static/" -}}
  {{- $rest := strings.TrimPrefix "../static/" $src -}}
  {{- $src = printf "/blog/%s" $rest -}}
{{- end -}}
{{/* --------------------------------------------------- */}}

{{- if not $u.IsAbs -}}
  {{- $path := strings.TrimPrefix "./" $u.Path }}
  {{- with or (.PageInner.Resources.Get $path) (resources.Get $path) -}}
    {{- $src = .RelPermalink -}}
    {{- with $u.RawQuery -}}
      {{- $src = printf "%s?%s" $src . -}}
    {{- end -}}
    {{- with $u.Fragment -}}
      {{- $src = printf "%s#%s" $src . -}}
    {{- end -}}
  {{- end -}}
{{- end -}}

{{- $attributes := merge .Attributes (dict "alt" .Text "src" $src "title" (.Title | transform.HTMLEscape) "loading" "lazy") -}}
<img
  {{- range $k, $v := $attributes -}}
    {{- if $v -}}
      {{- printf " %s=%q" $k $v | safeHTMLAttr -}}
    {{- end -}}
  {{- end -}}>
{{- /**/ -}}

```

> ⚠️注意：把代码中的`/blog/`替换成你的仓库名，如果是用户主页仓库（username.github.io）直接改为`/`。



图片存放目录：项目根目录`static/` Typora 图片保存路径设置：`../../static/${filename}`

> 备选：图床方案（推荐）：PicGo + Github 图床，避免仓库体积过大。



# 八、扩展功能配置

## 8.1 数学公式支持（KaTeX）

PaperMod 开启公式，在`hugo.toml`的`[params]`下增加

```toml
[params]
  math = true
```

文章中使用：

```markdown
$$ E=mc^2 $$
```

## 8.2 RSS 订阅

配置文件中已经开启 home 输出 RSS，访问地址：`https://xxx.github.io/blog/index.xml`

## 8.3 自定义域名（CNAME）

1. 在域名服务商添加 CNAME 记录：`blog CNAME 用户名.github.io`
2. 在项目`static/`目录新建文件，文件名`CNAME`，内容填写你的域名，例如：`blog.example.com`
3. git 提交推送，GitHub Pages 会识别 CNAME，自动配置 HTTPS。

> 不要把 CNAME 放到 themes 文件夹，必须放在 static。

## 8.4 备份博客

博客源文件全部保存在 GitHub 仓库，不要只备份 public 产物；源仓库就是完整备份。 迁移新电脑：

```
git clone --recursive git@github.com:xxx/blog.git
cd blog
hugo server
```

> `--recursive`自动拉取子模块主题。



# 九、高频踩坑清单

## 9.1 error: src refspec main does not match any

原因：没有 commit；本地分支名称是 master 不是 main。 完整修复命令：

```bash
git add .
git commit -m "first commit"
git branch -M main
git push -u origin main
```

> 如果远程仓库预先创建了 README（有提交历史）

```bash
git pull origin main --allow-unrelated-histories
git push -u origin main
```

排查命令

```bash
git branch     #查看本地分支名字
git log        #无输出代表没有提交记录
git status     #查看文件状态
```

## 9.2 git submodule 超时、下载失败（国内网络）

报错：`RPC failed; curl 92 HTTP/2 stream 0 was not closed cleanly`

方案 1：使用 ghproxy 镜像代理地址添加子模块

```
git submodule add https://mirror.ghproxy.com/https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

方案 2：手动下载主题 zip 包（放弃 submodule，不能自动升级主题）

1. github 主题页面 Code → Download ZIP
2. 解压放到 themes 目录
3. 删除旧半成品文件夹 Windows：`rmdir /s /q themes\PaperMod`；Linux/mac `rm -rf themes/PaperMod`

> ⚠️手动 zip 方式，`.gitmodules`要删除，同时修改.gitignore，不要忽略 themes 文件夹。

## 9.3 GitHub Action 构建成功，网页打开样式错乱 / 静态资源 404

1. baseURL 末尾缺少斜杠`/`；
2. Hugo 版本本地和 Action 配置版本不一致；
3. submodules 没有递归拉取：workflow 文件必须写`submodules: recursive`；
4. 浏览器按 Ctrl+F12 打开控制台看 Network，看 css/js 请求地址。

## 9.4 本地 hugo server 正常，线上页面缺少搜索 / 归档页面

确认`search.md`、`archives.md`的`draft: false`已经提交到 git。

## 9.5 图片线上 404

1. static 路径问题；
2. render-image.html 里面的路径前缀没有改成自己仓库名；
3. 使用绝对路径 `/images/demo.png`。



# 十、性能&SEO

1. 图片压缩后再上传 static，大图尽量使用图床，避免 git 仓库体积膨胀；
2. 开启`--minify`压缩 HTML、CSS、JS（Action 脚本已配置）；
3. 文章合理填写 title、description、tags、categories，利于 SEO；
4. 不要提交大量图片到 git 仓库，图片建议使用外部图床；
5. 定期清理无用草稿文件。



# 十一、附录常用命令速查表

```bash
#本地预览
hugo server
hugo server --noBuildDrafts #不渲染草稿

#新建文章
hugo new content/posts/xxx.md

#本地构建静态文件
hugo --minify

#子模块操作
git submodule update --init --recursive    #首次拉取主题
git submodule update --remote              #更新全部主题

#git推送
git add .
git commit -m "msg"
git push
```
