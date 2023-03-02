
### 一、WebGL基本概念

> WebGL (Web Graphics Library) is a JavaScript API for rendering interactive 3D and 2D graphics within any compatible web browser without the use of plug-ins. WebGL does so by introducing an API that closely conforms to OpenGL ES 2.0 that can be used in HTML5 canvas elements.  --MDN


以上是MDN对于WebGL的描述，简单来说，WebGL 就是一组基于 JavaScript 语言的图形规范，浏览器厂商按照这组规范进行实现，为 Web 开发者提供一套3D图形相关的 API。

我们可以通过这些API直接使用JavaScript直接和GPU进行通信，从而实现一些非常炫酷的图形。而webGL是在GPU上运行的，因此我们需要使用能够在GPU上运行的代码，首先我们需要一种叫做GLSL的语言，它是一种和C or CPP类似的强类型的语言，所以写起来很麻烦（这也是很多人吐槽WebGL的一个方面），其次这样的代码需要提供成对的方法，每对方法中，一个叫做顶点着色器，一个叫做片段着色器，这样的每一对组合起来就称作一个program(着色程序)。其中顶点着色器的作用是计算顶点的位置，根据计算出来的一系列的顶点的位置，WebGL就可以对点、线以及三角形在内的一些图元进行光栅化处理。当对这些图元进行光栅化处理的时候就需要使用片段着色器方法了，它的作用是计算出当前绘制图元中的每个像素的颜色值。



#### 1.1 什么是GLSL

上面我们提到了GLSL，其中文的意思是OpenGL着色语言，它是用来在 OpenGL 编写着色器程序的语言，全称为 OpenGL Shading Language。而着色器程序则是在GPU上运行的简短的程序，代替了GPU固定渲染管线的一部分，使GPU渲染过程中的某些部分允许开发者通过编程进行控制。

而GPU渲染过程中具体允许我们对其进行控制的部分有以下几个方面：

- JavaScript程序，处理着色器所需要的顶点坐标、法向量、颜色、纹理等。
- 顶点着色器，接受JavaScript传递过来的顶点信息，将顶点绘制到对应的坐标。
- 图元装配阶段，将三个顶点装配成指定的图元类型。
- 光栅化阶段，将三角形内部区域用空像素进行填充。
- 片元着色器，为三角形内部的像素填充颜色信息。



#### 1.2 WebGL工作流程

上面对WebGL的基本情况进行了一个简单的概述，但好像也没有解答webGL将3D模型显示到屏幕上的工作原理及流程。其实这个过程就好比富士康工作流水一样，按照既定的工作流程来对原材料进行加工，从而生产出完整的产品。WebGL大致也是如此，按照工作流水线的方式，将3D的模型数据渲染到2D屏幕上的，这个渲染方式的过程一般被称之为图形管线或者渲染管线。

上面我们又说到过点、线、三角形这些基本图元，但我们经常看见很多通过WebGL所绘制出来的诸如球体、圆柱、各式的立方体等模型，也看见了很多炫酷、复杂的模型，很显然这些并不属于这些基本图元里面，但其实这些模型本质上都是有一个个顶点组成的，GPU将这些点用三角线图元绘制成一个个微小的小平面，然后通过这些小平面的互相连接，来组成各种各样的的立体模型。因此通常来说，我们首先要做的就是创建组成模型的顶点数据。

一般情况下，最初的顶点坐标是相对于模型中心的，我们需要对顶点坐标按照一系列步骤执行模型转换、视图转换、投影转换，在通过这一系列的转换后的坐标叫做裁剪空间坐标，这个坐标才是WebGL可以接受的坐标。我们把最后的变换矩阵和原始顶点坐标传递给GPU，GPU的渲染管线然后对他们执行流水工作，主要过程如下：

1. 进入顶点着色器，利用GPU的并行计算优势对顶点逐个进行坐标变换。
2. 进入图元装配阶段，将会顶点按照图元类型组装成图形
3. 进入光栅化阶段，光栅化阶段对图像用不包含颜色信息的像素进行填充
4. 进入着色器阶段，为像素着色，并最终显示在屏幕上



### 二、WebGL初体验

上面将WebGL的大致情况进行了描述，下面就是真刀实枪的来搞事情了，和Three.js一样（这是废话。），我们在使用WebGL进行开发的时候首先需要使用canvas，我们可以再HTML文件里的这样声明一个canvas。顺便对浏览器对canvas的支持情况进行一个检查：

```html
<body onload="main()">
  <canvas id="glcanvas" width="640" height="480">
    Your browser doesn't appear to support the HTML5 <code>&lt;canvas&gt;</code> element.
  </canvas>
</body>
```

webGL应用主要包含两个要素：JavaScript程序和着色器程序。首先让我们来准备着色器程序，使用GLSL编写顶点着色器和片元着色器。

顶点着色器的任务我们在上面已经说了，它主要是告诉GPU我们所要形成的图形在裁剪坐标系的位置，下面这个代码就是告诉GPU我们需要在裁剪坐标系的原点，即屏幕中心画一个大小为20的点：

```javascript
void main(){
    //声明顶点位置
    gl_Position = vec4(0.0, 0.0, 0.0, 1.0)
    //声明所需绘制的点的大小
    gl_PointSize = 20.0
}
```

当顶点着色器中的数据经过图元装配和光栅化之后，来到了片元着色器，从而通过片元着色器将像素渲染成我们所需要的颜色：

```javascript
void main(){
    //设置像素的填充颜色为红色。
    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0) 
}
```

在这里，gl_Position、gl_PointSize、gl_FragColor 是 GLSL 的内置属性：

- gl_Position：顶点的裁剪坐标系坐标，包含X、Y、Z、W四个坐标分量。顶点着色器接收坐标后，会对它进行透视除法，即将各个分量同时除以 W，从而转换成 NDC 坐标，NDC 坐标每个分量的取值范围都在（-1, 1）之间，GPU 获取这个属性值作为顶点的最终位置进行绘制。
- gl_FragColor：片元（像素）颜色，包含 R, G, B, A 四个颜色分量，且每个分量的取值范围在（0,1）之间，GPU 会获取这个值，作为像素的最终颜色进行着色。
- gl_PointSize：绘制到屏幕的点的大小，gl_PointSize只有在绘制图元是点的时候才会生效。

然后我们就可以着手写我们的JavaScript部分的代码了，首先我们需要获取webGL的绘图环境：

```javascript
const canvas = document.querySelector('#canvas')
const gl = canvas.getContext('webgl')
```

然后创建顶点着色器：

```javascript
// 获取顶点着色器源码
const vertexShaderSource = document.querySelector('#vertexShader').innerHTML
// 创建顶点着色器对象
const vertexShader = gl.createShader(gl.VERTEX_SHADER)
// 将源码分配给顶点着色器对象
gl.shaderSource(vertexShader, vertexShaderSource)
// 编译顶点着色器程序
gl.compileShader(vertexShader)
```

再就是创建片元着色器：

```javascript
// 获取片元着色器源码
const fragmentShaderSource = document.querySelector('#fragmentShader').innerHTML
// 创建片元着色器程序
const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER)
// 将源码分配给片元着色器对象
gl.shaderSource(fragmentShader, fragmentShaderSource)
// 编译片元着色器
gl.compileShader(fragmentShader)
```

以上就将我们的着色器对象创建完成了，接下来我们就可以创建着色器程序了：

```javascript
//创建着色器程序
const program = gl.createProgram()
//将顶点着色器挂载在着色器程序上。
gl.attachShader(program, vertexShader)
//将片元着色器挂载在着色器程序上。
gl.attachShader(program, fragmentShader)
//链接着色器程序
gl.linkProgram(program)
```

我们在进行webgl开发的时候，可能会在一个WebGL应用里包含多个program，因此我们在使用某狗program绘制前，要先启用它，才能进行绘制：

```javascript
gl.useProgram(program)
// 绘制
gl.clearColor(0.0, 0.0, 0.0, 1.0)
gl.clear(gl.COLOR_BUFFER_BIT)
gl.drawArrays(gl.POINTS, 0, 1)
```

如此我们完成了我们的第一个webGL代码了，效果如下：<br />
![](https://upload-images.jianshu.io/upload_images/4116027-a3e4b13954be668d.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240#align=left&display=inline&height=602&originHeight=602&originWidth=803&status=done&width=803)<br />
完整代码如下：

```javascript
<body onload="main()">
	<!-- 顶点着色器源码 -->
	<script type="shader-source" id="vertexShader">
	 void main(){
  		//声明顶点位置
  		gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
  		//声明要绘制的点的大小。
  		gl_PointSize = 10.0;
  	}
	</script>
	
	<!-- 片元着色器源码 -->
	<script type="shader-source" id="fragmentShader">
	 void main(){
	 	//设置像素颜色为红色
		gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0); 
	}
	</script>
	<canvas id="canvas" width="640" height="480">
	Your browser doesn't appear to support the HTML5 <code>&lt;canvas&gt;</code> element.
	</canvas>
<script type="text/javascript">
function main() {
	// 获取webGL的绘图环境
	const canvas = document.querySelector("#canvas")
  	const gl = canvas.getContext("webgl")
	// 创建顶点着色器
	const vertexShaderSource = document.querySelector('#vertexShader').innerHTML
	const vertexShader = gl.createShader(gl.VERTEX_SHADER)
	gl.shaderSource(vertexShader, vertexShaderSource)
	gl.compileShader(vertexShader)
	// 创建片元着色器
	const fragmentShaderSource = document.querySelector('#fragmentShader').innerHTML
	const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER)
	gl.shaderSource(fragmentShader, fragmentShaderSource)
	gl.compileShader(fragmentShader)
	// 创建着色器程序
	const program = gl.createProgram()
	gl.attachShader(program, vertexShader)
	gl.attachShader(program, fragmentShader)
	gl.linkProgram(program)

	gl.useProgram(program)
	// 绘制
	gl.clearColor(0.0, 0.0, 0.0, 1.0)
	gl.clear(gl.COLOR_BUFFER_BIT)
	gl.drawArrays(gl.POINTS, 0, 1)
}
</script>
</body>
```

参考资料：

- [https://webglfundamentals.org/webgl/lessons/zh_cn/webgl-fundamentals.html](https://webglfundamentals.org/webgl/lessons/zh_cn/webgl-fundamentals.html)
- [https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API/Tutorial/Getting_started_with_WebGL](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API/Tutorial/Getting_started_with_WebGL)
- [https://juejin.im/book/5baaf635f265da0ab915cc9f/section/5baaf635e51d450e7125482f](https://juejin.im/book/5baaf635f265da0ab915cc9f/section/5baaf635e51d450e7125482f)
