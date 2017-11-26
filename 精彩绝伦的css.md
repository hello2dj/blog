### 调整字体值的顺序
 * font: <font-size> <font-family>; 这顺序是不能改变，若是改变了浏览器会忽略的，所以还是单开写font-size和font-family的好
 * font: bold italic 100% sans-serif;字号前面的属性值顺序可以改变，但是不能插入到字号和font-family之间

### 行高
  * line-height可以接受无单位的值，也可以使用带单位的行高值(比如em, rem， %)---尽管一般情况不推荐这么做
  * 若是定义了一个带单位的值，，例如1em或者100%时，就会将`计算后的行高值`传给全部的后代元素,如下例使用em为单位(em是使用元素本身计算后的字号值来进行计算的，百分比也是一样的)
  ```
  // css的书写
  ul {
    font-size: 15px;
    line-height: 1em;
  }
  li {
    font-size: 10px;
  }
  small {
    font-size: 80%;
  }
  
  <ul>
    <li>I'm a list item 
      <small>small text</small>
    </li>
  </ul>

  //应用的样式
  ul {font-size: 15px; line-height: 1;}
  li {font-size: 10px; leigth: 15px;}
  small {font-size: 80%; line-height: 15px}
  ```
  * 继续上面的栗子，若是把em单位去掉，则样式会变成line-height: 1;现在传给后代元素的是这个原始的数字，用来表示后代元素所使用的一个`换算系数(比如一个乘数)`,而不是计算后的结果值,
  ```
  //css书写
  ul { font-size: 15ox; line-height: 1;}
  li { font-size: 10px;}
  small {font-size: 80%;}

  //应用的样式
  ul {
    font-size: 15px;
    line-height: 1;
  }
  li {
    font-size: 10px;
    line-height: 10px;
  }
  small {
    font-size: 80%;
    line-height: 8px;
  }
  ```
  * 这所带来的结果就是应当在html或者body元素以及任何可能包含后代的元素上应用行高时使用无单位值的缘故
### 避免缺少样式的边框值
  例如 form { border: 2px gray; },这是不会出现边框的因为缺少border-style, 而border-style的默认是none，即啥都木有
### 抑制元素的显示
  * 隐藏元素的方法最显而易见的就是display:none，但是这个方法有两个问题
    * 1是若是使用js，设置一个元素display:none,那么我再恢复显示的时候我应该把display属性设置为啥啊，inline?block?显然是要根据具体的元素来定义，但是我们可以把display设置为``，这样就会把display属性值设置为其余css中设置的值
    ```
    var obj = document.getElementById(`linker`);
    obj.style.display = 'none';
    //恢复时
    obj.style.display = '' ?
    ```
  * 另一个问题是，屏幕阅读器无法识别这些隐藏元素

### 抑制元素的显示
  使用visibility: hidden即可隐藏元素但是元素依然参与布局只是，看不见而已

### 将元素移除屏幕
  若是想隐藏一个元素，同时阅读器也可以识别，但是又看不见，则可以这样做
  ```
  .hide {position: absolute; top: -10000em; left: -10000em;}
  ```
  若是想让他再回来
  ```
  .show{top: 0; left: 0;}
  ```
  若是想让他再回到正常的文档流中，则可以这样
  ```
  .show{position: static;} 
  ```
  定位默认值，浏览器会忽略top和left的

  若是想让元素再以包含块的身份回到正常文档流的话，则应当这么使用
  ```
  .show {position: relative; top: 0; left: 0;}
  ```
  > 包含块，就是元素用来进行定位参考的元素，具体是什么：可以使根元素，可以是position不是static的元素，等等

### 图像替换
  它是一类允许你用图像替换文本的技术，通过这种方式，文本仍然是可打印的，可访问的，但一般用在小型的，有限的应用上，诸如公司标志，页面标题，并不适合替换整段文本
  ```
  h1 {
    height: 140px;
    text-indent: -99999px;
    background: url('page-header.gif');
  }
  ```

  然后在打印样式中这么写，将内容移回来就好了
  ```
  h1 {text-indent: 0; background: none;}
  ```
  图像替换的技术有很多，大约有10几种
### 打印样式
  嵌入打印样式的方法有三种
  * <style type="text/css" media="print">...</style>
  * <link type="text/css" rel="stylesheet" media="print" href="print.css">
  * @import url(print.css) print;

### 