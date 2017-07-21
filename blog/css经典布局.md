<style>
    div{
        padding: 0;
        margin: 0;
        /*overflow: hidden;*/
        box-sizing: border-box;
    }
    .containe{
        border: 3px solid gold;
        height:300px;
    }
    .left{
        height: 100%;
        border:  2px solid green;
    }
    .right{
        height: 100%;
        border: 2px solid red;
    }
    .col{
        height: 100%;
        border:2px solid blue;
    }
    .row{
        border:2px solid orange;
    }
    .top,.bottom{
        border:2px solid cyan;
    }
</style>

# 一.两栏布局
 该布局方式非常常见，适用于定宽的一侧常为导航，自适应的一侧为内容的布局
## 1.左列定宽，右列自适应
### 1>实现一：float+margin

**示例**
```
<div class="containe">
    <div class="left" style="float:left;width:200px;">这里是左侧导航栏区域，固定宽度为200px；</div>
    <div class="right" style="margin-left:200px;">这里是可伸缩区域</div>
</div>
```
**效果**
<div class="containe">
    <div class="left" style="float:left;width:200px;">这里是左侧导航栏区域，固定宽度为200px；</div>
    <div class="right" style="margin-left:200px;">这里是可伸缩区域</div>
</div>
### 2>实现二：float+BFC

overflow:hidden，触发bfc模式，浮动无法影响，隔离其他元素，IE6不支持，左侧left设置margin-left当作left与right之间的边距，右侧利用overflow:hidden 进行形成bfc模式.

**示例**
```
<div class="containe">
    <div class="left" style="width:200px;float:left;"">左侧定宽200；而且左浮动</div>
    <div class="right" style="overflow:hidden">这里是自动调节区域，并使用overflow:hidden从而触发了BFC</div>
</div>
```
**效果**
<div class="containe">
    <div class="left" style="width:200px;float:left;"">左侧定宽200；而且左浮动</div>
    <div class="right" style="overflow:hidden">这里是自动调节区域，并使用overflow:hidden从而触发了BFC</div>
</div>

**这里若不触发BFC，那么，.right将与父对象同宽（100%），尽管其中的内容不会进入到左侧浮动区域，但是当左侧导航栏较短时可能出现下面的情况(如下左)；若右侧形成了BFC就不会出现这样的问题了（如下右）**
<div style="display:flex;border:0;justify-content:space-around;">
  <div class="containe" style="width:40%">
    <div class="left" style="width:200px;height:30px;float:left;"">左侧定宽200；而且左浮动</div>
    <div class="right">这里是自动调节区域，没有使用overflow:hidden。BFCffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff</div>
    </div> 
    <div class="containe" style="width:40%">
    <div class="left" style="width:200px;height:30px;float:left;"">左侧定宽200；而且左浮动</div>
    <div class="right" style="overflow:hidden">这里是自动调节区域，没有使用overflow:hidden。BFCffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff</div>
</div> 
</div>




### 3>实现三：table
**示例**
```
<div class="containe" style="display:table;table-layout:fixed;width:100%">
    <div class="left" style="width:200px;display:table-cell;"></div>
    <div class="right" style="display:table-cell;"></div>
</div>
```
**效果**
<div class="containe" style="display:table;table-layout:fixed;width:100%">
    <div class="left" style="width:200px;display:table-cell;"></div>
    <div class="right" style="display:table-cell;"></div>
</div>
### 4>实现四：flex布局
利用右侧容器的flex:1，均分了剩余的宽度，也实现了同样的效果。而align-items 默认值为stretch，故二者高度相等
**示例**
```
<div class="containe" style="display:flex;">
    <div class="left" style="width:200px;"></div>
    <div class="right" style="flex:1"></div>
</div>
```
**效果**
<div class="containe" style="display:flex;">
    <div class="left" style="width:200px;"></div>
    <div class="right" style="flex:1"></div>
</div>
## 2.右列定宽，左列自适应
### 1>float+margin
**示例**
```
<div class="containe" style="">
    <div class="right" style="float:right;width:200px;"></div>
    <div class="left" style="margin-right:200px;"></div>
</div>
```
**效果**
<div class="containe" style="">
    <div class="right" style="float:right;width:200px;"></div>
    <div class="left" style="margin-right:200px;"></div>
</div>
### 2>table
**示例**
```
<div class="containe" style="display:table;width:100%">
    <div class="left" style="display:table-cell;flex:1"></div>
    <div class="right" style="display:table-cell;width:200px;"></div>
</div>
```
**效果**
<div class="containe" style="display:table;width:100%">
    <div class="left" style="display:table-cell;"></div>
    <div class="right" style="display:table-cell;width:200px;"></div>
</div>
### 3>flex
**示例**
```
<div class="containe" style="display:flex;">
    <div class="left" style="flex:1"></div>
    <div class="right" style="width:200px;"></div>
</div>
```
**效果**
<div class="containe" style="display:flex;">
    <div class="left" style="flex:1"></div>
    <div class="right" style="width:200px;"></div>
</div>
## 3.一列不定宽，一列自适应
### 1> flex
**示例**
```
<div class="containe" style="display:flex;">
    <div class="left" style="flex:1"></div> 
    <div class="right"></div>
</div>
```
**效果**
<div class="containe" style="display:flex;">
    <div class="left" style="flex:1">这一列填充右侧剩下的空间</div> 
    <div class="right">这一列根据自身内容调节自身的宽度</div>
</div>
### 2> float+overflow
**示例**
```
<div class="containe">
    <div class="left" style="float:left">这一列自动分局自身的内容调节宽度</div>
    <div class="right" style="overflow:hidden;">这一列自动填充左侧剩下的空间</div>
</div>
```
**效果**
<div class="containe">
    <div class="left" style="float:left">这一列自动分局自身的内容调节宽度</div>
    <div class="right" style="overflow:hidden;">这一列自动填充左侧剩下的空间</div>
</div>

### 3> table
**示例**
```
<div class="containe" style="display:table;width:100%;">
    <div class="col" style="width:200px;display:table-cell;"></div>
    <div class="col" style="width:200px;display:table-cell;"></div>
    <div class="right" style="display:table-cell;"></div>
</div>
```
**效果**
<div class="containe" style="display:table;width:100%;">
    <div class="left" style="display:table-cell;width:0.1%">这一列自动扩展，根据自身的内容调节自身的div宽度</div>
    <div class="right" style="display:table-cell;">这一列自动填充剩下的空间</div>
</div>
# 二.多栏布局
## 1.两列定宽，一列自适应
### 1> flex
**示例**
```
<div class="containe" style="display:flex;">
    <div class="col" style="width:200px;"></div>   
    <div class="col" style="width:200px;"></div>
    <div class="right" style="flex:1"></div>
</div>
```
**效果**
<div class="containe" style="display:flex;">
    <div class="col" style="width:200px;"></div>   
    <div class="col" style="width:200px;"></div>
    <div class="right" style="flex:1"></div>
</div>
### 2> float+overflow
**示例**
```
<div class="containe">
    <div class="col" style="width:200px;float:left;"></div>
    <div class="col" style="width:200px;float:left;"></div>
    <div class="right" style="overflow:hidden;"></div>
</div>
```
**效果**
<div class="containe">
    <div class="col" style="width:200px;float:left;"></div>
    <div class="col" style="width:200px;float:left;"></div>
    <div class="right" style="overflow:hidden;"></div>
</div>
### 3> float+margin
**示例**
```
<div class="containe">
    <div class="col" style="width:200px;float:left;"></div>
    <div class="col" style="width:200px;float:left;"></div>
    <div class="right" style="margin-left:400px;"></div>
</div>
```
**效果**
<div class="containe">
    <div class="col" style="width:200px;float:left;"></div>
    <div class="col" style="width:200px;float:left;"></div>
    <div class="right" style="margin-left:400px;"></div>
</div>

### 4> table
**示例**
```
<div class="containe" style="display:table;width:100%;">
    <div class="col" style="width:200px;display:table-cell;"></div>
    <div class="col" style="width:200px;display:table-cell;"></div>
    <div class="right" style="display:table-cell;"></div>
</div>
```
**效果**
<div class="containe" style="display:table;width:100%;">
    <div class="col" style="width:200px;display:table-cell;"></div>
    <div class="col" style="width:200px;display:table-cell;"></div>
    <div class="right" style="display:table-cell;"></div>
</div>
## 2.双飞翼布局（两侧定宽，中栏自适应）
### 1> flex
**示例**
```
<div class="containe" style="display:flex;">
    <div class="left" style="width:200px;"></div>     
    <div class="col" style="flex:1"></div> 
    <div class="right" style="width:200px;"></div>
</div>
```
**效果**
<div class="containe" style="display:flex;">
    <div class="left" style="width:200px;"></div>     
    <div class="col" style="flex:1"></div> 
    <div class="right" style="width:200px;"></div>
</div>
### 2> float+overflow
**示例**
```
<div class="containe">
    <div class="col" style="width:200px;float:left;"></div>
    <div class="col" style="width:200px;float:right;"></div>
    <div class="right" style="overflow:hidden;"></div>
</div>
```
**效果**
<div class="containe">
    <div class="col" style="width:200px;float:left;"></div>
    <div class="col" style="width:200px;float:right;"></div>
    <div class="right" style="overflow:hidden;"></div>
</div>
### 3> float+margin
**示例**
```
<div class="containe">
    <div class="col" style="width:200px;float:left;"></div>
    <div class="col" style="width:200px;float:right;"></div>
    <div class="right" style="margin:0 200px 0 200px;"></div>
</div>
```
**效果**
<div class="containe">
    <div class="col" style="width:200px;float:left;"></div>
    <div class="col" style="width:200px;float:right;"></div>
    <div class="right" style="margin:0 200px 0 200px;"></div>
</div>

### 4> table
**示例**
```
<div class="containe" style="display:table;width:100%;">
    <div class="col" style="width:200px;display:table-cell;"></div> 
    <div class="right" style="display:table-cell;"></div>
    <div class="col" style="width:200px;display:table-cell;"></div>
</div>
```
**效果**
<div class="containe" style="display:table;width:100%;">
    <div class="col" style="width:200px;display:table-cell;"></div> 
    <div class="right" style="display:table-cell;"></div>
    <div class="col" style="width:200px;display:table-cell;"></div>
</div>
## 3.多列等分
### 1>float
**示例**
```
```
**效果**
<div class="containe">
    <div class="col" style="float:left;width:25%;padding:20px;"></div>
    <div class="col" style="float:left;width:25%;padding:20px;""></div>
    <div class="col" style="float:left;width:25%;padding:20px;""></div>
    <div class="col" style="float:left;width:25%;padding:20px;""></div>
</div>
### 2>table
**示例**
```
<div class="containe" style="display:table;width:100%;">
    <div class="col" style="display:table-cell;"></div>
    <div class="col" style="display:table-cell;"></div>
    <div class="col" style="display:table-cell;"></div>
    <div class="col" style="display:table-cell;"></div>
</div>
```
**效果**
<div class="containe" style="display:table;width:100%;">
    <div class="col" style="display:table-cell;"></div>
    <div class="col" style="display:table-cell;"></div>
    <div class="col" style="display:table-cell;"></div>
    <div class="col" style="display:table-cell;"></div>
</div>
### 3>flex
**示例**
```
<div class="containe" style="display:flex">
    <div class="col" style="flex:1;"></div>
    <div class="col" style="flex:1;"></div>
    <div class="col" style="flex:1;"></div>
    <div class="col" style="flex:1;"></div>
</div>
```
**效果**
<div class="containe" style="display:flex">
    <div class="col" style="flex:1;"></div>
    <div class="col" style="flex:1;"></div>
    <div class="col" style="flex:1;"></div>
    <div class="col" style="flex:1;"></div>
</div>
# 三.九宫格布局
### 1>table
**示例**
```
<div class="containe" style="display:table;width:100%;">
    <div class="row" style="display:table-row;">
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
    </div>
    <div class="row" style="display:table-row;">
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
    </div>
    <div class="row" style="display:table-row;">
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
    </div>
</div>
```
**效果**
<div class="containe" style="display:table;width:100%;">
    <div class="row" style="display:table-row;">
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
    </div>
    <div class="row" style="display:table-row;">
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
    </div>
    <div class="row" style="display:table-row;">
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
        <div class="col" style="display:table-cell;height:100px;"></div>
    </div>
</div>

### 2>flex

**示例**
```
<div class="containe" style="display:flex;flex-direction:column;">
    <div class="row" style="height:100px;display:flex;">
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
    </div>
    <div class="row" style="height:100px;display:flex;">
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
    </div>
    <div class="row" style="height:100px;display:flex;">
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
    </div>
</div>
```
**效果**
<div class="containe" style="display:flex;flex-direction:column;">
    <div class="row" style="height:100px;display:flex;">
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
    </div>
    <div class="row" style="height:100px;display:flex;">
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
    </div>
    <div class="row" style="height:100px;display:flex;">
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
        <div class="col" style="flex:1"></div>
    </div>
</div>

# 四.全屏布局
### １>绝对定位
**示例**
```
<div class="containe" style="position:relative;">
    <div class="top" style="position:absolute;top:0;left:0;height:50px;width:100%;">top:宽度为100%；高度固定为50px</div>
    <div class="left" style="position:absolute;top:50px;left:0;bottom:50px;height:auto;width:100px;">左侧为固定宽度50px</div>
    <div class="right" style="position:absolute;top:50px;bottom:50px;left:100px;right:0;overflow:auto;height:auto;">右侧为自适应区域</div>
    <div class="bottom" style="position:absolute;bottom:0;left:0;height:50px;width:100%">底部为固定高度50</div>
</div>
```
**效果**
<div class="containe" style="position:relative;">
    <div class="top" style="position:absolute;top:0;left:0;height:50px;width:100%;">top:宽度为100%；高度固定为50px</div>
    <div class="left" style="position:absolute;top:50px;left:0;bottom:50px;height:auto;width:100px;">左侧为固定宽度50px</div>
    <div class="right" style="position:absolute;top:50px;bottom:50px;left:100px;right:0;overflow:auto;height:auto;">右侧为自适应区域</div>
    <div class="bottom" style="position:absolute;bottom:0;left:0;height:50px;width:100%">底部为固定高度50</div>
</div>

### 2>flex
**示例**
```
<div class="containe" style="display:flex;flex-direction:column;">
    <div class="top" style="height:50px;">top:宽度为100%；高度固定为50px</div>
    <div class="middle" style="flex:1;display:flex;">
        <div class="left" style="width:200px;height:auto;"></div>
        <div class="right" style="flex:1;height:auto;"></div>
    </div>
    <div class="bottom" style="height:50px;">底部为固定高度50</div>
</div>
```
**效果**
<div class="containe" style="display:flex;flex-direction:column;">
    <div class="top" style="height:50px;">top:宽度为100%；高度固定为50px</div>
    <div class="middle" style="flex:1;display:flex;">
        <div class="left" style="width:200px;height:auto;"></div>
        <div class="right" style="flex:1;height:auto;"></div>
    </div>
    <div class="bottom" style="height:50px;">底部为固定高度50</div>
</div>
# 总结：
 根据上面的各种各样的css样式布局，可以看出来，其实不管是怎样布局，怎么都是下面几种思路：

 + 绝对布局
 + float+margin
 + float+BFC
 + table
 + flex

## 绝对布局

其中，**绝对布局**主要是针对两种，一类是对于整个页面视窗的布局，如悬浮的导航区和回到顶部等功能按钮，这是就是借助于position:fixed；另一种是相对于父容器内的绝对布局，即position:absolute，这里需要注意的是，其布局参考对象应该是包含该元素的第一个position不为static的祖先元素，因此，我们在使用的时候一般指定父对象的position为relative；

##　float+margin

**float+margin**的思路比较简单，一般用于左右的布局，而且其中一方有固定宽度，而另一方则设定该方向的margin为固定宽度值。

## float+BFC

**float+BFC**其实主要是利用了BFC的形成条件和其特性，这里简要说一下（个人理解），首先是形成条件：
以下满足其一即可：

+ float不为none；
+ overflow不为visible；
+ 绝对定位（absolute和fixed）
+ display为inline-block、table-caption或table-cell；

此外，flex似乎会破坏BFC的形成，这点尚需验证；

那么BFC有什么特性呢？
BFC中的元素布局不受外界影响，在同一个BFC中两个毗邻的块级盒在垂直方向的margin会重叠，每个盒子的左外边框紧挨包含块的左边框；换一种说法：

在一个BFC中，垂直方向上，盒子是从包含块顶部开始一个挨着一个布局的，两个相邻的盒子的垂直距离是由margin属性决定的，在一个BFC中的两个相邻的块级盒子的垂直外边距会产生塌陷。
在一个BFC中，水平方向上，每个盒子的左边缘都会接触包含块的左边缘（从右向左的格式则相反）。除非出现浮动元素和其他元素相互作用的情况（当有浮动元素时，行盒可能因浮动元素而收缩，如果有盒子形成了新的BFC，那这个盒子也可能因浮动元素而变窄）。

如果想要更加深入的研究可以参考[http://www.jianshu.com/p/8858828987be](http://www.jianshu.com/p/8858828987be)

## table与flex布局
 
 这两者布局主要是利用了响应布局的一些默认属性，使用起来非常方便，具体可以参看资料

 + [http://www.runoob.com/w3cnote/flex-grammar.html](http://www.runoob.com/w3cnote/flex-grammar.html)
 + [css Table布局-display:table](http://www.cnblogs.com/guoyong-feng/p/6076058.html)


# 注：
为了效果明显，文中使用的css类如下：
```
    div{
        padding: 0;
        margin: 0;
        /*overflow: hidden;*/
        box-sizing: border-box;
    }
    .containe{
        border: 3px solid gold;
        height:300px;
    }
    .left{
        height: 100%;
        border:  2px solid green;
    }
    .right{
        height: 100%;
        border: 2px solid red;
    }
    .col{
        height: 100%;
        border:2px solid blue;
    }
    .row{
        border:2px solid orange;
    }
    .top,.bottom{
        border:2px solid cyan;
    }
```




