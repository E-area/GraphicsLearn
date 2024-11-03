# Rasterization(光栅化)

在经过$MVP(Model ~Transform,View~Transform,~Projection~Transform)$,（分别代表放置物体、放置相机和投影变换）之后，就改把我们得到的内容投在屏幕上了。

我们定义摄像机的视野$fovY:field~of~view~Vertical$，长宽比($Aspect~Ratio$)如图：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241016170433990.png" alt="image-20241016170433990" style="zoom:50%;" />

## Screen Space(屏幕空间)

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

  


<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241103174440730.png" alt="image-20241103174440730" style="zoom:50%;" />

总结一下整个渲染空间变换流程：

1. **模型空间（Model Space）到世界空间（World Space）**：

   - 通过模型矩阵（Model Matrix）进行变换。每个模型都有自己的局部坐标系，这个变换将其坐标转换到整个场景的世界坐标系中。

   ```
   float4 worldPos = mul(unity_ObjectToWorld, objPos);
   ```

2. **世界空间到视图空间（View Space）**：

   - 通过视图矩阵（View Matrix）进行变换。这个变换将世界坐标转换到相机坐标系中，使得相机位于原点，且视线沿 -Z 轴方向。

   ```
   float4 viewPos = mul(unity_WorldToView, worldPos);
   ```

3. **视图空间到裁剪空间（Clip Space）**：

   - 通过投影矩阵（Projection Matrix）进行变换。这个步骤将视图空间的三维坐标转换为裁剪空间的齐次坐标（Clip Coordinates），范围通常在 [-1, 1] 之间，用于确定哪些顶点在视锥体内。

   ```
   float4 clipPos = mul(unity_ViewToProjection, viewPos);
   ```

4. **裁剪空间到归一化设备坐标（NDC）**：

   - 进行齐次除法，将裁剪空间坐标转换为归一化设备坐标（Normalized Device Coordinates，NDC）。这一步将齐次坐标的 `x`, `y`, `z` 除以 `w` 分量。

   ```
   float3 ndcPos = clipPos.xyz / clipPos.w;
   ```

5. **NDC到屏幕空间（Screen Space）**：

   - 通过视口变换（Viewport Transform）将归一化设备坐标映射到屏幕坐标系。这个步骤将 NDC 的范围 [−1, 1] 转换为屏幕坐标的像素位置。

   ```
   // 示意性代码，实际由图形硬件完成
   screenPos.x = ((ndcPos.x + 1) / 2) * screenWidth;
   screenPos.y = ((ndcPos.y + 1) / 2) * screenHeight;
   ```

## Rasterlizing Triangle Meshes

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

## Anti-Aliasing (反走样)

- 为什么会走样`Aliasing`？因为采样`Sampling`是不连续的，比如：动画是对时间的采样，你们帧率就类似于采样率。
- 一般的走样有以下几种：
  - `Jaggies`锯齿
  - `Moire Patterns`摩尔纹
  - `Wagon Wheel Illusion`车轮效应

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020190600103.png" alt="image-20241020190600103" style="zoom:50%;" />

- 那么如何反走样？有一种方法是：先进行模糊操作，再进行采样，这样出来的效果就会好一些。

  `Pre-Filter then Sample.`

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020190639399.png" alt="image-20241020190639399" style="zoom:50%;" />

这里就不得不提到**傅里叶展开**：任何一个函数都可以展开成一个常数加上一大堆三角函数的近似：

> 即正余弦是完备正交基。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020211719740.png" alt="image-20241020211719740" style="zoom:50%;" />

所以，任何一个函数都可以分解成许多不同的频率，我们将分解的过程称为**傅里叶变换**。这个过程为一个积分，此处不赘述。

将$f(x)$傅里叶变换`FT`后得到的$F(\omega)$，还可以逆傅里叶变换`Inv.FT`回去得到$F(x).$

下面是对于一个频域`Frequency Domain`进行采样的结果：可以看到，更高的频率需要更快的采样才会精确，否则很容易失真，这也是走样产生的本质。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020192223290.png" alt="image-20241020192223290" style="zoom:50%;" />

比如如图，高频率的蓝色与低频率的黑色两者按照图示采样得到的结果是一样的。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020192615699.png" alt="image-20241020192615699" style="zoom:50%;" />

对于一个时域，对其`FT`就可以得到其频域，反之`Inv.FT`得到时域。

比如这里对一个时域（图片）进行傅里叶变换得到的频域如图：（`FT`能让我们“看到”图片在频率里长什么样。）

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020192958132.png" alt="image-20241020192958132" style="zoom:50%;" />

如此集中在中间是因为很多自然图的低频信息很多，高频信息很少。

为什么呢？因为图像易分辨边缘的信号变化很大，而图像边缘占比很少，所以高频信息就很少。

- 滤波`(Filter)`是一种去掉一部分特定频率的信息的方法。
- 比如对上述图片`FT`之后的频域施加一个`高通滤波High-pass Filter`之后，只保留高频信息，再`Inv.FT`回去得到时域，会发现只剩下边缘：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020193224151.png" alt="image-20241020193224151" style="zoom:50%;" />

- 而对应的使用低通滤波之后，则会失去边缘变得很模糊：（没错，我们上面要的模糊方法就在这里！）

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020193426801.png" alt="image-20241020193426801" style="zoom:50%;" />

或者施加一个即去掉过低频率也去掉过高频率的滤波：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020193637930.png" alt="image-20241020193637930" style="zoom:50%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020193830621.png" alt="image-20241020193830621" style="zoom:50%;" />

- 为了方便理解，通常**滤波`Filter`=平均`Averaging`=卷积`Convolution`**.
- 卷积定理：时域的卷积等于频域的乘积。
- 即，时域上A应用了卷积核B，等价于频域上AB二者相乘。如图所示：

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020194352058.png" alt="image-20241020194352058" style="zoom:50%;" />

图上的卷积核也叫做`Box Filter`;

而下图展示了采样的过程：其中c图称为冲激函数，如之前所讲，$f(x)\times$冲激函数得到的就是采样结果。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020195525675.png" alt="image-20241020195525675" style="zoom:50%;" />

同时也知道了走样的原因：因为如上图所示，采样在频域看来就是重复原始信号的频谱罢了。

那么有没有办法解决走样？

- 增加采样率，当然这不是我们考虑的范畴，这并非反走样。
- `MSAA(Multi-Sampling Anti-Aliasing)`,多重采样抗锯齿，即先对图片卷积再采样。具体过程如下：
  - 把一个像素具体细分成`n*n`个点，分别根据每个点是否在三角面内来判断覆盖率，进而确定当前像素应该着色多少。
  - 这是一种增加样本数的方法，使用的正是上文提到的`Box Filter`.

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020202847888.png" alt="image-20241020202847888" style="zoom:50%;" />

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241020203420456.png" alt="image-20241020203420456" style="zoom:50%;" />

- `FXAA(Fast Approximate AA)`：快速近似抗锯齿
- `TAA(Temporal AA)`：时域抗锯齿，也是多重采样但是将样本分布在了时间是。
