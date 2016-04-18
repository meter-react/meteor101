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
      <h1>Hello World</h1>
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
    return (<Griddle ..../>);
  }
}
```

如果你要写一个包装了这样一个组件的 Atmosphere package，需要更多的步骤。

#### Blaze 中的 React 组件 

If you’d like to use React within a larger app built with Blaze (which is a good strategy if you’d like to incrementally migrate an app from Blaze to React), you can use the react-template-helper component which renders a react component inside a Blaze template. First run meteor add react-template-helper, then use the React helper in your template:

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

The component argument is the React component to include, which should be passed in with a helper.

Every other argument is passed as a prop to the component when it is rendered.

Note that there a few caveats:

- React components must be the only thing in the wrapper element. Due to a limitation of React (see facebook/react #1970, #2484), a React component must be rendered as the only child of its parent node, meaning it cannot have any siblings.

- This means a React component also can’t be the only thing in a Blaze template, because it’s impossible to tell where the template will be used.

##### 传回调函数给 React 组件

To pass a callback to a React component that you are including with this helper, simply make a template helper that returns a function, and pass it in as a prop, like so:

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

To use it in Blaze:

``` html
<template name="userDisplay">
  <div>
    {{> React component=UserAvatar userId=_id onClick=onClick}}
  </div>
</template>
```

### 在  React 中 使用 Blaze 模板

Just like we can use React components in Blaze templates, we can also use Blaze templates in React components. This is simularly useful for a gradual transition strategy; but more importantly, it allows us to continue to use the multitude of Atmosphere packages built for Blaze in our React projects, as well as core packages like accounts-ui.

One easy way to do this is with the gadicc:blaze-react-component package. First run meteor add gadicc:blaze-react-component, then import and use it in your components as follows:

``` js
import React from 'react';
import Blaze from 'meteor/gadicc:blaze-react-component';

const App = () => (
  <div>
    <Blaze template="itemsList" items={items} />
  </div>
);
````

The <Blaze template="itemsList" items={items} /> line is the same as if you had written {{> itemsList items=items}} inside of a Blaze template. For other options and further information, see the package’s project page.

## 使用 Meteor 的数据系统

React is a front-end rendering library and as such doesn’t concern itself with how data gets into and out of components. On the other hand, Meteor has strong opinions about data! Meteor operates in terms of publications and methods, used to subscribe to and modify the data in your application.

To integrate the two systems, we’ve developed a react-meteor-data package which allows React components to respond to data changes via Meteor’s Tracker reactivity system.

### 使用 createContainer

Once you’ve run meteor add react-meteor-data, you’ll be able to import the createContainer function, which allows you to create a container component which provides data to your presentational components.

Note that “container components” are analogous to the “smart components” and “presentational components” to the “reusable components” in the pattern we document in the UI\/UX article, if you’d like to read more about how this philosophy relates to Meteor.

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

The container component created by createContainer() will reactively rerender the wrapped component in response to any changes to reactive data sources accessed from inside the function provided to it.

Although this ListContainer container is intended to be instantiated by the React Router (which passes in the props automatically), if we did ever want to create one manually, we would need to pass in the props to the container component (which then get passed into our data function above):

``` html
<ListContainer params={{id: '7'}}/>
```

#### 防止重新渲染

Sometimes changes in your data can trigger re-computations which you know won’t affect your UI. Although React is in general quite efficient in the face of unnecessary re-renders, if you need to control re-rendering, the above pattern allows you to easily use React’s shouldComponentUpdate on the presentational component to avoid re-renders.

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

If you are writing an Atmosphere package and want to depend on React or an npm package that itself depends on React, you can’t use Npm.depends() and Npm.require(), as this will result in 2 copies of React being installed into the application (and besides Npm.require() only works on the server).

Instead, you need to ask your users to install the correct npm packages in the application itself. This will ensure that only one copy of React is shipped to the client and there are no version conflicts.

In order to check that a user has installed the correct versions of npm packages, you can use the tmeasday:check-npm-versions package to check dependency versions at runtime.

