---
title: git-ssh 配置和使用
date: 2022-03-21 15:48:32
tags:
  - Notes
---

## 重拾 hexo 不会使用了

---

一晃已经快要两年没有更新过自己的 Github Page 了，过去一直都是使用 hexo 进行部署的，那么还是继续使用 hexo 了，不过在这次发布页面的时候遇到了一个问题，也许我曾经遇到过，但是我已经不记得如何处理了，为了避免后续的部署遇到同样的问题，那还是做个小小的笔记比较好，把这个问题的解决过程写下来，避免后续浪费时间。

<!--more-->

## 遇到的问题

---

当我使用 hexo d 进行发布的时候，遇到了如下的报错：

![error](https://raw.githubusercontent.com/ksana410/mystorage/master/img/Snipaste_2022-03-22_00-09-57.png)

这个时候就需要求助搜索引擎了，搜索了一圈之后，发现我过去把项目的上传方式设置成了 ssh，知道了原因就好办了

## 解决方法

---

只需要在本地服务器上配置 SSH Keys 就可以了：

```shell
ssh-keygen -t rsa -C email地址 //此处的 email 地址需要和你的 Github 的 email 地址一致
cat ~/.ssh/id_rsa.pub // 获取公钥，将获取到的公钥添加到你的 Github 的 SSH Keys 中就可以了
```

![SSH KEY](https://raw.githubusercontent.com/ksana410/mystorage/master/img/SSH%20KEY.png)

> 名字可以随意，只要能区分每个 key 的所属就可以了

## 总结

---

荒废了太久了，希望现在重拾还不算晚

## 历史

---

* **2022.03.21**: 难得的一气呵成
