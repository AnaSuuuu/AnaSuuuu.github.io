---
title: 博客搭建踩坑指北
date: 2023-06-27 17:48:30
tags:
- 博客搭建
---

从踩坑到掉坑里

<!--more-->

本博客采用 Hexo + GitHub 搭建，具体教程网上很多不再赘述

不过这里还是推荐两个我主要参考的教程 [hexo博客搭建指北](https://ouuan.github.io/post/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E6%8C%87%E5%8C%97/) 和 [从零搭建 Hexo + Github 博客](https://venusnero.github.io/2019/01/23/build_hexo_github_blog/)

下面分享一些自己踩过的坑

### 公式块中公式无法正常换行

#### 问题简述

用 $$ 插入公式块时，本地上用 typora 显示公式换行，但网页上未显示出来

`\\` 无法正常换行

#### 解决方案

在公式块前面加上 `\begin{gather}` 末尾加上 `\end{gather}` 即可



### Majax 渲染

#### 问题简述

1. 无法正常渲染二重闭曲面积分

2. ```latex
    $\iiint_{\Omega}(\frac{\partial P}{\partial x} + \frac{\partial Q}{\partial y} + \frac{\partial R}{\partial z})dV = \unicode{8751}_{\sum}Pdydz + Qdzdx + Rdxdy $
    ```
    
     可以正常渲染出来

    $\iiint_{\Omega}(\frac{\partial P}{\partial x} + \frac{\partial Q}{\partial y} + \frac{\partial R}{\partial z})dV = \unicode{8751}_{\sum}Pdydz + Qdzdx + Rdxdy $

    ```latex
    $\unicode{8751}_{\sum}Pdydz + Qdzdx + Rdxdy = \iiint_{\Omega}(\frac{\partial P}{\partial x} + \frac{\partial Q}{\partial y} + \frac{\partial R}{\partial z})dV$
    ```
    
    渲染异常

    $\unicode{8751}_{\sum}Pdydz + Qdzdx + Rdxdy = \iiint_{\Omega}(\frac{\partial P}{\partial x} + \frac{\partial Q}{\partial y} + \frac{\partial R}{\partial z})dV$

#### 解决方案

1. 用 `\unicode{8751}` 表示即可

2. 很神奇的问题，解决方法也很玄学，将

    ```latex
    \unicode{8751}_{\sum}Pdydz + Qdzdx + Rdxdy = \iiint_{\Omega}(\frac{\partial P}{\partial x} + \frac{\partial Q}{\partial y} + \frac{\partial R}{\partial z})dV
    ```

    改为

    ```latex
    \unicode{8751}_{\sum} Pdydz + Qdzdx + Rdxdy = \iiint_\Omega(\frac{\partial P}{\partial x} + \frac{\partial Q}{\partial y} + \frac{\partial R}{\partial z})dV
    ```

    即可

    $\unicode{8751}_\sum Pdydz + Qdzdx + Rdxdy = \iiint_\Omega(\frac{\partial P}{\partial x} + \frac{\partial Q}{\partial y} + \frac{\partial R}{\partial z})dV$

    感觉可能是 `{}` 和 mathjax 渲染有一些冲突，具体原因还不完全清楚（？）



### 表格中的 |

#### 问题简述

由于 markdown 本身的语法，我们无法直接在表格中打出竖线 |

#### 解决方案

- 当表格内容不用 $\LaTeX$ 时，可通过转义符 `\|` 或 `&#124;` 来实现竖杠或绝对值； 
- 但是用 $\LaTeX$ 时第一个会显示为 "∥"，第二个会报错，故要用 `\vert` 

#### 参考

[Markdown表格数学公式中使用绝对值“| |”或竖杠"|"](https://blog.csdn.net/skytruine/article/details/105710349)



### 背景 和 透明度

#### 问题简述

在 Hexo 7.x 版本下设置 背景 和 透明度

#### 解决方案

- 新建 _custom.styl 文件，并在 main.styl 中引用该文件

    ```
    //个人添加
    @import "_custom.styl"
    ```

- ```stylus
    //背景图片
    body {
        background:url(https://pic.heson10.com/img/image-20200712231958010.png);
        background-repeat: no-repeat;
        background-attachment:fixed;
        background-size: cover;
        background-position:50% 50%;
    }
    
    //博客内容透明化
    //文章内容的透明度设置
    .content-wrap {
      opacity: 0.9;
    }
    
    //侧边框的透明度设置
    .sidebar {
      opacity: 0.9;
    }
    
    //菜单栏的透明度设置
    .header-inner {
      background: rgba(255,255,255,0.9);
    }
    
    //搜索框（local-search）的透明度设置
    .popup {
      opacity: 0.9;
    }
    ```

#### 参考

[Hexo+Next7.X 博客美化教程合集](https://www.heson10.com/volantis/posts/52911.html)



### 自定义文章排序

#### 问题简述

希望将个别博客置顶而设置自定义排序

#### 解决方案

在`node_modules\hexo-generator-index-pin-top\lib\generator.js`文件中找到了似乎是用于排序的代码，该文件是为了添加文章置顶功能的，但当置顶等级设置相同时，按照发布日期进行排序。

修改后如下：

```js
'use strict';
var pagination = require('hexo-pagination');
module.exports = function(locals){
  var config = this.config;
  var posts = locals.posts;
    posts.data = posts.data.sort(function(a, b) {
        if(a.top && b.top) { // 当两篇文章top都有定义时
            if(a.top == b.top) return b.updated - a.updated; // 若top值一样，则按照文章更新日期降序排列
            else return b.top - a.top; // 否则按照top值降序排列
        }
        else if(a.top && !b.top) { // 以下两种情况是若只有一篇文章top有定义，则将有top的排在前面(这里用异或操作居然不行233)
            return -1;
        }
        else if(!a.top && b.top) { //上一条已解释
            return 1;
        }
        else return b.updated - a.updated; // 若都没定义，则按照文章更新日期降序排列
    });
  var paginationDir = config.pagination_dir || 'page';
  return pagination('', posts, {
    perPage: config.index_generator.per_page,
    layout: ['index', 'archive'],
    format: paginationDir + '/%d/',
    data: {
      __index: true
    }
  });
};
```

#### 参考

[hexo 自定义文章排序](https://blog.csdn.net/qq_40790680/article/details/128971322)
