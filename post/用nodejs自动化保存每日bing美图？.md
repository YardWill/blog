>特别喜欢bing的每日美图，所以每天日常都是去bing保存下每日更新的那张图片，然后在一次朋友圈回复下---“你为什么不搞成自动化，每天保存不累？？？”，开始了这次的代码之旅。先贴上github地址：https://github.com/YardWill/get-bing-bg 。

###一、找接口
在一些大型网站上的比较大的背景图片都是异步加载的，在bing上也验证了一下，在网速慢的情况下，背景显示的是黑色，之后才显示图片。所以我想，图片地址应该是从接口返回，之后异步加载的。然后去使用chrome的调试工具Network，在里面发现了这样一个接口：http://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=zh-CN 
点开来一看你就知道，图片信息都在里面。

![图片接口](http://upload-images.jianshu.io/upload_images/2419083-890a1877897a7d7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###二、得到图片并保存
这里使用的是request模块，也是比较常用的取接口内容的模块。
```
const request = require('request');
const fs = require('fs');
const path = require('path');

request.get('http://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=zh-CN', (error, response, body) => {
    const img = JSON.parse(body).images[0];
    const arr = img.url.split('/');
    const str = arr[arr.length - 1];
    request(img.url).pipe(fs.createWriteStream(path.join('./img', str)));
    console.log(`${new Date()}${str} is ok!`);
});
```
代码其实很简单
先将img的资源取出来，然后使用fs文件模块将img资源放到目录下的'img'文件夹内。

>但是问题来了，这下代码只是用来保存当前一天的图片存到文件夹里，每天还是得敲一个node app.js才能获取图片，和之前的右键保存并没提升多少效率。

###三、定时任务
既然要每天跑一次，那么我们使用定时任务去做不就好了，这里又使用了一个新的模块叫node-schedule。
node-schedule是专门用来做定时任务的，详情可以去看官方文档，我这里就只简单的贴下代码。
```
const schedule = require('node-schedule');

const rule = new schedule.RecurrenceRule();
rule.dayOfWeek = [0, new schedule.Range(1, 6)];
rule.hour = 8;
rule.minute = 0;

schedule.scheduleJob(rule, () => {
    require('./src/get.js');
});
```
>那么问题又来了，我总不能每天都开着电脑吧？解决方法其实很简单，放到服务器去执行不就行了。但是放在服务器我本地怎么用壁纸呢？这个更简单了，直接使用git，同步壁纸呀！ 一个git pull 命令所有之前保存的图片就都来了！！

###四、服务器执行代码，并自动上传git
在node内使用git命令行，我们又需要一个模块child_process，这个模块是node自带的，可以直接使用。
```
const childProcess = require('child_process');

const cmd = (c) => c;
const shell = 'bash';
const config = {
    env: {
        NODE_ENV: 'production',
        encoding: 'utf8',
    },
    shell,
};

const exec = (c) => {
    return new Promise((resolve, reject) => {
        childProcess.exec(c, config, (err, stdout, stderr) => {
            if (err) {
                reject(err);
            } else {
                resolve(stdout, stderr);
            }
        });
    });
};

console.log('Deploy start.');
exec(cmd('git status'))
    .then(() => exec(cmd('git add .')))
    .then(() => exec(cmd('git commit -m "ss"')))
    .then(() => exec(cmd('git push -u origin master')))
    .then(() => console.log('Deploy end.'))
    .catch(err => {
        console.error(err);
    });
```
这里的代码略微多了一点，但其实也很简单，就是让node去执行命令行，所有的命令行都在最下面。

###五、上线
将代码放置服务器运行，执行request改为如下：
```
const request = require('request');
const fs = require('fs');
const path = require('path');

request.get('http://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=zh-CN', (error, response, body) => {
    const img = JSON.parse(body).images[0];
    const arr = img.url.split('/');
    const str = arr[arr.length - 1];
    request(img.url).pipe(fs.createWriteStream(path.join('./img', str)));
    setTimeout(() => {
        require('./cmd.js');
    }, 20000);
    console.log(`${new Date()}${str} is ok!`);
});
```
这里的20s延迟是为了确保fs将文件保存完再进行git上传的一种简单的处理方式。当然还有其他解决方案，这样粗暴的解决只是因为这是个定时任务，对时间没有什么要求。

###六、验收成果
几天之后在本地
```
git pull
```
这几天保存的壁纸都存到了本地！
真的很爽！
>目前代码只能运行在*nix上，而且还没做很多git的容错处理，以后发现问题再进行修复吧。windows用户如果只是获取壁纸也只需要git pull命令即可。对代码不是很清晰的话，可以借助github内容进行解读，因为觉得比较简单，所以就没做多少注释。



