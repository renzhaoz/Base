# React基础理论

## 1. 事件SyntheticEvent

- 阻止事件不能使用return false,必须显式使用event.preventDefault || event.stopPropagation
,event是一个合成事件

- 获取浏览器事件可以使用nativeEvent访问

## 2. 生命周期

**初始化defaultProps.**
**错误处理:**

    static getDerivedStateFromError()
    componentDidCatch()

**执行组件的生命周期狗子函数.生命周期调用顺序如下:**

### React V16之前

----

#### 首次创建实例调用顺序

- constructor(props){} 接受props参数
- componentWillMount()
- render()
- componentDidMount()

#### 组件更新

- componentWillReceiveProps(nextProps)
- shouldComponentUpdate(nextProps, nextState)
- componentWillUpdate()
- render()
- componentDidUpdate()

### 组件卸载

- componentWillUnmount()

---

### React V16之后

---

#### 首次创建实例调用顺序

- constructor()
- static getDerivedStateFromProps(nextProps, prevState)
- render()
- componentDidMount()

#### 组件更新

- static getDerivedStateFromProps(nextProps, prevState)
- shouldComponentUpdate(nextProps, nextState)
- render()
- getSnapshotBeforeUpdate(prevProps, prevState) // 在最近一次渲染结果挂载到真实DOM树之前调用
  例获取滚动位置, 操作后的元素属性等

- componentDidUpdate(prevProps, prevState, snapshot) snapshot是getSnapshotBeforeUpdate钩子函数返回的值

### 组件卸载

- componentWillUnmount()

---

## 3.Fiber

问题起始： 复杂页面频繁刷新会掉帧。

问题原因： 大量的同步计算任务阻塞了浏览器的UI渲染.
  默认情况下js运行,页面布局,页面绘制都是运行与浏览器主线程中.js执行时别的就会被阻塞。当react更新组件时,react会遍历所有的节点,计算出差异然后再更新UI(具体请查看diff计算过程)。如果页面元素很多，js执行会超过16ms.页面会出现掉帧现象。

解题思路： 
借鉴EventLoop使用交替执行,js执行和其它浏览器工作交替限时执行.

难点：

- 跳出执行栈使用requestIdleCallback(),当前polyfill使用了requestAnimationFrame

  window.requestIdleCallback()会在浏览器空闲时期依次调用函数，这就可以让开发者在主事件循环中执行后台或低优先级的任务，而且不会对像动画和用户交互这些延迟触发但关键的事件产生影响。函数一般会按先进先调用的顺序执行，除非函数在浏览器调用它之前就到了它的超时时间。

- js运算分割 根据虚拟DOM树(组件实例)进行拆分,在每个节点上进行拆分逻辑,然后每次运行一个节点的逻辑代码(真实DOM渲染), 执行完成后判断当前是否有优先级更高的js需要执行或判断当前分配给该节点的执行时间剩余 若无优先级的js需要执行 或 剩余时间充足 则开始执行下一个代码片段 若时间不够则下一轮继续.

- 保留上一次运算结果
  若该轮时间内代码片段未执行完毕,且有优先级更高的任务需要执行则会退出当前执行片段.等到下一轮执行时会重新从头执行.则会出现某些组件的钩子函数重复执行:componentWillMount,componentWillUpdate,componentWillReceiveProps这些钩子函数都是在未挂载组件实例之前(协调diff之前)被调用的。所以这些函数都在17之后会慢慢的移除.

## 4. diff过程

在某一时间节点调用 React 的 render() 方法，会创建一棵由 React 元素组成的树。在下一次 state 或 props 更新时，相同的 render() 方法会返回一棵不同的树。React 需要基于这两棵树之间的差别来判断如何高效的更新 UI，以保证当前 UI 与最新的树保持同步。

具体步骤如下：

- 1.比较两个树的根节点,若节点类型变化则卸载重绘

  span改为div节点,内部当前节点及所有节点肯定会卸载然后重绘

- 2.比较根节点的属性,若属性有变动则只改变动的属性

- 3.比较当前根节点的子节点(详见1及2的步骤), 然后对所有节点进行步骤1、2、3的递归.

  当组件的props改变时,组件的实例会保持不变包括其state.然后依次调用componentWillReceiveProps、componentWillUpdate、componentDidUpdate、render.接下来组件实例仅仅会更改与新props相关的节点信息(通过diff算法).

***注意：***

  若子节点有同级兄弟节点.React会同时遍历新旧虚拟DOM树之间的差异.同级列表中末尾添加dom的性能优于头部新增(详见diff比较1、2、3).React在计算过程中会清理所有的列表然后重汇,因为它并不知道是否需要保存之前就存在的dom列表.keys就是为了解决这个问题.


## 5. HOOK