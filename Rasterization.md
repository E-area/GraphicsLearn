# Rasterization(光栅化)

在经过$MVP(Model ~Transform,View~Transform,~Projection~Transform)$,（分别代表放置物体、放置相机和投影变换）之后，就改把我们得到的内容投在屏幕上了。

我们定义摄像机的视野$fovY:field~of~view~Vertical$，长宽比($Aspect~Ratio$)如图：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241016170433990.png" alt="image-20241016170433990" style="zoom:50%;" />

## Screen(屏幕)

定义屏幕$(Screen)$:一个由像素组成的二维数组。

- `Size of the array:`Resolution(分辨率).
- `A typical kind of raster display.`

- 光栅化($Rasterize$) 的过程就是`drawing things onto screen.`

对于`Screen Space:`

- `pixels' indices are (x,y) (both are intergers).`
- `pixel(x,y) is centered at (x+0.5,y+0.5).`
- `The screen is from (0,0) to (w-1,h-1),but it covers from (0,0) to (w,h).`

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241016171722991.png" alt="image-20241016171722991" style="zoom:50%;" />

- 从$[-1,1]^3$的`Projection Transform`到`Screen`的视口变换:

  - 1.`Irrelevant to Z`;
  - 2.`transform [-1,1]² to [0,w]×[0,h].`

  $$
  M_{viewport}=\begin{bmatrix}
  \frac{width}{2}&0&0&\frac{width}{2}\\
  0&\frac{height}{2}&0&\frac{height}{2}\\
  0&0&0&0\\
  0&0&0&1
  
  \end{bmatrix}
  $$

  

  

## Rasterlize Triangle Meshes

三角面的优势如下：

- `The most basic polygon.`
- 其他多边形都可以拆分成三角面。
- `Guaranteed to be planar.`必定是平面的，相比某些空间多边形。
- `Well defined interior.`定义清晰，方便插值。

对三角面进行采样操作(`Sampling`):

- ```cpp
  for(int x=0;x<xmax;x++){
      for(int y=0;y<ymax;y++){
          image[x][y]=inside(tri,x+0.5,y+0.5);
      }
  }
  ```

- 其中`inside`函数用来判断一个像素的中点坐标是否被该三角面包含。

- <img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241016192632889.png" alt="image-20241016192632889" style="zoom:50%;" />

- 你们该函数如何实现？在`Linear Algebra.md`中提到可以用矢量叉乘来判断一个点是否位于一个多边形内：

  >（eg.如果点$P$在多边形$A_1...A_N$的内部，则所有$\vec{A_i A_{i+1}\times\vec{A_iP}}$正负均一致，反之则在外部。）

- 这样，我们就能把一个三角形画到屏幕上啦！（至于颜色什么的，后来再说......)

- 

  <img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241016192657659.png" alt="image-20241016192657659" style="zoom:50%;" />

- 也有一些节约开支的算法，比如`AABB(Axis Aligned Bounding Box) `通过缩小范围到有内容的边界来加速；

- <img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241016193533730.png" alt="image-20241016193533730" style="zoom:50%;" />

- 再比如扫描线算法等，但都各有优劣。

- 在采样过程中，因为采样率不高会导致走样(`Aliasing`)，进而产生锯齿(`Jaggie`).















