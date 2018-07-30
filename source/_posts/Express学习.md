---
title: Express学习
date: 2018-07-28 21:18:48
tags: [Node, Express, 技术]
---

# 前言

做 B/S 结构数据库作业的时候，发现需要一种后台语言来做数据处理并且与前台交互，因为学的是 JS，所以很自然就选择了 Node.js 来做这件事情，找了一些教程，用 Node 实现了 MySQL 的基本操作，又觉得写得代码太差劲，所以推翻，决定用框架写一个真正的后台，不需要多复杂，但至少该有的都要有。因而决定使用 Express 这个最基本也最火的 Node 框架来做。
可真正接触之后发现，这和前端基本就是两个世界，工程目录都看不懂，于是开始模仿一些教程来做，想着，反正是做个后台，又不是以后学后端，知其然就行了。然而发现，我又错了，看都看不懂，又怎么会知其然呢？于是我开始找有没有一个系统的完整的 Express 教程，可惜的是，并没能找到。网易云倒是有视频教程，可我负担不起费用。
因此种种，我决定从官方文档到社区零散教程再到 GitHub 开源项目一点点学习，完成我的项目。至此，我要完成已经不仅仅是一个后台数据传输处理的服务了，而是一个真正的有着前后端的 Web 项目。

<!--more-->

与此同时，我决定写下这个教程，用以记录并分享我的学习历程。

跟着这个教程，你最后会完成一个基于 Express 框架做后端的学生信息管理系统。

        最后更新日期： 2016/11/23（如距离当前日期较长，请慎重）

---

## 声明：

**本教程试用人员为已经有一定前端基础后想尝试后端或者想要理解后端的前端学习人员。毕竟，我们学习了 JS，为什么不努力让自己成为一个侧重前端的全栈呢？我们可以不写后端，但是我们能看懂后端做了什么，怎么做的**

## 技术要求：

毕竟，这是一个框架使用教程，而非一种编程语言教程。

- HTML、Bootstrap（能看懂即可，前端页面会用到）
- JS 基础（能基本理解 JS 语法。可通过菜鸟教程或者廖雪峰官方网站学习）
- Node 基础（能够独立安装使用，能理解函数、回调、路由等基本名词。可通过菜鸟教程或者廖雪峰官方网站学习）

# 正式开始

## 安装 Express 框架

安装 Node 时已自动安装 npm，名词解释，自行百度。

使用 npm 命令

    npm install express --save

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fntylv2n38j30sc0fjgn7.jpg)

body-parser - node.js 中间件，用于处理 JSON, Raw, Text 和 URL 编码的数据。

    npm install body-parser --save

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fntyjx2lkfj30st0gtdhh.jpg)

cookie-parser - 这就是一个解析 Cookie 的工具。通过 req.cookies 可以取到传过来的 cookie，并把它们转成对象。

    npm install cookie-parser --save

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fntykd6blpj30st0dkq49.jpg)

multer - node.js 中间件，用于处理 enctype="multipart/form-data"（设置表单的 MIME 编码）的表单数据。

    npm install multer --save

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fntyksxr1mj30rs0etjsj.jpg)

完成 Express 安装，通过下列语句查看版本号

    express -v

##使用 Express 生成项目
Express 框架有着完整的结构，所以我们可以直接通过命令行来生成一个项目结构。
进入想要生成项目的目录，执行以下命令
express ExStuMS //项目名称
![image](https://ws1.sinaimg.cn/large/0064OUUqly1fntyl3atr8j30rr0hcwg6.jpg)

    cd ExStuMS  //进入项目根目录
    npm install //安装项目依赖

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fntymt3z6vj30pe0fejso.jpg)

    npm start   //启动项目

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntynkal9uj30hp042jre.jpg)

此时打开浏览器，输入http://localhost:3000/，看到以下页面，表示成功

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyntkjlij30kl070mxd.jpg)

快捷键 Ctrl+C 结束服务

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyo3xpkwj30cp02zdfs.jpg)

## 项目目录简介

打开我们刚刚创建的 ExStuMS 工程 这里我推荐使用 WebStorm 或者 HBuilder， 作为演示我们使用 WebStorm 打开工程

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyoi6kmaj30lq0c0t9h.jpg)

- node_modules 是 nodejs 所需要用到的依赖包
- public 提供前台的 css 和 js
- routes 提供路由
- views 提供视图模板
- app.js 程序的启动口和入口

## 配置 MySQ

确保电脑中已经安装 MySQL。

建立所需数据库及相关表格，并插入数据。这里使用 navicat for mysql

新建数据库名为：software15 字符集编码：utf-8

新建管理员信息表：user

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyp3ds9rj30nz02eaaa.jpg)

新建学生信息表：studentInfo

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntypdivfgj30nu04xwf0.jpg)

这两张是目前的基础表，当项目完成之后，想要更新一些功能，会再次扩展

## 安装 MySQL 模块

打开 package.json 在 dependencies 选项中添加

```
"mysql": "latest"
```

控制台执行如下命令

    npm install //安装mysql依赖
    npm start   //启动服务器

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntypweb2nj30fg07qmxj.jpg)
接着我们向工程中添加一个 db 目录 用于存放 MySQL 配置信息
并在 db 目录中新建一个 DBConfig.js 文件并添加如下内容

```javascript
module.exports = {
        mysql: {
            host: '127.0.0.1',
            user: 'root',
            password: '123',   //你的数据库Coonection密码
            database:'software15', // 前面建的表位于这个数据库中
            port: 3306
        }
    };
```

完成后工程目前的结构如下

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyqe2etuj30ye0ar75y.jpg)

## 添加 API 接口调用 SQL 语句

### 首先添加用户数据表相关操作，db 目录下新建一个 userSQL.js :

    var userSQL={
        //注册添加用户
        insert:'insert into user values (?,?)',
        //注销用户
        deleteByUN:'delete from user where userName = ?',
        //更改密码
        updatePw:'update user set password = ? where userName = ? and password = ?',
        //查询密码，用以登录页以及更改密码页验证密码
        selectPwByUN:'select password from user where usrName = ?'
    };
    module.exports = userSQL;   //  暴露借口为userSQL

### 添加学生信息表相关操作，同样在 db 目录下新建一个 StuInfoDao.js :

    var StuSQL={
        //增加学生信息
        insert:'insert into studentInfo values (?,?,?,?,?)',
        //删除学生信息
        deleteByAll:'delete from student where Sno like ?',
        //更新学生信息，默认Sno不可改
        updatePw:'update user set Sname = ?,Ssex = ?,Sclass = ?,Sphone = ? where Sno = ? ',
        //查询所有学生信息
        queryAll:'select * from student'
        /*至于多条件查询的相关语句，则不必调用SQL语句实现，smartTable已经帮我们搞定了一切*/
    };
    module.exports = StuSQL;

## 定义所需前端内容

目前只是简单实现，所以只需要两个页面，一个登陆页面，一个主操作页面，前端的增删查改全部由 smartTable 完成

### 首先在 public 目录下新建 HTML 目录用来存放 HTML 页面

定义 index.html:

```html
<!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>用户登陆</title>
        <!-- 新 Bootstrap 核心 CSS 文件 -->
        <link href="http://cdn.static.runoob.com/libs/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
        <!-- jQuery文件。务必在bootstrap.min.js 之前引入 -->
        <script src="http://cdn.static.runoob.com/libs/jquery/2.1.1/jquery.min.js"></script>
        <!-- 最新的 Bootstrap 核心 JavaScript 文件 -->
        <script src="http://cdn.static.runoob.com/libs/bootstrap/3.3.7/js/bootstrap.min.js"></script>
        <!-- 引入angular.js -->
        <script src="http://cdn.static.runoob.com/libs/angular.js/1.4.6/angular.min.js"></script>
        //连接到外部样式表，写在stylesheets目录下
        <link href="../stylesheets/Login.css" rel="stylesheet">
    </head>
    <body>
    <div id="mainDiv" class="col-sm-10">
        <h1 class="text-center">
            管理员登陆
        </h1>
        <div  ng-app="myApp"  ng-controller="validateCtrl">
            <form name="myForm" novalidat>
                <div class="form-group col-sm-10">
                    <h3><label for="name" class="col-sm-2 control-label label label-primary">用户名：</label></h3>
                    <div class="col-sm-8">
                        <input type="text" class="form-control " id="name" placeholder="请输入名称">
                    </div>
                </div>
                <div class="form-group col-sm-10">
                    <h3><label for="password" class="col-sm-2 control-label label label-primary">密 码：</label></h3>
                    <div class="col-sm-8">
                      <input type="password" class="form-control" id="password" placeholder="请输入密码">
                    </div>
                </div>
                <div class="form-group col-sm-10">
                    <button class="btn btn-primary btn-lg">登陆系统</button>
                </div>
            </form>
        </div>
    </div>
    //连接到脚本文件，写在javascripts目录下
    <script src="../javascripts/index.js"></script>
    </body>
    </html>
```

index.css:

```css
//样式表自由更改即可
    #mainDiv{
        width: 80%;
        height: 75%;
        margin-top: 10%;
        margin-left: 20%;
    }
    h1{
        width: 70%;
        background-color: #a7dbd8;
        font-family: "李旭科毛笔行书", Gadget, sans-serif;
    }
    button{
        margin-left: 77%;
    }
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyr04epjj30xf0fvgma.jpg)

定义 main.html

```html
<!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>学生信息管理页面</title>
        <link href="http://cdn.static.runoob.com/libs/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
        <script src="http://cdn.static.runoob.com/libs/jquery/2.1.1/jquery.min.js"></script>
        <script src="http://cdn.static.runoob.com/libs/bootstrap/3.3.7/js/bootstrap.min.js"></script>
        <script src="http://cdn.static.runoob.com/libs/angular.js/1.4.6/angular.min.js"></script>
        <script src="https://cdn.bootcss.com/angular-smart-table/2.1.8/smart-table.min.js"></script>
        <link href="../stylesheets/main.css" rel="stylesheet">
    </head>
    <body>
    <header>
        <h1 class="text-center">
            15级软件学生信息管理
        </h1>
        <p class="text-center f1">
            当前登陆：{{userName}}--<a href="Login.html">退出登陆</a>
        </p>
    </header>
    <h3 class="text-center">
        学生信息一览
    </h3>
    <div ng-app="myApp" ng-controller="myCont">
    <button class="btn btn-primary" ng-click="addNewStu()">AddNewStu</button>
    <table st-table="displayedCollection" st-safe-src="rowCollection" class="table table-hover table-condensed">
        <thead>
        <tr>
            <th>学号</th>
            <th>姓名</th>
            <th>性别</th>
            <th>班级</th>
            <th>联系方式</th>
        </tr>
        <tr>
            <th>
                <input st-search="Sno" class="input-sm form-control search-input" placeholder="Search Sno" type="search"/>
            </th>
            <th>
                <input st-search="Sname" class="input-sm form-control search-input" placeholder="Search Sname" type="search"/>
            </th>
            <th>
                <select class="form-control">
                    <option></option>
                    <option>男</option>
                    <option>女</option>
                </select>
            </th>
            <th colspan="2">
                <input st-search="Sclass" class="input-sm form-control search-input" placeholder="Search Ssex" type="search"/>
            </th>
        </tr>
        </thead>
        <tbody>
        <tr ng-repeat="student in displayedCollection">
            <td>
                <input type="text" id="{{student.Sno}}_no" ng-model="student.Sno" class="input-sm form-control" placeholder="暂无学号" readonly>
            </td>
            <td>
                <input type="text" id="{{student.Sno}}_{{student.Sname}}" ng-model="student.Sname" class="input-sm form-control" placeholder="暂无姓名" readonly>
            </td>
            <td>
                <select class="form-control" disabled="true">
                    <option>男</option>
                    <option>女</option>
                </select>
            </td>
            <td>
                <input type="text" id="{{student.Sno}}_{{student.Sclass}}" ng-model="student.Sclass" class="input-sm form-control" placeholder="暂无班级信息" readonly>
            </td>
            <td>
                <input type="text" id="{{student.Sno}}_{{student.Sphone}}" ng-model="student.Sphone" class="input-sm form-control" placeholder="暂无联系信息" readonly>
            </td>
            <td>
                <button type="button" id="{{user.Sno}}_ediBtn" class="btn btn-primary" ng-click="edit(user)" >edit</button>
                <button type="button" id="{{user.Sno}}_delBtn" class="btn btn-warning" ng-click="delete(user)" >delete</button>
                <button type="button" id="{{user.Sno}}_savBtn" class="btn btn-primary" ng-click="save(user)" style="display: none">Save</button>
                <button type="button" id="{{user.Sno}}_canBtn" class="btn btn-default" ng-click="cancel(user)" style="display: none">Cancel</button>
            </td>
        </tr>
        </tbody>
    </table>
    </div>
    <script src="../javascripts/main.js"></script>
    </body>
    </html>
```
main.css:
```css
header{
        background-color: beige;
    }
    h1{
        font-family: "李旭科毛笔行书", Gadget, sans-serif;
    }
    .f1{
        font-family: "黑体", Gadget, sans-serif;
    }
```

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fntyrbgn0lj31430coabu.jpg) ##把写好的网页挂载到服务器

1.  在写好的 index.html 的 body 标签中添加 ng-view 属性
2.  更改 app.js 文件
    ![image](https://ws1.sinaimg.cn/large/0064OUUqly1fntys77imjj30or06zgm6.jpg)

---

此时我们已经完成大部分工作。
控制台启动服务，浏览器打开http://localhost:3000/ 出现以下界面

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fntysjo13mj312w0eaab8.jpg)

---

## 接下来编写可用 API 供前台调用

修改 route.jS 文件
![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyswslnnj30bg053q32.jpg)

写好的源码如下：

```javascript
// 导入MySQL模块
    var express = require('express');
    var router = express.Router();
    // 导入MySQL模块
    var mysql = require('mysql');
    var dbConfig = require('../db/DBConfig');
    var StuSQL = require('../db/StuInfoDao');
    var userSQL = require('../db/userSQL');
    var pool = mysql.createPool( dbConfig.mysql );
    //查询用户
    router.get('/queryUser',function(req, res, next){
      pool.getConnection(function(err, connection){
        var param = req.query || req.params;
        connection.query(userSQL.selectByUN,[param.name], function(err, result){
          if(err){
            res.json(err);
            console.log("ERROR 无对应用户");
          }
          if(result){
            console.log(result[0].password);
            res.json(result);
          }
          connection.release();
        })
      })
    });
    //添加用户
    router.get('/addUser', function(req, res, next){
      pool.getConnection(function(err, connection){
        var param = req.query || req.params;
        connection.query(userSQL.insert,[param.name,param.pwd], function(err, result){
          if(err){
            throw err;
            //console.log('添加失败');
          }
          if(result){
            console.log('----------INSERTUSER----------');
            console.log('插入数量为：',result.affectedRows);
          }
          connection.release();
        })
      })
    });
    //更改密码
    router.get('/updatePw', function(req, res, next){
      pool.getConnection(function(err, connection){
        var param = req.query || req.params;
        connection.query(userSQL.updatePw,[param.newpwd,param.name,param.pwd], function(err, result){
          if(err){
            throw err;
          }
          if(result){
            console.log('----------UPDATEPWD----------');
            console.log('更新数量为：',result.affectedRows);
          }
          connection.release();
        })
      })
    });
    //获取全部学生信息
    router.get('/queryAll', function(req, res, next){
      pool.getConnection(function(err, connection) {
        connection.query(StuSQL.queryAll,  function(err, result) {
          if(result) {
            res.json(result);
            /*for(var i = 0; i < result.length; i++){
              console.log("%s\t%s\t%s",result[i].Sno,result[i].Sname,result[i].Ssex);
            }*/
            console.log('----------queryAll----------')
            console.log('总信息数量为：',result.length);
          }
          // 释放连接
          connection.release();
        });
      });
    });
    //删除学生
    router.get('/deleteOne', function(req, res, next){
      pool.getConnection(function(err, connection){
        var param = req.query || req.params;
        connection.query(StuSQL.deleteByAll,[param.sno], function(err, result){
          if(err){
            throw (err);
            //req.json({msg:'删除失败'});
          }
          console.log('-------------DELETE--------------');
          console.log('DELETE affectedRows',result.affectedRows);
          res.json({msg:'成功'});
          connection.release();
        })
      })
    });
    //添加学生
    router.get('/addOne', function(req, res, next){
      pool.getConnection(function(err, connection){
        var param = req.query || req.params;
        connection.query(StuSQL.insert,[param.sno,param.sname,param.ssex,param.sclass,param.sphone], function(err, result){
          if(err){
            throw (err);
          }
          console.log('-----------INSERT------------');
          console.log('INSERT affectedRows:',result.affectedRows);
          res.json({msg:'成功'});
          connection.release();
        })
      })
    });

    //更新学生信息
    router.get('/updateOne', function(req, res, next){
      pool.getConnection(function(err, connection){
        var param = req.query || req.params;
        connection.query(StuSQL.updateSname,[param.name,param.sex,param.class,param.phone,param.sno],function(err,result){
          if(err){
            throw err;
          }
          if(result){
            console.log('-----------UPDATE------------');
            console.log('UPDATE affectedRows:',result.affectedRows);
            res.json({msg:'成功'});
          }
          connection.release();
        })
      })
    });
    module.exports = router;
```

接下来，我们可以在我们前端的脚本文件里用 ajax 来实现数据的传输获取，展示到前端
index.js：

```javascript
/**
     * Created by acer on 2016/11/22.
     */
    var app = angular.module('myApp', []);
    app.controller('customersCtrl', function($scope,$window) {
        //$scope.inputName = '未输入用户名';
        // $scope.displayedCollection = [].concat($scope.rowCollection);
        $scope.login = function(){
            var res, userName, password;
            $.ajax({
                type: "get",
                url: "http://localhost:3000/users/queryUser?name="+$scope.inputName,
                dataType: "json",
                async:false,
                success: function(data){
                    if(data){
                        res = data;
                        //alert($scope.rowCollection[0].Sno);
                    } else{
                        alert('失败');
                    }
                }
            });
            userName = res[0].userName;
            password = res[0].password;
            if($scope.inputName == userName){
                if($scope.inputPassword == password) {
                    $window.location.href = "http://localhost:3000/main.html?"+$scope.inputName;
                }else alert("密码错误！");
            }else alert("用户名错误！")
        };
    });
```

main.js:

```javascript
var getPare = function (){
        var url = document.location.toString();
        var arrUrl = url.split("?");
        var para = arrUrl[1];
        return para;
    };
    var dataAll;

    var app = angular.module('myApp',['smart-table']);
    app.controller('myCont',function($scope){
        $scope.userName = getPare();

        $scope.displayedCollection = [].concat($scope.rowCollection);
      $.ajax({
        type: "get",
        url: "http://localhost:3000/users/queryAll",
        dataType: "json",
        async:false,
        success: function(data){
          if(data){
            $scope.rowCollection = data;
            //dataAll=data;
            //alert($scope.rowCollection[0].Sno);
          } else{
            alert('失败');
          }
        }
      });
      $scope.itemsByPage=12;
      //$scope.tablePages = $scope.rowCollection
      $scope.tablePages = Math.ceil($scope.rowCollection.length);
       // alert($scope.rowCollection.length);
        $scope.delete = function(user){
            //$scope.rowCollection.splice(index-1,1);
            //alert(index);
            var index = $scope.rowCollection.indexOf(user);
            if (index !== -1) {
                $scope.rowCollection.splice(index, 1);
            }
            $.ajax({
               type: "get",
                url: "http://localhost:3000/users/deleteOne?sno="+user.Sno,
                dataType: "json",
                async:false,
                success: function(data){
                    if(data){
                        //alert(data.msg);
                    }
                },
                error: function(){
                  alert('删除失败！') ;
                }
            });
        };
        $scope.edit = function(user){
            $('#id_'+user.Sno+'_'+user.Sname).removeAttr('readonly').css('border','1px').css('background-color','#1ECEE1');
            $('#id_'+user.Sno+'_'+user.Ssex).removeAttr('disabled');
            $('#id_'+user.Sno+'_'+user.Sclass).removeAttr('disabled');
            $('#id_'+user.Sno+'_'+user.Sphone).removeAttr('readonly').css('border','1px').css('background-color','#1ECEE1');
            $('#'+user.Sno+'_ediBtn').toggle();
            $('#'+user.Sno+'_delBtn').toggle();
            $('#'+user.Sno+'_savBtn').show();
            $('#'+user.Sno+'_canBtn').show();
        };
        $scope.save = function(user){
            $('#id_'+user.Sno+'_'+user.Sname).attr('readonly',true).css('border','0px').css('background-color','transparent');
            $('#id_'+user.Sno+'_'+user.Ssex).attr('disabled',true);
            $('#id_'+user.Sno+'_'+user.Sclass).attr('disabled',true);
            $('#id_'+user.Sno+'_'+user.Sphone).attr('readonly',true).css('border','0px').css('background-color','transparent');
            $('#'+user.Sno+'_ediBtn').show();
            $('#'+user.Sno+'_delBtn').show();
            $('#'+user.Sno+'_savBtn').toggle();
            $('#'+user.Sno+'_canBtn').toggle();
            $.ajax({
               type: "get",
                url: "http://localhost:3000/users/updateOne?name="+user.Sname+"&sex="+user.Ssex+"&class="+user.Sclass+"&phone="+user.Sphone+"&sno="+user.Sno,
                dataType: "json",
                async: false,
                success: function(data){
                    if(data.msg == '成功'){
                        //alert('成功');
                    }
                }
            });
        };
        $scope.addNewStu = function(){
            $scope.inserted = {
                id: $scope.rowCollection.length+1,
                Sname: '',
                Ssex: '男',
                Sclass: '软件1501',
                Sphone:''
            };
            $scope.rowCollection.push($scope.inserted);
        };
    });
```

---

现在我们可以在数据库传入数据，进行实际测试

1.  控制台进入工程目录，npm start 开启服务器
2.  登陆http://localhost:3000/(我们以root为用户名，root123位密码进行测试。
3.  跳转到数据显示页面。
    ![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyu9skxzj31360dlaat.jpg)
    ![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyujke9qj31320jsq60.jpg)

## 大功告成！

当然，我们可以通过使用一些图表使得数据显示更加多样化。这属于前端内容，需要自己尝试了。

我的做成了这样：
![](https://ws1.sinaimg.cn/large/0064OUUqly1fntyuyzr7jj31480llgr5.jpg)
