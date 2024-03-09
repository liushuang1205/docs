## JSX 本质

- `React.createElement` 即`h`函数，返回`vnode`
- 第一个参数，可能是组件，也可能是`html tag`
- 组件名，首字母必须是大写（`React`规定）

```jsx
  // React.createElement 写法
  React.createElement('tag', null, [child1,child2])
  React.createElement('tag', props, child1,child2,child3)
  React.createElement(Comp, props, child1,child2,'文本节点')

  // jsx 基本用法
    <div className="container"> <p>tet</p> <img src={imgSrc} /> </div>

  // 编译后 https://babeljs.io/repl
  React.createElement( "div", { className: "container" }, React.createElement("p", null, "tet"), React.createElement("img", { src: imgSrc }) );

  // jsx style
  // const styleData = {fontSize:'20px',color:'#f00'}
  // const styleElem = <p style={styleData}> 设置 style</p>

  // 编译后
  const styleData = { fontSize: "20px", color: "#f00" };
  const styleElem = React.createElement( "p", { style: styleData }, "\u8BBE\u7F6Estyle" );

  // jsx 加载组件
  const app = <div> <Input submitTitle={onSubmitTitle} /> <List list={list} /> </div>

  // 编译后
  const app = React.createElement( "div", null, React.createElement(Input, { submitTitle: onSubmitTitle }), React.createElement(List, { list: list }) );

  // jsx 事件
  const eventList = <p onClick={this.clickHandler}>text</p>

  // 编译后
  const eventList = React.createElement( "p", { onClick: (void 0).clickHandler }, "text" );

  // jsx 列表
  const listElem = <ul> { this.state.list.map((item,index)=>{ return <li key={index}>index:{index},title:{item.title}</li> }) }</ul>

  // 编译后
  const listElem = React.createElement( "ul", null, (void 0).state.list.map((item, index) => { return React.createElement( "li", { key: index }, "index:", index, ",title:", item.title ); }) );
```

## React 合成事件机制

- `React16`事件绑定到`document`上
- `React17`事件绑定到`root`组件上，有利于多个`react`版本共存，例如微前端
- `event`不是原生的，是`SyntheticEvent`合成事件对象
- 和`Vue`不同，和`DOM`事件也不同

![1cd8ea55c04e3266f5c3c895287d2075](_media/1cd8ea55c04e3266f5c3c895287d2075.png)

**合成事件图示**

![9d7b2ea2d0b66c81b588b26b384123f9](_media/9d7b2ea2d0b66c81b588b26b384123f9.png)

**为何需要合成事件**

- 更好的兼容性和跨平台，如`react native`
- 挂载到`document`或`root`上，减少内存消耗，避免频繁解绑
- 方便事件的统一管理（如事务机制）

```
  // 获取
  event clickHandler3 = (event) => {
    event.preventDefault()
    // 阻止默认行为
    event.stopPropagation()
    // 阻止冒泡
    console.log('target', event.target)
    // 指向当前元素，即当前元素触发
    console.log('current target', event.currentTarget)
    // 指向当前元素，假象！！！
    // 注意，event 其实是 React 封装的。可以看 __proto__.constructor 是 SyntheticEvent 组合事件
    console.log('event', event) // 不是原生的 Event ，原生的 MouseEvent
    console.log('event.__proto__.constructor', event.__proto__.constructor)

    // 原生 event 如下。其 __proto__.constructor 是 MouseEvent
    console.log('nativeEvent', event.nativeEvent)
    console.log('nativeEvent target', event.nativeEvent.target)  // 指向当前元素，即当前元素触发
    console.log('nativeEvent current target', event.nativeEvent.currentTarget) // 指向 document ！！！

    // 1. event 是 SyntheticEvent ，模拟出来 DOM 事件所有能力
    // 2. event.nativeEvent 是原生事件对象
    // 3. 所有的事件，都被挂载到 document 上
    // 4. 和 DOM 事件不一样，和 Vue 事件也不一样

  }
```

## setState 和 batchUpdate 机制

- `setState`在 react 事件、生命周期中是异步的（在`react`上下文中是异步）；在`setTimeout`、自定义`DOM`事件中是同步的
- 有时合并（对象形式`setState({})` =\> 通过`Object.assign`形式合并对象），有时不合并（函数形式`setState((prevState,nextState)=>{})`）

**核心要点**

1.`setState`主流程

- `setState`是否是异步还是同步，看是否能命中`batchUpdate`机制，判断`isBatchingUpdates`
- 哪些能命中`batchUpdate`机制
  - 生命周期
  - `react`中注册的事件和它调用的函数
  - 总之在`react`的上下文中
- 哪些不能命中`batchUpdate`机制
  - `setTimeout`、`setInterval`等
  - 自定义`DOM`事件
  - 总之不在`react`的上下文中，`react`管不到的

![0a28831178c602332201837c149ec903](_media/0a28831178c602332201837c149ec903.png)

2. `batchUpdate`机制

![77594456f103cc4287a87c7a5b84c58d](_media/77594456f103cc4287a87c7a5b84c58d.png)

![f7383652c7c3796f90b88379341ba5d5](_media/f7383652c7c3796f90b88379341ba5d5.png)

    // setState batchUpdate原理模拟
    let isBatchingUpdate = true;

    let queue = [];
    let state = {number:0};
    function setState(newSate){
      //state={...state,...newSate}
      // setState异步更新
      if(isBatchingUpdate){
        queue.push(newSate);
      }else{
        // setState同步更新
        state={...state,...newSate}
      }
    }

    // react事件是合成事件，在合成事件中isBatchingUpdate需要设置为true
    // 模拟react中事件点击
    function handleClick(){
      isBatchingUpdate=true; // 批量更新标志

      /**我们自己逻辑开始 */
      setState({number:state.number+1});
      setState({number:state.number+1});
      console.log(state); // 0
      setState({number:state.number+1});
      console.log(state); // 0
      /**我们自己逻辑结束 */

      state= queue.reduce((newState,action)=>{
        return {...newState,...action}
      },state);

      isBatchingUpdate=false; // 执行结束设置false
    }
    handleClick();
    console.log(state); // 1

3. `transaction`事务机制

![1eed36740c0f9abd4a697c6f127f9e3c](_media/1eed36740c0f9abd4a697c6f127f9e3c.png)

![d2015431aeb3ab7a6effbc47ebc642d7](_media/d2015431aeb3ab7a6effbc47ebc642d7.png)

![d37bf9e9a1aaf2d3800053ec29f87f1a](_media/d37bf9e9a1aaf2d3800053ec29f87f1a.png)

    // setState现象演示

    import React from 'react'

    // 函数组件（后面会讲），默认没有 state
    class StateDemo extends React.Component {
        constructor(props) {
            super(props)

            // 第一，state 要在构造函数中定义
            this.state = {
                count: 0
            }
        }
        render() {
            return <div>
                <p>{this.state.count}</p>
                <button onClick={this.increase}>累加</button>
            </div>
        }
        increase = () => {
            // // 第二，不要直接修改 state ，使用不可变值 ----------------------------
            // // this.state.count++ // 错误
            // this.setState({
            // count: this.state.count + 1 // SCU
            // })
            // 操作数组、对象的的常用形式

            // 第三，setState 可能是异步更新（有可能是同步更新） ----------------------------

            // this.setState({
            // count: this.state.count + 1
            // }, () => {
            // // 联想 Vue $nextTick - DOM
            // console.log('count by callback', this.state.count) // 回调函数中可以拿到最新的 state
            // })
            // console.log('count', this.state.count) // 异步的，拿不到最新值

            // // setTimeout 中 setState 是同步的
            // setTimeout(() => {
            // this.setState({
            // count: this.state.count + 1
            // })
            // console.log('count in setTimeout', this.state.count)
            // }, 0)

            // 自己定义的 DOM 事件，setState 是同步的。再 componentDidMount 中

            // 第四，state 异步更新的话，更新前会被合并 ----------------------------

            // 传入对象，会被合并（类似 Object.assign ）。执行结果只一次 +1
            // this.setState({
            // count: this.state.count + 1
            // })
            // this.setState({
            // count: this.state.count + 1
            // })
            // this.setState({
            // count: this.state.count + 1
            // })

            // 传入函数，不会被合并。执行结果是 +3
            this.setState((prevState, props) => {
                return {
                    count: prevState.count + 1
                }
            })
            this.setState((prevState, props) => {
                return {
                    count: prevState.count + 1
                }
            })
            this.setState((prevState, props) => {
                return {
                    count: prevState.count + 1
                }
            })
        }
        // bodyClickHandler = () => {
        // this.setState({
        // count: this.state.count + 1
        // })
        // console.log('count in body event', this.state.count)
        // }
        // componentDidMount() {
        // // 自己定义的 DOM 事件，setState 是同步的
        // document.body.addEventListener('click', this.bodyClickHandler)
        // }
        // componentWillUnmount() {
        // // 及时销毁自定义 DOM 事件
        // document.body.removeEventListener('click', this.bodyClickHandler)
        // // clearTimeout
        // }
    }

    export default StateDemo

    // -------------------------- 我是分割线 -----------------------------

    // 不可变值（函数式编程，纯函数） - 数组
    // const list5Copy = this.state.list5.slice()
    // list5Copy.splice(2, 0, 'a') // 中间插入/删除
    // this.setState({
    // list1: this.state.list1.concat(100), // 追加
    // list2: [...this.state.list2, 100], // 追加
    // list3: this.state.list3.slice(0, 3), // 截取
    // list4: this.state.list4.filter(item => item > 100), // 筛选
    // list5: list5Copy // 其他操作
    // })
    // // 注意，不能直接对 this.state.list 进行 push pop splice 等，这样违反不可变值

    // 不可变值 - 对象
    // this.setState({
    // obj1: Object.assign({}, this.state.obj1, {a: 100}),
    // obj2: {...this.state.obj2, a: 100}
    // })
    // // 注意，不能直接对 this.state.obj 进行属性设置，这样违反不可变值

    // setState笔试题考察 下面这道题输出什么
    class Example extends React.Component {
      constructor() {
      super()
      this.state = {
        val: 0
      }
    }
    // componentDidMount中isBatchingUpdate=true setState批量更新
    componentDidMount() {
      // setState传入对象会合并，后面覆盖前面的Object.assign({})
      this.setState({ val: this.state.val + 1 }) // 添加到queue队列中，等待处理
      console.log(this.state.val)
      // 第 1 次 log
      this.setState({ val: this.state.val + 1 }) // 添加到queue队列中，等待处理
      console.log(this.state.val)
      // 第 2 次 log
      setTimeout(() => {
        // 到这里this.state.val结果等于1了
        // 在原生事件和setTimeout中（isBatchingUpdate=false），setState同步更新，可以马上获取更新后的值
        this.setState({ val: this.state.val + 1 }) // 同步更新
        console.log(this.state.val)
        // 第 3 次 log
        this.setState({ val: this.state.val + 1 }) // 同步更新
        console.log(this.state.val)
        // 第 4 次 log
        }, 0)
      }
      render() {
        return null
      }
    }

    // 答案：0, 0, 2, 3

## 根据 jsx 写出 vnode 和 render 函数

    <!-- jsx -->
    <div className="container">
      <p onClick={onClick} data-name="p1">
        hello <b>{name}</b>
      </p>
      <img src={imgSrc} />
      <MyComponent title={title}></MyComponent>
    </div>

**注意**

- 注意`JSX`中的常量和变量
- 注意`JSX`中的`HTML tag`和自定义组件

```jsx
  const vnode = {
    tag: 'div',
    props: {
      className: 'container'
    },
    children: [
      // <p>
      {
        tag: 'p',
        props: {
          dataset: {
            name: 'p1'
          },
          on: {
            click: onClick // 变量
          }
        },
        children: [
          'hello',
          {
            tag: 'b',
            props: {},
            children: [name] // name 变量
          }
        ]
      },
      // <img />
      {
        tag: 'img',
        props: {
          src: imgSrc // 变量
        },
        children: [
          /**无子节点**/
        ]
      },
      // <MyComponent>
      { tag: MyComponent, // 变量
        props: {
          title: title, // 变量
        },
        children: [/**无子节点**/]
      }
    ]
  }

  // render 函数 function render() { // h(tag, props, children) return h('div', { props: { className: 'container' } }, [

        // p
        h('p', {
          dataset: {
            name: 'p1'
          },
          on: {
            click: onClick
          }
        }, [
          'hello',
          h('b', {}, [name])
        ])

        // img
        h('img', {
          props: {
            src: imgSrc
          }
        }, [/**无子节点**/])

        // MyComponent
        h(MyComponent, {
          title: title
        }, [/**无子节点**/])
      ]
      )

  }
```

**在 react 中 jsx 编译后**

    // 使用https://babeljs.io/repl编译后效果

    React.createElement(
      "div",
      {
        className: "container"
      },
      React.createElement(
        "p",
        {
          onClick: onClick,
          "data-name": "p1"
        },
        "hello ",
        React.createElement("b", null, name)
      ),
      React.createElement("img", {
        src: imgSrc
      }),
      React.createElement(MyComponent, {
        title: title
      })
    );

## 虚拟 DOM（vdom）真的很快吗

- `virutal DOM`，虚拟`DOM`
- 用 JS 对象模拟`DOM`节点数据
- `vdom`并不快，`JS`直接操作`DOM`才是最快的
  - 以`vue`为例，`data`变化 =\> `vnode diff` =\> 更新`DOM` 肯定是比不过直接操作`DOM`节点快的
- 但是"数据驱动视图"要有合适的技术方案，不能全部`DOM`重建
- `dom`就是目前最合适的技术方案（并不是因为它快，而是合适）
- 在大型系统中，全部更新`DOM`的成本太高，使用`vdom`把更新范围减少到最小

> 并不是所有的框架都在用`vdom`，`svelte`就不用`vdom`

![d490ce91deb1d80baf4d7a27cf36c825](_media/d490ce91deb1d80baf4d7a27cf36c825.png)

## react 组件渲染过程

- `JSX`如何渲染为页面
- `setState`之后如何更新页面
- 面试考察全流程

**1.组件渲染过程**

- 分析
  - `props`、`state` 变化
  - `render()`生成`vnode`
  - `patch(elem, vnode)` 渲染到页面上（`react`并一定用`patch`）
- 渲染过程

  - `setState(newState)` =\> `newState`存入`pending`队列，判断是否处于`batchUpdate`状态，保存组件于`dirtyComponents`中（可能有子组件）

    ![0a28831178c602332201837c149ec903](_media/0a28831178c602332201837c149ec903.png)

  - 遍历所有的`dirtyComponents`调用`updateComponent`生成`newVnode`
  - `patch(vnode,newVnode)`

**2.组件更新过程**

- `patch`更新被分为两个阶段
  - **reconciliation 阶段** ：执行`diff`算法，纯`JS`计算
  - **commit 阶段** ：将`diff`结果渲染到`DOM`中
- 如果不拆分，可能有性能问题
  - `JS`是单线程的，且和`DOM`渲染共用一个线程
  - 当组件足够复杂，组件更新时计算和渲染都压力大
  - 同时再有`DOM`操作需求（动画、鼠标拖拽等）将卡顿
- **解决方案 Fiber**
  - `reconciliation`阶段拆分为多个子任务
  - `DOM`需要渲染时更新，空闲时恢复在执行计算
  - 通过`window.requestIdleCallback`来判断浏览器是否空闲

## React setState 经典面试题

    // setState笔试题考察 下面这道题输出什么
    class Example extends React.Component {
      constructor() {
      super()
      this.state = {
        val: 0
      }
    }
    componentDidMount() {
      this.setState({ val: this.state.val + 1 })
      console.log(this.state.val)
      // 第 1 次 log
      this.setState({ val: this.state.val + 1 })
      console.log(this.state.val)
      // 第 2 次 log
      setTimeout(() => {
        this.setState({ val: this.state.val + 1 })
        console.log(this.state.val)
        // 第 3 次 log
        this.setState({ val: this.state.val + 1 })
        console.log(this.state.val)
        // 第 4 次 log
        }, 0)
      }
      render() {
        return null
      }
    }

    // 答案
    0
    0
    2
    3

- **关于 setState 的两个考点**
  - 同步或异步
  - `state`合并或不合并
    - `setState`传入函数不会合并覆盖
    - `setState`传入对象会合并覆盖`Object.assigin({})`
- 分析
  - **默认情况**
    - `state`默认异步更新
    - `state`默认合并后更新（后面的覆盖前面的，多次重复执行不会累加）
  - `setState`在合成事件和生命周期钩子中，是异步更新的
  - **react 同步更新，不在 react 上下文中触发**
    - 在`原生事件`、`setTimeout`、`setInterval`、`promise.then`、`Ajax`回调中，`setState`是同步的，可以马上获取更新后的值
      - 原生事件如`document.getElementById('test').addEventListener('click',()=>{this.setState({count:this.state.count + 1}})`
    - 原因: 原生事件是浏览器本身的实现，与事务流无关，自然是同步；而`setTimeout`是放置于定时器线程中延后执行，此时事务流已结束，因此也是同步
  - **注意：在 react18 中不一样**
    - 上述场景，在`react18`中可以异步更新（`Auto Batch`）
    - 需将`ReactDOM.render`替换为`ReactDOM.createRoot`

如需要实时获取结果，在回调函数中获取 `setState({count:this.state.count + 1},()=>console.log(this.state.count)})`

    // setState原理模拟
    let isBatchingUpdate = true;

    let queue = [];
    let state = {number:0};
    function setState(newSate){
      //state={...state,...newSate}
      // setState异步更新
      if(isBatchingUpdate){
        queue.push(newSate);
      }else{
        // setState同步更新
        state={...state,...newSate}
      }
    }

    // react事件是合成事件，在合成事件中isBatchingUpdate需要设置为true
    // 模拟react中事件点击
    function handleClick(){
      isBatchingUpdate=true; // 批量更新标志

      /**我们自己逻辑开始 */
      setState({number:state.number+1});
      setState({number:state.number+1});
      console.log(state); // 0
      setState({number:state.number+1});
      console.log(state); // 0
      /**我们自己逻辑结束 */

      state= queue.reduce((newState,action)=>{
        return {...newState,...action}
      },state);
    }
    handleClick();
    console.log(state); // 1

    // setState笔试题考察 下面这道题输出什么
    class Example extends React.Component {
      constructor() {
      super()
      this.state = {
        val: 0
      }
    }
    // componentDidMount中isBatchingUpdate=true setState批量更新
    componentDidMount() {
      // setState传入对象会合并，后面覆盖前面的Object.assign({})
      this.setState({ val: this.state.val + 1 }) // 添加到queue队列中，等待处理
      console.log(this.state.val)
      // 第 1 次 log
      this.setState({ val: this.state.val + 1 }) // 添加到queue队列中，等待处理
      console.log(this.state.val)
      // 第 2 次 log
      setTimeout(() => {
        // 到这里this.state.val结果等于1了
        // 在原生事件和setTimeout中（isBatchingUpdate=false），setState同步更新，可以马上获取更新后的值
        this.setState({ val: this.state.val + 1 }) // 同步更新
        console.log(this.state.val)
        // 第 3 次 log
        this.setState({ val: this.state.val + 1 }) // 同步更新
        console.log(this.state.val)
        // 第 4 次 log
        }, 0)
      }
      render() {
        return null
      }
    }

    // 答案：0, 0, 2, 3

在`React 18`之前，`setState`在`React`的合成事件中是合并更新的，在`setTimeout`的原生事件中是同步按序更新的。例如

    handleClick = () => {
      this.setState({ age: this.state.age + 1 });
      console.log(this.state.age); // 0
      this.setState({ age: this.state.age + 1 });
      console.log(this.state.age); // 0
      this.setState({ age: this.state.age + 1 });
      console.log(this.state.age); // 0
      setTimeout(() => {
        this.setState({ age: this.state.age + 1 });
        console.log(this.state.age); // 2
        this.setState({ age: this.state.age + 1 });
        console.log(this.state.age); // 3
      });
    };

而在`React 18`中，不论是在合成事件中，还是在宏任务中，都是会合并更新

    function handleClick() {
      setState({ age: state.age + 1 }, onePriority);
      console.log(state.age);// 0
      setState({ age: state.age + 1 }, onePriority);
      console.log(state.age); // 0
      setTimeout(() => {
        setState({ age: state.age + 1 }, towPriority);
        console.log(state.age); // 1
        setState({ age: state.age + 1 }, towPriority);
        console.log(state.age); // 1
      });
    }

    // 拓展：setState传入函数不会合并
    class Example extends React.Component {
      constructor() {
      super()
      this.state = {
        val: 0
      }
    }
    componentDidMount() {
      this.setState((prevState,props)=>{
        return {val: prevState.val + 1}
      })
      console.log(this.state.val) // 0
      // 第 1 次 log
      this.setState((prevState,props)=>{ // 传入函数，不会合并覆盖前面的
        return {val: prevState.val + 1}
      })
      console.log(this.state.val) // 0
      // 第 2 次 log
      setTimeout(() => {
        // setTimeout中setState同步执行
        // 到这里this.state.val结果等于2了
        this.setState({ val: this.state.val + 1 })
        console.log(this.state.val) // 3
        // 第 3 次 log
        this.setState({ val: this.state.val + 1 })
        console.log(this.state.val) // 4
        // 第 4 次 log
        }, 0)
      }
      render() {
        return null
      }
    }
    // 答案：0 0 3 4

    // react hooks中打印

    function useStateDemo() {
      const [value, setValue] = useState(100)

      function clickHandler() {
        // 1.传入常量，state会合并
        setValue(value + 1)
        setValue(value + 1)
        console.log(1, value) // 100
        // 2.传入函数，state不会合并
        setValue(value=>value + 1)
        setValue(value=>value + 1)
        console.log(2, value) // 100

        // 3.setTimeout中，React18也开始合并state（之前版本会同步更新、不合并）
        setTimeout(()=>{
          setValue(value + 1)
          setValue(value + 1)
          console.log(3, value) // 100
          setValue(value + 1)
        })

        // 4.同理 setTimeout中，传入函数不合并
        setTimeout(()=>{
          setValue(value => value + 1)
          setValue(value => value + 1)
          console.log(4, value) // 100
        })
      }
      return (
        <button onClick={clickHandler}>点击 {value}</button>
      )
    }

**连环问：setState 是宏任务还是微任务**

- **setState 本质是同步的**
  - `setState`是同步的，不过让`react`做成异步更新的样子而已
    - 如果`setState`是微任务，就不应该在`promise.then`微任务之前打印出来（`promise then`微任务先注册）
  - 因为要考虑性能，多次`state`修改，只进行一次`DOM`渲染
  - 日常所说的"异步"是不严谨的，但沟通成本低
- **总结**

  - `setState`是同步执行，`state`都是同步更新（只是我们日常把`setState`当异步来处理）
  - 在微任务`promise.then`之前，`state`已经计算完了
  - 同步，不是微任务或宏任务

```js
    import React from 'react'

    class Example extends React.Component { constructor() { super() this.state = { val: 0 } }

    clickHandler = () => { // react 事件中 setState 异步执行 console.log('--- start ---')

        Promise.resolve().then(() => console.log('promise then') /* callback */)

        // "异步"
        this.setState(
          { val: this.state.val + 1 },
          () => { console.log('state callback...', this.state) } // callback
        )

        console.log('--- end ---')

        // 结果:
        // start
        // end
        // state callback {val:1}
        // promise then

        // 疑问？
        // promise then微任务先注册的，按理应该先打印promise then再到state callback
        // 因为：setState本质是同步的，不过让react做成异步更新的样子而已
        // 因为要考虑性能，多次state修改，只进行一次DOM渲染

    }

    componentDidMount() { setTimeout(() => { // setTimeout 中 setState 是同步更新 console.log('--- start ---')

          Promise.resolve().then(() => console.log('promise then'))

          this.setState(
            { val: this.state.val + 1 }
          )
          console.log('state...', this.state)

          console.log('--- end ---')
        })

        // 结果:
        // start
        // state {val:1}
        // end
        // promise then

    }

    render() { return <p id="p1" onClick={this.clickHandler}> setState demo: {this.state.val} </p> } }
```

## React useEffect 闭包陷阱问题

问：按钮点击三次后，定时器输出什么？

    function useEffectDemo() {
      const [value,setValue] = useState(0)

      useEffect(()=>{
        setInterval(()=>{
          console.log(value)
        },1000)
      }, [])

      const clickHandler = () => {
        setValue(value + 1)
      }

      return (
        <div>
          value: {value} <button onClick={clickHandler}>点击</button>
        </div>
      )
    }

答案一直是`0` `useEffect`闭包陷阱问题，`useEffect`依赖是空的，只会执行一次。`setInterval`中的`value`就只会获取它之前的变量。而`react`有个特点，每次`value`变化都会重新执行`useEffectDemo`这个函数。点击了三次函数会执行三次，三次过程中每个函数中`value`都不一样，`setInterval`获取的永远是第一个函数里面的`0`

    // 追问：怎么才能打印出3？

    function useEffectDemo() {
      const [value,setValue] = useState(0)

      useEffect(()=>{
        const timer = setInterval(()=>{
          console.log(value) // 3
        },1000)
        return ()=>{
          clearInterval(timer) // value变化会导致useEffectDemo函数多次执行，多次执行需要清除上一次的定时器，否则多次注册定时器
        }
      }, [value]) // 这里增加依赖项，每次依赖变化都会重新执行

      const clickHandler = () => {
        setValue(value + 1)
      }

      return (
        <div>
          value: {value} <button onClick={clickHandler}>点击</button>
        </div>
      )
    }

## Vue React diff 算法有什么区别

**diff 算法**

- `Vue React diff` 不是对比文字，而是 `vdom` 树，即 `tree diff`
- 传统的 `tree diff` 算法复杂度是 `O(n^3)` ，算法不可用。

![d9124cf9b1541750b04fac441d2d9f21](_media/d9124cf9b1541750b04fac441d2d9f21.png)

**优化**

> `Vue React` 都是用于网页开发，基于 `DOM` 结构，对 `diff` 算法都进行了优化（或者简化）

- 只在同一层级比较，不跨层级（`DOM` 结构的变化，很少有跨层级移动）
- `tag` 不同则直接删掉重建，不去对比内部细节（`DOM` 结构变化，很少有只改外层，不改内层）
- 同一个节点下的子节点，通过 `key` 区分

> 最终把时间复杂度降低到 `O(n)` ，生产环境下可用。这一点 `Vue React` 都是相同的。

![a810bab365ec139ac934355376ddc260](_media/a810bab365ec139ac934355376ddc260.png)

**React diff 特点 - 仅向右移动**

> 比较子节点时，仅向右移动，不向左移动。

![6b0f90e125981b2eab53ed13b6a76be4](_media/6b0f90e125981b2eab53ed13b6a76be4.png)

**Vue2 diff 特点 - 双端比较**

![53719e49505f01faf1448e3b24d40de7](_media/53719e49505f01faf1448e3b24d40de7.png)

定义四个指针，分别比较

- `oldStartNode` 和 `newStartNode` 头头
- `oldStartNode` 和 `newEndNode` 头尾
- `oldEndNode` 和 `newStartNode` 尾头
- `oldEndNode` 和 `newEndNode` 尾尾

> 然后指针继续向中间移动，直到指针汇合

**Vue3 diff 特点 - 最长递增子序列**

> 例如数组 `[3，5，7，1，2，8]` 的最长递增子序列就是 `[3，5，7，8 ]` 。这是一个专门的算法。

![b5622d70456b1271d3bb263620d90548](_media/b5622d70456b1271d3bb263620d90548.png)

**算法步骤**

- 通过"前-前"比较找到开始的不变节点 `[A, B]`
- 通过"后-后"比较找到末尾的不变节点 `[G]`
- 剩余的有变化的节点 `[F, C, D, E, H]`
  - 通过 `newIndexToOldIndexMap` 拿到 `oldChildren` 中对应的 `index` `[5, 2, 3, 4, -1]` （`-1` 表示之前没有，要新增）
  - 计算**最长递增子序列** 得到 `[2, 3, 4]` ，对应的就是 `[C, D, E]` ，即这些节点可以不变
  - 剩余的节点，根据 `index` 进行新增、删除

> 该方法旨在尽量减少 `DOM` 的移动，`达到最少的DOM操作`。

**总结**

- `React diff` 特点 - 仅向右移动
- `Vue2 diff` 特点 - `updateChildren`双端比较
- `Vue3 diff` 特点 - `updateChildren`增加了最长递增子序列，更快
  - `Vue3`增加了`patchFlag`、静态提升、函数缓存等

**连环问：diff 算法中 key 为何如此重要**

无论在 `Vue` 还是 React 中，`key` 的作用都非常大。以 `React` 为例，是否使用 `key` 对内部 `DOM` 变化影响非常大。

![e59105a477b3529f94c7d34961303497](_media/e59105a477b3529f94c7d34961303497.png)

    <ul>
      <li v-for="(index, num) in nums" :key="index">
        {{num}}
      </li>
    </ul>

    const todoItems = todos.map((todo) =>
      <li key={todo.id}> {todo.text} </li>
    )

## 如何统一监听 React 组件报错

- **ErrorBoundary 组件**
  - 在`react16`版本之后，增加了`ErrorBoundary`组件
  - 监听所有`下级组件`报错，可降级展示`UI`
  - 只监听组件渲染时报错，不监听`DOM`事件错误、异步错误
    - `ErrorBoundary`没有办法监听到点击按钮时候的在`click`的时候报错
    - 只能监听组件从一开始渲染到渲染成功这段时间报错，渲染成功后在怎么操作产生的错误就不管了
    - 可用`try catch`或者`window.onerror`（二选一）
  - 只在`production`环境生效(需要打包之后查看效果)，`dev`会直接抛出错误
- **总结**

  - `ErrorBoundary`监听组件渲染报错
  - 事件报错使用`try catch`或`window.onerror`
  - 异步报错使用`window.onerror`

```js
// ErrorBoundary.js

import React from "react";

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      error: null
      // 存储当前的报错信息
    };
  }
  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染能够显示降级后的 UI
    console.info("getDerivedStateFromError...", error);
    return { error };
    // return 的信息会等于 this.state 的信息
  }
  componentDidCatch(error, errorInfo) {
    // 统计上报错误信息
    console.info("componentDidCatch...", error, errorInfo);
  }
  render() {
    if (this.state.error) {
      // 提示错误
      return <h1>报错了</h1>;
    }

    // 没有错误，就渲染子组件
    return this.props.children;
  }
}

// index.js 中使用
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import ErrorBoundary from "./ErrorBoundary";

ReactDOM.render(
  <React.StrictMode>
    <ErrorBoundary>
      <App />
    </ErrorBoundary>
  </React.StrictMode>,
  document.getElementById("root")
);
```

## 在实际工作中，你对 React 做过哪些优化

- **修改 CSS 模拟 v-show**

      // 原始写法
      {!flag && <MyComonent style={{display:'none'}} />}
      {flag && <MyComonent />}

      // 模拟v-show
      {<MyComonent style={{display:flag ? 'block' : 'none'}} />}

- **循环使用 key**
  - `key`不要用`index`
- **使用 Flagment 或\<\>\</\>空标签包裹减少多个层级组件的嵌套**
- **jsx 中不要定义函数** ：`JSX`会被频繁执行的

      // bad
      // react中的jsx被频繁执行（state更改）应该避免函数被多次新建
      <button onClick={()=>{}}>点击</button>
      // goods
      function useButton() {
        const handleClick = ()=>{}
        return <button onClick={handleClick}>点击</button>
      }

- **使用 shouldComponentUpdate**
  - 判断组件是否需要更新
  - 或者使用`React.PureComponent`比较`props`第一层属性
  - 函数组件使用`React.memo(comp, fn)`包裹 `function fn(prevProps,nextProps) {// 自己实现对比，像shouldComponentUpdate}`
- **Hooks 缓存数据和函数**
  - `useCallback`: 缓存回调函数，避免传入的回调每次都是新的函数实例而导致依赖组件重新渲染，具有性能优化的效果
  - `useMemo`: 用于缓存传入的 `props`，避免依赖的组件每次都重新渲染
- **使用异步组件**

      import React,{lazy,Suspense} from 'react'
      const OtherComp = lazy(/**webpackChunkName:'OtherComp'**/ ()=>import('./otherComp'))

      function MyComp(){
        return (
          <Suspense fallback={<div>loading...</div>}>
            <OtherComp />
          </Suspense>
        )
      }

- **路由懒加载**

      import React,{lazy,Suspense} from 'react'
      import {BrowserRouter as Router,Route, Switch} from 'react-router-dom'

      const Home = lazy(/**webpackChunkName:'h=Home'**/()=>import('./Home'))
      const List = lazy(/**webpackChunkName:'List'**/()=>import('./List'))

      const App = ()=>(
        <Router>
          <Suspense fallback={<div>loading...</div>}>
            <Switch>
              <Route exact path='/' component={Home} />
              <Route exact path='/list' component={List} />
            </Switch>
          </Suspense>
        </Router>
      )

- **使用 SSR** ：`Next.js`

**连环问：你在使用 React 时遇到过哪些坑**

- **自定义组件的名称首字母要大写**

      // 原生html组件
      <input />

      // 自定义组件
      <Input />

- **JS 关键字的冲突**

      // for改成htmlFor，class改成className
      <label htmlFor="input-name" className="label">
        用户名 <input id="username" />
      </label>

- **JSX 数据类型**

      // correct
      <Demo flag={true} />
      // error
      <Demo flag="true" />

- **setState 不会马上获取最新的结果**

  - 如需要实时获取结果，在回调函数中获取 `setState({count:this.state.count + 1},()=>console.log(this.state.count)})`
  - `setState`在合成事件和生命周期钩子中，是异步更新的
  - 在**原生事件** 和`setTimeout`中，`setState`是同步的，可以马上获取更新后的值；
  - 原因: 原生事件是浏览器本身的实现，与事务流无关，自然是同步；而`setTimeout`是放置于定时器线程中延后执行，此时事务流已结束，因此也是同步；

```js
// setState 原理模拟
let isBatchingUpdate = true;

let queue = [];
let state = { number: 0 };
function setState(newSate) {
  state = { ...state, ...newSate };
  // setState 异步更新
  if (isBatchingUpdate) {
    queue.push(newSate);
  } else {
    // setState 同步更新
    state = { ...state, ...newSate };
  }
}

// react 事件是合成事件，在合成事件中 isBatchingUpdate 需要设置为 true
// 模拟 react 中事件点击
function handleClick() {
  isBatchingUpdate = true; // 批量更新标志

  /**我们自己逻辑开始 */
  setState({ number: state.number + 1 });
  setState({ number: state.number + 1 });
  console.log(state); // 0
  setState({ number: state.number + 1 });
  console.log(state); // 0
  /**我们自己逻辑结束 */

  state = queue.reduce((newState, action) => {
    return { ...newState, ...action };
  }, state);
}

handleClick();
console.log(state);
// 1

// setState 笔试题考察 下面这道题输出什么
class Example extends React.Component {
  constructor() {
    super();
    this.state = { val: 0 };
  }
  // componentDidMount 中 isBatchingUpdate=true setState 批量更新
  componentDidMount() {
    this.setState({ val: this.state.val + 1 });

    // 添加到 queue 队列中，等待处理
    console.log(this.state.val);

    // 第 1 次 log
    this.setState({ val: this.state.val + 1 });

    // 添加到 queue 队列中，等待处理
    console.log(this.state.val);

    // 第 2 次 log
    setTimeout(() => {
      // 在原生事件和 setTimeout 中（isBatchingUpdate=false），setState 同步更新，可以马上获取更新后的值
      this.setState({ val: this.state.val + 1 });
      // 同步更新
      console.log(this.state.val);

      // 第 3 次 log
      this.setState({ val: this.state.val + 1 });
      // 同步更新
      console.log(this.state.val);

      // 第 4 次 log
    }, 0);
  }
  render() {
    return null;
  }
}

// 答案：0, 0, 2, 3
```

## React 真题

**1. 函数组件和 class 组件区别**

- 纯函数，输入`props`，输出`JSX`
- 没有实例、没有生命周期、没有`state`
- 不能拓展其他方法

**2. 什么是受控组件**

- 表单的值，受到`state`控制
- 需要自行监听`onChange`，更新`state`
- 对比非受控组件

**3. 何时使用异步组件**

- 加载大组件
- 路由懒加载

**4. 多个组件有公共逻辑如何抽离**

- `HOC`高阶组件
- `Render Props`
- `React Hooks`

**5. react router 如何配置懒加载**

![4c227e65da58245e14d23944ccb5f31b](_media/4c227e65da58245e14d23944ccb5f31b.png)

## React 和 Vue 的区别（常考）

**共同**

- 都支持组件化
- 都是数据驱动视图
- 都用`vdom`操作`DOM`

**区别**

- `React`使用`JSX`拥抱`JS`，`Vue`使用模板拥抱`HTML`
- `React`函数式编程，`Vue`是声明式编程
- `React`更多的是自力更生，`Vue`把你想要的都给你

**当比较 React 和 Vue 时，以下是一些详细的区别：**

1. 构建方式：

- React：React 是一个用于构建用户界面的 JavaScript 库。它使用 JSX 语法，将组件的结构和逻辑放在一起，通过组件的嵌套和组合来构建应用程序。
- Vue：Vue 是一个渐进式框架，可以用于构建整个应用程序或仅用于特定页面的一部分。它使用模板语法，将 HTML 模板和 JavaScript 代码分离，通过指令和组件来构建应用程序。

2. 学习曲线：

- React：React 相对来说更加灵活和底层，需要对 JavaScript 和 JSX 有一定的了解。它提供了更多的自由度和灵活性，但也需要更多的学习和理解。
- Vue：Vue 则更加简单和易于上手，它使用了模板语法和一些特定的概念，使得学习和使用起来更加直观。Vue 的文档和教程也非常友好和详细。

3. 数据绑定：

- React：React 使用单向数据流，通过 props 将数据从父组件传递到子组件。如果需要在子组件中修改数据，需要通过回调函数来实现。
- Vue：Vue 支持双向数据绑定，可以通过 v-model 指令实现数据的双向绑定。这使得在 Vue 中处理表单和用户输入更加方便。

4. 组件化开发：

- React：React 的组件化开发非常灵活，组件可以通过 props 接收数据，通过 state 管理内部状态。React 还提供了生命周期方法，可以在组件的不同阶段执行特定的操作。
- Vue：Vue 的组件化开发也非常强大，组件可以通过 props 接收数据，通过 data 属性管理内部状态。Vue 还提供了生命周期钩子函数，可以在组件的不同阶段执行特定的操作。

5. 生态系统：

- React：React 拥有庞大的生态系统，有许多第三方库和工具可供选择。React 还有一个强大的社区支持，提供了大量的教程、文档和示例代码。
- Vue：Vue 的生态系统也很活跃，虽然相对 React 来说规模较小，但也有许多第三方库和工具可供选择。Vue 的文档和教程也非常友好和详细。

6. 性能：

- React：React 通过虚拟 DOM（Virtual DOM）和高效的 diff 算法来提高性能。它只更新需要更新的部分，减少了对实际 DOM 的操作次数。
- Vue：Vue 也使用虚拟 DOM 来提高性能，但它采用了更细粒度的观察机制，可以精确追踪数据的变化，从而减少不必要的更新操作。

## 7 React Hooks

## class 组件存在哪些问题

- **函数组件的特点**
  - 没有组件实例
  - 没有生命周期
  - 没有`state`和`setState`，只能接收`props`
- **class 组件问题**
  - 大型组件很难拆分和重构，很难测试
  - 相同的业务逻辑分散到各个方法中，逻辑混乱
  - 复用逻辑变得复杂，如`Mixins`、`HOC`、`Render Props`
- **react 组件更易用函数表达**
  - React 提倡函数式编程，`View = fn(props)`
  - 函数更灵活，更易于拆分，更易测试
  - 但函数组件太简单，需要增强能力------ 使用`hooks`

## 用 useState 实现 state 和 setState 功能

**让函数组件实现 state 和 setState**

- 默认函数组件没有`state`
- 函数组件是一个纯函数，执行完即销毁，无法存储`state`
- 需要`state hook`，即把`state`"钩"到纯函数中（保存到闭包中）

**hooks 命名规范**

- 规定所有的`hooks`都要以`use`开头，如`useXX`
- 自定义`hook`也要以`use`开头

```js
  // 使用 hooks
  import React, { useState } from 'react'

  function ClickCounter() { // 数组的解构 // useState 就是一个 Hook "钩"，最基本的一个 Hook const [count, setCount] = useState(0) // 传入一个初始值

        const [name, setName] = useState('test')

        // const arr = useState(0)
        // const count = arr[0]
        // const setCount = arr[1]

        function clickHandler() {
            setCount(count + 1)
            setName(name + '2020')
        }

        return <div>
            <p>你点击了 {count} 次 {name}</p>
            <button onClick={clickHandler}>点击</button>
        </div>

  }

  export default ClickCounter


  // 使用 class

  import React from 'react'

  class ClickCounter extends React.Component { constructor() { super()

            // 定义 state
            this.state = {
                count: 0,
                name: 'test'
            }
        }
        render() {
            return <div>
                <p>你点击了 {this.state.count} 次 {this.state.name}</p>
                <button onClick={this.clickHandler}>点击</button>
            </div>
        }
        clickHandler = ()=> {
            // 修改 state
            this.setState({
                count: this.state.count + 1,
                name: this.state.name + '2020'
            })
        }

  }

  export default ClickCounter
```

## 用 useEffect 模拟组件生命周期

**让函数组件模拟生命周期**

- 默认函数组件没有生命周期
- 函数组件是一个纯函数，执行完即销毁，自己无法实现生命周期
- 使用`Effect Hook`把生命周期"钩"到纯函数中

**useEffect 让纯函数有了副作用**

- 默认情况下，执行纯函数，输入参数，返回结果，无副作用
- 所谓副作用，就是对函数之外造成影响，如设置全局定时器
- 而组件需要副作用，所以需要有`useEffect`钩到纯函数中

**总结**

- 模拟`componentDidMount`，`useEffect`依赖`[]`
- 模拟`componentDidUpdate`，`useEffect`依赖`[a,b]`或者`useEffect(fn)`没有写第二个参数
- 模拟`componentWillUnmount`，`useEffect`返回一个函数
- 注意`useEffect(fn)`没有写第二个参数：同时模拟`componentDidMount` + `componentDidUpdate`

```js
  import React, { useState, useEffect } from 'react'

  function LifeCycles() { const [count, setCount] = useState(0) const [name, setName] = useState('test')

        // // 模拟 class 组件的 DidMount 和 DidUpdate
        // useEffect(() => {
        // console.log('在此发送一个 ajax 请求')
        // })

        // // 模拟 class 组件的 DidMount
        // useEffect(() => {
        // console.log('加载完了')
        // }, []) // 第二个参数是 [] （不依赖于任何 state）

        // // 模拟 class 组件的 DidUpdate
        // useEffect(() => {
        // console.log('更新了')
        // }, [count, name]) // 第二个参数就是依赖的 state

        // 模拟 class 组件的 DidMount
        useEffect(() => {
            let timerId = window.setInterval(() => {
                console.log(Date.now())
            }, 1000)

            // 返回一个函数
            // 模拟 WillUnMount
            return () => {
                window.clearInterval(timerId)
            }
        }, [])

        function clickHandler() {
            setCount(count + 1)
            setName(name + '2020')
        }

        return <div>
            <p>你点击了 {count} 次 {name}</p>
            <button onClick={clickHandler}>点击</button>
        </div>

  }

  export default LifeCycles
```

## 用 useEffect 模拟 WillUnMount 时的注意事项

**useEffect 中返回函数**

- `useEffect`依赖项`[]`，组件销毁时执行`fn`，等于`willUnmount`
- `useEffect`第二个参数没有或依赖项`[a,b]`，组件更新时执行`fn`，即下次执行`useEffect`之前，就会执行`fn`，无论更新或卸载（`props`更新会导致`willUnmount`多次执行）

```js
  import React from 'react'

  class FriendStatus extends React.Component {
  constructor(props) {
  super(props)
  this.state = { status: false // 默认当前不在线
  }
  }
  render() {
   return <div> 好友 {this.props.friendId} 在线状态：{this.state.status} </div> }
   componentDidMount() {
   console.log(`开始监听 ${this.props.friendId} 的在线状态`)
   }
    componentWillUnMount() {
    console.log(`结束监听 ${this.props.friendId} 的在线状态`) } // friendId 更新
     componentDidUpdate(prevProps) {
     console.log(`结束监听 ${prevProps.friendId} 在线状态`)
     console.log(`开始监听 ${this.props.friendId} 在线状态`)
   }
   }

  export default FriendStatus

  import React, { useState, useEffect } from 'react'

  function FriendStatus({ friendId }) { const [status, setStatus] = useState(false)

        // DidMount 和 DidUpdate
        useEffect(() => {
            console.log(`开始监听 ${friendId} 在线状态`)

            // 【特别注意】
            // 此处并不完全等同于 WillUnMount
            // props 发生变化，即更新，也会执行结束监听
            // 准确的说：返回的函数，会在下一次 effect 执行之前，被执行
            return () => {
                console.log(`结束监听 ${friendId} 在线状态`)
            }
        })

        return <div>
            好友 {friendId} 在线状态：{status.toString()}
        </div>

  }

  export default FriendStatus
```

## useRef 和 useContext

**1. useRef**

```js
import React, { useRef, useEffect } from "react";

function UseRef() {
  const btnRef = useRef(null); // 初始值

  // const numRef = useRef(0)
  // numRef.current

  useEffect(() => {
    console.log(btnRef.current); // DOM 节点
  }, []);

  return (
    <div>
      <button ref={btnRef}>click</button>
    </div>
  );
}

export default UseRef;
```

**2. useContext**

```js
import React, { useContext } from "react";

// 主题颜色
const themes = {
  light: {
    foreground: "#000",
    background: "#eee"
  },
  dark: {
    foreground: "#fff",
    background: "#222"
  }
};

// 创建 Context
const ThemeContext = React.createContext(themes.light); // 初始值

function ThemeButton() {
  const theme = useContext(ThemeContext);

  return <button style={{ background: theme.background, color: theme.foreground }}>hello world</button>;
}

function Toolbar() {
  return (
    <div>
      <ThemeButton></ThemeButton>
    </div>
  );
}

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar></Toolbar>
    </ThemeContext.Provider>
  );
}

export default App;
```

## useReducer 能代替 redux 吗

- `useReducer`是`useState`的代替方案，用于`state`复杂变化
- `useReducer`是单个组件状态管理，组件通讯还需要`props`
- `redux`是全局的状态管理，多组件共享数据

```js
import React, { useReducer } from "react";

const initialState = { count: 0 };

const reducer = (state, action) => {
  switch (action.type) {
    case "increment":
      return {
        count: state.count + 1
      };
    case "decrement":
      return {
        count: state.count - 1
      };
    default:
      return state;
  }
};

function App() {
  // 很像 const [count, setCount] = useState(0) const [state, dispatch] = useReducer(reducer, initialState)

  return (
    <div>
      count: {state.count}
      <button onClick={() => dispatch({ type: "increment" })}>increment</button>
      <button onClick={() => dispatch({ type: "decrement" })}>decrement</button>
    </div>
  );
}

export default App;
```

## 使用 useMemo 做性能优化

- 状态变化，React 会默认更新所有子组件
- `class`组件使用`shouldComponentUpdate`和`PureComponent`优化
- `Hooks`中使用`useMemo`缓存对象，避免子组件更新
- `useMemo`需要配合`React.memo`使用才生效

```js
  import React, { useState, memo, useMemo } from 'react'

  // 子组件
  // function Child({ userInfo }) {
  // console.log('Child render...', userInfo)

  // return <div>
  // <p>This is Child {userInfo.name} {userInfo.age}</p>
  // </div>
  // }
  // 类似 class PureComponent ，对 props 进行浅层比较 const Child = memo(({ userInfo }) => { console.log('Child render...', userInfo)

        return <div>
            <p>This is Child {userInfo.name} {userInfo.age}</p>
        </div>

  })

  // 父组件 function App() { console.log('Parent render...')

        const [count, setCount] = useState(0)
        const [name, setName] = useState('test')

        // const userInfo = { name, age: 20 }
        // 用 useMemo 缓存数据，有依赖
        // useMemo包裹后返回的对象是同一个，没有创建新的对象地址，不会触发子组件的重新渲染
        const userInfo = useMemo(() => {
            return { name, age: 21 }
        }, [name])

        return <div>
            <p>
                count is {count}
                <button onClick={() => setCount(count + 1)}>click</button>
            </p>
            <Child userInfo={userInfo}></Child>
        </div>

  }

  export default App
```

## 使用 useCallback 做性能优化

- `Hooks`中使用`useCallback`缓存函数，避免子组件更新
- `useCallback`需要配合`React.memo`使用才生效

```js
  import React, { useState, memo, useMemo, useCallback } from 'react'

  // 子组件，memo 相当于 PureComponent const Child = memo(({ userInfo, onChange }) => { console.log('Child render...', userInfo)

        return <div>
            <p>This is Child {userInfo.name} {userInfo.age}</p>
            <input onChange={onChange}></input>
        </div>

  })

  // 父组件 function App() { console.log('Parent render...')

        const [count, setCount] = useState(0)
        const [name, setName] = useState('test')

        // 用 useMemo 缓存数据
        const userInfo = useMemo(() => {
            return { name, age: 21 }
        }, [name])

        // function onChange(e) {
        // console.log(e.target.value)
        // }
        // 用 useCallback 缓存函数，避免在组件多次渲染中多次创建函数导致引用地址不一致
        const onChange = useCallback(e => {
            console.log(e.target.value)
        }, [])

        return <div>
            <p>
                count is {count}
                <button onClick={() => setCount(count + 1)}>click</button>
            </p>
            <Child userInfo={userInfo} onChange={onChange}></Child>
        </div>

  }

  export default App
```

## 什么是自定义 Hook

- 封装通用的功能
- 开发和使用第三方`Hooks`
- 自定义`Hooks`带来无限的拓展性，解耦代码

```js
  import { useState, useEffect } from 'react' import axios from 'axios'

  // 封装 axios 发送网络请求的自定义 Hook function useAxios(url) { const [loading, setLoading] = useState(false) const [data, setData] = useState() const [error, setError] = useState()

        useEffect(() => {
            // 利用 axios 发送网络请求
            setLoading(true)
            axios.get(url) // 发送一个 get 请求
                .then(res => setData(res))
                .catch(err => setError(err))
                .finally(() => setLoading(false))
        }, [url])

        return [loading, data, error]

  }

  export default useAxios

  // 第三方 Hook // https://nikgraf.github.io/react-hooks/ // https://github.com/umijs/hooks

  import { useState, useEffect } from 'react'

  function useMousePosition() { const [x, setX] = useState(0) const [y, setY] = useState(0)

        useEffect(() => {
            function mouseMoveHandler(event) {
                setX(event.clientX)
                setY(event.clientY)
            }

            // 绑定事件
            document.body.addEventListener('mousemove', mouseMoveHandler)

            // 解绑事件
            return () => document.body.removeEventListener('mousemove', mouseMoveHandler)
        }, [])

        return [x, y]

  }

  export default useMousePosition

  // 使用 function App() { const url = 'http://localhost:3000/' // 数组解构 const [loading, data, error] = useAxios(url)

        if (loading) return <div>loading...</div>

        return error
            ? <div>{JSON.stringify(error)}</div>
            : <div>{JSON.stringify(data)}</div>

        // const [x, y] = useMousePosition()
        // return <div style={{ height: '500px', backgroundColor: '#ccc' }}>
        // <p>鼠标位置 {x} {y}</p>
        // </div>

  }
```

## 使用 Hooks 的两条重要规则

- 只能用于函数组件和自定义`Hook`中，其他地方不可以
- 只能用于顶层代码，不能在判断、循环中使用`Hooks`
- `eslint`插件`eslint-plugin-react-hooks`可以帮助检查`Hooks`的使用规则

![565c97217253afa34c4566a9c42f18b8](_media/565c97217253afa34c4566a9c42f18b8.png)

## 为何 Hooks 要依赖于调用顺序

- 无论是`render`还是`re-render`，`Hooks`调用顺序必须一致
- 如果`Hooks`出现在循环、判断里，则无法保证顺序一致
- `Hooks`严重依赖调用顺序

```js
import React, { useState, useEffect } from "react";

function Teach({ couseName }) {
  // 函数组件，纯函数，执行完即销毁
  // 所以，无论组件初始化（render）还是组件更新（re-render）
  // 都会重新执行一次这个函数，获取最新的组件
  // 这一点和 class 组件不一样：有组件实例，组件实例一旦声声明不会销毁（除非组件销毁）

  // render: 初始化 state 的值 '张三'
  // re-render: 读取 state 的值 '张三'
  const [studentName, setStudentName] = useState("张三");

  // if (couseName) {
  // const [studentName, setStudentName] = useState('张三')
  // }

  // render: 初始化 state 的值 'poetry'
  // re-render: 读取 state 的值 'poetry'
  const [teacherName, setTeacherName] = useState("poetry");

  // if (couseName) {
  // useEffect(() => {
  // // 模拟学生签到
  // localStorage.setItem('name', studentName)
  // })
  // }

  // render: 添加 effect 函数
  // re-render: 替换 effect 函数（内部的函数也会重新定义）
  useEffect(() => {
    // 内部函数执行完就销毁
    // 模拟学生签到
    localStorage.setItem("name", studentName);
  });

  // render: 添加 effect 函数
  // re-render: 替换 effect 函数（内部的函数也会重新定义）
  useEffect(() => {
    // 内部函数执行完就销毁
    // 模拟开始上课
    console.log(`${teacherName} 开始上课，学生 ${studentName}`);
  });

  return (
    <div>
      课程：{couseName}， 讲师：{teacherName}， 学生：{studentName}
    </div>
  );
}

export default Teach;
```

## class 组件逻辑复用有哪些问题

- **高级组件 HOC**
  - 组件嵌套层级过多，不易于渲染、调试
  - `HOC`会劫持`props`，必须严格规范
- **Render Props**
  - 学习成本高，不利于理解
  - 只能传递纯函数，而默认情况下纯函数功能有限

## Hooks 组件逻辑复用有哪些好处

- 变量作用域很明确
- 不会产生组件嵌套

## Hooks 使用中的几个注意事项

- `useState`初始化值，只有第一次有效
- `useEffect`内部不能修改`state`，第二个参数需要是空的依赖`[]`
- `useEffect`可能出现死循环，依赖`[]`里面有对象、数组等引用类型，把引用类型拆解为值类型

  // 第一个坑：`useState`初始化值，只有第一次有效 import React, { useState } from 'react'

  // 子组件 function Child({ userInfo }) { // render: 初始化 state // re-render: 只恢复初始化的 state 值，不会再重新设置新的值 // 只能用 setName 修改 const [ name, setName ] = useState(userInfo.name)

        return <div>
            <p>Child, props name: {userInfo.name}</p>
            <p>Child, state name: {name}</p>
        </div>

  }

```js
  function App() { const [name, setName] = useState('test') const userInfo = { name }

        return <div>
            <div>
                Parent &nbsp;
                <button onClick={() => setName('test1')}>setName</button>
            </div>
            <Child userInfo={userInfo}/>
        </div>

  }

  export default App

  // 第二个坑：`useEffect`内部不能修改`state` import React, { useState, useRef, useEffect } from 'react'

  function UseEffectChangeState() { const [count, setCount] = useState(0)

        // 模拟 DidMount
        const countRef = useRef(0)
        useEffect(() => {
            console.log('useEffect...', count)

            // 定时任务
            const timer = setInterval(() => {
                console.log('setInterval...', countRef.current) // 一直是0 闭包陷阱
                // setCount(count + 1)
                setCount(++countRef.current) // 解决方案使用useRef
            }, 1000)

            // 清除定时任务
            return () => clearTimeout(timer)
        }, []) // 依赖为 []

        // 依赖为 [] 时： re-render 不会重新执行 effect 函数
        // 没有依赖：re-render 会重新执行 effect 函数

        return <div>count: {count}</div>

  }

  export default UseEffectChangeState
```
