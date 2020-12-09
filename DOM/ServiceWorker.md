# Service Worker入门
## 一.常见作用
1. 离线缓存。Server worker安装启动后对当前配置的文件进行缓存，界面刷新或者重进进入界面时只要server Worker未被注销则可从缓冲中读取命中的目标文件，以达到类applacationCache
2. 消息推送。推送消息到被server worker接收管理的界面。
3. 后台数据同步,实现类app的静默更新。
4. 请求劫持。
5. 地理围栏,时间日期的响应,相同域名之间的切换，处理运算成本比较高的数据，预取资源比如下一组列表数据等等。

## 二.概念及特点

  serviceWorker作为一个后台运行的服务长期运行与浏览器后台。除非手动暂停或者重新安装否则从下载安装完成后就一直运行。

- 安装了serviceWorker的页面会一直被serviceWorker控制；

- 不能访问操作DOM

- 独立运行，不阻塞主线程

- 只能在https或者loacalhost下使用

## 三.使用Service Worker

1. 注册

        navigator.serviceWorker.register('sw.js').then(() => {
          console.info('注册成功');
        }).catch((err) => {
          console.error('注册失败');
        })
    **注意：首次访问server worker控制的网站或者页面时，其会自动下载、安装、激活相关的文件和代码、为了不影响界面首次加载运行的cpu和网络占用建议推迟执行register方法，例如在load回调函数中执行。<br>一个项目可以注册多个serviceWorker但是需要注意必须指定不同的sw.js文件和不同的scope限制。<br>由于server worker出于安全考虑限定其只能在指定源下使用且只能在localhost和https下使用，所以注册文件一般放于和index.html同级目录下(生产环境非开发环境)**
  - 'sw.js'

    浏览器要执行的serviceWorker的脚本文件路径，sw.js的路径是相对与origin的路径不是本地的路径(vue&react等使用构建工具需要注意路径)。通常在该文件中进行消息分发、请求拦截、存储缓存等等。
  - scope参数

    scope规定了需要响应sw的域名范围，不设置即控制所有的页面。

2. 安装完成的生命周期
  sw执行register方法后sw就会自己安装运行，可以在sw.js文件中监听其是否安装成功。

        self.addEventListener('install', event => {
          event.waitUntil(() => console.info('安装完成的回调'))
        })

3. 激活完成的生命周期
  sw先下载后安装再执行都是自动的，和install类似，在sw.js中可以获取其激活的生命周期。

        self.addEventListener('activate', event => {
          event.waitUntil(() => console.info('激活完成的回调'))
        })

4. 更新

    - 在默认情况下Service Worker必定每24小时被下载一次，如果下载的文件是最新文件，那么就会被重新注册和安装，单不会被激活，当不再有页面使用旧的Service Worker时新的sw被激活，旧的sw被卸载，所以有可能出现两个sw。如果这是首次启用service worker，页面会首先尝试安装，安装成功后激活。
    - 浏览器调试工具选择Update on reload
    - 重新调用register方法

## 四.功能简介-通信

-  通信 页面发送消息到ServiceWorker

    从页面给已经注册激活的serviceWorker传递消息，要确定当前serviceWorker已经激活安装完成（可以判断navigator.serviceWorker.controller对象是否被构建）。具体如下代码所示：
    
        // app.js 发送
        if(navigator.serviceWorker.controller){
          navigator.serviceWorker.controller.postMessage("this message is from page");
        }

        // sw.js 接收
        self.addEventListener('message', function (event) {
            console.log(event.data); // this message is from page
        });
      
-  通信 ServiceWorker发送消息到页面

    1. 从serviceWorker到页面的信息传递最简单的方式是通过监听页面发送到serverWorker的消息然后获取windowClients，然后在windowClients上调用window的postMessage发送消息到页面，但是局限性也很大，必须要接收到页面发送到serviceWorker的消息后才能发送消息。例：


            // sw.js 接收到消息后再发送信息
            this.addEventListener('message', function (event) {
              event.source.postMessage('this message is from sw.js, to page');
            });

            // app.js 通过监听message消息接收信息
            navigator.serviceWorker.addEventListener('message', function (e) {
              console.log(e.data); // this message is from sw.js, to page
            });

    2. 直接通过client对象的postMessage方法发送消息。在sw.js文件中是可以通过serverWorker本身访问到所有的受其控制或者说注册了该serviceWoker的页面的clients对象(即windows对象)，然后直接发送消息。例：

            // sw.js 发送 接收如上1类似
            clients.matchAll().then(client => {
                client[0].postMessage('this message is from sw.js, to page');
            })

      ***以上就是使用serviceWorker的通信部分，当然目前看来所有的通信过程都与postMessage方法类似。那么纵向对比也许浏览器的广播系统也可能是一个更好的方式去实现通信，比如可以在多个serviceWorker之间实现通信和消息推送。有兴趣的话可以看下MessageChannel或者BroadcastChannel的API。***


## 五.功能简介-资源缓存
这个功能可以说是浏览器实现或者说兼容该api的重要原因即静态资源缓存。一般情况下打开一个网页浏览器会自动download所有的资源文件包括但不限于js/css/img/html等。受限于网络状态和网络能力能即使本地客户端很好有时也会出现很差的用户体验。serviceWorker可以实现即使在offline的情况下也能正常展示当前页面。sw安装完成之后可以缓存指定静态资源，当页面再次刷新或者重新打开页面sw未被重新安装或者删除时，页面首先会在其cacheSotre中读取其静态资源减少资源请求。

- 缓存指定资源

        // sw.js 缓存指定静态资源
        this.addEventListener('install', function (event) {
          console.log('install');
          event.waitUntil(caches.open('sw_demo').then(function (cache) {
            return cache.addAll(['/style.css','/panda.jpg','./main.js'])
          }));
        });

    CacheStroage在浏览器中的接口名称是caches，我们可以通过caches.open方法新建或者打开一个已存在的缓存。Cache.addAll方法的作用是请求指定链接页面的资源把他们储存到之前打开的缓存中（具体操作详解请查看CacheStroage API）。由于IO操作是异步操作锦衣将方法放置于event.waitUntil方法，其能保证资源被缓存完成前sw本身不被重新安装或者其余操作导致终止或者退出。缓存完成后可以在Chrome开发工具中的Application的Cache Strogae中可以看到我们缓存的资源。

- 缓存所有网络资源
    缓存所有网络资源是通过监听fetch事件，当用户发起网络请求时会被触发。但是需要注意这是无差别缓存。而且要注意scope的origin路径限制。在回掉函数中我们使用事件对象提供的respondWith方法，它劫持http请求，并把一个Promise作为响应结果返回给用户。然后我们使用用户的请求对Cache Stroage进行匹配，如果匹配成功，则返回存储在缓存中的资源；如果匹配失败，则向服务器请求资源返回给用户，并使用cache.put方法把这些新的资源存储在缓存中。因为请求和响应流只能被读取一次，所以我们要使用clone方法复制一份存储到缓存中，而原版则会被返回给用户。（post请求也可以被缓存，但是缓存的位置indexedDB是一个更好的选择）。

        this.addEventListener('fetch', function (event) {
          console.log(event.request.url);
          event.respondWith(caches.match(event.request).then(res => {
              return res || fetch(event.request).then(responese => {
                      const responeseClone = responese.clone();
                      caches.open('sw_demo').then(cache => {
                          cache.put(event.request, responeseClone);
                      })
                      return responese;
              }).catch(err => {
                  console.log(err);
              });
          }))
      });

## 六.总结与分析
- 当用户第一次访问页面的时候，资源的请求是早于Service Worker的安装的，所以静态资源是无法缓存的；只有当Service Worker安装完毕，用户第二次访问页面的时候，这些资源才会被缓存起来；

- Cache Stroage中的缓存不会过期，但是浏览器对它的大小是有限制的，所以需要我们定期进行清理；

- 仅仅广播消息使用sw是不划算的。

- 缓存的清理需要手动操作，基本分为两步，重命名存储空间，清理就缓存。代码如下：

        // sw.js 更换缓存命名空间
        this.addEventListener('install', function (event) {
            console.log('install');
            event.waitUntil(caches.open('sw_demo_v2').then(function (cache) { 
                // 更换缓存命名空间
                return cache.addAll(['/style.css','/panda.jpg','./main.js'])
            }))
        });

        const cacheNames = ['sw_demo_v2'];
        // sw.js 校验过期的缓存进行清除
        this.addEventListener('activate', function (event) {
            event.waitUntil(caches.keys().then(keys => {
                return Promise.all[keys.map(key => {
                    if (!cacheNames.includes(key)) {
                        console.log(key);
                        return caches.delete(key); // 删除不在白名单中的 Cache Stroage
                    }
                })]
          }))
        });

## 完整DEMO

        // service_worker.js (在index.html中引入)
        navigator.serviceWorker.register('sw.js').then(() => {
          console.info('注册成功');
        }).catch((err) => {
          console.error('注册失败');
        })

        self.addEventListener('install', event => {
          event.waitUntil(() => console.info('安装完成的回调'))
        })

        self.addEventListener('activate', event => {
          event.waitUntil(() => console.info('激活完成的回调'))
        })


        // sw.js
        // 接收到消息后给所有web页面发送信息
        this.addEventListener('message', function (event) {
          clients.matchAll().then(clientLists => {
            clientLists.forEach(client => {
              client[0].postMessage('this message is from sw.js to page!!');
            })
          })
        });


        // main.js(在index.html引入)，发送消息到sw
        navigator.serviceWorker.controller.postMessage('This massage is come form web page!')
        // 接收sw的消息
        navigator.serviceWorker.addEventListener('message', function (e) {
          console.log(e.data); // this message is from sw.js, to page
        })

## API注意事项

- event.waitUntil(fun()) 生命周期中的方法。字面意思，让程序继续运行直到传入的fun执行完毕再卸载或者重装sw。

- self.clients.claim() 假如sw的scope值为/s/,sw在/s/a/域名下注册的前提下，用户先打开/s/b/页面，再打开/s/a/。那么sw默认只会控制/s/a,要控制/s/b那么就需要执行该方法。

- self.skipWaiting() 在install生命周期中调用，则新sw不用等待立即激活。

- self.importScripts('./indexDB.js') 引入外部js，路径是服务器路径

- self.clients 存储被控制的页面,只存不删，除非sw被卸载或者重装。

- clients.matchAll({includeUncontrolled: true,type: 'window'})，从clients中获取目前还在打开的clientwindow

- client.postMessage() 只能传递能json化的信息。负责会报错(例对象中的方法).
