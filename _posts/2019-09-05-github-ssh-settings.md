---
layout: post
title: github ssh 配置
categories: [开发工具]
tags: Git
---

> Git 可以使用四种主要的协议来传输数据：本地传输，SSH 协议，Git 协议和 HTTP 协议[1]。其中，本地协议由于目前大都是进行远程开发和共享代码所以一般不常用，而git协议由于缺乏授权机制且较难架设所以也不常用。
>
> 因此，大家使用github时，最常用的方式就是：https url 克隆；ssh url 克隆
>
> - https url 克隆：
>   - 优点：
>     - 简单，不需要事先做任何限制，只要安装了git之后，连上网就可以clone项目
>     - 不需要成为项目的管理者或者拥有者就可以克隆项目
>   - 缺点：
>     - 慢
>     - 每次push都要账号密码，这个属于长痛
> - ssh url 克隆：
>   - 优点：
>     - 如果设置ssh key的时候没有设置密码，那么push的时候就不需要每次都输入密码
>   - 缺点：
>     - 使用前需要一个配置过程，不过这个属于短痛

## github 的 SSH 配置

### 第一步：桌面（哪都行）打开git bash

​	![](https://ws1.sinaimg.cn/large/9cd40bd3gy1g6nvgv7d1bj20ds0eqdlc.jpg)

### 第二步：  配置全局的git用户名和邮箱

```properties
git config --global user.name "github 用户名"
git config --global user.email "github 注册邮箱" 
```

### 第三步 ：配置ssh key

```properties
ssh-keygen -t rsa -C "github 注册邮箱"
```

输入命令后，然后连续按三个回车即可

### 进入.ssh 文件夹目录

.ssh文件夹的目录位置：C:\用户\用户名文件夹\.ssh

![](https://ws1.sinaimg.cn/large/9cd40bd3gy1g6nvpdqb09j20hp05jt8y.jpg)

打开id_rsa.pub （使用notepad++），然后将内容复制之后添加到github中的，github个人设置里面的SSH and GPG keys

点击 New SSH key按钮

![](https://ws1.sinaimg.cn/large/9cd40bd3gy1g6pqwsobyjj20ug09cq3m.jpg)

title 起个名字，key就是前面复制的id_rsa.pub 的内容，点击Add SSH key 保存即可

![](https://ws1.sinaimg.cn/large/9cd40bd3gy1g6pqwxfl91j20x10czq3g.jpg)

### 第四步：验证github

```
ssh -T git@github.com
```

回车，输入yes 然后出现最后的红框，代表配成功和完成。

![](https://ws1.sinaimg.cn/large/9cd40bd3gy1g6pqwytqxfj20pt06374p.jpg)

## Tips 

### git bash 窗口的所有命令和结果

![](https://ws1.sinaimg.cn/large/9cd40bd3gy1g6nvnjclb8j20qw0o1tav.jpg)

## 参考文献：

[1].[ 服务器上的 Git - 协议]