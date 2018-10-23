> 先上github地址[https://github.com/YardWill/react-webpack-gulp-eslint](https://github.com/YardWill/react-webpack-gulp-eslint)


首先贴上README
---
![README.png](http://upload-images.jianshu.io/upload_images/2419083-194b51125d56243c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

集成功能
---
* less预编译成css
* webpack打包处理jsx文件和ES6语法
* eslint作为代码规范，使用airbnb-react规范
* 所有以上功能统一使用gulp自动化工具管理，只需要一个命令执行以上所用功能。

![gulp.png](http://upload-images.jianshu.io/upload_images/2419083-fdcd58e4bce714b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

webpack优化
---
![webpack.png](http://upload-images.jianshu.io/upload_images/2419083-e8b2094561353ed1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 用过webpack的人都应该遇到过webpack打包文件过大的问题，因为webpack会将npm里面的依赖包一起打包进入一个文件，而对于前端的规范而言，这是一种不优雅的实现，类似react和react-dom这样的公共库我们可以通过webpack提取出来。

![压缩后.png](http://upload-images.jianshu.io/upload_images/2419083-a1a2a0293b098f5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
但在开发环境下就不要压缩代码了，这样做既增加到时间也减少代码的可调试性，在上线的时候才对js进行压缩。
** 对于小型项目关于js的优化已经足够 **

尾巴
---
开发环境已经搭建了好了，而且全部自动化实现，可以直接拿去用，也希望有读者能够一起交流学习，之后还会引入react-router构建，因为目前还没有完全实现react-router分模块的异步按需下载功能，先不贴上去。之后简书上会继续更新。