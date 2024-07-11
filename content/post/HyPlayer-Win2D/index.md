---
title: "HyPlayer 的 Win2D 歌词面板的后期改进"
description: Win2D, 你啥时候可以修下Geometry重叠时错误渲染的问题
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
## 起因
HyPlayer在24年的时候引入了 Win2D 作为新的歌词面板, 之前主力后端开发者写了一篇[知乎文章](https://zhuanlan.zhihu.com/p/642338137)做介绍。所以这边就先不讲大部分的初期的实现细节。
那讲啥呢？ 众所周知，在后期版本，为了 ~~高度自定义~~ 界面样式来 ~~压榨~~ Win2D 的潜力，开发团队引入了字体自定义功能。问题也就出在这里。发版之后的那天晚上，我们用户群里就有人反馈，部分字体会导致高亮出错。具体表现就像是[这篇Issue](https://github.com/microsoft/Win2D/issues/781)；唯一的不同是，这不是字与字之间的重叠问题。后经过确认，是在特定字体下，字体内部存在重叠，导致在上文提到的 Geometry 取交集的时候无法正确进行计算。重叠的部分于是出现了空洞。
## 过程
这就比较麻烦了 不是吗？ 主要是因为高亮需要找到对应位置才能进行，不能硬做。但是Geometry肯定是不能作为后续方案了。幸运的是，我在查文档的时候看到了lindexi老师关于 CanvasActiveLayer 的[文章](https://www.cnblogs.com/lindexi/p/12085807.html)。
在讲解决方案前，我们要先回忆一下刚刚的内容。歌词高亮/渐变的核心原理都是根据字生成一个Rect，作为高亮区域； 然后将生成的Rect和由字生成的 Geometry 取交集，然后再次绘制到画布上。这时候你应该已经想到了对应解决方案了，既然部分字体会导致 Geometry 在取交集的时候略过重叠部分导致字中出现空洞；那么，我们多次绘制字体，然后利用 CanvasActiveLayer 进行裁切，便可以达成同等的效果。（仍然是进行多次绘制，只不过一个绘制的是 Geometry ，另一个是 TextLayout。 Bug也就迎刃而解了。后来请教蓝火火，蓝火火说 Geometry 取交集是使用 CPU 进行计算，而 CanvasActiveLayer 是利用 GPU 的分层渲染；理论上性能会更高，但是我这里没有办法定量进行测试。于是这个只能先打一个问号。
## 结果
这次的修复已经发到了 [GitHub](https://github.com/HyPlayer/HyPlayer/commit/8879fd875f3aa07a073dc848fc041fb1d1b530a8) 上，只是因为最近一段时间没办法更新Blog，所以到现在才汇报。Blog到这里就结束了，Bye!