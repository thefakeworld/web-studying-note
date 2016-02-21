# 使用React构建一个应用

* 此笔记大部分内容来自官方文档的一片[文章](http://www.reactjs.cn/react/docs/thinking-in-react.html)

## 1.分析原型和数据

举一个例子，原型如图：
    
![原型](http://www.reactjs.cn/react/img/blog/thinking-in-react-mock.png)

来源的数据如下：

```
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```
我们先将这个原型拆分成组件，拆分原则是一个组件做一件事，一个拆分的方法如图：

![拆分图](http://www.reactjs.cn/react/img/blog/thinking-in-react-components.png)

这个不同颜色的框代表一个组件，从中可以看到其包含关系，下面列出组件树

* FilterableProductTable
    * SearchBar
    * ProductTable
        * productCategoryRow
        * ProductRow

该组件树同时也为组件命了名

## 2.根据组件树用React创建组件

如:
```javascript
var ProductCategoryRow = React.createClass({
  render: function() {
    return (<tr><th colSpan="2">{this.props.category}</th></tr>);
  }
});
```
React的数据是单向流动的，上面的例子代码中的props属性便是来自其父组件传递过来的数据

## 3.确定组件的最小state

React中组件的数据除了来自props属性外，还来自state属性，两者的区别是state一般保存动态的，只与该组件本身相关的数据(不由其父级决定)。

因此该例子中的state数据为SearchBar内的输入文本以及复选框的值(来自用户)

## 4.确认state的生命周期
此步骤其实是确认state数据应该来自哪些组件，应当遵循以下原则：
*有关state数据的渲染应该在拥有该state数据的组件本身或其子组件

因此找state数据的附属组件的方法是先找到渲染该数据的组件，再找出他们的公共祖先组件，若没有则需在组件树中创建一个。

## 5.添加反向数据流

先看下面代码:

```javascript
var SearchBar = React.createClass({
  handleChange: function() {
    this.props.onUserInput(
      this.refs.filterTextInput.value,
      this.refs.inStockOnlyInput.checked
    );
  },
  render: function() {
    return (
      <form>
        <input
          type="text"
          placeholder="Search..."
          value={this.props.filterText}
          ref="filterTextInput"
          onChange={this.handleChange}
        />
        <p>
          <input
            type="checkbox"
            checked={this.props.inStockOnly}
            ref="inStockOnlyInput"
            onChange={this.handleChange}
          />
          {' '}
          Only show products in stock
        </p>
      </form>
    );
  }
});

var FilterableProductTable = React.createClass({
  getInitialState: function() {
    return {
      filterText: '',
      inStockOnly: false
    };
  },

  handleUserInput: function(filterText, inStockOnly) {
    this.setState({
      filterText: filterText,
      inStockOnly: inStockOnly
    });
  },

  render: function() {
    return (
      <div>
        <SearchBar
          filterText={this.state.filterText}
          inStockOnly={this.state.inStockOnly}
          onUserInput={this.handleUserInput}
        />
        <ProductTable
          products={this.props.products}
          filterText={this.state.filterText}
          inStockOnly={this.state.inStockOnly}
        />
      </div>
    );
  }
});
```
这里我们可以看到父组件传递了一个回调函数handleUserInput给子组件SearchBar，SearchBar内定义了handleChange函数，以在状态改变时调用父组件传来的回调函数，更新state数据。

这样做的原因是组件只能更新自身的state，然而state的变化来自于子组件SearchBar，因此需要通过调用来自父组件的回调函数来更新父组件的state。

这就是所谓的反向数据流，因为数据从顶向下传递，自底向上更新。