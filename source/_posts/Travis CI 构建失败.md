---
title: Travis CI 构建失败
tags:
  - null
categories:
  - 疑难杂症
date: 2019-10-29 20:49:00
updated:
description:
---

很长时间没有更新过 Github Project，今天小幅更改后发现 Travis CI 构建失败，错误提示如下：
<img src='https://raw.githubusercontent.com/drtrang/blog/master/7FhyOz.jpg'/>

初步怀疑是 os 或者 jdk 版本问题，根据错误提示 google 一圈后在 Travis CI 社区网站找到了[问题讨论](https://travis-ci.community/t/error-installing-oraclejdk8-expected-feature-release-number-in-range-of-9-to-14-but-got-8/3766/4)，但是貌似也没人解释明白，只是提供了解决方案，实测 SomioNaArashi 的方案是可行的：
<img src='https://user-images.githubusercontent.com/13851701/67770024-8085d580-fa90-11e9-8f27-a49a35bdd277.png' height='55%' width='55%'/>

### 参考资料

* 构建失败链接：https://www.travis-ci.org/drtrang/druid-spring-boot/builds/604401869
* 构建成功链接：https://www.travis-ci.org/drtrang/druid-spring-boot/builds/604420886
* 社区讨论：https://travis-ci.community/t/error-installing-oraclejdk8-expected-feature-release-number-in-range-of-9-to-14-but-got-8/3766
