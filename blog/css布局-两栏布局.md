# 1.两栏布局
## 1.1借助float与margin

**示例**
```
<div class="container" style="height:200px;border:solid red">
    <div class="sidebar" style="float:right;width:200px;height:100%;border:solid yellow;">sidebar面板：使用float向右浮动，宽度固定</div>
    <div class="main" style="margin:0 200px 0 0;border:solid green;height:100%;">main面板：宽度可伸缩</div>
</div>
```
**效果**
<div class="container" style="height:200px;border:solid red">
    <div class="sidebar" style="float:right;width:200px;height:100%;border:solid yellow;">sidebar面板：使用float向右浮动，宽度固定</div>
    <div class="main" style="margin:0 200px 0 0;border:solid green;height:100%;">main面板：宽度可伸缩</div>
</div>

## 1.2借助float与overflow

**示例**
```
<div class="container" style="height:200px;border:solid red">
    <div class="sidebar" style="float:right;width:200px;height:100%;border:solid yellow;">sidebar面板：使用float向右浮动，宽度固定</div>
    <div class="main" style="overflow:hidden;">main面板：宽度可伸缩<br>
    overflow:hidden，触发bfc模式，浮动无法影响，隔离其他元素，IE6不支持，左侧left设置margin-left当作left与right之间的边距，右侧利用overflow:hidden 进行形成bfc模式</div>
</div>
```
**效果**
<div class="container" style="height:200px;border:solid red">
    <div class="sidebar" style="float:right;width:200px;height:100%;border:solid yellow;">sidebar面板：使用float向右浮动，宽度固定</div>
    <div class="main" style="overflow:hidden;">main面板：宽度可伸缩<br>
    overflow:hidden，触发bfc模式，浮动无法影响，隔离其他元素，IE6不支持，左侧left设置margin-left当作left与right之间的边距，右侧利用overflow:hidden 进行形成bfc模式</div>
</div>

## 1.2绝对定位
**示例**
```
<div class="container" style="position:relative;height:200px;border:solid red">
    <div class="main" style="margin:0 200px 0 0;border:solid green;height:100%;">main面板：宽度可伸缩</div>
        <div class="sidebar" style="position:absolute;top:0;right:0;width:200px;height:100%;border:solid yellow;">sidebar面板：使用float向右浮动，宽度固定</div>
</div>
```
**效果**
<div class="container" style="position:relative;height:200px;border:solid red">
    <div class="main" style="margin:0 200px 0 0;border:solid green;height:100%;">main面板：宽度可伸缩</div>
        <div class="sidebar" style="position:absolute;top:0;right:0;width:200px;height:100%;border:solid yellow;">sidebar面板：使用float向右浮动，宽度固定</div>
</div>

## 1.3表格布局
+ 分别设置百分百宽度 

**示例**

```
<div class="container" style="display:table;width:100%;height:200px;border:solid red">
    <div class="row" style="display:table-row;">
        <div class="main" style="display:table-cell;vertical-align:top;width:70%%;border:solid green;">main面板：宽度70%</div>
        <div class="sidebar" style="display:table-cell;vertical-align:top;width:30%;border:solid yellow;">sidebar面板：宽度30%</div>
    </div>
</div>
```
**效果**


<div class="container" style="display:table;width:100%;height:200px;border:solid red">
    <div class="row" style="display:table-row;">
        <div class="main" style="display:table-cell;vertical-align:top;width:70%%;border:solid green;">main面板：宽度70%</div>
        <div class="sidebar" style="display:table-cell;vertical-align:top;width:30%;border:solid yellow;">sidebar面板：宽度30%</div>
    </div>
</div>
+ 不设置宽度，根据表格内容自动调整
```
<div class="container" style="display:table;width:100%;height:200px;border:solid red">
    <div class="row" style="display:table-row;">
        <div class="main" style="display:table-cell;vertical-align:top;border:solid green;">main面板：这边内容较多，所以自动调整为较宽</div>
        <div class="sidebar" style="display:table-cell;vertical-align:top;border:solid yellow;">sidebar面板</div>
    </div>
</div>
```
<div class="container" style="display:table;width:100%;height:200px;border:solid red">
    <div class="row" style="display:table-row;">
        <div class="main" style="display:table-cell;vertical-align:top;border:solid green;">main面板：这边内容较多，所以自动调整为较宽</div>
        <div class="sidebar" style="display:table-cell;vertical-align:top;border:solid yellow;">sidebar面板</div>
    </div>
</div>
+ 设置sidebar固定宽度，main自动调整
```
<div class="container" style="display:table;width:100%;height:200px;border:solid red">
    <div class="row" style="display:table-row;">
        <div class="main" style="display:table-cell;vertical-align:top;border:solid green;">main面板：没有设置宽度，自动调整填充剩余部分</div>
        <div class="sidebar" style="display:table-cell;vertical-align:top;border:solid yellow;width:200px;">sidebar面板：固定宽度200px</div>
    </div>
</div>
```
<div class="container" style="display:table;width:100%;height:200px;border:solid red">
    <div class="row" style="display:table-row;">
        <div class="main" style="display:table-cell;vertical-align:top;border:solid green;">main面板：没有设置宽度，自动调整填充剩余部分</div>
        <div class="sidebar" style="display:table-cell;vertical-align:top;border:solid yellow;width:200px;">sidebar面板：固定宽度200px</div>
    </div>
</div>