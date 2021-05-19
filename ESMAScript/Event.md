# 事件

## 简介

远古时期,IE和 Netscape开发团队提出了完全相反的事件流方案。IE:事件冒泡流,NetscapeCommunicator:事件捕获流.

事件冒泡(IE)顺序: div -> body -> html -> document
事件捕获(NetscapeCommunicator): document -> html -> body -> div

DOM2规范重新定义了以下事件流,具体分为三个部分.事件捕获,目标,事件冒泡.(除ie8不支持其余浏览器都已支持)
事件捕获最先发生,为提前拦截提供可能,捕获顺序:
document -> html -> body

目标阶段:
body -> div(div不会接收到事件)

冒泡阶段:

div -> body -> html -> document

现在几乎所有的浏览器都支持事件捕获,虽然DOM2规范规定事件流从document开始,但浏览器基本都是从window对象开始.规范虽然明确表示捕获阶段不命中事件目标,但浏览器都会在捕获阶段在事件目标上触发事件回调.造成的结果就是会有两个机会处理事件流(即捕获阶段和冒泡阶段).

这里简答描述下DOM0和DOM2的区别：
- DOM0

dom.onclick = () => {};

- DOM2 可以添加多次监听
dom.addEventListener('click', () => {}, false); // 默认false在冒泡阶段调用事件回调, true表示在捕获阶段调用事件处理

## API

- event对象,触发事件回调时会返回event对象,描述事件属性.
- event.target 目标对象
- event.cancelable 是否可以取消事件的默认行为
- event.detail 事件信息
- event.eventPhase 返回事件阶段.1代表捕获阶段，2 代表
到达目标，3 代表冒泡阶段
- event.preventDefault() 用于取消事件的默认行为。只有 cancelable 为 true 才
可以调用这个方法
- event.stopImmediatePropagation() 用于取消所有后续事件捕获或事件冒泡，并阻止调用任何后续事件处理程序（DOM3 Events 中新增）
- event.stopPropagation() 用于取消所有后续事件捕获或事件冒泡。只有event.bubbles为true才可以调用这个方法
- event.trusted true浏览器生成,false自定义事件

## 拓展

1. 阻止事件的区别

- event.preventDefault() 阻止事件的默认行为(比如checkbox的选中行为),事件流还是会继续走,除非使用event.stopPropagation()或者stopImmediatePropagation().

- event.stopPropagation() 阻止事件流中捕获和冒泡阶段中当前事件的进一步传播.但是不会阻止默认行为的发生(比如连接的点击).

- event.stopImmediatePropagation() 阻止监听同一事件的其它监听器触发.如果有多个监听器被触发时第一个监听器调用该方法则后续的监听器不会被触发

- 在事件触发的回调中 ```return false;```相当于同时调用event.preventDefault()和event.stopPropagation()(addEventListener不能阻止,HTML5规范中有指出在mouseover等几种特殊事件情况下，return false;并不一定能终止事件.所以尽量不使用)

## 思考

- React事件的实现,如何阻止
- 合成事件
- 自定义事件
