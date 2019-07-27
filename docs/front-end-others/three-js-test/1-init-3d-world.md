---
title: 1. 初始化 3D 世界
---

Three.js 是一个 3D 库，该系列是在使用 three.js 中的一些过程和笔记。本文记录初始化 3D 世界的基本过程。

<!--more-->

# Three.js 基础

## 概要

- OpenGL
  OpenGL 是最常用的跨平台图形库。

- WebGL
  WebGL 是基于 OpenGL 设计的面向 web 的图形标准，提供了一系列 JavaScript API，通过这些 API 进行图形渲染将得以利用图形硬件从而获得较高性能。

- Three.js
  Three.js 是通过对 WebGL 接口的封装与简化而形成的一个易用的图形库。
  除了 WebGL 之外，Three.js 还提供了基于 Canvas、SVG 标签的渲染器。

## 基本概念

- 场景(Scene)
  场景是所有物体的容器，也对应着我们创建的三维世界。

- 照相机(Camera)
  照相机是三维世界中的观察者，为了观察这个世界，首先我们要描述空间中的位置。
  Three 中使用采用常见的右手坐标系定位。

- 物体(Object)
  创建物体需要指定几何形状和材质。
  其中，几何形状决定了物体的顶点位置等信息，材质决定了物体的颜色、纹理等信息。

- 光源(Light)
  Three 提供了包括环境光 AmbientLight、点光源 PointLight、 聚光灯 SpotLight、方向光 DirectionalLight、半球光 HemisphereLight 等多种光源。

- 着色器(Renderer)
  Renderer 绑定一个 canvas 对象，并可以设置大小，默认背景颜色等属性。
  调用 Renderer 的 render 函数，传入 scene 和 camera，就可以把图像渲染到 canvas 中了。

# 初始化场景

## 添加基本元素

我们创建一个 3D 世界的基本步骤如下：

1. 创建场景(Scene)
2. 创建照相机(Camera)
3. 创建着色器(Renderer)，绑定 canvas
4. 加载 renderer 并传入 scene 和 camera

## 创建场景

```javascript
// 设置场景
var scene = new THREE.Scene();
```

- 场景相关函数
  - `scene.add(obj)`: 在场景中添加物体
  - `scene.remove(obj)`: 在场景中移除物体
  - `scene.children()`: 获取场景中所有子对象的列表
  - `scene.getChildByName()`: 利用 name 属性，获取场景中某个特定的物体

## 创建照相机

照相机定义了三维空间到二维屏幕的投影方式：

- 透视投影照相机
  `THREE.OrthographicCamera(left, right, top, bottom, near, far)`
  获得的结果是类似人眼在真实世界中看到的有“近大远小”的效果。

- 正交投影照相机
  `THREE.PerspectiveCamera(fov, aspect, near, far)`
  对于在三维空间内平行的线，投影到二维空间中也一定是平行的。

```javascript
// 这里我们创建正交投影照相机
var camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
// 设置相机的坐标位置
camera.position.set(0, -5, 2);
// 设置相机的焦点
camera.lookAt(scene.position);
```

## 创建着色器

- 渲染
  将模型数据在屏幕上显示出来的过程。

- 着色器
  着色器可以用来渲染高级的效果。
  Three.js 可不定义着色器，采用默认的方法渲染。

```javascript
// 定义着色器
var renderer = new THREE.WebGLRenderer();
// 设置画布宽高
renderer.setSize(window.innerWidth, window.innerHeight);
// 添加到document中
document.body.appendChild(renderer.domElement);
```

这里，若我们在 html 中早已添加 canvas，则可通过定义时传入该 canvas 元素来初始化着色器：

```javascript
renderer = new THREE.WebGLRenderer({
  canvas: document.getElementById("canvas_id")
});
```

## 渲染

我们需要调用着色器的`render()`方法，同时传入 scene 和 camera 来进行画布渲染：

```javascript
// 渲染
renderer.render(scene, camera);
```

- requestAnimationFrame
  这个函数就是让浏览器去执行一次参数中的函数，就形成了我们通常所说的游戏循环了。

若我们需要场景动起来，则需要添加动画帧渲染：

```javascript
// 让世界动起来
function render() {
  requestAnimationFrame(render);

  // 此处可添加动画处理

  renderer.render(scene, camera);
}
render();
```

此时，我们的 3D 世界已初始化完成。
但目前我们无法看到效果，因为我们还未曾给该 3D 世界添加光源和物体，后面章节我们会陆续完善。

## 完整代码

```javascript
// 设置场景
var scene = new THREE.Scene();

// 创建正交投影照相机
var camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, -5, 2);
camera.lookAt(scene.position);

// 定义着色器
var renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 让世界动起来
function render() {
  requestAnimationFrame(render);

  // 此处可添加动画处理

  renderer.render(scene, camera);
}
render();
```

## 参考

- [《Three.js 入门指南》](http://www.ituring.com.cn/minibook/792)
- [《Three.js 快速入门教程》](http://www.jb51.net/article/92350.htm)

# 结束语

这节主要讲了 Three.js 中的一些基本概念，创建并初始化了一个 3D 世界。这里我们并不能看到效果，后面章节会逐步往这个世界中添加光源、物体和一些事件交互。
