从计算机图形学的角度来看，关于相机对象的东西有很多，在大的范围有经典观察以及计算机观察，在这两种观察下面又有很多细分的投影模式。而在Three.js中主要的投影方式则只有以下两种：

1. [正投影](https://zh.wikipedia.org/wiki/%E6%AD%A3%E6%8A%95%E5%BD%B1)
2. [透视投影](https://zh.wikipedia.org/wiki/%E9%80%8F%E8%A7%86%E6%8A%95%E5%BD%B1)

而相机拍照的本质也很简单，简单来说就是将三维的物体，人看到的物体相当于物体在一个方向上的投影，而我们就可以根据自身的需求来选择合适的投影方式：

- 正投影相机（OrthographicCamera）：适用于小的场景，比如产品的在线展示、机械、工业设计类三维建模软件模型的显示。

```javascript
THREE.OrthographicCamera(left, right, top, bottom, near, far)
```

- 透视投影相机（PerspectiveCamera）：适用于大的场景，比如游戏场景、场景实况演示等。

```javascript
THREE.PerspectiveCamera(fov, aspect, near, far)
```

其实他们两者最大的区别就在距离，正投影不会受到物体与相机距离的影响，而透视投影则会。下面是具体的代码演示：

```javascript
const scene = new THREE.Scene()
const geometry = new THREE.BoxGeometry(100, 100, 100)
const material = new THREE.MeshLambertMaterial({
  color: 0x0000ff
})
const mesh = new THREE.Mesh(geometry, material)
scene.add(mesh)
const point = new THREE.PointLight(0xffffff)
point.position.set(400, 200, 300)
scene.add(point)

const ambient = new THREE.AmbientLight(0x444444)
scene.add(ambient)

const width = window.innerWidth
const height = window.innerHeight

const camera = new THREE.PerspectiveCamera(60, width / height, 1, 1000) // 透视投影
//  const camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000) // 正投影
camera.position.set(200, 300, 200)
camera.lookAt(scene.position)

const renderer = new THREE.WebGLRenderer()
renderer.setSize(width, height)
renderer.setClearColor(0xb9d3ff, 1)
document.body.appendChild(renderer.domElement)

const render = () => {
    renderer.render(scene,camera)
    requestAnimationFrame(render)
}
render()

const controls = new THREE.OrbitControls(camera)
```

值的注意的是PerspectiveCamera或者是OrthographicCamera他们的父类是Camera，和mesh一样，Camera的父类也是Object3D，因此他们也就都能够继承Object3D的属性position，因此我们在设定我们所需要的投影相机后，就需要设定相机的摆放位置以及相机的朝向。


#### 窗口变化自适应渲染

窗口变化自适应渲染的用处很简单，就是用于适应不同窗口的大小的变化。其最重要的核心就在于实时读取窗口的变化，并将变化的数值体现于相机。在这种情况下我们就需啊哟使用[window.onresize](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onresizehttps://note.youdao.com/)。onresize 事件会在窗口被调整大小时发生，因此当我们需要在窗口大小发生变动时，来引发自适应渲染，在正投影中我们可以这么做：

```javascript
window.onresize = function(){
  // 重置渲染器输出的canvas的尺寸
  renderer.setSize(window.innerWidth, window.innerHeight)
  // 重置相机投影的相关参数
  k = window.innerWidth/window.innerHeight
  camera.left = - s * k
  camera.right = s * k
  camera.top = s
  camera.bottom = -s
  camera.updateProjectionMatrix ()
}
```

而在透视投影中，较之正投影要就简单一些，我们只需要重置长宽比aspect即可:

```javascript
window.onresize=function(){
  // 重置渲染器输出的canvas的尺寸
  renderer.setSize(window.innerWidth, window.innerHeight);
  // 全屏情况下：设置观察范围长宽比aspect为窗口宽高比
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix ();
}
```
