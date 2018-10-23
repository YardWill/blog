
> 为了方便团队内部成员代码的共用，不写重复而有无意义的代码，打算搭建团队内部私有的cnpm仓库。

## Start
从cnpm.org clone 整个项目。
```
git clone https://github.com/cnpm/cnpmjs.org.git
```
## Install
clone之后，我们运行以下命令install
```
npm install --build-from-source --registry=https://registry.npm.taobao.org --disturl=https://npm.taobao.org/mirrors/node
```
在install过程中或许会遇到些问题，最好按照终端提示的err去排查错误，笔者就遇到了两个问题
1. install依赖与node-gyp，原本服务器的node-gyp版本为0.10，过低导致编译出错，重新安装之后便可以解决。
2. g++ commnd is not found ，本地没有g++环境，也是安装以下就好。

此时cnpm就安装好了，但是远还没部署完成。
## DB
cnpm需要DB支持，并且兼容"mysql"、"sqlite"、"postgres"、"mariadb"四种数据库。
> 自行搜索安装DB。

## Config
完成部署之前要先修改配置文件，新建一个config/config.js文件，参照config/index.js里的内容对比着来，看着comments应该没什么问题，需要什么就在config.js内覆盖。
config/index.js的bind: 127.0.0.1需要注释掉才能被外网访问到，comments里面都写的很清楚，这里和大家提个醒。

## OpsDev
最简单的部署方法，使用pm2模块。
```
npm i pm2 -g
```
全局安装pm2
```
pm2 start dispatch.js
```

## Test
cnpm文档访问地址 127.0.0.1:7002
cnpm镜像源地址 127.0.0.1:7001
可以在命令行 ping或者curl测试
如果两个端口都能正常访问，怎么cnpm的部署已经完美结束了。







