
### 前言

前段时间将《Three.js入门指南》匆匆的翻了一翻，算是对Three.js有了一个较为基本的印象，但脑海中对其中的一些概念还是没有什么理解，因此决定对这些知识都进行一个较为深入的理解与学习。首先便从几何体的基本构成开始吧。<br />
下面就是关于几何体对象的思维导图（如有错误敬请指正！）：

![](https://images.gitee.com/uploads/images/2019/0115/202632_47506164_1575229.png#align=left&display=inline&height=1863&originHeight=1863&originWidth=2793&status=done&width=2793)



### 一、几何体对象概述

在Three.js中，我们如果可以通过各式各样的API来构建一个几何体，下面是部分构建几何体的API，我们可以通过官方文档来进行查阅：

> [https://threejs.org/docs/index.html#manual/en/introduction/Creating-a-scene](https://threejs.org/docs/index.html#manual/en/introduction/Creating-a-scene)


- BoxBufferGeometry
- BoxGeometry
- CircleBufferGeometry
- CircleGeometry
- ConeBufferGeometry
- ConeGeometry
- CylinderBufferGeometry
- CylinderGeometry
- ...

很明显，如果使用There.js所提供的API构建集合体是非常方便的，比如这样：

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100);
```

上面的代码就可以让我们生成一个长、宽、高都为100的长方体，我们在后面所需做的就是将材质、灯光等属性设置完毕即可。这虽然简单便捷，但本质上就是一个黑盒，我们如果不加以深究，就只能在Three.js所封装好的API的基础进行开发，一旦有Three.js所提供的API所不能完成的需求出现时，就很可能无能为力。此外，如果对这些几何体的构成如果进行一些了解，也能有助于我们理解其几何体的本质。

通过上面那些封装好的对象的名字我们不难发现，大致的几何体类型有两大类：

1. BufferGeometry
2. Geometry

这两者的区别就在于 BufferGeometry 顶点数据是使用类型化数组来表示的，而Geometry的顶点数据则是用对象来表示的。



### 二、BufferGeometry

当我们使用 BufferGeometry 时，我们可以这么来声明几何体顶点数据：

```javascript
const geometry = new THREE.BufferGeometry()
// 使用类型数组来创建顶点数据
const vertices = new Float32Array([
  0, 0, 0, // 顶点1的坐标，下面分别是顶点2,3,4,5,6的坐标
  100, 0, 0, 
  0, 100, 0, 
  0, 0, 100, 
  0, 0, 100, 
  100, 0, 10, 
])
// 创建属性缓冲区对象，每3个为一组，分别表示一个顶点的xyz坐标
const attribue = new THREE.BufferAttribute(vertices, 3)
// 设置几何体attributes属性的位置属性
geometry.attributes.position = attribue
```

通过上面的代码我们就能够声明几何体对象顶点的位置属性了，同样的我们也能设置颜色属性：

```javascript
const colors = new Float32Array([
  0, 1, 0,
  1, 0, 0,
  0, 0, 1,
  0, 1, 1,
  1, 1, 0,
  1, 0, 1,
])
// 设置几何体attributes属性的颜色color属性，三个为一组
geometry.attributes.color = new THREE.BufferAttribute(colors, 3)
```

同样的法向量也是如此：

```javascript
const normals = new Float32Array([
  0, 0, 1,
  0, 0, 1,
  0, 0, 1,
  0, 1, 0,
  0, 1, 0,
  0, 1, 0,
])
// 设置几何体attributes属性的位置normal属性。三个为一组
geometry.attributes.normal = new THREE.BufferAttribute(normals, 3)
```

通过上面的示例我们不难看出，当我们使用BufferGeometry来声明一个几何体对象时，我们可以通过一个个类型化的数据来声明顶点的具体情况，具体情况可参阅官方文档：

> [https://threejs.org/docs/index.html#api/en/core/BufferGeometry](https://threejs.org/docs/index.html#api/en/core/BufferGeometry)


当然，这种申明方式既有优点又有缺点，优点就是实现了高度的可定制化，可以手动的设置每个顶点的属性，而缺点就在于一是手动设置麻烦，而是如果顶点过多，每个顶点都进行渲染的话会造成性能上的浪费，毕竟在绘制大的图形时，必定会有不少的顶点是重合的。为了减少这些缺点给项目带来的负面影响，Three.js又为我们提供了indedx。下面是官方文档对于index的描述：

> .index : （BufferAttribute）<br />
Allows for vertices to be re-used across multiple triangles; this is called using "indexed triangles" and works much the same as it does in Geometry: each triangle is associated with the indices of three vertices. This attribute therefore stores the index of each vertex for each triangular face. If this attribute is not set, the renderer assumes that each three contiguous positions represent a single triangle. Default is null.


具体的使用情况如下：

```javascript
const indexes = new Uint16Array([
  0, 2, 3, 1, 2, 3,
])
// 设置为一个为一组，则0代表上面属性的第一组
geometry.index = new THREE.BufferAttribute(indexes, 1)
```



### 三、Geometry

与BufferGeometry不同的是，Geometry是使用对象来描述顶点数据的就像这样：

```javascript
const geometry = new THREE.Geometry()
// 使用Vector3向量对象来表示顶点位置数据
const p1 = new THREE.Vector3(50, 60, 0)
const p2 = new THREE.Vector3(0, 70, 20)
const p3 = new THREE.Vector3(80, 70, 0)
// vertices是一个数组，因此我们只需将顶点push进去即可
geometry.vertices.push(p1, p2, p3)

// 使用Color对象来表示顶点颜色数据
const color1 = new THREE.Color(0x00ff00)
const color2 = new THREE.Color(0x0000ff)
const color3 = new THREE.Color(0xff0000)
// 将顶点颜色数据添加到geometry对象
geometry.colors.push(color1, color2, color3)
```

这样是不是很简单呀，但需要注意的是，在Geometry这个积类中，顶点的颜色颜色属性只能作用于点材质与线材质的几何体，对mesh类型的几何体没有作用。因此当我们需要使用mesh类型的几何体时，我们就需要使用face3属性来帮我们实现颜色上的定义，此外face3的用法与BufferGeometry中的index也很相像，都运用了顶点复用的思想：

```javascript
const geometry = new THREE.Geometry()
// 使用Vector3向量对象来表示顶点位置数据
const p1 = new THREE.Vector3(50, 60, 0)
const p2 = new THREE.Vector3(0, 70, 20)
const p3 = new THREE.Vector3(80, 70, 0)
// vertices是一个数组，因此我们只需将顶点push进去即可
geometry.vertices.push(p1, p2, p3)

// 使用Color对象来表示顶点颜色数据
const color1 = new THREE.Color(0x00ff00)
const color2 = new THREE.Color(0x0000ff)
const color3 = new THREE.Color(0xff0000)
// 将顶点颜色数据添加到geometry对象
geometry.colors.push(color1, color2, color3)

// 使用Face3构造函数创建一个三角面
const face1 = new THREE.Face3(0, 1, 2);
//设置每个顶点的法向量
const n1 = new THREE.Vector3(0, 0, -1);
const n2 = new THREE.Vector3(0, 0, -1);
const n3 = new THREE.Vector3(0, 0, -1);
// 等于这样的：face1.normal = new THREE.Vector3(0, 0, -1)
face1.vertexNormals.push(n1, n2, n3);
// 设置三角面face1三个顶点的颜色
face1.vertexColors = [
  new THREE.Color(0xffff00),
  new THREE.Color(0xff00ff),
  new THREE.Color(0x00ffff),
]
// 若颜色一样只需这么声明：face1.color = new THREE.Color(0x00ff00);

geometry.faces.push(face1)
```

