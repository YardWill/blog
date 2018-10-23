## CommonJS简介
CommonJS是NodeJS内置的模块化方案。

CommonJS基于文件系统，每一个文件都是一个模块，它们都有自己的作用域，模块内部的各种变量都是局部变量，在其他文件内不可用。只有在module.exports声明暴露出去之后，才可以在其他文件内使用。
每一个文件都是一个module，module代表当前模块，module.exports是对外的接口。

## 核心module
Node内部实现了module机制。
以下是node中关于module的简易实例。
https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L103
```
function updateChildren(parent, child, scan) {
  var children = parent && parent.children;
  if (children && !(scan && children.includes(child)))
    children.push(child);
}
function Module(id, parent) {
  this.id = id; // 模块ID
  this.exports = {}; // 模块对外导出的内容
  this.parent = parent; // 父级模块
  updateChildren(parent, this, false); // 更新父模块的子模块
  this.filename = null; // 当前文件的文件名，作为缓存的key值(绝对路径）
  this.loaded = false; // 当前模块是否已经加载（是否缓存），加载完之后会放入this._cache内
  this.children = []; // 当前模块的子模块
}
```
module.exports 是一个对象，也就是需要对parent模块暴露的接口。
另外exports = module.exports，而且在实际使用上exports并没什么用，只是让写法更快一点，但也可能造成不必要的bug。

## require
require命令的基本功能是，读入并执行一个JavaScript文件，然后返回该模块的exports对象。如果没有发现指定模块，会报错。
一下是源码中的部分require实现。
https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L516
```
Module.prototype.require = function(id) {
  if (typeof id !== 'string') {
    throw new ERR_INVALID_ARG_TYPE('id', 'string', id);
  }
  if (id === '') {
    throw new ERR_INVALID_ARG_VALUE('id', id,
                                    'must be a non-empty string');
  }
  return Module._load(id, this, /* isMain */ false);
};
```
我们看到require几乎就是_load方法的映射，那么我们来看看_load方法。
```
Module._load = function(request, parent, isMain) {
  if (parent) {
    debug('Module._load REQUEST %s parent: %s', request, parent.id);
  }

  var filename = Module._resolveFilename(request, parent, isMain);

  var cachedModule = Module._cache[filename];
  if (cachedModule) {
    updateChildren(parent, cachedModule, true);
    return cachedModule.exports;
  }

  if (NativeModule.nonInternalExists(filename)) {
    debug('load native module %s', request);
    return NativeModule.require(filename);
  }

  // Don't call updateChildren(), Module constructor already does.
  var module = new Module(filename, parent);

  if (isMain) {
    process.mainModule = module;
    module.id = '.';
  }

  Module._cache[filename] = module;

  tryModuleLoad(module, filename);

  return module.exports;
};
```
我们忽略debug内容，我们按照顺序来看这里都做了什么。
1. 获取模块的filename，也就是上面讲的用于缓存的key。
2. 查找是否有当前模块的缓存，有的话直接return 当前缓存内的module，反之则继续以下步骤。
3. 如果是NodeJS的内部模块，则直接return此模块。
4. 使用new Module生成module实例，也就是生成依赖树。
5. 在Module的静态变量_cache内缓存当前的module。
6. tryModuleLoad(module, filename)，内部去compile文件，读取到内存。
7. 把当前的module.exports暴露出去。

那么tryModuleLoad里面做了什么呢？
我们再来看看代码。
```
function tryModuleLoad(module, filename) {
  var threw = true;
  try {
    module.load(filename);
    threw = false;
  } finally {
    if (threw) {
      delete Module._cache[filename];
    }
  }
}
```
解析出错就删除当前的缓存（缓存指向先于load前运行），再看下module.load方法。
```
// Given a file name, pass it to the proper extension handler.
Module.prototype.load = function(filename) {
  debug('load %j for module %j', filename, this.id);

  assert(!this.loaded);
  this.filename = filename;
  this.paths = Module._nodeModulePaths(path.dirname(filename));

  var extension = path.extname(filename) || '.js';
  if (!Module._extensions[extension]) extension = '.js';
  Module._extensions[extension](this, filename); // compile
  this.loaded = true;

  // 暂不考虑esmodule的适配
```
当this.loaded是false的时候才会去load，最后把loaded置为true。

这里的Module._nodeModulePaths为require去查找位置的顺序，paths是一个数组，优先为本地path.dirname同级的node_modules，逐次向上，在/node_modules下停止，直到命中文件（直接读取文件）或者文件夹（优先取package内定义的main文件指向，其次读取文件下下的index.js）。

然后再Module._extensions[extension](this, filename); 内部去compile文件，也就是将代码内容转换成内存中的对象，代码的执行也在这一步。
```
Module._extensions['.js'] = function(module, filename) {
  var content = fs.readFileSync(filename, 'utf8');
  module._compile(stripBOM(content), filename);
};
```
```
Module.prototype._compile = function(content, filename) {

  content = stripShebang(content);

  // create wrapper function
  var wrapper = Module.wrap(content);

  var compiledWrapper = vm.runInThisContext(wrapper, {
    filename: filename,
    lineOffset: 0,
    displayErrors: true
  });

  var inspectorWrapper = null;
  if (process._breakFirstLine && process._eval == null) {
    if (!resolvedArgv) {
      // we enter the repl if we're not given a filename argument.
      if (process.argv[1]) {
        resolvedArgv = Module._resolveFilename(process.argv[1], null, false);
      } else {
        resolvedArgv = 'repl';
      }
    }

    // Set breakpoint on module start
    if (filename === resolvedArgv) {
      delete process._breakFirstLine;
      inspectorWrapper = process.binding('inspector').callAndPauseOnStart;
    }
  }
  var dirname = path.dirname(filename);
  var require = makeRequireFunction(this);
  var depth = requireDepth;
  if (depth === 0) stat.cache = new Map();
  var result;
  if (inspectorWrapper) {
    result = inspectorWrapper(compiledWrapper, this.exports, this.exports,
                              require, this, filename, dirname);
  } else {
    result = compiledWrapper.call(this.exports, this.exports, require, this,
                                  filename, dirname);
  }
  if (depth === 0) stat.cache = null;
  return result;
};
```
模块将解析成一个立即执行函数，然后使用vm这个内置库来compile文件。https://nodejs.org/api/vm.html

另外当前module下的缓存将存在于require.cache内。可以在node内将require.cache log出来，你就可以看到这个cache里的内容。

