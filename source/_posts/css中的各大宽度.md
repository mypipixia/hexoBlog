---
title: css中div的各大宽度,高度
date: 2019-12-09 09:42:57
tags: css
categories: 前端
---

css 盒子模型是由内容区域，内边距，边框，外边距组成

![](/pipixia.github.io/images/box/box.png)

## offsetWidth + offsetHeight

offsetWidth：内容区域 width + 左右 padding + 左右 border-width
offsetHeight：内容区域 height + 上下 padding + 上下 border-width

![](/pipixia.github.io/images/box/offset.png)

## clientWidth + clientHeight

clientWidth：内容区域 width + 左右 padding
clientHeight：内容区域 width + 上线 padding

![](/pipixia.github.io/images/box/client.png)

## scrollWidht + scrollHeight

scrollWidht：内容真实区域的宽度，在没有滚动条时，等于 clientWidth。
scrollHeight： 元素内容真实的高度，在没有滚动条时，等于 clientHeight
