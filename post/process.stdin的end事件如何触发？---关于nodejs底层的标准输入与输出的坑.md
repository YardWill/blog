> 最近想用js来写一点简单的算法题，node是使用process.stdin和process.stdout来实现标准输入和输出的，我的目标是实现****循环输入****，遇到没有输入时，输入结束。听起来好像很简单，那么接下来我们就来试试。

官方文档
===
首先我们去看看官方文档

![process.stdin](http://upload-images.jianshu.io/upload_images/2419083-d6219021b30ffbc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
代码如下
```
process.stdin.setEncoding('utf8');

process.stdin.on('readable', () => {
  var chunk = process.stdin.read();
  if (chunk !== null) {
    process.stdout.write(`data: ${chunk}`);
  }
});

process.stdin.on('end', () => {
  process.stdout.write('end');
});
```
运行这段程序，接着输入，结果如下，变成了无无限循环的输入，***无论怎么输入都无法触发'end'事件***。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2419083-7711723d01b4ac7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是为什么呢？
我们继续去查阅官方文档

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2419083-c3389d08ab258c61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

文档里写着：当完成没有内容输入时就可以触发'end'事件。
看到这里，感觉前面的代码并没有错，那么为什么他不会结束循环输入呢？

我们把代码修改如下：
```
process.stdin.setEncoding('utf8');

process.stdin.on('readable', () => {
  var chunk = process.stdin.read();
  if(typeof chunk === 'string'){
    process.stdout.write(`stringLength:${chunk.length}\n`);
  }
  if (chunk !== null) {
    process.stdout.write(`data: ${chunk}`);
  }
});

process.stdin.on('end', () => {
  process.stdout.write('end');
});
```
运行结果如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2419083-fb3bc6d4f786a35e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当输入一个字符的时候字符串长度为3，之后都是字符串长度加2，这是什么原因呢？
***你还记得你每次输入结束之后都要敲的回车键吗？回车键的字符就是'\n'***
知道问题的原因就好解决了，既然加了回车字符，那么我们就将回车字符去掉，最简单的方法当然是切片。
```
chunk = chunk.slice(0,-2);
```


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2419083-2092bf06773745cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
现在的数据就没问题了。

原代码修改如下：
```
process.stdin.setEncoding('utf8');

process.stdin.on('readable', () => {
  var chunk = process.stdin.read();
  if(typeof chunk === 'string'){
    chunk = chunk.slice(0,-2);
    process.stdout.write(`stringLength:${chunk.length}\n`);
  }
  if(chunk === ''){
    process.stdin.emit('end');
    return
  }
  if (chunk !== null) {
    process.stdout.write(`data: ${chunk}\n`);
  }
});

process.stdin.on('end', () => {
  process.stdout.write('end');
});
```
就可以做到当没有输入时触发'end'事件。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2419083-d0f9b3bde0132f73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果要做字符串处理就在end事件内执行。