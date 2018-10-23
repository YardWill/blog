## 主要内容

https://babeljs.io/docs/en

1. **babel工作流**
2. **Toolings**
3. **Plugins**
4. **Presets**
5. Polyfills
6. babel7做的更新


## 1.babel工作流
```
输入字符串 -> @babel/parser parser -> AST -> transformer[s] -> AST -> @babel/generator -> 输出字符串
```
AST
```
抽象语法树（abstract syntax tree 或者缩写为AST），或者语法树（syntax tree），是源代码的抽象语法结构的树状表现形式，这里特指编程语言的源代码。
和抽象语法树相对的是具体语法树（concrete syntax tree），通常称作分析树（parse tree）。
一般的，在源代码的翻译和编译过程中，语法分析器创建出分析树。一旦AST被创建出来，在后续的处理过程中，比如语义分析阶段，会添加一些信息。
```
[AST示例](http://esprima.org/demo/parse.html)


## 2.Toolings
1. @babel/parser 将源代码解析成 AST。
2. @babel/generator 将AST解码生 js代码。
3. @babel/core 包括了整个babel工作流，也就是说在@babel/core里面我们会使用到@babel/parser、transformer[s]、以及@babel/generator。
4. @babel/code-frame 用于生成错误信息并且打印出错误原因和错误行数。（其实就是个console工具类）
5. @babel/helpers 也是工具类，提供了一些内置的函数实现，主要用于babel插件的开发。
6. @babel/runtime 也是工具类，但是是为了babel编译时提供一些基础工具库。作用于transformer[s]阶段，当然这是一个工具库，如果要使用这个工具库，还需要引入@babel/plugin-transform-runtime，它才是transformer[s]阶段里面的主角。
7. @babel/template 也是工具类，主要用途是为parser提供模板引擎，更加快速的转化成AST
8. @babel/traverse 也是工具类，主要用途是来便利AST树，也就是在@babel/generator过程中生效。
9. @babel/types 也是工具类，主要用途是在创建AST的过程中判断各种语法的类型。

通过了解这每一个工具的用途 我们对前面简略的工作流做下填充。

```
@babel/code-frame 为全局错误捕获工具类

@babel/core
├── 输入字符串
├── @babel/parser
│   └── @babel/template
│       └── @babel/types
├── AST
├── transformer[s]
│   ├── @babel/helpers
│   └── @babel/traverse
├── AST
├── @babel/generator
```


## 3.Plugins
每一个Plugin处理自己的一种AST Type语法。
重点讲一下@babel/plugin-transform-runtime
用来解析@babel/runtime工具类内的函数

@babel/runtime 已经被分为 @babel/runtime 和 @babel/runtime-corejs2。
https://babeljs.io/docs/en/babel-runtime-corejs2#difference-from-babel-runtime

corejs是一个核心库 如果需要解析 需要plugin
https://github.com/babel/babel/pull/8266
前者仅包含 Babel 的辅助函数，后者包含辅助函数以及 polyfill 函数（例如 Symbol、Promise）。
core-js

```
{
  "plugins": [
    ["@babel/plugin-transform-runtime", {
      "corejs": false, //是否启用 corejs https://babeljs.io/docs/en/babel-plugin-transform-runtime#corejs
      "helpers": true, // 各种辅助函数
      "regenerator": true, // 启用generator 支持async await
      "useESModules": false
    }]
  ]
}
```

[Plugins List](https://babeljs.io/docs/en/plugins)

## 4.Presets
而Presets可能理解起来更为简单，翻译过来是预设的意思，他几乎就是一个Plugins数组和配置的集合。

@babel/preset-env 重点讲一下，他是以前es2015、es2016和es2017的集合。需要注意的是@babel/preset-env不支持所有在stage-x的plugin。[browserslist](https://github.com/browserslist/browserslist) browserslist会和caniuse数据结合来判断当前语法是否需要转换。target属性完全按照browserslist规则实现。
```
{
  "presets": [
    [
      "env",
      {
        "targets": { // 目标环境
          "browsers": [ // 浏览器
            "last 2 versions",
            "ie >= 10"
          ],
          "node": "current" // node
        },
        "modules": true,  // 是否转译module syntax，默认是 commonjs
        "debug": true, // 是否输出启用的plugins列表
        "spec": false, // 是否允许more spec compliant，但可能转译出的代码更慢https://babeljs.io/docs/en/babel-preset-env#spec
        "loose": false, // 是否允许生成更简单es5的代码，但可能不那么完全符合ES6语义
        "useBuiltIns": false, // 怎么运用 polyfill"usage" | "entry" | false 测试了一下 usage的包最小
        "include": [], // 总是启用的 plugins
        "exclude": [],  // 强制不启用的 plugins
        "forceAllTransforms": false, // 强制使用所有的plugins，用于只能支持ES5的uglify可以正确压缩代码
        "configPath": string //browserslist的路径
        "ignoreBrowserslistConfig": boolean //是否忽略browserslist的配置
      }
    ]
  ],
}
```
useBuiltIns: 'usage' 实验性
而usage就是上面提到的按照目标兼容性和按需原则引入
```
/// input
var a = new Promise(); // a.js
var b = new Map(); // b.js
/// output
// a.js
import "core-js/modules/es6.promise";
var a = new Promise();
// b.js
import "core-js/modules/es6.map";
var b = new Map();
```
useBuiltIns: 'entry'
entry应该是按照目标浏览器的兼容性把所有需要的polyfill都放上去。
```
// input
import "@babel/polyfill";
// output
import "core-js/modules/es7.string.pad-start";
import "core-js/modules/es7.string.pad-end";
```
useBuiltIns: false
就是不管三七二十一，把polyfill里面所有的内容都引入
其他presets列表
stage将被废弃

每一项新特性，要最终纳入ECMAScript规范中，TC39拟定了一个处理过程，称为TC39 process。

其中共包含5个阶段，Stage 0 ~ Stage 4。

**Stage 0: strawman**

一种推进ECMAScript发展的自由形式，任何TC39成员，或者注册为TC39贡献者的会员，都可以提交。

**Stage 1: proposal**

该阶段产生一个正式的提案。

（1）确定一个**带头人**来负责该提案，带头人或者联合带头人必须是TC39的成员。

（2）描述清楚要解决的问题，解决方案中必须包含例子，API以及关于相关的语义和算法。

（3）潜在问题也应该指出来，例如与其他特性的关系，实现它所面临的挑战。

（4）polyfill和demo也是必要的。

**Stage 2: draft**

草案是规范的第一个版本，与最终标准中包含的特性不会有太大差别。

草案之后，原则上只接受增量修改。

（1）草案中包含新增特性语法和语义的，尽可能的完善的形式说明，允许包含一些待办事项或者占位符。

（2）必须包含**2个实验性的具体实现**，其中一个可以是用转译器实现的，例如Babel。

**Stage 3: candidate**

候选阶段，获得具体实现和用户的反馈。

此后，只有在实现和使用过程中出现了重大问题才会修改。（1）规范文档必须是完整的，评审人和ECMAScript的编辑要在规范上签字。

（2）至少要有**两个符合规范的具体实现**。

**Stage 4: finished**

已经准备就绪，该特性会出现在年度发布的规范之中。

（1）通过[Test 262](https://link.jianshu.com?t=http://test262.ecmascript.org/)的**验收测试**。

（2）有2个通过测试的实现，以获取使用过程中的重要实践经验。

（3）ECMAScript的编辑必须规范上的签字。

**通常在stage-1以及之后就会有babel的支持**。
```
stage-0
stage-1
stage-2
stage-3
flow
react
typescript
minify  //比较鸡肋的压缩preset 我也不知道为什么放在presets里面 我觉得他更应该是一个plugin。当然看名字也看的出来babel-preset-minify 这么模块并没有在这次升级中被移到@babel的私有域下。
```

## 5.Polyfills
@babel/polyfill和runtime的差别

runtime提供的其实是一个工具类合集，例如 _extend，是为了减少编译时产生的冗余代码，它不包括所有新的es API比如Set Map Promise等。

而polyfill则是用来提供上述的新的ES API，其中也包括了Array.form Object.assign等新增的原型方法。

使用方法
https://babeljs.io/docs/en/babel-polyfill#usage-in-node-browserify-webpack



## 6.babel7做的更新
1. babel7的npm包正式更名为@babel/core、@babel/cli等。
2. babel7不在支持Node.js 0.10, 0.12, 4 和 5版本。
3. 移除“Stage” 预设（@babel/preset-stage-0 等），同时移除了 @babel/polyfill 中的提议，现在@babel/polyfill几乎只是core-js v2的部分映射。https://github.com/babel/babel/pull/8440 https://babeljs.io/docs/en/v7-migration#remove-proposal-polyfills-in-babel-polyfill-https-githubcom-babel-babel-issues-8416
https://github.com/babel/babel/blob/master/packages/babel-polyfill/src/index.js
4. babylon现在被重命名为@babel/parser
6. 移除（并停止发布）任何年度预设（preset-es2015 等），@babel/preset-env 取代了对这些内容的需求 ;
7. 将react和flow预设分离。
8.重命名了某些包：任何 TC39 提议插件现在都是 -proposal 而不是 -transform，所以 @babel/plugin-transform-class-properties 变成了 @babel/plugin-proposal-class-properties；



core-js（https://github.com/zloirock/core-js）