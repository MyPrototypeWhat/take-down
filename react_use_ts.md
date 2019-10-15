# 安装 React、 React-dom 类型定义文件

`yarn add @types/react `

`yarn add @types/react-dom `

# 有状态组件开发

定义state：

```tsx
interface IProps {
  color: string,
  size?: string,
}
interface IState {
  count: number,
}
class App extends React.PureComponent<IProps, IState> {
  readonly state: Readonly<IState> = {
    count: 1,
  }
  render () {
    return (
      <div>Hello world</div>
    )
  }
  componentDidMount () {
    this.state.count = 2
  }
}
export default App
```

# 无状态组件开发

在 React 的声明文件中 已经定义了一个 SFC 类型，使用这个类型可以避免我们重复定义 children、 propTypes、 contextTypes、 defaultProps、displayName 的类型。

使用 SFC 进行无状态组件开发。

```js
import { SFC } from 'react'
import { MouseEvent } from 'react'
import * as React from 'react'
interface IProps {
  onClick (event: MouseEvent<HTMLDivElement>): void,
}
const Button: SFC<IProps> = ({onClick, children}) => {
  return (
    <div onClick={onClick}>
      { children }
    </div>
  )
}
export default Button
```

``` 

# 事件处理

> 大家可以想到直接把 event 设置为 any 类型，但是这样就失去了我们对代码进行静态检查的意义。

`function handleEvent(event: any) { console.log(event.clientY) } `

**试想下当我们注册一个 Touch 事件，当我们通过 event.clientY 访问时就有问题了，因为 Touch 事件的 event 对象并没有 clientY 这个属性。**

## Event 事件对象类型 

常用 Event 事件对象类型：

- ClipboardEvent<T = Element> 剪贴板事件对象
- DragEvent<T = Element> 拖拽事件对象
- ChangeEvent<T = Element> Change 事件对象
- KeyboardEvent<T = Element> 键盘事件对象
- MouseEvent<T = Element> 鼠标事件对象
- TouchEvent<T = Element> 触摸事件对象
- WheelEvent<T = Element> 滚轮事件对象
- AnimationEvent<T = Element> 动画事件对象
- TransitionEvent<T = Element> 过渡事件对象

​```tsx
import { MouseEvent } from 'react’ 

interface IProps { 
   onClick (event: MouseEvent<HTMLDivElement>): void, 
} 
```

# Promise 类型

> Promise<T> 是一个泛型类型，T 泛型变量用于确定使用 then 方法时接收的第一个回调函数（onfulfilled）的参数类型。

```tsx
interface IResponse<T> {
  message: string,
  result: T,
  success: boolean,
}
async function getResponse (): Promise<IResponse<number[]>> {
  return {
    message: '获取成功',
    result: [1, 2, 3],
    success: true,
  }
}
getResponse()
  .then(response => {
    console.log(response.result)
  })

```

# 工具泛型使用技巧

### typeof

> 一般我们都是先定义类型，再去赋值使用，但是使用 typeof 我们可以把使用顺序倒过来。

```tsx
const options = {
  a: 1
}
type Options = typeof options
```

##### 使用 **Partial **将所有的 props 属性都变为可选值

`type Partial<T> = { [P in keyof T]?: T[P] }; `

---

**上面代码的意思是 keyof T 拿到 T 所有属性名, 然后 in 进行遍历, 将值赋给 P , 最后 T[P]取得相应属性的值，中间的 ? 用来进行设置为可选值。**

**如果 props 所有的属性值都是可选的我们可以借助 Partial 这样实现。**

<u>(**keyof** 是索引类型查询操作符。假设T是一个类型，那么keyof T产生的类型是T的属性名称字符串字面量类型构成的联合类型。)</u>

```
import { MouseEvent } from 'react'
import * as React from 'react'
interface IProps {
  color: 'red' | 'blue' | 'yellow',
  onClick (event: MouseEvent<HTMLDivElement>): void,
}
const Button: SFC<Partial<IProps>> = ({onClick, children, color}) => {
  return (
    <div onClick={onClick}>
      { children }
    </div>
  )
```

---

#### 条件类型

> TypeScript2.8引入了条件类型，条件类型可以根据其他类型的特性做出类型的判断。

`T extends U ? X : Y `

原先

```
interface Id { id: number, /* other fields */ }
interface Name { name: string, /* other fields */ }
declare function createLabel(id: number): Id;
declare function createLabel(name: string): Name;
declare function createLabel(name: string | number): Id | Name;
```

interface Name { name: string, /* other fields */ } 

declare function createLabel(id: number): Id; 

declare function createLabel(name: string): Name; 

declare function createLabel(name: string | number): Id | Name; 

**使用条件类型之后**

```ts
type IdOrName<T extends number | string> = T extends number ? Id : Name;
declare function createLabel<T extends number | string>(idOrName: T): T extends number ? Id : Name;
```



declare function createLabel<T extends number | string>(idOrName: T): T extends number ? Id : Name; 

**通过传入的类型判断，如果T是number 那么就为Id，否则为Name**

---

#### Exclude<T,U>

>  从 T 中排除那些可以赋值给 U 的类型。

`type Exclude<T, U> = T extends U ? never : T; `

实例：

`type T = Exclude<1|2|3|4|5, 3|4> // T = 1|2|5 `

**此时 T 类型的值只可以为 1 、2 、 5 ，当使用其他值时 TS 会进行错误提示。**

`Error:(8, 5) TS2322: Type '3' is not assignable to type '1 | 2 | 5’.`

---

### Extract<T,U>

> 从 T 中提取那些可以赋值给 U 的类型。

`type Extract<T, U> = T extends U ? T : never; `

`type T = Extract<1|2|3|4|5, 3|4> // T = 3|4 `

**此时T类型的值只可以为 3 、4 ，当使用其他值时 TS 会进行错误提示：**

`Error:(8, 5) TS2322: Type '5' is not assignable to type '3 | 4'.`

---

### Pick<T,K>

> 从 T 中取出一系列 K 的属性。

```
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

<u>假如我们现在有一个类型其拥有 name 、 age 、 sex 属性，当我们想生成一个新的类型只支持 name 、age 时可以像下面这样：</u>

```
interface Person {
  name: string,
  age: number,
  sex: string,
}
let person: Pick<Person, 'name' | 'age'> = {
  name: '小王',
  age: 21,
}
```

 name: string, 

 age: number, 

 sex: string, 

} 

let person: Pick<Person, 'name' | 'age'> = { 

 name: '小王', 

 age: 21, 

} 

---

### Record<K,T>

> 将 K 中所有的属性的值转化为 T 类型。

```
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

[P in K]: T;

}; 

将 **name** 、 **age** 属性全部设为 **string** 类型。

```
let person: Record<'name' | 'age', string> = {
  name: '小王',
  age: '12',
}
```

 name: '小王', 

 age: '12', 

} 

---

### Omit<T,K>（没有内置）

> 从对象 T 中排除 key 是 K 的属性。

> > 由于 TS 中没有内置，所以需要我们使用 Pick 和 Exclude 进行实现。

`type Omit<T, K> = Pick<T, Exclude<keyof T, K>> `

排除 **name** 属性。

```
interface Person {
  name: string,
  age: number,
  sex: string,
}

let person: Omit<Person, 'name'> = {
  age: 1,
  sex: '男'
}
```

---

### NonNullable <T>

> 排除 T 为 null 、undefined。

`type NonNullable<T> = T extends null | undefined ? never : T; `

`type T = NonNullable<string | string[] | null | undefined>; `<!-- string | string[] `-->

---

### ReturnType<T>

>获取函数 T 返回值的类型。。

`type ReturnType<T extends (...args: any[]) => any> = T extends (...args: any[]) => infer R ? R : any; `

 **<u>infer R</u> 相当于声明一个变量，接收传入函数的返回值类型。**

```
type T1 = ReturnType<() => string>; // string 

type T2 = ReturnType<(s: string) => void>; // void 
```

