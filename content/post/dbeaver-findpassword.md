+++
date = '2025-10-20T17:42:29+08:00'
draft = false
title = 'DBeaver 查看保存的密码'
tags = ['数据库','工具']
categories = ['分享']

+++

DBeaver 保存的密码怎么查看（MAC）？

当我们在DBeaver中保存了数据库的密码后没有类似小眼睛的按钮让我们直接查看，下面记录了在mac下如何查看密码。

首先，我们找到工作空间的路径，一般点击设置->常规->工作空间，这时我们会看到工作空间的完整路径：

![dbeaver setting](https://file.hubianluanma.com/u/TRD0Eo.png)

然后进入到这个目录下，再进入到 General/.dbeaver 下面，我们会看到 `credentials-config.json`，这里面就存储了用户和密码信息，只不过它是被加密的，我们需要对它进行解密：

```bash
openssl aes-128-cbc -d \                                                               🐍 base
  -K babb4a9f774ab853c96c2d653dfe544a \
  -iv 00000000000000000000000000000000 \
  -in credentials-config.json | \
  dd bs=1 skip=16 2>/dev/null
```

执行上面的命令后在控制台就会输出一串json字符串，里面包含了用户名和对应的密码。

![image-20251019162114056](https://file.hubianluanma.com/u/T6QlGx.png)

