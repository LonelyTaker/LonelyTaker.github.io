---
title: 同时配置github和gitlab
date: 2022-11-22 11:35:57
categories: 其他
---



# 1. 前置条件

* 安装git
* 拥有github和gitlab账号

<br />

# 2. 步骤

整体思路：针对不同`HOST`，使用不同公钥

## 2.1 生成不同秘钥

`git bash`打开命令行窗口，执行以下命令

```shell
# 生成github账号的秘钥
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa_github -C "GithubAccount"
# 生产gitlab账号的秘钥
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa_gitlab -C "GitlabAccount"
```

## 2.2 修改config配置文件

### 2.2.1 切换至~/.ssh目录

```shell
$ cd ~/.ssh
```

### 2.2.2 修改配置文件

```shell
$ vim config
```

```
# gitlab
    # Host 与 HostName 需要相同
    Host gitlab.com
    HostName gitlab.com
    # 指定rsa秘钥文件
    IdentityFile ~/.ssh/id_rsa_gitlab

# github
    Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_rsa_github
```

**注意：**`Host`取代码托管平台的地址，如果是局域网的gitlab地址，请使用局域网地址，下文中测试的指令也需要同步修改

## 2.3 在github和gitlab上添加个人公钥

以github为例

### 2.3.1 登录github账号，并进入设置

![设置](GitHub-设置.png)

### 2.3.2 进入公钥配置

![公钥配置](GitHub-公钥配置.png)

### 2.3.3 添加公钥

![添加公钥-1](GitHub-添加公钥-1.png)

![添加公钥-2](GitHub-添加公钥-2.png)

因为配置的是github账号，所以复制`~/.ssh/id_rsa_github.pub`中的内容到上图的`Key`中

![添加公钥-3](GitHub-添加公钥-3.png)

## 2.4 测试

```shell
# github测试
$ ssh -T git@github.com
Hi GithubAccount! You've successfully authenticated, but GitHub does not provide shell access.
# gitlab测试
$ ssh -T git@gitlab.com
Hi GitlabAccount! 
```

