---
title: {{ title }}  # 自动填充文章标题
date: {{ date }}    # 自动填充创建时间
tags: {% if tags %}{{ tags }}{% else %}[]{% endif %}  # 有标签则用标签，无则为空数组
categories: {% if categories %}{{ categories }}{% else %}Linux运维{% endif %}  # 默认分类：Linux运维
author: {% if author %}{{ author }}{% else %}Judy{% endif %}  # 默认作者：Judy
#cover: {% if cover %}{{ cover }}{% else %}/images/default-cover.jpg{% endif %}  # 默认封面图
published: true
---

# {{ title }}
> 本文首发于我的个人博客，转载请注明出处。