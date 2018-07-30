---
title: Mongo安装以及对应Node基本操作
date: 2018-07-28 20:59:39
tags: [Node, Mongo, 安装配置, 技术]
---

# 下载

https://www.mongodb.com/
下载对应系统版本的 msi 安装程序，根据提示安装到硬盘。安装界面选 compents 来指定安装目录。

<!--more-->

# 初始化及其启动

1.  手动在盘符下创建 data 文件夹，再进入创建 db 文件夹用以保存数据。
2.  打开 cmd（windows 键+r 输入 cmd）命令行，进入 D:\mongodb\bin 目录（如图先输入 d:进入 d 盘然后输入 cd d:\mongodb\bin），输入如下的命令启动 mongodb 服务：D:/mongodb/bin>mongod --dbpath D:\data\db **_（data 文件夹位置）_**
3.  mongodb 默认连接端口 27017，如果出现如图的情况，可以打开http://localhost:27017查看（笔者这里是chrome），发现如图则表示连接成功，如果不成功，可以查看端口是否被占用。

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntv6hy1wjj30hh03eglx.jpg)

# 设置 mongo 自启动服务

1.  在 data 下新建文件夹 log（存放日志文件）并且新建文件 mongodb.log，在 mongodb（你的 mongo 安装位置）新建文件 mongo.config
2.  用记事本打开 mongo.config 输入：dbpath=D:\mongodb\data\db
    logpath=D:\mongodb\data\log\mongo.log
    //将位置改为自己的
    ![](https://ws1.sinaimg.cn/large/0064OUUqly1fntv70s292j30hn05l74w.jpg)
3.  用**管理员身份**打开 cmd 命令行，进入 D:\mongodb\bin 目录，输入如下的命令：
    mongod --config “D:\mongodb\mongo.config” --install --serviceName "MongoDB"
4.  打开 cmd 输入 services.msc 查看服务可以看到 MongoDB 服务，点击可以启动。

---

# 相关 node 操作

```javaScript
var mongo = require('mongodb'),
    Server = mongo.Server,
    Db = mongo.Db;

    var server = new Server('172.16.101.252', 27017, {auto_reconnect: true});
    var db = new Db('foo', server);

    db.open(function(err, db) {
        if(!err) {
            console.log("已经连接mongoDB数据库！");
            insertData(db);
            selectData(db, function(result) {
                console.log(result);
                db.close();
            });
            updateData(db);
            selectData(db, function(result) {
                console.log(result);
                db.close();
            });
            deleteData(db);
        }
    });
    //增加
    function insertData(db)
    {
        var devices = db.collection('mycollection');
        var data = {"name":"node3号","age":22,"addr":"nb","addTime":new Date()};
        devices.insert(data,function(error, result){
            if(error)
            {
                console.log('Error:'+ error);
            }else{
                console.log("已添加node3号");
                console.log(result.result.n);
            }
            db.close();
        });
    }
    //查找
    var selectData = function(db, callback) {
        //连接到表
        var collection = db.collection('mycollection');
        //查询数据
        var whereStr = {"name": 'node3号'};
        collection.find(whereStr, function (error, cursor) {
            cursor.each(function (error, doc) {
                if (doc) {
                    //console.log(doc);
                    if (doc) {
                        console.log("已经查询到node3号");
                        console.log(doc);
                    }
                }
            });

        });
    }
    //更改
    function updateData(db)
    {
        var devices = db.collection('mycollection');
        var whereData = {"name":"node3号"};
        var updateDat = {$set: {"age":26}}; //如果不用$set，替换整条数据
        devices.update(whereData, updateDat, function(error, result){
            if (error) {
                console.log('Error:'+ error);
            }else{
                console.log("已更新node3号");
                console.log(result.result.n);
            }
            db.close();
        });
    }
    //删除
    function deleteData(db)
    {
        var devices = db.collection('mycollection');
        var data = {"name":"node3号"};
        devices.remove(data, function(error, result){
            if (error) {
                console.log('Error:'+ error);
            }else{
                console.log("已删除node3号");
                console.log(result.result.n);
            }
            db.close();
        })
    }
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntv7mmb2cj30ki0g0mxs.jpg)

---

# 更新依赖库： Mongoose
