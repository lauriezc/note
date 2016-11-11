---
layout : z
title : 'MVC4 AJAX POST Array'
category : 'csharp'
---


##### MVC4下AJAX往Action传递Array参数的方法

Entity


    public class Entity
    {
        public int id { get; set; }
        public string name { get; set; }
    }


Action


    public ActionResult ReceiveArray(List<Entity> list)
    {
        return View();
    }


JQuery


    var list = [];
    for(var i=0;i<10;i++) {
        var item = {};
        item.id = i;
        item.name = 'name' + i.toString();
        list.push(item);
    }
    $.ajax({
       'url' : '/controller/ReceiveArray',
       'type' : 'POST',
       'data' : { 'list' : list },
       'success' : function(data) {
            //do something
        }  
    });



在MVC5下AJAX这样写是没有问题的，Action能接收到对应的RouteData，但在MVC4下能接收到浏览器传过来的数组但是却读不到里面对应属性的值（值全是NULL）。


JQuery


    var list = [];
    for(var i=0;i<10;i++) {
        var item = {};
        item.id = i;
        item.name = 'name' + i.toString();
        list.push(item);
    }
    $.ajax({
       'url' : '/controller/ReceiveArray',
       'type' : 'POST',
       'data' : JSON.stringify({'list' : list }),
       'contentType' : 'application/json' ,
       'success' : function(data) {
            //do something
        }  
    });



MVC4下要将list对象格式化成字符串,将contentType头设置成json格式才Action才能接收到完整的数组对象。

这个差异估计是参数解析的差异，需要对比framework4.0和4.5这部分源码才能知道。



---

2016-11-11 10:33:35 UPDATE

JQuery AJAX在提交数据之前会格式化data，而且默认提交的数据内容格式是：application/x-www-form-urlencoded。
以下是对应参数下AJAX产生的Request数据：

###### 1 以表单的格式

    $.ajax({
       'url' : '/controller/ReceiveArray',
       'type' : 'POST',
       'data' : { 'list' : list },
       'success' : function(data) {
            //do something
        }  
    });


![请求信息](https://github.com/lauriezc/note/raw/master/resource/image/simple.png)

###### 2 格式化的对象


    $.ajax({
       'url' : '/controller/ReceiveArray',
       'type' : 'POST',
       'data' : {'list' : list },
       'contentType' : 'application/json' ,
       'success' : function(data) {
            //do something
        }  
    });



![请求信息](https://github.com/lauriezc/note/raw/master/resource/image/nostringify.png)

###### 3 json格式


    $.ajax({
       'url' : '/controller/ReceiveArray',
       'type' : 'POST',
       'data' : JSON.stringify({'list' : list }),
       'contentType' : 'application/json' ,
       'success' : function(data) {
            //do something
        }  
    });



![请求信息](https://github.com/lauriezc/note/raw/master/resource/image/best.png)


MVC4只能解析上述第三种格式的请求参数，MVC5能解析第一种和第三种格式的参数。
