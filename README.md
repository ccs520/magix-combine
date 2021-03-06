# magix-combine [![Version Number](https://img.shields.io/npm/v/magix-combine.svg)](https://github.com/thx/magix-combine/ "Version Number") [![THX Team](https://img.shields.io/badge/team-THX-green.svg)](https://thx.github.io/ "THX Team") [![License](https://img.shields.io/badge/license-MIT-orange.svg)](https://opensource.org/licenses/MIT "License")
合并Magix View的html,js,css文件成一个js文件，需要配合其它工具使用。

# 功能
1. 合并html,css,js成一个js文件。[为什么不在打包上线时合并？](https://github.com/thx/magix-combine/issues/5)
2. css只在当前区块内生效。[关于style的scope](https://github.com/thx/magix-combine/issues/6) [css模块](http://www.75team.com/post/1049.html)
3. 基于mx-keys的子模板离线数据提取及预处理
4. 类node模块的写法，由工具加上web loader

# gulp使用示例

## package.json

```js
{
    "name": "magix-test",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "dependencies": {
        "magix-combine": "",
        "gulp": "",
        "gulp-watch": "",
        "gulp-uglify": "",
        "gulp-cssnano": "",
        "del": ""
    }
}
```

## gulpfile.js

```js

var tmplFolder = 'tmpl'; //template folder
var srcFolder = 'src'; //source folder
var buildFolder = 'build'; //build folder

var excludeTmplFolders = [//忽略哪些文件夹或文件的处理
    'tmpl/boot.js'
];
var onlyAllows = {//只处理哪些文件
    '.html': 1,
    '.css': 1,
    '.json': 1,
    '.sass': 1,
    '.less': 1
};

var gulp = require('gulp');
var watch = require('gulp-watch');
var fs = require('fs');
var combineTool = require('magix-combine');
var del = require('del');


combineTool.config({
    tmplFolder: tmplFolder,
    srcFolder: srcFolder,
    buildFolder: buildFolder,
    excludeTmplFolders: excludeTmplFolders,
    onlyAllows: onlyAllows,
    loaderType:'cmd' //or amd
});

gulp.task('cleanSrc', function() {
    return del(srcFolder);
});
gulp.task('combine', ['cleanSrc'], function() {
    combineTool.combine();
});
gulp.task('watch', ['combine'], function() {
    watch(tmplFolder + '/**/*', function(e) {
        console.log(e.path);
        if (fs.existsSync(e.path)) {
            combineTool.processFile(e.path);
        } else {
            combineTool.removeFile(e.path);
        }
    });
});

var uglify = require('gulp-uglify');
var cssnano = require('gulp-cssnano');
gulp.task('cleanBuild', function() {
    return del(buildFolder);
});
gulp.task('build', ['cleanBuild'], function() {
    combineTool.build();
    gulp.src(buildFolder + '/**/*.js')
        .pipe(uglify({
            compress: {
                drop_console: true
            }
        }))
        .pipe(gulp.dest(buildFolder));

    gulp.src(buildFolder + '/**/*.css')
        .pipe(cssnano())
        .pipe(gulp.dest(buildFolder));
});
```

## 内置@命令

### @filename.html
把filename.html文件的内容输出在当前位置

### @filename.html:data
把filename.html文件的子模板对象内容输出在当前位置，需要配合mx-updater

### @filename.html:keys
把filename.html文件的mx-keys所有keys输出在当前位置

### @filename.html:events
把filename.html文件的mx-event事件对象以数组对象输出在当前位置

### @filename.[css,less,scss]
把filename.[css,less,scss]文件的内容输出在当前位置，同时修改css的类名，确保在当前项目唯一，请配合Magix.applyStyle方法使用

### names@filename.[css,less,scss]
把filename.[css,less,scss]文件中的类名映射到新类名的对象，因为编译工具会修改类名，所以通过该对象访问修改后的类名

### names@filename.[css,less,scss][s1,s2,s3]
把filename.[css,less,scss]文件中的类名映射到新类名的对象，该对象仅包含指定的s1,s2和s3，比names@filename.[css,less,scss]更节省资源，但书写略不方便

### global@filename.[css,less,scss]
把filename.[css,less,scss]文件的内容输出在当前位置，但不会对css的类名做任何变动

### ref@filename.[css,less,scss]
引用filename.[css,less,scss]文件的内容，但不输出在当前位置，仅使用编译后的类名替换掉模板中的类名，如果你需要访问编译后的类名，请参考names@filename.[css,less,scss]。

### @filename.[css,less,scss]:className
把filename.[css,less,scss]中经工具编译转换后的className字符串输出在当前位置

### @filename.css:$prefix
**[DELETE] 从1.1.0版本开始删除，与compressCssNames冲突故删除**
获取工具编译filename.css时自动添加的前缀，并输出在当前位置

### @moduleId
当前文件的模块id，如app/views/default

### @../path 或 @x/y/z
相对路径转完整的模块路径，或完整的模块路径转相对路径




## 配置参数说明

### tmplFolder
string 包含html,css,js的模板目录，默认tmpl

### srcFolder
string 把html,css,js合并后的目录，默认src

### buildFolder
string 上线压缩的目录，默认build

### cssnanoOptions
object css压缩选项，更多信息请参考https://www.npmjs.com/package/cssnano

### htmlminifierOptions
object html压缩选项，更多信息请参考https://www.npmjs.com/package/html-minifier

### lessOptions

### sassOptions
object https://www.npmjs.com/package/node-sass

### excludeTmplFolders
array 在处理tmplFolder目录时，跳过某些目录或文件的处理

### snippets
object 代码片断对象，在处理复杂html时有用

### compressCssNames
boolean 是否压缩css名称，默认false

### removeRequire
**[DELETE] 从1.1.0版本开始删除，由loaderType代替**
boolean 是否移除require代码

### generateJSFile
**[DELETE] 从1.1.0版本开始删除，由loaderType和内置的file:loader处理器代替**
function 如何把html,js和css生成最终的js文件，需要开发者实现该方法

### addProcessor
function 添加或重写内容处理器的实现

### atAttrProcessor
function

### compressTmplCommand
function 如何压缩模板引擎的命令

### processAttachedFile
function 如何处理附属js的html和css文件，像css文件在某些情况下无法打包进js，需要发布到上线文件夹里

### processContent
function 处理js文件内容

### walk
function 遍历某个文件夹

### copyFile
function 复制文件

### removeFile
function 删除文件，同时移除相应的缓存内容

### config
function 配置

### combine
function 把tmplFolder中的文件编译合并到srcFolder目录中

### processFile
function 处理单个文件

### build
function 同步srcFolder目录下的文件到buildFolder目录下
