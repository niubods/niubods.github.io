---
layout: post
title: Play的HTTP数据封装
category: 研究备忘
tags: [play框架, 数据封装]
excerpt: >
  Play的HTTP数据封装机制异常强大和灵活，官方文档中给出的例子已经足够应付一般需要。但是作为一个考究癖来说仅仅应付一般的需要是远远不够的，另外确实在实际项目中也需要了一些棘手的应用场景...
---

Play的HTTP数据封装机制异常强大和灵活，官方文档中给出的例子已经足够应付一般需要。但是作为一个考究癖来说仅仅应付一般的需要是远远不够的，另外确实在实际项目中也需要了一些棘手的应用场景。

首先，关于HTTP数据的直接获取、自动类型转换和高级模型封装这些官方文档中已经讲得很详细了，这里就不在赘述。

下面开始正题。在Play中，除了自动手动获取参数和依赖自动封装之外，还有一种“半自动”的封装方式。模型对象提供了一个`edit()`方法，方法签名是：

``` java
<T extends GenericModel> T edit(ParamNode rootParamNode, java.lang.String name)
```

第一个参数可以使用`params.getRootParamNode`获取，但是第二个参数`name`又是个啥？

其实这个`name`是在表单数据的名字中表示对象名的那部分。例如在一个表单里，有一个文本框，`name`属性为“dog.name”，如果用过自动模型封装的话，对这样的名字应该是很熟悉的，那么这里的`name`就是“dog”。

这个半自动封装的过程就是先创建一个模型对象，然后调用它的edit方法，这样就可以把表单数据封装进模型对象了。

模型对象还有一个已经过时的edit重载方法，这个没有研究。另外模型类还提供了两个相应的静态方法，其中一个也已经过时，另一个多了两个参数，一个是要封装表单数据的对象，另外一个是注解对象数组。

还有一个问题，对于像我这样的懒人，既想使用模型封装，在表单元素的name中又懒得写对象名，这样也可以吗？是的，在Play中，这样也完全没问题。下面请出Play的数据封装辅助类Binder，这里用到的是他的bindBean方法。

bindBean有两个重载方法，其中`public static void bindBean(RootParamNode rootParamNode, java.lang.String name, java.lang.Object bean)`方法和模型对象的`edit()`方法类似，也需要`name`字符串，同样，将`name`设为空字符串也不能实现我们想要的效果。

这里我们要使用另一个`public static void bindBean(ParamNode paramNode, java.lang.Object bean, java.lang.annotation.Annotation[] annotations) throws java.lang.Exception` 。第三个参数注解数组，暂时不知道要怎么用，目前传入一个null值就可以。这个方法可以将直接以属性名命名的HTTP请求参数封装到指定对象中。这么宽泛的限制可能会产生一些无法匹配的问题，所以这个方法会抛异常。


