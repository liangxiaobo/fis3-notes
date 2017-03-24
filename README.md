# fis3-notes

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
### 举例
下面介绍说说自己目前要解决一个开发后项目的前端问题,项目用**java spring**开发的，页面前面主要采用**velocity**模板开发；

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

后续等待....