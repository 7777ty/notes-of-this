# JS中的this

>本文主要用于对本人在学习前端过程中的知识梳理，其中包括了大量其他前端前辈们的思想精华，非本人原创。


JS中的this的指向与其他语言不同的是：**函数的调用方式决定了this的指向**。
JavaScript 中，普通的函数调用方式有三种：直接调用、方法调用和 new 调用。除此之外，还有一些特殊的调用方式，比如通过 bind() 将函数绑定到对象之后再进行调用、通过 call()、apply() 进行调用等。而 es6 引入了箭头函数之后，箭头函数调用时，其 this 指向又有所不同。下面就来分析这些情况下的 this 指向，这些特殊情况在本文中将不再提及，因为我的另一篇博客已对该情况进行了梳理与总结。

## 直接调用

直接调用时，函数内部的this指向<font color=red>全局对象</font>，即浏览器中的<font color=red>window</font>或NodeJs中的<font color=red>global</font>。

举个例子🌰：
```JS
function test() {...}

test();    // 直接调用
```
需要注意的是，直接调用并不是指在全局作用域下进行调用，在任何作用域下，直接通过 <font color=red>函数名()</font> 来对函数进行调用的方式，都称为直接调用。

## 方法调用
方法调用则是指通过对象来调用其方法函数，它是<font color=red>对象.函数名()</font> 这样的调用形式。这种情况下，除特殊调用，函数中this指向调用该方法的对象。
```JS
const obj = {
    // 第一种方式，定义对象的时候定义其方法
    test() {
        console.log(this === obj);
    }
};

// 第二种方式，对象定义好之后为其附加一个方法(函数表达式)
obj.test2 = function() {
    console.log(this === obj);
};

// 第三种方式和第二种方式原理相同
// 是对象定义好之后为其附加一个方法(函数定义)
function t() {
    console.log(this === obj);
}
obj.test3 = t;

// 这也是为对象附加一个方法函数
// 但是这个函数绑定了一个不是 obj 的其它对象
obj.test4 = (function() {
    console.log(this === obj);
}).bind({});

obj.test();     // true
obj.test2();    // true
obj.test3();    // true

// 受 bind() 影响，test4 中的 this 指向不是 obj
obj.test4();    // false
```
这里需要注意的是，后三种方式都是预定定义函数，再将其附加给 obj 对象作为其方法。再次强调，函数内部的 this 指向与定义无关，受调用方式的影响。

### 方法中 this 指向全局对象的情况
注意这里说的是**方法**中而不是**方法调用**中。方法中的 this 指向全局对象，如果不是因为 bind()，那就一定是因为不是用的方法调用方式，比如
```JS
const obj = {
    test() {
        console.log(this === obj);
    }
};

const t = obj.test;
t();    // false
```
t 就是 obj 的 test 方法，但是 t() 调用时，其中的 this 指向了全局。

之所以要特别提出这种情况，主要是因为常常将一个对象方法作为回调传递给某个函数之后，却发现运行结果与预期不符——因为忽略了调用方式对 **this** 的影响。比如下面的例子是在页面中对某些事情进行封装之后特别容易遇到的问题：

```JS
class Handlers {
    // 这里 $button 假设是一个指向某个按钮的 jQuery 对象
    constructor(data, $button) {
        this.data = data;
        $button.on("click", this.onButtonClick);
    }

    onButtonClick(e) {
        console.log(this.data);
    }
}

const handlers = new Handlers("string data", $("#someButton"));
// 对 #someButton 进行点击操作之后
// 输出 undefined
// 但预期是输出 string data
```

this.onButtonClick 作为一个参数传入 on() 之后，事件触发时，**理论上是对这个函数进行的直接调用**，而不是方法调用，所以其中的 this 会指向全局对象，**但实际上**由于**调用事件处理函数的时候，this 指向会绑定到触发事件的 DOM 元素上**，所以这里的 this 是指向触发事件的的 DOM 元素<font color=grey>(注意：this 并非 jQuery 对象)</font>，即 $button.get(0)<font color=grey>(注意代码前注释中的假设)</font>。

## new调用
**在 es6 之前，每一个函数都可以当作是构造函数，通过 new 调用来产生新的对象<font color=grey>(函数内无特定返回值的情况下)</font>**。而 es6 改变了这种状态，虽然 class 定义的类用 typeof 运算符得到的仍然是 "function"，但它不能像普通函数一样直接调用；同时，**class 中定义的方法函数，也不能当作构造函数用 new 来调用**。

而在 es5 中，**用 new 调用一个构造函数，会创建一个新对象，而其中的 this 就指向这个新对象**。这没有什么悬念，因为 new 本身就是设计来创建新对象的。例子🌰如下：

```JS
var data = "Hi";    // 全局变量

function AClass(data) {
    this.data = data;
}

var a = new AClass("Hello World");
console.log(a.data);    // Hello World
console.log(data);      // Hi

var b = new AClass("Hello World");
console.log(a === b);   // false
```

## 箭头函数中的 this
**箭头函数没有自己的 this 绑定**。箭头函数中使用的 this，其实是直接包含它的那个函数或函数表达式中的 this。比如
```JS
const obj = {
    test() {
        const arrow = () => {
            // 这里的 this 是 test() 中的 this，
            // 由 test() 的调用方式决定
            console.log(this === obj);
        };
        arrow();
    },

    getArrow() {
        return () => {
            // 这里的 this 是 getArrow() 中的 this，
            // 由 getArrow() 的调用方式决定
            console.log(this === obj);
        };
    }
};

obj.test();     // true

const arrow = obj.getArrow();
arrow();        // true
```

**示例中的两个 this 都是由箭头函数的直接外层函数(方法)决定的，而方法函数中的 this 是由其调用方式决定的**。上例的调用方式都是方法调用，所以 this 都指向方法调用的对象，即 obj。

箭头函数让大家在使用闭包的时候不需要太纠结 this，不需要通过像 _this 这样的局部变量来临时引用 this 给闭包函数使用。来看一段 Babel 对箭头函数的转译可能能加深理解：

```JS
// ES6
const obj = {
    getArrow() {
        return () => {
            console.log(this === obj);
        };
    }
}   

```

```JS
// ES5，由 Babel 转译
var obj = {
    getArrow: function getArrow() {
        var _this = this;
        return function () {
            console.log(_this === obj);
        };
    }
};
```

另外需要注意的是，**箭头函数不能用 new 调用**，**不能 bind() 到某个对象**<font color=grey>(虽然 bind() 方法调用没问题，但是不会产生预期效果)</font>。不管在什么情况下使用箭头函数，它本身是没有绑定 this 的，**它用的是直接外层函数<font color=grey>(即包含它的最近的一层函数或函数表达式)</font>绑定的 this**。