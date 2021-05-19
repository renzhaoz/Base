# History

## 概念

History对象记录用户首次使用以来的所有导航历史记录.用于前进或者后退以便查阅浏览记录。

## 特点

- History是window的属性,每个window都有自己的history对象.

- History不能暴露用户的具体URL只能操作history的前进或者后退.

- 修改浏览器Url会导致浏览器重刷，hash的修改则不会

- 单页面应用常会监听hashchange用于局部刷新界面部分数据

- ```history.pushState```操作也不会重刷页面,也可用于单页面应用刷新局部数据,```pushState```方法的第一个参数有大小限制500KB~1M,且只能是可以序列化的数据.第三个参数要确保该地址在服务端存在，否则刷新404.

## 起步API

- ```history.go(-1)``` 后退一页

- ```history.go(1)``` 前进一页

- ```history.go("a.com")``` 旧版本浏览器会导航到最近的该页面

- ```history.back()``` 后退
- ```history.forward()``` 前进

- ```history.pushState({state},'title','new.html')``` 传入的参数state可以通过history.state访问,初次进入页面时state的值为null

- ```history.popState()``` 回退状态和页面,可以被监听```window.addEventListener('popstate', ()=>{})```

- ```history.replaceState``` 和pushState的参数相同用于更新state值

## 总结

  history 对象提供了操纵浏览器历史记录的能力,开发者可以确
  定历史记录中包含多少个条目,并以编程方式实现在历史记录中导航,而且也可以修改历史记录。

## 拓展

获取url传递的拼接到Url参数信息:

```
  let getQueryStringArgs = function() {

    // 取得没有开头问号的查询字符串
    let qs = (location.search.length > 0 ? location.search.substring(1) : ""),

    // 保存数据的对象
    args = {};

    // 把每个参数添加到 args 对象
    for (let item of qs.split("&").map(kv => kv.split("="))) {
      let name = decodeURIComponent(item[0]),
      value = decodeURIComponent(item[1]);
      if (name.length) {
        args[name] = value;
      }
    }

    return args;
  }
```