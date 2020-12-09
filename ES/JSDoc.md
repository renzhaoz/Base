# JS注释规范

## 1.常用注释标签

以下是常用的JavaScript注释标签。

- ```@file``` 文件描述
- ```@author``` 作者
- ```@version``` 版本
- ```@date``` 日期
- ```@copyright``` 版权
- ```@event``` 事件
- ```@method``` 函数
- ```@listens``` 事件监听
- ```@module``` 模块
- ```@param``` 变量
- ```@returns``` 返回值
- ```@description``` 描述
- ```@example``` 例子

- ```@class``` 类
- ```@classdesc``` class描述
- ```@private``` 类的私有属性(类内声明的变量)
- ```@public``` 类的公有属性(赋值给this的值)
- ```@extends``` 类的继承
- ```@static``` 类内的静态方法

- ```@see``` 详见{@link http://example.com|some MDN}
- ```@todo``` 待未实现功能
- ```@type``` { Object } 描述一个变量的类型；
- ```@callback``` 回调
- ```@namespace``` 命名空间

        const obj = {
          name: ...，
          age: ...
        }

- ```@deprecated``` 已弃用的代码
- ```@constructor``` 构造器(es5)即class
- ```@async``` 异步函数
- ```@fires``` 可能会触发的事件
- ```@generator``` 迭代器
- ```@global``` 全局变量
- ```@override``` 重写
- ```@readonly``` 只读
- ```@static``` 静态属性
- ```@summary``` 简述类似desc
- ```@this``` 这里的this是指
- ```@tutorial``` 教程

## 2.完整Demo

        const shape = function () {
          /**
            * @private {normalSize}
            */
          const normalSize = 2;

          /**
            * @public {size}
            */ 
          this.size = 12 * normalSize;

        }

        const Shape = new shape();

        /**
          * @class Circle
          * @extends { Shape }
          */ 
        class Circle extends Shape{
          /**
            * Creates an instance of Circle.
            *
            * @constructor
            * @author: moi
            * @param {number} r The desired radius of the circle.
            */
          constructor(r) {
            /** @private */ this.radius = r;
            /** @private */ this.circumference = 2 * Math.PI * r;
          }

          /**
            * Creates a new Circle from a diameter.
            *
            * @param {number} d The desired diameter of the circle.
            * @return {Circle} The new Circle object.
            */
          static fromDiameter(d) {
            return new Circle(d / 2);
          }

          /**
            * Calculates the circumference of the Circle.
            *
            * @deprecated since 1.1.0; use getCircumference instead
            * @return {number} The circumference of the circle.
            */
          calculateCircumference() {
            return 2 * Math.PI * this.radius;
          }

          /**
            * Returns the pre-computed circumference of the Circle.
            * @return {number} The circumference of the circle.
            * @since 1.1.0
            */
          getCircumference() {
            return this.circumference;
          }

          /**
            * Find a String representation of the Circle.
            * @override
            * @return {string} Human-readable representation of this Circle.
            */
          toString() {
            return `[A Circle object with radius of ${this.radius}.]`;
          }
        }

        /**
          * Prints a circle.
          * @param {Circle} circle
          */
        function printCircle(circle) {
          /** @this {Circle} */
          function bound() {
            console.log(this);
          }
          bound.apply(circle);
        }
