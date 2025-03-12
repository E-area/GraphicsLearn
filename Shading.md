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

### Barycentric Coordinate

前面讲到，逐像素着色的时候需要对顶点进行插值；那么如何插值？我们引入**重心坐标**`Barycentric Coordinate`：

我们通过重心坐标来计算平滑过渡的插值，插值的内容可以是坐标、颜色、法线向量等。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105155919140.png" alt="image-20241105155919140" style="zoom: 33%;" />

为了计算权重，我们可以使用面积比来计算重心坐标：

显而易见，当$\alpha=\beta=\gamma=\frac{1}{3}$时，这个点就是三角形的重心。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105160403808.png" alt="image-20241105160403808" style="zoom: 33%;" />

由于已知矢量就可以求得三角形的面积，我们化简之后得到如下公式：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105160625572.png" alt="image-20241105160625572" style="zoom:33%;" />

于是我们就可以在计算插值时利用每个点的重心坐标了：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105160810265.png" alt="image-20241105160810265" style="zoom:33%;" />

ps，重心坐标没有`投影不变性`，所以最好先在三维里插值再后投影。

### Texture Mapping

我们知道了求插值的方法，就可以把贴图的每一个纹素`(texel)，纹理贴图的每一个像素`与屏幕的像素联系上了。

我们把得到的`texcolor`拿来当`diffuse`里反射率`albedo的Kd`漫反射系数，就可以给物体上贴图了。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105161328806.png" alt="image-20241105161328806" style="zoom:33%;" />

但是这样就会带来一个问题：贴图过大或者过小时采样的问题。

### Texture Magnification

当贴图过小时，可能好多像素共用了一个纹素。我们有下列几种解决办法：

- `Nearest`：找到每个像素最近的纹素，四舍五入，把那个纹素的颜色值赋给当前像素，十分暴力。
- `Bilinear`：双线性插值。把每个像素最近的四个纹素拿来做插值，如下图所示： 

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105161747669.png" alt="image-20241105161747669" style="zoom:33%;" />

引入线性插值`Linear Interpolate , Lerp`并分别从水平和数值做两个插值，即可根据四个纹素距离当前像素的远近作为权重，求一个平均插值给当前的像素着色。这样就综合考虑了像素周围的四个纹素，效果更好一些。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105162219155.png" alt="image-20241105162219155" style="zoom:33%;" />

当然，还有一种方法：

- `Bicubic`：双立方插值，取16个纹素插值，每次用四个纹素做三次插值，效果更好但运算量较大。

三种优化的对比参见下图：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105162610830.png" alt="image-20241105162610830" style="zoom: 50%;" />

当贴图过大时，也会遇见问题，比如下图，出现了锯齿和摩尔纹的走样：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105162859214.png" alt="image-20241105162859214" style="zoom:33%;" />

究其本质，是远处一个像素覆盖了过多纹素。随着距离的拉远，一个像素覆盖的纹素也越来越多，如图：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105163006500.png" alt="image-20241105163006500" style="zoom:33%;" />

如果再运用超采样来抗锯齿，则开销会过于巨大。对此，我们有一种解决方法：不采样，而：

- 如果再深挖一层，就会发现这其实是点查询`Point Query`VS范围查询`Range Query`.

- `Mipmap`：一种**快速的，近似的，正方形的**范围查询方法。

具体方法是生成各种分辨率的不同层数的图，俗称图像金字塔：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\MipMap_Example_STS101.jpg" alt="img" style="zoom: 50%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105163758481.png" alt="image-20241105163758481" style="zoom:33%;" />

由于新图的分辨率以平方指数递减，所以额外储存这样一个金字塔只多占用了原分辨率三分之一的大小。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105163934559.png" alt="image-20241105163934559" style="zoom:33%;" />

计算Level D的`Mipmap`的方法：把屏幕空间中一个像素点和相邻的三个像素对应的点映射到贴图UV上，

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105164430912.png" alt="image-20241105164430912" style="zoom:33%;" />

并用如下偏导公式求出这个像素(浅红色区域)覆盖的范围对应UV图上的区域的【近似正方形】的边长。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105164536342.png" alt="image-20241105164536342" style="zoom:33%;" />

求出来的这个L就是该像素对应贴图UV区域的近似正方形的边长了。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105164629916.png" alt="image-20241105164629916" style="zoom:33%;" />

$D=\log_2{L}$就可以得到，$L\times L$的区域一定在第$D$层上对应**一个像素**的大小。

直接把这个像素拿过来赋给原像素，就非常快速的完成了着色。效果如下图所示，颜色越红说明一个像素覆盖的纹素越小，D就越小；

颜色越蓝说明一个像素覆盖的纹素越多，D就越大。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105165159412.png" alt="image-20241105165159412" style="zoom:33%;" />

但是我们可以看见：这个着色不连续，每个层之间是*离散*的。

但我们当然知道如何对付不连续：插值！

- `Trilinear`：三线性插值，这样就可以算出第k层$k \in R$了。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105165334032.png" alt="image-20241105165334032" style="zoom:33%;" />

如此一来，一次查询就可以得到一个像素覆盖的面积了，现在看起来就非常顺滑了：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105165609405.png" alt="image-20241105165609405" style="zoom:33%;" />

然而Mipmap也有限制，比如会出现一个问题：`Overblur`，如下图所示。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105165651761.png" alt="image-20241105165651761" style="zoom:33%;" /><img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105165807389.png" alt="image-20241105165807389" style="zoom:33%;" />

解决方法是：`Anisotropic Filtering`各向异性过滤。

我们要知道，对于这张图来说一个像素覆盖的并不都是正方形，也有的像素覆盖的是长方形区域的纹素：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105170104045.png" alt="image-20241105170104045" style="zoom:33%;" />

所以各向异性过滤,即`Ripmap`，它同样有层数，但额外储存了长方形的情况，这种预计算就可以进行矩形的范围查询了！

当然也有代价，那就是要额外占用三倍的储存开销，而Mipmap只占三分之一。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241105170141091.png" alt="image-20241105170141091" style="zoom:33%;" />

还有一种方法叫EWA过滤，但是多次查询代价就过大，不再赘述。

