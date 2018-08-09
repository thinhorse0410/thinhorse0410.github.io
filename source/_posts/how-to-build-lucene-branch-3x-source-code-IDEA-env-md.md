---
title: 如何在IDEA里搭建lucene 3.x源码MAVEN环境
date: 2018-08-09 22:07:36
tags: 
    - full-text search
    - lucene
    - maven
    - IDEA
    - source code
categories:
    - 编程
---

#### 说明

##### lucene 7.0都出来了为啥还研究3.x的代码?

- 核心的东西变化不大,熟悉了一个版本,再来看最新的版本问题不大.
- 3.x的教材比较多,如: 刘超觉先,JavenStudio等写的文章都很好,便于搭配源码一起看.

#### 克隆lucene3.x的代码

- clone整个代码
    - git clone https://github.com/apache/lucene-solr.git
    - cd /xxx/lucene-solr/
    - 创建分支branch_3x 基于remotes/origin/branch_3x
- 直接clone指定分支
    - git clone -b branch_3x https://github.com/apache/lucene-solr.git

#### 生成IDEA项目

- cd /xxx/branch_3x/lucene-solr/
- ant idea

#### 生成pom.xml文件

- cd /xxx/branch_3x/lucene-solr/
- ant get-maven-poms(如果先切换到branch_3x分支后再运行,直接下载了个pom.xml文件.跟直接在master分支上运行不一样.)

#### 直接下载生成好的项目

- https://github.com/thinhorse0410/lucene_branch_3x

<!--more-->








