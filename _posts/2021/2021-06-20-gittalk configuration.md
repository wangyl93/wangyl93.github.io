---
layout:     post
title:      为博客添加 Gitalk 评论插件
no-post-nav: true
category: other
tags: [other]
excerpt: 每个程序员都想拥有一个博客
---

# 为博客添加 Gitalk 评论插件

1. 配置 _config.yml

```yaml
gitalk:
    enable: true
    owner: wangyl93
    repo: wangyl93.github.io
    clientID: 32a9035277eec0fdae21
    clientSecret: xxxx92275d1eeeb0b6c5805ac330b42e44281
```

2. 以上clientID clientSecret 需要在你的github 里面的Settings--> Developer Settings -> OAuth apps 里面添加 

![image-20210620123859694](C:\java_restful\wangyl93.github.io\assets\images\2021\springboot\image-20210620123859694.png)

解释一下：

1）Application name 可以随便填一个。

2）Homepage URL 必须是博客仓库的域名（GitHub Pages 的）。

3）Authorization callback URL 必须是博客的域名（https://wangyl93.github.io）

同时生成你的 Client ID and Client secrets

![image-20210620124110429](C:\java_restful\wangyl93.github.io\assets\images\2021\springboot\image-20210620124110429.png)

3.  gitalk.js 的修改

   找到  https://github.com/login/oauth/access_token 前面的代理，

   修改前：

   proxy:"https://cors-anywhere.herokuapp.com/https://github.com/login/oauth/access_token"

   修改后：

   proxy:"https://shielded-brushlands-08810.herokuapp.com/https://github.com/login/oauth/access_token"

4. 打开博客项目的 issue

项目--> settings--> issues

![image-20210620125557621](C:\java_restful\wangyl93.github.io\assets\images\2021\image-20210620125557621.png)

5. 测试你的评论吧

   以上是我配置 gitalk 遇到的问题，希望能帮助到你，欢迎评论：）