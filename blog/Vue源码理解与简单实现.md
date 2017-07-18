# 思路整理
 已经了解到vue是通过数据劫持的方式来做数据绑定的，
其中最核心的方法便是通过Object.defineProperty()来实现对属性的劫持，
达到监听数据变动的目的，无疑这个方法是本文中最重要、最基础的内容之一，
如果不熟悉defineProperty，猛戳这里 整理了一下，要实现mvvm的双向绑定，
就必须要实现以下几点：

+ 1、实现一个数据监听器Observer，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者 
+ 2、实现一个指令解析器Compile，对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数
+ 3、实现一个Watcher，作为连接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图 
+ 4、mvvm入口函数，整合以上三者

![avatar](https://raw.githubusercontent.com/DMQ/mvvm/master/img/2.png)

# 具体实现
## 1 . Observer
    实现一个数据监听器Observer，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者

首先定义Observer的构造函数，需要将需添加的键值属性对保存，并添加对其变化的检测，如下，将原始数据作为data属性存下来；
```
// 1. 实现observer，监听对象的数据变化
function Observer(data) {
    // 存下原始属性/值对
    this.data = data;
    // 通过拦截setter和getter添加对数据变化的检测
    this.walk(data);
}
```
下面定义一些Observer必需的方法，这里主要是拦截对数据的取值、赋值，从而检测其数值的变化；代码中巧妙的运用了闭包的特性，如在set函数的定义当中，不能使用data[key]=newVal来更新数值，这会导致set函数的无限重复调用，所以这里选择直接将新值赋给value，因为value实在defineProperty函数域内定义的，所以set和get函数都可以访问该值，即在set当中重新设定了value值，那么get调用时也能获取最新的值；

在这里要说明的是**var dep = new Dep();**，Dep在后面会说到，我们暂且可以这样看待它，Dep是一个数据监测的管理者，他拥有一个数组，该数组记录了数据变化时应该给谁发送信息使其更新；

```
Observer.prototype = {
    // 监测数据变化【定义getter和setter】
    walk: function(data) {
        var me = this;
        Object.keys(data).forEach(function(key) {
            me.convert(key, data[key]);
        });
    },
    convert: function(key, val) {
        this.defineReactive(this.data, key, val);
    },

    defineReactive: function(data, key, val) {
        var dep = new Dep();
        // 对本节点添加数据变化监测
        var childObj = observe(val);

        Object.defineProperty(data, key, {
            enumerable: true, // 可枚举
            configurable: false, // 不能再define
            get: function() {
                console.log(key+':get!')
                // target存在，而且该订阅者不再订阅队列当中，则添加该订阅者
                // 否则可能会每次调用set之后触发订阅者重新调用get时添加重复的订阅者
                Dep.target && dep.subs.indexOf(Dep.target)<0 && dep.addSub(Dep.target);
                return val;
            },
            set: function(newVal) {
                console.log(key+':set '+val+'=>'+newVal);
                if (newVal === val) {
                    return;
                }
                val = newVal;
                // 新的值是object的话，进行监听（判断object逻辑在observer中）
                childObj = observe(newVal);
                // 通知订阅者
                dep.notify();
            }
        });
    }
}
// 对数据data添加数据变化监测，并返回该数据
function observe(data){
    if(!data||typeof data !=='object'){
        return;
    }
    return new Observer(data);
 }
```

## 2 . 实现Dep，管理订阅者，并发送订阅
首先，Dep构造函数当中定义了subs属性，初始化为一个空数组，并且定义了addSub方法用于向subs数组当中添加成员（Watcher实例），而且定义了方法notify，在方法中，遍历subs当中的所有Watcher实例，调用其update函数，实现所有订阅者的视图刷新；最后的Dep.target作为一个全局变量的作用存储当前订阅者，并在第一次获取关联数据时将其加入到subs当中；

```
 function Dep(){
    // 定义subs为订阅者watcher管理数组，
    this.subs= [];
 }
// 添加订阅者
 Dep.prototype.addSub = function (sub){
    this.subs.push(sub);
 }
// 发送订阅（数据变化时调用，将消息发送给所有订阅者）
 Dep.prototype.notify = function (){
    this.subs.forEach(function(sub){
        sub.update();
    })
 }
 // 这里定义一个Dep.target，每当需要添加订阅者时将订阅者的节点node赋给target，
 // 这就相当于有了一个全局变量存储了当前的订阅者，这样当第一次去获取管理数据时，
 // 就可以在get函数中获取Dep.target,并将其加入到subs当中
Dep.target = null;
```

## 3. 实现watcher（订阅者）
首先，同样是构造函数，这里需要的是归属对象（vm），订阅的数据属性名称（exp）和响应函数（cb），即发生变化是需要调用的函数；最重要的一点是最后一句：**this.value = this.get(); **这里显示的调用get方法，从而实现了咋构造watcher实例的同时第一次进入到了get函数，从而对应上Observer的get函数当中的**Dep.target && dep.subs.indexOf(Dep.target)<0 && dep.addSub(Dep.target);**，即实现了当前订阅者加入到Dep.subs当中；


```
 //     vm:当前MVVM对象
 //     exp:属性名；
 //     cb:更新视图函数
function Watcher(vm, exp, cb) {
    this.cb = cb;
    this.vm = vm;    //  MVVM本身
    this.exp = exp;  //  属性名
    // 此处为了触发属性的getter，从而在dep添加自己，结合Observer更易理解
    this.value = this.get(); 
}
```
其次，定义Watcher的必需方法，首先是update，之前在Observer的set函数当中调用了Dep.notify函数，其中就对Dep.subs当中的所有Watcher调用了update函数，所以Watcher.update函数需要更新绑定视图当中的值；由于不同的directiv指令需要有不同的更新函数，所以这里不做统一的处理，而使用传入处理函数cb的方式灵活处理；之前也说了，在构建函数的最后一步是调用get方法，为的是将Watcher通过Dep.tatget添加到订阅者队列当中，因此这里get函数需要定义，并且在其中将this赋值给Dep.target；并在触发Observer.getter之后将其置为null；
```
Watcher.prototype = {
    update: function() {
        this.run(); // 属性值变化收到通知
    },
    // 更新视图
    run: function() {
        var value = this.get(); // 取到最新值
        var oldVal = this.value; // 旧值
        if (value !== oldVal) {
            this.value = value;
            this.cb.call(this.vm, value, oldVal); // 执行Compile中绑定的回调，更新视图
        }
    },
    // 获取值
    get: function() {
        Dep.target = this;  // 将当前订阅者指向自己
        var value = this.vm[this.exp];  // 触发getter，添加自己到属性订阅器中
        Dep.target = null;  // 添加完毕，重置
        return value;
    }
};
```

## 4. 实现complier，编译得到fragment，并添加订阅
compile主要做的事情是解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，更新视图，如图所示：
![avatar](https://raw.githubusercontent.com/DMQ/mvvm/master/img/3.png)

这里参考[https://github.com/DMQ/mvvm](https://github.com/DMQ/mvvm),其这里只对其源码进行了理解并添加了自己的理解（注释）
```
function Compile(el,vm) {
    this.$vm = vm;
    // 获取关联组件（dom元素）
    this.$el = this.isElementNode(el) ? el : document.querySelector(el);
    if (this.$el) {
        // 获取关联组件的fragment
        this.$fragment = this.node2Fragment(this.$el);
        // 编译该fragment
        this.init();
        // 将编译后的fragment添加到指定关联组件当中
        this.$el.appendChild(this.$fragment);
    }
}
Compile.prototype = {
    init: function() { this.compileElement(this.$fragment); },

    // 生成关联节点的fragment
    node2Fragment: function(el) {
        var fragment = document.createDocumentFragment(), child;
        // 将原生节点拷贝到fragment
        while (child = el.firstChild) {
            fragment.appendChild(child);
        }
        return fragment;
    },

    // 编译节点
    compileElement: function(el) {
        var childNodes = el.childNodes, me = this;
        [].slice.call(childNodes).forEach(function(node) {
            var text = node.textContent;
            var reg = /\{\{(.*)\}\}/;   // 表达式文本
            // 按元素节点方式编译
            if (me.isElementNode(node)) {
                // 若是dom元素节点，则使用compile编译该节点
                me.compile(node);
            } else if (me.isTextNode(node) && reg.test(text)) {
                // 若是文本节点，而且当中有插值表达式，编译该文字节点
                me.compileText(node, RegExp.$1);
            }
            // 若为dom元素节点，且有子节点，遍历编译子节点
            if (node.childNodes && node.childNodes.length) {
                me.compileElement(node);
            }
        });
    },
    // 编译DOM元素节点（nodeType=1）
    compile: function(node) {
        var nodeAttrs = node.attributes, me = this;
        [].slice.call(nodeAttrs).forEach(function(attr) {
            // 规定：指令以 v-xxx 命名
            // 如 <span v-text="content"></span> 中指令为 v-text
            var attrName = attr.name;   // v-text
            // 当前节点属性为需编译指令
            if (me.isDirective(attrName)) {
                var exp = attr.value; // content
                var dir = attrName.substring(2);    // text
                if (me.isEventDirective(dir)) {
                    // 事件指令, 如 v-on:click
                    compileUtil.eventHandler(node, me.$vm, exp, dir);
                } else {
                    // 普通指令
                    compileUtil[dir] && compileUtil[dir](node, me.$vm, exp);
                }
            }
        });
    },

    // 编译文本节点
    compileText: function(node, exp) {
        compileUtil.text(node, this.$vm, exp);
    },

    // 判断给定属性是否为需编译指令属性
    isDirective: function(attr) {
        return attr.indexOf('v-') == 0;
    },

    // 判断给定属性是否为事件绑定属性
    isEventDirective: function(dir) {
        return dir.indexOf('on') === 0;
    },

    // 判断给定节点是否为dom元素节点
    isElementNode: function(node) {
        return node.nodeType == 1;
    },

    // 判断给定节点是否为文本节点
    isTextNode: function(node) {
        return node.nodeType == 3;
    }
};

// 指令处理集合
// 指令处理集合
var compileUtil = {
    // v-text绑定方法：
    //      node：文本节点；
    //      vm: MVVM对象本身；
    //      exp:v-text的表达式;
    text: function(node, vm, exp) {
        this.bind(node, vm, exp, 'text');
    },
    // v-model绑定方法：
    //      node：文本节点；
    //      vm: MVVM对象本身；
    //      exp:v-model的表达式;
    model: function(node, vm, exp) {
        this.bind(node, vm, exp, 'model');

        var me = this,
            val = this._getVMVal(vm, exp);
        node.addEventListener('input', function(e) {
            var newValue = e.target.value;
            if (val === newValue) {
                return;
            }

            me._setVMVal(vm, exp, newValue);
            val = newValue;
        });
    },

    // v-bind绑定方法：
    //      node：文本节点；
    //      vm: MVVM对象本身；
    //      exp:v-bind的表达式;
    bind: function(node, vm, exp, dir) {
        var updaterFn = updater[dir + 'Updater'];

        updaterFn && updaterFn(node, this._getVMVal(vm, exp));

        new Watcher(vm, exp, function(value, oldValue) {
            updaterFn && updaterFn(node, value, oldValue);
        });
    },

    // 事件处理
    //      node：文本节点；
    //      vm: MVVM对象本身；
    //      exp:v-bind的表达式;
    //      dir:绑定事件名称
    eventHandler: function(node, vm, exp, dir) {
        var eventType = dir.split(':')[1],
            fn = vm.$options.methods && vm.$options.methods[exp];

        if (eventType && fn) {
            node.addEventListener(eventType, fn.bind(vm), false);
        }
    },

    // 获取MVVM实例的exp属性值
    _getVMVal: function(vm, exp) {
        var val = vm;
        exp = exp.split('.');
        exp.forEach(function(k) {
            val = val[k];
        });
        return val;
    },

    // 设置MVVM实例的exp属性值
    _setVMVal: function(vm, exp, value) {
        var val = vm;
        exp = exp.split('.');
        exp.forEach(function(k, i) {
            // 非最后一个key，更新val的值
            if (i < exp.length - 1) {
                val = val[k];
            } else {
                val[k] = value;
            }
        });
    }
};

// 更新视图函数
var updater = {
    // 更新通过v-text绑定数据的节点node的视图
    textUpdater: function(node, value) {
        node.textContent = typeof value == 'undefined' ? '' : value;
    },
    // 更新通过v-绑定数据的节点node的视图
    htmlUpdater: function(node, value) {
        node.innerHTML = typeof value == 'undefined' ? '' : value;
    },
    // 更新通过v-html绑定数据的节点node的视图
    classUpdater: function(node, value, oldValue) {
        var className = node.className;
        className = className.replace(oldValue, '').replace(/\s$/, '');

        var space = className && String(value) ? ' ' : '';

        node.className = className + space + value;
    },
    // 更新通过v-model绑定数据的节点node的视图
    modelUpdater: function(node, value, oldValue) {
        node.value = typeof value == 'undefined' ? '' : value;
    }
};
```

## 5. 提供公共接口，将以上对象连接起来
```
function MVVM(options) {
    this.$options = options;
    // 获取对象待双向绑定的数据data
    var data = this._data = this.$options.data, me = this;
    // 属性代理，实现 vm.xxx -> vm._data.xxx
    Object.keys(data).forEach(function(key) {
        me._proxy(key);
    });
    // 设定数据变化检测
    observe(data, this);
    // 编译，添加订阅
    this.$compile = new Compile(options.el || document.body, this)
}

// 设定代理，使得vm.xxx -> vm._data.xxx
MVVM.prototype = {
    _proxy: function(key) {
        var me = this;
        Object.defineProperty(me, key, {
            configurable: false,
            enumerable: true,
            get: function proxyGetter() {
                return me._data[key];
            },
            set: function proxySetter(newVal) {
                me._data[key] = newVal;
            }
        });
    }
};
```

## 测试
js
```
window.onload = function (){
         vm = new MVVM({
            // el: '#main',
            data: {
                name: 'hello ',
                age:23
            },
            methods:{
                functest:function(){
                    alert('test');
                }
            }
        });
}
```
html

```
        <div id="main">
            <h1>{{name}}</h1>
            <input type="text" v-model="name"><br>
            <h1 v-text="age"></h1>
            <input type="text" v-model="age"><br>
            <input type="button" v-on:click="functest">
        </div>
```

### 参考
[https://github.com/DMQ/mvvm](https://github.com/DMQ/mvvm)

[https://segmentfault.com/a/1190000006866881](https://segmentfault.com/a/1190000006866881)

[https://cn.vuejs.org/v2/guide/](https://cn.vuejs.org/v2/guide/)

[https://github.com/fwing1987/MyVue](https://github.com/fwing1987/MyVue)

###　源码：
[codepen](https://codepen.io/hustchen/pen/EXrLwO?editors=1111)