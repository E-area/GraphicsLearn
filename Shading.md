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

#### 漫反射部分

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

#### 高光部分

高光产生的条件是，当观察方向$\vec{v}$和光照在平面产生的镜面反射方向$\vec{R}$足够接近。如图：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029162243006.png" alt="image-20241029162243006" style="zoom:50%;" />

而Blinn-Phong经验模型告诉我们：

- $\vec{v}与\vec{R}接近 = 法线方向\vec{n}与半程向量\vec{h}接近。$

于是我们得到了高光的计算公式如图：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029162257252.png" alt="image-20241029162257252" style="zoom:50%;" />

> 使用v与R的模型称为Phong模型，而使用n与h的模型称为Blinn-Phong模型。

式中ks为镜面反射系数（含颜色），式中也考虑了光照的衰减问题。

p为为了避免高光面积过大而使用的指数系数，一般p越大越能体现两个向量越接近；一般取100-200.

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029162921066.png" alt="image-20241029162921066" style="zoom:50%;" />

关于ks和p对高光的影响可以参考下图：（该材质仅有diffuse和specular）

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029163108313.png" alt="image-20241029163108313" style="zoom:50%;" />

#### 环境光部分

我们认为（然而并非正确），任何一点接收到的环境光均相同。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029163213502.png" alt="image-20241029163213502" style="zoom:50%;" />

然后，把三者结合起来就是一个Blinn-Phong模型的材质了！

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029163447748.png" alt="image-20241029163447748" style="zoom:50%;" />

### Shading Frequency

除了着色方法，着色频率也十分重要。如图：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029163813709.png" alt="image-20241029163813709" style="zoom:50%;" />

上述三种分别是逐平面着色、逐顶点着色、逐像素着色。

#### 逐平面着色

使用三角形的两边做叉乘得到面的法线方向。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029164131515.png" alt="image-20241029164131515" style="zoom:50%;" />

#### 逐顶点着色

用三角形的面做插值求顶点法线。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029164240990.png" alt="image-20241029164240990" style="zoom:50%;" />

插值方法：

由于任何一个顶点都与几个面相联系，所以可以用周围面的法线进行求加权平均或平均来求得面的法线方向。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029164837645.png" alt="image-20241029164837645" style="zoom:50%;" />

#### 逐像素着色

（这里的Phong shading不是phong模型！只是同名）

对顶点插值求出。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029164323962.png" alt="image-20241029164323962" style="zoom:50%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029165153718.png" alt="image-20241029165153718" style="zoom:50%;" />

当然，除了着色频率外，模型本身的面数也有影响，如图：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029164446233.png" alt="image-20241029164446233" style="zoom:50%;" />

推荐网站:http://shadertoy.com/

## Rendering Pipeline

渲染管线如图所示：

（通常一个fragment可以类比成一个像素之类的单位元）

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029165614950.png" alt="image-20241029165614950" style="zoom:50%;" />

结合之前的所学内容：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029165813372.png" alt="image-20241029165813372" style="zoom:50%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029165828522.png" alt="image-20241029165828522" style="zoom:50%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029165844573.png" alt="image-20241029165844573" style="zoom: 50%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029165906478.png" alt="image-20241029165906478" style="zoom:50%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029170039731.png" alt="image-20241029170039731" style="zoom:50%;" />

## Texture Mapping

纹理的坐标使用uv表示：uv分别对应rgb中的rg，即u越大越绿色，v越大越红色。

- u和v默认的范围都是$(0,1)$.

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241029172225292.png" alt="image-20241029172225292" style="zoom:50%;" />

可以无缝衔接循环的纹理称为四方纹理（Tilable Texture）.





<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105155919140.png" alt="image-20241105155919140" style="zoom: 33%;" />



<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105160403808.png" alt="image-20241105160403808" style="zoom: 33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105160625572.png" alt="image-20241105160625572" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105160810265.png" alt="image-20241105160810265" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105161328806.png" alt="image-20241105161328806" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105161747669.png" alt="image-20241105161747669" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105162219155.png" alt="image-20241105162219155" style="zoom:33%;" />



<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105162610830.png" alt="image-20241105162610830" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105162859214.png" alt="image-20241105162859214" style="zoom:33%;" />



<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105163006500.png" alt="image-20241105163006500" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105163758481.png" alt="image-20241105163758481" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105163934559.png" alt="image-20241105163934559" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105164430912.png" alt="image-20241105164430912" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105164536342.png" alt="image-20241105164536342" style="zoom:33%;" />



<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105164629916.png" alt="image-20241105164629916" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105165159412.png" alt="image-20241105165159412" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105165334032.png" alt="image-20241105165334032" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105165609405.png" alt="image-20241105165609405" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105165651761.png" alt="image-20241105165651761" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105165807389.png" alt="image-20241105165807389" style="zoom:33%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105170104045.png" alt="image-20241105170104045" style="zoom:33%;" />



<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105170141091.png" alt="image-20241105170141091" style="zoom:33%;" />

