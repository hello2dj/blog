![](http://taligarsiel.com/Projects/layers.png)

这里
![](http://taligarsiel.com/Projects/webkitflow.png)

![](http://taligarsiel.com/Projects/image025.png)

Each renderer represents a rectangular area usually corresponding to the node's CSS box, as described by the CSS2 spec. It contains geometric information like width, height and position. 
The box type is affected by the "display" style attribute that is relevant for the node (see the style computation section). Here is Webkit code for deciding what type of renderer should be created for a DOM node, according to the display attribute.

