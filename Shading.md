# Shading

## Visibility

### Z-buffering 深度缓冲

在渲染画面时，古老的方法是使用`Painter's Algorithm`画家算法：

- 先光栅化远处的物体，再光栅化近处的物体。
- 具体实现是对每一个物体的远近进行排序，时间复杂度`O(NlogN)`.
- 而且遇到下面这种情况时，渲染结果会出错，因为无法判断到底谁在最前面：



<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241027185302287.png" alt="image-20241027185302287" style="zoom:50%;" />

所以就有了我们现在采用的方法：Z-buffering 深度缓冲。

Z-buffer具体流程：

- 逐像素，对每一个采样的深度值z的最小值进行储存。

- 同步生成两个内容：

  - `Frame buffer`:储存像素的颜色值
  - `Depth buffer`:储存深度值。（近实远虚，z越小越近）

  >  我们知道z通常是负的，但这里的深度值z总视为正值，可以取个绝对值。

- 就会在渲染物体时生成下面两张图：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241027185653642.png" alt="image-20241027185653642" style="zoom:50%;" />

其中越黑越近代表深度值越小。具体实现过程如下：

- ```c++
  During Rasterization:
  - Initialize depth buffer to all infinity.
  - Then:
  for(each triangle T){
      for(each sample(x,y,z) in T){
          if(z < zBuffer[x,y]){
              frameBuffer[x,y]=rgb;
              zBuffer[x,y]=z;
          }
          else{
              ......
          }
      }
  }
  ```

- 实现图例如下（其中默认初始化为全部无穷）

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241027190218460.png" alt="image-20241027190218460" style="zoom:50%;" />

- 时间复杂度：`O(N) for N triangles`

> 深度缓存算法与顺序无关。
>
> 处理不了透明物体。

## Shading

我们在之前学习了，把世界坐标转换为相机坐标系、进而转化为屏幕坐标系，并对物体进行光栅化：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241027191228743.png" alt="image-20241027191228743" style="zoom:50%;" />

接下来学习着色。

>  定义：引入明暗、颜色的不同。
>
> 通俗理解:Applying a material to an object.

### Blinn-Phong光照模型

通常一个物理材质（PBR,Physically Based Rendering）的材质都分为三大部分：

- Specular 高光
- Diffuse 漫反射
- Ambient 环境光

着色对于模型来说是本地的（不会生成阴影！）

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241027192653073.png" alt="image-20241027192653073" style="zoom:50%;" />

我们需要的Input:

- View direction $\vec{v}$
- Surface normal $\vec{n}$
- Light direction $\vec{l}$
- Surface parameters(color,....)

其中前三个都是单位向量。

首先我们知道。反射光在一点会均匀分散：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241027193020574.png" alt="image-20241027193020574" style="zoom:50%;" />

且光照会随着照射到球壳面的范围扩大而衰减，如图：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241027194032953.png" alt="image-20241027194032953" style="zoom:50%;" />

然后我们有**兰伯特余弦定律**：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241027193603541.png" alt="image-20241027193603541" style="zoom:50%;" />

每个单位接收到的光的量正比于$\cos \theta = l · n$.

于是我们得到了一点的光照结果：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241027194111965.png" alt="image-20241027194111965" style="zoom:50%;" />

其中，`max(0,n·l)`的作用是把那些反方向照过来的点乘结果为负的光照剔除。

其中$k_d$是一个向量，在代表反射强度的同时也能表示颜色的RGB三维。

漫反射与观察方向无关。