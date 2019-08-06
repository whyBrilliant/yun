# JS随记

## 数组

### 1. 无loop生成指定长度的数组

```
const List = len => [...new Array(len).keys]
const list = List(10)
```

## 字符串

## Number

## Boolean

## Function

## JS常见的技术

### 懒加载

```
<ul>
	<li><img src="./img/default.png" data="./img/1.png" alt=""></li>
	<li><img src="./img/default.png" data="./img/2.png" alt=""></li>
	<li><img src="./img/default.png" data="./img/3.png" alt=""></li>
	<li><img src="./img/default.png" data="./img/4.png" alt=""></li>
	<li><img src="./img/default.png" data="./img/5.png" alt=""></li>
	<li><img src="./img/default.png" data="./img/6.png" alt=""></li>
	<li><img src="./img/default.png" data="./img/7.png" alt=""></li>
	<li><img src="./img/default.png" data="./img/8.png" alt=""></li>
</ul>

const imgs = document.querySelectorAll('img')
// 窗口可视区高度
const clientHeight = window.innnerHeith || document.documentElement.clientHeight || document.body.clientHeight;
// img 距离窗口可视区顶部的距离 imgs[i].getBoundingClientRect().top
function lazyLoadImg() {
  for (let i = 0; i < imgs.length; i++ ) {
    if ((imgs[i].getBoundingClientRect().top + imgs[i].height) >= 0 && imgs[i].getBoundingClientRect().top < clientHeight) {
      imgs[i].src = imgs[i].getAttribute('data')
    }
  }
}
window.addEventListener('scroll', lazyLoadImg)
```

## 临时记录，待验证

### 解决键盘弹出后挡表单的问题

```
window.addEventListener('resize', function () {
if (
  document.activeElement.tagName === 'INPUT' ||
  document.activeElement.tagName === 'TEXTAREA' ||
  document.activeElement.getAttribute('contenteditable') == 'true'
) {
  window.setTimeout(function () {
    if ('scrollIntoView' in document.activeElement) {
      document.activeElement.scrollIntoView();
    } else {
      // @ts-ignore
      document.activeElement.scrollIntoViewIfNeeded();
    }
  }, 0);
}
})

```

### 移动端打开指定App或者下载App

```
navToDownApp() {
  let u = navigator.userAgent
  if (/MicroMessenger/gi.test(u)) {
    // 如果是微信客户端打开，引导用户在浏览器中打开
    alert('请在浏览器中打开')
  }
  if (u.indexOf('Android') > -1 || u.indexOf('Linux') > -1) {
    // Android
    if (this.openApp('en://startapp')) {
      this.openApp('en://startapp') // 通过Scheme协议打开指定APP
    } else {
      //跳转Android下载地址
    }
  } else if (u.indexOf('iPhone') > -1) {
    if (this.openApp('ios--scheme')) {
      this.openApp('ios--scheme') // 通过Scheme协议打开指定APP
    } else {
      // 跳转IOS下载地址
    }
  }
},
openApp(src) {
  // 通过iframe的方式试图打开APP，如果能正常打开，会直接切换到APP，并自动阻止a标签的默认行为
  // 否则打开a标签的href链接
  let ifr = document.createElement('iframe')
  ifr.src = src
  ifr.style.display = 'none'
  document.body.appendChild(ifr)
  window.setTimeout(function() {
    // 打开App后移出这个iframe
    document.body.removeChild(ifr)
  }, 2000)
}

```