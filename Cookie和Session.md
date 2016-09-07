#### 一、数据库
Ajax分页案例，当时调用了JSON，有一个参数叫做page=3，生成的JSON不一样。这个就是分页，就是我们想寻找所有的新闻，但是是位于第3页的新闻。那么有两种做法：
1） 错误的做法： 就是讲所有的result都读取到数组，然后进行数据操作，进行分页；
2） 正确的做法： 就是真的在数据库中，只读取这么多内容。


错误的，我们试图每次都读取全部数据，但是这样开销很大。 

```javascript
var a = [];
db.find("student",{},function(err,result){
for(var i = 10 * page ; i < 10 * (page + 1) ; i++){
            a.push(result[i]);
       }
        res.send(a);
    });
```

<!--more-->

所以，mongodb提供了两个呆萌的函数：

**limit()**

**skip()**
 
加入，第一页是page=0，每页10条，所以当前页的查询语句

`db.student.find({}).limit(10).skip(page*10)；`

数据总数怎么得到？

**shell中**

`db.student.stats().count;`
 
ejs页面中，如果我们想使用underscore的模板，就会有模板冲突的问题。

underscore用的：

```javascript
<script type="text/template" id="moban">
<div class="list-group">
<a href="#" class="list-group-item active">
<h4 class="list-group-item-heading"><%= teacher %></h4>
<p class="list-group-item-text"><%= cangjingkong %></p>
</a>
</div>
</script>
```
ejs以为是自己的模板。所以报错，提示你没有船体teacher参数；

解决方法就是underscore源码。在源码中搜索<%即可。
 
#### 二、Cookie和Session
**2.1 Cookie**

● HTTP是无状态协议。简单地说，当你浏览了一个页面，然后转到同一个网站的另一个页面，服务器无法认识到，这是同一个浏览器在访问同一个网站。每一次的访问，都是没有任何关系的。

那么世界就乱套了，比如我上一次访问，登陆了，下一次访问，又让我登陆，不存在登陆这事儿了。

● Cookie是一个简单到爆的想法：当访问一个页面的时候，服务器在下行HTTP报文中，命令浏览器存储一个字符串；浏览器再访问同一个域的时候，将把这个字符串携带到上行HTTP请求中。

第一次访问一个服务器，不可能携带cookie。 必须是服务器得到这次请求，在下行响应报头中，携带cookie信息，此后每一次浏览器往这个服务器发出的请求，都会携带这个cookie。

特点
```javascript
● cookie是不加密的，用户可以自由看到；
● 用户可以删除cookie，或者禁用它
● cookie可以被篡改
● cookie可以用于攻击
● cookie存储量很小。未来实际上要被localStorage替代，但是后者IE9兼容。
```

express中的cookie，你肯定能想到。 res负责设置cookie， req负责识别cookie。

**2.2 Session**

会话。 Session不是一个天生就有的技术，而是依赖cookie。
 
session依赖cookie，当一个浏览器禁用cookie的时候，登陆效果消失； 或者用户清除了cookie，登陆也消失。
session比cookie不一样在哪里呢？  session下发的是乱码，并且服务器自己缓存一些东西，下次浏览器的请求带着乱码上来，此时与缓存进行比较，看看是谁。

所以，一个乱码，可以对应无限大的数据。

任何语言中，session的使用，是“机理透明”的。他是帮你设置cookie的，但是足够方便，让你感觉不到这事儿和cookie有关。

**2.3 session**

```javascript
var session = require("express-session");
app.use(session({
    secret: 'keyboard cat',
    resave: false,
    saveUninitialized: true
	}))
	app.get("/",function(req,res){
		if(req.session.login == "1"){
			res.send("欢迎" + req.session.username);
		}else{
			res.send("没有成功登陆");
		}
	});
	
	app.get("/login",function(req,res){
		req.session.login = "1";	//设置这个session
		req.session.username = "考拉";
		res.send("你已经成功登陆");
	});
```

**加密使用的是MD5加密**
1B0AF4986F1275FDF11114B758AB774B
我爱苍老师
	
8FAB3144FF4F334F94F31F5C8374CF6B
和我签订契约成为魔法少女吧

不管你加密多大的东西，哪怕10M文字，都会加密为32位的字符串，就是密码。并且神奇的，数学上能够保证，哪怕你更改1个文字，都能大变。所以MD5也能用于比对版本。

MD5是数学上，不能破解。不能反向破解。
也就是说，C4CA4238A0B923820DCC509A6F75849B 没有一个函数，能够翻译成为1的；
但是，有的人做数据库，就是把1~999999所有数字都用MD5加密了，然后进行了列表，所以**有破解的可能**。

#### 三、补充
**3.1、cookie**

cookie是在res中设置，req中读取的。第一次的访问没有cookie。

cookie的存储大小有限，kv对儿。对用户可见，用户可以禁用、清除Cookie、可以被篡改。

cookie用来制作记录用户的一些信息，必须购买历史、猜你喜欢。

HTTP是无状态的协议，所以两次的访问，服务器不能认识到是同一个客户端的访问，就要用cookie来巧妙的解决这个问题。

Session就是利用cookie，实现的“会话”。就是第一次访问的时候，可以在服务器上为这个用户缓存一些信息，别的用户是不能看见这个用户的信息的。服务器会下发一个秘钥（cookie），客户端每次访问都携带这个秘钥，那么服务器如果发现这个秘钥吻合，就能够显示这个用户曾经保存的信息。

登陆就是用Session来制作的。任何语言的session都是透明的，不会体现cookie机理。

```javascript
var session = reqiure("express-session");
app.use(session({
		..一些配置
		..一些配置
		..一些配置
	}));
	app.get("/",function(req,res){
		console.log(req.sission.login);
	});	
	app.get("/login",function(req,res){
		 req.session.login = "1";
	});
```
都是使用req对象。

**3.2、加密**

永远不要用明码写密码。CSDN今年泄露用户密码了，并且泄露的明码；

黑客拿到的用户的密码的加密信息，所以也没用。因为他无法翻译成为明码；

MD5加密是函数型加密。就是每次加密的结果一定相同，没有随机位。

特点：


    ● 不管加密的文字，多长多短，永远都是32位英语字母、数字混合。
    ● 哪怕只改一个字，密文都会大变。
    ● MD5没有反函数破解的可能，网上的破解工具，都是通过字典的模式，通过大量列出明-密对应的字典，找到明码。两次加密网上也有对应的字典。所以我们不要直接用一层md5，这样对黑客来说和明码是一样。

MD5常用于作为版本校验。可以比对两个软件、文件是否完全一致。

**node中自带了一个模块，叫做crypto模块，负责加密。**

首先创建hash，然后update和digest：

```javascript
var md5 = crypto.createHash('md5');
var password = md5.update(fields.password).digest('base64');
```

#### 四、图片处理
http://www.graphicsmagick.org/

GraphicsMagick is the swiss army knife of image processing. 

只要服务器需要处理图片，那么这个服务器就要安装graphicsmagick软件。免费的。

装完之后，可视化工具一点用都没有，从桌面上删除。我们要把安装目录设置为环境变量。

控制台CMD命令：

```javascript
//格式转换
gm convert a.bmp a.jpg

//更改当前目录下*.jpg的尺寸大小，并保存于目录.thumb里面
gm mogrify -resize 320x200 danny.jpg

```
nodejs要使用graphicsmagick，需要npm装一个gm的包。

node.js缩略图的制作：

```javascript
var fs = require('fs');
var gm = require('gm')；
gm('./danny.jpg')
    .resize(50, 50,"!")
    .write('./danny2.jpg', function (err) {
        if (err) {
            console.log(err);
      	 }
    });
```

node.js头像裁切：
```javascript
gm("./danny.jpg").crop(141,96,152,181).write("./2.jpg",function(err){
    //141  96 是宽高 。  152  181是坐标
	});
```

























