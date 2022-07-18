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

```js
<Form form={form}>
  <Field
    name="nickname"
    rules={[
      {
        required: true,
        message: '昵称必填',
      },
    ]}
  >
    <Input placeholder="请输入昵称" />
  </Field>
</Form>
```

基本使用一看就会，和 antd 的 form.item 一模一样，直接来看`Field`的实现

组件加载的时候，做了以下三件事：

- 组件获取了`context`的内容（即 form 实例），同时从 form 实例上拿到了`registerField`方法，在`componentDidMount`执行了该方法,并把当前组件传给`registerField`当做实参，返回的是组件卸载的处理方法，在组件卸载时调用该方法来删除`form`实例中该控件绑定的状态
- 定义了一个`onStoreChange`方法，可以在控件值发生改变的时候使用，来刷新数据
- 定义了一个`validateRules`方法，用来做该控件的字段校验

```js
componentDidMount() {
  const { registerField } = this.context;
  this.cancelRegister = registerField(this);
}

componentWillUnmount() {
  this.cancelRegister && this.cancelRegister();
}

onStoreChange = () => {
  //值改变调用react的forceUpdate重新render，因为数据不是响应式的
  this.forceUpdate();
};

/*
  该方法提供给FormStore消费，当前组件先被存入fieldEntities这个保存所有Field的数组中，
  等表单校验时，循环数组拿出组件，执行组件的validateRules进行校验
*/
validateRules = () => {
    const { rules, name } = this.props;
    if (!name || !rules || !rules.length) return [];
    const cloneRule = [...rules];
    const { getFieldValue } = this.context;
    const value = getFieldValue(name);

    // validateRules是表单的校验方法，具体看源码
    const promise = validateRules(name, value, cloneRule);

    promise
      .catch(e => e)
      .then(() => {
        if (this.validatePromise === promise) {
          this.validatePromise = null;
          this.onStoreChange();
        }
      });

    return promise;
  };
```

`registerField`方法主要是初始化处理当前`Field`组件的内容，并把当前组件存进`fieldEntities`数组中，方法的实现如下：

```js
class FormStore {
+  private fieldEntities = [];

+  registerField = (entity) => {
     // 用fieldEntities保存页面上的所有Field组件
     this.fieldEntities.push(entity);
     const { name, initialValue } = entity.props;
     // 初始化Field传入的initialValue
     if (initialValue !== undefined && name) {
       this.initialValues = {
         ...this.initialValues,
         [name]: initialValue,
       };
       // store中添加控件的初始化值
       this.setFieldsValue({
         ...this.store,
         [name]: initialValue,
       });
     }
     // 返回一个函数，当组件卸载时调用该函数，移除改组件的所有状态
     return () => {
       this.fieldEntities = this.fieldEntities.filter(item => item !== entity);
       // this.store = setValue(this.store, namePath, undefined); // 删除移除字段的值
     };
   };

  getForm = () => {
    return {
+     registerField,
    };
  };
}
```

Field 返回的 jsx 是这样的：

```js
render() {
  const { children } = this.props;
  // 为form.item附加上form中的属性
  const returnChildNode = React.cloneElement(
    children,
    this.getControled(),
  );
  return returnChildNode;
}
```

可以看到，`Field`主要干的事情就是把他的子组件，即真正的输入控件使用`React.cloneElement()`拿来拓展

`React.cloneElement`接收三个参数

- type (ReactElement)
- props (object)
- children (ReactElement)

他的作用是克隆并返回一个新的 `ReactElement` （内部子元素也会跟着克隆），新返回的元素会保留有旧元素的 props、ref、key，也会集成新的 props（只要在第二个参数中有定义），第三个参数为添加的新的子元素

从上面的代码里可以看到，子组件扩展了`getControled`方法里返回的东西，`getControled`方法的实现如下：

```js
getControled = () => {
  // 指定this.context 等于FieldContext.Provider传递过来的form实例对象
  const contextType = FieldContext;

  // 拿到 Field的name，就是表单的label
  const { name } = this.props;
  // this.context
  const { getFieldValue, setFieldsValue } = this.context;
  return {
    value: getFieldValue(name),
    onChange: (...args) => {
      const event = args[0];
      if (event && event.target && name) {
        setFieldsValue({
          [name]: event.target.value,
        });
      }
    },
  };
};
```

从代码可以看出，`Field`为空间扩展了`value`和`onChange`，其中，`value`是通过`getFieldValue`拿到`form`实例对应的值来进行绑定，`onChange`触发了`setFieldsValue`方法来对表单数据进行修改。这两个方法也要在 `FormStore`对象中定义一下

```js
class FormStore {

+  private initialValues = {};

+  getFieldValue = (name) => this.store[name];

+  setFieldsValue = (values, reset) => {
     const nextStore = {
       ...this.store,
       ...values,
     };
     this.store = nextStore;
     this.fieldEntities.forEach(({ props, onStoreChange }) => {
       const name = props.name;
       Object.keys(values).forEach(key => {
         if (name === key || reset) {
           onStoreChange();
         }
       });
     });
     const { onValuesChange } = this.callbacks;
     if (onValuesChange) {
       onValuesChange(values, nextStore);
     }
   };

  getForm = () => {
    return {
+     getFieldValue,
+     setFieldsValue
    };
  };
}
```

到此为止，`Form`、`Field`、`useForm`的核心就完成了，表单可以正常使用，补上一个表单提交的逻辑就行了

# 表单提交

`form`实例提供了一个`submit`方法，调用`form.submit()`就能够实现对表单的提交,`submit`方法中从`fieldEntities`中拿到了所有表单控件，调用了表单控件组件自身上的校验方法

```js
class FormStore {

+ validateFields = () => {
    const promiseList: Promise<{
      name: string;
      errors: string[];
    }>[] = [];
    // 获取到所有在field初始化时保存进来的组件
    this.getFieldEntities(true).forEach(entity => {
      const promise = entity.validateRules();
      const { name } = entity.props;
      promiseList.push(
        promise
          .then(() => ({ name, errors: [] }))
          .catch((errors: any) =>
            Promise.reject({
              name,
              errors,
            }),
          ),
      );
    });

    let hasError = false;
    let count = promiseList.length;
    const results: FieldError[] = [];

    const summaryPromise = new Promise((resolve, reject) => {
      promiseList.forEach((promise, index) => {
        promise
          .catch(e => {
            hasError = true;
            return e;
          })
          .then(result => {
            count -= 1;
            results[index] = result;
            if (count > 0) {
              return;
            }
            if (hasError) {
              reject(results);
            }
            resolve(this.getFieldsValue());
          });
      });
    });

    return summaryPromise as Promise<Store>;
  };

+ submit = async () => {
    this.validateFields()
      .then(values => {
        const { onFinish } = this.callbacks;
        if (onFinish) {
          try {
            onFinish(values);
          } catch (err) {
            console.error(err);
          }
        }
      })
      .catch(e => {
        const { onFinishFailed } = this.callbacks;
        if (onFinishFailed) {
          onFinishFailed(e);
        }
      });
  };
  getForm = () => {
    return {
+    submit,
    };
  };
}
```

这样 一个简单可用的`Form`组件就封装完了，建议配合源码食用
