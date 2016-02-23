# State
* react的组件通过state属性更新来实现其动态的变换，每个组件都是一个状态机，组件通过调用setState方法更新其state属性，然后重新渲染界面，由此使用户界面和数据保持一致性。
## State的定义
* state应包括可能被组件的事件处理器改变并出发用户界面更新的数据，其中不包括:
** 计算所得数据
** 基于props的重复数据
##基于state的动态交互流程
以下面代码为例:
```javascript
var LikeButton = React.createClass({
  getInitialState: function() {
  return {liked: false};
  },
  handleClick: function(event) {
    this.setState({liked: !this.state.liked});
  },
  render: function() {
    var text = this.state.liked ? 'like' : 'haven\'t liked';
    return (
      <p onClick={this.handleClick}>
        You {text} this. Click to toggle.
      </p>
    );
  }
});

React.render(
  <LikeButton />,
  document.getElementById('example')
);
```
这里通过组件内元素的点击事件触发，调用组件的setState方法，更新liked属性，从而影响渲染的text变量.

以上内容来自官方文档的[指南](http://www.reactjs.cn/react/docs/interactivity-and-dynamic-uis.html)
