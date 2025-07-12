+++
date = '2025-07-12T16:29:51+08:00'
draft = false
title = 'Mainroad 主题如何配置 Giscus 评论系统'
tags = ['建站笔记']
+++

Hello，本文介绍和记录一下如何对 **Mainroad** 这个主题配置评论模块，Mainroad 这个主题它默认支持的是 Disqus 评论系统，但是我更喜欢使用 Giscus 这个评论系统，因为它是基于 GitHub Discussions，这样既保护用户隐私，又方便管理，最重要的是它免费且纯净无广告。

结下来的步骤默认已经有 GitHub 账号了，没有的话请自行方法 [GitHub 官网](https://github.com) 进行注册。接下来我们需要做的就是配置 Giscus 应用了。

首先我们访问 Giscus 官网：[https://giscus.app](https://giscus.app)，然后找到配置部分，官网中其实已经有很明确的步骤，只要跟着步骤走就可以，不过这里我还是简单的说一下吧，共有三个步骤：

1. 首先将我们要存储评论的仓库设置为公开的，不可以是 private 的，例如我这里使用了 GitHub Pages 部署我的博客，那我就将我的仓库 [hubianluanma.github.io](https://github.com/hubianluanma/hubianluanma.github.io) 设置为了公开（本来就是 public 的，因为 pages 功能开启的时候就必须要求仓库是 public 的）。
2. 第二步就是给我们的 github 安装 giscus app，只有安装了以后才可以使用，可以直接点击 giscus 提供的链接跳转安装，或访问 [这个网址](https://github.com/apps/giscus)。
3. 最后我们需要将对应仓库的 Discussion 功能开启，这个设置位置在仓库的 settings 菜单中，往下拉就可以看到，如下图：

![github discussion](https://moodrecorder.cn/u/GrBQXE.png)

完成上面的设置后接下来在 giscus 提供的仓库下面的输入框中输入：**用户名/仓库名** 注意这里的斜杠不是或的意思。

然后下面的其他配置默认就可以，除非你知道自己在干嘛而设置其他选项，拉到最下面会看到生成的一段 script 脚本，接下来我们就会用到它。

现在我们返回到 hugo 的目录，进入到 mainroad 的主题根目录，在进入到 layouts/partials 下面新建一个文件：giscus.html 然后插入下面的代码：

```html
<div id="giscus-comments">
    <script src="https://giscus.app/client.js"
            data-repo="你的用户名/你的仓库名"
            data-repo-id="你的repo ID"
            data-category="你选择的分类名"
            data-category-id="你选择的分类ID"
            data-mapping="pathname"
            data-strict="0"
            data-reactions-enabled="1"
            data-emit-metadata="0"
            data-input-position="bottom"
            data-theme="preferred_color_scheme"
            data-lang="zh-CN"
            crossorigin="anonymous"
            async>
    </script>
</div>
```
其中 script 部分的内容就是 giscus 官网中生成的，直接粘贴进去就可以，然后我们在回到 layouts 文件夹，进入 _default 文件夹中，找到 single.html 文件，打开文件找到 disqus.html 替换为 giscus.html ，当然如果你想配置为根据 hugo 根目录的配置文件——hugo.toml 文件进行控制是否开启的话，可以仿照 comments.html 中添加相关的判断。

OK，大功告成，这下我们可以愉快的使用 Giscus 评论系统了，免费、简洁、安全。
