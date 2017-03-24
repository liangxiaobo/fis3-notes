# fis3-notes

[TOC]
## 关于F.I.S

Fis3是百度团队推荐的Fis第三个版本，和**webpack**一样都需要 *node.js*环境的支持。Fis3的**资源定位**非常棒，它能在编译文件的时候，自动将html、css、中的url转换为对应的资源绝对路径.

**知乎上有一篇关于FIS与webpack的区别的文章 ** [https://www.zhihu.com/question/50829160](https://www.zhihu.com/question/50829160)

F.I.S官网：[http://fis.baidu.com](http://fis.baidu.com)

cnpm淘宝镜像地址：[https://npm.taobao.org](https://npm.taobao.org)

### 安装

```bash
cnpm install -g fis3
```

### 升级fis3
```bash
cnpm update -g fis3 # 或者重装 cnpm install -g fis3
```
## 举例
下面介绍说说自己目前要解决一个开发后项目的前端问题,项目用**java spring**开发的，页面前面主要采用**velocity**模板开发；

### 处理图片、css、js
**war包结构**
一般打包后的war包解压后是这样的：
```bash
├── META-INF
├── WEB-INF
├── res
```
res下是资源文件，也就是需要用fis3来处理的表态资源,在开发过程中资源有很多冗余
```bash
├── css
├── css2.0
├── fontcss
├── images
├── images2.0
├── js
├── plugin
├── scripts
├── subject
├── subjectcss
└── video
```
我需要处理之后的目录应该像这样:
```bash
├── css
├── fontcss
├── images
├── plugin
└── scripts
```
把冗余去掉，整合到相应的目录中去。
```javascript
/**
 * fis-conf.js
 * 这里简单的的举例
 */

// 只编译指定目录下的js
fis.match('/res/{js,scripts}/**.js', {
  optimizer: fis.plugin('uglify-js'),
});

/*去除js里的console.log()*/
fis.config.set('settings.optimizer.uglify-js', {
    compress : {
        drop_console: true
    }
});

fis.match('*.css', {
  // fis-optimizer-clean-css 插件进行压缩，已内置
  optimizer: fis.plugin('clean-css'),
});

fis.match('*.png', {
  // fis-optimizer-png-compressor 插件进行压缩，已内置
  optimizer: fis.plugin('png-compressor')
});

// 这个是将模板页面中的css压缩
fis.match('*.vm:css', {
  optimizer: fis.plugin('clean-css')
});

/**
 *以上是对css,js,png的压缩处理
 *下面是对资源的整合
 */

/**
 *
 * 把css放到res/css目录
 * $0 表示匹配到的整个规则
 * $1 表示匹配到的**.css
 */
fis.match('/res/{css,css2.0,subjectcss,subject,video}/**.css', {
  useHash: true,
  release: 'res/css/$1'
});

/**
 *
 * 把js放到res/scripts目录
 */
fis.media(media_key_build).match('/res/{js,scripts,subject,video}/**.js', {
  useHash: true,
  release: 'res/scripts/$1'
});

/**
 *
 * 把jpg,png,gif放到res/images目录
 */
fis..match('**.{gif,jpg,png,ico}', {
  useHash: true,
  release: 'res/images/$1'
});

/**
 * 如果需要把某些文件按原文件路径和原文件名的话，可以这样做
 * 将 "/res/images2.0/account/info" 目录下的 *.{png,jpg,gif}按原目录存放
 * fis.match('/res/images2.0/account/info/*.{png,jpg,gif}', {
 * useHash: false,
 * release: '$0'
 * });
 */
```
这样配置完后运行
```bash
# 我将产出的文件发布到上一层的output目录
fis3 release -d ../output
```
**处理后的res目录**
```bash
├── css
├── fontcss
├── images
├── plugin
└── scripts
```
### WEB-INF下面的文件处理
下面说一说前端页面问题，Fis的资源定位，在处理相应的资源时，会替换掉页面中相应的url,src,href的路径为绝对路径，模板页面不是纯html其中有些url拼接上会有$request.getContextPath()的模板脚本，如果`<script type="text/javascript" src="$request.getContextPath()/res/scripts/a.js"></script>` 资源定位是定位不到的，解析不了这个url，我的解决方案是先将$request.getContextPath()替换成`''(空) /res/scripts/a.js`这样就可以找到 可以利用F.I.S中的 插件`fis3-deploy-replace `这个插件需要安装的
```bash
npm install [-g] fis3-deploy-replace
# 加上 "-g" 表示全局安装，不加表示安装本项目
``` 
### fis3-deploy-replace 使用

```javascript
/**
 * media是FIS用来处理不同环境时的配置比如“开发、测试、生产”
 * 我这样配置就可以单独执行替换处理 `fis3 release release_vm -d ./`
 * ./是将当前目录的文件替换
 */
var media_key_release_vm = "release_vm";
fis.media(media_key_release_vm).match('*.vm', {
  deploy: [
    // 把src的路径上带$request.getContextPath()的替换成''(空)
    fis.plugin('replace', {
      from: 'src\=\"\$request\.getContextPath\(\)',
      to:  'src\=\"' 
    }),
    // 也可以用正则表达式来处理
    fis.plugin('replace', {
        from: /href="(\$request\.getContextPath\(\))(.*)\.css/g,
        to: function($0, $1){
          return $0.replace('\$request\.getContextPath\(\)', '');
        }
    }),
    fis.plugin('local-deliver')
  ]
});
```
关于**media**可以查看F.I.S的官网说明 [http://fis.baidu.com/fis3/docs/api/config-api.html#fismedia](http://fis.baidu.com/fis3/docs/api/config-api.html#fismedia)；
当处理完之后，再执行上面处理资源的fis-conf.js就能把**模板中的资源路径**处理掉绝对路径了

这两个操作可以写成一样**fis-conf.js**配置文件，用**media**方式来配置就可以了，正确的方式如下:
```javascript
var media_key_release_vm = "release_vm", 
    media_key_build = "build";

/*去除js里的console.log()*/
fis.config.set('settings.optimizer.uglify-js', {
    compress : {
        drop_console: true
    }
});

/** 
 * project.files 
 * 实际是过滤哪些文件需要产出的设置，规则前面加!是取反，
 * 故也可以用来过滤掉不需要的文件,
 * 这个可以把发布出来的项目瘦身
 */
fis.set('project.files', 
['!**.svntmp',
'!**.bak', // 过滤掉.bak的备份文本
'!bak.**']);// 过滤掉bak.*的备份文本

// 只编译指定目录下的js
fis.media(media_key_build).match('/res/{js,scripts}/**.js', {
  optimizer: fis.plugin('uglify-js'),
}).

match('*.css', {
  // fis-optimizer-clean-css 插件进行压缩，已内置
  optimizer: fis.plugin('clean-css'),
}).

match('*.png', {
  // fis-optimizer-png-compressor 插件进行压缩，已内置
  optimizer: fis.plugin('png-compressor')
}).

// 这个是将模板页面中的css压缩
match('*.vm:css', {
  optimizer: fis.plugin('clean-css')
}).

/**
 *以上是对css,js,png的压缩处理
 *下面是对资源的整合
 */

/**
 *
 * 把css放到res/css目录
 * $0 表示匹配到的整个规则
 * $1 表示匹配到的**.css
 */
match('/res/{css,css2.0,subjectcss,subject,video}/**.css', {
  useHash: true,
  release: 'res/css/$1'
}).

/**
 *
 * 把js放到res/scripts目录
 */
fis.media(media_key_build).match('/res/{js,scripts,subject,video}/**.js', {
  useHash: true,
  release: 'res/scripts/$1'
}).

/**
 *
 * 把jpg,png,gif放到res/images目录
 */
match('**.{gif,jpg,png,ico}', {
  useHash: true,
  release: 'res/images/$1'
});

/**
 * 如果需要把某些文件按原文件路径和原文件名的话，可以这样做
 * 将 "/res/images2.0/account/info" 目录下的 *.{png,jpg,gif}按原目录存放
 * fis.match('/res/images2.0/account/info/*.{png,jpg,gif}', {
 * useHash: false,
 * release: '$0'
 * });
 */


// 下面是替换部分的配置 fis3 release release_vm -d ./

fis.media(media_key_release_vm).match('*.vm', {
  deploy: [
    // 把src的路径上带$request.getContextPath()的替换成''(空)
    fis.plugin('replace', {
      from: 'src\=\"\$request\.getContextPath\(\)',
      to:  'src\=\"' 
    }),
    // 也可以用正则表达式来处理
    fis.plugin('replace', {
        from: /href="(\$request\.getContextPath\(\))(.*)\.css/g,
        to: function($0, $1){
          return $0.replace('\$request\.getContextPath\(\)', '');
        }
    }),
    fis.plugin('local-deliver')
  ]
});
```
关于 **fis.set()** 看F.I.S官网 [http://fis.baidu.com/fis3/docs/api/config-api.html](http://fis.baidu.com/fis3/docs/api/config-api.html)
最后的 **fis-conf.js** 需要执行两次才能完成处理:

```bash
# 先执行
fis3 release release_vm -d ./
# 再执行
fis3 release build -d ../output
```
还可以直接把处理后的文件重新打包成**war**包自动化发布，后续等待...
