---
layout: post
title: Love2D中的摄像机 第二部分：视差卷动
category: 文章翻译
tags: [love2d, lua, 游戏开发]
excerpt: >
  在第一部分中，我们创建了一个基本的摄像机。现在我们将要扩展它，添加一些视差卷动。注意我用的这些方法可能不是最漂亮的，毕竟这只是我在半小时内想出来的。如论如何，这对你来说将是个开发你自己系统的一个起点...
---

原文：[Cameras in Love2D Part 2: Parallax Scrolling](http://nova-fusion.com/2011/04/22/cameras-in-love2d-part-2-parallax-scrolling/)，请自备梯子。

在[第一部分]({% post_url  2013-06-04-cameras-in-love2d-part-1-the-basics%})中，我们创建了一个基本的摄像机。现在我们将要扩展它，添加一些视差卷动。注意我用的这些方法可能不是最漂亮的，毕竟这只是我在半小时内想出来的。如论如何，这对你来说将是个开发你自己系统的一个起点。

## 这是什么？

现在，有些人可能还不知道，什么是视差卷动？这是一种获取伪3D效果的方法，并且所有的图像和游戏性还是基于2D的。它目前广泛用于2D游戏中，因为它加入了通常不存在的深度感。

这个我们将要用来实现这个效果的技术称为图层法。我们所要做的就是渲染几个图层的图像，它们在摄像机运动的时候受到的影响（移动）比例不同。通常游戏场地（玩家所处的地方，也是所有动作发生的地方）将会按照1:1的比例受到影响，比游戏场地“靠近”摄像机的图层会比摄像机的移动速度要快。比游戏场地“远离”摄像机的图层将会比摄像机的移动速度要慢。

## 实现

好了，闲话少说，现在开始进行实现。为了实现视差卷动，我们将要使用一个粗糙的图层系统，它由摄像机管理。这里是对我们最初制作的那个 `camera`  的修改版。首先把这个添加到顶部，在设置完其他值如 `x` 和 `y` 之后。

``` lua
camera.layers = {}
```

作用很明显，保持你的图层。接下来添加这个方法：

``` lua
function camera:newLayer(scale, func)
  table.insert(self.layers, { draw = func, scale = scale })
  table.sort(self.layers, function(a, b) return a.scale < b.scale end)
end
```
这样我们就创建了新的图层。`scale` 是在绘制图层之前摄像机的 `x` 和 `y` 位置要乘上的值。`func` 是要在图层上绘制物体的函数。之后我们还要根据 `scale` 对图层排序，这样较小的 `scale` 会首先绘制；也就是说，图层越远，就越早被绘制。

接下来添加这个方法：

``` lua
function camera:draw()
  local bx, by = self.x, self.y
  
  for _, v in ipairs(self.layers) do
    self.x = bx * v.scale
    self.y = by * v.scale
    camera:set()
    v.draw()
    camera:unset()
  end
 end
```

这个函数负责绘制所有图层。`bx` 和 `by` 是“基本（base）”的 x/y 位置，在摄像机的初始位置。在调用 `camera:set` 和绘制函数之前，我们将摄像机的位置乘上图层的 `scale` 。然后我们进行通常的绘制步骤。

那么一切就绪之后，我们的 `love.draw` 定义将变成：

``` lua
function love.draw()
  camera:draw()
end
```

## 一个简单的例子

我做了一个简单的例子来展示这个视差系统。这是它的代码：

``` lua
require('camera')

math.randomseed(os.time())math.random()math.random()math.random()

function love.load(args)
  camera.layers = {}
  
  for i = .5, 3, .5 do
    local rectangles = {}
    
    for j = 1, math.random(2, 15) do
      table.insert(rectangles, {
        math.random(0, 1600),
        math.random(0, 1600),
        math.random(50, 400),
        math.random(50, 400),
        color = { math.random(0, 255), math.random(0, 255), math.random(0, 255) }
      })
    end
    
    camera:newLayer(i, function()
      for _, v in ipairs(rectangles) do
        love.graphics.setColor(v.color)
        love.graphics.rectangle('fill', unpack(v))
        love.graphics.setColor(255, 255, 255)
      end
    end)
  end
end

function love.update(dt)
  camera:setPosition(love.mouse.getX() * 2, love.mouse.getY() * 2)
end

function love.draw()
  camera:draw()
  love.graphics.print("FPS: " .. love.timer.getFPS(), 2, 2)
end

function love.keypressed(key, unicode)
  if key == ' ' then
    love.load()
  elseif key == 'escape' then
    love.event.push('q')
  end
end
```

这段代码所做的事情就是创建6个图层， `scale` 从.5开始，增长到3。 `rectangles` 这个 table 保存了随机数量的矩形绘制在当前图层上。我们不需要把它定义成全局变量，因为唯一使用这个 table 的函数是在当前的局部作用域中创建的。这个函数只是根据每个矩形的 table 中包含的信息在每个图层进行绘制。

你可以点击空格键重置所有东西。这会调用 `love.load` ，并且这也就是为什么我们重建 `camera.layers`  这个 table 的原因。你可以点击 esc 键退出。


还最后一件事需要注意，在顶部我们调用了 `math.random` 三次，因为之前有报告说前三次还是几次当你调用 `math.random` 得到的并不是一个非常随机的数。我自己倒是没遇到过这种事情，但是为了以防万一事先调用几次也没什么大不了的。

## 总结

好了，实现了一个基本的视差卷动。从现在开始你可以继续为你自己的项目创造一些更漂亮的设计了。如果你有任何想法或建议，非常欢迎你留下评论。

感谢阅读！
