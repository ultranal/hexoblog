---
layout: post
title: 从 GitHub Pages 迁移到 Hexo：Hexo + Travis CI + VPS + GPages，双镜像站点侧记
date: 2019-02-15 11:31:25.366545678 +08:00
categories:
  - blog
tags: 
  - hexo
  - travis
  - jekyll
---

这个博客最早是建立在 GitHub Pages 之上的，之后由于 GitHub 不支持自定义域名的 HTTPS 迁移到阿里云。当时由于 GitHub Pages 直接迁移的原因，在阿里云上仍旧使用的是 Jekyll。前些天在调整站点布局的过程中 Jekyll 正式寿终正寝了，修复颇烦，故借此机会迁移到 Hexo。

# 对比：Hexo vs Jekyll

Jekyll 是一个很有历史的静态 web 生成框架，早在 2008 年就推出首版，且至今仍是 GitHub Pages 直接支持的唯一生成框架。Hexo 则是由台湾人创建的一套框架，在中文支持方面有较强的天生优势。

总体来说，Jekyll 和 Hexo，一个功能，两个轮子，在很多方面并没有本质的区别。促使我转投 Hexo 的原因主要有二：
1. Hexo 的官方主题维护比较完善，其中不乏 NexT 这样的「大杀器」；
2. Hexo 使用 Node.js，部署、维护比较方便。

但 Hexo 也有自己的劣势：Hexo 不受 GitHub Pages 支持，需要本地编译 + 本地部署，而这为后面的部署造成了不小的[麻烦](#为何不选择本地部署？)。

# Hexo 部署

## 本地部署

参考 [Hexo 官方文档](https://hexo.io/zh-cn/docs/deployment)部署 Hexo：
```
$ npm install -g hexo-cli # 安装 Hexo 核心组件
$ npm install hexo-deployer-git --save # 安装 Git 自动部署工具
```

本地版的使用十分简单，例如创建一个站点只需要三步：
```
$ hexo init <my-awesome-website>
$ cd <my-awesome-website>
$ npm install
```
再根据需要修改配置文件 `_config.yml` 即可。

也可以自动部署，调整配置文件：
```yaml
deploy:
  type: git
  repo: https://github.com/<your_username>/<your_repository>.git 
  branch: <your_branch>
```
再执行 `hexo deploy` 就可以实现自动部署到 GitHub Pages 了。

## 为何不选择本地部署？

Hexo 自动部署功能依赖本地的 Hexo 环境来编译页面。而现在的多数人手边恐怕都有超过一台终端设备，依赖本地部署就无法实现在所有终端上编辑博客了。

换言之，我们要实现的目标是「多端编辑 + 自动构建 + 自动部署到 VPS 和 GitHub Pages」。参考以下几篇文章，决定使用 Travis CI 实现自动部署：
* [利用 CI 自动部署 hexo 博客](https://segmentfault.com/a/1190000013286548)
* [使用 Travis CI 部署你的 Hexo 博客](https://zhuanlan.zhihu.com/p/37014376)
* [使用 Travis 自动部署 Hexo 到 Github 与 自己的服务器](https://segmentfault.com/a/1190000009054888)
* [Hexo遇上Travis-CI：可能是最通俗易懂的自动发布博客图文教程](https://juejin.im/post/5a1fa30c6fb9a045263b5d2a)

# Travis CI 自动构建

## 持续集成、持续交付和持续部署

持续集成（Continuous Integration）是随着敏捷开发的流行而发展出的概念，指的是开发者「持续地」将代码集成到主干。具体来说就是，通过「小步快跑」的快速提交和自动化的软件测试，最大限度地提高软件的开发效率，保证快速迭代；同时，「小步快跑」模式也保证了及时、有效地发现 bug，使得修改也比较容易。

在持续集成之外，还产生了持续交付（Continuous Delivery）和持续部署（Continuous Deployment）两个概念。持续交付指快速地将通过自动测试的代码打包、发布，交付给最终用户；持续部署则是指交付后自动化将产品部署到生产环境。三个「持续」，实现了「问题早发现，修改早合并，功能早分发」的敏捷开发过程，可以说是敏捷开发、自动化开发部署的良好实践。

目前流行的持续集成技术有 [Jenkins](https://jenkins.io/) 和 [Travis CI](https://travis-ci.org) 等。其中 Travis CI 跨平台支持性以及和 GitHub 的兼容性都比较好，且对开源项目免费，是目前比较流行的持续集成实践。

## Travis CI 使用和配置

Travis CI 主要通过 `.travis.yml` 来配置。注册 Travis CI，打开所需项目的持续集成开关后，新建 `.travis.yml`:

```yaml
language: node_js
node_js: stable

addons:
  ssh_known_hosts: lpnal.top

cache:
  directories:
  - node_modules
before_install:
- openssl aes-256-cbc -K $encrypted_0xdeadbeef_key -iv $encrypted_0xdeadbeef_iv
  -in id_rsa.enc -out /tmp/id_rsa -d
- eval "$(ssh-agent -s)"
- chmod 600 /tmp/id_rsa
- ssh-add /tmp/id_rsa
- npm install -g hexo-cli
install:
- npm install
- npm install hexo-deployer-git --save
- npm install hexo-deployer-rsync --save
- npm install hexo-renderer-pug --save
- npm install hexo-renderer-scss --save
script:
- hexo clean
- hexo generate
after_success:
- git config --global user.name "ultranal"
- git config --global user.email "ultranal@gmail.com"
- sed -i "s/github_token/${GITHUB_TOKEN}/g" ./_config.yml
- hexo deploy
```

### 格式说明

详细的说明可以参考 [Travis CI 官方文档](https://docs.travis-ci.com/user/customizing-the-build/)。

`.travis.yml` 里的各个分段代表了 Travis CI 构建过程中的各个生命周期。比较核心的是这两个：
1. install：安装编译所需的所有依赖
2. script：执行编译脚本

除此以外的生命周期有：
1. before_install：安装前配置
2. before_script：编译前配置
3. after_success：编译成功后配置
4. after_failure：编译失败后配置
5. 可选的 deploy：自动化部署阶段
6. 可选的 cache 缓存，addons 插件等各个阶段

### 自动部署是怎么实现的？

我们的 `.travis.yml` 中，`install` 和 `script` 两个分段分别执行的是标准的依赖安装和构建，这里不做赘述。

重点说明一下 `before_install` 和 `after_success` 两部分。Travis CI 自带的部署脚本不支持推送到 GitHub Pages 或者 VPS，所以我们使用的是 hexo 的部署功能实现部署，`before_install` 和 `after_success` 的存在主要是为了调整环境、配合部署。

看一下 `.config.yml` 的部署配置：
```yaml
deploy:
  -  type: rsync
     host: lpnal.top
     user: <username>
     root: <web_directory>
  -  type: git
     repo: https://github_token@github.com/<your_username>/<your_repository>.git 
     branch: master
```
第一条是通过 rsync 推送到 VPS，第二条则是推送到 GitHub Pages。

```bash
sed -i "s/github_token/${GITHUB_TOKEN}/g" ./_config.yml
```
这一条命令主要配合 GitHub 的部署。我们生成了一个具有项目访问权限的 API Key 并放到 Travis 的项目环境变量内，这里利用 `sed` 将配置文件的占位符替换成真实的 API Key。


```bash
openssl aes-256-cbc -K $encrypted_0xdeadbeef_key -iv $encrypted_0xdeadbeef_iv -in id_rsa.enc -out /tmp/id_rsa -d
eval "$(ssh-agent -s)"
chmod 600 /tmp/id_rsa
ssh-add /tmp/id_rsa
```
这四条命令用来配合 rsync 部署。Hexo 的 rsync deployer 比较怪异，不是通过 rsync 协议去配置，反倒是基于 ssh 协议的。上面的 `_config.yml` 内配置了 ssh 的登录名和网站根目录，这四条命令则是用于解密并配置 ssh 登陆私钥（加密后存储在 repo 根目录下）。

在本地安装 Travis 的命令行工具 `travis`（基于 Ruby）：
```
$ gem install travis
```

通过 `travis` 加密私钥并修改 `.travis.yml` 配置文件：
```
$ travis encrypt-file ./id_rsa --add -r <your_username>/<your_repository>
```
这会在 `.travis.yml` 下自动生成上述四条命令中的第一条 `openssl`，解密所需的 `key` 和 `iv` 也会自动放到环境变量里 。

编辑 `.travis.yml`，配置 Travis 自动读取私钥：
```yaml
addons:
  ssh_known_hosts: lpnal.top

before_install:
- openssl aes-256-cbc -K $encrypted_0xdeadbeef_key -iv $encrypted_0xdeadbeef_iv
  -in id_rsa.enc -out /tmp/id_rsa -d
- eval "$(ssh-agent -s)"
- chmod 600 /tmp/id_rsa
- ssh-add /tmp/id_rsa
```
其中 `ssh_known_hosts` 插件是用来跳过 ssh 初次登陆的证书检查的。


将修改提交到 Git：
```
$ git add id_rsa.enc .travis.yml
$ git commit
$ git push origin master
```

好了，大功告成，提交一个 commit 作为测试：
```
$ git commit
$ git push origin master
```

回到 Travis 的 Dashboard，如果一切正常，你将可以看到「build: passing」的标志。大功告成！