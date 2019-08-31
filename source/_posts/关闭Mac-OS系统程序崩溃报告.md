---
title: 关闭Mac OS系统程序崩溃报告
top: false
cover: false
password: ''
toc: true
mathjax: true
summary: >-
  如果你希望关闭MacOS系统软件崩溃报告窗口在桌面的显示，只需在「终端」中输入如下命令，回车确认即可。下次如果有应用程序崩溃或意外退出，崩溃报告将不再会在桌面中显示。
tags:
  - Mac
categories:
  - Other
date: 2019-08-28 16:51:26
---

# 关闭崩溃报告显示
如果你希望关闭崩溃报告窗口在桌面的显示，只需在「终端」中输入如下命令，回车确认即可。下次如果有应用程序崩溃或意外退出，崩溃报告将不再会在桌面中显示：
```shell
defaults write com.apple.CrashReporter DialogType none
```
恢复成默认的对话框形式的话，只需执行如下命令：
```shell
defaults write com.apple.CrashReporter DialogType crashreport
```
# 让崩溃报告在「通知中心」显示
如果你希望让崩溃报告在「通知中心」显示，只需在「终端」中输入如下命令，回车确认即可。下次如果有应用程序崩溃或意外退出，崩溃报告将以通知的形式显示在屏幕右上角：
```shell
defaults write com.apple.CrashReporter UseUNC 1
```
恢复成默认的对话框形式的话，只需执行如下命令：
```shell
defaults write com.apple.CrashReporter UseUNC 0
```

# 说明
通常来说，最好让崩溃报告保持默认显示设置，因为发送崩溃报告给 Apple 或软件开发商能够帮助其调试和修正错误。但如果你并不喜欢的话，那就根据个人偏好修改默认设置好了。

需要特别说明的是，不论是关闭崩溃报告在桌面的显示，还是令其在「通知中心」显示，都不会对崩溃报告本身造成任何影响。前述命令只不过是令其不再在用户界面显示，或让其换一种方式显示罢了。通过「控制台」或点击崩溃报告通知，我们依然能够正常查看到相关信息。