### An object size
 * #### Shallow size
  * just the object 就是对象本身的大小，就是一个浅copy后的大小，不包括引用的对象的大小， 比如: number 就是一个8字节的大小， 若是对象中包含两个引用ref，那么这个对象shallow size就是8byte(32位的)
  * 通常比较小
* #### Retained
  * shallow size 以及他所引用的所有的对象的大小（包括他引用的对象的引用，递归下去）
  为了更好理解retained size 上图
  ![](http://www.yourkit.com/docs/90/help/retained_objects.gif)


  ![](http://www.yourkit.com/docs/90/help/retained_objects_2.gif)

  从obj1入手，上图中蓝色节点代表仅仅只有通过obj1才能直接或间接访问的对象。因为可以通过GC Roots访问，所以左图的obj3不是蓝色节点；而在右图却是蓝色，因为它已经被包含在retained集合内。

  所以对于左图，obj1的retained size是obj1、obj2、obj4的shallow size总和；右图的retained size是obj1、obj2、obj3、obj4的shallow size总和。
  对于obj2，它的retained size是：在左图中，是obj2和obj4的shallow size的和；在右图中，是obj2、obj3和obj4的shallow size的和。

### 那么V8的GC root都有哪些呢
  * 还有一些全局变量

  * built-in object maps: 内建的对象

  * symbol table: 符号表(没搞明白这是个啥子鬼)

  * stacks of VM threads;(这个应该是指栈中的 变量)

  * compilation cache;(编译的缓存)

  * handle scopes;(v8中的属术语，v8中每个对象都是被封在handle中的，句柄的scope)

  * global handles;(全局的句柄)

### 那么哪些动作会导致新的分配动作呢
  * new(也包括字面量对象分配)
    * 从yong 内存池中分配
    * 一直很cheap(时间效率高，空间回收快不会一直占用)直到被升级为old
  * yong 内存池用尽了
    * 运行时强制执行一次GC
  * 交互式的app应道注意对象的分配模式
    * 游戏类的应当争取一帧内都不要有分配行为

### 使用devtools进行profiling的tips
  * 打开一个匿名窗口
  * 查找时要注意的
    * 忽略所有在括号里面的
    * 忽略所有暗色的
  * 在snapshot开始之前会进行一次GC

### three snapshot method (https://docs.google.com/presentation/d/1wUVmf78gG-ra5aOxvTfYdiLkdGaR9OhXRnOlIcEmu2s/pub?start=false&loop=false&delayms=3000&slide=id.g31ec7af_0_58)
* 先来一个snapshot1
* 执行一些能够进行分配的动作再来一个snapshot2
* 再执行以下相同的操作来一个snapshot3
* 最后在snapshot3的summary下查看在1和2之间分配的对象
