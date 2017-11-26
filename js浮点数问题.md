(http://www.imooc.com/article/19963)

(https://mp.weixin.qq.com/s?__biz=MjM5NDMwNjMzNA==&mid=400999635&idx=1&
sn=5b52165c6ea0c4b31448cfa31ada1fa8)
(http://yanhaijing.com/javascript/2014/03/14/what-every-javascript-developer-should-know-about-floating-points/)
(http://www.cnblogs.com/ziyunfei/archive/2012/12/10/2777099.html)
(https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)

(https://www.zhihu.com/question/62732293)

### int 最大值
-2147483648－－2147483647

### toFixed(n)提供小数点后的n长度; toPrecision(x)提供x总长度

### float and double buffer(), v8里面根本就没有float
```
const buffer = require('buffer');
let buf1 = buffer.Buffer.alloc(1024);
buf1.writeFloatBE(1.3);
buf1.readFloatBE();

buf1.writeDoubleBE(1.3);
buf1.readDoubleBE();
```

### 
* v8 引擎中 js 的 Number 对象的内部实现只有两种，一是smi（也就是小整数），二是 double。Node.js 根本没有 float! 如果使用了 float , 注意存在精度的缺失。
* 位运算， javascript 只支持32位，超过32位的，需要用大数模拟。 https://github.com/justmoon/node-bignum

### +0 与 -0
```
-0 + -0

0 + 0
```

### 处理
```
function accMul(arg1,arg2) { 
  var m=0,s1=arg1.toString(),s2=arg2.toString(); 
  
  try{m+=s1.split(".")[1].length}catch(e){} 

  try{m+=s2.split(".")[1].length}catch(e){} 

  return Number(s1.replace(".",""))*Number(s2.replace(".",""))/Math.pow(10,m); 
}
```