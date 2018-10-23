
>我相信肯定有很多前端开发者和我一样都厌倦了修改完代码再切换到浏览器刷新去看新的实现。今天我们就用一种最简单的文件监听模式和一个gulp插件（gulp-livereload）来实现这个功能。


1. 首先安装gulp
```
npm install gulp -g
```
2. 安装gulp-livereload插件
```
npm i gulp-livereload --save-dev
```
(npm i 为npm install 缩写）
3. 配置gulp配置文件
自动刷新配置
```
//监听所有打包之后的文件变动，自动刷新页面
gulp.task('watch', function () {
  // Create LiveReload server
  livereload.listen();
  // Watch any files in dist/, reload on change
  gulp.watch(['dist/**']).on('change', livereload.changed);
});
```
要使这个能够工作，还需要在浏览器上安装LiveReload插件，或者在你需要自动刷新的页面上加上
1.本地
```
<script>document.write('<script src="http://' + (location.host || 'localhost').split(':')[0] + ':35729/livereload.js?snipver=1"></' + 'script>')</script>
```
2.远程
```
<script src="http://192.168.0.1:35729/livereload.js?snipver=1"></script>
```
整个gulp
```
var gulp = require('gulp');
var less = require('gulp-less');
var minifyCSS = require('gulp-minify-css');
var babel = require('gulp-babel');
var livereload = require('gulp-livereload');
//编译less
gulp.task('less', function () {
  return gulp.src('./src/less/**.less')
    .pipe(less())
    //.pipe(minifyCSS())
    .pipe(gulp.dest('dist/css'));
});
//监听less文件
gulp.task('autoless', function () {
  gulp.watch('./src/less/**.less', ['less'])
})
//编译es6
gulp.task('babel', () => {
  return gulp.src(['./src/js/**.js'])
    .pipe(babel({ presets: ['es2015'] }))
    .pipe(gulp.dest('dist/js'));
});
//监听js文件
gulp.task('autojs', function () {
  gulp.watch('./src/js/**.js', ['babel'])
});
//监听所有打包之后的文件变动，自动刷新页面
gulp.task('watch', function () {
  // Create LiveReload server
  livereload.listen();
  // Watch any files in dist/, reload on change
  gulp.watch(['dist/**']).on('change', livereload.changed);
});
// 使用 gulp.task('default') 定义默认任务
gulp.task('default', ['less', 'autoless', 'babel', 'autojs', 'watch'])
```