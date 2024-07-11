---
title: "记一次将SoftEther部署Let's Encrypt的过程"
description: 要是我学习过程也有这么流畅就好了
date: 2024-05-22T09:10:41Z
image: 
math: 
license: CC BY-NC-SA 4.0
hidden: false
comments: true
draft: false
categories:
    - Work
---
## 部署起因
众所周知，我是一个怪人；毕竟 SoftEther 安装之后默认行为是不认证SSL证书，而且就算部署也不一定真的能用上。但是我还是想部署，主要是 ~~\(闲得慌\)~~ 想说好不容易买了个域名 ~~\(就不说我域名跑到帮我付钱的群友身上的事情了\)~~ 不上证书怎么行。但是Porkbun自带的Wildcard Certificates需要每三个月手动部署一次。~~\(太麻烦了\)~~ 实质上也是Let's Encrypt 签发的证书。所以干脆用Certbot DNS Challenge直接签发，还能用systemd的Timer自动触发证书更新。何乐而不为？
## 部署过程
我走的路线是用 Certbot 的 [snap](https://certbot.eff.org/instructions?ws=other&os=snap) 包以及 infinityofspace 开发的 [snap Plugin](https://github.com/infinityofspace/certbot_dns_porkbun) ~~\(口口声声说不喜欢 snap 实际上真到想偷懒的时候还是直接上了 snap 我感觉我有点傲娇了\)~~。然后就是不停试错，记得证书文件夹给个755权限，免得vpncmd读不到（我也不知道为啥最开始死活读不到，给了755就行了。但是执行者是root啊，怪欸。）
## 部署结果和工具
还是很顺利地部署上了~~要是没有就不会有这篇 blog 了~~ 这次所用到的脚本已经使用Unlicense license开源在[GitHub](https://github.com/Raspberry-Monster/SoftEther-LetsEncrypt-Script)，欢迎各位大佬前来参观和批评指正