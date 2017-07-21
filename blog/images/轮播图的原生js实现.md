# 实现原理
 首先我们看看最后实现的效果图：

 <iframe height='500' scrolling='no' title='轮播的原生js实现' src='//codepen.io/hustchen/embed/jwgmXm/?height=265&theme-id=0&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/hustchen/pen/jwgmXm/'>轮播的原生js实现</a> by hustchen (<a href='https://codepen.io/hustchen'>@hustchen</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>


**1.图片的显示，可通过左右两侧的按钮切换图片；**

**2.自动轮播，给定轮播时间，实现自动切换;**

**3.底部按钮跟随轮播变化，点击可跳转到相应图片;**

## 原理

这里我们从原理出发,轮播其实就是一行多图，依次将个图片移动到视窗当中，就像是小时后的胶片电影放映机一样：

![avatar](http://img.bitscn.com/upimg/allimg/c160802/14F10222506250-1W53.jpg)


## 容器及资源创建
也就是说我们要先定义一个容器（以下交视窗容器，container），给其固定的高宽，恰好是一张图片的尺寸，然后在其中再定义一个容器（list），与视窗等高，但是为了横排摆放我们的图片，需要设定其宽度为图片的宽度之和（container.clientWidth*图片张数），这里的div横排可以使用左浮动来实现，注意设定container的overflow为hidden，否则所有图片将横排显示了。

为了之后的调用方便，我们将图片的插入放在js当中处理，这样可以实现简单的函数调用创建一个轮播：
```
function setImgs(imgs){
    // 设定图片容器宽度，使得能够横排容下所有图片
    //（这里+1是将第一张复制到最后，实现无缝轮播）
    this.list.style.width = (imgs.length+1) + "00%";
    if(Object.prototype.toString.call(imgs)){
        var i=0;
        for(;i<imgs.length;i++){
            this.addItem(imgs[i],i);
        }
        // 将第一张添加到最后，实现无缝轮播
        this.addItem(imgs[0]);              
    }
    this.btns = document.querySelectorAll(".btns");
}

//位置
function addItem(src,index){
    // 新建图片容器，并添加到list容器中
    var item = document.createElement("div");
    item.setAttribute('class',"item");
    var img_el = document.createElement("img");
    img_el.setAttribute('src',src);
    item.style.width = this.width + "px";
    item.appendChild(img_el);
    this.list.appendChild(item);

    // 若调用时给定了index则放置在index张位置
    // 若没有则不需要添加（针对复制第一张放置于最后，不需要相应的按钮）
    if(index!==undefined){
        var li = document.createElement("a");
        li.setAttribute("data-index",index);
        li.setAttribute("href","javascript:void(0)");                           
        li.setAttribute("class","btns");
        (index==0)&&(li.setAttribute("class","btns cur"));
        this.circles.appendChild(li);
    }
                    
}
```
代码当中的this.width为container的宽度；至于为什么要在最后将第一张图片复制一份到最后，原因很简单，即消除首末之间的跳转转换的突兀，有这么一帧在中间，首末之间转换要经过这一帧，而我们在当中设定该帧转化时关闭动画效果，从而实现平滑过渡。如下中的move函数所示;

此外，我们还在底部添加了小圆点的效果，因此在添加图片的同时我们可以顺便吧小圆点（li）给加上；

## 切换实现
接下来要实现图片的向左向右切换，这里的原理就是list容器的left偏移来实现，所以list容器的position必须为absolute；改变list容器的left的代码如下：其中setBtns是切换完之后需要改变小圆点的状态的函数；
```
// 通过图片容器的left属性来实现平移切换
function move(offset){
    // 当前向左偏移
    var oldLeft = parseInt(this.list.style.left||0);
    // 将要实现的偏移
    var newLeft = oldLeft+offset;
    // 当前是最后一张了（实际上就是第一张的复制），
    // 不能再增大向左的偏移了，归零
    if(newLeft<-this.list.clientWidth+this.width){
        newLeft = 0;
    }
    // 左侧是空白了，转到最后去
    if(newLeft>0){
        newLeft = -this.list.clientWidth+this.width;
        // return;
    }
    // 当是第一张和最后一张之间的转化时
    // 关闭动画效果，避免横向过长动画移动，而且由于两者是一样的
    // 所以视觉上没有感觉
    if(Math.abs(newLeft-oldLeft)==this.list.clientWidth-this.width){
        this.list.style.transitionProperty = 'none';
    }else{
        this.list.style.transitionProperty = 'all';
    }
    this.list.style.left = newLeft + 'px';
    // 转化之后，设定对应的按钮样式
    this.setBtns((-newLeft/this.width)%this.btns.length);
}

// 设定当前情况下底部按钮的样式，index为当前视窗内的图片序号
function setBtns(index){
    // btns = document.querySelector(".btns");
    for(var i=0;i<this.btns.length;i++){
        if(i==index){
            this.btns[i].className = "btns cur";
        }else{
            this.btns[i].className = "btns";
        }
    }
}
```

##自动轮播
至于自动轮播，无非是使用setInterval来实现：

```
// 轮播设定函数
function play(){
    var self = this;
    self.timer = setInterval(function(){self.move(-self.width)},self.interval);
}

```

##模块化及调用
将上述功能函数进行封装：
```
function Casual(source){
    return {
        init:init,
        timer:null,
        width:'600px',
        setImgs:setImgs,
        addItem:addItem,
        move:move,
        setBtns:setBtns,
        play:play,
        interval:3000
    }
    function init(source){
        this.source = source;
        this.list = document.querySelector(".list");
        this.pre = document.querySelector(".pre");
        this.next = document.querySelector(".next");
        this.container = document.querySelector(".container");
        

        this.width = this.container.clientWidth;

        this.circles = document.querySelector('.circles');
        var self = this;
        // 添加左侧点击事件，即向左
        this.pre.addEventListener('click',function(){
            self.move(self.width);
        });
        // 添加右侧点击事件，即向左
        this.next.addEventListener('click',function(){
            self.move(-self.width);
        })
        // 添加鼠标进入到容器内部的响应事件，即停止滚动
        this.container.addEventListener('mouseenter',function(){
            clearInterval(self.timer);
        })
        // 添加鼠标离开容器的响应事件，即恢复滚动
        this.container.addEventListener('mouseleave',function(){
            self.play(self.interval);
        })
        // 给底部的按钮添加点击响应，即点击之后定位到相应的图片
        this.circles.addEventListener('click',function(e){
            var index = e.target.getAttribute("data-index");
            self.setBtns(index)
            self.list.style.left = -self.width*index+"px";
        });
        this.setImgs(this.source)
    // 开始轮播，轮播间隔为3500ms
    }

    ...

    ...
```

然后就可以进行简单的调用了，当然html和css还是需要提前写好的

**css**
```


*{
    padding: 0;
    margin: 0;
}
div{
    box-sizing: border-box;
}
.container{
    width: 800px;
    height:600px;
    position: relative;         
    overflow: hidden;
margin:auto;
}
.pre,.next{
    background: rgba(20,20,20,0.2);
    position: absolute;
    width: 50px;
    height: 100%;
    text-align: center;

    line-height:400px;  
    display: table;
}
a{
    font-size: 20px;
    text-decoration: none;
    color: gray;
    margin: auto; 

    display: table-cell;
    vertical-align: middle;
}
.pre{
    top:0;
    left:0px;
}
.next{
    top: 0;
    right:0;
}
.list{
    height: 100%;
    position: absolute;
    z-index: -1;
    left:0;

    transition-property: all;
    transition-duration: 1s;
    transition-timing-function: linear;
}
.item{
    height:100%;
    overflow: hidden;
    float: left;
}

.circles{
    height: 50px;
    width: 100%;
    padding-bottom: 10px;
    position: absolute;
    bottom: 0;
    display: flex;
    justify-content: center;
}
.btns{
    width: 20px;
    height: 20px;
    display: block;
    border-radius: 50%;
    background: rgba(23,33,33,.4);
    padding: 0;
    margin: 0 2px 0 2px;

    transition-property: all;
    transition-duration: 1s;
    transition-timing-function: linear;
}
.cur{
    background-color: rgba(22,23,22,0.8);
}
```

**html**
```
<div class="container">
    <div class="pre">
        <a href="javascript:void(0)">
            <svg class="icon" style="width: 1em; height: 1em;vertical-align: middle;fill: currentColor;overflow: hidden;" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="979"><path d="M182.06752 496.576l319.936-308.448c29.44-28.384 29.44-74.432 0-102.816a77.44 77.44 0 0 0-106.656 0L22.09952 445.152a70.88 70.88 0 0 0 0 102.816L395.34752 907.84a77.44 77.44 0 0 0 106.656 0c29.44-28.384 29.44-74.432 0-102.816l-319.936-308.448z m435.936 0l319.936-308.448c29.44-28.384 29.44-74.432 0-102.816s-77.216-28.384-106.656 0L458.03552 445.152a70.848 70.848 0 0 0 0 102.816L831.28352 907.84a77.44 77.44 0 0 0 106.656 0c29.44-28.384 29.44-74.432 0-102.816l-319.936-308.448z" p-id="980"></path></svg>
        </a>
    </div>
    <div class="next">
        <a href="javascript:void(0)">
            <svg class="icon" style="width: 1em; height: 1em;vertical-align: middle;fill: currentColor;overflow: hidden;" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4081"><path d="M604.568 513.578l-0.074-0.074 1.578-1.578-404.766-403.313c-10.993-10.811-26.083-17.487-42.732-17.487-0.107 0-0.214 0-0.321 0.001-33.898 0-61.462 27.453-61.462 61.243 0 8.053 1.504 15.87 4.486 23.178 4.697 11.207 12.257 20.43 21.778 27.022l309.167 309.418-317.86 316.717c-10.847 10.954-17.548 26.028-17.548 42.668 0 0.069 0 0.138 0 0.206 0 33.769 27.578 61.257 61.478 61.257 0.098 0 0.215 0.001 0.331 0.001 16.457 0 31.387-6.535 42.337-17.153l403.607-402.103z" p-id="4082"></path><path d="M966.484 509.837l-404.766-403.286c-10.992-10.803-26.077-17.474-42.72-17.474-0.115 0-0.233 0-0.347 0.001-33.897 0-61.461 27.453-61.461 61.243 0 8.053 1.504 15.87 4.486 23.178 4.697 11.207 12.257 20.43 21.778 27.022l309.181 309.404-317.872 316.704c-10.847 10.957-17.548 26.034-17.548 42.677 0 0.066 0 0.13 0 0.195 0 33.769 27.578 61.258 61.467 61.258 16.206 0 31.441-6.176 42.665-17.125l405.139-403.796z" p-id="4083"></path></svg>
        </a>
    </div>
    <div class="circles">
    </div>
    <div class="list">
    </div>
</div>
```

**js调用**
```
window.onload = function (){
    var source = [
        "http://az619822.vo.msecnd.net/files/CrescentCityConnection_EN-US11247361628_1366x768.jpg",
        "http://az619519.vo.msecnd.net/files/LagazuoiRefuge_EN-US12120955316_1366x768.jpg",
        "http://az619822.vo.msecnd.net/files/TuileriesGardenWheel_EN-US11916079727_1366x768.jpg",
        "http://az608707.vo.msecnd.net/files/ColorfulSalt_EN-US13586718897_1366x768.jpg",
        "http://az608707.vo.msecnd.net/files/FelgueirasLighthouse_EN-US11198579022_1366x768.jpg"
    ];

    var ins = new Casual();
    ins.init(source);
    ins.play();
}
```

## 效果演示及源码：
[轮播的原生js实现](https://codepen.io/hustchen/pen/jwgmXm)