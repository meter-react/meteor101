## React

读完该指南，你将知道：

- React 是什么，以及为什么你要考虑和Meteor 一起使用它。
- 如何在你的 Meteor 应用中安装 React，以及如何正确使用。
- 如何与 Meteor 的实时数据层整合React。
- 在 React/Meteor 应用中如何路由。

### 简介

React 是一个 JavaScript 库，由 Facebook 的团队开发和发布，用于构建反应式用户界面。React是目前 Meteor 支持的三种渲染库之一；是 Blaze 和 Angular 的替代。

React 有一个生机勃勃并且不断增长的生态系统，与不同的框架结合，被广泛应用于生产环境中。

建议阅读 React 文档，特别是 [thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) 的帖子。

#### 安装和使用 React

在 Meteor 1.3 中安装 React，运行命令：

``` sh
npm install --save react react-dom
```

在你的项目中安装 React 并可以使用 import React from 'react'. 大部分 React 代码是用 JSX，项目包括了缺省安装的 ecmascript package 即可使用。

``` js
import React from 'react';

export default class HelloWorld extends React.Component {
  render() {
    return (
      <h1>Hello World<\/h1>
    );
  }
}
```

你可以使用 react-dom package 渲染一个组件层到DOM：

``` js
import { Meteor } from 'meteor/meteor';
import { render } from 'react-dom';
import HelloWorld from './HelloWorld.jsx';

Meteor.startup(() => {
  render(<HelloWorld/>, document.getElementById('app'));
});
```

需要包括一个 <div id="app"></div> 到 body 的 HTML。

每个新的 Meteor app 缺省包括了 Meteor 的缺省模板系统 Blaze。如果你不打算一起使用 React 和 Blaze，你可以从项目中移除 Blaze：

``` js
meteor remove blaze-html-templates
meteor add static-html
```

#### 使用第三方 packages

如果你想使用在 npm 上发布的第三方 React 组件（例如你在 React Components 网站找到的那些），你可以用下面的命令。

例如，使用出色的 Griddle React package，运行

``` sh
npm install --save griddle-react
```

随后，在应用中导入组件：

``` js
import React from 'react';
import Griddle from 'griddle-react';

export default class MyGriddler extends React.Component {
  render() {
    return (<Griddle ....\/>);
  }
}
```

如果你要写一个包装了这样一个组件的 Atmosphere package，需要更多的步骤。

#### Blaze 中的 React 组件 

如果你要在一个用 Blaze 构建较大的 app 中使用 React （如果你想逐步将一个 app 从 Blaze 移植到 React，这是一个好主意），你可以使用 react-template-helper 组件，它在一个 Blaze 模板中渲染一个 React 组件。首先运行 meteor add react-template-helper，随后中你的模板中使用 React helper：

``` sh
meteor add react-template-helper
```

``` html
<template name="userDisplay">
  <div>Hello, {{username}}</div>
  <div>{{> React component=UserAvatar userId=_id}}</div>
</template>
```

你需要用一个 helper 传入一个组件类：

``` js
import { Template } from 'meteor/templating';

import './userDisplay.html';
import UserAvatar from './UserAvatar.jsx';

Template.userDisplay.helpers({
  UserAvatar() {
    return UserAvatar;
  }
})
```

组件参数是要包含的 React 组件，应该用 helper 传递。

每个其他参数在渲染时被作为属性（prop）传递给组件。

注意下面这些说明：

- React 组件必须是包装（wrapper）元素中惟一的。由于 React 的限制（见 facebook/react [#1970](https://github.com/facebook/react/issues/1970), [#2484]())，一个 React 组件必须作为其父节点的唯一子节点被渲染，这意味着它不能有任何兄弟结点。

- 这意味着一个 React 组件也不能是一个 Blaze 模板中的唯一，因为不可能知道模板将在哪里使用。

##### 传回调函数给 React 组件

为了将一个回调函数传给你使用这个 helper包含的 React 组件，只需使一个模板 helper 返回一个函数，并作为属性传入，如下所示：

``` js
Template.userDisplay.helpers({
  onClick() {
    const instance = Template.instance();

    // Return a function from this helper, where the template instance is in
    // a closure
    return () => {
      instance.hasBeenClicked.set(true)
    }
  }
});
```

在 Blaze 中使用它：

``` html
<template name="userDisplay">
  <div>
    {{> React component=UserAvatar userId=_id onClick=onClick}}
  </div>
</template>
```

### 在  React 中 使用 Blaze 模板

和我们在 Blaze 模板中使用 React 组件一样，我们也能在 React 组件中使用 Blaze 模板。这对逐步转换策略同样有用；更重要的是，它允许我们在项目中继续使用大量为 Blaze 构建的 Atmosphere packages，以及核心 packages，如 accounts-ui。

一种简单的办法是使用 gadicc:blaze-react-component package。首先，运行 meteor add gadicc:blaze-react-component，随后导入并在你的组件中使用如下：

``` js
import React from 'react';
import Blaze from 'meteor/gadicc:blaze-react-component';

const App = () => (
  <div>
    <Blaze template="itemsList" items={items} />
  </div>
);
````

<Blaze template="itemsList" items={items} /> 与在 Blaze 模板中写 {{> itemsList items=items}} 是一样的。对于其他选择和更多信息，参见 package 的项目网页。

## 使用 Meteor 的数据系统

React 是一个前端渲染库，因此，它不关心数据是如何进出组件的。另一方面，Meteor 有强大的数据选项！Meteor 用发布和方法（publications 和 methods）操作， 用于订阅和修改你的应用中的数据。

为整合这两个系统，我们开发了一个 react-meteor-data package，允许 React 组件通过 Meteor 的 Tracker 反应系统响应数据变化。

### 使用 createContainer

一旦你运行了 meteor add react-meteor-data，就能够导入 createContainer 函数，它允许你创建一个容器组件，用于给你的表示组件提供数据。

注意，如果你愿意阅读更多和 Meteor 相关的哲学，“容器组件” 是 “聪明组件” 的同义语，而 “表示组件” 是 “可重用组件”的同义语，其模式是我们在 UI\/UX 文章中介绍的。

For example, in the Todos example app, we have a ListPage component, which renders list metadata and the tasks in the list. In order to do so, it needs to subscribe to the todos.inList publication, check that subscription’s readiness, then fetch the list of todos from the Todos collection.

It also needs to be responsive to reactive changes in the state of those actions (for instance if a todo changes due to the action of another user). All this data loading complexity is a typical use-case for a container-presentational component split, and the createContainer() function makes it simple to do this.

We simply define the ListPage component as a presentational component that expects its data to be passed in using React props:

``` js
import React from 'react';

export default class ListPage extends React.Component {
  ...
}

ListPage.propTypes = {
  list: React.PropTypes.object,
  todos: React.PropTypes.array,
  loading: React.PropTypes.bool,
  listExists: React.PropTypes.bool,
};
```

Then we create a ListContainer container component which wraps it and provides a data source:

``` js
import { Meteor } from 'meteor/meteor';
import { Lists } from '../../api/lists/lists.js';
import { createContainer } from 'meteor/react-meteor-data';
import ListPage from '../pages/ListPage.jsx';

export default createContainer(({ params }) => {
  const { id } = params;
  const todosHandle = Meteor.subscribe('todos.inList', id);
  const loading = !todosHandle.ready();
  const list = Lists.findOne(id);
  const listExists = !loading && !!list;
  return {
    loading,
    list,
    listExists,
    todos: listExists ? list.todos().fetch() : [],
  };
}, ListPage);
```

由 createContainer() 创建的容器组件将反应式地重现渲染被包裹的组件，以回应从为其提供的函数访问的数据源的任何变化。

尽管这个 ListContainer 容器被设计为由 React Router（它自动传入属性）实例化，如果我们想手动创建，需要传入属性到容器组件（随后会被传入上面的数据函数）：

``` html
<ListContainer params={{id: '7'}}/>
```

#### 防止重新渲染

有时，数据中的变化可能触发重新计算，而你不希望它影响你的UI。尽管一般来说 React 在不必要的重新渲染相当高效，如果你需要控制重新渲染，以上的模式允许你在表示组件上轻松地使用 React 的 shouldComponentUpdate 以避免重新渲染。

## 路由

Meteor and React 有两个主要的路由方法。

kadira:flow-router 是 Meteor 专用的路由，React 和 Blaze 都可以用。

react-router 是一个 React 专用的路由，在 React 社区非常流行。它也可以很容易地用于 Meteor。

### Flow Router

在 React 中使用 Flow Router 与在 Blaze 中使用非常类似。唯一的区别是路由动作，你应该使用 react-mounter package 在一个布局中安装组件。运行 npm install --save react-mounter 之后，你可以：

``` js
import { FlowRouter } from 'meteor/kadira:flow-router';
import { mount } from 'react-mounter';

import AppContainer from '../../ui/containers/AppContainer.jsx';
import ListContainer from '../../ui/containers/ListContainer.jsx';

FlowRouter.route('/lists/:_id', {
  name: 'Lists.show',
  action() {
    mount(AppContainer, {
      main: () => <ListContainer/>,
    });
  },
});
```

注意：react-mounter 自动安装布局组件到一个 #react-root 结点，你可以使用 withOptions() 函数修改。

### React Router

使用 React Router 也是直接明了的。运行 npm install --save react-router 后，你可以轻易导出一列嵌套的路由，和在任何 React Router 驱动的 React 应用中一样：

``` js
import React from 'react';
import { Router, Route, browserHistory } from 'react-router';

// route components
import AppContainer from '../../ui/containers/AppContainer.jsx';
import ListContainer from '../../ui/containers/ListContainer.jsx';
import AuthPageSignIn from '../../ui/pages/AuthPageSignIn.jsx';
import AuthPageJoin from '../../ui/pages/AuthPageJoin.jsx';
import NotFoundPage from '../../ui/pages/NotFoundPage.jsx';

export const renderRoutes = () => (
  <Router history={browserHistory}>
    <Route path="/" component={AppContainer}>
      <Route path="lists/:id" component={ListContainer}/>
      <Route path="signin" component={AuthPageSignIn}/>
      <Route path="join" component={AuthPageJoin}/>
      <Route path="*" component={NotFoundPage}/>
    </Route>
  </Router>
);
```

使用 React Router，你将需要中一个启动函数中明确地渲染导出的路由：

``` js
import { Meteor } from 'meteor/meteor';
import { render } from 'react-dom';
import { renderRoutes } from '../imports/startup/client/routes.jsx';

Meteor.startup(() => {
  render(renderRoutes(), document.getElementById('app'));
});
```

在 Meteor 中使用 React Router 时，你可以大致遵循与使用 Flow Router 一样的原则，但是你也应该考虑在 React Router 自己的文档中列出的风格。

一些显著的区别是：

React Router 鼓励你将 URL 设计和布局层次结合中路由定义中。Flow Router 更灵活，尽管它最终包含了更多的样板。

React Router 欢迎特定于 React 的功能，如使用上下文，尽管你也可以显式地将 FlowRouter 实例在上下文中传递。（这可能是最好的做法）。

## Meteor 和 React

### 在 Atmosphere Packages 中使用 React

如果你在写一个 Atmosphere package，并想要依赖于 React 或是一个本身依赖于 React 的 npm package，你无法使用 Npm.depends() 和 Npm.require()，因为这样将导致两份 React 拷贝被安装到应用中（另外， Npm.require() 只工作中服务器上）。

反之，你需要让你的用户中应用中安装正确的 npm packages。这将确保直邮一个 React 拷贝被发送到客户端而且没有版本冲突。

为了检查用户安装了正确的 npm packages 版本，你使用 tmeasday:check-npm-versions package 在运行时检查依赖版本。
