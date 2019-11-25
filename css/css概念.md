# CSS

## 常见约束

1.建议使用padding代替margin

 (1) 属于同一个BFC的两个相邻Box的margin会发生重叠，所以如果margin使用的过于频繁，Box的垂直距离可能就会发生重叠。

  (2) 所有毗邻的两个或者多个元素的margin将会合并为一个margin共享。比如第一个元素的margin-top的值会加在父元素上。


##  px | em | rem | %
在 pc 端不需要考虑的这么复杂，所以这里我们主要讲讲这些单位在移动端中的使用

### 基础单位px

1. 比较小的图案

- 比如需要我们画一个 r 为 5px 的圆，如果我们使用 rem 作为单位，我们很快会发现在一些机型上的图案不圆，会呈现椭圆形。这是由于 rem 转 px 会存在精度丢失问题。所以这个时候我们就需要使用 px 配合 dpr 来实现

```
// less 
/*@size 建议取双数*/
.circle(@size, @backgroundColor) {  
    width: @size;
    height: @size;
    background-color: @backgroundColor;
    [data-dpr="1"] & {
        width: @size * 0.5;
        height: @size * 0.5;
    }
    [data-dpr="3"] & {
        width: @size * 1.5;
        height: @size * 1.5;
    }
}
```

### rem
以 rem 单位设计的弹性布局，是需要在头部加载一段脚本来进行监听分辨率的变化来动态改变根元素字体大小，使得 CSS 与 JS 耦合了在一起。

rem 是 CSS3 新增的一个相对单位（root em），即相对 HTML 根元素的字体大小的值。rem 应该是自适应使用的最广泛的单位了。

一般情况字体的大小我也会使用 rem 作为单位，因为精度丢失我认为在可以接受的范围之内。

### em

em 也是一个相对单位，却是相对于当前对象内文本的字体大小。

一般建议在 line-height 使用 em。因为在需要调整字体大小的时候，只需修改 font-size 的值，而 line-height 已经设置成了相对行高了。在存在首行缩进的需求，也会长使用这个单位。

### vw | vh
vw | vh 可以很好的解决rem使得css和JS耦合在一起的问题。

以下是网上的方案
```
$vm_fontsize: 75;
@function rem($px) {
     @return ($px / $vm_fontsize ) * 1rem;
}
$vm_design: 750;
html {
    font-size: ($vm_fontsize / ($vm_design / 2)) * 100vw; 
    @media screen and (max-width: 320px) {
        font-size: 64px;
    }
    @media screen and (min-width: 540px) {
        font-size: 108px;
    }
}
// body 也增加最大最小宽度限制，避免默认100%宽度的 block 元素跟随 body 而过大过小
body {
    max-width: 540px;
    min-width: 320px;
}
```

## 变量和Mixin

### 变量

- sass和less变量

```
// sass
$direction: left;
// less
@direction: left;
```
- css 变量
1. 变量定义的语法是：--*; // *是变量名称
2. 变量使用的语法是: var();
3. 无论是变量的定义还是使用都只能在声明块{}里面
4. CSS变量字符限制为[0-9]、[a-zA-Z]、_、-、中文和韩文等。

```
:root {
    --blue_color: #3388ff;
    --main_bgcolor: #fafafa;
    --font_size_12: 12px;
    --font_size_14: 14px;
    --color: 20px;
}
.div1{
    background-color: var(--main_bgcolor);
    font-size: var(--font_size_12);
}
```

### Mixin
和重复变量一样，重复的样式也可以归类。之前我们可能将可复用的样式写在一个个class中，这样的确达到了一定的复用性，不过最后的效果就是一个元素里放了很多的类名。如

```
<div class="a b c d e f ...">{children}</div>
```

使用Mixin的方式

```
.font-description {
    .font-des-style(24px,#fff,1.5em);
    .line-camp(2);
}

// less
/* 多行显示 */
.line-camp( @clamp:2 ) {
    text-overflow: -o-ellipsis-lastline;
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: @clamp;
    -webkit-box-orient: vertical; 
}

.font-des-style( @fontSize, @color, @lineHeight, @textAlign:left ) {
    font-size: @fontSize;
    color: @color;
    line-height: @lineHeight;
    text-align: @textAlign;
}
```

## 属性

### box-sizing

我们在设置元素的 border 和 padding 并不希望改变元素的 width,height 值，这个时候我们就可以为该元素设置 box-sizing:border-box;
我不希望每次都重写一遍，而是希望他是继承而来的，那么我们可以使用如下代码：
```
html {
  box-sizing: border-box;
}
*, *:before, *:after {
  box-sizing: inherit;
}
```
这样的好处在于他不会覆盖其他组件的 box-sizing 值，又无需为每一个元素重复设置 box-sizing: border-box;

## 选择器

### nth-child

#### 所有li元素中间添加border

```
.nav li {
  border-right: 1px solid #666;
}

.nav li:last-child {
  border-right: none;
}

// 还可以使用:not()伪类来达到同样效果
.nav li:not(:last-child) {
  border-right: 1px solid #666;
}
```

#### 使用负的nth-child可以选择1至n个元素

```
显示前三个li

// 方式一：
li {
  dispaly: none;
}

li:nth-child (-n+3) {
  display: block;
}

// 方式二:
li {
  display: block;
}
li:not(:nth-child(-n+3)) {
  display: none;
}

```

#### nth-of-type 选择器匹配同类型中的第n个同级兄弟元素
eg: 每行4个元素，第四个元素不显示border-right
```
li:nth-of-type(4n) {
  border-right: none;
}
```

### 属性选择器
1. a元素没有文本内容，但有href属性的时候，显示它的href属性


## 布局

### table布局

格子等宽的表格

```
.table {
  width: 100%;
  table-layout: fixed;
  height: 20px;
  background: #ccc;
}
```



## flex

> Flex 是 Flexible Box 的缩写，意为"弹性布局"，任何一个容器都可以指定为 Flex 布局。 采用 Flex 布局的元素，称为 Flex 容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"。 

注意： 设为 Flex 布局以后，子元素的`float`、`clear`和`vertical-align`属性将失效。 

### 基本概念

容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做`main start`，结束位置叫做`main end`；交叉轴的开始位置叫做`cross start`，结束位置叫做`cross end`。

项目默认沿主轴排列。单个项目占据的主轴空间叫做`main size`，占据的交叉轴空间叫做`cross size`。



### 容器属性

```

1. flex-direction
2. flex-wrap
3. flex-flow
4. justify-content
5. align-items
6. align-content

```

1. flex-direction 属性决定主轴的方向（即项目的排列方向）。 

   ```
   .box {
     flex-direction: row | row-reverse | column | column-reverse;
   }
   ```

2.  flex-wrap 属性定义，如果一条轴线排不下，如何换行。 

   ```
   .box{
     flex-wrap: nowrap | wrap | wrap-reverse;
   }
   ```

3.  flex-flow 属性是`flex-direction`属性和`flex-wrap`属性的简写形式，默认值为`row nowrap`。 

   ```
   .box {
     flex-flow: <flex-direction> || <flex-wrap>;
   }
   ```

4. justify-content 属性定义了项目在主轴上的对齐方式。 

   ```
   .box {
     justify-content: flex-start | flex-end | center | space-between | space-around;
   }
   ```

5. align-items  属性定义项目在交叉轴上如何对齐。 

   ```
   .box {
     align-items: flex-start | flex-end | center | baseline | stretch;
   }
   
   flex-start：交叉轴的起点对齐。
   flex-end：交叉轴的终点对齐。
   center：交叉轴的中点对齐。
   baseline: 项目的第一行文字的基线对齐。
   stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。
   
   ```

   

6. align-content

   ```
   
   .box {
     align-content: flex-start | flex-end | center | space-between | space-around | stretch;
   }
   
   flex-start：与交叉轴的起点对齐。
   flex-end：与交叉轴的终点对齐。
   center：与交叉轴的中点对齐。
   space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
   space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
   stretch（默认值）：轴线占满整个交叉轴。
   
   ```

   

### item的属性

```
1. flex-grow
2. flex-shrink
3. flex-basis
4. flex
5. order
6. align-self
```

1. flex-grow  属性定义项目的放大比例，默认为`0`，即如果存在剩余空间，也不放大。 

   ```
   .item {
     flex-grow: <number>; /* default 0 */
   }
   ```

    如果所有项目的`flex-grow`属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的`flex-grow`属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍。 

   

2. flex-shrink 属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。 

   ```
   .item {
     flex-shrink: <number>; /* default 1 */
   }
   ```

   如果所有项目的`flex-shrink`属性都为1，当空间不足时，都将等比例缩小。如果一个项目的`flex-shrink`属性为0，其他项目都为1，则空间不足时，前者不缩小。负值对该属性无效。

   

3. flex-basis  属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为`auto`，即项目的本来大小。 

   ```
   .item {
     flex-basis: <length> | auto; /* default auto */
   }
   ```

    它可以设为跟`width`或`height`属性一样的值（比如350px），则项目将占据固定空间。 

   

4. flex  属性是`flex-grow`, `flex-shrink` 和 `flex-basis`的简写，默认值为`0 1 auto`。后两个属性可选。 

   ```
   .item {
     flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
   }
   ```

   该属性有两个快捷值：`auto` (`1 1 auto`) 和 none (`0 0 auto`)。

   建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值。



5. order 属性定义项目的排列顺序。数值越小，排列越靠前，默认为0。 

   ```
   .item {
     order: <integer>;
   }
   ```



6. align-self  属性允许单个项目有与其他项目不一样的对齐方式，可覆盖`align-items`属性。默认值为`auto`，表示继承父元素的`align-items`属性，如果没有父元素，则等同于`stretch`。 

   ```
   .item {
     align-self: auto | flex-start | flex-end | center | baseline | stretch;
   }
   ```

    该属性可能取6个值，除了auto，其他都与align-items属性完全一致。 

   

