---
title: 从此以后，gitpages将不再受主题限制
date: 2017-11-30
categories: 上古神器
tags: git 
---
今天上github，突然发现了一个令人愉快的事，写个博客庆祝一波，那就是git pages 功能支持任何 public的git themes 了。赶紧把自己的一个工具给换了个主题，瞬间感觉高大上了，:smile: [小伙伴们可以参考一下](http://jokinglove.com/tools/) <!-- more -->

- 更换theme很方便，下面是官方消息的截图：
![](http://joking.oss-cn-beijing.aliyuncs.com/blog/post_images/git-pages-themes.png)
- 更换themes灰常简单，在自己要作为gitpages  的项目更目录新建一个_config.yml的文件，在里面加上这一句就ok了，然后git在构建的时候，会自动去相应的仓库下面拿到主题，应用的你的git pages 上面：
~~~yml
    remote_theme: owner/name
~~~
- 其中  owner是主体所有者的名称，name 就是对应的主题了。就像这样：
~~~yml
    remote_theme: pages-themes/leap-day
~~~
- 小伙伴们可以赶紧给自己的git pages换个高大上的主题玩玩了，有好的主题，也分享一波，我也换一个，一直是一个选择困难症患者，不知道什么样的主题比较适合我。但是不可否认的是，git，github很屌，屌屌的。💯 