---
layout: article
title: GitHub 哈气了！
key: clone-while-you-can
tags: GitHub Proxy
---

如何在 GitHub 哈气的情况下使用 git clone、submodule 与 CMake download。

<!-- more -->

GitHub 的直连一直是坑坑巴巴的，最近甚至还出了实习生误操作导致整个大陆访问不了的事故。尽管官方说明已经修复，但不可否认直连状况比原来更差了。

本篇介绍一些方法来绕过因为 GitHub 哈气导致的访问问题。

* 对于 `git` 命令，可以配置 URL 改写通过镜像站访问

        git config --global url."https://ghfast.top/https://github.com/".insteadOf "https://github.com/"

* 一些项目可能会在 CMake 构建过程中从 GitHub 下载依赖，我们可以等 CMake 自然失败后，根据报错信息手动 `wget https://ghfast.top/https://github.com/xxxx/xxx/archive/xxxx` 下载依赖到同一位置，然后重试 CMake。
