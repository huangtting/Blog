# 如何判断网页已经滚动到页面底部？
![dataScroll](https://raw.githubusercontent.com/huangtting/Blog/master/images/dataScroll.png)
```
function scrollTop() { //获取页面顶部被卷起来的高度
return Math.max(
document.body.scrollTop, // 除了FireFox Opera
document.documentElement.scrollTop);//
}
function scrollHeight() { //获取页面的总高度
return Math.max(document.body.scrollHeight,
document.documentElement.scrollHeight);
}
function clientHeight() { //获取页面浏览器客户区的高度
//document.compatMode=BackCompat：标准兼容模式关闭；=CSS1Compat：标准兼容模式开启。
return (document.compatMode=="CSS1Compat")?document.documentElement.clientHeight
:document.body.clientHeight;
}
```
如果以下条件满足，说明网页已经滚动到(接近)页面底部：
```
scrollTop()+clientHeight()>=scrollHeight()-50px
```


# 如何判断一个图片是否出现在浏览器客户区？

![imageScroll](https://raw.githubusercontent.com/huangtting/Blog/master/images/imageScroll.png)

一个图片对象img的边框相对于浏览器客户区的位置可以采img.getBoundingClientRect()取到，img.getBoundingClientRect().top是相对客户区上边沿的距离，可以取负值
```
.top > 0 && .top < clientHeight ||
(.bottom) > 0 && (.bottom) < clientHeight)
```
判断一个图片出现在浏览器客户区的方法是img的上边框或下边框处于浏览器客户区