# web
1. 
  ### 现象
    * 单元格宽度无法设置
    * 表格内文本无法换行
  ### 原因
    * 单元格宽度无法设置是因为表格的布局属性
    ```
    table-layout: fixed | auomatic | inherit(从父元素继承)
    固定表格布局：在这个情况下是可以固定宽度设置的
      固定表格布局与自动表格布局相比，允许浏览器更快地对表格进行布局。
      在固定表格布局中，水平布局仅取决于表格宽度、列宽度、表格边框宽度、单元格间距，而与单元格的内容无关。
      通过使用固定表格布局，用户代理在接收到第一行后就可以显示表格。
    自动表格布局：
      在自动表格布局中，列的宽度是由列单元格中没有折行的最宽的内容设定的。
      此算法有时会较慢，这是由于它需要在确定最终的布局之前访问表格中所有的内容。
    ```
    * 这个问题是由于我把文本放到了一个div中而这个div的宽度被我设置的很窄，让我产生了，换行就成了一列的假象，应当把div的宽度设置为合适的
    * 换行的属性
    ```
    white-space: 处理文本中的空格
    word-break: normal	使用浏览器默认的换行规则。
                break-all	允许在单词内换行。
                keep-all	只能在半角空格或连字符处换行。
    word-wrap or overflow-wrap: normal 就是大家平常见得最多的正常的换行规则。
                                break-word 一行单词中实在没有其他靠谱的换行点的时候换行。
    ```

    [上述两个属性的解释](http://www.zhangxinxu.com/wordpress/2015/11/diff-word-break-break-all-word-wrap-break-word/)

  ### 解决
  ```
    table {
      table-layout: fixed;
      word-break: break-all;
    }
  ```