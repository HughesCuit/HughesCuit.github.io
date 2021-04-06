---
layout: post
title: "[译]Docker化你的React应用"
date: 2019-07-03 10:46:38
tags:
  - Docker
  - React
categories:
  - 开发笔记
---

原文：[Dockerizing a React App](https://mherman.org/blog/dockerizing-a-react-app/)

[Docker](https://www.docker.com/)是一个可以帮助你加速开发和部署流程的容器工具。如果你使用微服务架构，Docker 可以帮助你将各种小型的独立服务更方便地整合起来。并且因为你可以复制你本地的生产环境，它还可以帮你排除因为环境不同所带来的 bug。

这个教程演示了怎样使用[Create React App](https://facebook.github.io/create-react-app/)将一个 React 应用 Docker 化。我们十分关注的点有-

1. 配置一个具有代码热重启功能的开发环境
2. 使用多阶段构建配置一个准生产环境镜像[注：原文为 production-ready image]

![](https://mherman.org/assets/img/blog/docker-logo.png)

_更新:_

- May 2019:
  - 更新到最新版本的 Docker, Node, React, 和 Nginx.
  - 增加了对各个 Docker 命令和标记的解释.
  - 根据读者评论和反馈添加了许多注释.
- Feb 2018:
  - 更新到最新版本的 Node, React, 和 Nginx.
  - 增加匿名卷.
  - 增加了关于配置 Nginx 来配合使用 React Router 的详细内容.
  - 增加了使用多阶段构建生产环境的部分.

_本文使用的软件版本:_

- Docker v18.09.2
- Create React App v3.0.1
- Node v12.2.0

## 目录

- [项目安装](#项目安装)
- [Docker](#Docker)
- [Docker Machine](#Docker-Machine)
- [生产环境](#生产环境)
- [React Router 和 Nginx](#React-Router-和-Nginx)
- [接下来](#接下来)

## 项目安装

全局安装 [Create React App](https://github.com/facebookincubator/create-react-app):

```
$ npm install -g create-react-app@3.0.1

```

生成一个新应用:

```
$ create-react-app sample
$ cd sample
```

## Docker

在项目的根目录下添加一个 _Dockerfile_ :

    # 基础镜像
    FROM node:12.2.0-alpine

    # 设置工作目录
    WORKDIR /app

    # 把 `/app/node_modules/.bin` 加到$PATH中
    ENV PATH /app/node_modules/.bin:$PATH

    # 安装并缓存应用依赖
    COPY package.json /app/package.json
    RUN npm install --silent
    RUN npm install react-scripts@3.0.1 -g --silent

    # 启动应用
    CMD ["npm", "start"]

> 如果你需要的话，你可以通过`--silent`选项将 NPM 的输出过滤掉。通常不推荐这样，因为它会把错误输出也给屏蔽掉。随时注意这个可以避免在调试时浪费太多时间

新建一个*.dockerignore*:

    node_modules

这将加速 Docker 构建过程，因为我们的本地依赖项将不会发送到 Docker 守护程序。

构建并标记这个 Docker 镜像:

    $ docker build -t sample:dev .

构建完成之后启动这个容器:

    $ docker run -v ${PWD}:/app -v /app/node_modules -p 3001:3000 --rm sample:dev

> 如果你得到一个`"ENOENT: no such file or directory, open '/app/package.json".`这样的错误, 你可能需要添加一个额外的卷: `-v /app/package.json`.

这里发生了什么?

1. [docker run](https://docs.docker.com/engine/reference/commandline/run/) 的命令用我们刚刚创建的 Docker 镜像创建了一个容器实例,然后将它跑起来.
2. `-v ${PWD}:/app` 将代码装载至容器中的 “/app”目录.

   > `{PWD}`在 Windows 下可能没用. 可以在[这里](https://stackoverflow.com/questions/41485217/mount-current-directory-as-a-volume-in-docker-on-windows-10)查看 Stack Overflow 上的具体问题.

3. 为了使用容器中的“node_modules”目录, 我们再配置一个卷: `-v /app/node_modules`. 你现在就可以删除本地的 “node_modules” 目录了.
4. `-p 3001:3000` 将 3000 端口暴露给同一个网络下的 Docker 容器 (用作容器间通信) 并且将 3001 端口暴露给主机.

   > 在 Stack Overflow 上查看[该问题](https://stackoverflow.com/questions/22111060/what-is-the-difference-between-expose-and-publish-in-docker)以获取更多信息。

5. 最后, `--rm` 在容器退出后[移除](https://docs.docker.com/engine/reference/run/#clean-up---rm)这个容器和卷.

打开浏览器访问[http://localhost:3001/](http://localhost:3001/) 就可以访问这个应用了. 尝试在编辑器中对 `App` 组建上进行修改， 就能发现应用已经热重启了。在完成后关闭服务器。

> 你加了`-it`选项时发生了什么?
>
>     $ docker run -it -v ${PWD}:/app -v /app/node_modules -p 3001:3000 --rm sample:dev
>
> 按照你的理解自己验证一下.

希望使用 [Docker Compose](https://docs.docker.com/compose/)吗? 在项目根目录新增一个 _docker-compose.yml_ 文件:

    version: '3.7'

    services:

      sample:
        container_name: sample
        build:
          context: .
          dockerfile: Dockerfile
        volumes:
          - '.:/app'
          - '/app/node_modules'
        ports:
          - '3001:3000'
        environment:
          - NODE_ENV=development

注意卷的配置， 如果没有这个 [匿名](https://success.docker.com/article/Different_Types_of_Volumes) 卷 (`'/app/node_modules'`), _node_modules_ 这个目录在运行时就会被你项目中的那个目录覆盖。换句话说, 会发生下面的事:

- _Build_ - `node_modules`这个目录在镜像中被创建。
- _Run_ - 当前的目录被装载至容器中, 覆盖掉我们在 Build 时创建的`node_modules`目录.

构建镜像并运行容器:

    $ docker-compose up -d --build

再次确认应用运行成功并且可以热重启，然后在继续之前停止容器:

    $ docker-compose stop

> _Windows 用户_: 在配置卷时遇到问题的话，可以查看下列资源:
>
> 1.  [Docker on Windows–Mounting Host Directories](https://rominirani.com/docker-on-windows-mounting-host-directories-d96f3f056a2c)
> 2.  [Configuring Docker for Windows Shared Drives](https://blogs.msdn.microsoft.com/stevelasker/2016/06/14/configuring-docker-for-windows-volumes/)
>
> 你也可以在 Docker Compose 的 evironment 部分中添加 `COMPOSE_CONVERT_WINDOWS_PATHS=1`。查看 [Declare default environment variables in file](https://docs.docker.com/compose/env-file/) 教程以获取更多信息.

## Docker Machine

如果要在 [Docker Machine](https://docs.docker.com/machine/) 和 [VirtualBox](https://docs.docker.com/machine/get-started/)上搭建这个热重启服务，你需要通过[chokidar](https://github.com/paulmillr/chokidar)来[启用](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#troubleshooting)一个轮询机制 (封装了 `fs.watch`, `fs.watchFile`, 和 `fsevents`).

创建一个新 Machine:

    $ docker-machine create -d virtualbox sample
    $ docker-machine env sample
    $ eval $(docker-machine env sample)

拿到 IP:

    $ docker-machine ip sample

然后构建镜像并运行容器:

    $ docker build -t sample:dev .

    $ docker run -v ${PWD}:/app -v /app/node_modules -p 3001:3000 --rm sample:dev

访问[http://DOCKER_MACHINE_IP:3001/](http://DOCKER_MACHINE_IP:3001/)再测试一遍应用， (注意将 `DOCKER_MACHINE_IP` 替换成你真正的 Docker Machine 的 IP). 并且发现热重启 _不再_ 工作. 用 Docker Compose 也一样。

要重新启用热重启, 得加一个环境变量: `CHOKIDAR_USEPOLLING=true`.

    $ docker run -v ${PWD}:/app -v /app/node_modules -p 3001:3000 -e CHOKIDAR_USEPOLLING=true --rm sample:dev

再测试一次. 确认热重启又工作了.

更新后的 _docker-compose.yml_ 文件:

    version: '3.7'

    services:

      sample:
        container_name: sample
        build:
          context: .
          dockerfile: Dockerfile
        volumes:
          - '.:/app'
          - '/app/node_modules'
        ports:
          - '3001:3000'
        environment:
          - NODE_ENV=development
          - CHOKIDAR_USEPOLLING=true

## 生产环境

创建一个单独的 Dockerfile 命名为 _Dockerfile-prod_:

    # build environment
    FROM node:12.2.0-alpine as build
    WORKDIR /app
    ENV PATH /app/node_modules/.bin:$PATH
    COPY package.json /app/package.json
    RUN npm install --silent
    RUN npm install react-scripts@3.0.1 -g --silent
    COPY . /app
    RUN npm run build

    # production environment
    FROM nginx:1.16.0-alpine
    COPY --from=build /app/build /usr/share/nginx/html
    EXPOSE 80
    CMD ["nginx", "-g", "daemon off;"]

这里我们利用了 [多阶段构建](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) 模式来创建一个临时镜像，用来构建已经为生产环境准备的 React static 文件，然后复制到生产镜像中. 然后这个临时构建镜像和原始文件和目录一起被丢弃，就生成了一个精简后的，生产环境镜像。

> 通过这篇博客可以更多的了解多阶段构建 [Builder pattern vs. Multi-stage builds in Docker](https://blog.alexellis.io/mutli-stage-docker-builds/).

使用生产环境 Dockerfile, 构建并标记镜像:

    $ docker build -f Dockerfile-prod -t sample:prod .

运行容器:

    $ docker run -it -p 80:80 --rm sample:prod

如果你还在用同一个 Docker Machine 的话, 使用浏览器访问 [http://DOCKER_MACHINE_IP](http://DOCKER_MACHINE_IP)。

使用一个新的 Docker Compose 文件命名为 _docker-compose-prod.yml_:

    version: '3.7'

    services:

      sample-prod:
        container_name: sample-prod
        build:
          context: .
          dockerfile: Dockerfile-prod
        ports:
          - '80:80'

运行容器:

    $ docker-compose -f docker-compose-prod.yml up -d --build

再用浏览器测试一次. 然后就完成了, 然后销毁 Docker Machine:

    $ eval $(docker-machine env -u)
    $ docker-machine rm sample

## React Router 和 Nginx

如果你使用 [React Router](https://reacttraining.com/react-router/), 那么你需要再构建时修改默认的 nginx 配置:

    RUN rm /etc/nginx/conf.d/default.conf
    COPY nginx/nginx.conf /etc/nginx/conf.d

把这个修改加入到 _Dockerfile-prod_:

    # build environment
    FROM node:12.2.0-alpine as build
    WORKDIR /app
    ENV PATH /app/node_modules/.bin:$PATH
    COPY package.json /app/package.json
    RUN npm install --silent
    RUN npm install react-scripts@3.0.1 -g --silent
    COPY . /app
    RUN npm run build

    # production environment
    FROM nginx:1.16.0-alpine
    COPY --from=build /app/build /usr/share/nginx/html
    RUN rm /etc/nginx/conf.d/default.conf
    COPY nginx/nginx.conf /etc/nginx/conf.d
    EXPOSE 80
    CMD ["nginx", "-g", "daemon off;"]

创建以下的目录并且添加 _nginx.conf_ 这个文件:

    └── nginx
        └── nginx.conf

_nginx.conf_:

    server {

      listen 80;

      location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
      }

      error_page   500 502 503 504  /50x.html;

      location = /50x.html {
        root   /usr/share/nginx/html;
      }

    }

## 接下来

现在, 你就可以将 React 添加到开发和生产环境的更大的 Docker 驱动的项目中使用了. 如果你想学习使用 React 和 Docker 来构建和测试微服务, 请访问 [Microservices with Docker, Flask, and React](https://testdriven.io/) 教程.
