---
layout: z
title: "JavaScript 作用域链"
category: "javascript"
---


> 每一段JavaScript代码（全局代码或函数）都有一个与之关联的作用域链（scope chain)。这个作用域链是一个对象列表或者链表，这组对象定义了这段代码“作用域中”的变量。
> 

当代码中需要用到一个变量的值时，它会去按顺序作用域链表中的每一个对象里面查找对应变量名的属性，如果链表中的所有对象都没有查找到对应变量名的属性，将会抛出一个引用错误，如果有则直接使用查找到的属性的值作为当前变量的值。


每个代码块有单独的一个作用域链表，链表内的对象首先是当前代码块内的参数、局部变量等活动对象，第二个对象是代码块外层代码块的活动对象，第三个对象是外层的外层代码块的活动对象，以此类推，gobal的作用域对象只有一个，就是gobal内的活动对象们。

当一个函数被调用时，一个与此函数对应的作用域会被创建，嵌套的方法之间的作用域链是完全独立的,同一个函数的每一次调用创建的作用域链也是独立的。


变量的作用域、变量的提前声明、闭包与作用域链有很紧密的关系。

##### 提前声明
    function test(o) {
        var i = 0;
        console.log(j);//j已经声明，但是没有初始化
        if (typeof o == "object") {
            var j = 0;
            for(var k=0; k < 10; k++) {
                console.log(k);
            }
            console.log(k);
        }
        console.log(j);//j已经声明，可能没有初始化
    }
    test(new Date());
    test();
![输出结果](https://github.com/lauriezc/note/raw/master/resource/image/55170F46-9F57-4415-A925-FD3C85696B6A.png)

######作用域



    //作用域
    cope = "global";
    function checkscope2() {
        scope = "local1";
        myscope = "local2";//定义了一个全局变量
        var myscope1 = "local3"//定义了一个checkscope2函数内部的局部变量
        return [scope, myscope, myscope1];
    }
    console.log(checkscope2());
    console.log(scope);
    console.log(myscope);
    console.log(myscope1);

![输出结果](https://github.com/lauriezc/note/raw/master/resource/image/AD759BC6-9F48-40C4-AA7B-02905405C435.png)

