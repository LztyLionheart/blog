---
title: gulp版本化
date: 2016-09-03 03:58:59
tags: 
- Gulp
categories: 
- Javascript
---

## 乐村淘新架构 /leliuji目录下基于gulp前端版本化

**1. 安装gulp和组件**
```
npm install --save-dev gulp
npm install --save-dev gulp-rev
npm install --save-dev gulp-rev-collector
npm install --save-dev run-sequence
npm install --save-dev gulp-watch
npm install --save-dev minimist
```
**1.1 如没有安装node，请先安装nodejs  https://nodejs.org/en/download/**

**1.2 版本化需要更改部分依赖库代码  更改地方如下**

```
打开node_modules\gulp-rev\index.js
第144行 manifest[originalFile] = revisionedFile;
更新为：manifest[originalFile] = originalFile + ‘?v=’ + file.revHash;

打开nodemodules\gulp-rev\nodemodules\rev-path\index.js
10行 return filename + ‘-‘ + hash + ext;
更新为: return filename + ext;

打开node_modules\gulp-rev-collector\index.js 31行
if ( !_.isString(json[key]) || path.basename(json[key]).replace(new RegExp( opts.revSuffix ), ” ) !== path.basename(key) ) {
更新为: if ( !_.isString(json[key]) || path.basename(json[key]).split(‘?’)[0] !== path.basename(key) ) {

打开node_modules\gulp-rev-collector\index.js 第107行 regexp: new RegExp( ‘([\/\\\\\'”])’ + pattern, ‘g’ ),
 更新为: regexp: new RegExp( ‘([\/\\\\\'”])’ + pattern+'(\\?v=\\w{10})?’, ‘g’ ),
```

**2.以leliuji为根目录 新建gulpfile.js 代码如下**
```
//引入gulp组件
var gulp = require('../../node_modules/gulp'),
    runSequence = require('../../node_modules/run-sequence'),
    rev = require('../../node_modules/gulp-rev'),
    revCollector = require('../../node_modules/gulp-rev-collector'),
    watch = require('../../node_modules/gulp-watch'),
    minimist = require('../../node_modules/minimist');

//定义css js img路径
var cssSrc = 'shop/templates/default/css/*.css',
    jsSrc = 'shop/resource/js/*.js',
    imgSrc = 'shop/templates/default/images/**/',
    cssDest = 'shop/templates/default/css',
    jsDest = 'shop/resource/js',
    imgDest = 'shop/templates/default/images';

//定义目录
var publicDest  = '';
//定义目标文件
var knownOptions = '';

var options = minimist(process.argv.slice(2));
    i = 0;
    for (var key in options) {
        if (key === 'paths') {
            publicDest = options[key];
        }
        if (i>1) {
            knownOptions += ''+options[key]+',';
        }
        i++;
    }
    knownOptions = knownOptions.substring(0,knownOptions.length-1);

//css 生成文件hash编码并生成rev-manifest.json文件名对照映射
gulp.task('revCss',function(){
    return gulp.src(cssSrc)
        .pipe(rev())
        .pipe(rev.manifest())
        .pipe(gulp.dest(cssDest));
});
//js 生成文件hash编码并生成rev-mainfest.json文件名对照映射
gulp.task('revJs',function(){
    return gulp.src(jsSrc)
        .pipe(rev())
        .pipe(rev.manifest())
        .pipe(gulp.dest(jsDest));
});
//Images 生成文件hash编码并生成rev-manifest.json文件名对照映射
gulp.task('revImg',function(){
    return gulp.src(imgSrc)
        .pipe(rev())
        .pipe(rev.manifest())
        .pipe(gulp.dest(imgDest));
});

gulp.task('revdevcss', function () {
    return gulp.src([cssDest+'/*.json', knownOptions],{ buffer :'false' })
        .pipe(revCollector())
        .pipe(gulp.dest(publicDest));
});
gulp.task('revdevjs', function () {
    return gulp.src([jsDest+'/*.json', knownOptions],{ buffer :'false' })
        .pipe(revCollector())
        .pipe(gulp.dest(publicDest));
});
gulp.task('revdevimg', function () {
   return gulp.src([imgDest+'/*.json',knownOptions],{ buffer :'false' })
       .pipe(revCollector())
       .pipe(gulp.dest(publicDest));
});
//构建应用
gulp.task('dev',function(done) {
    condition = false;
    runSequence(
        ['revCss'],
        ['revJs'],
        ['revImg'],
        ['revdevcss'],
        ['revdevjs'],
        ['revdevimg'],
    done);
});

gulp.task('Css',function(done){
    condition = false;
    runSequence(
        ['revCss'],
        ['revdevcss'],
        done);
    gulp.watch(cssSrc,['Css']);
});

gulp.task('Js',function(done){
    condition = false;
    runSequence(
        ['revJs'],
        ['revdevjs'],
        done);
    gulp.watch(jsSrc,['Js']);
});

gulp.task('Img',function(done){
    condition = false;
    runSequence(
        ['revImg'],
        ['revdevimg'],
        done);
    gulp.watch(imgSrc,['Img']);
});

```
**2.1 因项目模版文件过多 监控速度较慢  改为命令行传递文件参数 进行版本化**

**3. 以leliuji目录下 css添加版本化为例 操作步骤如下**
```
命令行进入leliuji根目录
工具书写方法： 运行 gulp 方法名  --paths为 dest输出目录  后面可添加多个监控文件
例: gulp Css --paths "shop/templates/default/layout"  --x "shop/templates/default/layout/buy_layout.php" --r "shop/templates/default/store/article.php"
```
**3.1 命令行注意使用规范双引号 **

**3.2 后期如需进行前端编译打包 可引入webpack   参考：http://www.w2bc.com/Article/50764**
