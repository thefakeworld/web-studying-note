# React-canvas

## 1.介绍
   react-canvas是Filpboard推出的一个基于react的框架，它可以实现将DOM结构的内容渲染到canvas中.
   这样解决了原生DOM渲染速度慢的问题，并且canvas可以被硬件加速，提高了性能。虽然它还有一些缺陷，
   如canvas无语义性，目前仍未完全实现原生dom的功能等，但或许这是一个不错的尝试。
   
   
   我读到的一篇有关[文章](http://www.ruanyifeng.com/blog/2015/02/future-of-dom.html)
   
   官方github中的[介绍](https://github.com/Flipboard/react-canvas)
   
## 2.组件
react-canvas基于react，因此也是以组件的形式使用，其包含的基本组件有：

* Surface
* Layer
* Group
* Text
* Image
* Gradient
* ListView

## 3.例子
以下是官方github中的一个代码例子:
```javascript
var React = require('react');
var ReactCanvas = require('react-canvas');

var Surface = ReactCanvas.Surface;
var Image = ReactCanvas.Image;
var Text = ReactCanvas.Text;

var MyComponent = React.createClass({

  render: function () {
    var surfaceWidth = window.innerWidth;
    var surfaceHeight = window.innerHeight;
    var imageStyle = this.getImageStyle();
    var textStyle = this.getTextStyle();

    return (
      <Surface width={surfaceWidth} height={surfaceHeight} left={0} top={0}>
        <Image style={imageStyle} src='...' />
        <Text style={textStyle}>
          Here is some text below an image.
        </Text>
      </Surface>
    );
  },

  getImageHeight: function () {
    return Math.round(window.innerHeight / 2);
  },

  getImageStyle: function () {
    return {
      top: 0,
      left: 0,
      width: window.innerWidth,
      height: this.getImageHeight()
    };
  },

  getTextStyle: function () {
    return {
      top: this.getImageHeight() + 10,
      left: 0,
      width: window.innerWidth,
      height: 20,
      lineHeight: 20,
      fontSize: 12
    };
  }

});
```