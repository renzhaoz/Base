# Web Worker

## 一.概念

  JavaScript 语言采用的是单线程模型，也就是说，所有任务只能在一个线程上完成，一次只能做一件事.

  Web Worker 的作用，就是为 JavaScript 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行。在主线程运行的同时，Worker 线程在后台运行，两者互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程。这样的好处是，一些计算密集型或高延迟的任务，被 Worker 线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。

## 二. 特点

- 同源限制
  让Worker执行的脚本必须与主线程同源。

- 不能获取DOM
  无法访问DOM对象，例如```document,window,parent```.但是```navigator，location除外```.

- Worker与主线程之前的通信只能通过message传递.发布订阅者模式.

- Worker不能执行alert,confirm.

- Worker可以发送ajax请求.

- 脚本源限制,不能通过相对绝对路径访问,只能通过网络.

## 三.基本API

- 初始化

        worker = new Worker('http://www/a.com/w.js');

- 发送消息到worker,值可以是任何js类型

        worker.postMessage({methods: 'clear'});

- worker接受主线程发送的消息

        self.addEventListener('message', e => {
          console.log('Get msg from web page', e.data);
        });

        self.onmessage = e => {
          console.log('Get msg from web page', e.data);
        });

- 接受worker返回的消息

        worker.onmessage = e => {
          console.log('Get new msg', e.data);
        }

- 关闭worker

        // 主线程关闭
        worker.terminate();

        // worker内部关闭
        self.close();

- 在worker中引入外部脚本

        self.importScritps('caculate.js');

- 错误处理，可以在主线程监听worker是否有错误。

        worker.onerror(err => {console.log(err)});
        worker.adEventListener('error', err => {console.log(err)})

- 二级制文件的传递
主线程和worker之间的数据传递方式是拷贝传递，各自修改互不影响。但是拷贝方式传递较大二进制文件会有性能问题，如果文件较大拷贝耗时就越久。javascritp允许直接把二进制文件直接传递给worker处理,这样主线程会无法使用这些二进制文件，别的worker也不能在使用。这种方式叫```Transferable Objects```

        // Transferable Objects 格式
        worker.postMessage(arrayBuffer, [arrayBuffer]);

        // 例子
        var ab = new ArrayBuffer(1);
        worker.postMessage(ab, [ab]);

## 五.完整DEMO

## 六.小结

参考