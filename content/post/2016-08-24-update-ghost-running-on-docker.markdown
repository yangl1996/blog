---
date: "2016-08-24T04:13:16Z"
image: /content/images/2016/08/Rear_Deck_of_Maersk_EEE.jpg
title: 升级运行在 docker 中的 ghost 站点
---

我的 Vultr VPS 中的所有应用全部放在 docker 里跑，一开始选择这种方法考虑的就是降低维护成本。最近把这个站点的 ghost 从 0.7 升级到 0.9，也确实地感觉到容器化减轻了我维护站点的工作量。

下面是升级的简单流程，作为备忘。如果你想参考这个流程操作，请务先必看完全文再三确认，以免丢失数据。

首先，要停下原来的 container。container name 是 `blog`。

```
docker stop blog
```

站点的数据我了用独立的 [data volume container](https://docs.docker.com/engine/tutorials/dockervolumes/#/creating-and-mounting-a-data-volume-container) 存储，名称为 `ghoststore`，所以在删除 `blog` 的时候，里面的数据不会丢，因为事实上所有由我产生的数据（配置、文章、图片等）都在 `ghoststore` 里安全地存放着。顺便一说，一个理想的 container，应该是“无状态”的，也就是，随着 app 的运行，它里面的文件不会改变。利用 data volume container 很好地实现了这一点。

然后放心地删除 `blog`。

```
docker rm blog
```

下面从 docker hub pull 下来新的 ghost image。

```
docker pull ghost
```

然后直接运行，并把 data volume container 挂载上就可以了。

```
docker run --name blog --volumes-from ghoststore --net=blog_isolated -e "NODE_ENV=production" -d ghost
```

顺便介绍一下几个命令行参数的意义：如前所述，所有数据放在一个 data volume container 里，名字 `ghoststore`。 ghost 运行在 `production` 模式。另外，为了 SSL，我用了一个 nginx 做反向代理，nginx 与 ghost 之间的网络走的不是 docker 默认的 “`bridge`”，而是我自建的 `blog_isolated`，所以 run 的时候，把新 container 连接上。

整个过程不会超过 5 分钟，比直接替换源文件要简便很多，也放心很多。因为只要 data volume container 不丢失，数据不会有任何危险。