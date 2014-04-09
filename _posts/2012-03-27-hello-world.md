---
layout: post
title: "测试页面"
category: 测试
tags: [helloworld, markdown, showcase]
excerpt: >
  使用Jekyll搭建了一个个人博客，用Markdown语法（实际上是GitHub方言版）写博客。初步尝试了一下，感觉还行。能部署自己的博客系统感觉很不错。下面是一些测试用的语法示例：
---

使用Jekyll搭建了一个个人博客，用Markdown语法（实际上是GitHub方言版GFM）写博客。初步尝试了一下，感觉还行。能部署自己的博客系统感觉很不错。下面是一些测试用的语法示例：

标题一
================

标题二
----------------

### 标题三

#### 标题四

##### 标题五

###### 标题六

有序和无序列表
------------

**有序列表:**

1. 项目1
1. 项目2
1. 项目3

**无序列表：**

* 项目1
  + 项目1-1
    - 项目1-1-1
* 项目2
* 项目3

加粗和斜体
-----------

这个应该是**粗体**,这是_斜体_可这个又是***什么***?

超链接和图片
------------

从[Jekyll的官方网站](http://jekyllrb.com/) 可以查到一些很有很有帮助的文档。
但是[JekyllBootstrap](http://jekyllbootstrap.com/) 虽然在Jekyll基础上做了很多工作但是官方网站样式并不漂亮，并且JekyllBootstrap本身也貌似很久没更新了的样子。

王国之心的图片，非常漂亮。

![王国之心](https://lh6.googleusercontent.com/-bLnw-E0z_SA/T2vlXSehfMI/AAAAAAAAAW4/K14Rb-6K5YQ/s620/2-1.jpg)

小米手机拍照的原始图片，分辨率比较高

![两只小猫](https://www.filepicker.io/api/file/RD4g9YWgQ4CiUCwnvlr4 "两只小猫")

代码高亮
-------------------

相对比较冷门的Lua也是支持的

``` lua
-- Does a simple tokenization of html data. Returns the data as a list of tokens. 
-- Each token is a table with a type field (which is either "tag" or "text") and
-- a text field (which contains the original token data).
function tokenize_html(html)
  local tokens = {}
    local pos = 1
      while true do
    local start = find_first(html, {"<!%-%-", "<[a-z/!$]", "<%?"}, pos)
    if not start then
      table.insert(tokens, {type="text", text=html:sub(pos)})
      break
    end
    if start ~= pos then table.insert(tokens, {type="text", text = html:sub(pos, start-1)}) end

    local _, stop
    if html:match("^<!%-%-", start) then
      _,stop = html:find("%-%->", start)
    elseif html:match("^<%?", start) then
      _,stop = html:find("?>", start)
    else
      _,stop = html:find("%b<>", start)
    end
    if not stop then
      -- error("Could not match html tag " .. html:sub(start,start+30)) 
      table.insert(tokens, {type="text", text=html:sub(start, start)})
      pos = start + 1
    else
      table.insert(tokens, {type="tag", text=html:sub(start, stop)})
      pos = stop + 1
    end
  end
  return tokens
end
```

通常做法是用Liquit的`highlight`标签，但是使用redcarpet渲染markdown时也可以用GFM的三个反引号，唯一的不同就是用这种方法没办法加入行号。

另外Jekyll还支持用Liquit标签嵌入一段gist

{% gist 5539127 main.lua %}
