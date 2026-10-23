
Three.js 是一个尽可能简化在网页端获取 3D 内容的库，使用 WebGL 来绘制三维效果。 WebGL 是一个只能画点、线和三角形的非常底层的系统。想要用 WebGL 来做一些实用的东西通常需要大量的代码， 这就是 Three.js 的用武之地。它封装了诸如场景、灯光、阴影、材质、贴图、空间运算等一系列功能，让你不必要再从底层 WebGL 开始写起。

![](_imgs/Pasted%20image%2020240624135628.png)




# 项目开发

## Python Flask

1. 项目结构
```
project/
├── app.py
├── templates/
│   └── index.html
├── src/
│   └── ...
```

2. `app.py`
```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(debug=True)
```

3. `index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Three.js with Flask</title>
    <style>
        body { margin: 0; }
        canvas { display: block; }
    </style>
</head>
<body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        // 创建场景
        const scene = new THREE.Scene();

        // 创建透视相机
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.z = 5;

        // 创建 WebGL 渲染器
        const renderer = new THREE.WebGLRenderer();
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // 创建一个立方体几何体
        const geometry = new THREE.BoxGeometry();
        // 创建一个简单的材质并设置颜色
        const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
        // 将几何体和材质结合到一个网格
        const cube = new THREE.Mesh(geometry, material);
        // 将网格添加到场景中
        scene.add(cube);

        // 创建一个动画循环函数
        function animate() {
            requestAnimationFrame(animate);

            // 在每一帧中旋转立方体
            cube.rotation.x += 0.01;
            cube.rotation.y += 0.01;

            // 渲染场景和相机
            renderer.render(scene, camera);
        }

        // 开始动画循环
        animate();
    </script>
</body>
</html>
```


## React

1. 项目结构
```
project/
├── node_modules/
├── public/
├── src/
│   ├── components/
│   │   └── ThreeScene.js
│   ├── App.css
│   ├── App.js
│   ├── index.css
│   ├── index.js
├── package.json
└── ...

```

2. `src/App.js`
```js
import React from 'react';
import './App.css';
import ThreeScene from './components/ThreeScene';

function App() {
    return (
        <div className="App">
            <ThreeScene />
        </div>
    );
}
export default App;
```

3. `src/App.css`
```css
.App {
    margin: 0;
    padding: 0;
    overflow: hidden;
    width: 100vw;
    height: 100vh;
}
```


4. `src/components/ThreeScene.js`
```js
import React, { useRef, useEffect } from 'react';
import * as THREE from 'three';

const ThreeScene = () => {
    const mountRef = useRef(null);

    useEffect(() => {
        // 创建场景
        const scene = new THREE.Scene();

        // 创建透视相机
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.z = 5;

        // 创建 WebGL 渲染器
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        mountRef.current.appendChild(renderer.domElement);

        // 创建一个立方体几何体
        const geometry = new THREE.BoxGeometry();
        // 创建一个简单的材质并设置颜色
        const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
        // 将几何体和材质结合到一个网格
        const cube = new THREE.Mesh(geometry, material);
        // 将网格添加到场景中
        scene.add(cube);

        // 动画循环函数
        const animate = () => {
            requestAnimationFrame(animate);

            // 在每一帧中旋转立方体
            cube.rotation.x += 0.01;
            cube.rotation.y += 0.01;

            // 渲染场景和相机
            renderer.render(scene, camera);
        };

        // 开始动画循环
        animate();

        // 在组件卸载时清理
        return () => {
            mountRef.current.removeChild(renderer.domElement);
            renderer.dispose();
        };
    }, []);

    return <div ref={mountRef} />;
};
export default ThreeScene;
```