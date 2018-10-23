> 之前使用了hexo搭建个人博客，但感觉更新比较麻烦，就又转到了gitpage和简书写博客。相比较起来简书的写作体验还是不错的，但想了想还是决定重构了自己服务器上的hexo代码。写一份在简书，然后备份一份在自己的网站上。先把项目地址贴上[https://github.com/YardWill/markdown-blog](https://github.com/YardWill/markdown-blog)

### 想法
重构之前想了好几个方案，最后还是决定使用markdown来写，这样博客迁移的时候直接复制粘贴文件就可以了，不需要拖库。

### 开始
首先要选择一个webserver，我选择了Koa，简介易用，中间件拆分合理，而且几行代码就能完成一个简易的webserver。
```
const Koa = require('koa2');

const app = new Koa();

// x-response-time
app.use(async (ctx, next) => {
    const start = Date.now();
    await next();
    const ms = Date.now() - start;
    ctx.set('X-Response-Time', `${ms}ms`);
});

// logger
app.use(async (ctx, next) => {
    const start = Date.now();
    await next();
    const ms = Date.now() - start;
    console.log(`${ctx.method} ${ctx.url} - ${ms}`);
});

app.listen(9999);
console.log(`listening on port ${9999}`);
```
这样webserver就搭建好了。接下来我们开始写路由。

### 路由
总共两个页面。List 页面和detail路由，分别是博客首页和博客文章页面。我们把首页的img提取出来是为了快速的替换首页图片。
```
const Router = require('koa-router');
const fs = require('fs');
const marked = require('marked');
const list = require('../config/postsData');

const img = 'http://oexd4utdf.qnssl.com/cdn/share/activity/bg2.jpg';
const router = new Router();

router.get('/', async (ctx) => {
    await ctx.render('index', {
        list,
        img,
    });
});

router.get('/detail/:name', async (ctx) => {
    const name = ctx.params.name;
    const post = fs.readFileSync(`./posts/${name}.md`, 'utf8');
    await ctx.render('detail', {
        detail: marked(post),
        img,
        name,
    });
});

module.exports = router;

```

### 文件列表
因为没有纯数据库，那么这个博客文章列表的数据是从哪里来的呢？在这里我直接使用了文件存储，因为本身数据量也不大，就直接这样快速操作了。在每次项目更新之前，都会重新跑一次生成目录文件的脚本，大致如下：
```
const fs = require('fs');
const path = require('path');
const _ = require('lodash');

const files = fs.readdirSync(path.join(__dirname, '../posts'));
const data = files.map((e) => {
    const time = new Date(fs.statSync(path.join(__dirname, '../posts', e)).birthtime).toLocaleString();
    return {
        name: e.slice(0, -3),
        time,
    };
});

fs.writeFileSync(path.join(__dirname, '../config/postsData.js'), `module.exports = ${JSON.stringify(_.sortBy(data, e => -new Date(e.time)))};`, error => error && console.log(error));

```
然后生成时间倒叙的文章目录数据。

### 解析markdown
server和route都写完了，现在开始解析markdown了，最简单的方式就是直接使用github上开源的markdown parser，这次使用了一个叫 marked 的模块。文档和使用也很简单。
```
marked(post)
```
就能生成解析好的文字。
但是还有个问题，解析完成的字符串会把<>标签转义掉，导致页面上展示的都变成了转义字符，也不是生成好的markdown样式。
查看文档和代码之后发现marked会使用一个默认的escape函数去转义那些标签字符，解决方式就是在marked解析的过程中将escape的默认函数覆盖，作者也在options项里透出，只要添加这个。
```
// template
app.use(views(path.join(__dirname, './views'), {
    extension: 'ejs',
    options: { escape: e => e },
}));
```
当然这个options在哪里传，还看了koa-view的文档。

### Styles
样式什么的就不说了，直接使用了github的markdown解析样式（逃.....
反正想要替换样式也就两个页面。

### demo
> [www.yardwill.top](http://www.yardwill.top/)