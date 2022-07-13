---
title: 实现一个antd-form
date: 2022-07-13 16:53:47
sidebar: auto
categories:
  - 框架
tags:
  - 造轮子
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
permalink: /pages/fd6620/
---

源码地址：[https://github.com/gezhicui/mini-rc-field-form](https://github.com/gezhicui/mini-rc-field-form)

## 实现一个 antd-form

在前端学习的入门阶段，使用组件库的过程中，我总是觉得能写出这种东西的人真的太厉害了，但是随着深入学习，我发现之所以觉得厉害其实是因为你不知道他是怎么实现的，人总是对未知的东西感到神秘，当了解了他的实现方式时，你就会茅塞顿开

<!-- more -->

`antd-form`是基于`rc-field-form`实现的，所以本文从 0 实现一个最简单的`rc-field-form`，是为了了解核心原理

`antd-form`表单的开发有三个最重要的组成部分，分别是

- useForm
- Form
- Form.Item

其中，`Form.Item`是由`rc-field-form`中的`Field`封装而来，所以下文中我提到的`Field`其实就是`Form.Item`

## useForm

先来写一个表单的最简使用：

```js
const [form] = useForm();

return (
  <Form
    form={form}
    onFinish={(values: any) => {
      console.log('onFinish执行，值为：', values);
    }}
  >
    <Field name="nickname">
      <Input placeholder="请输入昵称" />
    </Field>
    <Field name="doing">
      <Input placeholder="请输入在做的事" />
    </Field>
  </Form>
);
```

我们可以看到，表单在使用之前通过`useForm`拿到了`form`这个东西，`form`被传入到`Form`组件中，那这个`form`到底是个啥？

我们把这个`form`打印出来:

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220712171308.png)

发现他是一个挂载了许多方法的对象实例，`useForm`的实现如下：

```js
const useForm = (form) => {
  // 创建一个ref保存表单
  const formRef = useRef();
  // 防止表单重复创建
  if (!formRef.current) {
    // 如果有传参,ref为传进来的表单，否则创建表单实例对象
    if (form) {
      formRef.current = form;
    } else {
      const formStore = new FormStore();
      formRef.current = formStore.getForm();
    }
  }
  // 返回数组 方便扩展
  return [formRef.current];
};
```

在`useForm`的实现中，其实暴露出去的`form`就是`FormStore`这个类的实例上的`getForm`方法,`getForm`方法获取了类的所有可访问属性，这个`FormStore`是表单的核心所在，是保存表单所有状态和处理表单操作的中心。他的基本结构是这样的：

```js
class FormStore {
  // 用来保存表单数据
  private store: Store = {};

  //...各种方法,主要是对表单数据的操作

  getForm = () => {
    return {
     // 暴露出去供外界使用的各种方法
    };
  };
}
```

总结一下`useForm`中的操作就是 new 了一个`FormStore`对象，获取到了`FormStore`对象实例中所有能供外界访问的数据和方法。

## Form

`Form`就是表单组件了，`form`实例被传到`Form`中,进行了以下操作:

```js
// 由于可能没有传入form，所以这里useForm执行一下
const [formInstance] = useForm(form);
// 从form实例中拿到设置用户自定义方法和设置初始值的方法
const { setCallbacks, setInitialValues } = formInstance;
```

`setCallbacks, setInitialValues`这两个方法在`FormStore`中实现如下：

```diff
class FormStore {
// 使用callbacks保存用户自定义方法
+  private callbacks = {};
// 使用initialValues保存初始值，后面做表单重置会用到
+  private initialValues = {};


+  setCallbacks = (callbacks) => {
+    this.callbacks = callbacks;
+  };

+  setInitialValues = (initialValues) => {
+    this.initialValues = initialValues || {};
+    this.setFieldsValue(initialValues);
+  };

  getForm = () => {
    return {
+     setCallbacks,
+     setInitialValues
    };
  };
}
```

拿到了`setCallbacks`和 `setInitialValues`,就要使用这两个函数了

```js
// 获取到传入Form的表单事件处理方法
const { onFinish, onFinishFailed, onValuesChange, initialValues } = props;

// setCallbacks保存用户自定义回调函数
setCallbacks({
  onFinish,
  onFinishFailed,
  onValuesChange,
});

// 设置表单初始值
setInitialValues(initialValues || {});
```

这时候 就在`Form`实例中初始化了数据和用户自定义方法，接下来返回一个组件，**通过`context`把`form`实例透传下去**,这样`Field`组件中也可以获取`form`实例

```js
const FieldContext = React.createContext();

return (
  <form
    {...restProps}
    onSubmit={(event) => {
      event.preventDefault();
      event.stopPropagation();
      formInstance.submit();
    }}
  >
    {/* 把form对象实例透传下去 */}
    <FieldContext.Provider value={formInstance}>
      {children}
    </FieldContext.Provider>
  </form>
);
```

这样,最简单的`Form`组件就完成了

# Field

`Field`是`Form`的子组件，在`antd`中是`Form.Item`,它的基本使用如下：
