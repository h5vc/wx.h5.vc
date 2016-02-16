---
date: 2016-02-16 18:43
status: public
title: '[译]使用Node和Express4开发一套RESTful API'
---

前几天 [Express 4.0](http://expressjs.com/4x/api.html) 发布了（译者注：原文是2014年8月15日写的），我们的很多 Node 应用都会在处理路由这部分有一些变化了。基于 [ Express Router](http://expressjs.com/4x/api.html#router) 的改进，我们可以在定义业务路由的时候更加灵活了。

今天我们将会一起来看看怎样使用Node创建一套 RESTful API,借助[Express 4 和它的路由控制](https://scotch.io/tutorials/javascript/learn-to-use-the-new-router-in-expressjs-4)，结合 Mongoose 和 MongoDB 进行数据交互。我们也会使用 [Postman](https://chrome.google.com/webstore/detail/postman-rest-client-packa/fhbjgbiflinjbdggehcddcbncdddomop) 来在 Chrome 里测试我们创建的API。

让我们一起来看看我们将要创建的API到底能做什么 。

## 我们的应用
我们将要创建一套具备如下功能的API：
* 处理一种条目的增删改查（本例中假设是bears）
* 拥有标准的URL（类似 http://example.com/api/bears 和 http://example.com/api/bears/:bear_id）
* 使用对应的 HTTP 方法以便 RESTful（`GET`,`POST`,`PUT`和`DELETE`）
* 返回 JSON 数据
* 在控制台里面输出所有请求的日志

所有这些都是非常标准的 [ RESTful APIs](https://scotch.io/bar-talk/designing-a-restful-web-api) 特性。你可以很轻松地将我们例子中的 bears 换成其他你需要的任何角色（比如像 users,superheros,beers,等等）

## 让我们开始吧
让我们来看看总共需要创建哪些文件。我们需要定义我们的 Node packages, 使用 Express 提供服务器支持, 定义数据模型, 使用Express定义路由, 最后必不可少的， 测试我们的API。

这是我们的文件结构。我们不需要太多的文件，我们会保持简单以便演示。当你要用于生产或者开发大型应用时，你可能需要拆分它们到一个更科学的结构里面（比如路由定义在单独的文件里面）。
```shell
- app/
    ----- models/
    ---------- bear.js  // our bear model
    - node_modules/     // created by npm. holds our dependencies/packages
    - package.json      // define all our node app and dependencies
    - server.js         // configure our application and create routes
```
### 定义包依赖 <small>package.json</small>
就像其他所有的Node项目一样，我们也需要通过`package.json`来定义我们依赖哪些包。创建完成之后看上去应该是这样的：
```js
// package.json

{
    "name": "node-api",
    "main": "server.js",
    "dependencies": {
        "express": "~4.0.0",
        "mongoose": "~3.6.13",
        "body-parser": "~1.0.1"
    }
}
```
这些包都是干啥的？`express`是Node的开发框架。`mongoose` 是我们用来跟 MongoDB数据库进行通信的ORM。`body-parser`让我们可以解析客户端通过HTTP协议POST过来的内容以便我们创建一条 bear。

### 安装我们的依赖
这应该是最简单的步骤了。在命令行终端里面切换到项目的根目录后输入：
```shell
$ npm install
```
npm会安装所有的依赖到一个名为`node_modules`的文件夹里。

`npm` 是Node的包管理器，会帮我们管理 `package.json` 里面声明的所有依赖。简单方便。现在我们已经装好依赖了。让我们继续，使用这些包来创建我们的API。

我们将一起来看看`server.js`文件，这是我们通过 `package.json`里面的`main`字段定义的入口文件。

### 配置服务器 <small>server.js</small>
Node会在应用启动的时候读这个文件以获取我们定义的配置信息。

我们将从一个bear最必要的部分（能够获取）开始我们的应用。我们会保持代码干净并有清晰的注释以便于理解每一步都做了什么。

```js
// server.js

// BASE SETUP
// =============================================================================

// call the packages we need
var express    = require('express');        // call express
var app        = express();                 // define our app using express
var bodyParser = require('body-parser');

// configure app to use bodyParser()
// this will let us get the data from a POST
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

var port = process.env.PORT || 8080;        // set our port

// ROUTES FOR OUR API
// =============================================================================
var router = express.Router();              // get an instance of the express Router

// test route to make sure everything is working (accessed at GET http://localhost:8080/api)
router.get('/', function(req, res) {
    res.json({ message: 'hooray! welcome to our api!' });   
});

// more routes for our API will happen here

// REGISTER OUR ROUTES -------------------------------
// all of our routes will be prefixed with /api
app.use('/api', router);

// START THE SERVER
// =============================================================================
app.listen(port);
console.log('Magic happens on port ' + port);
```
哇，我们在这一步做了很多事情！虽然都很简单，但是我们还是挨个儿解释一下。

**基本配置** 在我们的基本配置里面，我们把npm安装好的包都引入了。我们调用了express，定义了我们自己的app,引入 bodyParser 并配置我们的app使用它。我们也支持了自定义端口。
**路由配置** 这一段里面我们会处理所有路径的请求。我们按照 Express Router 的格式创建了一个示例。然后我们就可以定义路由并应用这些路由到一个根URL里面（在这里例子里面是API）。
**启动服务器** 我们会让我们的express app 监听前面定义的端口。然后我们的应用就跑起来并可以测试啦！
## 启动服务器试试
让我们确认一下一切都按照预期在正常的在运行。启动我们的 Node app 然后发送一个请求到我们定义好的路由上以确认我们确实可以收到响应。

让我们启动服务。在命令行里输入：
```shell
$ node server.js
```
你应该会看到 Node app 已经跑起来了，并且 Express会创建一个服务器。

![](~/22-20-38.jpg)

现在我们可以确定应用已经跑起来了，然我们来测试一下。
###使用Postman来测试我们的API
Postman会帮我们测试API，它仅仅是向我们设定的URL发送HTTP请求。我们也可以传递参数（很快就会用到）和进行身份认知（在这篇教程里面我们不会用到）。

打开Postman让我们一起来看看怎么使用。

![](~/22-24-32.jpg)

你仅仅需要输入URL，选择一种HTTP方法，然后单击 『发送』 按钮。够简单吧？
下面就是见证奇迹的时刻了。我们的应用真的像我们预期的那样跑起来了吗？在地址栏里输入`http://localhost:8080/api`。默认的`GET`已经是我们需要的方法了因为我们仅仅是希望获取数据而已。现在点击 『发送』。

![](~/22-28-20.jpg)

真棒！我们得到了完全符合预期的返回。现在我们掌握了如何把想要的信息返回给请求。让我们连上数据库以便演示对于bear的增删改查操作。
## 数据库和 Bear 模型
我们会保持简短并轻松，所以我们继续进行API路由当中非常有趣的部分。我们只需要创建一个MongoDB数据库然后让我们的应用连上它。我们也需要创建一个 bear 的mongoose模型以便使用mongoose来用数据进行交互。
### 创建我们的数据库并连接
我们会使用一个由[Modulus](http://modulus.io/)公司提供的数据库。你当然也可以在本地创建一个数据库，或者使用非常棒的[Mongolab](https://mongolab.com/)云服务。你仅仅只需要一个类似下面这样的URL以便让你的应用连接。

一旦你创建好了数据库并得到了可用于连接的URI，我们就可以把它加入都我们的应用中去了。在`server.js` 的基本配置那一段里面，让我们添加两行。
```js
// server.js

// BASE SETUP
// =============================================================================

...

var mongoose   = require('mongoose');
mongoose.connect('mongodb://node:node@novus.modulusmongo.net:27017/Iganiq8o'); // connect to our database

...

```
那两行代码会调用mongoose包并连接到我们托管到 Modulus上的远程数据库。现在我们已经连上数据库了，让我们创建一个mongoose数据模型来处理我们的 bears。
### Bear 模型 <small>app/models/bear.js</small>
鉴于这个数据模型是我们本篇教程的核心内容，在这里我们仅仅只创建一个包含name字段的简单模型。嗯，就是这样。让我们创建一个文件并定义这个模型。

```js
// app/models/bear.js

var mongoose     = require('mongoose');
var Schema       = mongoose.Schema;

var BearSchema   = new Schema({
    name: String
});

module.exports = mongoose.model('Bear', BearSchema);
```
创建完这个文件之后，我们把它引入到我们的`server.js`里面以便在应用里面调用它。我们会添加下面这一行。
```js
// server.js

// BASE SETUP
// =============================================================================

...

var Bear     = require('./app/models/bear');

...
```
现在我们整个应用都已经准备好并且连上数据库啦，所以我们可以继续写路由啦。这些路由会定义我们的API接口，这也是我们为什么写这篇教程的主要原因。让我们继续！
##Express Router 和 路由表
我们会使用一个 Express Router 的实例来处理所有的路由。下面这个表里是我们需要的全部路由的一个概览，以及它们分别是干什么的，包括各自应该用什么HTTP方法去访问。

Route | HTTP Verb | Description
--- | --- | ---
/api/bears | GET |	Get all the bears.
/api/bears |	POST |	Create a bear.
/api/bears/:bear_id |	GET	|Get a single bear.
/api/bears/:bear_id |	PUT	|Update a bear with new info.
/api/bears/:bear_id |	DELETE|	Delete a bear.

这个表覆盖了一个RESTful API必须的基本路由。并且使用合适的HTTP方法（GET, POST, PUT 和 DELETE）来表达对应的行为让我们的API保持了良好的格式。
## 路由中间件
我们之前已经定义了第一条路由并且看到它生效了。也感受了一下 Express Router 让我们在处理路由时具备非常棒的灵活性。
假设我们**希望访问我们API每一个请求都会触发执行某些操作**。在这个例子里面我们仅仅是使用 `console.log()` 输出一条消息。让我们来添加一个这样的中间件。

```js
// server.js

...

// ROUTES FOR OUR API
// =============================================================================
var router = express.Router();              // 实例化一个 express Router

// 响应所有请求的中间件
router.use(function(req, res, next) {
    // 打出log
    console.log('Something is happening.');
    next(); // 确保我们的请求会进入下一个路由而不是停在这里。
});

// 测试一下路由确保一切正常 (使用GET请求访问 http://localhost:8080/api)
router.get('/', function(req, res) {
    res.json({ message: 'hooray! welcome to our api!' });   
});

// 我们API的更多路由会在这里定义

// REGISTER OUR ROUTES -------------------------------
// 我们所有的路由都会带 /api 前缀
app.use('/api', router);

...
```
我们仅仅只需要使用 `router.use(function())` 这样的代码就可以定义一个中间件。定义路由的代码的顺序是很重要的。得益于 [Express 4.0 的改进](https://scotch.io/bar-talk/expressjs-4-0-new-features-and-upgrading-from-3-0) 它们会按照定义的顺序依次执行，我们不再需要面对 Express 3.0 里面那样的问题。一切都会按照正确的顺序执行。

我们返回 **JSON data**格式的数据。这也是符合RESTful标准的API格式并且便于我们的用户获取数据。

我们也添加了 `next()` 方法表明我们的应用应该继续处理其他路由请求。这一点非常重要因为如果不加这一条的话我们的应用会停在这个中间件里。

**习惯用中间件的用户**像这样使用中间件会非常有用。我们可以在这里面做校验以保证所有的请求都是安全有效的。如果有什么不对的话我们也可以在这里抛出异常。我们还可以在这个步骤里打出一些额外的日志或者做一些我们想做的任何统计分析。可以说使用中间件可以创造出无限可能。嗨起来吧！
#### 测试我们的中间件
现在当我们使用 Postman 像我们的应用发送一个请求的时候，`Something is happening` 这条日志会被输出到我们的 Node 控制台（命令行窗口）。

![](~/22-38-54.jpg)
借助中间件，我们可以在接受到请求的时候干一些漂亮的事情。比如我们也许需要确保用户在访问API的时候已授权。我们会在下一篇教程里面聊这个，不过现在我们仅仅只是用中间件打一些日志。
## 创建基本路由
我们现在来创建处理 **获取所有bears** 和 **创建一个bear** 请求的路由。这两个操作都是由`/api/bears`路由处理的。我们先来实现创建一个bear，这样我们就有了bear数据来做后面的事情了。
### 创建一个 Bear <small>POST /api/bears</small>
我们会添加一条新路由来处理POST请求然后使用Postman进行测试。
```js
// server.js

...

// ROUTES FOR OUR API
// =============================================================================

... // <-- route middleware and first route are here

// more routes for our API will happen here

// on routes that end in /bears
// ----------------------------------------------------
router.route('/bears')

    // create a bear (accessed at POST http://localhost:8080/api/bears)
    .post(function(req, res) {

        var bear = new Bear();      // create a new instance of the Bear model
        bear.name = req.body.name;  // set the bears name (comes from the request)

        // save the bear and check for errors
        bear.save(function(err) {
            if (err)
                res.send(err);

            res.json({ message: 'Bear created!' });
        });

    });

// REGISTER OUR ROUTES -------------------------------
// all of our routes will be prefixed with /api
app.use('/api', router);

...

```
现在我们已经创建好了处理`POST`方法的路由了。我们将使用 Express的 `router.route()` 来处理同一个URL的不同路由。在本例中意味着我们可以处理所有以`/bears`结尾的路由。
好了，现在让我们打开Postman来创建我们的 bear。

![](~/22-50-08.jpg)
注意这里我们使用了`x-www-form-urlencoded`格式来发送`name`数据。这会将我们的数据编码成query字符串发送到服务器端。

我们得到了bear创建成功的返回消息。让我们继续来写获取所有bear的路由，写完之后我们就可以看到我们刚才创建的bear了。

### 获取所有 Bears <small>GET /api/bears</small>
我们将会在上一步中基于 `router.route()`创建的POST路由上添加一条简单的路由规则。得益于 router.route()方法，我们可以使用链式语法把相同url的不同类型路由串起来。这可以让我们的代码看起来更加干净并更有条理。
```js
// server.js

...

// ROUTES FOR OUR API
// =============================================================================

... // <-- route middleware and first route are here

// more routes for our API will happen here

// on routes that end in /bears
// ----------------------------------------------------
router.route('/bears')

    // create a bear (accessed at POST http://localhost:8080/api/bears)
    .post(function(req, res) {

        ...

    })

    // get all the bears (accessed at GET http://localhost:8080/api/bears)
    .get(function(req, res) {
        Bear.find(function(err, bears) {
            if (err)
                res.send(err);

            res.json(bears);
        });
    });

// REGISTER OUR ROUTES -------------------------------
// all of our routes will be prefixed with /api
app.use('/api', router);

...
```
一目了然，只需要发送 `GET`请求到 `http://localhost:8080/api/bears` 就能获得以JSON格式返回的全部bears。

![](~/17-16-31.jpg)
## 为单条数据创建路由
前面我们已经写好了以 `/bears` 结尾的获取全部bears的路由。让我们继续来处理带id参数的路由。

这种路由的格式会以`/bears/:bear_id` 结尾，并且具备如下功能：
* 获取单条 bear。
* 修改单条 bear 信息。
* 删除单条 bear。
得益于我们在前面调用的`body-parser`包，我们可以轻松拿到请求中的`:bear_id`。

### 获取单条 Bear <small>GET /api/bears/:bear_id</small>
我们将新增一条 `router.route()` 来处理所有带`:bear_id`的请求。
```js
// server.js

...

// ROUTES FOR OUR API
// =============================================================================

...

// on routes that end in /bears
// ----------------------------------------------------
router.route('/bears')
    ...

// on routes that end in /bears/:bear_id
// ----------------------------------------------------
router.route('/bears/:bear_id')

    // get the bear with that id (accessed at GET http://localhost:8080/api/bears/:bear_id)
    .get(function(req, res) {
        Bear.findById(req.params.bear_id, function(err, bear) {
            if (err)
                res.send(err);
            res.json(bear);
        });
    });

// REGISTER OUR ROUTES -------------------------------
// all of our routes will be prefixed with /api
app.use('/api', router);

...

```
我们可以从获取全部bears的接口里面查到某条bear的id。让我们用获取到的id在Postman里面来测试获取单条bear。

![](~/17-49-32.jpg)

我们现在已经可以通过API获取单条bear了！让我们试试修改那条bear的name字段。我们假设他希望更加精确一些，所以我们将他从 Klaus 重命名为**Sir Klaus**。
### 修改单条 Bear 信息<small>PUT /api/bears/:bear_id</small>
让我们用链式语法添加`.put()`方法吧。
```js
// server.js

...

// on routes that end in /bears
// ----------------------------------------------------
router.route('/bears')
    ...

// on routes that end in /bears/:bear_id
// ----------------------------------------------------
router.route('/bears/:bear_id')

    // get the bear with that id (accessed at GET http://localhost:8080/api/bears/:bear_id)
    .get(function(req, res) {
        ...
    })

    // update the bear with this id (accessed at PUT http://localhost:8080/api/bears/:bear_id)
    .put(function(req, res) {

        // use our bear model to find the bear we want
        Bear.findById(req.params.bear_id, function(err, bear) {

            if (err)
                res.send(err);

            bear.name = req.body.name;  // update the bears info

            // save the bear
            bear.save(function(err) {
                if (err)
                    res.send(err);

                res.json({ message: 'Bear updated!' });
            });

        });
    });

// REGISTER OUR ROUTES -------------------------------
// all of our routes will be prefixed with /api
app.use('/api', router);

...
```
我们将会通过 PUT 请求中的`id`参数定位到那条特定的bear，修改信息并存回数据库。

![](~/17-57-19.jpg)
我们也可以通过前面用过的 `GET /api/bears` 请求来观察他的名称已发生变化。

![](~/18-00-02.jpg)
### 删除单条 Bear <small>DELETE /api/bears/:bear_id</small>
当我们需要删除某条bear时，只需要发送一个 DELETE请求到`/api/bears/:bear_id`
让我们加入删除功能的代码。
```js
// server.js

...

// on routes that end in /bears
// ----------------------------------------------------
router.route('/bears')
    ...

// on routes that end in /bears/:bear_id
// ----------------------------------------------------
router.route('/bears/:bear_id')

    // get the bear with that id (accessed at GET http://localhost:8080/api/bears/:bear_id)
    .get(function(req, res) {
        ...
    })

    // update the bear with this id (accessed at PUT http://localhost:8080/api/bears/:bear_id)
    .put(function(req, res) {
        ...
    })

    // delete the bear with this id (accessed at DELETE http://localhost:8080/api/bears/:bear_id)
    .delete(function(req, res) {
        Bear.remove({
            _id: req.params.bear_id
        }, function(err, bear) {
            if (err)
                res.send(err);

            res.json({ message: 'Successfully deleted' });
        });
    });

// REGISTER OUR ROUTES -------------------------------
// all of our routes will be prefixed with /api
app.use('/api', router);

...
```
好了，现在当我们使用`DELETE`方法带着正确的`bear_id`发送请求时，我们会删除这条bear了。

![](~/18-06-21.jpg)
当我们尝试获取所有bears时，会发现现在什么也没有了。

![](~/18-06-53.jpg)
## 总结
我们的API现在已经能够处理某种数据（本例中是我们可爱的熊熊）常规的增删改查操作了。使用上面介绍的技术会帮你开发更大型更健壮的API打下一个良好的基础。
在这篇里面我们快速过了一遍如何使用Express 4开发一个基于Node的API。如果这是你真实的API的话还有很多的事情可以做。比如你可以添加身份认证，创建更友好的错误信息，添加不同的业务逻辑等。
如果你有任何疑问以及希望在未来看到某些特定领域的话题，欢迎在评论里留言。
## 关注我们
![微信扫一扫关注我们](http://7xp3fo.com1.z0.glb.clouddn.com/_image/yufe2015/h5.vc.jpg)