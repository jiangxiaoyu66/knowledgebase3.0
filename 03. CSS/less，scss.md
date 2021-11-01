https://www.cnblogs.com/wangpenghui522/p/5467560.html



## Less

```
运算，变量，类的嵌套，函数
```



### 3、运算

在`less`中，可以在书写属性时直接进行加减乘除

例子：`header`插入了一个`padding`

```
@fullWidth: 1200px;
.header{
    width: @fullWidth – 20px * 2;
    padding: 0px 20px*2;
}
复制代码
```

### 4、变量

#### (1)格式：以@开头

```
@headergray: #c0c0c0;
@fullWidth: 1200px;
@logoWidth: 35%;
复制代码
```

#### (2)字符串插值

```
@name: banner;
background: url("images/@{name}.png") no-repeat;
复制代码
```

编译：

```
background: url("images/banner.png") no-repeat;
复制代码
```

#### (3)避免编译

```
background: ~"red";
复制代码
```

编译：

```
background: red;
复制代码
```

#### (4)移动端rem布局中的使用

```
@fullWidth: 750;
@toRem: unit(@fullWidth/10,rem);
header{
    height: 150/@toRem;
}
复制代码
```

编译：

```
header{
    height: 2rem;
}
复制代码
```

### 5、混合

#### (1)在一个类中继承另一个类

```
.class1{
    color: red;
}
.class2{
    background: green;
    .class1;
}
复制代码
```

编译后：

```
.class1{
    color: red;
}
.class2{
    background: green;
    color: red;
}
复制代码
```

#### (2)用&替换当前选择器

```
a{
    color: #000;
    &:hover{
        color: #f00;
    }
}
复制代码
```

编译后：

```
a{
    color: #000;
}

a:hover{
    color: #f00;
}
复制代码
```

#### (3)在父类中嵌套子类

```
.class1{
    p{
        span{
            a{ }
        }
        &:hover{  }
    }
    div{  }
}
复制代码
```

编译后：

```
.class1{ }
.class1 p{ }
.class1 p span{
.class1 p span a{ }
.class1 p:hover{  }
.class1 div{  }
复制代码
```

#### (4)带参混合

```
.color(@color=red){
    color: @color;
}
.class1{
    .color(#0f0);
}
.class2{
    .color();
}
复制代码
```

编译后：

```
.class1{
    color: #0f0;
}
.class2{
    color: red;
}
复制代码
```

#### (5)块定义

```
@demo: {
	color: #f00;
}
body {
	@demo()
}
复制代码
```

编译后：

```
body {
	color: #f00;
}
复制代码
```

该方式和类继承的区别在于该块不会出现在编译的CSS中。

### 6、函数

#### (1)逻辑控制

- 格式：statement when(conditons)、prop: if((conditions),value);
- 例子1：在less中使用一个带参类名展示上下左右四个方向的纯CSS三角形

**index.less**

```
.base(){
    width: 0;
    height: 0;
}
@normal: 20px solid transparent;
@anger: 20px solid #f00;
.triangle(@val) when(@val=left){
    .base();
    border-left: none;
    border-right: @anger;
    border-top: @normal;
    border-bottom: @normal;
}
.triangle(@val) when(@val=right){
    .base();
    border-right: none;
    border-left: @anger;
    border-top: @normal;
    border-bottom: @normal;
}
.triangle(@val) when(@val=top){
    .base();
    border-left: @normal;
    border-right: @normal;
    border-top: none;
    border-bottom: @anger;
}
.triangle(@val) when(@val=bottom){
    .base();
    border-left: @normal;
    border-right: @normal;
    border-top: @anger;
    border-bottom: none;
}
.div1{
    .triangle(left);
}
.div2{
    .triangle(right);
}
.div3{
    .triangle(top);
}
.div4{
    .triangle(bottom);
}
复制代码
```

**index.html**

```
<!DOCTYPE html>
<html>
    <head>
        <link rel="stylesheet/less" href="index.less">
        <script src="../less-1.3.3.min.js"></script>
    </head>

    <body>
        <div class="div1"></div>
        <div class="div2"></div>
        <div class="div3"></div>
        <div class="div4"></div>
    </body
</html>
复制代码
```

- 例子2：

```
background: if( (true), #f00 );
复制代码
```

#### (2)循环

例子：将8个td的背景依次更换为bg_1.png、bg_2.png、…、bg_8.png

```
table td{
    width: 200px;
    height: 200px;
    .loop(@i) when(@i<9){
        &:nth-child(@{i}){
            background: url(~"../images/partner_@{i}.png") no-repeat;
        }
        .loop(@i+1);
    }
    .loop(1);
}
复制代码
```

#### (3)列表

```
@backgroundlist: apple, pear, coconut, orange;
复制代码
```

#### (4)less函数库

```
image-size(“bg.png”) //获取图片的Width和Height
image-width() //获取图片的Width和Height
image-height() //获取图片的Width和Height
convert(9s, ms) //转换9秒为毫秒
convert(14cm, mm) //转换14厘米为毫米
复制代码
```

更多函数参考官方函数库，包括混合函数、数学函数、字符串函数、列表函数等等

### 7、使用JS表达式

- less中还可以引用js表达式，不过一般都不推荐使用，此种方式在使用nodejs将less编译css时可能会报错。
- 格式：**`javascript`**
- 例子：将高度设置为当前获取到的浏览器的高度

```
@fullHeight: unit(` window.screen.height `, px);
div{
    height: @fullHeight;
}
复制代码
```

- 尝试将 **@width: unit(` window.screen.width `, px)** 引进 **vw布局** ？ 不推荐，不建议less在正式环境中使用，使用LESS时需要在头部引入js，而在js执行时的时候，会消耗时间，而less编译需要在js执行后，会在一定程度上影响到性能。



作者：failte
链接：https://juejin.cn/post/6844903798658301959
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
