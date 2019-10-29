---
title: Hexo101
date: 2019-04-28 15:29:56
tags: 
---

使用Hexo和GitHub一分钟快速搭建属于自己的个人学习网站

<!--more-->

### 准备工作

```
npm i hexo-cli -g
npm install hexo-deployer-git --save
```

### 部署

```
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com/child
root: /child/
```

```
deploy:
  type: git
  repo: https://github.com/yourname/your_repo.git
  branch: gh-pages
```

### 完成

```
https://yourname.github.io/your_path/
```
新鲜出炉

就是这么迅速😏，接下来就是自己探索了
