# React-为什么要加super()??????

> 首先要搞懂es6 的class constructor：

## Es6

父类：

```js
class Father{
    constructor(){
        this.name='father_name';
        this.age='28'
        console.log('fa_constructor')
    }
}
```

子类：

```js
class son extends Father{
    constructor(){
        console.log('son_constructor')
        super();
        console.log('son_constructor_super')
        this.name='son_name'
    }
    who(){
        console.log('123123')
        // console.log(this.name)
        return this.name
    }
}
console.log(new son().who())
```

执行结果：

>son_constructor
>
>>fa_constructor
>>
>>>son_constructor_super
>>>
>>>> 123123
>>>>
>>>> > son_name

**可以看出：执行顺序**

```flow
st=>start: 子类的constructor触发
op=>operation:  super()触发父类的constructor
op1=>operation:  父类的constructor执行完成
op2=>operation:  who()
sub1=>subroutine: son_constructor
sub2=>subroutine: fa_constructor
sub3=>subroutine: son_constructor_super
sub4=>subroutine: 123123
sub5=>subroutine: son_name
e=>end
st(left)->sub1->op(right)->sub2->op1(left)->sub3->op2(right)->sub4->sub5(right)->e->
```

![liucheng1](https://github.com/MyPrototypeWhat/take-down/blob/master/liucheng1.png)

注意：**

**你是无法在父类的constructor调用之前在constructor中使用this的，JavaScript不允许你这样做，**

```js
class son extends Father{
    constructor(){
        console.log(this)// 报错
        super();// 相当于执行了父类的constructor，并把父类的属性继承过来 
        console.log(this)// son { name: 'father_name', age: '28' }，this指向是子类 但是属性是由父类继承过来的
        this.name='son_name' // 相当于重写了{ name: 'father_name', age: '28' }中的name
    }
}
```



> JavaScript 强制要求， 如果想在constructor里使用this，就必须先调用super

## 回归正题为什么super里面要传props？？？

> 是为了在React.Component的constructor里初始化this.props：

```js
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  // We initialize the default updater but the real one gets injected by the
  // renderer.
  this.updater = updater || ReactNoopUpdateQueue;
}
```

**但在一些时候，即使在调用`super`时不传`props`参数，你仍然可以在`render`和其他方法中获取到`this.props`**

这是如何实现的呢？

<u>原因是，在你的组件实例化后，会赋值props属性给实例对象。</u>

`const instance = new YourComponent(props); instance.props = props; `

所以即使忘记传`props`给`super`，`React`仍然会在之后设置它们，这是有原因的。

当React添加对类的支持时，它不仅仅增加了对ES6的支持，目标是尽可能支持广泛的类抽象化。当时我们还[不清楚](https://link.juejin.im?target=https%3A%2F%2Freactjs.org%2Fblog%2F2015%2F01%2F27%2Freact-v0.13.0-beta-1.html%23other-languages)如ClojureScript, CoffeeScript, ES6, Fable, Scala.js, TypeScript或其他解决方案怎样算成功地定义组件，所以React也就不关心是否需要调用super()了——即便是ES6。

所以说是可以只用`super()`来替代`super(props)`吗？

最好不要。因为这样仍然有问题。没错，React可以在你的`constructor`运行后给`this.props`赋值。但`this.props`在调用`super`和`constructor`结束前仍然是`undefined`：

```jsx
// Inside React
class Component {
  constructor(props) {
    this.props = props;
    // ...
  }
}

// Inside your code
class Button extends React.Component {
  constructor(props) {
    super(); // 😬 We forgot to pass props
    console.log(props);      // ✅ {}
    console.log(this.props); // 😬 undefined 
 }
  // ...
}
```

所以若在`constructor`中使用`this.props`,就必须`super(props)`

---

## super的用法

- 当做函数使用

```js
class A {}
class B extends A {
  constructor() {
    super();  // ES6 要求，子类的构造函数必须执行一次 super 函数，否则会报错。
  }
}
```

**注：在 constructor 中必须调用 super 方法，因为子类没有自己的 this 对象，而是继承父类的 this 对象，然后对其进行加工,而 super 就代表了父类的构造函数。**

**super 虽然代表了父类 A 的构造函数，但是返回的是子类 B 的实例，即 super 内部的 this 指的是 B**，

因此 `super()` 在这里相当于 ```A.prototype.constructor.call(this, props)``。

```js
class A {
  constructor() {
    console.log(new.target.name); // new.target 指向当前正在执行的函数
  }
}

class B extends A {
  constructor() {
    super(); //相当于A.prototype.constructor.call(this)
  }
}

new A(); // A
new B(); // B
```

可以看到，在 `super()` 执行时，它指向的是 `子类 B` 的构造函数，而不是`父类 A` 的构造函数。也就是说，`super()` 内部的 `this` 指向的是 `B`。

- 当做对象使用

> 在普通方法中，指向父类的原型对象；

> 在静态方法中，指向父类。

**通过 super 调用父类的方法时，`super` 会绑定子类的 `this`。**

```js
class A {
  constructor() {
    this.x = 1;
  }
  s() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {  //super相当于调用A.proptotype
    super.s();//相当于 super.s.call(this) 绑定子类的this
  }
}

let b = new B();
b.m(); // 2
```

上面代码中，`super.s()` 虽然调用的是 `A.prototytpe.s()`，但是 `A.prototytpe.s()`会绑定`子类 B` 的 `this`，<u>导致输出的是 2，而不是 1。</u>

也就是说，实际上执行的是 `super.s.call(this)。`

注意，使用 super 的时候，必须显式指定是作为函数，还是作为对象使用，否则会报错。

```js
class A {}
class B extends A {
  constructor() {
    super();
    console.log(super); // 报错
  }
}
```

