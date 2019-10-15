# React.forwardRef

```
    const FancyButton = React.forwardRef((props, ref) => (
        <button ref={ref} className="FancyButton">
            {props.children}
        </button>
        ));
    // You can now get a ref directly to the DOM button:
    const ref = React.createRef();
    <FancyButton ref={ref}>Click me!</FancyButton>;
```

FordRef创建一个Reaction组件，将它接收到的ref属性转发到树中的另一个组件。
这种技术并不常见，但在两种情况下特别有用：
1.高阶组件的ref是转发
2.将refs转发到DOM组件

# React.createRef

```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);

    this.inputRef = React.createRef();
  }

  render() {
    return <input type="text" ref={this.inputRef} />;
  }

  componentDidMount() {
    this.inputRef.current.focus();
  }
}
```
createRef创建一个ref，它可以通过ref属性附加到Reaction元素。

# React.Fragment
可以简写为`<></>`

# React.cloneElement()
React.cloneElement()克隆并返回一个新的 ReactElement （内部子元素也会跟着克隆），新返回的元素会保留有旧元素的 props、ref、key。可以传入三个参数 
1.要克隆的ReactElement；2.需要新添加的属性props；3.重新设置的子节点（会替换掉原本的子节点）
注意：当第二个参数传入名字为key值属性时，克隆后的组件拿不到this.props.key的值

```
    render() {
        let span = <span ref="span">aaa</span>;
        let spanChange = React.cloneElement(span, {name:'aaa'} ,<em>bbb</em>);
        return (
        <div>
            {spanChange}
        </div>
        );
    }             
    //结果：<span name="aaa"><em>bbb</em><span>
```

# this.forceUpdate()
forceUpdate() 就是重新运行render，有些变量不在state上，但是又想达到变量更新，重新render的效果的时候，就可以使用此方法手动触发render

# ReactDOM.createPortal
ReactDOM.createPortal将组件渲染到父节点之外的节点

`ReactDOM.createPortal(<App></App>,Node)`

# React.Children.count
返回子元素中的组件总数，等于传递给map或forEach的回调调用次数。
`React.Children.count(children)`
```
    render: function() {
        console.log(React.Children.count(this.props.children)); //2
        return (
        <ol>
            {
            this.props.children.forEach(function (child) { //和这个次数一样
                return <li>{child}</li>
            })
            }
        </ol>
        );
    }
```

# React.Children.map
```
    React.Children.map(this.props.children,function(child){
        return <li>{child}</li>
    })
```
在每一个直接子级（包含在 children 参数中的）上调用 fn 函数，此函数中的 this 指向 上下文。
如果 children 是一个内嵌的对象或者数组，它将被遍历：不会传入容器对象到 fn 中。
如果 children 参数是 null 或者 undefined，那么返回 null 或者 undefined 而不是一个空对象。

# React.Children.only
`React.Children.only(object children)`
`console.log(React.Children.only(this.props.children[0]));`

返回children中仅有的子级。否则抛出异常。
only方法接受的参数只能是一个对象，不能是多个对象（数组）
