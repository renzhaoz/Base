# CommonJs requireJs ES6的区别

## 简言

 所有的JavaScript模块化实现方案都是为了模拟模块化开发模式.
 'CMD'属于CommonJS的一种规范,但并未广泛使用(国内大佬)仅仅SeaJS.这里不做讨论.

## CommonJs

  同步加载,只执行一次,缓存执行结果.
  node应用的实现采用了CommonJs规范,基本只存在node应用中.

- 实例

```
  var x = 5;
  var addX = function (value) {
    return value + x;
  };
  module.exports.x = x;
  module.exports.addX = addX;

  // 引入
  var example = require('./example.js');

  console.log(example.x); // 5
  console.log(example.addX(1)); // 6
```

- 特点
  1. 所有代码都运行在模块作用域，不会污染全局作用域。
  2. 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
  3. 模块加载的顺序，按照其在代码中出现的顺序。

- API
  1. module对象
  - node内部提供一个Module的构建函数，所有的模块都是Module的实例.可以直接在模块中打印```module```对象

  ```
    function Module(id, parent) {
      this.id = id; // 模块的标识符 一般是带有绝对路径的文件名
      this.filename = string; // 带有绝对路径的文件名
      this.loaded = Fun; // 返回布尔值 模块是否加载完毕
      this.exports = {}; // 标识模块对外输出的值
      this.parent = Object; // 调用该模块的模块
      this.children = Array // 标识该模块要用到的其它模块
      ...
    }
  ```


## requireJS

 非同步加载模块,有指定回调函数(即AMD规范).可以兼容CommonJs模块方案.
 主要有两个Javascript库实现了AMD规范：require.js和curl.js.

- 实例

  ```
    // 纯碎AMD规范
    define(['lib'], function(lib){
      function foo(){
        lib.log('hello world!');
      }

      return {
        foo: foo
      };
    });

    // 兼容CommonJs规范
    define(function (require, exports, module){
      var someModule = require("someModule");
      var anotherModule = require("anotherModule");

      someModule.doTehAwesome();
      anotherModule.doMoarAwesome();

      exports.asplode = function (){
        someModule.doTehAwesome();
        anotherModule.doMoarAwesome();
      };
    });

    // 引入 
    require([module], callback); // 注意这里的引入也使用require 但是有两个参数
  ```




```

AMD 依赖前置，提前执行 (require.js)，语法是define，require
CMD 依赖就近，延迟执行 （sea.js），语法是 define，seajs.use([],cb)
CommonJs 语法 module.exports=fn或者exports.a=1; 通过require('./a1')来引入
CommonJs 模块首次执行会被缓存，再次加载只返回缓存结果，require返回的值是输出值的拷贝（对于引用类型是浅拷贝）
es6 module 语法是 export {...}, import ...from..., export输出的是值得引用。
NodeJS、webpack都是基于commonJs该规范来实现的
```

## ES6

ECMAScript为了实现模块化开发提出的一种解决方案规范.在未来将替代哪些模块化方案.

- 简介

  JavaScript由于不能像Java Python等那些后台语言进行模块化开发.在越来越复杂的前端业务场景下,运维 整理等工作越来越复杂. ECMAScript开始在版本6.0提出模块发方案规范.

- 实例

```
  // 方式1
  export var demo1 = 123;
  import { demo1 as aa } from 'path'; // 引入时重命名

  // 方法2
  let modules1 = 1;
  let modules2 = 2;
  export { modules1 as aa, modules2 as bb }; // 导出时重命名

  import { aa, bb } from 'path';
  import * as mos from 'path'; // 导入所有的模块引用到mos对象上 mos.modules1 mos.modules2

  // 方法3
  import modules from 'path';
  import { a, b, c} from 'path';

  // 方法4
  export { modules as aa } from 'path'; // 导入导出汇合 一行代码实现
  export {modules as default} from 'path'; // 导出默认值 注意default

  // 方法5
  let modules = 'modules';
  export default modules; // default 设置默认导出值

  import aa from 'path'; // aa === 'modules';

```

- API

- 拓展

 1. import()

  区别与import...from...指令的今天执行,代码只能出现在代码顶层，不可以出现在块级作用域中. 这导致改方式引入模块的方式不能在js运行中执行(类似AMD缺点).于是import方法出现了,类似requireJS.但是import方法实现了异步加载(webpack会自动代码分割出使用了import方法引入的文件)也是依赖引入的实现.

  ```
    import('path').then(()=>{ 
      // to do
    })
  ```

 2. import 'path'
  不使用import...from...,紧跟路径. 这样的引入方法只会运行路径文件内容不导入任何模块内容

  3. 不同模块之间的互相引用

  - ES 引入 CommonJS模块
  ```
    // CommonJS模块
    module.exports = {aa: 'aaa'}; // 等同于 export default {aa};

    import { aa } from 'commonJSPath'; // aa: 'aaa'
    import * as common from 'path'; // 当导出多模块时 尽量使用这种方式导入CommonJS模块, CommonJS是运行时确定输出接口
  ```

  - RequireJS 导入 ES6模块

  ```
    // ES6模块
    const aa = 'aa';
    export default aa;

    // require引入

    const module = require('path'); // aa === module.default;
  ```


## 总结

1. CommonJS输入的是一个值的复制.
2. CommonJS会缓存输出值,代码只执行一次,多次调用永远只输出第一次的运算值.
3. 当CommonJS a 和 b模块存在互相引用时.modulesA引入a时的执行逻辑如下：

  ```
    // a.js
    exports.done = false; // 1
    var b = require('./b.js'); // 2
    console.log('在 a.js 之中，b.done = %j', b.done); // 5
    exports.done = true; //6
    console.log('a.js 执行完毕'); // 7

    // b.js
    exports.done = false; // 3
    var a = require('./a.js'); // 4 这时a.done === false
    console.log('在 b.js 之中，a.done = %j', a.done); // 8
    exports.done = true; // 9
    console.log('b.js 执行完毕'); // 10
  ```

  modulesA同时引入a和b时逻辑如下:
  ```
    // modulesA
    const a = require('a.js'); // 完整执行a.js 和 b.js
    const b = require('b.js'); // 不运行任何代码 直接读取b.js运行完毕时exports的值. 即b.done === true;
  ```

4. ES6输出的是值的引用.ES6的运行机制和CommonJS不一致,当执行到加载模块import时不会去执行模块,仅仅生成一个引用,等真的需要时再到模块里去取值(Webpack tree shaking).

5. ES6模块的循环引用,如果在ModulesA中引入a.js.则执行步骤如下:

  ```
    // a.js
    import b from 'b.js'; // 1
    console.log(b); // 5
    export default b; // 6

    // b.js
    import a from 'a.js'; // 2
    console.log(a); // 3 & 这时a.js只运行了第一行,a === undefined

    export default a; // 4 & 此时a === undefined

    // ModulesA
    import a from 'a.js'; // 执行步骤如上
    console.log(a);
  ```
