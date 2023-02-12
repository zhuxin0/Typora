## 库和框架、包的区别

- 库，**库**（Library）是一系列预先编写好的代码集合
- 框架包含了多个库，是一套通用的解决方案，接管程序的主控制流

## 更新数据setState特点

- 对多个setState进行进行合并，一次性更新
- 需要传一个新的对象
- 传递对象和state进行数据对比，进行更新的时候默认将修改了值的内容进行合并页面更新

## 受控组件与非受控组件

- 表单组件的value和state中的数据绑定，只能修改state中值，表单的value才会改变
- 非受控组件指的就是表单组件没有跟state的状态绑定在一起。你可以直接通过表单节点获取对应的数据表单数据回显通过defaultValue实现的。

## 生命周期

- 挂载阶段
  - componentWillMount
  - componentDidMount
- 运行阶段
  - shouldComponentUpdate
  - componentWillUpdate
  - componentDidUpdate
- 销毁阶段
  - componentWillUnMount

- 

  

## 建立项目

- 官方方法

  ```javascript
  npx create-react-app projrct-one
  
  ```

- vite方法

    ```javascript
    npm create vite@latest
    输入项目名
    选择模式
    npm i 下载所有包
    ```

##   React中如何更新数据?

- 使用setSatte,setSatte会根据state地址是否变化来更新数据

  - ```react
     this.setState(
          {list: this.state.list,a: 3,},() => {console.log(this.state.a);}
        );//回调函数在数据更新后立即执行操作
    ```

- forceUpdata

  ```react
  //数据改变后
  this.forceUpdata()//相当于手动调用render函数
  ```

  

## React中数据更新策略是怎么样?

- React中的setState有Batch模式(批量更新模式)和普通模式。

- 普通模式下,setState能够即时更新state，重新调用 render 方法，然后把render方法所渲染的最新的内容显示到页面上。

- Batch模式下,React不会立刻修改state。而是把这个对象放到一个更新队列中，稍后才会从队列中把新的状态提取出来合并到 state中，然后再触发组件更新。

- 1.由 React 控制的事件处理过程 setState 不会同步更新 this.state。如我们使用React库中的表单组件，例如 select、input、button等，它都是处于React库的控制之下，因此setState会以异步的方式执行。
- 2.React 控制之外的情况， setState 会同步更新 this.state。通过JavaScript原生addEventListener直接添加的事件处理函数，使用setTimeout/setInterval 等setState会以同步的方式执行。
  

## 如果页面不用setState更新,你还有其他办法吗?

- forceUpdata

```react
//数据改变后
this.forceUpdata()//相当于手动调用render函数
```

