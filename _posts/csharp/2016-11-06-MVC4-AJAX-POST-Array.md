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
