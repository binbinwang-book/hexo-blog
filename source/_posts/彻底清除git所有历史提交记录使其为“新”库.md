---
title: 彻底清除git所有历史提交记录使其为“新”库
date: 2022-06-05 23:17
tags: [git]
categories: [技术方案]
---

# 彻底清除git所有历史提交记录使其为“新”库

## 前言

近期公司要求提交自己的github地址，然后会对自己项目里的github敏感信息进行扫描，防止泄露公司机密。但git提交即使你删除了敏感信息，还可能在历史的commit中找回，所以最好的方法就是彻底清楚所有提交记录，让它成为一个新的库。

## 操作步骤

### 1. 创建孤儿分支

>>> git checkout --orphan <new_branch>

### 2. 添加所有文件

>>> git add .

### 3. commit代码

>>> git commit -m "自定义提交说明"

### 4. 删除原来的主分支(master/main)

>>> git branch -D master
>>> git branch -D main

一般仓库默认的主分支为 master 分支，如果原来的主分支不是 master, 用实际的主分支名代替。

### 5. 当前分支重命名为master

>>>  git branch -m master

### 6. 最后把代码推送到远程仓库

注意： 有些仓库有 master 分支保护，不允许强制 push，需要在远程仓库项目里暂时把项目保护关掉才能推送。

>>> git push -f origin master
