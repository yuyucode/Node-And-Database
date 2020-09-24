# express
```最后一次更新笔记时间：2020-09-23```

![三层架构](assets/三层架构.jpg)
### 提问：为什么使用express，而不使用http模块
1、根据不同的请求路径、请求方法、做不同的事情，处理起来比较麻烦  
2、读取请求体和写入响应体是通过流的方式，比较麻烦
```js
const http =require('http')
http.createServer((req, res) =>{
     // 复杂的代码处理请求
})
```

### 第三方库 
- express 
  -  生态环境完整   
  -  民间中文网：https://www.expressjs.com.cn/  
  
- koa2 
  -  基础先进性和提供的API友好性

### 创建一个express应用
```js
const express = require('express');
const app = express();  // 创建一个express应用

// app实际上是一个函数用于处理请求的函数
const port = 9527;  // 监听9527窗口
app.listen(port, () => {
    console.log("Listen 9527 server")
})

// 配置一个请求映射，如果请求方法和请求的路径均满足匹配，交给处理函数进行处理
// app.请求方法("请求路径",处理函数)

// 匹配任何get处理请求
app.get('*', (req, res)=>{
    // req 和 res 是被express封装过的   
  
    res.send("hello但21321是分都是")  // 响应给客户端
})
```

## nodemon 自动重启工具

### 安装

命令行安装
``` 
 局部：$ yarn add nodemon 
 全局：$ yarn add global nodemon 
```

局部安装运行
```
$ npx nodemon index.js
$ 或者自行配置package.json，更改命令
```

全局安装运行
```
需要配置环境变量
$nodemon index.js
```
### 配置
```package.json```
```json
{
 "scripts": {
     "start": "npx nodemon index"
  }
}
```
- npm run start

```nodemon.json```
```json
{
  "env": {
    "NODE_ENV": "development"
  },
  "watch": ["*.json", "*.js"],
  "ignore": ["node_modules", ".idea", "yarn.lock", "package*.json"]
}
```
- watch: 监听的文件类型
- env: 启动的环境
- ignore: 忽略的文件

## express 中间件

![中间件示意图](assets/中间件示意图.jpg)


### 中间件是一个处理函数
- 当匹配到了请求后
  - 交给第一个处理中间件
  - 处理中间件中需要手动的提交给后续中间件处理
  - 想要运行后续的处理函数，需要调用第三个参数next()  
    
#### 中间件处理的细节
- 如果后续已经没有了中间件,判断有没有调用res.end() 或者res.send()   
  - 没有调用： 发送404  
  - 调用： 正常响应  
```js
app.get("/news",
    (req , res, next) =>{
       console.log("第1个处理中间件")
       next();  // 交给后续的处理中间件
    },
    (req , res, next) =>{
       console.log("第2个处理中间件")
       next();  // 交给后续的处理中间件
       // 没有调用res.end() 和 res.send()  
       // 返回 404
    }
)
```

- 如果中间件发生了错误
  - 不会停止服务器
  - 相当于调用了next(错误对象) 
  - 寻找后续的错误处理中间件，如果没有，则相应服务器内部错误 500，  
 
```js
app.get("/news",
    (req , res, next) =>{
       console.log("第1个处理中间件")
       next(new Error("自己写的报错"));  // 交给后续的错误处理中间件
    },
    (req , res, next) =>{
       console.log("第2个处理中间件")
       next();                         // 交给后续的处理中间件
    }
    // ...中间件
)
```

- 当前面的中间件，调用res.end()时  
  - 后续的中间件还是会依次调用(可以做日志记录)  
  - 后续的中间件不能再次调用res.end()和res.send()，会报错
```js
app.get("/news",
    (req , res, next) =>{
       console.log("第1个处理中间件")
       res.status(200)   
       res.end();
       next();  // 交给后续的处理中间件
    },
    (req , res, next) =>{
       //会运行，可以做日志记录
       console.log("第2个处理中间件")  
       next();  // 交给后续的处理中间件
    }
    // ...中间件
)
```

#### 处理错误的中间件

使用```app.use()```方法
 - app.use,不管任何形式的请求，都会调用
 - app.use,采用模糊匹配规则，不和app.get 和app.post 等方法请求一样是精确匹配
```js
// errorMiddleware.js
module.exports = (err, req, res, next) => {
    if (err) {
        // 发生了错误
        const errObj = {
            code: 500,
            msg: err instanceof Error ? err.message : err
        }
        res.status(500).send(errObj)
    }
    else {
        next();
    }
}

// init.js  
// 处理错误的中间件，一般放到最后
// /news 为基本路径， 可以匹配/news、 /newst 、/news/fd 等正则包含/news的路径，不能匹配/n等其他路径
app.use('/news',require('./errorMiidewear'))

// 或者
// 匹配任何路径
app.use(require('./errorMiidewear'))
```

#### 处理静态资源的中间件
```staticMiddleware.js```
```js
// 静态资源

module.exports = (req, res, next) =>{
    if(req.path.startsWith('/api')){
        // 说明你请求的是api接口
        next()
    }else {
        // 判断静态资源是否存在

        if(静态资源){
            res.send("静态资源")
        }else{
            next();
        }
    }
}
```

## express常用中间件

#### express.static() 

- 静态资源服务器
- https://www.expressjs.com.cn/5x/api.html#express.static
```js
// dirname 为静态资源目录

/**
 * 下面的这段代码的作用：
 * 当请求时，会根据请求路径，从指定的目录中寻找是否存在改文件，如果存在，直接响应文件内容，而不再移交给后续的中间件
 * 默认情况下，如果映射的结果是一个目录，则会自动还是用idnex.html文件
 */
app.use(express.static(dirname))

/**
 * 下面的代码作用：
 * 第一个参数为基础路径，模糊匹配 /next等
 * 第二个参数为静态资源陌路
 */
app.use("/next",express.static(dirname))
```



#### express.urlencoded() 

- 获取请求头，匹配类型为 "application/x-www-form-urlencoded"格式  
- https://www.expressjs.com.cn/5x/api.html#express.urlencoded
```js
app.use(
    express.urlencoded({
            extended: true //使用新的库，必须设置
        }
    )
)
```
- 手写
```js
/**
 * 手写 express.urlencoded
 * @param req
 * @param res
 * @param next
 */
const qs = require("querystring")
module.exports = (req, res, next) =>{
    if((req.headers["content-type"]) === "application/x-www-form-urlencoded"){
        // 自行解析消息体
        let result = "";
        req.on("data", chunk =>{
            result += chunk.toString("utf-8")
        });
        req.on("end", ()=>{
            // 把result解析成一个对象，然后放到req.body 
            req.body = qs.parse(result);
            next();
        })
    }else{
        next();
    }

}
```


#### express.json()

- https://www.expressjs.com.cn/5x/api.html#express.json
- 解析JSON格式的消息体
```js
app.use(express.json())
```

## express路由

#### express.router()

- https://www.expressjs.com.cn/5x/api.html#express.router
```js
// routes/init.js 
// 处理api 的请求
app.use('/api/student', require('./api/student'));

// routes/api/student.js
// 设置路由
const express = require('express')
const router = express.Router();

// get -> /api/student
router.get('/', async (req, res) => {
    console.log("获取学生")
    const info = {
        page:req.query.page || 1,
        limit: req.query.limit || 10,
        sex: req.query.sex || -1,
        name: req.query.name || ""
    }
    const data = await studentServ.getStudent(info);
    
    // 举个例子，可以自己封装
    res.send({
        code: 0,
        msg: "",
        data
    })
});
// get -> /api/student/xxx
router.get('/:id', (req, res) => {
    console.log("获取单个学生")
});

// post -> /api/student
router.post('/', (req, res) => {
    console.log("添加学生")
});

// delete ->/api/student:id
router.delete('/:id', (req, res) => {
    console.log('删除学生')
})

// put -> /api/student/xxx
router.put("/:id", (req, res) => {
    console.log("修改学生")
})

module.exports = router
```

## cookie的基本概念
### 一个不大不小的问题

假设服务器有一个接口，通过请求这个接口，可以添加一个管理员

但是，不是任何人都有权力做这种操作的

那么服务器如何知道请求接口的人是有权力的呢？

答案是：只有登录过的管理员才能做这种操作

可问题是，客户端和服务器的传输使用的是http协议，http协议是无状态的，什么叫无状态，就是**服务器不知道这一次请求的人，跟之前登录请求成功的人是不是同一个人**

![](assets/cookie1.png)

![](assets/cookie2.png)

由于http协议的无状态，服务器**忘记**了之前的所有请求，它无法确定这一次请求的客户端，就是之前登录成功的那个客户端。

> 你可以把服务器想象成有着严重脸盲症的东哥，他没有办法分清楚跟他说话的人之前做过什么

于是，服务器想了一个办法

它按照下面的流程来认证客户端的身份

1. 客户端登录成功后，服务器会给客户端一个出入证（令牌 token）
2. 后续客户端的每次请求，都必须要附带这个出入证（令牌 token）

![修饰](assets/cookie3.png)

服务器发扬了认证不认人的优良传统，就可以很轻松的识别身份了。

但是，用户不可能只在一个网站登录，于是客户端会收到来自各个网站的出入证，因此，就要求客户端要有一个类似于卡包的东西，能够具备下面的功能：

1. **能够存放多个出入证**。这些出入证来自不同的网站，也可能是一个网站有多个出入证，分别用于出入不同的地方
2. **能够自动出示出入证**。客户端在访问不同的网站时，能够自动的把对应的出入证附带请求发送出去。
3. **正确的出示出入证**。客户端不能将肯德基的出入证发送给麦当劳。
4. **管理出入证的有效期**。客户端要能够自动的发现那些已经过期的出入证，并把它从卡包内移除。

能够满足上面所有要求的，就是cookie

cookie类似于一个卡包，专门用于存放各种出入证，并有着一套机制来自动管理这些证件。

卡包内的每一张卡片，称之为**一个cookie**。

### cookie的组成

cookie是浏览器中特有的一个概念，它就像浏览器的专属卡包，管理着各个网站的身份信息。

每个cookie就相当于是属于某个网站的一个卡片，它记录了下面的信息：

- key：键，比如「身份编号」
- value：值，比如袁小进的身份编号「14563D1550F2F76D69ECBF4DD54ABC95」，这有点像卡片的条形码，当然，它可以是任何信息
- domain：域，表达这个cookie是属于哪个网站的，比如`yuanjin.tech`，表示这个cookie是属于`yuanjin.tech`这个网站的
- path：路径，表达这个cookie是属于该网站的哪个基路径的，就好比是同一家公司不同部门会颁发不同的出入证。比如`/news`，表示这个cookie属于`/news`这个路径的。（后续详细解释）
- secure：是否使用安全传输（后续详细解释）
- expire：过期时间，表示该cookie在什么时候过期

当浏览器向服务器发送一个请求的时候，它会瞄一眼自己的卡包，看看哪些卡片适合附带捎给服务器

如果一个cookie**同时满足**以下条件，则这个cookie会被附带到请求中

- cookie没有过期
- cookie中的域和这次请求的域是匹配的
  - 比如cookie中的域是`yuanjin.tech`，则可以匹配的请求域是`yuanjin.tech`、`www.yuanjin.tech`、`blogs.yuanjin.tech`等等
  - 比如cookie中的域是`www.yuanjin.tech`，则只能匹配`www.yuanjin.tech`这样的请求域
  - cookie是不在乎端口的，只要域匹配即可
- cookie中的path和这次请求的path是匹配的
  - 比如cookie中的path是`/news`，则可以匹配的请求路径可以是`/news`、`/news/detail`、`/news/a/b/c`等等，但不能匹配`/blogs`
  - 如果cookie的path是`/`，可以想象，能够匹配所有的路径
- 验证cookie的安全传输
  - 如果cookie的secure属性是true，则请求协议必须是`https`，否则不会发送该cookie
  - 如果cookie的secure属性是false，则请求协议可以是`http`，也可以是`https`

如果一个cookie满足了上述的所有条件，则浏览器会把它自动加入到这次请求中

具体加入的方式是，**浏览器会将符合条件的cookie，自动放置到请求头中**，例如，当我在浏览器中访问百度的时候，它在请求头中附带了下面的cookie：

![](assets/4.png)

看到打马赛克的地方了吗？这部分就是通过请求头`cookie`发送到服务器的，它的格式是`键=值; 键=值; 键=值; ...`，每一个键值对就是一个符合条件的cookie。

**cookie中包含了重要的身份信息，永远不要把你的cookie泄露给别人！！！**否则，他人就拿到了你的证件，有了证件，就具备了为所欲为的可能性。

### 如何设置cookie

由于cookie是保存在浏览器端的，同时，很多证件又是服务器颁发的

所以，cookie的设置有两种模式：

- 服务器响应：这种模式是非常普遍的，当服务器决定给客户端颁发一个证件时，它会在响应的消息中包含cookie，浏览器会自动的把cookie保存到卡包中
- 客户端自行设置：这种模式少见一些，不过也有可能会发生，比如用户关闭了某个广告，并选择了「以后不要再弹出」，此时就可以把这种小信息直接通过浏览器的JS代码保存到cookie中。后续请求服务器时，服务器会看到客户端不想要再次弹出广告的cookie，于是就不会再发送广告过来了。

#### 服务器端设置cookie

服务器可以通过设置响应头，来告诉浏览器应该如何设置cookie

响应头按照下面的格式设置：

```js
// set-cookis: cookie 
res.header('set-cookie', "cookie1")  
res.header('set-cookie', "cookie2")  
res.header('set-cookie', "cookie3")  
```

通过这种模式，就可以在一次响应中设置多个cookie了，具体设置多少个cookie，设置什么cookie，根据你的需要自行处理

其中，每个cookie的格式如下：

```
键=值; path=?; domain=?; expire=?; max-age=?; secure; httponly
```

每个cookie除了键值对是必须要设置的，其他的属性都是可选的，并且顺序不限

当这样的响应头到达客户端后，**浏览器会自动的将cookie保存到卡包中，如果卡包中已经存在一模一样的卡片（其他key、path、domain相同），则会自动的覆盖之前的设置**。

下面，依次说明每个属性值：

- **path**：设置cookie的路径。如果不设置，浏览器会将其自动设置为当前请求的路径。比如，浏览器请求的地址是`/login`，服务器响应了一个`set-cookie: a=1`，浏览器会将该cookie的path设置为请求的路径`/login`
- **domain**：设置cookie的域。如果不设置，浏览器会自动将其设置为当前的请求域，比如，浏览器请求的地址是`http://www.yuanjin.tech`，服务器响应了一个`set-cookie: a=1`，浏览器会将该cookie的domain设置为请求的域`www.yuanjin.tech`
  - 这里值得注意的是，如果服务器响应了一个无效的域，浏览器是不认的
  - 什么是无效的域？就是响应的域连根域都不一样。比如，浏览器请求的域是`yuanjin.tech`，服务器响应的cookie是`set-cookie: a=1; domain=baidu.com`，这样的域浏览器是不认的。
  - 如果浏览器连这样的情况都允许，就意味着张三的服务器，有权利给用户一个cookie，用于访问李四的服务器，这会造成很多安全性的问题
- **expire**：设置cookie的过期时间。这里必须是一个有效的GMT时间，即格林威治标准时间字符串，比如`Fri, 17 Apr 2020 09:35:59 GMT`，表示格林威治时间的`2020-04-17 09:35:59`，即北京时间的`2020-04-17 17:35:59`。当客户端的时间达到这个时间点后，会自动销毁该cookie。
- **max-age**：设置cookie的相对有效期。expire和max-age通常仅设置一个即可。比如设置`max-age`为`1000`，浏览器在添加cookie时，会自动设置它的`expire`为当前时间加上1000秒，作为过期时间。
  - 如果不设置expire，又没有设置max-age，则表示会话结束后过期。
  - 对于大部分浏览器而言，关闭所有浏览器窗口意味着会话结束。
- **secure**：设置cookie是否是安全连接。如果设置了该值，则表示该cookie后续只能随着`https`请求发送。如果不设置，则表示该cookie会随着所有请求发送。
- **httponly**：设置cookie是否仅能用于传输。如果设置了该值，表示该cookie仅能用于传输，而不允许在客户端通过JS获取，这对防止跨站脚本攻击（XSS）会很有用。 
  - 关于如何通过JS获取，后续会讲解
  - 关于什么是XSS，不在本文讨论范围

下面来一个例子，客户端通过`post`请求服务器`http://yuanjin.tech/login`，并在消息体中给予了账号和密码，服务器验证登录成功后，在响应头中加入了以下内容：

```yaml
res.header('set-cookie': 'token=123456; path=/; max-age=3600; httponly')
// set-cookie: token=123456; path=/; max-age=3600; httponly
```

当该响应到达浏览器后，浏览器会创建下面的cookie：

```yaml
key: token
value: 123456
domain: yuanjin.tech
path: /
expire: 2020-04-17 18:55:00 #假设当前时间是2020-04-17 17:55:00
secure: false  #任何请求都可以附带这个cookie，只要满足其他要求
httponly: true #不允许JS获取该cookie
```

于是，随着浏览器后续对服务器的请求，只要满足要求，这个cookie就会被附带到请求头中传给服务器：

```yaml
cookie: token=123456; 其他cookie...
```

现在，还剩下最后一个问题，就是如何删除浏览器的一个cookie呢？

如果要删除浏览器的cookie，只需要让服务器响应一个同样的域、同样的路径、同样的key，只是时间过期的cookie即可

**所以，删除cookie其实就是修改cookie**

下面的响应会让浏览器删除`token`

```yaml
cookie: token=; domain=yuanjin.tech; path=/; max-age=-1
```

浏览器按照要求修改了cookie后，会发现cookie已经过期，于是自然就会删除了。

> 无论是修改还是删除，都要注意cookie的域和路径，因为完全可能存在域或路径不同，但key相同的cookie
>
> 因此无法仅通过key确定是哪一个cookie

#### 客户端设置cookie

既然cookie是存放在浏览器端的，所以浏览器向JS公开了接口，让其可以设置cookie

```js
document.cookie = "键=值; path=?; domain=?; expire=?; max-age=?; secure";
```

可以看出，在客户端设置cookie，和服务器设置cookie的格式一样，只是有下面的不同

- 没有httponly。因为httponly本来就是为了限制在客户端访问的，既然你是在客户端配置，自然失去了限制的意义。
- path的默认值。在服务器端设置cookie时，如果没有写path，使用的是请求的path。而在客户端设置cookie时，也许根本没有请求发生。因此，path在客户端设置时的默认值是当前网页的path
- domain的默认值。和path同理，客户端设置时的默认值是当前网页的domain
- 其他：一样
- 删除cookie：和服务器也一样，修改cookie的过期时间即可

### 总结

以上，就是cookie原理部分的内容。

如果把它用于登录场景，就是如下的流程：

**登录请求**

1. 浏览器发送请求到服务器，附带账号密码
2. 服务器验证账号密码是否正确，如果不正确，响应错误，如果正确，在响应头中设置cookie，附带登录认证信息（至于登录认证信息是设么样的，如何设计，要考虑哪些问题，就是另一个话题了，可以百度 jwt）
3. 客户端收到cookie，浏览器自动记录下来



**后续请求**

1. 浏览器发送请求到服务器，希望添加一个管理员，并将cookie自动附带到请求中
2. 服务器先获取cookie，验证cookie中的信息是否正确，如果不正确，不予以操作，如果正确，完成正常的业务流程

## 登录和认证

### 使用cookie-parser中间件
- 加入```cookie-parser```中间件
- 加入之后，会在req对象中注入cookies属性，用于获取所有请求传递过来的cookie
- 加入之后，会在res对中注入cookie方法， 用于设置cookie
```js
const cookieParser = requier('cookie-parser');
app.use(cookieParser())

app.use((req,res) => {
   res.cookie("token", "唯一令牌字符串", {
      path: '/',  // 路径
      domain: "localhost", // 域 
      maxAge: 7 * 24 * 3600 * 1000, //毫秒数
      httpOnly: true
   })
})
```
### 登录成功后给予token

- 通过cookie给予： 适配浏览器
```js
const cookieParser = requier('cookie-parser');
app.use(cookieParser())

app.use((req,res) => {
   var result = "唯一令牌字符串";

   res.cookie("token",  result , {
      path: '/',  // 路径
      domain: "localhost", // 域 
      maxAge: 7 * 24 * 3600 * 1000, //毫秒数
      httpOnly: true
   })
})
```
- 通过header给予： 适配其他终端
```js
res.header("authorization", result )
```

- 对后续请求进行认证
  + 解析cookie或者header中的token
  + 验证token  
    * 通过：继续后续处理  
    * 未通过：给予错误 
         
```tokenMiddleware.js```
```js
module.exports = (req, res, next) => {
    let token = req.cookies.token;
    if(!token){
        // 从header的authorization中获取
        token = req.headers.authorization;
    }
    if(!token){
        // 没有认证
        handleNoToken(req, res, next)
    }

};

// 处理没有认证的情况
function handleNoToken(req, res, next){
    
}
```
### 对称加密 aes 128
- 对称加密token
- 128位的密钥

```js
// 使用对称加密算法： aes 128
// 128 位的密钥

const secret = Buffer.from("mm7h3ck87ugk9l4a");
const crypto = require("crypto");

// 准备一个iv， 随机向量
const iv = Buffer.from("7uzue51hw7dsxe0b");


exports.encrypt = function (str){
    const cry = crypto.createCipheriv("aes-128-cbc", secret, iv);
    // 第一个参数 加密的数据
    // 第二个参数 传入加密数据是什么类型，
    // 第三个参数 加密数据之后的输出类型
    let result = cry.update(str, "utf-8", "hex");
    result += cry.final("hex"); // 输出的encoding
    return result;
}

exports.decrypt = function (str){
    // str 加密过后的字符串
    const decry = crypto.createDecipheriv("aes-128-cbc", secret, iv);
    // 第一个参数 解密的数据
    // 第二个参数 传入解密数据是什么类型，
    // 第三个参数 解密数据之后的输出类型
    let result = decry.update(str, "hex", "utf-8")
    result += decry.final("utf-8");  // 输出的encoding
    return result;
}

```
### node 断点调试

**node --inspect启动模块**  
**node进程会监听9299端口**


## 跨域值JSONP

- 同源策略
  - 同源
    1. 协议
    2. 端口
    3. 主机名
    4. 完全相同
  - 浏览器不允许使用非同源的数据

- 解决方案
  - JSONP
  - CORS
- JSONP  
  - 1、浏览器端生成一个script元素，访问数据接口
  - 2、服务器响应一段JS代码，调用某一个函数，并把响应的数据传入
- JSONP的缺陷
  - 1、会严重影响服务器的正常响应格式
  - 2、只能使用GET请求
  
## 跨域之CORS

> 阅读本文，你需要首先知道：
>
> 1. 浏览器的同源策略
> 2. 跨域问题
> 3. JSONP原理
> 4. cookie原理

JSONP并不是一个好的跨域解决方案，它至少有着下面两个严重问题：

1. **会打乱服务器的消息格式**：JSONP要求服务器响应一段JS代码，但在非跨域的情况下，服务器又需要响应一个正常的JSON格式
2. **只能完成GET请求**：JSONP的原理会要求浏览器端生成一个`script`元素，而`script`元素发出的请求只能是`get`请求

所以，CORS是一种更好的跨域解决方案。

### 概述

`CORS`是基于`http1.1`的一种跨域解决方案，它的全称是**C**ross-**O**rigin **R**esource **S**haring，跨域资源共享。

它的总体思路是：**如果浏览器要跨域访问服务器的资源，需要获得服务器的允许**

![image-20200421152122793](assets/3.png)

而要知道，一个请求可以附带很多信息，从而会对服务器造成不同程度的影响

比如有的请求只是获取一些新闻，有的请求会改动服务器的数据

针对不同的请求，CORS规定了三种不同的交互模式，分别是：

- **简单请求**
- **需要预检的请求**
- **附带身份凭证的请求**

这三种模式从上到下层层递进，请求可以做的事越来越多，要求也越来越严格。

下面分别说明三种请求模式的具体规范。

### 简单请求

当浏览器端运行了一段ajax代码（无论是使用XMLHttpRequest还是fetch api），浏览器会首先判断它属于哪一种请求模式

#### 简单请求的判定

当请求**同时满足**以下条件时，浏览器会认为它是一个简单请求：

1. **请求方法属于下面的一种：**
   - get
   - post
   - head
2. **请求头仅包含安全的字段，常见的安全字段如下：**
   - `Accept`
   - `Accept-Language`
   - `Content-Language`
   - `Content-Type`
   - `DPR`
   - `Downlink`
   - `Save-Data`
   - `Viewport-Width`
   - `Width`

3. **请求头如果包含`Content-Type`，仅限下面的值之一：**
   - `text/plain`
   - `multipart/form-data`
   - `application/x-www-form-urlencoded`

如果以上三个条件同时满足，浏览器判定为简单请求。

下面是一些例子：

```js
// 简单请求
fetch("http://crossdomain.com/api/news");

// 请求方法不满足要求，不是简单请求
fetch("http://crossdomain.com/api/news", {
  method:"PUT"
})

// 加入了额外的请求头，不是简单请求
fetch("http://crossdomain.com/api/news", {
  headers:{
    a: 1
  }
})

// 简单请求
fetch("http://crossdomain.com/api/news", {
  method: "post"
})

// content-type不满足要求，不是简单请求
fetch("http://crossdomain.com/api/news", {
  method: "post",
  headers: {
    "content-type": "application/json"
  }
})
```

#### 简单请求的交互规范

当浏览器判定某个**ajax跨域请求**是**简单请求**时，会发生以下的事情

1. **请求头中会自动添加`Origin`字段**

比如，在页面`http://my.com/index.html`中有以下代码造成了跨域

```js
// 简单请求
fetch("http://crossdomain.com/api/news");
```

请求发出后，请求头会是下面的格式：

```
GET /api/news/ HTTP/1.1
Host: crossdomain.com
Connection: keep-alive
...
Referer: http://my.com/index.html
Origin: http://my.com
```

看到最后一行没，`Origin`字段会告诉服务器，是哪个源地址在跨域请求

2. **服务器响应头中应包含`Access-Control-Allow-Origin`**

当服务器收到请求后，如果允许该请求跨域访问，需要在响应头中添加`Access-Control-Allow-Origin`字段

该字段的值可以是：

- *：表示我很开放，什么人我都允许访问
- 具体的源：比如`http://my.com`，表示我就允许你访问

> 实际上，这两个值对于客户端`http://my.com`而言，都一样，因为客户端才不会管其他源服务器允不允许，就关心自己是否被允许
>
> 当然，服务器也可以维护一个可被允许的源列表，如果请求的`Origin`命中该列表，才响应`*`或具体的源
>
> **为了避免后续的麻烦，强烈推荐响应具体的源**

假设服务器做出了以下的响应：

```
HTTP/1.1 200 OK
Date: Tue, 21 Apr 2020 08:03:35 GMT
...
Access-Control-Allow-Origin: http://my.com
...

消息体中的数据
```

当浏览器看到服务器允许自己访问后，高兴的像一个两百斤的孩子，于是，它就把响应顺利的交给js，以完成后续的操作

下图简述了整个交互过程

![image-20200421162846480](assets/jiaohu1.png)

#### 需要预检的请求

简单的请求对服务器的威胁不大，所以允许使用上述的简单交互即可完成。

但是，如果浏览器不认为这是一种简单请求，就会按照下面的流程进行：

1. **浏览器发送预检请求，询问服务器是否允许**
2. **服务器允许**
3. **浏览器发送真实请求**
4. **服务器完成真实的响应**

比如，在页面`http://my.com/index.html`中有以下代码造成了跨域

```js
// 需要预检的请求
fetch("http://crossdomain.com/api/user", {
  method:"POST", // post 请求
  headers:{  // 设置请求头
    a: 1,
    b: 2,
    "content-type": "application/json"
  },
  body: JSON.stringify({ name: "袁小进", age: 18 }) // 设置请求体
});
```

浏览器发现它不是一个简单请求，则会按照下面的流程与服务器交互

1. **浏览器发送预检请求，询问服务器是否允许**

```
OPTIONS /api/user HTTP/1.1
Host: crossdomain.com
...
Origin: http://my.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: a, b, content-type
```

可以看出，这并非我们想要发出的真实请求，请求中不包含我们的响应头，也没有消息体。

这是一个预检请求，它的目的是询问服务器，是否允许后续的真实请求。

预检请求**没有请求体**，它包含了后续真实请求要做的事情

预检请求有以下特征：

- 请求方法为`OPTIONS`
- 没有请求体
- 请求头中包含
  - `Origin`：请求的源，和简单请求的含义一致
  - `Access-Control-Request-Method`：后续的真实请求将使用的请求方法
  - `Access-Control-Request-Headers`：后续的真实请求会改动的请求头

2. **服务器允许**

服务器收到预检请求后，可以检查预检请求中包含的信息，如果允许这样的请求，需要响应下面的消息格式

```
HTTP/1.1 200 OK
Date: Tue, 21 Apr 2020 08:03:35 GMT
...
Access-Control-Allow-Origin: http://my.com
Access-Control-Allow-Methods: POST
Access-Control-Allow-Headers: a, b, content-type
Access-Control-Max-Age: 86400
...
```

对于预检请求，不需要响应任何的消息体，只需要在响应头中添加：

- `Access-Control-Allow-Origin`：和简单请求一样，表示允许的源
- `Access-Control-Allow-Methods`：表示允许的后续真实的请求方法
- `Access-Control-Allow-Headers`：表示允许改动的请求头
- `Access-Control-Max-Age`：告诉浏览器，多少秒内，对于同样的请求源、方法、头，都不需要再发送预检请求了

3. **浏览器发送真实请求**

预检被服务器允许后，浏览器就会发送真实请求了，上面的代码会发生下面的请求数据

```
POST /api/user HTTP/1.1
Host: crossdomain.com
Connection: keep-alive
...
Referer: http://my.com/index.html
Origin: http://my.com

{"name": "袁小进", "age": 18 }
```

4. **服务器响应真实请求**

```
HTTP/1.1 200 OK
Date: Tue, 21 Apr 2020 08:03:35 GMT
...
Access-Control-Allow-Origin: http://my.com
...

添加用户成功
```



可以看出，当完成预检之后，后续的处理与简单请求相同

下图简述了整个交互过程

![image-20200421165913320](assets/jiaohu2.png)

#### 附带身份凭证的请求

默认情况下，ajax的跨域请求并不会附带cookie，这样一来，某些需要权限的操作就无法进行

不过可以通过简单的配置就可以实现附带cookie

```js
// xhr
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;

// fetch api
fetch(url, {
  credentials: "include"
})
```

这样一来，该跨域的ajax请求就是一个*附带身份凭证的请求*

当一个请求需要附带cookie时，无论它是简单请求，还是预检请求，都会在请求头中添加`cookie`字段

而服务器响应时，需要明确告知客户端：服务器允许这样的凭据

告知的方式也非常的简单，只需要在响应头中添加：`Access-Control-Allow-Credentials: true`即可

对于一个附带身份凭证的请求，若服务器没有明确告知，浏览器仍然视为跨域被拒绝。

另外要特别注意的是：**对于附带身份凭证的请求，服务器不得设置 `Access-Control-Allow-Origin 的值为*`**。这就是为什么不推荐使用*的原因

#### 一个额外的补充

在跨域访问时，JS只能拿到一些最基本的响应头，如：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma，如果要访问其他头，则需要服务器设置本响应头。

`Access-Control-Expose-Headers`头让服务器把允许浏览器访问的头放入白名单，例如：

```
Access-Control-Expose-Headers: authorization, a, b
```

这样JS就能够访问指定的响应头了。

## CORS 中间件
https://github.com/expressjs/cors
```yarn add cors```
```js
app.use(cors({
  // 配置
}))
```


## session 

- cookie 
  - 存储在客户端
  - 优点
    - 存储在客户端，不占用服务器资源
  - 缺点
    - 只能是字符串格式
    - 存储量有限
    - 数据容易被获取
    - 数据容易被篡改
    - 容易丢失

- session
  - 存储在服务器端
  - 优点
    - 可以是任何格式
    - 存储量理论上是无限的
    - 数据难以被获取
    - 数据难以篡改
    - 不易丢失
  - 缺点
    - 占用服务器资源
    
![](assets/session1.jpg)    

```yarn add express-session```
```js
const session = require("express-session")
app.use(session({
   sercet: "werewrrwe",
   name: "sessionId"
}))
```


## jwt

随着前后端分离的发展，以及数据中心的建立，越来越多的公司会创建一个中心服务器，服务于各种产品线。

而这些产品线上的产品，它们可能有着各种终端设备，包括但不仅限于浏览器、桌面应用、移动端应用、平板应用、甚至智能家居

![image-20200422163727151](assets/jwt1.png)

> 实际上，不同的产品线通常有自己的服务器，产品内部的数据一般和自己的服务器交互。
>
> 但中心服务器仍然有必要存在，因为同一家公司的产品总是会存在共享的数据，比如用户数据

这些设备与中心服务器之间会进行http通信

一般来说，中心服务器至少承担着认证和授权的功能，例如登录：各种设备发送消息到中心服务器，然后中心服务器响应一个身份令牌（参见[cookie原理详解](http://www.yuanjin.tech/article/98)）

当这种结构出现后，就出现一个问题：它们之间还能使用传统的cookie方式传递令牌信息吗？

其实，也是可以的🐶，因为cookie在传输中无非是一个消息头而已，只不过浏览器对这个消息头有特殊处理罢了。

但浏览器之外的设备肯定不喜欢cookie，因为浏览器有着对cookie完善的管理机制，但是在其他设备上，就需要开发者自己手动处理了

jwt的出现就是为了解决这个问题

### 概述

jwt全称`Json Web Token`，强行翻译过来就是`json格式的互联网令牌`（算了，还是不要强行翻译了🐷）

它要解决的问题，就是为多种终端设备，提供**统一的、安全的**令牌格式

![image-20200422165350268](assets/jwt2.png)

因此，jwt只是一个令牌格式而已，你可以把它存储到cookie，也可以存储到localstorage，没有任何限制！

同样的，对于传输，你可以使用任何传输方式来传输jwt，一般来说，我们会使用消息头来传输它

比如，当登录成功后，服务器可以给客户端响应一个jwt：

```
HTTP/1.1 200 OK
...
set-cookie:token=jwt令牌
authorization:jwt令牌
...

{..., token:jwt令牌}
```

可以看到，jwt令牌可以出现在响应的任何一个地方，客户端和服务器自行约定即可。

> 当然，它也可以出现在响应的多个地方，比如为了充分利用浏览器的cookie，同时为了照顾其他设备，也可以让jwt出现在`set-cookie`和`authorization或body`中，尽管这会增加额外的传输量。

当客户端拿到令牌后，它要做的只有一件事：存储它。

你可以存储到任何位置，比如手机文件、PC文件、localstorage、cookie

当后续请求发生时，你只需要将它作为请求的一部分发送到服务器即可。

虽然jwt没有明确要求应该如何附带到请求中，但通常我们会使用如下的格式：

```
GET /api/resources HTTP/1.1
...
authorization: bearer jwt令牌
...
```

> 这种格式是OAuth2附带token的一种规范格式
>
> 至于什么是OAuth2，那是另一个话题了

这样一来，服务器就能够收到这个令牌了，通过对令牌的验证，即可知道该令牌是否有效。

它们的完整交互流程是非常简单清晰的

![image-20200422172837190](assets/jwt3.png)

### 令牌的组成

为了保证令牌的安全性，jwt令牌由三个部分组成，分别是：

1. header：令牌头部，记录了整个令牌的类型和签名算法
2. payload：令牌负荷，记录了保存的主体信息，比如你要保存的用户信息就可以放到这里
3. signature：令牌签名，按照头部固定的签名算法对整个令牌进行签名，该签名的作用是：保证令牌不被伪造和篡改

它们组合而成的完整格式是：`header.payload.signature`

比如，一个完整的jwt令牌如下：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9.BCwUy3jnUQ_E6TqCayc7rCHkx-vxxdagUwPOWqwYCFc
```

它各个部分的值分别是：

- `header：eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`
- `payload：eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9`
- `signature: BCwUy3jnUQ_E6TqCayc7rCHkx-vxxdagUwPOWqwYCFc`

下面分别对每个部分进行说明

### header

它是令牌头部，记录了整个令牌的类型和签名算法

它的格式是一个`json`对象，如下：

```json
{
  "alg":"HS256",
  "typ":"JWT"
}
```

该对象记录了：

- alg：signature部分使用的签名算法，通常可以取两个值
  - HS256：一种对称加密算法，使用同一个秘钥对signature加密解密
  - RS256：一种非对称加密算法，使用私钥加密，公钥解密
- typ：整个令牌的类型，固定写`JWT`即可

设置好了`header`之后，就可以生成`header`部分了

具体的生成方式及其简单，就是把`header`部分使用`base64 url`编码即可

> `base64 url`不是一个加密算法，而是一种编码方式，它是在`base64`算法的基础上对`+`、`=`、`/`三个字符做出特殊处理的算法
>
> 而`base64`是使用64个可打印字符来表示一个二进制数据，具体的做法参考[百度百科](https://baike.baidu.com/item/base64/8545775?fr=aladdin)

浏览器提供了`btoa`函数，可以完成这个操作：

```js
window.btoa(JSON.stringify({
  "alg":"HS256",
  "typ":"JWT"
}))
// 得到字符串：eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

同样的，浏览器也提供了`atob`函数，可以对其进行解码：

```js
window.atob("eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9")
// 得到字符串：{"alg":"HS256","typ":"JWT"}
```

> nodejs中没有提供这两个函数，可以安装第三方库`atob`和`bota`搞定
>
> 或者，手动搞定

### payload

这部分是jwt的主体信息，它仍然是一个JSON对象，它可以包含以下内容：

```json
{
  "ss"："发行者",
	"iat"："发布时间",
	"exp"："到期时间",
	"sub"："主题",
	"aud"："听众",
	"nbf"："在此之前不可用",
  "jti"："JWT ID"
}
```

以上属性可以全写，也可以一个都不写，它只是一个规范，就算写了，也需要你在将来验证这个jwt令牌时手动处理才能发挥作用

上述属性表达的含义分别是：

- ss：发行该jwt的是谁，可以写公司名字，也可以写服务名称
- iat：该jwt的发放时间，通常写当前时间的时间戳
- exp：该jwt的到期时间，通常写时间戳
- sub：该jwt是用于干嘛的
- aud：该jwt是发放给哪个终端的，可以是终端类型，也可以是用户名称，随意一点
- nbf：一个时间点，在该时间点到达之前，这个令牌是不可用的
- jti：jwt的唯一编号，设置此项的目的，主要是为了防止重放攻击（重放攻击是在某些场景下，用户使用之前的令牌发送到服务器，被服务器正确的识别，从而导致不可预期的行为发生）

可是到现在，看了半天，没有出现我想要写入的数据啊😂

当用户登陆成功之后，我可能需要把用户的一些信息写入到jwt令牌中，比如用户id、账号等等（密码就算了😳）

其实很简单，payload这一部分只是一个json对象而已，你可以向对象中加入任何想要加入的信息

比如，下面的json对象仍然是一个有效的payload

```json
{
  "foo":"bar",
  "iat":1587548215
}
```

`foo: bar`是我们自定义的信息，`iat: 1587548215`是jwt规范中的信息

最终，payload部分和header一样，需要通过`base64 url`编码得到：

```js
window.btoa(JSON.stringify({
  "foo":"bar",
  "iat":1587548215
}))
// 得到字符串：eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9
```

### signature

这一部分是jwt的签名，正是它的存在，保证了整个jwt不被篡改

这部分的生成，是对前面两个部分的编码结果，按照头部指定的方式进行加密

比如：头部指定的加密方法是`HS256`，前面两部分的编码结果是`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9`

则第三部分就是用对称加密算法`HS256`对字符串`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9`进行加密，当然你得指定一个秘钥，比如`shhhhh`

```js
HS256(`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9`, "shhhhh")
// 得到：BCwUy3jnUQ_E6TqCayc7rCHkx-vxxdagUwPOWqwYCFc
```

最终，将三部分组合在一起，就得到了完整的jwt

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9.BCwUy3jnUQ_E6TqCayc7rCHkx-vxxdagUwPOWqwYCFc
```

由于签名使用的秘钥保存在服务器，这样一来，客户端就无法伪造出签名，因为它拿不到秘钥。

换句话说，之所以说无法伪造jwt，就是因为第三部分的存在。

而前面两部分并没有加密，只是一个编码结果而已，可以认为几乎是明文传输

> 这不会造成太大的问题，因为既然用户登陆成功了，它当然有权力查看自己的用户信息
>
> 甚至在某些网站，用户的基本信息可以被任何人查看
>
> 你要保证的，是不要把敏感的信息存放到jwt中，比如密码

jwt的`signature`可以保证令牌不被伪造，那如何保证令牌不被篡改呢？

比如，某个用户登陆成功了，获得了jwt，但他人为的篡改了`payload`，比如把自己的账户余额修改为原来的两倍，然后重新编码出`payload`发送到服务器，服务器如何得知这些信息被篡改过了呢？

这就要说到令牌的验证了

### 令牌的验证

![image-20200422172837190](assets/jwt4.png)

令牌在服务器组装完成后，会以任意的方式发送到客户端

客户端会把令牌保存起来，后续的请求会将令牌发送给服务器

而服务器需要验证令牌是否正确，如何验证呢？

首先，服务器要验证这个令牌是否被篡改过，验证方式非常简单，就是对`header+payload`用同样的秘钥和加密算法进行重新加密

然后把加密的结果和传入jwt的`signature`进行对比，如果完全相同，则表示前面两部分没有动过，就是自己颁发的，如果不同，肯定是被篡改过了。

```
传入的header.传入的payload.传入的signature
新的signature = header中的加密算法(传入的header.传入的payload, 秘钥)
验证：新的signature == 传入的signature
```

当令牌验证为没有被篡改后，服务器可以进行其他验证：比如是否过期、听众是否满足要求等等，这些就视情况而定了

注意：这些验证都需要服务器手动完成，没有哪个服务器会给你进行自动验证，当然，你可以借助第三方库来完成这些操作

### 总结

最后，总结一下jwt的特点：

- jwt本质上是一种令牌格式。它和终端设备无关，同样和服务器无关，甚至与如何传输无关，它只是规范了令牌的格式而已
- jwt由三部分组成：header、payload、signature。主体信息在payload
- jwt难以被篡改和伪造。这是因为有第三部分的签名存在。


## 登录和认证-服务器开发

#### jsonwebtoken库
```js
const jwt = require('jsonwebtoken')
const secrect = "余榆";
const token = jwt.sign(
    {
        id: 1,
        name: "成哥",
    },
    secrect,
    {
        expiresIn: 3600
    });


// 只是解密不验证
jwt.decode(token)

// 解密+验证
jwt.verify(token, secrect)

```

#### 颁发jwt

- 确定过期时间
- 确定主体
- 确定密钥
- 确定传输方式
  - cookie (req.cookies.token)
  - authorization  (req.headers.authorization)
    - 带bearer
    - 不带bearer

```js
const secrect = "yuyu"; //密钥
const cookieKey = "token"
const jwt =require("jsonwebtoken")
// 颁发jwt
exports.publish = function (res, maxAge = 3600 * 24, info={}){
    const token = jwt.sign(info, secrect, {
        expiresIn: maxAge
    });
    // 添加到cookie 原生就有
    res.cookie(cookieKey, token, {
        maxAge: maxAge * 1000,
        path:"/"
    })

    // 添加其他传输
    res.header('authorization', token)
}

```


#### 认证jwt

```js
// 认证jwt


exports.verify = function (req){
    let token;
    // 先尝试从cookie中获取
    token = req.cookies[cookieKey];

    // cookie没有
    if(!token){
        // 尝试从header中获取
        token = req.headers.authorization
        if(!token){
            // 还是没有token
            return null
        }
        token = token.split(" ");
        token = token.length === 1 ? token[0] : token [1];
    }
    try{
        return jwt.verify(token, secrect)
    }catch {
        return  null
    }
}
```

#### 添加whoami接口



## 登录和认证-客户端开发

封装axios
```js
// 1、发送请求的时候，如果有token，需要附带到请求头中
// 2、响应的时候，如果有token,保存token到本地（localstorage）
// 3、响应的时候，如果响应的消息码是403（没有token，token失效）本地删除token


import axios from 'axios'
export default function (){
    // 1、
    const token = localStorage.getItem("token");
    let instance = axios;
    if(token){
        instance = axios.create({
            headers:{
                authorization: "bearer " + token;
            }
        })
    }

    instance.interceptors.response.use(resp=>{
        // 2、
        if(resp.headers.authorization){
            localStorage.setItem("token", resp.headers.authorization);
        }
        return resp
    }, error => {
        // 3、
        if(error.response.status === 403){
            localStorage.removeItem('token');
        }
        return Promise.reject(error)
    })
    return instance
}
```

