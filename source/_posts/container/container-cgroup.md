---
title: [container之cgroup] 
categories: container
date: 2018-01-28 12:21:00
tags: [总结2017知识点]
---

  开篇先说下为什么要开始写博客，以及为什么会有 [总结2017知识点] 这个tag

1. 为什么开始写博客
  写的第一篇博客是在13年，博客从xx搬到csdn搬到oschia，很杂乱，后来因为工作忙(懒)，放下了写博客，现在开始写博客的主要目的是，总结自己学习到的知识点，逼迫自己多总结

2. 为什么会有 [总结2017知识点] 这个tag
  知识点是2017年学到的，没有总结成文档，所以这个tag只是标注下学习/初学是在2017年。那么为什么放在2018年的归档，个人认为，没有总结的技术，很容易遗忘，又因为工作中可能很多没用上，所以放在2018年。

3. 博客样式不好看
  个人有点 强迫+追究极致，关于不美观应该是暂时的，随着博客文章的增加，会点点美化(样式+内容)，文章同样需要refactor，博客是写给自己看的，看官如果看到别人的文章，应该总结成自己的知识，写到自己博客。
4. 为什么 使用travis 自动化部署博客
  懒

# cgroup 层级规则
## n个子系统可以附加到单层上
![cgroup_rule_1](/img/container/cgroup_rule_1.png)
> cpu memory 可以挂到单层上，前提是这个子系统没有附加到到其他层级上

## 一个子系统不能同时附加到到多个层上
![cgroup_rule_2](/img/container/cgroup_rule_2.png)
> 单系统不能同时附加到一层或多层，如果被附加层已经附加了其他单系统

## 一个task 可以在多个层级
![ccgroup_rule_3](/img/container/cgroup_rule_3.png)
> 同一个task不能同一层级的不同group，但可以在不同层级

## fork出来的task，继承同一cgroup
![ccgroup_rule_3](/img/container/cgroup_rule_4.png)
> 可以机那个task移动到不同group中


