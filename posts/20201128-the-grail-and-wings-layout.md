# 撸一个圣杯布局和双飞翼布局

> 发布日期：2020-11-28 08:50:02

淘宝首页一直在用的布局方式，怎么能过时呢~

<!--more-->

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201128-the-grail-and-wings-layout/the-gogoses-under-fuji-mountain.jpeg)

<center><font face="黑体" size=2>富士山下的鹅 | 2018-11-25 | 拍摄于 iphone 7p</font></center>

<br/>

## 从基础布局谈起

现在大多数网页的布局无非是两列布局（比如各种管理系统）或三列布局（比如淘宝）。而实现这种基本布局的的最佳方案无疑是经典的圣杯布局和双飞翼布局。为什么这两种布局比较好，我们先从一个简单的布局实现谈起。

我们先来看一个基础的三列布局代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      .header {
        height: 30px;
        background-color: aqua;
      }

      .footer {
        height: 30px;
        background-color: blue;
      }

      /* key code */
      .main {
        height: 300px;
      }

      .main .left {
        width: 150px;
        float: left;
        background-color: burlywood;
        height: 300px;
      }

      .main .middle {
        height: 300px;
        background-color: crimson;
      }

      .main .right {
        width: 150px;
        float: right;
        background-color: darkgreen;
        height: 300px;
      }
    </style>
  </head>
  <body>
    <div>
      <div class="header">Header</div>
      <div class="main">
        <div class="left">Left menu</div>
        <div class="right">Right menu</div>
        <div class="middle">Content</div>
      </div>
      <div style="clear: both;"></div>
      <div class="footer">Footer</div>
    </div>
  </body>
</html>
```

代码的如下：

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201128-the-grail-and-wings-layout/basic_layout_result.png)

这样我就通过简单 `float` 布局实现了一个三列布局，但是这里面有个问题。在页面中红色的部分是整个页面最重要的一部分，在代码中是`<div class="middle">Content</div>` ，这段代码在 Left menu 和 Right menu 的后面，根据 html 的渲染顺序，最重要的的 content  是最后才被进行渲染的，因此这不是最优的三列布局方式。

两外，当我们调整 Left menu 或 Right menu 的透明度的时候，我们会发现其实 Left menu 和 Right menu 是压在 Content 上面的，这样导致在 Content 内部布局的时候无形中是收到两侧 menu 元素的影响的，中间 Content 元素并不是一个完全独立的容器元素。

因此下面开始引申出了经典的圣杯布局和双飞翼布局。因为双飞翼布局其实是国内对圣杯布局的一种改进，两种布局的思路是有很大部分的重叠的，因此我们从相同的部分说起。

## 圣杯布局和双飞翼布局相同布局部分

首先我们先调整代码，让 `<div class="middle">Content</div>` 获得最高的渲染优先级，因此代码变为

```html
<div class="main">
  <div class="middle">Content</div> <!-- 从最下面移动到了最上面 -->
  <div class="left">Left menu</div>
  <div class="right">Right menu</div>
</div>
```

毫无疑问，布局肯定是错乱的状态，这里不再展示错乱显示。

接下来我们让 left middle 和 right 都左浮动，并且让 middle 的宽度为 100%

```css
.main {
  height: 300px;
}
.main .left {
  width: 150px;
  float: left; /* Changes */
  background-color: burlywood;
  height: 300px;
}
.main .middle {
  width: 100%; /* Changes */
  float: left; /* Changes */
  height: 300px;
  background-color: crimson;
}
.main .right {
  width: 150px;
  float: left; /* Changes */
  background-color: darkgreen;
  height: 300px;
}
```

我们得到如下布局：

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201128-the-grail-and-wings-layout/grial_layout_goon_01.png)

Left 和 Right 因为 Content 的宽度是 100% 的原因被挤下来了，因此我们要想办法让 Left 和 Right 上去。这里就用到了`-margin `的技巧。

<img src="https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201128-the-grail-and-wings-layout/grial_layout_goon_02.png" style="zoom:50%;" />

如上图，假如 Content 的宽度不是 100%，Content Left 和 Right 会浮动的在同一行，我们给 Left 的透明度设置为 0.5，然后再设置 `margin-left: -60px` 我们看到 Left 的一部分盖在了 Content 的上面，形成了一部分的重叠。

根据这个特性我们也给 Left 和 Right 设置 `-margin` 的外边距来实现 Left 和 Right 的归位。

```css
.main {
  height: 300px;
}
.main .left {
  width: 150px;
  float: left;
  margin-left: -100%; /* Changes */
  background-color: burlywood;
  height: 300px;
}
.main .middle {
  width: 100%;
  float: left;
  height: 300px;
  background-color: crimson;
}
.main .right {
  width: 150px;
  float: left;
  margin-left: -150px; /* Changes */
  background-color: darkgreen;
  height: 300px;
}
```

但是我们发现 Content 中的文字被压在了 Left 的后面，也就是说 Left 和 Right 会分别压住 Content 的左右两侧。

其实圣杯布局和双飞翼布局的布局就在如何解决这个问题这里区别的，下面我们分别来谈。

## 完成圣杯布局

我们首先在 mian 容器中设置 padding-left 和 padding-right 将 Left 和 Right 的位置让出来。然后分别 Left 和 Right 设置相对定位将本身分别移动到让出来的位置上，这样就完成了整体的布局。代码如下：

```css
.main {
  height: 300px;
  padding-left: 150px; /* Changes */
  padding-right: 150px; /* Changes */
}
.main .left {
  width: 150px;
  float: left;
  margin-left: -100%;
  position: relative; /* Changes */
  left: -150px; /* Changes */
  background-color: burlywood;
  height: 300px;
}
.main .middle {
  width: 100%;
  float: left;
  height: 300px;
  background-color: crimson;
}
.main .right {
  width: 150px;
  float: left;
  margin-left: -150px;
  position: relative; /* Changes */
  left: 150px; /* Changes */
  background-color: darkgreen;
  height: 300px;
}
```

结果如下：

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201128-the-grail-and-wings-layout/basic_layout_result.png)

## 完成双飞翼布局

双飞翼布局的理念是，不希望使用 position 定位的方式让出 Left 和 Right 的位置，而是通过往 Content 元素中在加入一个额外的 div，并且设置其左右 margin 从而让出 Left 和 Right 的位置。

代码如下：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      .header {
        height: 30px;
        background-color: aqua;
      }

      .footer {
        height: 30px;
        background-color: blue;
      }

      /* key code */
      .main {
        height: 300px;
      }

      .main .left {
        width: 150px;
        float: left;
        margin-left: -100%;
        background-color: burlywood;
        height: 300px;
      }

      .main .middle {
        width: 100%;
        float: left;
        height: 300px;
        background-color: crimson;
      }

      .main .middle .middle-inner {
        margin-left: 150px; /* Changes */
        margin-right: 150px; /* Changes */
      }

      .main .right {
        width: 150px;
        float: left;
        margin-left: -150px;
        background-color: darkgreen;
        height: 300px;
      }
    </style>
  </head>
  <body>
    <div>
      <div class="header">Header</div>
      <div class="main">
        <div class="middle">
          <!-- Changes -->
          <div class="middle-inner">Content</div>
        </div>
        <div class="left">Left menu</div>
        <div class="right">Right menu</div>
      </div>
      <div style="clear: both"></div>
      <div class="footer">Footer</div>
    </div>
  </body>
</html>
```

结果如下：

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201128-the-grail-and-wings-layout/basic_layout_result.png)

## 进一步优化

我们在以上代码中发现，main 容器中的元素的高度都是写死的，这对于开发来说是不够优雅和友好的。我们可以通过模拟等高的方法让其实现等高。

代码如下：

```html
<style>
  .header {
    height: 30px;
    background-color: aqua;
  }
  .footer {
    height: 30px;
    background-color: blue;
  }
  /* key code */
  .main {
    height: 300px;
    overflow: hidden;
  }
  .main .left {
    float: left;
    margin-left: -100%;
    background-color: burlywood;
    height: 300px;
    /* width: 150px; */ /* Changes */
    padding-bottom: 99999px; /* Changes */
    margin-bottom: -99999px; /* Changes */
  }
  .main .middle {
    width: 100%;
    float: left;
    background-color: crimson;
    /* height: 300px; */ /* Changes */
    padding-bottom: 99999px; /* Changes */
    margin-bottom: -99999px; /* Changes */
  }
  .main .middle .middle-inner {
    margin-left: 150px;
    margin-right: 150px;
  }
  .main .right {
    width: 150px;
    float: left;
    margin-left: -150px;
    background-color: darkgreen;
    /* height: 300px; */ /* Changes */
    padding-bottom: 99999px; /* Changes */
    margin-bottom: -99999px; /* Changes */
  }
</style>
```

## 总结

以上实现这两种经典布局方式并不是一成不变的，比如我们可以让 Left Right 和 Middle 同时向左浮动，我们也可以同时让其同时向右浮动实现，取决于代码的喜好。

这两种布局方式在兼容各个浏览器的基础上，实现了中间 Content 区域的优先渲染，是一种在大型复杂页面场景下比较完美的解决方案。

## Reference
[1]圣杯布局和双飞翼布局的理解和区别: https://www.cnblogs.com/jiguiyan/p/11425276.html  
[2]圣杯布局和双飞翼布局的区别与实现: https://www.jianshu.com/p/bd792382459d