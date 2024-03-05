---
title: "macOS 下安装或更新 Dart 到最新版本"
date: 2018-07-14T00:54:31+08:00
draft: false
categories: [Dart]
---

首先检查 macOS 系统是否有

`/usr/local/Homebrew/Library/Taps/dart-lang/homebrew-dart`。

<!--more-->

如果没有就执行

`brew tap dart-lang/dart`，

然后进入 `homebrew-dart`：

`cd /usr/local/Homebrew/Library/Taps/dart-lang/homebrew-dart`。

将 `dart.rb`、`dart@1.rb`、`dart@2.rb` 中的 `storage.googleapis.com` 替换为 `storage.flutter-io.cn`。

然后运行

`brew install dart --devel` 或 `brew upgrade dart --devel --force` 安装最新的 Dart，

最后用 `dart --version` 验证安装结果。
