---
title: Unity SpriteRender和Image的区别
tags: Unity引擎组件
notebook: Unity
---

### Image组件

#### 渲染

---

* 基于CanvasRender和Image组件进行渲染
* 默认材质渲染队列为Transparent Geometry，开启模板测试，渲染顺序是基于Canvas的Order In Layer层级，由Canvas逻辑进行管理
* 默认的渲染网格为正方形，更多的片元着色器操作
* 可以实现基于图集打包和网格数据合并，在此Canvas下实现渲染批处理

#### 交互

---

##### 检测逻辑

* 根据Unity层级面板顺序把Image组件加入队列，通过UGUI的EventSystem的Update来每帧遍历所有Canvas下需要检测的对象。通过Image组件的IsRaycastLocationValid方法来检测是否能穿透点击区域，中间的测试包含渲染图片的Alpha值，以及距形区域的交点测试。可以通过开关组件上的Raycast Target选项来控制是否需要响应对应的检测逻辑。

##### 事件响应

* 可以通过挂载UGUI自身继承IEventSystemHandler接口的组件，对相应组件添加事件来实现事件逻辑的响应，如Button,Toggle

#### 使用

---

通常应用于UI界面，由于是UGUI自身的组件，提供了更多UI组件交互支持，而且Image组件统一交由Canvas进行管理，渲染顺序可以通过Unity检视面板层级直接看到，方便开发。完美的UI自适应方案，可以通过Canvas Scaler来实现渲染分辨率大小的自适应，以及基于RectTransform锚点位置来控制Image组件渲染网格的大小和位置。还有交互逻辑是通过二维的交点测试，性能更加优化。

### 总结

---

* 完美适配UI界面的开发，当然，因为他就是UGUI的组件。 = =。

### SpriteRender组件

#### 渲染

---

* 基于SpriteRender组件进行渲染
* 默认材质渲染队列为Transparent Geometry，渲染顺序由所在渲染队列以及组件面板上的Order In Layer层级进行管理，如果在同一层级，则通过和摄像机的距离也就是Z轴顺序进行渲染
* 默认的渲染网格根据图片透明通道，无像素着色的地方会被裁切掉，形成不规则网格，更多的顶点着色器操作
* 可以实现基于图集打包和网格数据合并实现渲染批处理

#### 交互

---

##### 检测逻辑

* 为GameObject挂载BoxCollider2D，通过Unity自身的射线进行逻辑检测


##### 事件响应

* 通过给GameObject挂载BoxCollider2D,实现MonoBehaviour上的方法来实现响应逻辑，如OnMouseDown。当然这样的方式大概不够优雅，我们可以通过编写一个实现所有MonoBehaviour事件方法的EventListener类再挂载到GameObject上为相应的脚本添加事件

#### 使用

---

因为Mesh不会根据UI自适应进行拉伸，但是可以通过其Z轴来改变渲染顺序，通常应用于平面特效和3D场景中的平面物体。在UI画面中可以通过控制Order In Layer层级来控制渲染的层级，基于其默认的渲染队列，可以很好的和UI的层级区分开。在3D场景中，在和3D物体同一个渲染队列的情况下，可以很好的通过动画控制Z轴来实现与3D物体的遮挡关系。因为是Sprite所以可以实现基于图集和网格合并的批处理

### 总结

---

除了UI画面哪都好用，当然在你需要其自建网格和批处理渲染的情况下。