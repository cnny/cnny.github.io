---
layout: post
title: Light7爬坑记录!
category: 前端
tags: light7
keywords:
description:
---

为缩短工期,节约设计及前端的时间,打算在项目中使用light7,可因为对light7的不熟悉,踩了无数坑,反而延误了工期.特此栽树,以供后人(以后的我)乘凉!

###每个.page都要设置一个独立的ID!
当去请求一个ajax页面时,light7会将目标页面的.page提取出来,并为其生成随机唯一ID,插入DOM.
这样会导致每取一次页面,就会在DOM中插入一次页面,在DOM中产生大量冗余.
解决办法是自己为每个.page赋予ID,
light7就不会为所取页面生成随机ID,当light7判断当前DOM中已存在此ID时,便不会插入新的页面,而是将旧有页面替换

###javascript要写在.page元素内!
上面已经说了,light7只会抓取新页面.page内的内容.所以要把javascript写在.page中,当然,你也可以全部写在单独JS文件中,然后在入口文件引用.

###事件不要绑定在document上!
由于SPA DOM不刷新的特性,当一个页面被重复加载多次,该页面中的js会在DOM重复绑定多次.

###页面title需要通过特殊代码设置!
还是老原因,由于SPA DOM不刷新, 所以通过router刷新页面时的title并不会变更.微信浏览器可以用以下方法解决:

```
    // 修改页面标题
    function setPageTitle(title) {

      document.title = title;

      // 黑科技（iOS微信浏览器专用）
      // 原因：微信浏览器的title在页面加载完成后就定了，不再监听 window.title 的 onchange 事件。
      // 所以这里修改了title后，立即创建一个请求，加载一个空的iframe
      if (isWeixin && isIOS) {
        var iframe = document.createElement('iframe');
        iframe.setAttribute('src', '/favicon.ico');
        iframe.setAttribute('style', 'display: none; width:0; height:0;');
        var handler = function () {
          setTimeout(function () {
            iframe.removeEventListener('load', handler);
            document.body.removeChild(iframe);
          }, 10);
        }
        iframe.addEventListener('load', handler);
        document.body.appendChild(iframe);
      }
    }
```

###按钮的disabled属性
经过我的实践测试,light7中的按钮是有.disabled样式的,在官方文档中并未提及,使用时在按钮中加上class="disabled"即可.但要注意:该样式只会在默认主题中生效,原因还未研究.

###picker的onchange
在我们要实现级联效果时,picker的onchange是非常重要的,但官方文档中并未提供该事件.但代码中onchange是存在的,使用方法也可参考light7中的cityPicker.

##一些不仅是light7,移动开发中都常遇到的问题
###h5页面滑动不顺畅
解决方案:在要滑动的元素中加入以下样式

```
    overflow:scroll;
    -webkit-overflow-scrolling: touch;
```
