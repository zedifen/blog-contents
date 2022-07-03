---
title: Hello, Gatsby!
date: 2022-05-23T22:59:41
updated: 2022-05-25 02:39:43
description: 包含一些 Gatsby 的配置记录.
tags: [gatsby]
---

The very first blog built with Gatsby.

## 环境配置

笔者采用 WSL 2 作为开发环境. 关于 WSL 2 的使用技巧可以参见上一篇博客.

### 安装 nvm 和 Node.js

首先需要安装 [Node.js][nodejs] 环境. 这里参照微软的文档 [在 WSL 2 上设置 Node.js][nodejs-wls2-ms-doc], 即先安装 [nvm][nvm] 用来管理 Node.js 的版本, 再通过 nvm 安装 Node.js.

需要注意, 如有需要, 要先在命令行环境设置代理, 参考:

```shell{promptUser: henry}
export http_proxy='http://172.23.112.1:7890'
export https_proxy='http://172.23.112.1:7890'
```

[nodejs]: https://nodejs.org/
[nodejs-wls2-ms-doc]: https://docs.microsoft.com/zh-cn/windows/dev-environment/javascript/nodejs-on-wsl "在 WSL 2 上设置 Node.js | Microsoft Docs"
[nvm]: https://github.com/nvm-sh/nvm

在安装好 nvm 后, 可能需要重启 bash 才能使配置的环境变量生效. 如若重启, 记得重新配置代理.

笔者配置时, 使用最新的 Node.js (18.2.0) 会有问题, 切换到 LTS (16.15.0) 则没有问题.

```shell{promptUser: henry}
nvm install --lts
```

### 安装 Gatsby

在用 npm 安装 Gatsby 时, 如果所处的网络条件有问题, 需要设置代理, 则似乎还需要显式地为 npm 设置代理:

```shell{promptUser: henry}
npm config set proxy 'http://172.23.112.1:7890'
npm config set strict-ssl false
```

之后再 (全局) 安装 `gatsby-cli`:

```shell{promptUser: henry}
npm install -g gatsby-cli
```

## 新建项目

开始之前, 建议先参考学习 [Gatsby 的教程](https://www.gatsbyjs.com/docs/tutorial/ "Tutorial | Gatsby").

### 使用博客模板

下面的命令使用 `gatsby new` 命令, 根据 Gatsby's blog starter 的仓库新建一个项目:

```shell{promptUser: henry}
gatsby new gatsby-starter-blog https://github.com/gatsbyjs/gatsby-starter-blog
```

对模板仓库的修改过程可参见该网站源代码仓库的提交记录.

### 本地预览

之后, 进入对应的目录, 并开启 development server (开发服务器):

```shell{promptUser: henry}
cd gatsby-starter-blog
gatsby develop --host=0.0.0.0
```

追加 `--host=0.0.0.0` 参数使得 development server 侦听来自站外的请求, 以便能够从主机访问 WSL 2 上的 server 程序.

## 发布

笔者计划使用 GitHub Pages 托管页面, 而使用 GitHub Actions 实现站点的自动构建和部署, 即 "持续集成 (Continuous Integration)".

### 创建在线仓库

为了使用 GitHub Actions 实现自动构建和部署, 需要将仓库上传至 GitHub.

笔者计划将该项目直接部署到 GitHub 提供给账户的二级域名下 (也被称为用户的个人页面仓库), 即 username.github.io, 对应的仓库与其同名.

如果是部署在域名的子目录下, 也就是部署为账户下某一仓库的 Pages, 则需要额外的配置. 将在后面介绍.

> 关于不同的 GitHub Pages 类型可参考 [About GitHub Pages - GitHub Docs](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages).

### 创建 PAT 并添加到仓库 secrets

仓库的自动构建和部署是通过 GitHub workflow (工作流) 实现的. 需要注意的是, 由于工作流的执行是在远程主机上, 因此为了能够成功部署页面 (将页面推送到仓库中的分支上), 该脚本需要能够访问对应的 GitHub 仓库才行, 即需要向其提供传递一个能够鉴权的凭据.

可以参照 [Creating a personal access token][create-pat] 创建一个具有仓库 (`repo`) 访问权限的 "个人访问凭据 (Personal Access Token)", 并将其添加到仓库的 "Secrets" 中.

在本例中, 即为添加一个名为 `ACCESS_TOKEN` 的 secret 到仓库的 Actions secrets 中, 其值为上一步中创建的 PAT.

在 Actions 运行时, 仓库用于 Actions 的环境变量密钥 (Environment secrets) 和仓库访问密钥 (Repositoy secrets) 将会传递过去, 使得 Actions 能够对仓库进行修改.

[create-pat]: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token "Creating a personal access token - GitHub Docs"

### 创建工作流

之后, 需要为仓库新建一个负责构建和部署的工作流. 创建工作流的过程可以在仓库的 GitHub 页面上进行. 也就是在仓库的根目录下新建一个 `.github/workflows/build.yml` 文件 (文件名无特殊要求).

在工作流中, 笔者调用了 [enriikke/gatsby-gh-pages-action][enriikke-gh-action] 这一 GitHub Action, 其将会在仓库根目录下执行 `gatsby build`, 并将其部署到 GitHub Pages.

<!-- 为了支持 `npm` 和 `yarn`, 该 GitHub Action 需要 `package.json` 中定义了 `build` 脚本. 不过, 使用 `gatsby new` 创建项目时, Gatsby 将会自动定义该脚本为调用 `gatsby build`. -->

文件的内容 (使用方式) 类似如下:

```yml
# 工作流的名称
name: Gatsby Build & Publish

# 控制该工作流何时被执行
on:
  # 仅在 main 分支上发生 push 时被触发
  push:
    branches: [ main ]

# 一个工作流由多个可以串行或并行的工作 (jobs) 组成
jobs:
  # 该工作流仅包含一个名为 "build" 的 job
  build:
    # 指定将执行该工作的执行者的类型
    runs-on: ubuntu-latest

    # 阶段 (steps) 表示该工作 (job) 下一系列将被执行的任务 (tasks)
    steps:
      # 将该仓库签出 (check out) 在 $GITHUB_WORKSPACE 指定的目录下,
      # 以便被工作 (job) 访问
      - uses: actions/checkout@v1
      
      # 调用 enriikke/gatsby-gh-pages-action
      # 进行页面的构建和部署
      - uses: enriikke/gatsby-gh-pages-action@v2
        # 指定传递给该 Action 的参数
        with:
          # 使用的访问权限口令
          access-token: ${{ secrets.ACCESS_TOKEN }}
          # 待部署的分支名
          deploy-branch: gh-pages
```

如果是在网页上进行的工作流的创建, 保存 (提交或合并到 `main` 分支) 之后, 将会触发该工作流. 如果是在本地编辑后推送到远端仓库应当同理.

### 部署到 GitHub Pages

该工作流将会在构建完成后, 将结果输出到 `gh-pages` 分支. 等待执行完毕后, 可以在仓库的 `gh-pages` 分支下查看到对应的结果. 

同时需要进入仓库设置页的 "Pages" 下, 确保将发布源设置为工作流将构建产物所部署到的 `gh-pages` 分支.

GitHub 默认使用 Jekyll 根据指定的发布源进行构建. 因此, 用户自定义的页面构建 CI 通常会将内容发布在 `gh-pages` 分支上, 并在根目录下写入一个 `.nojekyll` 文件, 以便 GitHub 不再对这些内容进行处理; 之后, GitHub 会将分支上的内容打包, 完成 **最终** 的部署. 这也是为什么仓库的 Actions 页面实际上会多出来一个名为 "pages build and deployment" 的 workflow 的原因.

可以参考 [配置 GitHub Pages 站点的发布源][configure-publishing-source].

[configure-publishing-source]: https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site "Configuring a publishing source for your GitHub Pages site - GitHub Docs"

### 部署到子目录下

需要注意, 如果是部署在域名的子目录下 (也就是账户下仓库开启 GitHub Pages 后所对应的页面地址), 则需要进行相应的配置.

首先在项目根目录的 `gatsby-config.js` 中添加如下的配置 (即指定路径前缀):

```js
module.exports = {
  pathPrefix: "/reponame",
}
```

并在调用 [enriikke/gatsby-gh-pages-action][enriikke-gh-action] 时, 选择传递 `--prefix-paths` 参数给 Gatsby.

```yml{16}
name: Gatsby Build & Publish

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - uses: enriikke/gatsby-gh-pages-action@v2
        with:
          access-token: ${{ secrets.ACCESS_TOKEN }}
          deploy-branch: gh-pages
          gatsby-args: --prefix-paths
```

[enriikke-gh-action]: https://github.com/enriikke/gatsby-gh-pages-action "Gatsby Publish: GitHub Action to build and deploy your Gatsby site to GitHub Pages ❤️🎩"
