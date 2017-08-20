### 相机
  相机默认是朝向z负轴的，改变相机位置，并不改变镜头朝向，若不改变镜头朝向，则镜头仍然是朝向负轴的即使改变了相机位置（可以使用lookAt设置镜头朝向）
  * #### 正交相机(OrthographicCamera)（物体的大小比例不会随着观看距离而改变，即不会是近大远小的效果）
  signature: three.OrthographicCamera(left, right, top, bottom, near, far)
  这6个参数代表了相机拍摄到的空间的6个面的位置，这6个面正好围成一个长方体，称为视景体(视锥)（—Frustum—)
  只有在视锥内的物体才会显示在屏幕上，而视锥之外的物体会被裁剪掉。

  ![](http://rungame.me/images/2014/5/orthographic-camera.jpg)
  为了保持相机的横竖比例，需要保证right-left 与top-bottom的比值与canvas的横竖比值相同（很明显若不一致会产生视觉问题待贴图）
  * #### 透视投影相机（符合人眼近大远小）
  signature: three.PerspectiveCamera(fov, aspect(width/height), near, far)
  fov: 控制了上下的张角， aspect: 控制了水平的张角，near和far控制了Z轴纵深显示的范围

  ![](http://rungame.me/images/2014/5/perspective-camera.jpg)
  这4个参数代表了相机拍摄到的空间体，称为视景体(视锥)（—Frustum—)
  只有在视锥内的物体才会显示在屏幕上，而视锥之外的物体会被裁剪掉。

### 几何形状
  几何形状的主要作用是存储物体的顶点信息，通过指定几何形状的特征来创建例如球体，需要半径
  * #### 立方体（长方体）
  signature: three.CubeGeometry(width, height, depth, widthSegments, heightSegments, depthSegments);
  前三个是在x，y, z上的长度，后三个代表在三个轴上分段数（可不设，默认是1, ）真的是分段，把相应的长度分为指定段然后标明）

  * #### 平面（PlaneGeometry）
  长方形
  signature: three.PlaneGeometry(width, height, widhtSegments, heightSegments)

  * #### 球体（SphereGeometry)
  signature: three.SphereGeometry(radius, segmentsWdith, segmentsHeight, phiStart, phiLength, thetaStart, thetaLength)
  phiStart表示经度开始的弧度(画半球的利器)
  phiLength表示经度跨过得弧度
  thetaStart表示纬度开始的弧度
  thetaLength表示纬度跨过得弧度

  * #### 圆形 （CircleGeometry)
  signature: three.CircleGeometry(radius, segments, thetaStart, thetaLength)

  * #### 柱体 （CylinderGeometry)参见文档吧。
  可以做很多东如: 圆台，无顶面底面
    [https://threejs.org/docs/index.html#api/geometries/CylinderGeometry]()

  * #### 圆环面（TorusGeometry）
    [https://threejs.org/docs/index.html#api/geometries/TorusGeometry]()

  * #### 等等不在赘述参见文档使用啥查啥就好了
    [https://threejs.org/docs/index.html#manual/introduction/Creating-a-scene]()

  * #### 文字形状也可以使用三维渲染，但需要额外的字体库
    后续详述
  
  * #### 自定义形状，需要手动指定每个顶点的位置，若形状复杂，则计算量过大，此时可以使用3d Max建模后导入到threejs场景中

### Material 材质
  材质是独立与物体空间信息之外的渲染效果信息，通过材质可以改变物体的颜色，纹理贴图，光照模式等
  * #### 基本材质（BasicMaterial)
  使用基本材质渲染的物体颜色为纯色，不会由于光照产生敏感，阴影等效果，颜色若未指定则随机
  signature: three.MeshBasicMaterial(opt)
  opt可省略太多了，[见文档](https://threejs.org/docs/index.html#api/materials/MeshBasicMaterial),基础的有color颜色，opacity透明度等
  visible: 是否可见
  side: 渲染物体的正面还是反面（FrontSide or BackSide or DoubleSide）
  wireframe: 是否渲染线而非面
  map: 使用文理贴图
  如：创建一个不透明度为0.75的黄色材质

  * #### Lambert材质（MeshLambertMaterial)
  这是符合Lambert光照模型的材质，特点是只考虑光照的漫反射而不考虑镜面反射，对于金属，镜子等物体就不合适了