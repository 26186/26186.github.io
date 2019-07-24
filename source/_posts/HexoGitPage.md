title: GitPage+Hexo搭建个人博客
author: 26186
tags:
  - GitPage
  - Hexo
categories:
  - 环境搭建
date: 2019-07-24 15:51:00
---
### 准备环境
[注册GitHub](https://github.com/)
[安装Git](https://gitforwindows.org/)
[安装GitHub Desktop](https://desktop.github.com/)
[安装Node.js](https://nodejs.org/en/)
>本文涉及到的命令都使用Git Bash操作
### 本地创建SSH并配置GitHub
#### 验证本地是否存在SSH，没有则创建。
```sh
$ cd ~/.ssh
```
> 目录不存在说明没有SSH

#### 创建SSH
```sh
$ ssh-keygen -t rsa -C "xxxxxx@qq.com"
```
> 不需要设置文件目录和密码，直接三个回车。
`xxxxxx@qq.com`替换成GitHub登陆邮箱。

#### 查看拷贝SSH
```sh
$ cat ~/.ssh/id_rsa.pub
```
> 或者在`C:\Users\用户名\.ssh`目录下打开id_rsa.pub编辑复制(windows用户)

#### GitHub上配置SSH
> GitHub上依次点击 账号下 -> Settings -> SSH and GPG keys -> New SSH key

![upload successful](/images/pasted-0.png)
#### 验证SSH
```sh
$ ssh -T git@github.com
```
> 成功会出现 Hi xx账号

### GitHub博客仓库
#### 创建GitPage仓库`账号.github.io`
> `账号.github.io`名字的仓库默认生成GitPage，`账号`是GitHub的登录账号。
> 创建完之后在仓库上依次找到 Settings -> GitHub Pages -> Choose a theme -> 提交默认文档页。
> 一个账号下面只能存在一个GitPage。
> 当然也可以自己用其他名字，需要后期配置GitPage。

![upload successful](/images/pasted-2.png)
> 此时博客已经创建完成地址如下：https://账号.github.io/

![upload successful](/images/pasted-1.png)
#### 创建Code分支存放博客原文件
> 在仓库上依次找到 Settings -> Branches -> 设置Code为默认分支
> 使用GitHub Desktop客户端Clone设置的Code分支仓库

### 创建Hexo模板博客
#### 安装Hexo
```sh
$ cd /D/hexo/                 #选个hexo目录
$ npm install -g hexo-cli     #安装hexo脚手架
$ hexo init                   #hexo初始化
```
> 把Hexo生成的文件拷进本地Code分支仓库。
#### 启动Hexo服务
> 命令进入本地Code分支仓库。
```sh
$ npm install    #安装依赖包
$ hexo g         #生成静态文件
$ hexo s         #启动服务器，用来本地预览
```
> Hexo博客预览地址：http://localhost:4000/

![upload successful](/images/pasted-3.png)

#### Hexo生成的博客页面提交到master分支仓库
> hexo g生成的是博客的静态文件，这些文件存放在publish文件夹下
> 配置_config.yml
```yml
deploy:
  type: git
  repository: git@github.com:账号/账号.github.io.git
  branch: master
```
```sh
$ npm install hexo-deployer-git --save  #hexo安装git插件
$ hexo d                                #提交到master分支仓库
```
> 出现异常执行以下命令
```sh
$ hexo clean && hexo g   #清理重新编译
```
> 等几分钟master分支仓库内容就会更新成hexo博客。

![upload successful](/images/pasted-4.png)

#### 本地Code分支仓库提交
> 使用GitHub Desktop客户端把博客的原文件提交到Code分支

![upload successful](/images/pasted-5.png)

### 搭建博客管理后台环境
```sh
$ npm install --save hexo-admin
```
![upload successful](/images/pasted-6.png)
> 管理地址：http://localhost:4000/admin/#/
> 友好支持MarkDown语法，此篇文章使用hexo-admin写作完成。
> 图片直接复制粘贴，windows用户注意图片路劲`\`改成`/`。
> 推荐几个MarkDown语在线编辑器 [editor.md](https://pandao.github.io/editor.md/) [dillinger](https://dillinger.io/) [mdeditor](https://www.mdeditor.com/)