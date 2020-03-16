---
title: hexo常用操作
date: 2020-01-19 16:05:13
tags: Hexo
categories: 其他
---

新建文章

```
hexo new post <title>

e.g.
hexo new post Hello
```

新建页面

```
hexo new page <pagename>

e.g.
hexo new page categories
```

生成静态页面至public目录

```
hexo g 
(== hexo generate)
```

部署到Github

```
hexo d
(== hexo deploy)
```

清除部署缓存

```
hexo cl
(== hexo clean)
```

本地调试

```
hexo s
(== hexo server)
```



生成 + 部署

```
hexo d -g
```



每次部署步骤

```
hexo cl
hexo d -g
```



