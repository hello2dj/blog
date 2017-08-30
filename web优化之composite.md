最近在进行web端的一些性能调优，看到了[taobaoFED的一篇文章](http://taobaofed.org/blog/2016/04/25/performance-composite/)([这是google chrome自己写的](http://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome)),里面讲到了利用合成层(compositing layers)来加速动画性能，其实就是在利用GPU加速，而合成层都会有单独的GraphicsLayer
这里先说几个概念
* ### Nodes(dom nodes)
* ### RenderObjects
  描述了某个dom对象的渲染方式，他和Dom node是一一对应的，这些RenderObjects保持了树结构，一个RenderObjects知道如何绘制一个node的内容， 他通过向一个绘图上下文（GraphicsContext）发出必要的绘制调用来绘制nodes。绘图上下文的责任就是向屏幕进行像素绘制(这个过程是先把像素级的数据写入位图中，然后再显示到显示器)，在chrome里，绘图上下文是包裹了的Skia, 一个chrome自己的2d图形绘制库。

  Each node in the DOM tree that produces visual output has a corresponding RenderObject. RenderObjects are stored in a parallel tree structure, called the Render Tree. A RenderObject knows how to paint the contents of the Node on a display surface. It does so by issuing the necessary draw calls to a GraphicsContext. A GraphicsContext is responsible for writing the pixels into a bitmap that eventually get displayed to the screen. In Chrome, the GraphicsContext wraps Skia, our 2D drawing library.

  Each node in the DOM tree that produces visual output has a corresponding RenderObject
  Each renderer represents a rectangular area usually corresponding to the node's CSS box, as described by the CSS spec. It contains geometric information like width, height and position.
* ### RenderLayers
  一般来说，拥有相同的坐标空间的 RenderObjects，属于同一个渲染层（PaintLayer), 这个和RenderObjects是一对多的,RenderLayers的存在就是为了，保证页面元素以正确的顺序合成（composite），这样才能正确的展示元素的重叠以及半透明元素等等，并且，RenderLayer也是以树的形式来组织结构的，一个RenderLayer节点的children是被存储在两个有序list中的，一个负Z值的有序list，另一个是正Z值的有序list
* ### GraphicsLayers
  为了充分利用（compositor，待解释）某些特殊的渲染层会被认为是合成层（Compositing Layers），合成层拥有单独的 GraphicsLayer，而其他不是合成层的渲染层，则和其第一个拥有 GraphicsLayer 父层公用一个。

  每个 GraphicsLayer 都有一个 GraphicsContext，GraphicsContext 会负责输出该层的位图，位图是存储在共享内存中，作为纹理上传到 GPU 中，最后由 GPU 将多个位图进行合成，然后 draw 到屏幕上，此时，我们的页面也就展现到了屏幕上。

这里是有一个顺序的见图：
    ![](http://www.chromium.org/_/rsrc/1479245083354/developers/design-documents/gpu-accelerated-compositing-in-chrome/the_compositing_forest.png)
按照google的文章里还有一些其他的layer
  From GraphicsLayers to WebLayers to CC Layers, Only a couple more layers of abstraction to go before we get to Chrome’s compositor implementation! GraphicsLayers can represent their content via one or more Web*Layers. These are interfaces that WebKit ports needed to implement; 
  当我们实现compositor时只需要几层抽象即可，GraphicsLayers可以通过一个或者多个Web*layers就可以展现他的内容，比如WebContentsLayer.h 或者 WebScrollbarLayer.h.

这个就是google文章里介绍 compositor的：
Chrome’s compositor is a software library for managing GraphicsLayer trees and coordinating frame lifecycles. Code for it lives in the src/cc directory, outside of Blink.

Introducing the Compositor

Recall that rendering occurs in two phases: first paint, then composite. This allows the compositor to perform additional work on a per-compositing-layer basis. For instance, the compositor is responsible for applying the necessary transformations (as specified by the layer's CSS transform properties) to each compositing layer’s bitmap before compositing it. Further, since painting of the layers is decoupled from compositing, invalidating one of these layers only results in repainting the contents of that layer alone and recompositing.

Every time the browser needs to make a new frame, the compositor draws. Note this (confusing) terminology distinction: drawing is the compositor combining layers into the final screen image; while painting is the population of layers’ backings (bitmaps with software rasterization; textures in hardware rasterization).

Whither the GPU?

So how does the GPU come into play? The compositor can use the GPU to perform its drawing step. This is a significant departure from the old software rendering model in which the Renderer process passes (via IPC and shared memory) a bitmap with the page's contents over to the Browser process for display (see “The Legacy Software Rendering Path” appendix for more on how that works).

In the hardware accelerated architecture, compositing happens on the GPU via calls to the platform specific 3D APIs (D3D on Windows; GL everywhere else). The Renderer’s compositor is essentially using the GPU to draw rectangular areas of the page (i.e. all those compositing layers, positioned relative to the viewport according to the layer tree’s transform hierarchy) into a single bitmap, which is the final page image.

其实上面这些有些就是单纯的从taobaoFED那篇文章中copy来，但我也看了一下google的那篇文章，我发现其实淘宝的那篇文章关于术语概念的描述就是google文章的翻译，但淘宝的栗子举的很详细，值得一看，里面提到了一旦renderLayer提升为了合成层就会有自己的绘图上下文，并且会开启硬件加速，有利于性能提升,里面列举了一些特点
  * ##### 合成层的位图，会交由 GPU 合成，比 CPU 处理要快
  * ##### 当需要 repaint 时，只需要 repaint 本身，不会影响到其他的层
  * ##### 对于 transform 和 opacity 效果，不会触发 layout 和 paint
有几个问题我们需要注意一下：
  1. 我们可以从里面看到，提升到合成层后合成层的位图会交GPU处理，但请注意，仅仅只是合成层的处理会GPU，那么生成合成层的位图处理呢，显然既然不是GPU，那就是CPU了。在google的文章里有句话`Each GraphicsLayer has a GraphicsContext for the associated RenderLayers to paint into. The compositor is ultimately responsible for combining the bitmap output of GraphicsContexts together into a final screen image in a subsequent compositing pass. `, 请注意这句话，说的是没个GraphicsLayer有自己的绘图上下文去绘制关联的RenderLayers，而 compositor是负责把绘图上下文的位图输出进行组合，那么组合这件事是交给了GPU，但是绘图上下文的工作依然是CPU啊，这里可以看一张google里的关于GPU的架构图
    ![](http://www.chromium.org/_/rsrc/1282925892145/developers/design-documents/gpu-accelerated-compositing-in-chrome/HandlingMultipleContexts.png)
  请仔细看，GPU的使用是通过，compositor向GPUprocess发命令来执行的，而GPU处理的其实是bitmap，那么问题就来了，bitmap哪里来的，显然也是cpu处理来的
  2. 当需要repaint的时候可以只repaint本身，不影响其他层，但是我们来看看浏览器的将一帧送到屏幕会采用的顺序：
  ![](https://img.alicdn.com/tps/TB1eabOLpXXXXX3XFXXXXXXXXXX-1093-167.jpg_720x720.jpg)
  我们可以看到在paint之前还有style， layout, 那就意味着即使合成层只是repaint了自己，但如果style和layout本身就很占用时间呢，可以通过chrome devtools看到一般在style/ layout后面都会有跟着一个update layer tree,很明显layout/style都变了，那么RenderObjects就得变，RenderObjects变了，那么RenderLayer是否要变呢？同理继续,上面的第三点，很明显的证明了我们的推论
  3. 要看清楚仅仅是transform和opacity不会引发layout 和paint，那么其他的属性就不保证了啊！
  4. 我们经常提到的reflow, repaint，显然当我们把某一个layer提升到了合成层后，对repaint是很有帮助的，可是对于reflow估计也不会有太好的提升如下图
    ![](http://taligarsiel.com/Projects/webkitflow.png)
  [样式的失效](https://docs.google.com/document/d/1vEW86DaeVs4uQzNFI5R-_xS9TcS1Cs_EUsHRSgCHGu8/edit?hl=zh-cn#)是会导致reattachment的，就是说整个render tree的重新绑定样式

然后淘宝的文章又举了几个具体的栗子来说明合成层优点：
  1. 提升动画效果的元素
    合成层的好处是不会影响到其他元素的绘制，因此，为了减少动画元素对其他元素的影响，从而减少 paint，我们需要把动画效果中的元素提升为合成层。

    提升合成层的最好方式是使用 CSS 的 will-change 属性。从上一节合成层产生原因中，可以知道 will-change 设置为 opacity、transform、top、left、bottom、right 可以将元素提升为合成层。
  2. 使用 transform 或者 opacity 来实现动画效果, 这样只需要做合成层的合并就好了
    注意：元素提升为合成层后，transform 和 opacity 才不会触发 paint，如果不是合成层，则其依然会触发 paint。具体见如下两个 demo
  3. 减少绘制区域
    对于不需要重新绘制的区域应尽量避免绘制，以减少绘制区域，比如一个 fix 在页面顶部的固定不变的导航 header，在页面内容某个区域 repaint 时，整个屏幕包括 fix 的 header 也会被重绘，见 [demo](http://taobaofed.github.io/demo/performance-composite-demo/paint/reduce/no-reduce.html)，结果如下：
      ![](https://img.alicdn.com/tps/TB1SK_9LpXXXXcaaXXXXXXXXXXX-699-304.png)
    
    而对于固定不变的区域，我们期望其并不会被重绘，因此可以通过之前的方法，将其提升为独立的合成层。

    减少绘制区域，需要仔细分析页面，区分绘制区域，减少重绘区域甚至避免重绘。
  接下来文章又提到了，合成层的一些问题
  看完上面的文章，你会发现提升合成层会达到更好的性能。这看上去非常诱人，但是问题是，创建一个新的合成层并不是免费的，它得消耗额外的内存和管理资源。实际上，在内存资源有限的设备上，合成层带来的性能改善，可能远远赶不上过多合成层开销给页面性能带来的负面影响。同时，由于每个渲染层的纹理都需要上传到 GPU 处理，因此我们还需要考虑 CPU 和 GPU 之间的带宽问题、以及有多大内存供 GPU 处理这些纹理的问题。

  1. 对于合成层占用内存的问题，我们简单做了几个 demo 进行了验证。

  [demo 1](http://taobaofed.github.io/demo/performance-composite-demo/memory/multi-layers-expect.html) 和 [demo 2](http://taobaofed.github.io/demo/performance-composite-demo/memory/multi-layers.html) 中，会创建 2000 个同样的 div 元素，不同的是 demo 2 中的元素通过 will-change 都提升为了合成层，而两个 demo 页面的内存消耗却有很明显的差别。

  2. 防止层爆炸
    通过之前的介绍，我们知道同合成层重叠也会使元素提升为合成层，虽然有浏览器的层压缩机制，但是也有很多无法进行压缩的情况。也就是说除了我们显式的声明的合成层，还可能由于重叠原因不经意间产生一些不在预期的合成层，极端一点可能会产生大量的额外合成层，出现层爆炸的现象。我们简单写了一个极端点但其实在我们的页面中比较常见的 [demo](http://taobaofed.github.io/demo/performance-composite-demo/memory/layer-explode.html)。
    解决层爆炸的问题，最佳方案是打破 overlap 的条件，也就是说让其他元素不要和合成层元素重叠。对于上述的示例，我们可以将 .animation 的 z-index 提高。修改后 [demo](http://taobaofed.github.io/demo/performance-composite-demo/memory/layer-explode-zIndex.html)

  之前无线开发时，大多数人都很喜欢使用 translateZ(0) 来进行所谓的硬件加速，以提升性能，但是性能优化并没有所谓的“银弹”，translateZ(0) 不是，本文列出的优化建议也不是。抛开了对页面的具体分析，任何的性能优化都是站不住脚的，盲目的使用一些优化措施，结果可能会适得其反。因此切实的去分析页面的实际性能表现，不断的改进测试，才是正确的优化途径。

接下来我也说一点其他的，当我们提升到合成层以后，也不是说可以为所欲为的，意味全都是GPU处理了，其实GPU只是在处理合成层的组合而已， 就像上面我提到的那样，接下来举个栗子大家就知道了
```
<style>
#a, #b {
  position: absolute;
  width: 100px;
  height: 100px;
  transform: translateZ(0);
}


#a {
 background-color: blue;
 left: 10px;
 top: 10px;
 z-index: 2;
 animation: move 1s infinite linear;
}

#b {
 background-color: red;
 left: 50px;
 top: 50px;
 z-index: 1;
}

@keyframes move {
  to { left: 100px; }
from { left: 30px; }
}
</style>

<div id="a">A</div>
<div id="b">B</div>
```
看上述代码，我把a，b都用translateZ提升到了合成层，然后加了个动画，但是我是直接改变left来实现动画的，此通过performance截图可以看到如下：
 ![](https://thumbnail0.baidupcs.com/thumbnail/ebc29b9454eaf8744ef81baa8863c949?fid=917901536-250528-732631519635699&time=1504011600&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-1WjIQcTAqJa6hkgmaP1Xu%2b1rO9I%3d&expires=8h&chkbd=0&chkv=0&dp-logid=5586966471304663325&dp-callid=0&size=c1920_u1080&quality=90&vuk=917901536&ft=image)
 动画依然在触发restyle, layout, update layer tree, GPU也是用了但是是在Composite阶段使用的
 在来看看绘制区域以及合成层的情况
  ![](https://thumbnail0.baidupcs.com/thumbnail/c9a1525028174a6e1012570c75ff16a9?fid=917901536-250528-1068075843533254&time=1504054800&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-z1X2wxf%2fsstl%2fhwQIg2PnXFNLS4%3d&expires=8h&chkbd=0&chkv=0&dp-logid=5599003877557123195&dp-callid=0&size=c1920_u1080&quality=90&vuk=917901536&ft=image)
  会发现他连重绘的区域都没有。。。，后来在performance中一看，发现这货根本就没有paint的阶段，其实在上图中也可以看出来，有些阶段确实是没有paint的

 那么接下来我直接把a,b 的translateZ去掉此时a,b就不再是合成层了，又会是如何呢见下图
  ![](https://thumbnail0.baidupcs.com/thumbnail/34e1c37e32f0d869902241955e90f709?fid=917901536-250528-983353382725428&time=1504058400&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-ZWdyp5E3KSJDUtDRzPJyrMgGnzk%3d&expires=8h&chkbd=0&chkv=0&dp-logid=5599409721730361736&dp-callid=0&size=c1920_u1080&quality=90&vuk=917901536&ft=image)

  ![](https://thumbnail0.baidupcs.com/thumbnail/9957e0f3d08e40b9d435f0e011b2d6cc?fid=917901536-250528-233356905656689&time=1504058400&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-0HVZzMia0GR%2fCBVpnibqARfrJhk%3d&expires=8h&chkbd=0&chkv=0&dp-logid=5599409721730361736&dp-callid=0&size=c1920_u1080&quality=90&vuk=917901536&ft=image)

  从上面两张图可以看出若是没有提升到合成层，他是会有paint的操作的，那么这两次操作我只是为了说一件事，那就是如果你的动画本身就触发了reflow等操作，那么即使提升至合成层，空也无法最大限度的优化，甚至没啥提升

  那么我们来看看，若是我们把动画改为使用translate来进行，就是把
  ```
  @keyframes move {
    from { left: 30px; }
    to { left: 100px; }
  }
  //换成
  @keyframes move {
  from { transform: translateX(0); }
  to { transform: translateX(70px); }
  }
  ```
  接下来看看我们改动后的效果
    ![](https://thumbnail0.baidupcs.com/thumbnail/804b33bc8d26d61de77f933e3d7992e9?fid=917901536-250528-512751169078126&time=1504058400&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-G6RXrqa1adzqkq4H8WXDI0t4T%2fk%3d&expires=8h&chkbd=0&chkv=0&dp-logid=5599586179262673234&dp-callid=0&size=c1920_u1080&quality=90&vuk=917901536&ft=image)
    ![](https://thumbnail0.baidupcs.com/thumbnail/3141d5f30e311d62188eb2412c72f0d0?fid=917901536-250528-768155206418161&time=1504058400&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-CObyulmt68g4Zb5RilHUV%2fqLuwE%3d&expires=8h&chkbd=0&chkv=0&dp-logid=5599586179262673234&dp-callid=0&size=c1920_u1080&quality=90&vuk=917901536&ft=image)
  
  很显然效果很明显，几乎全是GPU操作了，而cpu的操作几乎只有GC了，可对于我们上面看到的chrome的GPU架构，GPU是单独的进程，那若是全交给GPU了，那么GPU进程的资源消耗是否又会过高了呢？（如 cpu, memory等等);

  说了这么多，总结就是要合理的利用合成层，无论是否提升至合成层都应当注意reflow以及repaint的触发,上面的栗子出自 
  [CSS GPU Animation: Doing It Right](https://www.smashingmagazine.com/2016/12/gpu-animation-doing-it-right/)
