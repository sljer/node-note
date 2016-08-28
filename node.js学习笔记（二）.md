#### 一、模块  
在Node.js中，以模块为单位划分所有功能，并且提供了一个完整的模块加载机制，这时的我们可以将应用程序划分为各个不同的部分。

不可能用一个js文件去写全部的业务。肯定要有MVC。

● 狭义的说，每一个JavaScript文件都是一个模块；而多个JavaScript文件之间可以相互require，他们共同实现了一个功能，他们整体对外，又称为一个广义上的模块。

● Node.js中，一个JavaScript文件中定义的变量、函数，都只在这个文件内部有效。当需要从此JS文件外部引用这些变量、函数时，必须使用exports对象进行暴露。使用者要用require()命令引用这个JS文件。
<!--more-->
foo.js文件中的代码：  
```javascript
var msg = "你好";
exports.msg = msg;
```
msg这个变量，是一个js文件内部才有作用域的变量。
如果别人想用这个变量，那么就要用exports进行暴露。

使用者：
```javascript
var foo = require("./test/foo.js");
console.log(foo.msg);
```
使用者用foo来接收exports对象，也就是说，这里的foo变量，就是文件中的exports变量。

● 一个JavaScript文件，可以向外exports无数个变量、函数。但是require的时候，仅仅需要require这个JS文件一次。使用的它的变量、函数的时候，用点语法即可。所以，无形之中，增加了一个顶层命名空间。

js文件中，可以用exports暴露很多东西，比如函数、变量。
```javascript
var msg = "你好";
var info = "呵呵";
function showInfo(){
    console.log(info);
		}
exports.msg = msg;
exports.info = info;
exports.showInfo = showInfo;
```
在使用者中，只需要require一次。
`var foo = require("./test/foo.js");`
相当于增加了顶层变量。所有的函数、变量都要从这个顶层变量走：
```javascript
console.log(foo.msg);
console.log(foo.info);
foo.showInfo();
```

Node中，js文件和js文件，就是被一个个exports和require构建成为网状的。
不是靠html文件统一在一起的。

**可以将一个JavaScript文件中，描述一个类。
用module.export = 构造函数名的方式向外暴露一个类。
**

也就是说，js文件和js文件之间有两种合作的模式：

1） 某一个js文件中，提供了函数，供别人使用。 只需要暴露函数就行了； exports.msg=msg;

2） 某一个js文件，描述了一个类。 
module.exports = People;


● 如果在require命令中，这么写:
`var foo = require("foo.js");   //没有写./`， 所以不是一个相对路径。是一个特殊的路径
那么Node.js将该文件视为node_modules目录下的一个文件;

● node_modules文件夹并不一定在同级目录里面，在任何直接祖先级目录中，都可以。甚至可以放到NODE_PATH环境变量的文件夹中。这样做的好处稍后你将知道：分享项目的时候，不需要带着modules一起给别人。

● 我们可以使用文件夹来管理模块，比如
`var bar = require("bar"); `
那么Node.js将会去寻找node_modules目录下的bar文件夹中的index.js去执行。

每一个模块文件夹中，推荐都写一个package.json文件，这个文件的名字不能改。node将自动读取里面的配置。有一个main项，就是入口文件：

```javascript
{
  "name": "kaoladebar",
  "version": "1.0.1",
  "main" : "app.js"
}
```

package.json文件，要放到模块文件夹的根目录去。
 
模块就是一些功能的封装，所以一些成熟的、经常使用的功能，都有人封装成为了模块，可以免费下载。

npm也是一个工具名字  node package management

https://www.npmjs.com/

去社区搜索需求，然后点进去，看api。
如果要配置一个模块，那么直接在cmd使用
**npm install 模块名字**
就可以安装。 模块名字全球唯一。
安装的时候，要注意，命令提示符的所在位置。

1.我们的依赖包，可能在随时更新，我们永远想保持更新，或者某持某一个版本；

2.项目越来越大的时候，给别人看的时候，没有必要再次共享我们引用的第三方模块。

**我们可以用package.json来管理依赖。**
在cmd中，使用npm init可以初始化一个package.json文件，用回答问题的方式生成一个新的package.json文件。
使用`npm install`将能安装所有依赖。
npm也有文档，这是package.json的介绍：

https://docs.npmjs.com/files/package.json

require()别的js文件的时候，将执行那个js文件。

注意：
require()中的路径，是从当前这个js文件出发，找到别人。而fs是从命令提示符找到别人。
所以，桌面上有一个a.js， test文件夹中有b.js、c.js、1.txt
a要引用b：
`var b = require(“./test/b.js”);`
b要引用c：
`var b = require(“./c.js”);`

但是，**fs等其他的模块用到路径的时候，都是相对于cmd命令光标所在位置。**
所以，在b.js中想读1.txt文件，推荐用绝对路径：
```javascript
fs.readFile(__dirname + "/1.txt",function(err,data){
if(err) { throw err; }
console.log(data.toString());
	});
```
 
#### 二、post请求
```javascript
var alldata = "";
//下面是post请求接收的一个公式
//node为了追求极致，它是一个小段一个小段接收的。
//接受了一小段，可能就给别人去服务了。防止一个过大的表单阻塞了整个进程
req.addListener("data",function(chunk){
	alldata += chunk;
        });
//全部传输完毕
req.addListener("end",function(){
    console.log(alldata.toString());
		   res.end("success");
				 });

```
原生写POST处理，比较复杂，要写两个监听。文件上传业务比较难写。
所以，用第三方模块。**formidable**。

只要涉及文件上传，那么form标签要加一个属性：
```javascript
<form action="http://127.0.0.1/dopost" method="post" enctype="multipart/form-data">

```

`require(“fs”).rename(“./1.txt”,”./2.txt”);`

#### 三、模板引擎
```javascript
<a href="<%= url %>"><img src="<%= imageURL %>" alt=""></a>
```
数据绑定，就成为一个完整的html字符串了；

**后台模板，著名的有两个，第一个叫做ejs； 第二个叫做jade；**是npm的第三方包。

EJS简单介绍
**Embedded JavaScript templates**
后台模板引擎

   ```javascript
 <ul>
        <% for(var i = 0 ; i < news.length ; i++){ %>
           <li><%= news[i] %></li>
       <% } %>
    </ul>
```


```javascript
var dictionary = {
        a:6,
        news : ["留级了","不要慌","咩哈哈"]
	};
```
