---
layout: post
title: Love2D中的摄像机 第三部分：活动边界
category: 文章翻译
tags: [love2d, lua, 游戏开发]
excerpt: >
  因为在本系列的第三部分中有些有趣的东西，我就把它写出来了，而在这部分里我们会涉及到创建摄像机无法越过的界限。确保在继续阅读之前你已经读过了第一部分和第二部分...
---
原文：[Cameras in Love2D Part 3: Movement Bounds](http://nova-fusion.com/2011/05/09/cameras-in-love2d-part-3-movement-bounds/)，请自备梯子。

因为在本系列的第三部分中有些有趣的东西，我就把它写出来了，而在这部分里我们会涉及到创建摄像机无法越过的界限。确保在继续阅读之前你已经读过了[第一部分]({% post_url  2013-06-04-cameras-in-love2d-part-1-the-basics%})和[第二部分]({% post_url 2013-06-04-cameras-in-love2d-part-2-parallax-scrolling %})。

可能你不知道我说的是什么意思，我的意思是把摄像机的活动约束在一个“盒子”里。也就是，对摄像机来说，有一个最大和最小的x/y座标值。例如你在跟随游戏角色，当游戏角色来到边缘时你并不想摄像机呈现给你任何关卡之外的东西（一般都是黑色区域），这很容易理解。当然，活动边界可以远比一个简单的矩形框复杂，你可以把它约束在一条路径上或是类似的什么上，但是我们这里还是要保持简单。

### 实现

我们会通过限制摄像机的x/y座标到给定的最大和最小x/y值（边界）来实现。我们使用一个小巧便捷的夹具函数来做这个，这个函数我命名为 `math.clamp`：

``` lua
function math.clamp(x, min, max)
  return x < min and min or (x > max and max or x)
end
```

不过这是一个非常紧凑的版本，如果你对Lua的一些精妙之处不是很熟的话，你可能倾向于用这个更清晰的版本：

``` lua
function math.clamp(x, min, max)
  if x < min then
    return min
  elseif x > max then
    return max
  else
    return x
  end
end
```
解决掉这个问题之后，我们还需要找一个方法来约束x/y的值。我们可以只在它们变化的时候用一个设置器（setter）来做这个事，或者我们可以每一帧都在 `camera:set` 中限制它们。后面这个方法效能太低，所以我们用前者。因为新的模块非常大，我会把代码贴在Gist 中，点击链接阅读 `camera.lua` 来理解它。注意需要在某个地方定义 `math.clamp` 。

**[摄像机模块的代码](https://gist.github.com/961685#file_camera.lua)**

我们和第一部分的 `camera` 模块对比一下看改变了哪些地方： 

* x/y的值现在储存在 `camera._x`/`camera._y` 中而不是 `camera.x`/`camera.y` 。 在 `camera` 模块中引用这些变量的方法也相应改变。
* 我们限制使用 `setX`/`setY` 来设置 `x/y` 位置。
* `setPosition` 现在只是调用 `setX` 和 `setY`。
* 添加了 `getBounds`/`setBounds` 方法。


现在，我更倾向于用 metatable 的方式来写设置器，但是这可能只会增加无谓的麻烦；如果你想，可以把 camera 改写成那种方式。

总之，这东西是如何工作的？好吧，`setX` 和 `setY` 检查是不是存在 `camera._bound`，如果有，就使用各自的值来确保摄像机的 x/y 位置不会超过给定的最大值。我们可以用 `setBounds` 来设置边界，这将会创建 `camera._bounds` 这个table。

## 一个例子

解决了这些问题之后，我会给你一个简单的例子来演示如何使用。在我放 `main.lua` 的同一个 Gist 中就包含了这段示例代码。

**[示例代码](https://gist.github.com/961685#file_main.lua)**

`love.load` 建立了一些玩家要用的table，围墙（只是一个方框），和一些随机的白色方块（用来能看出你在移动）。`love.update` 干了些什么就非常显而易见了，允许玩家通过方向键来移动并且把玩家定位在摄像机的中心。`love.draw` 不过是将包含在这些table中的信息画出来而已。

那么作为最重要的部分。在 `love.load` 中我们调用了 `setBounds` 传入了一个最小 x/y 位置 0, 0，还有将屏幕的宽和高作为最大值。 0, 0 来自于方框的 x/y 位置，但是 `width` 和 `height` 是哪来的？好吧，方框的右下角是 `width * 2`, `height * 2`。我们想让摄像机最远能到右下角，而且因为摄像机的“宽”和“高”和屏幕的宽和高是相等的， x/y 的最大值等于 `width * 2 - width`, `height * 2 - height`，或者 `boxWidth - cameraWidth`, `boxHeight - cameraHeight`，也就当然等于 `width`, `height`。

## 总结

好了，所有内容都讲完了。我希望你学到了一些东西，另外感谢阅读。如果你有任何想法、评论、建议，请在下方的评论框中写下来。还有，记得摄像机模块的示例代码在 [gist #961685](https://gist.github.com/961685)。
