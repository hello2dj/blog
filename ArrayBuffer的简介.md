ArrayBuffer使用来处理二进制数据的，并且有一些新的就API依赖于ArrayBuffer，例如websockets, web intents, XMLHttpRequest version 2 以及webworkers. 
从语义上讲,ArrayBuffer就是一个通过一定的视窗去操作的简单字节数组，这个视窗就是一个ArrayBufferView的实例，定义了里面的字节数据是如何按照预期的数据结构进行组织的，举个栗子：如果你知道一个ArrayBuffer里面的字节数据是以16位的无符号整数展示的话，你就可以直接把ArrayBuffer用一个Uint16Array的视窗包裹起来，然后就可以通过Uint16Array所定义的有关Uint16方法来操作这些数据，就好像这个ArrayBuffer就是一个整数数组一样， ArrayBuffer的构造只有一个参数就是字节长度有且只有这一个

```
//例如 buf 包含了字节数据[0x02, 0x01, 0x03, 0x07]
//这里一定要注意硬件的大端与小端模式，x86就是小端模式
//对ArrayBuffer的读写操作其实都是同过TypedArray以及DataView操作的，也就是往里存数据也是一样的
let bufView = new Uint16Array(buf);
if (bufView[0] === 258) {
  console.log("ok");
}
bufView[0] = 255 //buf现在存贮的就是[0xFF, 0x00, 0x03, 0x07]
bufView[0] = 0xff05 //现在就是 [0x05, 0xFF, 0x03, 0x07]
bufView[1] = 0x0210 //现在就是 [0x05, 0xFF, 0x10, 0x02]


/******** 再举一个写入的栗子*******/
//比如我声明一个2字节长的ArrayBuffer
let arr = new ArrayBuffer(2);
//然后通过Uint8来观察以及操作ArrayBuffer
let view = new Uint8Array(arr);
// log view : Uint8Array [ 0, 0 ]
// 然后 设置0
view[0] = 255;
// log view : Uint8Array [ 255, 0 ]
//在设置为256，注意溢出8位无符号整形最大值就是255，溢出以后就是0了
view[0] = 256;
// log view : Uint8Array [ 0, 0 ]
//等等。。。

```
> BLOB (binary large object)，二进制大对象 
经常会遇到的一个很常见的实用性的问题就是如何把String转化到ArrayBuffer以及反转回来，但实际上ArrayBuffer是一个字节数组，这种转换需要两端都有一个统一的认识就是如何把string显示为字节，你可能已经看过这种协议了：字符串的编码（比如Unicode UTF-16和iso8859-1)，因此，假设你和其他参与者都认同UTF-16编码，这样转换就像下面了： 
```
function ab2str(buf) {
  return String.fromCharCode.apply(null, new Uint16Array(buf));
}

function str2ab(str) {
  let buf = new ArrayBuffer(str.length * 2); //bytes for each
  let bufView = new Uint16Array(buf);
  for (let i = 0; strLen = str.length; i < strLen; i++) {
    bufView[i] = str.charCodeAt(i);
  }
  return buf;
}
```
> 注意点，当我们使用Uint16Array的时候，他仅仅是ArrayBuffer的一个视窗层，只是在这个视窗层看来数据都是16位的元素，他并不处理字符编码本身，编码自身的处理依然需要String.fromCharCode和str.CahrCodeAt来处理

接下来，继续深入看看吧，译文到此就结束了
先总结一下ArrayBuffer的构造方法吧
* #### new ArrayBuffer(bytes.length),然后同过bufferView操作
* #### 还有一个就是同某个BufferView来构造，如下
```
let u8a = new Uint8Array.from([1,2,3,5]) 
//bufferView的from方法接受一个array-like的对象，可以构造出一个bufferView，当然底层还得new 一个ArrayBuffer
//所以我们就拿到了ArrayBuffer咯
let ab = u8a.buffer //只读属性
//画蛇添足，为什么就是为了告诉你，我们可以这么做，并且也是为了说明这些bufferView使用了共同的ArrayBuffer,意味着你修改了其中的一个另一个的展现方式也会改变
let u16a = new Uint16Array(ab);
//log u16a Uint16Array [ 513, 1027 ]
u16a[0] = 1234;
//log u16a Uint16Array [ 1234, 1027 ]
//log u8a Uint8Array [ 210, 4, 3, 4 ]
```
到底是什么是TypedBuffer呢，其实就是下面所列举的
* ### ArrayBufferView
  * ##### Int8Array
  * ##### Uint8Array
  * ##### Uint8ClampedArray
  * ##### Int16Array
  * ##### Uint16Array
  * ##### Int32Array
  * ##### Uint32Array
  * ##### Float32Array
  * ##### Float64Array
那么他们和ArrayBuffer又是什么关系呢？ 引用两张图来说明一下，图中的01数据呢，就是ArrayBuffer
![](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2017/06/02_05.png)
继续go on
![](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2017/06/02_06.png)
go go go
最后一张一看你就会更清楚了
![](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2017/06/02_07.png)
这个时候估计就会有疑问了，若是搞过多线程的话，就会问了，两个同时访问同一内存区域，不会出问题么？答案是在js里不会，因为js是单线程啊，但又有来了，那webworkers呢，根据一些文章解读当把ArrayBuffer传递以后是没有办法在访问到了(暂时未实验见[文章](https://hacks.mozilla.org/2017/06/a-cartoon-intro-to-arraybuffers-and-sharedarraybuffers/)), 那就没办法了么？有的，方法总比问题多？那就是SharedArrayBuffer，和他配套的就是Aomtic的一些操作详见也是上篇文章，至于TypedArray的操作已经在上面又说了，大部分都类似

最后在说一下DataView
DataView的签名 ：new DataView(buffer [, byteOffset [, byteLength]])
签名的参数很明显就不解释了，DataView给了我们对ArrayBuffer更多的操作空间，他可以让我们有选择性的操作某一范围的ArrayBuffer的数据
这里是DataView的方法[列表](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView)
从这些方法我们可以看出我可以以多种的方式操作这段范围的ArrayBuffer，例如以int8, int16都可以，但是当我们使用Uint8Array操作时就只能以uint8的方式操作，而DataView就不一样了，可以多种多样,注意大端小端模式，下列操作的参数是offset, value, littelEndianAble(false or true)
```
Read
DataView.prototype.getInt8()
DataView.prototype.getUint8()
DataView.prototype.getInt16()
DataView.prototype.getUint16()
DataView.prototype.getInt32()
DataView.prototype.getUint32()
DataView.prototype.getFloat32()
DataView.prototype.getFloat64()
Write
DataView.prototype.setInt8()
DataView.prototype.setUint8()
DataView.prototype.setInt16()
DataView.prototype.setUint16()
DataView.prototype.setInt32()
DataView.prototype.setUint32()
DataView.prototype.setFloat32()
DataView.prototype.setFloat64()
```
具体使用哪种方式还是要看具体的使用场景，大家自行发现吧，打完收工,下次写写Blob以及nodejs的buffer对象