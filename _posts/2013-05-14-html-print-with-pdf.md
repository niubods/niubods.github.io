---
layout: post
title: 尝试输出PDF实现网页套打
category: 研究备忘
tags: [HTML套打, iText, PDF输出]
excerpt: >
  以前在做项目的时候遇到过这个问题，在网页中实现套打。很奇怪的是居然没有很完美的解决方案。常用的是以ActiveX控件的方式实现套打，缺点很明显，你必须使用IE并且你需要下载安装一个个头似乎不怎么小的ActiveX控件，而且这东西技术控制在别人手里基本上你必须付费才能使用...
---

以前在做项目的时候遇到过这个问题，在网页中实现套打。很奇怪的是居然没有很完美的解决方案。常用的是以ActiveX控件的方式实现套打，缺点很明显，你必须使用IE并且你需要下载安装一个个头似乎不怎么小的ActiveX控件，而且这东西技术控制在别人手里基本上你必须付费才能使用。

网上搜到的一篇分析目前可用套打方案的一篇文章明显是篇软文，无非是想推广他的”轻量级“ActiveX套打控件罢了。当时我发现一个比较可行的方法是用一个带`media="print"`属性的css来设置打印样式。鬼使神差的还找到一个叫”blueprint“的开源项目，因为有个print就天真地认为可以帮助我设置打印样式。后来发现这不过是个用来辅助网格布局的css框架罢了，但是无论如何，借助网格布局我硬是把一个单证套打的功能做出来了。反复调整行间距和单元格之间的距离，然后调整打印时上下左右各个页边距，然后就真的可以实现套打了，而且定位还蛮精准。

当然这种方式问题多多。首先它对打印条件要求很苛刻，页边距需要精确设置，可能还只对一个打印机型号甚至一台打印机有效。其次它不够灵活，网格布局受限水平方向上单元格的数量，很难精确对齐要打印的空白位置。最后是部署太繁琐，只能到现场安装调试。当时我疯狂迷恋的chrome浏览器甚至连打印参数设置都没有，无法设置页边距的话只能用IE。总之，这个解决方法太过儿戏。

这段时间试用了几个基于google drive的chrome应用，很多都提供打印功能，从打印预览来看排版上很接近印刷品，于是冒出个想法来，是不是可以在服务端生成一个pdf文件来打印。因为pdf本身就很适合用来打印，并且套打定位要比css容易控制。

java的pdf库比较流行的是iText。虽然免费版不能用于商业使用，不过在天朝许可方面的问题根本不算问题。简单使用了一下，很容易就生成了一个pdf文件，当然我们并不需要在服务器硬盘上保存这个文件，以字节流的形式发给浏览器就可以了。

下面的问题是如何在浏览器端将这个pdf打印出来。网上有解决方法说用object标签，设置一些参数，然后调用它的打印方法。但是我在chrome上试了不行，这个object根本没有所说的那个打印方法。开始以为是IE的专有方法，但是我在IE9上试验了一下也没有成功。另一个方法是使用iframe。在页面中加载一个看不见的iframe，对这个iframe中的window对象调用print方法。chrome下可以顺利弹出打印预览对话框，但IE会直接弹出Adobe Reader打开这个pdf文件，并且需要再点一下Adobe Reader的打印按钮。这不好，在IE下还要多一步操作。

后来发现pdf中可以嵌入javascript。在pdf中加入一句`this.print()`就可以在pdf加载完成之后自动提示打印了。IE下虽然仍是打开Adobe Reader，但是马上就会提示打印。`this.print()`有一个布尔类型的参数，`true`代表静默打印，也就是不弹打印预览对话框，`false`则会弹出打印预览对话框。chrome下参数无影响，IE下即使选择静默打印仍会弹出一个确认对话框问你是否允许自动打印……挺烦人的，还不如弹打印预览的好。

下面的问题就是解决生成pdf时要打印的元素在页面上定位的问题了，看看iText的文档，应该不难。而且好像pdf还可以有个模板层，也就是可以拿空白单据的图片做模板层，显示打印后的实际效果，但只输出套打内容。嗯，这是个很有用的特性。