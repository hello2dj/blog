今天看了一篇很棒的文章[似是而非的JS - 异步调用可以转化为同步调用吗？](http://mp.weixin.qq.com/s?__biz=MzU0OTExNzYwNg==&mid=2247483792&idx=2&sn=80bd02f356ccb03081d713b6181f0abb&chksm=fbb58a59ccc2034fcb9ba800fd3099046dc3d52ab8f21feb5a9fb91c3c5433b799c80166dadb&mpshare=1&scene=23&srcid=0818JpFSvA29BwkfZEai6mYV#rd)
里面讲解了很多知识点，写篇文章总结总结,其实所有所有的就是想把异步代码调用，串行化执行（async, promise, co，async/await等等方法，而这片文章就是讲讲里面到底是啥）
* ### jquery ajax的async属性（对于我这种几乎不用jquery的
  人来说完全不知道这是什么鬼，只能靠猜...），[查阅文档](https://api.jquery.com/jQuery.ajax/)，当async
  设置为true时异步调用,设为false时，则是同步调用
  > By default, all requests are sent asynchronously (i.e. this is set to true by default). If you need synchronous requests, set this option to false. Cross-domain requests and dataType: "jsonp" requests do not support synchronous operation. Note that synchronous requests may temporarily lock the browser, disabling any actions while the request is active. As of jQuery 1.8, the use of async: false with jqXHR ($.Deferred) is deprecated; you must use the success/error/complete callback options instead of the corresponding methods of the jqXHR object such as jqXHR.done().
  
  可问题就来了，就像文章里说的，为啥jquery 对于jsonp的实现不允许async: false呢，在上述片段理由一句话`Note that synchronous requests may temporarily lock the browser, disabling any actions while the request is active.`，同不调用会阻塞browser的其他处理，而jsonp的实现是基于script标签实现的,就是因为这个么？不知道了，ok我们来继续, 我看到stackoverflow上有这么一段回答
  > It is completely impossible to do synchronous
   JSONP. JSONP works by creating a \<script> tag;
   the browser does not expose any way to wait for
   a \<script> tag. In addition, SJAX is never a good idea; it will freeze the browser until the server replies.
   You need to correctly use asynchrony. Consider using promises.
   按照这段回答呢是说没办法，显然这是洪荒年代了，我来看看
   文中提到的async属性[caniuse](http://caniuse.com/#search=async) 显然使用这个属性，我们是有办法的。我们再来看看文中提到的实现
  ```
  export const loadJsonpSync = (url) => {
    var result;
    window.callback1 = (data) => (result = data)
  
    let head = window.document.getElementsByTagName('head')[0];
    let js = window.document.createElement('script');
    js.setAttribute('type', 'text/javascript');
    js.setAttribute('async', 'sync');

    // 这句显式声明强调src不是按照异步方式调用的
    js.setAttribute('src', url);
    head.appendChild(js);
    return result
  }
  ```
  上面这段代码其实不难理解，主要是要设置script标签的
  async属性为sync,
  `window.callback1 = (data) => (result = data)`, 
  这是个jsonp实现，callback1显然就是， 要执行的回调， 可是作者说拿到的却是undefined,解释一下为什么jsonp中的async没有生效，现在解释起来真的是相当轻松，即document.appendChild的动作是交由dom渲染线程完成的，所谓的async阻塞的是dom的解析(关于dom解析请大家参考webkit技术内幕第九章资源加载部分)，而非js引擎的阻塞。实际上，在async获取资源后，与js引擎的交互依旧是push taskQueue的动作，也就是我们所说的async call, 就是说async其实是阻塞dom解析线程，而非js执行线程，这就意味着，dom渲染就不进行了，一直到script加载完成，而js线程没有被阻塞，所以在dom线程阻塞等待的时候，js线程已经在执行了就直接反回result,所以很明显是undefined

  * ### 异步转同步实现（即只考虑如何实现异步转同步实现）
  先看一段文中提到的异步与同步定义
  > 同步（英语：Synchronization），指对在一个系统中所发生的事件（event）之间进行协调，在时间上出现一致性与统一化的现象。在系统中进行同步，也被称为及时（in time）、同步化的（synchronous、in sync）。--摘自百度百科
  异步的概念和同步相对。即时间不一致，不统一

  （确定百度不会死人么2333333),还可以借助甘特图来表达一下
  ![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNibtHkhzoibbV2jmVnpzdicUibquBOmlmia4wDG4f6JoibwS3ukIw6XY3j7y5HSM9OVEHkpJA92micJFGnRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)
  显然, t1,t2是同步，t1,t3是异步
  作者提了一个实现例子
  ```
  spinLock() {
   
    // 自旋锁
    fork Wait 3000 unlock()//开启一个异步线程，等待三秒后执行解锁动作

    //本来新生成的线程与主线程的执行时不相关的，
    //两者异步主线程的执行与新生成的线程不同步
    loop until unlock

    Put
    ‘unlock’
  }

  //pv原语，当信号量为假时立即执行下一步，同时将信号量置真
  //反之将当前执行栈挂起，置入等待唤醒队列
  //uv原语，将信号量置为假，并从等待唤醒队列中唤醒一个执行栈

  Semaphore() {
    pv()
    fork Wait 3000 uv()
    pv() //这一步是为了争夺信号量，因为虽然新生成的线程uv释放了信号量，但不一定就是主线程执行
    uv()

    Put 'unlock'
  }
  ```
  对于这段的解读，注释已经很详细了，这时候作者就继续追问了，那么js可以么？作者查看了，jquery,node的底层实现，都是使用的原生代码实现的，并没有使用js, 那么可以使用setTimeout来模拟fork么？
  作者实现了第一个版本如下
  ```
  var lock = true;
  setTimeout(() => {
    lock = false;
  }, 5000);

  while(lock);
  console.log('unlock')
  ```
  显然这段代码，肯定是不行的，因为大家都知道，js是单线程的，一旦执行了while(lock)就不可能再执行其他代码了，setTimeout的执行也是由于[事件循环](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)的调度执行的，但执行体仍然是同一个线程,作者按照理论模拟实现了一个eventloop,关于这个话题可以在写篇文章，坐等吧
  接下来我们继续改进我们上面的代码片段
  ```
  var lock = true
  setTimeout(() => {
    lock = false
    console.log('unlock')
  }, 5000)

  function sleep() {
    var i = 5000
    while(i--);
  }

  var foo = () => setTimeout(() => {
    sleep()
    lock && foo()
  });

  foo()
  ```
  作者说了，[这是对while(true)做了切块处理](http://blog.csdn.net/kongls08/article/details/6996528)(显然这又是可以讲解的一个话题)，其实就是估计一下结束时间，不让while(true)阻塞了所有的代码，使用sleep估计一下运行时间，是有偏差的
  可是，如果把代码最后的foo() 变成 foo() && console.log('wait5sdo')，显然wait5sdo依然没有在foo中的sleep执行完成以后在执行，是直接输出了，在js代码中，
  ========`同步执行代码必然是在异步代码之前执行的`=======
  所以，无论从理论还是实际出发，我们都不得不承认，在js中，把异步方法改成同步方法这个命题是水月镜花.

### async&await
  ```
  async function () {
    var data = await getAjax1()
    console.log(data);
  }
  ```
  我去这段代码亲手推翻了==同步执行的代码片段必然在异步之前。== 的黄金定律！借助作者话就是意不意外，惊不惊喜，
  相信很多人都会说，async/await是CO的语法糖，CO又是generator/promise的语法糖，好的，那我们不妨去掉这层语法糖，来看看这种代码的本质, 关于CO，读的人太多了，我实在不好老生常谈，可以看看这篇文章，咱们就直接绕过去了,这里给出一个简易的实现
  http://www.cnblogs.com/jiasm/p/5800210.html

  ```
  function wrap(wait) {
    var iter;
    iter = wait() //生成generator对象
    const f = () => { //就是一个递归
      const { value } = iter.next(); //这里调用generator的next方法移交执行权， 执行wait里的yield
      value && value.then(f) //yield的返回值若是存在是promise,则继续执行，显然这里可以做thunk函数的适配
    }
    f()
  }

  function * wait() {
    var p = () => new Promise(resolve => {
      setTimeout(() => resolve(), 3000)
    });

    yield p();
    console.log('unlock');
    yield p();
    console.log('unlock2');
    console.log('it\'s sync');
  }
  ```
  显然这里的yield是个黑魔法啊，其实我们完全可以使用cb,来实现一个generator,见下
  ```
  //显然要实现generator，我们就得关注生成器函数的返回，是一个可迭代的状态机，就是要实现这个状态机
  function wait() {
    const p = () => (value: new Promise(resolve => setTimeout(() => resolve(), 3000)));

    //很显然这个状态机，每次都把next函数给替换掉
    let state = {
      next: () => {
        state.next = programPart
        return p()
      }
      function programPart() {
        console.log('unlocked1')
        state.next = programPart2
        return p()
      }
      function programPart2() {
        console.log('unlocked')
        console.log('it\'s sync!')
        retturn {value: void 0}
      }
      return state;
    }
  }
  ```
  接下来是作者原话了
  到此我们就完成了generator到function的转换，，这段代码本身也解释清楚了generator的本质，高阶函数，片段生成器，或者直接叫做函数生成器
  (这一部分有很多可以聊得，再聊)
  > 来补债了
  这个原因经过作者的指导问题是这么来的

  ```
  function* g() {
    yield  var a = 123
    yield  console.log(a);
  }
  function g() {  
    let state = { 
      next: () => { 
      state.next = f1 
    } 
  }  
  function f1() { 
    var a = 123    
    state.next = f2 
  } 
  function f2() { console.log(a) } }
  ```
  就好比上面代码的转换过程，由于有变量提升的问题, generator中的console.log（a）是有输出的，而转换过后的是没有的，这就是其中作用域的坑
  
  其实这个东西，如果把代码写出来估计大家都能看出问题来，可若是没有的话，会有点儿没注意这个代码的转换问题，我还以为是之前的转换代码本身有问题，其实是这种转换，本身对于js作用域的割裂（**************）

  其实，在不知不觉中，我们已经重新发明了计算机科学中大名鼎鼎的CPS变换https://en.wikipedia.org/wiki/Continuation-passing_style
  最后的最后，容我向大家介绍一下facebook的CPS自动变换工具--regenerator。他在我们的基础上修正了作用域的缺陷，让generator在es5的世界里自然优雅。我们向facebook脱帽致敬！！https://github.com/facebook/regenerator
  > 推荐阅读 计算机程序的构造和解释 第一章generator部分实际上我们提供的解决方式存在缺陷，请从作用域角度谈谈（没有搞懂。。。还是菜啊，真是没看出来）