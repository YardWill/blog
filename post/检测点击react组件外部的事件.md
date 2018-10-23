> 在写popover组件的时候，需要捕获当前组件外部的点击事件，而react本身没有提供这一块的api，怎么解决呢，在网上找了资料，答题解决方案 [如此](https://gxnotes.com/article/156068.html)。

看了以上这几种解决方案之后，觉得都不是很优雅，所以又开始寻找新的方法。

其实想法和逻辑都很简单：
1. 监听mousedown事件。
2. 组件内部阻止冒泡。

这样做就可以让内部点击执行内部的操作，外部点击执行外部的操作。

具体代码如下：
```
componentDidMount() {
  document.addEventListener('mousedown', this.handleClickOutside, false);
}
componentWillUnmount() {
  document.removeEventListener('mousedown', this.hiddenOptions, false);
}
handleClickOutside(e) {
  e.stopPropagation();
  this.setState({ show: false });
}
handleClickInside(e) {
  e.stopPropagation();
  this.setState({ show: true });
}
```