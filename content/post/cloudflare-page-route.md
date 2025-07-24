+++
date = '2025-07-24T18:21:11+08:00'
draft = true
title = 'Cloudflare 页面规则使用示例'
tags = ['Cloudflare']
categories = ['分享']
+++

目前，我所有和域名——hubianluanma.com相关的网站都是通过Cloudflare进行DNS解析和部署，其中主站的页面是通过它的Pages功能进行部署，好处是免费零成本。本文主要介绍如何让用户不管访问[https://hubianluanma.com](https://hubianluanma..com)还是[https://www.hubianluanma.com](https://www.hubianluanma.com)都可以访问到主站。

要做到这一点其实只要在Cloudflare中有CNAME为www的记录就可以，然后剩下的可以利用它的一个页面规则进行路由，具体我们来设置一下，首先在DNS中增加www记录，如下图：

![dns](https://file.hubianluanma.com/u/7SrSUH.png)

上图中主要要加的是两条，最重要的是 www 这条，它确保我们可以访问到部署到 Cloudflare 中的网站，而另外一个A类型名称为hubianluanma.com的记录主要作用是让它能够被DNS识别到，设置192.0.2.1表明拒绝请求，而是直接让它根据规则路由，页面规则的设置如下图：

![rules](https://file.hubianluanma.com/u/aNRYPL.png)

OK，通过上面的简单设置我们就可以实现加不加www都可以正常访问我们的网站了，Cloudflare还有很多强大的功能，后续我会继续探索，对于个人开发者它可以让我们零成本的上线一些东西，很值得学习。

