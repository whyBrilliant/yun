# 动画

浏览器中动画有两种实现形式：通过申明元素实现（如SVG中的元素）和脚本实现。

可以通过setTimeout和setInterval方法来在脚本中实现动画，但是这样效果可能不够流畅，且会占用额外的资源.它们有如下特征

1. 即使向其传递毫秒为单位的参数，它们也不能达到ms的准确性。这是因为javascript是单线程的，可能会发生阻塞。

2. 没有对调用动画的循环机制进行优化

3. 没有考虑到绘制动画的最佳时机，只是一味地以某个大致的事件间隔来调用循环。

>浏览器重绘频率一般会和显示器的刷新率保持同步。大多数浏览器采取 W3C 规范的建议，浏览器的渲染页面的标准帧率也为 60FPS（frames/ per second）

  显示器有固定的刷新频率（60Hz 或 75Hz），也就是说，每秒最多只能重绘 60 次或 75 次，requestAnimationFrame 的基本思想让页面重绘的频率与这个刷新频率保持同步。比如显示器屏幕刷新率为 60Hz，使用requestAnimationFrame API，那么回调函数就每1000ms / 60 ≈ 16.7ms执行一次；如果显示器屏幕的刷新率为 75Hz，那么回调函数就每1000ms / 75 ≈ 13.3ms执行一次

  使用setInterval或setTimeout来实现主循环，根本错误就在于它们抽象等级不符合要求。我们想让浏览器执行的是一套可以控制各种细节的api，实现如“最优帧速率”、“选择绘制下一帧的最佳时机”等功能。但是如果使用它们的话，这些具体的细节就必须由开发者自己来完成。

## Api

### requestAnimationFrame、cancelAnimationFrame

  requestAnimationFrame不需要使用者指定循环间隔时间，浏览器会基于当前页面是否可见、CPU的负荷情况等来自行决定最佳的帧速率，从而更合理地使用CPU

#### 名词说明

* 动画帧请求回调函数列表

每个Document都有一个动画帧请求回调函数列表，该列表可以看成是由< handle, callback>元组组成的集合。其中handle是一个整数，唯一地标识了元组在列表中的位置；callback是一个无返回值的、形参为一个时间值的函数（该时间值为由浏览器传入的从1970年1月1日到当前所经过的毫秒数）。 刚开始该列表为空。

* Document

Dom模型中定义的Document节点

* Active document

浏览器上下文browsingContext中的Document被指定为active document。

* browsingContext
浏览器上下文是呈现document对象给用户的环境。
浏览器中的1个tab或一个窗口包含一个顶级浏览器上下文，如果该页面有iframe，则iframe中也会有自己的浏览器上下文，称为嵌套的浏览器上下文。

* 当requestAnimationFrame

当requestAnimationFrame(callback)被调用时不会执行callback，而是会将元组< handle,callback>插入到动画帧请求回调函数列表末尾（其中元组的callback就是传入requestAnimationFrame的回调函数），并且返回handle值，该值为浏览器定义的、大于0的整数，唯一标识了该回调函数在列表中位置。

每个回调函数都有一个布尔标识cancelled，该标识初始值为false，并且对外不可见。浏览器在执行“采样所有动画”的任务时会遍历动画帧请求回调函数列表，判断每个元组的callback的cancelled，如果为false，则执行callback。

* cancelAnimationFrame

cancelAnimationFrame 方法用于取消先前安排的一个动画帧更新的请求。

当调用cancelAnimationFrame(handle)时，浏览器会设置该handle指向的回调函数的cancelled为true。

无论该回调函数是否在动画帧请求回调函数列表中，它的cancelled都会被设置为true。如果该handle没有指向任何回调函数，则调用cancelAnimationFrame 不会发生任何事情。

```
handle = requestAnimationFrame(callback);
cancelAnimationFrame(handle)
```

#### 处理模型

此处使用伪代码来说明“采样所有动画”任务的执行步骤：

```
var list = {};

var browsingContexts = 浏览器顶级上下文及其下属的浏览器上下文;

for (var browsingContext in browsingContexts) {
/*!将时间值从 DOMTimeStamp 更改为 DOMHighResTimeStamp 是 W3C 针对基于脚本动画计时控制规范的最新编辑草案中的最新更改，并且某些供应商仍将其作为 DOMTimeStamp 实现。较早版本的 W3C 规范使用 DOMTimeStamp，允许你将 Date.now 用于当前时间。
如上所述，某些浏览器供应商可能仍实现 DOMTimeStamp 参数，或者尚未实现 window.performance.now 计时函数。因此需要用户进行polyfill
*/
    //var time = 从1970年1月1日到当前所经过的毫秒数;
    var time = DOMHighResTimeStamp   //从页面导航开始时测量的高精确度时间。DOMHighResTimeStamp 以毫秒为单位，精确到千分之一毫秒。此时间值不直接与 Date.now() 进行比较，后者测量自 1970 年 1 月 1 日至今以毫秒为单位的时间。如果你希望将 time 参数与当前时间进行比较，请使用当前时间的 window.performance.now。



   var d = browsingContext的active document;   //即当前浏览器上下文中的Document节点
    //如果该active document可见
    if (d.hidden !== true) { 
        //拷贝active document的动画帧请求回调函数列表到list中，并清空该列表
        var doclist = d的动画帧请求回调函数列表
        doclist.appendTo(list);
        clear(doclist);
    }

    //遍历动画帧请求回调函数列表的元组中的回调函数
    for (var callback in list) {
        if (callback.cancelled !== true) {
            try {
                //每个browsingContext都有一个对应的WindowProxy对象，WindowProxy对象会将callback指向active document关联的window对象。
                //传入时间值time
                callback.call(window, time);
            }
            //忽略异常
            catch (e) {    
            }
        }
    }
}
```

示例

```
var id = null;

    function a(time) {
        console.log("animation");
        window.cancelAnimationFrame(id);    //不起作用
        id = window.requestAnimationFrame(a);
    }

    a();
```
1、执行a
（1）执行“a();”，执行函数a；
（2）执行“console.log("animation");”，打印“animation”；
（3）执行“window.cancelAnimationFrame(id);”，因为id为null，浏览器在动画帧请求回调函数列表中找不到对应的callback，所以不发生任何事情；
（4）执行“id = window.requestAnimationFrame(a);”，浏览器会将一个元组< handle, a>插入到Document的动画帧请求回调函数列表末尾，将id赋值为该元组的handle值；

2、a执行完毕后，执行第一个“采样所有动画”的任务
假设当前页面一直可见，因为动画帧请求回调函数列表不为空，所以浏览器会定期地加入一个“采样所有动画”的任务到线程队列中。
a执行完毕后的第一个“采样所有动画”的任务执行时会进行以下步骤：
（1）拷贝Document的动画帧请求回调函数列表到list变量中，清空Document的动画帧请求回调函数列表；
（2）遍历list的列表，列表有1个元组，该元组的callback为a；
（3）判断a的cancelled，为默认值false，所以执行a；
（4）执行“console.log("animation");”，打印“animation”；
（5）执行“window.cancelAnimationFrame(id);”，此时id指向当前元组的a（即当前正在执行的a），浏览器将当前元组的a的cancelled设为true。
（6）执行“id = window.requestAnimationFrame(a);”，浏览器会将新的元组< handle, a>插入到Document的动画帧请求回调函数列表末尾（新元组的a的cancelled为默认值false），将id赋值为该元组的handle值。

3、执行下一个“采样所有动画”的任务
当下一个“采样所有动画”的任务执行时，会判断动画帧请求回调函数列表的元组的a的cancelled，因为该元组为新插入的元组，所以值为默认值false，因此会继续执行a。
如此类推，浏览器会一直循环执行a。

#### 注意事项

1、在处理模型中我们已经看到，在遍历执行拷贝的动画帧请求回调函数列表中的回调函数之前，Document的动画帧请求回调函数列表已经被清空了。因此如果要多次执行回调函数，需要在回调函数中再次调用requestAnimationFrame将包含回调函数的元组加入到Document的动画帧请求回调函数列表中,从而浏览器才会再次定期加入“采样所有动画”的任务（当页面可见并且动画帧请求回调函数列表不为空时，浏览器才会加入该任务），执行回调函数。

下面代码只执行1次animate函数
```
var id = null;
             
function animate(time) {
    console.log("animation");
}

window.requestAnimationFrame(animate);
```

下面代码会一直执行animate函数
```
var id = null;

function animate(time) {
    console.log("animation");
    window.requestAnimationFrame(animate);
}
animate();
```

2. 如果在执行回调函数或者Document的动画帧请求回调函数列表被清空之前多次调用requestAnimationFrame插入同一个回调函数，那么列表中会有多个元组指向该回调函数（它们的handle不同，但callback都为该回调函数），“采集所有动画”任务会执行多次该回调函数。

例如下面的代码在执行“id1 = window.requestAnimationFrame(animate);”和“id2 = window.requestAnimationFrame(animate);”时会将两个元组（handle分别为id1、id2，回调函数callback都为animate）插入到Document的动画帧请求回调函数列表末尾。

因为“采样所有动画”任务会遍历执行动画帧请求回调函数列表的每个回调函数，所以在“采样所有动画”任务中会执行两次animate。

```
var id1 = null,
        id2 = null;
    function animate(time) {
        console.log("animation");
    }
    
    id1 = window.requestAnimationFrame(animate);
    id2 = window.requestAnimationFrame(animate);
```

[深入理解requestAnimationFrame](https://www.cnblogs.com/chaogex/p/3960175.html?utm_source=tuicool&utm_medium=referral)