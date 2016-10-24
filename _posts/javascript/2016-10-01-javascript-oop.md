---
layout: "z"
title: "OOP In JavaScript"
category: "javascript"
---

JavaScript是弱类型语言，它的OOP比CSharp要简单一些，其中的OOP有两个比较关键的点成员(prototype)和构造方法(constructor)。

    //构造方法
    Obj = function(a) {
        //公有成员
        this.A = a;
    };
    //公有方法
    Obj.prototype.method1 = function() { console.log('method1'); };
    //公有属性
    Obj.portotype.property1 = function() { console.log('property1'); return 'p1'; }
    //实例化和调用
    var obj = new Obj(123);
    //调用方法
    obj.method1();
    console.log(obj.property1);



##### prototype and constructor
constructor是一个对象开始的地方，当构造方法被调用的时候分配内存，初始化对象。
prototype是JavaScript的一个强制命名，prototype内主要是类对象可以继承的类属性和方法（共享属性和方法），当类的构造方法被调用时，类对应的prototype会被用作新对象的原型。JavaScript里面继承的实现需要借助这个机制来实现。

##### 继承
因为解释性语言编译的过程,JavaScript没有像Java和CSharp那样有简单的继承语法糖。
JavaScript的继承主要是对constructor和prototype的继承，对prototype里面的内容的重用，除了prototype之外还可以用call和apply来实现。
对上面的Obj进行派生子类：
    //prototype方式实现
    Child = function(a) {
        this.A = a;
    }
    Child.prototype=Obj.prototype;
    /*
     *apply方式
     * Child = function(a) {
     *    Obj.apply(this,[a]);
     *    this.A = a;
     * }
     */
    //重写
    Child.prototype.method1 = function() {
        console.log('override method1');
    }
    //调用
    var child = new Child('123');
    child.method1();


##### 多态
JavaScript的多态可以用call和apply来实现，而call和apply的区别也仅仅只是参数上的区别，下面是以apply为例的实现：
    Test = function(o) {
        this.O = o;
    }
    Test.prototype.method1 = function() {
        O.method1.apply(this,[]);
    }
    var t1 = new Test(new Obj(123));
    var t2 = new Test(new Child(123));
    t1.method1();
    t2.method1();
