# Map WeakMap Set WeakSet快速起步

## 1.Map

### Map概念

  键值对的集合，相当于```Object```的拓展，但又各自有优缺。

### Map起步API

- 初始化

      const map1 = new Map([['key1', 'value1']])
      const map2 = new Map({
        [Symbol.iterator]: function*(){
          yield ["key1", "val1"];
          yield ["key2", "val2"];
        }
      });
      const map3 = new Map(map1);
- ```set(key, value)``` 添加键/值

        map1.set('key', 'val').set('key1', 'value1');
- ```has(key)``` 查询属性
- ```get(key)``` 查询属性

        map1.has(key)/map1.get(key)
- ```size``` 返回map键值对的数量

        map1.size()
- ```delete(key)``` 删除
- ```clear()``` 清理
- ```entries()``` 返回Map集合的键值对数组,即[...map]
- ```keys()``` 返回keys插入顺序的可迭代集合
- ```values()``` 返回集合值的可迭代集合可用```for...of...```循环类数组集合，循环中对集合的修改不影响集合内部数据

### Map特点指南

- key可以是任何javascript数据类型。obj只能是```数值```/```字符串```/或```符号```
- Map内部相等的判断使用SameValueZero(ES6内部定义)来比较,类似===。[MDN文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)
- Map内部集合用作key或者value的引用值在发生改变时Map中的值不会被修改
- Map会维护键值对插入的顺序, value值的覆盖仍不影响顺序

### Map性能

这里与```Object```做比较:

- 内存

  由于不同浏览器对Map的实现存在明显分别，特别是内存分配上。键值对的新增删除的内存分配都是底层浏览器实现的。但如果内存大小固定，```Map```大约可以比```Object```多存储50%的键值对。若是value值是连续整数则有底层优化，```Object```内存占据会更少。

- 插入性能
  ```Object```和```Map```中插入新键值对的性能```Map```能稍微快一点，插入速度不会随集合数量的增多而线性增加。如果频繁插入新集合那么Map更快。

- 查找速度
  如果集合数量较少则```Object```更快。查找速度不会随集合数量的增多而线性增加。若涉及大量查找操作```Object```速度会更快(频繁使用指针的底层优化)。

- 删除
  Map性能会更好。```Object```的```delete```一直以来都有性能问题，只能中断引用才释放内存/手动设置```null```等缺点

## 2.WeakMap

### WeakMap概念

  '弱映射'是一种新的集合类型。与Map相比有增强的存储机制，所有会有更好的内存性能。

### WeakMap起步API

  WeakMap是Map的子集所以API也是通用的。详见Map起步API。

### WeakMap特点指南

- ```WeakMap``` 的key只能是object或者继承自object的类型。否则会
  TypeError，值的类型没有限制。

- 原始值可以包装成对象再用作键，例如使用```new String```/```Number```转化。

- weak弱指的是不属于正式的引用，不会阻止垃圾回收，只要键还在则仍存在于集合中，若键被删除或者清理回收则集合中的键值对会被清理。例如被作为key的DOM对象被从DOM树中删除，则集合中的键值对会被删除。

- WeakMap无法被迭代，其内容也无法被查看。

## 3.Set

### Set概念

  Set与Map类似是一种新的数据结构集合类型，set在很多方面就像是加强的Map,大多数API都是一样的。

### Set起步API

  Map的API大多数是和Map相似的。这里只做不同点。

- 初始化

        let set = new Set(['s1','s2']);
        let set = new Set({
          [Symbol.iterator]: function*(){
            yeild 's1';
            yeild 's2';
          }
        })

- ```add()``` 增加新值，可链式调用add方法，返回实例本神
- ```delete()``` 删除某个集合，返回布尔值以确定是否存在于集合中
- ```values()/keys()```返回集合的迭代器，可使用for...of循环，其中values是默认迭代器，所以可以直接对实例进行拓展操作即(```[...new Set()]```)，返回实例的数组类型。
- ```entries()```返回一个迭代器，循环时返回重复集合项的数组```['s1','s1']```

### Set特点指南

- Set可以存储任何JavaScript数据类型
- Set的内部取等也是采用```SameValueZero```
- Set内部的集合对象在本身发生改变时不影响集合内容
- Set会维护值插入时的顺序，迭代时会按顺序执行
- Set常常用于某些集合间关系运算，所以常常被增强使用，例如计算两个集合之间的交集，差集，并集，对称差集等。具体实现可见文末Demo。

## 4.WeakSet

### WeakSet概念

  WeakSet是一种新的集合类型是Set的兄弟类型，其API也是Set的子集.这里的Weak也是指的是JavaScript垃圾回收程序对弱集合中值的处理方式。

### WeakSet起步API
  
  详见```Set```及```Map```的API简介，基本一致。

### WeakSet特点指南

- WeakSet中的集合项都必须是对象。
- WeakSet中的对象若没有引用就被会被垃圾回收机制清理
- WeakSet集合中的对象本身若被手动赋值```null```则也会被清理与WeakMap一致。例当执行```removeReference```时集合会被清理。

        const ws = new WeakSet();
        const container = {
          val: {}
        };
        ws.add(container.val);
        function removeReference() {
          container.val = null;
        }

- WeakSet中的值不可迭代
- WeakSeet用于给标签对象打标签，比如收集被禁用的Button，通过查询禁用的Button在不在WeakSet中可获得所有存在于DOM树中的所有禁用的Button。

## 5.迭代与拓展操作

  对象本地定义了自己的默认迭代器的对象都可以使用拓展操作，拓展操作对于集合之间的复制修改变得更容易。在JavaScript中有四种集合类型实现了默认迭代器,```Array```/定型数组(例```Int16Array```)/```Map```/```Set```。

  实现了默认迭代器的集合类型都可以使用```for...of```进行迭代循环，取出子集。也就意味着可以使用拓展运算符```...```

### 拓展操作起步API

- 浅复制数组对象
        let arr1 = [1]
        let arr2 = [...arr1]

- 数组插入

        let arr1 = [1,2,3]
        let arr2 = ['s', ...arr1, 'e']

- of()/from()数组方法

        Array.from(set1/map1) // [1,2,3] 可以将集合转化为数组。
        Array.of('item1','item2') // [item1, item2] 可以将对象集合为数组

- 把集合转变为数组

        [...set1] // [1, 2, 3]

## 6.小结
