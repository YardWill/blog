> 本文使用开源的monaco-editor（https://github.com/Microsoft/monaco-editor）编辑工具来实现一个在线的code工具。在线预览http://code.yardwill.com/, 因为懒得上cdn，所以目前访问速度可能有点慢。

![image.png](https://upload-images.jianshu.io/upload_images/2419083-37119829729fd9d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1. monaco-editor
左边一块就是我们的editor，在这里我们可以实时编辑code。并且具有代码提示。
![image.png](https://upload-images.jianshu.io/upload_images/2419083-6e0ffb0f6d9f640a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 2. console结果
右边一块是我们的console结果。我们可以来测试一下。输入代码之后点击运行。
![image.png](https://upload-images.jianshu.io/upload_images/2419083-8a532b9fc0b6adef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然也会对错误进行处理。

![image.png](https://upload-images.jianshu.io/upload_images/2419083-ba74f05480db4964.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


可以通过清除日志来删除所有的console内容。


## 3. 原理
原理也很简单，获取editor里的text，然后通过eval在浏览器端执行代码，最后覆盖浏览器内的console方法，来达到console内容的获取。

## 4. 源码
https://github.com/YardWill/live-code