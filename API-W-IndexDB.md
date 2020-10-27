# IndexDB 浏览器数据库快速起步
## 概念

   IndexedDB是用于客户端存储大量结构化数据（包括文件/ blob）的低级API。此API使用索引来启用对这些数据的高性能搜索。虽然DOM Storage可用于存储少量数据，它对于存储大量结构化数据不太有用。IndexedDB提供了一种解决方案。

## 一.打开关闭数据库

        const request = window.indexedDB.open('name', 3, fallbackFun(request){}); // 数据库命名 数据库版本号 打开成功的回调
        let db; //数据库实例

- open()，有则打开数据库，无则新建数据库。该方法返回一个request对象。
- reqest.close()可以关闭数据库。
- open方法接收两个参数(仓库名称，仓库版本)。数据库版本默认为1或者当前版本。此参数不能使用浮点数(例如2.4)，负责就会被转化为整数四舍五入。有可能会导致其onupgradeneeded方法不会被触发。如果open打开的数据库版本低于当前本地数据库版本将会导致VER_ERR错误。
- open方法会返回request对象。其中包含三个回调函数：
  1. request.error db操作的错误回调，所有的db操作error时都会调用(事件冒泡)。

          request.error = (event) => {
            //打开失败
          };
  2. request.success db的成功回调，所有的db操作成功时时都会调用。

          request.success = (event) => {
            //打开成功
            db = request.result; //数据库实例
          };
  3. request.upgradeneeded 仓库版本升级，创建新的仓库时调用该回调。只能在这里创建删除仓库中的表和索引。

          request.upgradeneeded = (event) => {
            db = event.target.result; //数据库实例
          };

## 二.新建数据表

在已打开的仓库中创建新的数据表，注意只能在upgradeneeded方法中创建。一般步骤二和三一起进行，这里为了其详细信息进行拆分描述。具体实现见文章后续完整Demo。

        request.upgradeneeded = (event) => {
          db = event.target.result;
          // 创建一个一张叫做girl的表格，主键是id，主键不能重复。注意判断新建表是否存在。
          if(!db.objectStoreNames.contains('person')){
            let dataStore = db.createObjectStore('girl', {keyPath:'id'});

            // 为该表创建一个索引通过姓名来搜索客户。名字可能会重复，所以不能使用unique索引。若确定参数不重复可将参数改为true
            dataStore.createIndex('name', 'name', {unique: false});

            // 若该表创建完成 则可在其complete事件中进行后续操作，例如存储数据。
            dataStore.transaction.oncomplete = function(ev){
              // 新增数据
              var customerObjectStore = db.transaction("customers", "readwrite").objectStore("customers");
              const newDataArr = [{id:1, name:'贾静雯'， tel:999}];
              newDataArr.forEach(function(customer) {
                customerObjectStore.add(customer);
              });
            }
          }
        };

- 创建、删除新的表和索引只能在upgradeneeded方法中执行。
- 创建新的数据表之前需要判断是否该表已存在。同名会报错。
- 创建表时的第二个参数是设置数据的主键。若指定了固定值如id，则存储的数据中必须含有该字段
- 主键不提供时可让db自动生成。主键会依次递增。

      var dataStore = db.createObjectStore(
        'girl',
        { autoIncrement: true }
      );

- dataStore.transaction有参数回调函数oncomplete,onerror,onabort.其中onabort会在事务执行失败时事务回滚操作时调用。

## 三. 新建表的索引

  表的索引即要存储的数据结构中的键的集合。如果实在不理解可以抽象为-索引为表格的表头数据[id,name,age,tel],则表格数据为[{id:1,name:'林超越'，age：12,tel:''}].

        request.onupgradeneeded = function(event) {
          db = event.target.result;
          var objectStore = db.createObjectStore('girl', { keyPath: 'id' });
          objectStore.createIndex('name', 'name', { unique: false }); // 索引名称，索引所在的属性简介，配置(是否有重复值)
          objectStore.createIndex('email', 'email', { unique: true });
        }

- 和创建新的表类似，创建表的索引只能在onupgradeneeded方法中进行。所以打开仓库后在其onupgradeneeded方法中需要一次性创建完新的表和表的索引即表的数据结构.
- 存储的数据对象可以不含有索引字段，则通过该字段获取的数据中也不包含该数据。

## 四. 存储数据

  存储数据指的是**面向对象仓库**写入数据记录,所以所有的javascript对象都可以存入db，例如二进制blob对象。这需要通过事务完成。事务机制：要么成功要么失败。那么就不会出现只修改了一半数据的情况。

        const saveData = () => {
          const request = db
          .transaction(['gril'],'readwrite') // 第一个参数是个数组 传空则操作所有表
          .objectStore('gril')
          .add({
            id:'1',name:'闫妮',age：'38',tel:'119'
          })

          request.onsuccess = (ev) => {
            console.log('add success', ev);
          }

          request.onerror = () => {
            cosnole.log('add error');
          }
        };
        saveData();

- 写入数据需要新建事务transaction(['表名称',...]，权限).操作权限readonly、readwrite、versionchange。不指定操作默认则默认为readonly模式。
- 写入是异步操作.可以操作多个表。只在必要时指定readwrite。你同时可以执行多个readonly操作，但是readwrite只能同时执行一个在一个对象仓库上。
- add操作时主键不能重复，如果你不关心已存在的数据请使用put.

## 五.数据读取

数据的读取方式和存储方式类似。但是数据的获取通常分为两种一种获取全部数据，一种获取单个数据。

### 1.按主键读取单个数据，主键要为已知

        cosnt readDataOne = () => {
          const transaction = db.transaction(['gril']);
          const dataStore = transaction.objectStore('gril');
          const request = dataStore.get(1);// 读取数据主键值为1

          request.onerror= () => {
            console.log('读取失败！');
          }

          request.onsuccess = () => {
            if (request.result) {
              const {name, age, tel} = request.result;
              console.log(
                'name，age,tel',name,age,tel
              );
            } else {
              console.log('未获取到数据')
            }
          }
        }
        readDataOne();

- 按主键获取数据只能获取单个数据，没有该主键则返回undefined

### 2.按索引获取数据

#### 2.1.使用索引的get方法获取单个数据

        const transaction = db.transaction(['gril'], 'readonly');
        const dataStore = transaction.objectStore('gril');
        cosnt index = store.index('name');
        index.get('闫妮').onsuccess = function (e) {
          var result = e.target.result;
          if (result) {
            console.log('data', e.target.result);
          } else {
            console.log('未返回数据')
          }
        }

- name索引对应的数据会重复，对应的数据可能不会只有一条，在这种情况下你只能得到主键值最小的那个。而不是一次获得多条数据。

#### 2.2.使用索引的openCursor方法获取索引对应的所有数据

      const transaction = db.transaction(['gril'], 'readonly');
      const dataStore = transaction.objectStore('gril');
      cosnt index = store.index('name');
      index.openCursor().onsuccess=(ev)=>{
        const cursor = ev.target.result;
        if (cursor) {
          //cursor参数是一个值，是数据name字段对应的值比如是'贾静雯'。cursor.value是这条数据对象。
          cursor.continue(); // 继续操作则返回所有建立name的数据。负责返回主键值最小的那个。
        }
      }

- 使用openCursor()尽量不要通过ev.target.value访问目标完整对象生成数组。value值是依赖赋值。会有性能损耗。该方法常用语检索索引对应键的数据信息，比如检索name='张静初'的人有几个。
- openCursor接收两个参数搜索返回、搜索顺序。使用index.openCursor(范围,顺序)。范围的参数如下所示，顺序的值默认是正序，传递'prev'则为倒叙。第一个参数可传递null则不做限制。
        // 仅仅匹配吴超越
        const singleName = IDBKeyRange.on('吴超越')；

        // 匹配所有超过“吴超越”的，包括“吴超越”
        const lowerBoundKeyRange = IDBKeyRange.lowerBound("吴超越");

        // 匹配所有超过“吴超越”的，但不包括“吴超越”
        var lowerBoundOpenKeyRange = IDBKeyRange.lowerBound("吴超越", true);

        // 匹配所有不超过“柳岩”的，但不包括“柳岩”
        var upperBoundOpenKeyRange = IDBKeyRange.upperBound("柳岩", true);

        // 匹配所有在“吴超越”和“柳岩”之间的，但不包括“Donna”
        var boundKeyRange = IDBKeyRange.bound("Bill", "Donna", false, true);

- 获取name索引对应的完整数据(主键自动生成时是未知的)。

        // 获取name索引对应的所有数据
        cosnt index = store.index('name');
        index.openCursor().onsuccess = function(event) {
        const cursor = event.target.result;
        if (cursor) {
          // cursor.key是一个name值,就像 "贾静雯",cursor.value是整个数据对象。
          if(cursor.key === '贾静雯'){
            grilInfoArr.push(cursor.value);
            cursor.continue();
          }
        }
      };

      // 获取age对应的数据主键
      cosnt index = store.index('age');
      index.openKeyCursor().onsuccess = (ev) => {
        cosnt cursor = ev.target.result;
        if (cursor) {
          // cursor.key是一个数据的name值，例如'张艺兴'，cursor是该条数据的主键
          // 没有办法得到该条数据的其余部分
          cursor.continue(); // 继续获取其余值
        }
      }

### 3.遍历数据获取所有数据

#### 3.1.使用openCursor()方法获取

        const allDataArr = []; //所有的数据
        const readDataAll = () => {
          const dataStore = db.transaction('person').objectStore('gril');
          dataStore.openCursor().onsuccess = (ev) => {
            const cursor = event.target.result; //openCursor对象调用成功后返回的参数result指向数据表中的一条数据。
            if(cursor){
              const {age,name,tel} = cursor.value；
              allDataArr.push({age,name,tel});
              console.log('age,name,tel',age,name,tel);
              cursor.continue(); //查询下一个数据
            } else {
              console.log('无返回');
            }
          }
        }
        readDataAll();

- 执行cursor.continue()方法，获取下一个游标直到获取到undefined退出。
- 查看游标的cursor.value是会带来性能损耗的。因为游标对象是被懒生成的。

### 3.2.使用getAll方法获取所有数据

          dataStore.getAll().onsuccess = function(event) {
            alert("Got all customers: " + event.target.result);
          };

- getAll()，返回由对象仓库中所有对象组成的数组
- 由于需要一次性构建整个数组对象，所以低效，但是如果你想获取所有数据还是推荐使用次方法
- getAllKeys()和此方法类似，用于获取所有的主键

## 六.数据更新

按主键检索目标数据然后进行更新。无则创建，有则覆盖。如果数据主键未知则需要去手动获取数据主键。具体操作见数据获取。

        const update = () => {
          const request = db.transaction(['gril'], 'readwrite')
            .objectStore('gril')
            .put({
              id: 1,
              name: '闫妮',
              age: 38,
              tel: '120'
            });

          request.onsuccess = function (event) {
            console.log('数据更新成功');
          };

          request.onerror = function (event) {
            console.log('数据更新失败');
          }
        }
        update();

## 七.删除数据
db数据的删除也是按主键删除的。需要提前获取主键。

        const remove = () => {
          const request = db.transaction(['gril'], 'readwrite')
            .objectStore('gril')
            .delete(1); //参数为主键

          request.onsuccess = function (event) {
            console.log('数据删除成功');
          };
        }
        remove();

## 八.其他

**注意**

- 所有的IndexDB操作都是异步的事务机制。操作失败会回滚。虽然失败的场景很少但是类似复杂计算后的结果类似的数据还是需要存储的,IndexDB存储失败可以直接存储数据减少计算。在这里要注意回滚的回调函数onabort。

- 所有的IndexDB数据库操作都是基于open方法返回的request实例对象进行操作的。open方式虽然很少有失败场景但是也不是没有(用户拒绝浏览器开启IndexDB/隐私模式下打开应用/其他)。

- 所有的IndexDB操作的错误回调都是冒泡的，最终会冒泡到open方法返回的request对象上。所以所有的error都会被request的onerror监测。所以整个indexDB程序你都可以使用一个request.error错误回调处理。常见错误-版本低于本地版本。

- 创建新的数据仓库或者打开一个高于本地版本号的数据仓库时，request对象的onupgradeneeded的函数会被调用。可以在该函数中访问IndexDB对象实例，然后创建，删除新的表和索引。

- IndexDB的异步操作机制可能会导致意向不到的错误,例如意外关闭浏览器导致IndexDB没有执行完成。清空表后重新写入数据这种操作尽量在一个事物中操作;对于react/vue等框架用户尽量不要在卸载生命周期中执行db操作，防止浏览器关闭导致的卸载无法执行完毕。

**应用场景及拓展**

- 大量懒性数据的存储
  应用需要获取大量第三方数据进行复杂运算后和界面进行交互展示。

- 埋点记录网站性能数据
  记录网站性能数据，如资源下载，常用页面，错误信息等。后台定时同步用户实时使用信息，以便开发者了解应用状态和功能。

- 应用缓存
  和serviceWorker配套使用，实现AppCache。

- 静态资源缓存
  网络云相册音乐小视频的缓存，为了更快的首屏渲染速度可以对这些静态资源首屏内的内容进行缓存，等网络请求完成后再更新。


## 完整的ReactDemo

        import React from "react";

        const DBName = "demo";
        const version = 2;
        const DBBaseName = 'gril';

        export default class extends React.Component {
          constructor(props) {
            super(props);
            this.state = {

            };
          }

          componentDidMount() {
            let req = indexedDB.open(DBName, version);
            let that = this;

            req.onsuccess = function (ev) {
              // 这里的this指向req对象。
              that.db = ev.target.result;
              console.log("打开仓库成功", that.db);
              // 关闭方法 db.close();
              // 也有clear方法不过是在objectStore方法返回的对象上调用。
            };

            req.onversionchange = () => {
              this.db.close();
              alert('仓库已有新版本，本仓库关闭！');
            }

            req.onerror = function (evt) {
              console.error("打开仓库失败", evt.target.errorCode);
            };

            req.onupgradeneeded = function (ev) {
              // 创建新数据库 打开高版本数据库时调用
              console.log("openDb.onupgradeneeded....................");
              // 创建新表
              var store = ev.target.result.createObjectStore(DBBaseName, {
                keyPath: "id",
                autoIncrement: true,
              });

              // 创建新的索引
              store.createIndex("name", "name", { unique: false });
              store.createIndex("age", "age", { unique: false });
              store.createIndex("tel", "tel", { unique: false });
              store.createIndex("email", "email", { unique: false });

              // 创建表和索引完成的事件回调
              store.transaction.oncomplete = () => {
                console.log('创建新表及索引完成。')
              }
            };
          }

          saveData = () => {
            console.log(this.db);
            const request = this.db.transaction([DBBaseName], 'readwrite').objectStore(DBBaseName)
              .add({ id: Date.now(), name: '闫妮', age: "38", tel: "119", email: "unknow" });
            request.onsuccess = function (ev) {
              console.log(ev);
              console.log("数据添加成功。");
            };
            request.onerror = function (ev) {
              console.log("数据添加失败。",ev.target.error);
            };
          };

          getAllData = () => {
            const dataStore = this.db.transaction(DBBaseName).objectStore(DBBaseName);
            dataStore.getAll().onsuccess = (ev) => {
              console.log(ev.target.result);
            }
          }

          getAllData_1 = () => {
            this.db.transaction(DBBaseName).objectStore(DBBaseName).openCursor().onsuccess = (ev) => {
              console.log(ev.target.result);
              if(ev.target.result){
                // 循环完所有数据后返回的ev.target.result值为null所以要在这里判断
                // ev.target.result.value是该条数据对象，但是对象是懒挂载的，在这里访问value会有性能问题
                // ev.target.result.key/primaryKey值是该条数据对象的主键
                // ev.target.result.source IDBObjectStore对象
                ev.target.result.continue(); 
              }
            };
          }

          closeDB = () => {
            this.db.close();
            alert('仓库已关闭');
          }

          clearDataBase = () => {
            const request = this.db.transaction(DBBaseName,'readwrite').objectStore(DBBaseName).clear();
            request.onerror = (ev) => {
              console.log('清除表数据失败', ev);
            }
            request.onsuccess = (ev) => {
              console.log('清除表数据成功', ev)
            }
          }

          render() {
            return (
              <>
                <h2>存储数据</h2>
                <button onClick={this.saveData}>新建一条数据数据</button>
                <button onClick={this.getAllData}>使用getAll获取所有的数据</button>
                <button onClick={this.getAllData_1}>使用openCursor游标循环获取所有的数据</button>
                <button onClick={this.clearDataBase}>清理仓库中的此数据表</button>
                <button onClick={this.closeDB}>关闭该仓库</button>
              </>
            );
          }
        }


