
### 前言

最近几天事情很多，学校在教育评估，不能在宿舍待着，因此笔记也就没做了。但对于Three.js的学习还算不错，将其基本的概念算是了解的差不多了。但总是要有成果的，于是经过一番思索总算是搞出了一个3D效果的正方体，算是对这几天不怎么有效率的学习的一个总结。


### 一、动态3D的生成

要实现一个3D动态效果，首先要做的就是先实现一个2D的效果（废话。。）。而使用Three.js来实现一个3D效动态果的正方体倒是不难，首先我们需要一个纯白的HTML：

```html
<html>
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>My 3D show</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script src="./three.min.js"></script>
</head>
<body>
    <script></script>
</body>
</html>
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Page Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script src="./three.min.js"></script>
</head>
<body>
    <script>
        
    </script>
</body>
</html>
```

然后我们需要做的就是将我们的JavaScript代码写在script标签内就行了。在Three.js中，有三个非常重要的东西：

- 渲染器（Renderer）
- 场景（Scene）
- 照相机（Camera）

用比较通俗的话来讲，场景就是我们摄影的时候的背景或场景，而照相机则是可以定义我们所拍摄的物体的形状之内的东西，而渲染器则负责将场景与照相机所拍出来的东西给渲染出来。正是这三者的合力，才能让我们产生一个3D效果。首先我们需要生成一个canvas：

```javascript
const w = window.innerWidth
const h = window.innerHeight
const renderer = new THREE.WebGLRenderer()
renderer.setSize(w, h)
renderer.setClearColor(0xb9d3ff, 1.1)
document.body.appendChild(renderer.domElement)
```

现在我们就在我们页面插入一个canvas了，这段代码倒是很好理解，先是获取window的宽与高，然后是由渲染器设定大小与背景颜色。需要注意的是，现在添加背景颜色并没有用，因为还有相关的光源没有设置。接下来我们要做的就是解决场景方面的问题：

```javascript
const scene = new THREE.Scene();
// 创建网格模型
const geometry = new THREE.BoxGeometry(80, 80, 80)
const material = new THREE.MeshLambertMaterial({
    color: 0x0000ff
})
const mesh = new THREE.Mesh(geometry, material)
scene.add(mesh)
// 光源设置
const point = new THREE.PointLight(0xffffff)
point.position.set(400, 200, 300)
scene.add(point)
const ambient = new THREE.AmbientLight(0x444444)
scene.add(ambient)
```

如上的代码就是关于场景的设置，我们可以通过Three.js已经封装好的相关方法来创建我们需要的正方体，同时也将我们所需要的光源等元素加入到scene中，使用add加入的元素都可以在scene的children中找到：

![](https://cdn.nlark.com/yuque/0/2019/png/296173/1570534162311-43b26572-86b7-4f9c-af03-a7b9458f53ee.png#align=left&display=inline&height=135&originHeight=135&originWidth=1302&size=0&status=done&width=1302)

搞定场景后，我们就需要搞定照相机了。在图形学中的照相机与我们日常生活中所定义的照相机是不一样的，在图形学中的照相机，它定义了我们所构造的三维空间到投影在显示屏上的二维空间的投影的方式。而照相机只是一种类比而已。

而根据投影方式的不同，照相机又有以下两种分类：

1. 正交投影照相机（Orthographic Camera）
2. 透视投影照相机

我们这次所使用的就是正交投影照相机：

```javascript
const k = w / h // 窗口宽高比
const s = 200 // 三维场景显示范围
const camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000)
camera.position.set(200, 300, 200)
camera.lookAt(scene.position)
```

到这时，我们已经完成了渲染器，场景，照相机的相关设置了，接下来就只需将使用渲染器将照相机与场景渲染出来即可：

```javascript
renderer.render(scene, camera)
```

我们就能得到一个静态的3D正方体了：

![](https://cdn.nlark.com/yuque/0/2019/png/296173/1570534162333-db994e83-6b8c-483b-a98b-2f4d29d7c9cf.png#align=left&display=inline&height=516&originHeight=516&originWidth=959&size=0&status=done&width=959)


### 二、动态设置

想让这个静态的3D图像动态化，其实思路就是多次执行render方法，从而覆盖之前的render放来，从而达到动态的效果。而多次执行render总的来说有以下两个方案：

- setInterval()：按照所设定的周期去执行函数。
- requestAnimationFrame()：请求浏览器执行下一帧动画。

使用setInterval()我们可以这样做：

```javascript
const myRender = () => {
    renderer.render(scene, camera)
    mesh.rotateY(0.01)
}
setInterval("myRender()", 25)
```

但在我的理解来看,setInterval()还是比较适合与时钟等时间间隔要求很严格的应用，而requestAnimationFrame()则适合动画等应用，因为它请求的是下一帧的动画，所以其动画也会较为流畅：

```javascript
const myRender = () => {
    renderer.render(scene, camera)
    mesh.rotateY(0.01)
    requestAnimationFrame(myRender)
}
myRender()
```

这样我们就能实现一个旋转的3D正方体了。全部的JavaScript代码如下：

```javascript
const w = window.innerWidth
const h = window.innerHeight
// 场景
const scene = new THREE.Scene();
// 创建网格模型
const geometry = new THREE.BoxGeometry(80, 80, 80)
const material = new THREE.MeshLambertMaterial({
    color: 0x0000ff
})
const mesh = new THREE.Mesh(geometry, material)
scene.add(mesh)
// 光源设置
const point = new THREE.PointLight(0xffffff)
point.position.set(400, 200, 300)
scene.add(point)
const ambient = new THREE.AmbientLight(0x444444)
scene.add(ambient)
console.log(scene.children)
// 相机
const k = w / h // 窗口宽高比
const s = 200 // 三维场景显示范围
const camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000)
camera.position.set(200, 300, 200)
camera.lookAt(scene.position)
// 渲染器
const renderer = new THREE.WebGLRenderer()
renderer.setSize(w, h)
renderer.setClearColor(0xb9d3ff, 1.1)
document.body.appendChild(renderer.domElement)
const myRender = () => {
renderer.render(scene, camera)
mesh.rotateY(0.01)
requestAnimationFrame(myRender)
}
myRender()
```
