# Flux架构(参考[阮一峰老师的教程](http://www.ruanyifeng.com/blog/2016/01/flux.html))

之前用react的时候，感觉组件的数据传递处理挺不好弄，各种往组件对象里加属性，加方法，代码规模打起来就一团糟了，后来才发现是我没有用Flux去架构。

***
## 1.基本结构
***
Flux将应用分成四个部分进行架构：
	* View: 视图层，负责组件的显示
	* Action: 从视图层发出的消息(比如事件)
	* Dispatcher: 结构Action，执行对应的回调函数
	* Store：存放应用的状态数据, 由Dispatcher来操作，并且决定View的更新
下面是数据的流动过程:
	1)用户访问View
	2)View发出Action
	3)Dispatcher接收到Action，要求Store更新数据
	4)Store更新，向View发出'Change'事件
	5)View监听到'Change'事件，更新显示

## 2.例子(代码来自阮一峰老师的教程)
***

1)下面是View的建立:
```
// index.jsx
var React = require('react');
var ReactDOM = require('react-dom');
var MyButtonController = require('./components/MyButtonController');

ReactDOM.render(
  <MyButtonController/>,
  document.querySelector('#example')
);
```
这里看到渲染的组件为MyButtonController,这是View中专门保存组件状态的一层。下面是该组件的定义:
```
// components/MyButtonController.jsx
var React = require('react');
var ButtonActions = require('../actions/ButtonActions');
var MyButton = require('./MyButton');

var MyButtonController = React.createClass({
  createNewItem: function (event) {
    ButtonActions.addNewItem('new item');
  },

  render: function() {
    return <MyButton
      onClick={this.createNewItem}
    />;
  }
});

module.exports = MyButtonController;
```
Controller定义了对View层的行为，其中createNewItem是其对Action层发出的请求行为方法。Controller将组件的状态(属性、DOM事件绑定等)传到负责显示内容的子组件中，达到控制View的显示的目的。

```
// components/MyButton.jsx
var React = require('react');

var MyButton = function(props) {
  return <div>
    <button onClick={props.onClick}>New Item</button>
  </div>;
};

module.exports = MyButton;
```
子组件没有任何状态，纯定义显示的内容。

2)接下来定义Action和Dispatcher
***
Action实际是一种对象结构的规范，该对象包含ActionType属性说明其类型以及其它属性（数据）。
```
// actions/ButtonActions.js
var AppDispatcher = require('../dispatcher/AppDispatcher');

var ButtonActions = {
  addNewItem: function (text) {
    AppDispatcher.dispatch({
      actionType: 'ADD_NEW_ITEM',
      text: text
    });
  },
};
```
ButtonAction对象管理所有的Action以及负责将其派发给Dispatcher处理。
```
// dispatcher/AppDispatcher.js
var Dispatcher = require('flux').Dispatcher;
module.exports = new Dispatcher();
```
Dispatcher类需要用到官方的Dispatcher包类实现。
```
// dispatcher/AppDispatcher.js
var ListStore = require('../stores/ListStore');

AppDispatcher.register(function (action) {
  switch(action.actionType) {
    case 'ADD_NEW_ITEM':
      ListStore.addNewItemHandler(action.text);
      ListStore.emitChange();
      break;
    default:
      // no op
  }
});
```
这里是Dispatcher对各类Action的回调函数定义，根据Action类型对Store发出不同的请求操作。

3)最后是Store层，保存应用状态数据以及其操作接口

```
// stores/ListStore.js
var ListStore = {
  items: [],

  getAll: function() {
    return this.items;
  },

  addNewItemHandler: function (text) {
    this.items.push(text);
  },

  emitChange: function () {
    this.emit('change');
  }
};

module.exports = ListStore;
```
Store需要监测数据的更新以向View发送'change'事件，因此要实现事件接口
```
// stores/ListStore.js
var EventEmitter = require('events').EventEmitter;
var assign = require('object-assign');

var ListStore = assign({}, EventEmitter.prototype, {
  items: [],

  getAll: function () {
    return this.items;
  },

  addNewItemHandler: function (text) {
    this.items.push(text);
  },

  emitChange: function () {
    this.emit('change');
  },

  addChangeListener: function(callback) {
    this.on('change', callback);
  },

  removeChangeListener: function(callback) {
    this.removeListener('change', callback);
  }
});
```

4)最后，要给View层加上对'change'事件的监听
```
// components/MyButtonController.jsx
var React = require('react');
var ListStore = require('../stores/ListStore');
var ButtonActions = require('../actions/ButtonActions');
var MyButton = require('./MyButton');

var MyButtonController = React.createClass({
  getInitialState: function () {
    return {
      items: ListStore.getAll()
    };
  },

  componentDidMount: function() {
    ListStore.addChangeListener(this._onChange);
  },

  componentWillUnmount: function() {
    ListStore.removeChangeListener(this._onChange);
  },

  _onChange: function () {
    this.setState({
      items: ListStore.getAll()
    });
  },

  createNewItem: function (event) {
    ButtonActions.addNewItem('new item');
  },

  render: function() {
    return <MyButton
      items={this.state.items}
      onClick={this.createNewItem}
    />;
  }
});
```

