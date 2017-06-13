sass最佳实践
----------
### 1. sass是 世界上最成熟 最稳定 最强大的专业级扩展语言

提供 css缺失的样式层复用机制 减少冗余代码 提高样式代码的可维护性，除了sass之外 还有less postcss stylus

为什么要使用 css 预处理器 较为方便的实现浏览器兼容 变量定义 结构体之类的 代码更加简洁易于维护

### 2.使用

sass 有两种实现 一个基于ruby，ruby-sass 一个机遇c++， lib-sass 也就是我们常用的node-sass

```
npm  i node-sass -g


node-sass *.scss:*.css

// 监听文件变动自动 编译
node-sass --watch index.scss mian.css
```

**定义变量**
全局变量末尾加 !global标识符
**#{}** 实现动态插值 类似于模板字符串

```
$width: 20px;
$title-color: #444;
$prefixCls: left-menu;

// 声明一串值列表
$myColor: #F4F4F4, #CCC, gray;

.#{$prefixCls}-text{
    $border-style: solid !global;
    color: $title-color;
    width: $width;
}

// nth 取值 从位置0开始
.footer{
    background-color: nth($myColor, 0);
}
.header{
    color: nth($myColor, 1);
}

.box{
    border: $width/20 $border-style #CCC;
}
.p {
    font-size: $width/$width;
}
.span {
    font: #{$width-4}/#{$width} 'Consolas';
}
```
scss 文件是从上到下编译 所以变量应先声明后使用
后于声明的同名变量会覆盖之前声明过的。如果是被包裹起来，则不会影响前面的值。

**嵌套** 避免重复输入父选择器 结构清晰更易于管理

```


.main {
    width: 97%;
    .target, .title {
        font-size: 2em;
        a {
            font-weight: bold;
        }
    }
    .footer {
        font-size: 3em;
    }
}
```
编译后生成后代选择器

_更好的做法_ 使用 **&** 引用父选择器，将父选择器的类名作为css类名前缀

```
.main {
    width: 97%;
    &-target,
    &-title {
        font-size: 2em;
        a {
            font-weight: bold;
        }
    }
    &-footer {
        font-size: 3em;
    }
}

```

###why

考虑后代选择器
.content .header h3{
}
![dom树][1]


#### ？匹配规则

先找到所有的h3，沿着h3的父元素查找.header，中途找到了符合匹配规则的节点就加入结果集；如果直到根元素html都没有匹配，则不再遍历这条路径，从下一个h3开始重复这个过程（如果有多个最右节点为h3的话）。

@at-root


**属性嵌套**

```
.box{
    border:{
        width: 2px;
        color: #CCC;
        style: solid;
    };
    margin: 20px {
        left: 10px;
        bottom: 10px;
    }
}

```

#### 延伸
```

$myBlue: #12B7F5;
.footer{
    background-color: $myBlue;
}
.header{
    @extend .footer;
}

.hover{
    background-color:  lighten($myBlue, .5);
    color: #FFF;
}

// 多重延升

.content{
    color: #333;
    display: flex;
    @extend .footer;
    @extend .hover;
}


.article{
    @extend .content;
}


// 占位符 extend 配合 占位符

%center {
    display: block;
    margin-left: auto;
    margin-right: auto;
}

.container {
    line-height: 14px;
    @extend %center;
}

.image-cover {
    font-size: 16px;
    @extend .container;
}


```

#### 混合

```
@mixin large-text($size: 20px){
    font: {
        size: $size;
        weight: bold;
    }
    color: #444;
}


.card{
    &-title{
        @include large-text;
    }
    &-sub{
        @include large-text(16px);
    }
}

// 一串值列表作为参数
$value: 22px, #FCFCFC;

.text{
    @include large-text($value...);
}


// 利用@content 向mixin 导入内容
@mixin root($width: 200px){
    html{
        @content;
        background-color: #CCC;
    }
    div{
        width: $width;
    }
}

@include root(300px){
    font-size: 50px;
    p{
        color:#333;
    }
}

```


#### 函数
```
// 内置函数
rgba(#102030, .5)
rgba(blue, .4)

lighten(#CCC, .4);
darken(#CCC, .6);

// 自定义函数
@function calculate-width($col){
    @return 100% / $col
}

.my-title{
    width: calculate-width(2);  // width 50%
}


#### 循环

$icon-list: cross, tick, exam, plus, eye, edit;

@each $item in $icon-list{
    .#{$item}-icon{
        backgroud-image: url('./images/#{$item}.png');
    }
}


```

### Tips

#### 1. 构造合适的scss文件结构
尽量划分成若干个更小语义更明确的代码块 易于管理维护和扩展
```
Page/
|-- home.scss
|-- ...
base/
|-- _variables.scss
|-- _mixins.scss
|-- _placeholders.scss
global.scss
```

#### 2. 合适的选择@extend 与 @mixin

@mixin 适合那些需要通过传递参数快速创建的代码块
@extend 适合共同且复用的代码块

```
@mixin flex-basic($direction: row, $jc: center, $ai: center){
    display: flex;
    flex-direction: $direction;
    justify-content: $jc;
    align-items: $ai;
}

.wrapper{
    @include flex-basic(row, center, flex-end)
}


%placehoder{
    font-size: 16px;
    color: #444;
    margin: 0 10px;
    line-height: 10px;
    text-align: center;
}

.box-text{
    @extend %placehoder
}

```

#### 3.减少嵌套
一般嵌套都不应该超过三层，过度的嵌套会导致很多问题的发生，代码内容变的复杂，太过于依赖html, 一次结构上的改动都得重新组织样式代码，就常常导致需要引入!important的情况。

- 嵌套不要超过三级
- 保证css的简洁 可重用

@at-root 和 引用父级作为命名空间

  [1]: http://iexam-10034140.cos.myqcloud.com/1497705391673_WX20170617-210959.png
