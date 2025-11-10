
[GLM](https://glm.g-truc.net/0.9.8/index.html)（Open**GL** **M**athematics）是一个易于使用，专门为OpenGL量身定做的数学库，**只有头文件的**，不用链接和编译，把头文件的根目录复制到你的**includes**文件夹，然后你就可以使用这个库了。




## 使用


**计算缩放矩阵**，
```C++
glm::mat4 trans;
trans = glm::scale(trans, glm::vec3(0.5, 0.5, 0.5));
```
功能：得到一个变换 trans，物体乘上这个变换就会其大小缩小到原来的一半。


**计算位移矩阵**，
```c++
glm::vec4 vec(1.0f, 0.0f, 0.0f, 1.0f);
glm::mat4 trans;
trans = glm::translate(trans, glm::vec3(1.0f, 1.0f, 0.0f));
vec = trans * vec;
```
功能：得到一个变换 trans，应用这个变换可以把一个向量 vec `(1, 0, 0)` 位移 `(1, 1, 0)` 个单位。


**计算旋转矩阵**，
```c++
glm::mat4 trans;
trans = glm::rotate(trans, glm::radians(90.0f), glm::vec3(0.0, 0.0, 1.0));
```
功能：得到一个变换 trans，应用这个变换可以把一个物体沿 z 轴方向顺时针 90°，最后一个参数表示旋转方向，其维度的数值不重要。 `glm::radians()` 是将角度转化为弧度。


**计算正交投影**，
```c++
glm::mat4 projection = glm::ortho(T left, T right, T bottom, T top, T zNear, T zFar);
```
功能：在参数中给出左、右、上、下、近、远定义出类似立方体的平截头体，![](../Render%20Engine/_imgs/Pasted%20image%2020240725224742.png)
使用正交投影矩阵变换至裁剪空间之后处于这个平截头体内的所有坐标将不会被裁剪掉，任何出现在近平面之前或远平面之后的坐标都会被裁剪掉。



**计算透视投影**，
```c++
glm::mat4 projection = glm::perspective(T fovy, T aspect, T near, T far);
```
功能：在参数中给出视野、宽高比、近、远平面定义出一个类似于没有锥尖的锥形平截头体![](../Render%20Engine/_imgs/Pasted%20image%2020240725225650.png)
任何在这个平截头体以外的东西最后都不会出现在裁剪空间体积内，并且将会受到裁剪。`fov` 即视野（Field of View），通常设置为 `45.0f` ；宽高比 `aspect` 由视口的宽除以高所得；近距离 `near` 一般设置为 `0.1f`，远距离 `far` 一般设置为 `100.0f`。



**计算观察矩阵**，
```c++
glm::mat4 view = glm::lookAt(eye, center, up);
```
功能：计算出一个观察矩阵