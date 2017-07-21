# 1.水平居中
## 1.1 使用margin

**给需要水平居中的div指定其宽度（这是必须的），然后设置其margin：0 auto；**

**示例**
```
<div style="">parent
    <div style="width:200px;height:50px；margin:0 auto;"></div>
</div>
```
**效果**

<div style="border:solid 2px red;">parent
    <div style="width:200px;height:50px;border:solid;margin:0 auto;">child</div>
</div>
**当然，我们也可以使用max-width来设定宽度**
```
<div style="border:solid 2px red;">parent
    <div style="max-width:600px;height:50px;border:solid;margin:0 auto;">child</div>
</div>
```
<div style="border:solid 2px red;">parent
    <div style="max-width:600px;height:50px;border:solid;margin:0 auto;">child</div>
</div>

**若希望不设定居中div的宽度，那么可以接祖与table来实现**
**示例**
```
<div style="border:solid 2px red;">
    <div style="margin:0 auto;display:table;border:solid 2px gold;">   child   </div>
</div>
```
**效果**

<div style="border:solid 2px red;">
    <div style="margin:0 auto;display:table;border:solid 2px gold;">   child   </div>
</div>

## 1.2 绝对定位

**由于absolute布局的定位相对于包含他的元素（第一个非static布局祖先元素）的指定坐标，所以设定包含该元素的父对象的position：relative；该元素自身设定position：absolute；left：50%；transform：transla（-50%）；**

**示例**
```
<div style="position:relative">parent
    <div style="position:absolute;left:50%;transform:translate(-50%)"></div>
</div>
```
**效果**

<div style="border:solid 2px red;position:relative;">parent
    <div style="border:solid;position:absolute;left:50%;transform:translate(-50%)">child</div>
</div>


## 1.3 flex布局

### 1.3.1
第一种实现方法：设定包含容器的样式为display：flex；justify-content：center；这里的justify-content用于设置或检索弹性盒子元素在主轴（横轴）方向上的对齐方式。取值如下：

值|描述
:----|:-----
flex-start|默认值。项目位于容器的开头。
flex-end|项目位于容器的结尾。
center|项目位于容器的中心。
space-between|项目位于各行之间留有空白的容器内。
space-around|项目位于各行之前、之间、之后都留有空白的容器内。
initial|设置该属性为它的默认值。
inherit |从父元素继承该属性。


**示例1**
```
<div style="border:solid 2px red;display:flex;justify-content:center">
    <div style="border:solid;">child</div>
</div>
```
**效果1**

<div style="border:solid 2px red;display:flex;justify-content:center">
    <div style="border:solid;">child</div>
</div>

### 1.3.2
第二种实现方法：设定包含容器的样式为display：flex；该元素自身只需要设定margin：0 auto，相比只是用margin来进行水平居中而言，有点在与不用设定元素的宽度
**示例2**
```
<div style="border:solid 2px red;display:flex;">
    <div style="border:solid;margin:0 auto">child</div>
</div>
```
**效果2**

<div style="border:solid 2px red;display:flex;">
    <div style="border:solid;margin:0 auto">child</div>
</div>
## 1.4使用inline-block和text-align

**示例**
```
<div style="text-align:center;border:solid 2px red;">
    <div style="display: inline-block;border:solid;margin:0 auto;">child</div>
</div>
```

**效果**
<div style="text-align:center;border:solid 2px red;">
    <div style="display: inline-block;border:solid;margin:0 auto;">child</div>
</div>
**后续将补充完善**

# 2.垂直居中

##　2.1vertical-align
首先我们需要弄清楚vertical-align的使用，这个属性是**inline-block依赖型元素**，也就是说,只有一个元素属于inline或是inline-block（table-cell也可以理解为inline-block水平）水平，其身上的vertical-align属性才会起作用。

因此，可以使用一下方法来实现垂直居中：

**示例1**
```
<div class="parent" style="border:solid 2px gold;height:300px;width:300px;display:table-cell;vertical-align:middle;"</div>
    <div class="son" style="border:solid 1px red;">child</div>
</div>
```

**效果1**
<div class="parent" style="border:solid 2px gold;height:300px;width:300px;display:table-cell;vertical-align:middle;"</div>
    <div class="son" style="border:solid 1px red;">child</div>
</div>

## 2.2绝对定位
借助于absolute相对于其父对象（父对象position为非static）定位来实现居中；

**示例1**
```
<div class="parent" style="border:solid 2px gold;position:relative;height:200px;"</div>
    <div class="son" style="border:solid 1px red;position:absolute;top:50%;transform:translate(0,-50%);">child</div>
</div>
```

**效果1**
<div class="parent" style="border:solid 2px gold;position:relative;height:200px;"</div>
    <div class="son" style="border:solid 1px red;position:absolute;top:50%;transform:translate(0,-50%);">child</div>
</div>


## 2.3借助flex布局

**示例1**
```
<div class="parent" style="border:solid 2px gold;position:relative;height:200px;"</div>
    <div class="son" style="border:solid 1px red;position:absolute;top:50%;transform:translate(0,-50%);">child</div>
</div>
```

**效果1**
<div class="parent" style="border:solid 2px gold;height:200px;display:flex;align-items:center"</div>
    <div class="son" style="border:solid 1px red;">child</div>
</div>

