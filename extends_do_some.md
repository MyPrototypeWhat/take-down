# 在ES6中extends做了什么

> 继承的过程：一般继承分两步，call继承+原型的继承 （分别继承父类对象属行和原型属性）

## 第一步：

**继承父类的原型**

通过`Object.create`，第一个参数是创建一个对象的原型，定义原型上的属性`constructor`指向`subclass`；把父类的原型方法继承给子类原型；

```js
function inherits(subClass, superClass) {
   subClass.prototype = Object.create(superClass && superClass.prototype, {
     constructor: {
       value: subClass,
       enumerable: false,
       writable: true,
       configurable: true
     }
   });
   if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
 }
```

## 第二步：

**call继承**

就是`super()`的处理过程，`super()`的实质就是`People.call(this);`把父类的对象方法继承给子类对象；

这也是为什么在es6的继承时必须要加上`super()`，因为不加的话无法继承到父类的对象属性。

```js
function _possibleConstructorReturn(self, call) { 
   //call指的是Object.getPrototypeOf(lily)).call(this)，也就是People.call(this)
   return call && (typeof call === "object" || typeof call === "function") ? call : self;
}
```

`lily`是`People`的子类，首先`lily`也是一个类；`extends`的过程中创建了一个自执行函数，并将父类传进去，继承父类之后再返回该子类。

```js
var lily = function(_People) {
   inherits(lily, _People);  //第一步，继承父类原型
   function lily() {         //第二步，继承父类对象属性
     return _possibleConstructorReturn(this, (lily.__proto__ || Object.getPrototypeOf(lily)).call(this));
   }
     //此时Object.getPrototypeOf(lily))返回的原型已经是父类的原型了
   _createClass(lily, [{     //第三步，创建子类自己的方法
    key: "goodbye",
    value: function goodbye() {
    	alert("goodbye");
    }
   }]);
   return lily;
 }(People);
```

