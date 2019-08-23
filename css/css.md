# CSS

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