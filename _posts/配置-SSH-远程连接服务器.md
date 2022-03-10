---
title: 配置 SSH 远程连接服务器
date: 2022-03-10 11:19:33
update: 2022年03月10日11:20:24
tags:
  - SSH
  - 远程连接
categories:
  - 杂记
keywords:
  - SSH
  - 远程连接
cover: https://www.howtogeek.com/wp-content/uploads/2021/10/ssh-secure-shell-laptop.png?width=1198&trim=1,1&bg-color=000&pad=1,1
typora-copy-images-to: upload
---

# 配置 SSH 远程连接服务器

> 本机：macOS
>
> 服务器：Ubuntu 18.04
>
> ​	IP：192.168.88.156（局域网下）
>
> ​	User：g11

## 1. 生成 SSH 密钥对

通过生成 ssh 密钥可以实现本机对服务器的免密登录，ssh、scp 等操作不需要输入密码，避免密码被暴力破解的问题。

本机生产 ssh 密钥：

```shell
ssh-keygen -t rsa -C "lizijing" -f ~/.ssh/192

# -t rsa: 指定创建的密钥类型
# -C "lizijing": 提供一个注释
# -f ~/.ssh/192: 指定输出文件，~/.ssh 是用户 ssh 的配置目录，192 为密钥对名
```

执行后会要求输入 passphrase 密语，直接回车就可以。

> 使用密码的好处：密钥的安全性，如果受密码保护的私钥落入未经授权的用户手中，他们将无法登录到其关联帐户，直到他们找出密码短语。当然，使用密码短语的唯一缺点是每次使用密钥对时必须键入它。

执行后在 `~/.ssh` 目录下可见：

![image-20220310135201983](https://s2.loli.net/2022/03/10/MP5qXkoDuhsSEze.png)

其中 `192` 为生成的私钥，`192.pub` 为生成的公钥

## 2. 复制 SSH 密钥至服务器

生成密钥对后需要将 ssh 公钥信息复制到服务器的 SSH 配置中：

```shell
ssh-copy-id -i ~/.ssh/192.pub g11@192.168.88.156

# -i ~/.ssh/192.pub 指定公钥文件
# g11@192.168.88.156 指定服务器信息
```

此时再次通过 ssh 连接 g11 就不必输入密码

在服务器的 `~/.ssh` 目录下有文件：`authorized_keys` 保存了 ssh 公钥信息：

![image-20220310140949267](https://s2.loli.net/2022/03/10/cQDISHk9rCl2FER.png)

## 3. 配置本机的 SSH config

在本机的 `~/.ssh/` 目录下创建文件 `config` ，用来配置本机 ssh 的部分属性，比如保持连接：

```shell
Host *
# 配置Cline端的 ssh 每 30s 向 Server 的 sshd 发送 keep-alive 包
ServerAliveInterval 30
# 若发送 10 次 Server 无回应断开连接
ServerAliveCountMax 10

# 配置 g11@192.168.88.156
Host g11 # 主机别名
HostName 192.168.88.156 # 主机 IP
User g11 # 用户
IdentityFile ~/.ssh/192 # SSH 识别文件，即刚才创建的 ssh 密钥对中的私钥
```

完成后连接服务器只需要输入我们配置的主机别名即可：

```shell
# ssh g11@192.168.88.156
ssh g11
```

scp 操作也可以用主机别名，无需在输入完整的用户名和 IP。
