![](imgs/Pasted%20image%2020240515093458.png)
OpenCV（Open Source Computer Vision Library）是一个开源的计算机视觉和机器学习软件库。它由一系列的C函数和少量C++类构成，同时提供Python、Java和MATLAB等语言的接口，实现了图像处理和计算机视觉方面的很多通用算法。


## 历史





## 版本

#### opencv-python

只包含了主要模块的包

#### opencv-contrib-python

包含了主要模块以及扩展模块，扩展模块主要是包含了一些带专利的收费算法（如shift特征检测）以及一些在测试的新的算法（稳定后会合并到主要模块）。









1. `cv2.warpPerspective(src, M, dsize[, dst[, flags[, borderMode[, borderValue]]]])`
	- `src`：输入图像，通常是一个8位或32位浮点数的多通道图像。
	- `M`：透视变换矩阵，通常由cv2.getPerspectiveTransform函数计算得到。
	- `dsize`：输出图像的大小，格式为(宽度， 高度)。
	- `dst`：输出图像，与输入图像具有相同的类型和大小。如果设置为None，则将创建一个新图像。
	- `flags`：插值方法，默认为INTER_LINEAR。可选值有INTER_NEAREST、INTER_LINEAR、INTER_CUBIC、INTER_LANCZOS4等。
	- `borderMode`：边界处理模式，默认为BORDER_CONSTANT。可选值有BORDER_CONSTANT、BORDER_REPLICATE、BORDER_REFLECT等。
	- `borderValue`：边界填充值，当borderMode为BORDER_CONSTANT时有效。





3. 图像读取和显示
```python
import cv2

img = cv2.imread('image.jpg')

cv2.imshow('image', img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

