---
title: .NET复习
date: 2023-03-12 23:27:39
excerpt: .NET复习
categories: 学习
---

# NET

# 一.前端部分

## 1.VScode 常用插件

- `Live Server` **搭建具有实时加载功能的小型服务器**
- `Open-In-Browser`**直接在浏览器中查看页面**
- `HTML CSS Support`**HTML和CSS代码提示**
- `CSS Peak`**追踪至样式表中CSS类和id定义的地方**
- `Prettier-Code formatter`**代码格式化工具**
- `JavaScript（ES6)CODE SNIPPETS`**支持ES6和JavaScript代码片段插件**
- `Color Info`**CSS中使用颜色的相关信息**

## 

## 2.HTML

#### ◼html部分特殊符号

| 显示结果 | &nbsp  |
| :------: | :----: |
|   空格   | &nbsp  |
|    <     |  &lt   |
|    >     |  &gt   |
|    &     |  &amp  |
|    £     | &pound |
|    ¥     |  &yen  |
|    ©     | &copy  |
|    ®     |  &reg  |
|    ™     | &trade |
|    ×     | &times |

#### ◼html表格基本结构

```html
    <table border="1" width="600px">
        <caption>学生名单</caption>
        <tr>
            <th>学号</th>
            <th>姓名</th>
            <th>院系</th>
            <th colspan="2">操作</th>
        </tr>
        <tr>
            <td>2021001</td>
            <td>小明</td>
            <td>计算机学院</td>
            <td>编辑</td>
            <td>删除</td>
        </tr>
    </table>
```

- 每个`tr`表示一行每个`th`代表这一行中的每个元素值（第一行）`td`代表这一行中的每个元素值(非第一行)
- `colspan=3`表示这个一次包含3纵列
- `rowspan=4`表示这个一次包含4行



#### ◼form表单基本结构

```html
<form action="data.html" method="get">
        用户名：<input type="text" name="nusername"
        <br>
        密码：<input type="password" name="pwd"><br/>
        <input type="submit" value="提交"<br/>
        <input type="reset" value="重填"
</form>
```

- `http://127.0.0.1:5500/data.html?nusername=123&pwd=456`这里将值传入了data.html，值与值之间用**<u>&</u>**隔开
- `action=xxxx.xxxx`表示将这分表单的数据传入**xxxx.xxxx**
- `methon='get'`表示传入方法是`get`方法，同理还有`post`方法



#### ◼一些form表单元素及其属性作用

```html
<form>
        用户名：
        <input type="text" name="username" value="liulei" title="提示信息"
        <br/>
        密码：
        <input type="password" name="pwd" maxlength="8" placeholder="长度8个字符"
</form>
```

- `name="username"`表示将数据提交时url显示的参数名字是**username**
- `value=liulei` 属性的值表示的是`输入框中显示的初始值`
- `title=提示信息`鼠标移上去后会弹出“**<u>提示信息</u>**”
- `maxlength=8`最大位数是8
- `placeholder="长度8个字符"`当没有输入时显示“**<u>长度8个字符</u>**”





```html
    <form>
        专业特长：<br/>
        <select name="master" size="4" multiple="multiple">
            <option value="0">.NET</option>
            <option value="1">J2EE</option>
            <option value="2">Java</option>
            <option value="3">Android</option>
            <option value="4">C</option>
        </select>
    </form>
```

- `size` 属性规定下拉列表中可见选项的数目
- `multiple="multiple"`表示可以同时选择多个选项
- `value`属性如果我们选择的是**JAVA**那么我传入的数据就是**master=2**



```html
<form >
附件：<input type="file" name="myfile" accept="文件类型">
</form>
```

![图片](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151450148.png)





```html
<form>
	<input type="date" name="mydate">
</form>
```

![](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151450996.png)



## 3.css

#### ◼ CSS的一些单位

![图片-1667485429884](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151451939.png)

#### ◼常见选择器及优先级

| 选择器                     | 10000 |
| -------------------------- | ----- |
| style（内联样式）          | 1000  |
| id选择器                   | 100   |
| 类选择器、属性选择器、伪类 | 10    |
| 标签选择题、伪元素         | 1     |
| 通配符                     | 0     |

**一般来说：选择范围较大的级别较低**

#### ◼CSS 的 2 个示例



##### 1.悬浮变色、边框、阴影等效果

```css
div{
	background-color:lightgrey;
	width:130px;
	border-left:20px solid green /*做边框*/
	border-radius:5px;/*圆角*/
	padding:10px;
	margin:30px;
}
div:hover{
	background-color:rgb(151,248,39);
	cursor:pointer;
	box-shadow:3px 3px 5px 1px rgba(0,0,0.2);/*阴影*/
}
<div>武汉科技大学</div>
```

- `background-color`属性设置元素的背景颜色
- `width`设置段落宽度
- `border-left`设置左边框属性
- `border-radius`添加圆角边框
- `padding`内边距
- `margin`外边距
- `hover`选择鼠标指针浮动在其上的元素，并设置其样式
- `cursor:pointer`网页浏览时用户鼠标指针的样式或图形形状为一只伸出食指的手
- `box-shadow` 属性用于在元素的框架上添加阴影效果（ X 轴偏移量、Y 轴偏移量、模糊半径、扩散半径和颜色）



##### 2.简单导航

```html
<ul class="nav">
	<li><a href="#home">主页</a></li>
	<li><a href="#news">新闻</a></li>
	<li><a href="#contact">联系</a></li>
	<li><a href="#about">关于</a></li>
</ul>
```



```css
ul.nav {
	list-style-type: none;
}
.nav li {
	float: left;
}
.nav li a:link,
.nav li a:visited {
        display: block;
        width: 120px;
        text-align: center;
        padding: 4px;
        color: #fff;
        background-color: #98bf21;
        text-decoration: none;
}
.nav li a:hover,
.nav li a:active {
    background-color: #7a991a;
}
```

- `float:left`把图像像左浮动
- `list-style-type`设置列表样式类型
- `a:link`正常，未访问过的链接
- `a:visited`用户已访问过的链接
- `display:block`设置为块级元素
- `text-align`设置文本对齐方式
- `text-decoration`文本修饰

## 4.JavaScript



#### ◼ js基本特点

- JS是一种**<u>解释性脚本语言</u>**（代码不进行预编译，可在程序运行过程中逐行进行解释）
- JS是一种**<u>简单的弱类型脚本语</u>**言（未使用严格的数据类型
- JS是一种**<u>基于对象的语言</u>**（不仅可以创建对象，也能使用自身的对象或其他语言创建的对象）
- JS是一种**<u>跨平台脚本语言</u>**（不依赖于操作系统，仅需要浏览器的支持）

#### ◼JS箭头函数

**匿名函数可进一步用箭头函数来简化（箭头函数用`=>`定义，也成：lambda表达式**

```js
let foo=function(num){
    return num+1;
}
let a=foo(100);

//使用箭头函数

let foo =(num)=>num+1;
```

 **JS 数组操作：map、reduce、join、slice、splice、push、pop、shift、unshift 方法基本用法**



**JS 程序：五秒倒计时，跳转到学校官网**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <button id="ok">3秒后确定</button>
</body>
</html>
<script>
    let btn=document.getElementById("ok");
    btn.disabled=true;
    let t=3;
    function count(){
        t--;
        btn.innerHTML=t+"秒后确定";
        if(t<=0){
            clearInterval(id);
            btn.disabled=false;
            btn.innerHTML="确定";
            let url=window.location;
            window.loaction="https://www.wust.edu.cn"
        }
    }
    let id = setInterval(count,1000);
</script>
```

- `document.getElementById`方法可返回对拥有指定 ID 的第一个对象的引用
- `btn.disabled`禁用
- `btn.innerHTML` 属性设置或返回表格行的开始和结束标签之间的 HTML。
- `setInterval`方法可按照指定的周期（以毫秒计）来调用函数或计算表达式
- `clearInterval`方法可取消由 setInterval() 函数设定的定时执行操作
- `window.loaction`页面跳转

#### ◼DOM基本概念

**DOM:文档对象模型**

**DOM是载入到浏览器中的文档模型，以节点树的形式来表现文档，每个节点代表文档的构成部分**



#### ◼DOM常用操作

##### 1.查询操作

```html
<ul id="list">
<li class="item">1</li>
<li class="item">2</li>
<li class="item">3</li>
</ul>
```

```js
let list = document.getElementById('list') // 命中id="list"的元素，注意不带#，不是css选择器
console.log(list.innerHTML) //查看元素的HTML内容
let first = document.querySelector('.item') // 命中第一个 class="item" 参数为class选择器
console.log(first.innerHTML)
let second = document.querySelector('.item:nth-child(2)') // 命中第二个 .item 复杂的css选择器
console.log(second.innerHTML)
let items = document.querySelectorAll('.item') // 获得包含所有 .item 的集合（NodeList）
console.log(items.length) //查看集合长度
for ( let el of items ) { //遍历集合
console.log(el)
}
```

##### 2.创建操作：document.createElement

```html
<ul id="list">
<li class="item">1</li>
<li class="item">2</li>
<li class="item">3</li>
</ul>
```

```js
let list = document.createElement('ul') //创建1个ul元素
list.id = 'list' // 设置元素的id属性
for ( let i = 0; i < 3; i++ ) {
let item = document.createElement('li') //循环创建3个li元素
item.className = 'item' // 设置元素的class属性
item.innerText = `${i + 1}` // 设置元素的innerText属性 (使用了模板字符串)
list.appendChild(item) // 将创建好的元素添加到父节点
}
// 将list添加到body
document.body.appendChild(list)
```

#### JSON对象定义基本特点

- **<u>数据在键值对中（键名即属性名必须加双引号）</u>**
- **<u>数据由逗号分隔</u>**
- **<u>花括号保存对象</u>**
- **<u>方括号保存数组</u>**

**JSON可通过JavaScript进行解析，JSON的值可以是：数字 字符串 逻辑值 数组 对象 null**

**json不能存储Date对象，如果需要则用字符串表示**



#### JSON.parse程序示例：

**◼ JSON.parse(text [, reviver]) ：将JSON格式字符串转换为JavaScript对象**

**◼ 参数说明：**

- `text`：必需，一个有效的JSON字符串 (如格式不正确则解析会出错)
- `reviver`：可选，一个转换结果的函数， 将为对象的每个成员调用此函数

```js
let jstr = '{"name":"wust", "url":"www.wust.edu.cn","birthday":"1898-11-21"}';
let obj = JSON.parse( jstr, function(key, value) {
	if (key == "birthday") {
		let diff = new Date() - new Date(value); //计算距今的毫秒数
		let year = parseInt(diff / 1000 / 60 / 60 / 24 / 365); //相差的年数
		return year;
	}else{
        return value;
    }
})
console.log(jsonObj.birthday);
```

# 二.MVC后台部分

#### 1.MVC基本概念：

- MVC:是一种体系结构模式，他将应用程序分成3个主要组件：**<u>模型（Model）视图（View）和控制器（Controller）</u>**
- MVC模式有助于实现关注点分离：
- 关注点：**输入逻辑** **业务逻辑** **UI逻辑**

#### 2.MVC三个模块功能

- 控制器C：处理浏览器的请求，决定如何调用业务层数据的增、删、改、查等业务操作，以及如何将结果返回给视图进行渲染。
- 模型M：应用的实体类，用于在内存中暂时存储数据，并在数据变化时通知控制器。
- 视图V：主要用来解析、处理、显示内容，并进行模板的渲染。

#### 3.MVC体系结构优点：

◼ Controller与View完全分离(松耦合)，有利于前、后台分工合作

◼ 一个Model可建立多个视图，满足用户不同需求

◼ Model独立于视图，可移植到新的平台，代码重用高，易于维护

◼ 表现层的性能可以优化到极致

◼ 容易对界面逻辑进行单元测试

◼ 非常强大的URL映射组件，非常干净的URL来建造应用

◼ 有利于软件工程化管理

#### 4.MVC项目目录结构

![图片-1667485456621](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151451438.png)

#### 5.Razor基本概念和基本语法规则

`Razor概念`Razor是一个视图模板引擎，它提供了优雅的方法将基于服务器的代码（如C#）嵌入到HTML页面中

`Razor语法规则`

◼ C# 代码封装于 @{ ... } 中

◼ 代码语句以分号结尾

◼ 行内表达式（变量和函数）以 @ 开头

◼ C# 代码对大小写敏感

#### 6.ViewData传值特点

- ViewData是一个字典对象，用来从Controller向对应的View视图传值
- ViewData只在当前请求中有效，生命周期和View相同，其值不能在多个请求中共享
- 在重定向(新请求)后，ViewData存储的值将变为null
- 使用ViewData值时必须进行合适的类型转换和建议空值检查

#### 7.TrmpData传值特点

- TempData也是一个字典对象，但是基于Session存储机制
- TempData用在多个Action间或页面重定向(Redirection)时传递共享数据
- 但TempData存放的数据只一次访问中有效，一次访问完成后就会删除
- TempData用法和ViewData相同

#### 8.“新搭建基架的项目”时生成的一些内容：

![图片-1667485469907](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151451623.png)

#### 9.EF Core数据库迁移两个命令：

① **<u>Add-Migration InitialCreate</u>**

② **<u>Uppate-Database</u>**

#### 10.ORM的概念

> ORM (Object Relation Mapping) 是对象/关系映射，它将内存中的对象和数据库中的立映射关系

#### 11.ORM技术产生的背景原因

◼ 面向对象开发方法是当今企业级应用主流开发方法。

◼ 关系数据库是企业级应用永久存放数据的主流数据存储系统。

◼ 对象和关系数据是业务实体两种表现形式：业务实体在内存中表现为对象(非持久化存储)，在数据库中表

现为关系数据(持久化存储) 。

◼ 需要一种技术实现二者间映射，以简化编程，提高系统效率 

#### 12.模型的一些DateType注解

|     DataType类型值      |       说明       |
| :---------------------: | :--------------: |
|    DataType.Currency    |    表示货币值    |
|      DataType.Date      |    表示日期值    |
|  DataType.EmailAddress  | 表示电子邮件地址 |
| DataType.Multiline Text |   表示多行文本   |
|    DataType.Password    |    表示密码值    |
|      DataType.Time      |    表示时间值    |
|      DataType.Url       |  表示一个URL值   |

#### 13.模型的一些验证注解

◼ `[Required]`：验证字段是否不为 null

◼ `[StringLength]`：验证字符串属性值是否不超过指定的长度限制。

◼ `[Range]`：验证属性值是否位于指定范围内。

◼ `[Compare]`：验证模型中的两个属性是否匹配。

◼ `[RegularExpression]`：验证 属性值是否与指定的正则表达式匹配。

◼ `[EmailAddress]`：验证属性是否具有电子邮件格式。

注：`[DataType]`：只是帮助字段进行格式设置，不提供任何验证

#### 14.控制器方法的两个注解

◼ `[HttpPost] 注解：`表明只能由POST请求才能调用此Action方法，不写默认[HttpGet] (第一个Create就是GET)

◼ `[ValidateAntiForgeryToken] 注解：`用于防止请求伪造 (更安全)

#### 15.强类型传值

◼ 回顾：ViewData字典传值是一个弱类型传值方式 (使用时需要手工强转类型)

◼ 强类型传值则不需要手工强转类型

◼ 如何实现强类型传值：

		① 控制器在返回视图时，添加模型对象作为参数，即： return View(模型对象); 
	
		② 在视图中，先使用 @model 指令声明模型对象类型
	
		③ 然后在视图中使用Model对象来接收传来的模型对象，之后使用Model对象无需强转

#### 16.控制器编程

给出数据库表，以及模型类、数据库上下文类，查询所有记录或根据 id 查明细。控制器基本结构：

```
public class MoviesController : Controller
{
private readonly MvcMovieContext _context;
public MoviesController(MvcMovieContext context)
{
_context = context;
}
// …CRUD操作
}
```