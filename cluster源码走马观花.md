```
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`主进程 ${process.pid} 正在运行`);

  // 衍生工作进程。
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`工作进程 ${worker.process.pid} 已退出`);
  });
} else {
  // 工作进程可以共享任何 TCP 连接。
  // 在本例子中，共享的是一个 HTTP 服务器。
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('你好世界\n');
  }).listen(8000);

  console.log(`工作进程 ${process.pid} 已启动`);
}
```
* lib/cluster.js 调用内部cluster, 根据NODE_UNIQUE_ID来判断是否是master, NODE_UNIQUE_ID是一个自增长ID(lib/master.js#line27)
```
const childOrMaster = 'NODE_UNIQUE_ID' in process.env ? 'child' : 'master';
module.exports = require(`internal/cluster/${childOrMaster}`);
```
* cluster.fork
  * setupMaster // 设置参数
  * createWorkerProcess(master.js#102) // 创建worker的进程
  * child_process.fork 一个进程 /lib.js/child_process.js#line53
    * fork时的参数http://nodejs.cn/api/child_process.html#child_process_options_stdio
    * child.spwan (/lib/child_process#line506)
    * child.spwan (/lib/internal/child_process#line270) 在这里会把所有的IPC，pipe等建立
    * setupChannel (/lib/internal/child_process#line449)
    * subprocess的send方法(/lib/internal/child_process#line587)http://nodejs.cn/api/child_process.html#child_process_subprocess_send_message_sendhandle_options_callback
    * (/lib/internal/child_process#line39)handleConversion
    * StreamBase::WriteString(/src/stream_base.cc)StreamBase::WriteString 这
  ```
    // node a.js -c
    args: process.argv.slice(2), // 命令行参数 -c
    exec: process.argv[1], // 这个是执行的脚本 : a.js
    execArgv: process.execArgv, // 这个是?
  ```

  ```
  :(){ :|:& };:
  注解如下：
  :()      # 定义函数,函数名为":",即每当输入":"时就会自动调用{}内代码
  {        # ":"函數起始字元
      :    # 用递归方式调用":"函数本身
      |    # 並用管線(pipe)將其輸出引至...（因为有一个管線操作字元，因此會生成一個新的進程）
      :    # 另一次递归调用的":"函数
  # 综上,":|:"表示的即是每次調用函数":"的時候就會產生兩份拷貝
      &    # 調用間脱鉤,以使最初的":"函数被關閉後為其所調用的兩個":"函數還能繼續執行
  }        # ":"函數終止字元
  ;        # ":"函数定义结束后将要进行的操作...
  :        # 调用":"函数,"引爆"fork炸弹
  ```
  * 然后woker process监听internalMessage 这里queryServer很有意思
* lib/net.js line 1396 [how to backlog works](http://www.jianshu.com/p/7fde92785056)