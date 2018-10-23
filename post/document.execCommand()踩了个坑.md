> 不知大家对document.execCommand()这个api是否熟悉，在项目中使用这个api踩到了一个坑。

# 起始
刚开始发现这个问题是为了实现ajax请求结束后调用exec这个api去执行复制操作。但是在复制的时候显示复制错误，exec这个api返回false，然后去找了很多资料，最后在https://stackoverflow.com/questions/31925944/execcommandcopy-does-not-work-in-ajax-xhr-callback上找到答案，
```
You can only trigger a copy to the system clipboard in direct response to a trusted user action, such as a click event.
```
document.execCommand()这个api只能在真正的用户操作之后才能被触发，是为了安全考虑。原理大致是这样的，当用户操作之后，chrome会将当前作用域下的userAction变量置为True（编的变量名，逃），然后执行execCommand时就会去读取这个变量，当为True的时候才可以执行。
# 那么为什么ajax之后就不可以调用了呢？
因为ajax基本都是异步请求，而异步请求不同于同步请求的地方就在于重新创建了一个作用域去执行回调函数。
所以在重新创建一个作用域之后，之前作用域内的userAction就失效了，当前作用域下的userAction为false，所以复制不成功。

# 解决方法
1. 用真正的用户操作去执行execCommand。（可能需要修改交互流程）
2. 将异步请求改成同步请求。这样做就不会创建新的作用域，execCommand命令依旧在userAction为true的上下文下执行。（当然这种做法也不是很推荐，但为了满足需求只能这样做，只要把xhr.open里的最后一个参数改为false即可满足同步请求）