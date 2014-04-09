---
layout: post
title: Love2D中的摄像机 第一部分：基础
category: 文章翻译
tags: [love2d, lua, 游戏开发]
excerpt: >
  这是几篇关于如何在 LÖVE引擎中创建摄像机的博客中的第一篇。本部分将会设计创建一个摄像机的基本原理。第二部分会涉及视差卷动和创建图层...
---
原文：[Cameras in Love2D Part 1: The Basics](http://nova-fusion.com/2011/04/19/cameras-in-love2d-part-1-the-basics/) ，请自备梯子。

这是几篇关于如何在 LÖVE引擎中创建摄像机的博客中的第一篇。本部分将会设计创建一个摄像机的基本原理。[第二部分]({% post_url 2013-06-04-cameras-in-love2d-part-2-parallax-scrolling %})会涉及视差卷动和创建图层。那么，现在开始！

更新：我已经完成[第三部分]({% post_url 2013-06-04-cameras-in-love2d-part-3-movement-bounds %})的写作，包含了如何限定摄像机的活动范围。

## 函数

我们所需要的函数是以下这些：


* [love.graphics.pop](http://love2d.org/wiki/love.graphics.pop)
* [love.graphics.push](http://love2d.org/wiki/love.graphics.push)
* [love.graphics.translate](http://love2d.org/wiki/love.graphics.translate)
* [love.graphics.rotate](http://love2d.org/wiki/love.graphics.rotate)
* [love.graphics.scale](http://love2d.org/wiki/love.graphics.scale)

Love2D（我从现在开始会使用这个名字，因为它打字更容易）有一个变换栈，在一个栈中有一些独立的变换信息。`love.graphics.push`可以将当前的变换压入栈顶，然后在它下面插入一个新的变换，可以这么说。`love.graphics.pop`丢弃当前的变换，然后设置当前的变换到栈上那个之前的变换。让我们来看一些ASCII绘图：

```
Starting:
|-----------|
| Default   |
|-----------|

love.graphics.push:
|-----------|
| Default   |       ^ pushed up
|-----------|
|-----------|
| New       |       <- pushed under
|-----------|

love.graphics.pop:
|-----------|         |-----------|
| Default   |         | New       |  -> discarded
|-----------|         |-----------|
```

现在，`translate`， `scale`，和 `rotate` 改变了当前变换中的信息。传给 `translate` 的 `x` 和 `y` 参数会加到每个绘制到屏幕的 `x` 和 `y` 座标中。每个 `x` 和 `y` 座标会乘上传给 `scale` 的值。最后，每个给定的位置（一个x/y向量）会被传给 `rotate` 的值转动，屏幕的左上角作为旋转中心。

有了上面的解释，你可以像这样创建一个最简单的摄像机：

``` lua
function love.draw()
  love.graphics.translate(-x, -y)
  -- drawend
```

这里 `translate` 按照默认的变换进行操作。变换栈每帧都会重置，所以我们必须每一帧都要应用我们的改变。我们可以这样来实现一个更复杂些的摄像机：

``` lua
function love.draw()
  love.graphics.rotate(-rotation) -- this is in radians
  love.graphics.scale(1 / zoom, 1 / zoom)
  love.graphics.translate(-x, -y)
end
```

`-rotation` 和 `-x`，`-y`让 所有的位置“留在原处”，如果它们不随摄像机一起移动的话。传给`scale` 的值小于1使摄像机拉近，大于1的值使摄像机远离。

## camera 模块

让我们把这些组织起来做成一个camera 模块。这是模块的代码：

``` lua
camera = {}
camera.x = 0
camera.y = 0
camera.scaleX = 1
camera.scaleY = 1
camera.rotation = 0

function camera:set()
  love.graphics.push()
  love.graphics.rotate(-self.rotation)
  love.graphics.scale(1 / self.scaleX, 1 / self.scaleY)
  love.graphics.translate(-self.x, -self.y)
end

function camera:unset()
  love.graphics.pop()
end

function camera:move(dx, dy)
  self.x = self.x + (dx or 0)
  self.y = self.y + (dy or 0)
end

function camera:rotate(dr)
  self.rotation = self.rotation + dr
end

function camera:scale(sx, sy)
  sx = sx or 0
  self.scaleX = self.scaleX * sx
  self.scaleY = self.scaleY * (sy or sx)
end

function camera:setPosition(x, y)
  self.x = x or self.x
  self.y = y or self.y
end

function camera:setScale(sx, sy)
  self.scaleX = sx or self.scaleX
  self.scaleY = sy or self.scaleY
end
```

`camera` 模块是一个 table，包含了它需要被定位的信息。关键函数是 `set` 和 `unset`。`set` 应用 `camera` 模块中的信息并且为它创建一个新的变换（为了支持在一帧中有多个视图）。`unset` 只是使用 `pop` 移除最近的变换。剩下的函数只是简单的帮助调整 `camera` 中的属性。

你可以把这段代码放在 `love.draw` 中来使用这个摄像机：

``` lua
function love.draw()
  camera:set()
  -- draw stuff
  camera:unset()
end
```

然后在更新阶段你可以改变摄像机的位置，缩放，或者旋转。

``` lua
camera.x = 50
camera:scale(3) -- zoom by 3
```

## 鼠标

现在，如果你想在游戏世界中用鼠标来进行交互的话，你还需要使用 `love.mouse.getX/Y/Position()` 的返回值来调整座标。你可以重新定义这些方法，或者给 `camera` 模块添加一些新的函数。

``` lua
function camera:mousePosition()
  return love.mouse.getX() * self.scaleX + self.x, love.mouse.getY() * self.scaleY + self.y
end
```

它会根据鼠标的座标计算缩放和移动。如果你想把转动也计算在内，你需要做一些向量转动的计算。想了解这方面的东西，就去看 [HUMP的vector类](https://github.com/vrld/hump/blob/master/vector.lua#L127)。

### 总结

那么感谢你的阅读。我希望你学到了一些东西，请将你的想法写在评论框中告诉我。我们第二部分见！
