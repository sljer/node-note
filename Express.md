#### 一、Express框架
Express框架是后台的Node框架，和jQuery、zepto、yui、bootstrap都不一个东西。Express在后台的受欢迎的程度，和jQuery一样，就是企业的事实上的标准。

原生Node开发，会发现有很多问题。比如：    

	■ 呈递静态页面很不方便，需要处理每个HTTP请求，还要考虑304问题
	■ 路由处理代码不直观清晰，需要写很多正则表达式和字符串函数
	■ 不能集中精力写业务，要考虑很多其他的东西
<!--more-->
    EXPRESS的哲♂学是在你的想法和服务器之间充当薄薄的一层。这并不意味着他不够健壮，或者没有足够的有用特性，而是尽量少干预你，让你充分表达自己的思想，同时提供一些有用的东西。

英语官网：http://expressjs.com/
中文官网：http://www.expressjs.com.cn/

整体感知，Express框架：
安装Express框架，就是使用npm的命令。

`npm install --save express`

`--save`参数，表示自动修改package.json文件，自动添加依赖项。
       
路由能力：
```javascript
var express = require("express");
var app = express();
app.get("/",function(req,res){
    res.send("你好");
	});
app.get("/haha",function(req,res){
    res.send("这是haha页面，哈哈哈哈哈哈");
	});
app.get(/^\/student\/([\d]{10})$/,function(req,res){
    res.send("学生信息，学号" + req.params[0]);
	});	app.get("/teacher/:gonghao",function(req,res){
    res.send("老师信息，工号" + req.params.gonghao);
	});
	app.listen(3000);
```

静态文件伺服能力：

`app.use(express.static("./public"));`

模板引擎：
```javascript
var express = require("express");
var app = express();
app.set("view engine","ejs");
app.get("/",function(req,res){
    res.render("haha",{
        "news" : ["我是小黄图啊","我也是啊","哈哈哈"]
	    });
	});	
	app.listen(3000);

```
我们学习的是Express4.X，和Express3.X差别非常大。 

#### 二、路由
当用get请求访问一个网址的时候，做什么事情： 
```javascript
app.get("网址",function(req,res){
	});
```

当用post访问一个网址的时候，做什么事情：
```javascript
app.post("网址",function(req,res){
	});
```

如果想处理这个网址的任何method的请求，那么写all
```javascript
app.all("/",function(){
	});
```

这里的网址，不分大小写，也就是说，你路由是
```javascript
app.get("/AAb",function(req,res){
	    res.send("你好");
	});
```
实际上小写的访问也行。

所有的GET参数，? 后面的都已经被忽略。 锚点#也被忽略
你路由到/a ， 实际/a?id=2&sex=nan 也能被处理。

正则表达式可以被使用。正则表达式中，未知部分用圆括号分组，然后可以用req.params[0]、[1]得到。
req.params类数组对象。
```javascript
app.get(/^\/student\/([\d]{10})$/,function(req,res){
	    res.send("学生信息，学号" + req.params[0]);
	});

```
冒号是更推荐的写法。
```javascript
app.get("/student/:id",function(req,res){
   var id = req.params["id"];
   var reg= /^[\d]{6}$/;   //正则验证
    if(reg.test(id)){
        res.send(id);
    }else{
        res.send("请检查格式");
  	  }
	});
```
 

表单可以自己提交到自己上。 
```javascript
app.get("/",function(req,res){
    res.render("form");
	});

app.post("/",function(req,res){
    //将数据添加进入数据库
    res.send("成功");
	});
```

适合进行 RESTful路由设计。简单说，就是一个路径，但是http method不同，对这个页面的使用也不同。

**/student/345345**

get  读取学生信息
add	 添加学生信息
delete  删除学生新

#### 三、中间件
如果我的的get、post回调函数中，没有next参数，那么就匹配上第一个路由，就不会往下匹配了；
如果想往下匹配的话，那么需要写next()

```javascript
app.get("/",function(req,res,next){
    console.log("1");
    next();
	});
app.get("/",function(req,res){
	    console.log("2");
	});
```

下面两个路由，感觉没有关系：
```javascript
app.get("/:username/:id",function(req,res){
	    console.log("1");
	    res.send("用户信息" + req.params.username);
	});
	app.get("/admin/login",function(req,res){
	    console.log("2");
	    res.send("管理员登录");
	});
```
但是实际上冲突了，因为admin可以当做用户名 login可以当做id。

解决方法1：交换位置。 也就是说，express中所有的路由（中间件）的顺序至关重要。

匹配上第一个，就不会往下匹配了。 具体的往上写，抽象的往下写。
```javascript
app.get("/admin/login",function(req,res){
	    console.log("2");
	    res.send("管理员登录");
	});	
	app.get("/:username/:id",function(req,res){
	    console.log("1");
	    res.send("用户信息" + req.params.username);
	});
```

解决方法2： 
```javascript
app.get("/:username/:id",function(req,res,next){
	    var username = req.params.username;
	    //检索数据库，如果username不存在，那么next()
	    if(检索数据库){
	        console.log("1");
	        res.send("用户信息");
	    }else{
	        next();
	    }
	});
	app.get("/admin/login",function(req,res){
	    console.log("2");
	    res.send("管理员登录");
	});
```

路由get、post这些东西，就是中间件，中间件讲究顺序，匹配上第一个之后，就不会往后匹配了。next函数才能够继续往后匹配。

app.use()也是一个中间件。与get、post不同的是，他的网址不是精确匹配的。而是能够有小文件夹拓展的。

比如网址：  http://127.0.0.1:3000/admin/aa/bb/cc/dd
```javascript
app.use("/admin",function(req,res){ 
	    res.write(req.originalUrl + "\n");   //    /admin/aa/bb/cc/dd
	    res.write(req.baseUrl + "\n");  //   /admin
	    res.write(req.path + "\n");   //    /aa/bb/cc/dd
	    res.end("你好");
	});
```

如果写一个/
```javascript
//当你不写路径的时候，实际上就相当于"/"，就是所有网址
	app.use(function(req,res,next){
	    console.log(new Date());
	    next();
	});
```

app.use()就给了我们增加一些特定功能的便利场所。
实际上app.use()的东西，基本上都从第三方能得到。


● 大多数情况下，渲染内容用res.render()，将会根据views中的模板文件进行渲染。如果不想使用views文件夹，想自己设置文件夹名字，那么app.set("views","aaaa");

● 如果想写一个快速测试页，当然可以使用res.send()。这个函数将根据内容，自动帮我们设置了Content-Type头部和200状态码。send()只能用一次，和end一样。和end不一样在哪里？能够自动设置MIME类型。

● 如果想使用不同的状态码，可以：
```javascript
res.status(404).send('Sorry, we cannot find that!');
```
● 如果想使用不同的Content-Type，可以：
```javascript
res.set('Content-Type', 'text/html');
```

 
四、GET请求和POST请求的参数

*● GET请求的参数在URL中，在原生Node中，需要使用url模块来识别参数字符串。在Express中，不需要使用url模块了。可以直接使用req.query对象。*

*● POST请求在express中不能直接获得，必须使用body-parser模块。使用后，将可以用req.body得到参数。但是如果表单中含有文件上传，那么还是需要使用formidable模块。*



**Node中全是回调函数，所以我们自己封装的函数，里面如果有异步的方法，比如I/O，那么就要用回调函数的方法封装。**

错误：
```javascript
res.reder("index",{
		"name" : student.getDetailById(1234234).name
	});
```

正确:
```javascript
student.getDetailByXueHao(1234234,function(detail){
		res.render("index",{
			"name" : detail.name
		})
	});
```
