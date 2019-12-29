---
title: gulp 4.0 升级指南
date: 2017-10-01 18:18:13
tags:
- Gulp
categories:
- 前端

---
- gulp.task 移除了三参数语法，现在不能使用数组来指定一个任务的依赖。gulp 4.0 加入了 gulp.series 和 gulp.parallel 来实现任务的串行化和并行化。
任务函数中，如果任务是同步的，需要使用 done 回调。这样做是为了让 gulp 知道你的任务何时完成。类似这样：
```js
gulp.task(‘a’, function(done){
  // sync task…
  done()
})
```
如果任务是异步的，有多个方法可以标记任务完成。

1. 回调
```js
gulp.task('clean', function(done) {
  del('app/*')
    .then(function() {
      done()
    })
    .catch(function(err) {
      throw err
    })
})
```

2. 返回流（Stream）
通常可以通过返回 gulp.src 实现。一个 Stream 在完成或者发生错误后会调用 [end of stream](https://www.npmjs.com/package/end-of-stream) 模块，该模块会执行一个回调来标记是否完成。这里说的 Stream 仅包括 readable/writable/duplex stream 等真实流，因此类似 [event-stream](https://github.com/dominictarr/event-stream) 的伪流是不被支持的。

3. 返回 Promise
把异步任务包装成一个 Promise 并返回。Promise 在完成后调用 onFulfilled 方法来标记任务的完成。

4. 返回子进程
子进程同样也会在完成后调用 [end of stream](https://www.npmjs.com/package/end-of-stream) 模块。
```js
var spawn = require('child_process').spawn
gulp.task('clean', function() { 
  return spawn('rm', ['-rf', path.join(__dirname, 'build')])
});
```

事实上，所有符合 [asyncDone API](https://github.com/gulpjs/async-done#completion-and-error-resolution) 的任务函数，都可以用于 gulp task 中。
- gulp.dest 添加了 dirMode 参数，可以指定生成文件夹的模式。
- gulp.src 的 glob 参数按顺序求值，例如 gulp.src(['*.js', '!b*.js', 'bad.js’])，会去掉 bad.js 外所有 b 开头的 js 文件。
- gulp.src 添加 since 选项，只匹配在某个固定时间后有修改的文件，这可以用来做增量构建。

```js
gulp.task('img', function images() {
  return gulp.src(Path.img.src, {since: gulp.lastRun('img')})
    .pipe(plumber())
    .pipe(imagemin([
      imagemin.gifsicle({interlaced: true}),
      imagemin.jpegtran({progressive: true}),
      imagemin.optipng({optimizationLevel: 1}),
      imagemin.svgo({
        plugins: [
          {removeViewBox: true}
        ]
      })
    ]))
    .pipe(gulp.dest(Path[env]))
})
```

- gulp.dest 添加 overwrite 选项，用来控制是否覆盖已存在的文件。
- gulp.task 就是一个普通函数，因此在 gulp 4.0 中，我们不必把每个 task 函数 都加到 gulp.task 的回调中。

```js
gulp.task('local', gulp.series(clean, gulp.parallel(php, js), watch))

function clean() {
  return spawn('rm', ['-rf', Path[env]])
}

function php() {
  return gulp.src(Path.php.src, {since: gulp.lastRun(php)})
    .pipe(plumber())
    .pipe(gulp.dest(Path[env]))
    .pipe(livereload())
}

function js() {
  return gulp.src(Path.js.src, {since: gulp.lastRun(js)})
    .pipe(plumber())
    .pipe(gulp.dest(Path[env]))
    .pipe(livereload())
}

function watch(done) {
  livereload.listen()
  gulp.watch('app/php/**/*.php', gulp.series(php))
  gulp.watch('app/js/**/*.js',  gulp.series(js))
  done()
}
```
利用 js 函数的提升特性，我们可以在文件开头定义 task。而在 gulp 3.0，主 task 必须在所有子 task 定义完成后才能定义。另外，这里的 clean, php 等任务实际上变成了私有的了。