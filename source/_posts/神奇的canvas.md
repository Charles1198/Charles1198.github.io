---
title: 神奇的 Canvas
date: 2018-1-8 18:00:00
tags:
---
最近在浏览掘金网站的时候看到掘金小册中有一个很有（便）意（宜）思的册子:[如何使用 Canvas 制作出炫酷的网页背景特效](https://juejin.im/book/5a0ab8e2f265da43111fbab2/section)，便想到给我的博客添加一个炫酷的背景。顺便学习一下 canvas 这个元素的使用。

# 效果

最终效果就在[博客](http://jiayueji.cn)上就能看到啦。下面来说一下实现方式。

# 实现

建议对 canvas 还不了解的同学去掘金小册上学习学习，我这里不再讲解。

我的博客是用 Hexo 搭建的，使用了 [Archer](http://firework.studio/archer-demo/) 主题，博客的最上层样式作者定义在 layout.ejs 文件里。

```
<!DOCTYPE html>
<html>
   ...
    <div class="wrapper">
        ...
    </div>
    ...
</html>
```

既然是在 canvas 里面画炫酷的背景，那就需要在这里添加一个 canvas 元素，并且和 div:class="wrapper" 一样大。

改造 layout.ejs 文件，用一个 div 将 div:class="wrapper" 和我们的 canvas 包裹起来：

```
<!DOCTYPE html>
<html>
    ...
    <div id="container-wrapper-canvas" style="position:relative;">
        <div class="wrapper">
        ...
        </div>
        <canvas id="myCanvas" style="position:absolute;left:0;top:0;z-index:0;pointer-events:none;" />
        <script>
        </script>
        ...
    </div>
    ...
</html>
```

因为不想让 canvas 响应点击事件，所以在它的 style 里面加上：

```
pointer-events:none;
```

先定义一些变量（以下代码一股脑塞到 script 标签里就行啦）。

```
// 屏幕宽高
let container = document.getElementById('container-wrapper-canvas')
let WIDTH = container.offsetWidth
let HEIGHT = container.offsetHeight
// canvas
let canvas = document.getElementById('myCanvas')
let context = canvas.getContext('2d')
// 圆点数量
let roundCount = HEIGHT / 10
// 存放远点属性的数组
let round = []

// 令 canvas 铺满屏幕
canvas.width = WIDTH
canvas.height = HEIGHT
```

构造圆点位置颜色大小等属性

```
// 构造圆点位置颜色大小等属性
function roundItem(index, x, y) {
    this.index = index
    this.x = x
    this.y = y
    this.r = Math.random() * 4 + 1
    let alpha = (Math.floor(Math.random() * 5) + 1) / 10 / 2
    this.color = "rgba(0,0,0," + alpha + ")"
}
```

画圆点

```
// 画圆点
roundItem.prototype.draw = function() {
    context.fillStyle = this.color
    context.shadowBlur = this.r * 2
    context.beginPath()
    context.arc(this.x, this.y, this.r, 0, 2 * Math.PI, false)
    context.closePath()
    context.fill()
}
```

这里看着很熟悉，在做 android、iOS 开发自定义 View 的时候就遇到过类似的代码，在 draw() 函数里画图，这里我想到可以在移动端使用类似的方法画出类似的背景。

这个时候调用初始化函数就可以看到静态的一个个小圆点了

```
// 调用初始化函数
init();
function init() {
    for(var i = 0; i < roundCount; i++ ){
        round[i] = new roundItem(i,Math.random() * WIDTH,Math.random() * HEIGHT);
        round[i].draw();
    }
    animate()
}
```

为了让小圆点动起来，我们写下面的函数。

```
// 移动圆点
roundItem.prototype.move = function () {
    // y方向移动速度与圆点半径成正比
    this.y -= this.r / 20

    // x方向移动分两个方向，移动速度与圆点半径成正比
    if (this.index % 2 == 0) {
        this.x -= 0.08
    } else {
        this.x += this.r / 40
    }

    // 如果超出范围就把它拉回来
    if (this.y <= 0) {
        this.y += HEIGHT
    }
    if (this.x <= 0) {
        this.x += WIDTH
    }
    if (this.x >= WIDTH) {
        this.x -= WIDTH
    }

    this.draw()
}
```

```
// 定义动画
function animate() {
    context.clearRect(0, 0, WIDTH, HEIGHT);
    for (var i in round) {
        round[i].move()
    }
    requestAnimationFrame(animate)
}
```

这个时候就可以看到动态的一个个小圆点了。

是不是很炫酷呢。有空我再将它改造得更炫酷一点。
