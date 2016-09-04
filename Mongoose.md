#### 一、索引index
数据库中，根据一个字段的值，来寻找一个文档，是很常见的操作。比如根据学号来找一个学生。
这个学号，是唯一的，只要有学号，就能唯一确认一个学生的文档。学号这个属性，就非常适合建立索引，这样一来，查找学生就变得简单了。
这个语句，能够查看检索的过程：

`db.student.find({"name":"user888"});`

学生的姓名是唯一的，为了快速的进行检索，所以就把name属性建立成为“索引”。

`db.student.createIndex({"name":1});`

这样，今后通过name寻找student文档的时候，速度非常快。因为能够快速的从索引表中，找到这个文档。
缺点就是插入每条数据的时候，时间变慢了，效率低了。但是换回来的就是寻找的速度快了。
<!--more-->
索引这个属性，所有的文档都不能相同：
```javascript
db.members.createIndex( { "user_id": 1 }, { unique: true } );
```
#### 二、ejs模式和ajax模式
**ejs模式：**

```javascript
//写服务	app.get("/allstudent",function(req,res,next){
	db.find("students",{},function(err,result){
//寻找完毕之后，result就是一个数组，说实话服务已经成功了
//但是，必须界面必须要可视化，所以要有模板引擎呈递
	res.render("allstudent",{
		"result" : result;
		});
//如果不用模板引擎呈递，可以直接输出JSON，前台用Ajax或者Angular调用
	//var obj = {"result":result};
	//res.json(obj);
		});
	});
<% for(var i = 0 ; i < result.length ; i++) {%>
	<div class="grid">
	<p class="title"><%=result[i].name%></p>	<p class="xuehao"><%=result[i].xuehao%></p>			<p class="sex"><%=result[i].sex%></p>
		</div>
	<%}%>
```

**Ajax模式：**

```javascript
//写服务	app.get("/allstudent",function(req,res,next){
	db.find("students",{},function(err,result){
//如果不用模板引擎呈递，可以直接输出JSON，前台用Ajax或者Angular调用
		var obj = {"result":result};
		res.json(obj);
		});
	});
======================================
	<script type="text/template" id="tem">
//这里是一个underscore模板
	<div class="grid">
	<p class="title">{{=name}}</p>	
	<p class="xuehao">{{=xuehao}}</p>
	<p class="sex">{{=sex}}</p>	
	</div>
	</script>
//前端页面：
//jQuery片段：
//得到模板html
var templateString = $("#tem").html();
//ajax请求
$.get("/allsutdent",function(result){
//这个result就是一个json对象
//要放到页面上，所以为了不字符串拼接，可以用模板引擎underscore
	for(var i = 0 ; i < result.result.length ; i++){
//组装模板
var compiled = _.compiled(templateString,{
		"name" : result.result[i].name;
		"sex" : result.result[i].sex;
		"xuehao" : result.result[i].xuehao;
		});
//上DOM
		$(".liebiao").append($(compiled));
		}
	});
```

#### 三、Mongoose
**是一个将JavaScript对象与数据库产生关系的一个框架**，object related model。操作对象，就是操作数据库了；对象产生了，同时也持久化了。

这个思路是Java三大框架SSH中Hibernate框架的思路。彻底改变了人们使用数据库的方式。

http://mongoosejs.com/

mongoose就是:狐獴（参考《少年派的奇幻漂流》）
 

官网上的**hello world**：

```javascript
//引包，并不需要引用mongodb这个包
var mongoose = require('mongoose');
//链接数据库,haha是数据库名字	mongoose.connect('mongodb://localhost/haha');
//创建了一个模型。猫的模型。所有的猫，都有名字，是字符串。“类”。
var Cat = mongoose.model('Cat', { name: String });
//实例化一只猫
var kitty = new Cat({ name: 'Zildjian' });
//调用这只猫的save方法，保存这只猫
kitty.save(function (err) {
  console.log('喵喵喵');
	});	
	var tom = new Cat({"name":"汤姆猫"});
	tom.save(function(){
		console.log('喵喵喵');
	});
```
上面的代码中，没有一个语句是明显的操作数据库，感觉都在创建类、实例化类、调用类的方法。都在操作对象，但是数据库同步被持久了。

**mongoose的哲♂学，就是让你用操作对象的方式操作数据库。**

创建一个模型

```javascript
mongoose.model("Cat",{"name" : String , "age" : Integer}); 
```
就可以被实例化

`var kitty = new Cat({ name: 'Zildjian' });`

然后这个实例就可以被save。

中文博客：  https://cnodejs.org/topic/51ff720b44e76d216afe34d9

**mongoose首先要想明白一件事儿，所有的操作都不是对数据库进行的。所有的操作都是对类、实例进行的。但是数据库的持久化自动完成了。**

**3.1 数据库连接**

公式：

```javascript
var mongoose = require('mongoose');
//创建数据库连接
var db      = mongoose.createConnection('mongodb://127.0.0.1:27017/haha');
//监听open事件
db.once('open', function (callback) {
    console.log("数据库成功连接");
	});
```

**3.2 定义模型**

创造schema → 定义一些schema的静态方法 → 创造模型
创造schema用什么语句？  new mongoose.schema({});
创造模型用什么语句？ 	db.model(“Student”,schema名字);

```javascript
//创建了一个schema结构。
var studentSchema = new mongoose.Schema({
    name     :  {type : String},
    age      :  {type : Number},
    sex      :  {type : String}
});
//创建静态方法
studentSchema.statics.zhaoren = function(name, callback) {
    this.model('Student').find({name: name}, callback);   //this.model('Student')指的是当前这个类
};
//创建修改的静态方法
studentSchema.statics.xiugai = function(conditions,update,options,callback){
    this.model("Student").update(conditions, update, options, callback);
}
//创建了一个模型，就是学生模型，就是学生类。
//类是基于schema创建的。
var studentModel = db.model('Student', studentSchema);
```

解释一下，什么是静态方法，什么是类方法：

```javascript
//类
function Student(){
	}	
//实例化一个学生
var xiaoming = new Student();
//实例方法，因为这个sleep方法的执行者是类的实例
xiaoming.sleep();
//静态方法（类方法），这个方法的执行者是这个类，不是这个类的实例。
Student.findAllBuJiGe();
```

**前台界面：不操作数据库，只操作类！**


####四、MongoDB

MongoDB：安装、开启、导入数据、Shell管理数据库、Mongo Vue、Node.js做CRUD（增删改查）、DAO层的封装、索引、操作符$set $lt $gt $push $pull

Mongoose： ODM，不用直接操作数据库，操作对象，这个对象自动持久。

**Defining your schema 定义文档结构**

schema定义的时候，支持的类型：

The permitted SchemaTypes are
String、Number、Date、Buffer、Boolean、Mixed、ObjectId、Array

转换为对象：

`mongoose.model(modelName, schema):`

**定义对象（实例）方法：**

    // define a schema
    var animalSchema = new Schema({ name: String, type: String });
    // assign a function to the "methods" object of our animalSchema
    animalSchema.methods.findSimilarTypes = function (cb) {
    return this.model('Animal').find({ type: this.type }, cb);7	}

**虚拟属性：**

```javascript
// define a schema
var personSchema = new Schema({
  name: {
    first: String,
    last: String
	  }
	});	
// compile our model
var Person = mongoose.model('Person', personSchema);
// create a document
var bad = new Person({
   name: { first: 'Walter', last: 'White' }
	});
```

####五、web Socket和Socket.IO框架

**HTTP无法轻松实现实时应用：**

● HTTP协议是无状态的，服务器只会响应来自客户端的请求，但是它与客户端之间不具备持续连接。

● 我们可以非常轻松的捕获浏览器上发生的事件（比如用户点击了盒子），这个事件可以轻松产生与服务器的数据交互（比如Ajax）。

但是，反过来却是不可能的：服务器端发生了一个事件，服务器无法将这个事件的信息实时主动通知它的客户端。

只有在客户端查询服务器的当前状态的时候，所发生事件的信息才会从服务器传递到客户端。

方法：

● **长轮询：**客户端每隔很短的时间，都会对服务器发出请求，查看是否有新的消息，只要轮询速度足够快，例如1秒，就能给人造成交互是实时进行的印象。这种做法是无奈之举，实际上对服务器、客户端双方都造成了大量的性能浪费。

● **长连接：**客户端只请求一次，但是服务器会将连接保持，不会返回结果（想象一下我们没有写res.end()时，浏览器一直转小菊花）。服务器有了新数据，就将数据发回来，又有了新数据，就将数据发回来，而一直保持挂起状态。这种做法的也造成了大量的性能浪费。

WebSocket协议能够让浏览器和服务器全双工实时通信，互相的，服务器也能主动通知客户端了。

● WebSocket的原理非常的简单：**利用HTTP请求产生握手，HTTP头部中含有WebSocket协议的请求，所以握手之后，二者转用TCP协议进行交流（QQ的协议）。**现在的浏览器和服务器之间，就是QQ和QQ服务器的关系了。

所以WebSocket协议，需要浏览器支持，更需要服务器支持。

● 支持WebSocket协议的浏览器有：Chrome 4、火狐4、IE10、Safari5

● 支持WebSocket协议的服务器有：Node 0、Apach7.0.2、Nginx1.3

Node.js上需要写一些程序，来处理TCP请求。

● Node.js从诞生之日起，就支持WebSocket协议。不过，从底层一步一步搭建一个Socket服务器很费劲（想象一下Node写一个静态文件服务都那么费劲）。所以，有大神帮我们写了一个库**Socket.IO**。

● **Socket.IO是业界良心，新手福音。它屏蔽了所有底层细节，让顶层调用非常简单。**并且还为不支持WebSocket协议的浏览器，提供了长轮询的透明模拟机制。

● Node的单线程、非阻塞I/O、事件驱动机制，使它非常适合Socket服务器。
 
网址：http://socket.io/

先要npm下载这个库

`npm install socket.io`

写原生的JS，搭建一个服务器，server创建好之后，创建一个io对象

```javascript
var http = require("http");
var server = http.createServer(function(req,res){
	res.end("你好");
	});	
var io = require('socket.io')(server);
//监听连接事件
io.on("connection",function(socket){
	console.log("1个客户端连接了");
})
server.listen(3000,"127.0.0.1");
```

写完这句话之后，就会发现：
http://127.0.0.1:3000/socket.io/socket.io.js  就是一个js文件的地址了。

现在需要制作一个index页面，这个页面中，必须引用秘密js文件。调用io函数，取得socket对象。

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Document</title>
</head>
<body>
<h1>我是index页面，我引用了秘密script文件</h1>
<script type="text/javascript" src="/socket.io/socket.io.js"></script>
<script type="text/javascript">
		var socket = io();
</script>
</body>
</html>
```

此时，在服务器上，app.js中就要书写静态文件呈递程序，能够呈递静态页面。

```javascript
var server = http.createServer(function(req,res){
	if(req.url == "/"){
//显示首页	fs.readFile("./index.html",function(err,data){
		res.end(data);
		});
		}
	});
```
 
至此，服务器和客户端都有socket对象了。服务器的socket对象：

**服务器端的：**

```javascript
var io = require('socket.io')(server);
//监听连接事件
io.on("connection",function(socket){
	console.log("1个客户端连接了");
	socket.on("tiwen",function(msg){
		console.log("本服务器得到了一个提问" + msg);
		socket.emit("huida","吃了");
		});
	});
```
每一个连接上来的用户，都有一个socket。 由于我们的emit语句，是socket.emit()发出的，所以指的是向这个客户端发出语句。

**广播，就是给所有当前连接的用户发送信息：**

```javascript
//创建一个io对象 
var io = require('socket.io')(server);
//监听连接事件
io.on("connection",function(socket){
	console.log("1个客户端连接了");
	socket.on("tiwen",function(msg){
		console.log("本服务器得到了一个提问" + msg);
		io.emit("huida","吃了");
		});
	});
```























