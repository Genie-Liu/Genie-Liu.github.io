---
title:      "LibreWolf使用记录"
date:       2025-01-24T10:44:05+08:00
tags:       ["blog", "learn", "privacy"]
identifier: "20250124T104405"
draft: false
---

在我以前的生活中，Firefox 一直是我的主力浏览器，而在工作中，由于某些网页的兼容性问题，我则不得不使用 Chrome。直到我看了 Eric Murphy关于[浏览器的分类](https://www.youtube.com/watch?v=j5r6jFE8gic)的视频，受到了隐私保护的启发，我决定把 Chrome 替换成 Brave，同时对 Firefox 进行harden，或者尝试使用 LibreWolf。

## Hardened Firefox

视频中提到了两种方式来强化 Firefox： [Betterfox](https://github.com/yokoffing/Betterfox) 和 [Arkenfox](https://github.com/arkenfox/user.js)。Betterfox 旨在尽可能保护隐私而不破坏浏览器功能，而 Arkenfox 的方法则比较激进，可能会导致某些功能失效，因此建议在使用前仔细阅读 Arkenfox 的 [Wiki](https://github.com/arkenfox/user.js/wiki/2.1-User.js)。在尝试了这两种方法后，我发现有时会遇到一些奇怪的问题，但如果新建一个profile并应用 user.js，这些问题就会消失，怀疑是和之前的配置冲突所致。

后来，我发现 LibreWolf 基本上就是强化版的 Firefox，所以我决定尝试 LibreWolf。

## LibreWolf初体验

首先，我访问了 [Librewolf官网](https://librewolf.net/) 的 Installation 页面，选择了对应的系统进行安装。对于 macOS 用户，你可以通过 Homebrew 安装 LibreWolf，使用命令 `brew install --cask librewolf`，或者下载 DMG 文件进行安装。

这时候出现了一个小插曲：下载的 DMG 文件总是报错“文件损坏”，这让我怀疑下载过程出了问题。后来，我检查了一下文件的哈希值，发现并没有问题。最终，我发现需要添加 `--no-quarantine` 选项来解决这个问题（详细信息可以参考 [resolve Librewolf is damaged](https://www.reddit.com/r/LibreWolf/comments/1807jpv/just_a_quick_tip_for_resolving_librewolf_is/)）。这次错误信息实在是有点误导人，我也不知道为什么 LibreWolf 还没有解决这个问题。

目前使用下来，除了 B 站登录有点问题外，其他网站的兼容性都没有发现明显的问题。

B 站登录的问题主要有两点：

1. 二维码不显示：这是因为 Canvas 被阻止了。一般来说，我不常用这个二维码，可以通过[enable canvas](https://librewolf.net/docs/faq/#should-i-allow-canvas-access-how-do-i-do-it)解决这个问题。
2. 点击登录按钮没有反应：这是由于使用了 Enhanced Tracking Protection 的strict模式（大多数网站登录不会遇到这个问题）。要解决这个问题，只需将 B 站添加到 Enhanced Tracking Protection 的例外列表中，可以通过点击地址栏旁边的盾牌图标来进行禁用（涉及到验证码的网站也需要添加到例外列表中）。

## LibreWolf配置修改

在使用 Betterfox 强化 Firefox 的过程中，我发现了一个 `librewolf.override.cfg` 的配置文件。我查看了 LibreWolf 的 文档，发现可以通过 override 文件来修改配置，只需将 `librewolf.override.cfg` 放在 `~/.librewolf` 目录下即可。

PS：Betterfox 的这个文件在最近的提交中被删除了，原来的文件也提到最好还是使用 user.js。在尝试使用这个配置之后，我还是把它删掉了，因为如果你的隐私需求并没有那么强的话，可以考虑betterfox的harden方式，或者只需要在Firefox做一些简单的配置即可。（参考[ The Ultimate Guide to Firefox Hardening! ](https://www.youtube.com/watch?v=F7-bW2y6lcI)的 easy 设置）

## LibreWolf更新

最近，Chrome 出现了一个 [安全漏洞](https://thehackernews.com/2024/08/google-fixes-high-severity-chrome-flaw.html)，公司要求全员强制升级 Chrome。我查看了一下我的 Brave，果然紧跟 Chromium 内核，更新后已经修复了这个问题。出于好奇，我查看了 LibreWolf 和 Firefox 的版本差距，发现 LibreWolf 已经落后了一个月，期间大概有 2-3 个更新。我原以为 LibreWolf 会像 Firefox 一样有自动更新的功能，但发现需要手动升级。还好我是通过 Homebrew 安装的，只需重新运行 `brew install --cask librewolf` 即可完成升级。

为了避免忘记手动更新，我设置了一个 plist 任务，每天自动尝试更新 LibreWolf。

在 `~/Library/LaunchAgents/` 下创建了 `com.librewolf.updater.plist` 文件，并通过 `launchctl load com.librewolf.updater.plist` 添加。需要注意的是，两个输出文件 `/tmp/brewinstall.err` 和 `/tmp/brewinstall.out` 需要提前创建，否则会出现 78 错误。这个错误比较隐蔽，需要通过 launchctl list | grep libre 来查看，状态为 0 表示正常，权限问题会显示状态 78。

你可以使用以下命令来查看任务的状态：
```sh
launchctl list | grep libre
launchctl print gui/$(id -u)/com.librewolf.updater
```

下面是我使用的 plist 文件示例：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.librewolf.updater</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/brew</string> <!-- 根据你的 Homebrew 安装位置调整 -->
        <string>install</string>
        <string>--cask</string>
        <string>librewolf</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>10</integer> <!-- Update at 18 AM -->
        <key>Minute</key>
        <integer>30</integer> <!-- At the beginning of the hour -->
    </dict>
    <!-- <key>StartInterval</key> -->
    <!-- <integer>86400</integer> <!-\- 每天运行一次（86400秒 = 24小时） -\-> -->
    <key>StandardErrorPath</key>
    <string>/tmp/brewinstall.err</string>
    <key>StandardOutPath</key>
    <string>/tmp/brewinstall.out</string>
  </dict>
</plist>
```

## 总结

隐私和便利性总是一个权衡的问题。当前遇到的一些不便之处包括：

1. **默认清空 Cookie**：每次都需要重新登录。解决方案是将常用网站添加到例外列表中（LibreWolf 在这方面做得比强化版 Firefox 更好，可以直接在地址栏旁边添加，而无需进入设置中手动输入网址）。
2. **启用了 Resist Fingerprinting (RFP)**：窗口size变小了。对于这个问题，[LibreWolf 官网建议用户开启RFP，并适应 RFP 的常见缺点](https://librewolf.net/docs/faq/#what-are-the-most-common-downsides-of-rfp-resist-fingerprinting)。

目前遇到的就是这些问题，如果以后有新的发现，我会继续更新。

## Reference

1. [LibreWolf Docs](https://librewolf.net/docs/faq/)
2. [yokoffing/Betterfox](https://github.com/yokoffing/Betterfox)
3. [arkenfox/user.js](https://github.com/arkenfox/user.js)
3. [ The ULTIMATE Browser Tier List (Based Tier to Spyware Tier) ](https://www.youtube.com/watch?v=j5r6jFE8gic)
4. [ The Ultimate Guide to Firefox Hardening! ](https://www.youtube.com/watch?v=F7-bW2y6lcI)
