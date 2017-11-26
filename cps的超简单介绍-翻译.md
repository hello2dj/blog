关于CPS [原文地址](http://matt.might.net/articles/by-example-continuation-passing-style/)

state = (Store, Continuation, Env) 计算机的状态转换

cps是在1970s出现的一种编程风格，在1980s和1990s作为高级语言编译器的中间展示形式有着突出的表现
如今他作为非阻塞系统的编程风格再次被提起(通常是分布式)
cps在我的心中始终战友一席之地，因为他是我在Ph.D时期的秘密武器，他缩短了我取得Ph.D的时间，并且减少很多痛苦（agnoy）
这片文章介绍了cps的两个方面，一是在js中的异步编程风格，二是简单的介绍一下他在函数式语言中的中间（intermediate）形式

文章的主题：
  * #### CPS in JavaScript
  * #### CPS for Ajax programming
  * #### CPS for non-blocking programming (in node.js)
  * #### CPS for distributed programming
  * #### How to implement exceptions using CPS
  * #### A CPS converter for a minimal Lisp
  * #### How to implement call/cc in Lisp
  * #### How to implement call/cc in JavaScript

### what is continuation-passing style?
```
  What are continuations?
  Concretely, a continuation is a procedure that represents the remaining steps in a computation. For the expression 3 * (f() + 8), think about the remaining steps in the compuation after evaluating the expression f(). For example, in C/Java, the procedure current_continuation is the continuation of the call to f():

  void current_continuation(int result) {
    result += 8 ;
    result *= 3 ;
    (continuation of 3 * (f() + 8))(result) ;
  }
  The value passed to the continuation is the return value of the call.
```
如果一个语言支持continuation, 那么编程者就可以添加一些控制结构好比异常处理，回溯（backtracking），线程和生成器

不幸的是，许多关于continuations的解释让人感到模糊和不解，（Such power deserves a solid pedagogical foundation.Continuation-passing style is that foundation）这样强大的工具应当有一个坚实的教学基础，而cps就是那个基础。
cps通过代码给出了continuation的意义
有一个更好的方式可以让编程者自己了解cps，那就是通过遵守下面的一个限制或者约定：任何一段程序都禁止向他的调用者返回
```
//意味着调用a，是用不会返回的
function a () {...}
a()
```
> 这里有另一个对cps说法， cps使得控制流显示化----你不需要return，throw, break, continue, 不允许任意的跳转，甚至不允许在async function中使用for,while, 一般来说，手动编写cps是不直观并且容易出错的
### 举个栗子 Identity function
正常来写的话如下
```
function id(x) {
  return x ;
}
```
如果是cps
```
function id(x,cc) {
  cc(x) ;
}
### 再举个栗子 Naive factorial（简单的阶乘）
```
function fact(n) {
  if (n == 0)
    return 1 ;
  else
    return n * fact(n-1) ;
}
```
cps模式
```
function fact(n,ret) {
  if (n == 0)
    ret(1);
  else
    fact(n-1, function (t0) {
     ret(n * t0);
    });
}
```
我们来用一下
```
fact (5, function (n) { 
  console.log(n) ; // Prints 120 in Firebug.
})
```
### 尾递归的栗子 Tail-recursive factorial
```
function fact(n) {
  return tail_fact(n,1) ;
}
 
function tail_fact(n,a) {
  if (n == 0)
    return a ;
  else
    return tail_fact(n-1,n*a) ;
}
```
cps的栗子
```
function fact(n,ret) {
  tail_fact(n,1,ret) ;
} 
 
function tail_fact(n,a,ret) {
  if (n == 0)
    ret(a) ;
  else
    tail_fact(n-1,n*a,ret) ;
}
```

### cps和ajax
一个简单的fetch实现
```
/*
 fetch is an optionally-blocking 
 procedure for client->server requests.
 
 If only a url is given, the procedure 
 blocks and returns the contents of the url.
 
 If an onSuccess callback is provided, 
 the procedure is non-blocking, and the
 callback is invoked with the contents 
 of the file.
 
 If an onFail callback is also provided, 
 the procedure calls onFail in the event of 
 a failure.
 
*/
 
function fetch (url, onSuccess, onFail) {
 
  // Async only if a callback is defined: 
  var async = onSuccess ? true : false ;
  // (Don't complain about the inefficiency
  //  of this line; you're missing the point.)
 
  var req ; // XMLHttpRequest object.
 
  // The XMLHttpRequest callback:
  function processReqChange() {
    if (req.readyState == 4) {
      if (req.status == 200) {
        if (onSuccess) 
          onSuccess(req.responseText, url, req) ; 
      } else {
        if (onFail) 
          onFail(url, req) ;
      }
    }
  }
 
  // Create the XMLHttpRequest object:
  if (window.XMLHttpRequest) 
    req = new XMLHttpRequest();
  else if (window.ActiveXObject) 
    req = new ActiveXObject("Microsoft.XMLHTTP");
 
  // If asynchronous, set the callback:
  if (async) 
    req.onreadystatechange = processReqChange;
 
  // Fire off the request:
  req.open("GET", url, async);
  req.send(null);
 
  // If asynchronous,
  //  return request object; or else
  //  return the response.
  if (async) 
    return req ;
  else
    return req.responseText ;
} 
```

### cps 和非阻塞编程
一部分的cps编程对于nodejs来说是自然的
```
var sys = require('sys') ;
var http = require('http') ;
var url = require('url') ;
var fs = require('fs') ;
 
// Web server root:
var DocRoot = "./www/" ;
 
// Create the web server with a handler callback:
var httpd = http.createServer(function (req, res) {
  sys.puts(" request: " + req.url) ;
 
  // Parse the url:
  var u = url.parse(req.url,true) ;
  var path = u.pathname.split("/") ;
 
  // Strip out .. in the path:
  var localPath = u.pathname ;
  //  "<dir>/.." => ""
  var localPath = 
      localPath.replace(/[^/]+\/+[.][.]/g,"") ;
  //  ".." => "."
  var localPath = DocRoot +  
                  localPath.replace(/[.][.]/g,".") ;
 
  sys.puts(" local path: " + localPath) ;
   
  // Read in the requested file, and send it back.
  // Note: readFile takes the current continuation:
  fs.readFile(localPath, function (err,data) {
    var headers = {} ;
 
    if (err) {
      headers["Content-Type"] = "text/plain" ;
      res.writeHead(404, headers);
      res.write("404 File Not Found\n") ;
      res.end() ;  
    } else {
      var mimetype = MIMEType(u.pathname) ;
 
      // If we can't find a content type, 
      // let the client guess.
      if (mimetype)
        headers["Content-Type"] = mimetype ;
 
      res.writeHead(200, headers) ;
      res.write(data) ;
      res.end() ;   
    }
   }) ;
}) ;
 
// Map extensions to MIME Types:
var MIMETypes = {
 "html" : "text/html" ,
 "js"   : "text/javascript" ,
 "css"  : "text/css" ,
 "txt"  : "text/plain"
} ;
 
function MIMEType(filename) {
 var parsed = filename.match(/[.](.*)$/) ;
 if (!parsed)
   return false ;
 var ext = parsed[1] ;
 return MIMETypes[ext] ;
}
 
// Start the server, listening to port 8000:
httpd.listen(8000) ;
```

### cps 和分布式计算
假设你写了一个组合的choose函数，正常的写法：
```
function choose (n, k) {
  return fact(n) / (fact(k) * fact(n-k));
}
```
现在假设你想要再一个server上计算阶乘而不是在本地
你可能会重写fact来等待server的响应，那是不好的，相应的如果你使用cps来书写：
```
function choose(n, k, ret) {
  fact(n, function (factn) {
    fact(n - k, function (factnk) {
      fact(k, function (factk) {
        return (factn / (factnk * factk));
      });
    });
  });
}
```
现在可以很直观的重写server上的异步阶乘计算
```
function fact(n, ret) {
  fetch("./fact/" + n, function (res) {
    ret(eval(res));
  });
}
```

### 使用cps实现异常
一旦程序使用cps,他就打破了语言本身的标准异常原理，但幸运的是，使用cps可以很简单的实现异常处理
异常是一个特殊的continuation
通过沿着current continuation传递current exceptional continuation,可以替换try/catch 语法糖
可以看下面的栗子
```
function fact (n) {
  if (n < 0)
    throw "n < 0" ;
  else if (n == 0)
    return 1 ;
  else
    return n * fact(n-1) ;
}
 
function total_fact (n) {
  try {
    return fact(n) ;
  } catch (ex) {
    return false ;
  }
}
 
document.write("total_fact(10): " + total_fact(10)) ;
document.write("total_fact(-1): " + total_fact(-1)) ;
```

通过添加exceptional continuation, 见下
```
//与node不同，node只是部分cps,而这中处理就是完全的cps
function fact (n,ret,thro) {
 if (n < 0)
   thro("n < 0") 
 else if (n == 0)
   ret(1)
 else
   fact(n-1,
        function (t0) {
          ret(n*t0) ;
        },
        thro)
}
 
function total_fact (n,ret) {
  fact (n,ret,
    function (ex) {
      ret(false) ;
    }) ;
}
 
total_fact(10, function (res) {
  document.write("total_fact(10): " + res)
}) ;
 
total_fact(-1, function (res) {
  document.write("total_fact(-1): " + res)
}) ;
```
### cps与编译
再过去的几十年里，cps在函数式语言的编译器中是中间显示的重要工具
cps 去掉了retrun, 异常和first-class continuations(将continuation作为编程语言的一等公民)的语法糖
总之一句话，cps在编译领域起了很大的作用
接下来的就是Lisp语言的一些东西了，暂时还不懂，等看懂了再说吧流泪啊啊啊啊啊