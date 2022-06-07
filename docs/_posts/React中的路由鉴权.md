---
title: React中的路由鉴权
date: 2021-06-03 14:39:22
tags:
  - React
categories:
  - 框架
permalink: /pages/f16c02/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

前段时间在更新博客后台的功能，加入了 jwt 鉴权，也不免需要用到路由跳转鉴权，在 Vue 中我们可以很轻松的用路由导航守卫`beforeEach`来注册一个路由前置守卫，但是 react 好像没有提供这方面的 Api，那就自己来写吧。

经过百度，我发现了一种写法，把路由配置写在一个对象中，把对象传入自定义组件进行判断。来开始写这个自定义组件`FrontendAuth`吧。

首先，我们可以先对页面中的路由配置进行梳理，写成下面的样子(项目比较简陋，还在完善中)：

```js
const routerObj = [
  { path: '/', name: 'login', component: Login },
  { path: '/index', name: 'index', component: AdminIndex, auth: true },
  { path: '/404', name: '404', component: ErrorPage },
];
```

然后，把你编写的这个组件的引入

> import FrontendAuth from '../config/frontendAuth'

路由配置传入我们的组件

```js
<Router>
  {/* <Route path='/' exact component={Login} />
  <Route path='/index' component={AdminIndex} />原配置 */}
  <Switch>
    <FrontendAuth routerConfig={routerObj} />
  </Switch>
</Router>
```

现在，我们可以对路由配置进行处理了，在`FrontendAuth`组件中，我门可以这么处理，说明写在注释中：

```js
import React, { Component } from 'react';
import { Route, Redirect } from 'react-router-dom';
import { message } from 'antd';
class FrontendAuth extends Component {
  render() {
    const { routerConfig, location } = this.props;
    //获取即将要跳转到的路径名，默认赋值为'/'
    const { pathname = '/' } = location;
    //获取token,随你存在哪
    const isLogin = sessionStorage.getItem('token');
    console.log(isLogin);
    // 如果该路由不用进行权限校验，登录状态下登陆页除外
    // 因为登陆后，无法跳转到登陆页
    // 这部分代码，是为了在非登陆状态下，访问不需要权限校验的路由
    const targetRouterConfig = routerConfig.find((item) => {
      return item.path.replace(/\s*/g, '') === pathname;
    });
    if (targetRouterConfig && !targetRouterConfig.auth && !isLogin) {
      const { component } = targetRouterConfig;
      return <Route exact path={pathname} component={component} />;
    }
    if (isLogin) {
      console.log('yidengl');
      // 如果是登陆状态，想要跳转到登陆，重定向到主页
      if (pathname === '/') {
        return <Redirect to="/index" />;
      } else {
        // 如果路由合法，就跳转到相应的路由
        if (targetRouterConfig) {
          return (
            <Route path={pathname} component={targetRouterConfig.component} />
          );
        } else {
          // 如果路由不合法，重定向到 404 页面
          return <Redirect to="/404" />;
        }
      }
    } else {
      // 非登陆状态下，当路由合法时且需要权限校验时，跳转到登陆页面，要求登陆
      if (targetRouterConfig && targetRouterConfig.auth) {
        message.error('请先登录');
        return <Redirect to="/" />;
      } else {
        // 非登陆状态下，路由不合法时，重定向至 404
        return <Redirect to="/404" />;
      }
    }
  }
}
export default FrontendAuth;
```

大功告成！这时候我们在没有 token 的情况下访问需要鉴权的页面就会跳转到登录页，并提示**请先登录**

可以继续美滋滋的做其他的了

----------------------2021.7.12 更新------------
在使用过程中，我发现以上的路由鉴权方法存在问题：我们只能对`routerObj`中配置过的路由进行鉴权，当我们在页面中使用一些子路由时，无法跟踪到路由信息，就没办法实现路由鉴权的操作了。

经过一番折腾，我找到了一个更好的方法来解决 React 中的路由鉴权问题：**高阶组件**,贴上路由高阶组件实现鉴权代码：

```js
import React from 'react';
import { Route, Redirect } from 'react-router-dom';
//高阶组件
const PrivateRoute = ({ component: Component, ...props }) => {
  console.log(props);
  return (
    <Route
      {...props}
      render={(router) => {
        const isLogin = sessionStorage.getItem('token');
        console.log(isLogin);
        if (isLogin && isLogin !== 'undefined') {
          return <Component {...router}></Component>;
        } else {
          alert('您还没有登录');
          return <Redirect to={{ pathname: '/' }}></Redirect>;
        }
      }}
    ></Route>
  );
};
export default PrivateRoute;
```

然后，我们只需要在需要鉴权的地方使用该组件包裹：

```js
import PrivateRoute from '../config/privateRoute'
...
   <Router>
      <Switch>
        <PrivateRoute path='/index' component={AdminIndex} />
      </Switch>
    </Router>
...
```

这样就能监听到子路由的变化，同时进行路由鉴权，应该是一个最完美的办法了！瑞思拜！
