# Error错误类型

## 简介

JavaScript执行过程中会发生各种类型的错误。每种类型都会对应一个错误发生时抛出的错误对象。包含以下几种:

window错误:

- window.onerror

Javascript错误：

- Error 基础类型
- InternalError 浏览器底层错误(栈溢出)
- EvalError eval()函数执行异常时
- RangeError 数值越界时错误(new Array(-1))
- ReferenceError 目标对象不存在时报错(alert(xxx))
- SyntaxError 语法错误,给eval传入的字符串执行错误时
- TypeError 类型错误时错误(new 1)
- URIError encodeURL时和decodeURL方法执行时传入错误格式的URL

## 起步API

- 捕获错误

```
  try{
    //执行js
  }catch (error) {
    // 错误捕获
    // 错误类型可以instanceof判断
  }
```

- 抛出错误 throw

```
  throw 123;
  throw new Error('some things miss');
```
