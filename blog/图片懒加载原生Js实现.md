# 思路
首先，什么是懒加载，从字面意思就可以简单的理解为不到用时就不去加载，对于页面中的元素，我们可以这样理解：只有当滚动页面内容使得本元素进入到浏览器视窗时（或者稍微提前，需给定提前量），我们才开始加载图片；

那么我们知道，当不给img元素的src属性赋值时，不会发出请求【不能使src=""，这样即使只给src赋了空值也会发出请求】，而一旦给src属性赋予资源地址值，那么该请求发出，使得图片显示；所以这里我们利用这一点控制img元素的加载时机。
**在开始的时候将资源url放置在自定义属性data-src当中，然后在需要加载的时候获取该属性并赋值给元素的src属性**

# 难点
## 视窗内元素判断
从上面的分析可以看出来，主要要解决的问题就是怎么检测到元素是否在视窗当中，这里我们要借助于dom操作api当中的el.getBoundingClientRect()来获取其位置，并判断是否在视窗内，这里简单描述。

**Element.getBoundingClientRect()方法返回元素的大小及其相对于视口的位置。返回值是一个 DOMRect 对象，这个对象是由该元素的 getClientRects() 方法返回的一组矩形的集合, 即：是与该元素相关的CSS 边框集合 。DOMRect 对象包含了一组用于描述边框的只读属性——left、top、right和bottom，单位为像素。除了 width 和 height 外的属性都是相对于视口的左上角位置而言的。**

![avatar](https://mdn.mozillademos.org/files/15087/rect.png)

因此我们可以使用以下逻辑判断元素是否进入视窗：
```
        // 判断指定el是否在视窗内
        function isInSight(el){
            var eldom = typeof el == 'object'?el:document.querySelector(el);
            var bound = eldom.getBoundingClientRect();
            // 这里的bound包含了el距离视窗的距离；
            // bound.left是元素距离窗口左侧的距离值；
            // bound.top是袁术距离窗口顶端的距离值；

            // 以以上两个数值判断元素是否进入视窗；
            return (bound.top>=0&&bound.left>=0)&&(bound.top<=window.innerHeight+20)&&(bound.left<=window.innerWidth+20);
        }
```
其中window.innerHeight和window.innerWidth分别为视窗的高度和宽度，之所以加上20是为了让懒加载稍稍提前，使用户体验更好；

## 添加scroll事件监听
那么什么时候去检测元素是否在视窗内，并判断是否加载呢，这里由于页面的滚动会使得元素相对于视窗的位置发生变化，也就是说滚动会改变isInSight的结果，所以这里我们在window上添加scroll事件监听：
```
        // 当加载完成，检测并加载可视范围内的图片
        window.onload= checkAllImgs;
        // 添加滚动监听，即可视范围变化时检测当前范围内的图片是否可以加载了
        window.addEventListener("scroll",function(){
            checkAllImgs();
        })

        // 检测所有图片，并给视窗中的图片的src属性赋值，即开始加载；
        function checkAllImgs(){
            var imgs = document.querySelectorAll("img");
            Array.prototype.forEach.call(imgs,function(el){
                if(isInSight(el)){
                    loadImg(el);
                }
            })
        }
        // 开始加载指定el的资源
        function loadImg(el){
            var eldom = typeof el == 'object'?el:document.querySelector(el);
            if(!eldom.src){
               // 懒加载img定义如：<div class="img"><img  alt="加载中" data-index=7 data-src="http://az608707.vo.msecnd.net/files/MartapuraMarket_EN-US9502204987_1366x768.jpg"></div>
                var source = eldom.getAttribute("data-src");
                var index = eldom.getAttribute("data-index");
                eldom.src = source; 
                console.log("第"+index+"张图片进入视窗，开始加载。。。。")
            }
            
        }
```
这里就不考虑添加事件的各种兼容性了。

这样就实现了图片的懒加载的简单实现，当然还可以对scroll进行优化等操作，这里不做过多阐述了。

# 演示地址

[（LazyLoadForImg）图片的懒加载](https://codepen.io/hustchen/pen/ZydbRg)