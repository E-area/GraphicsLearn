# Viewing Transform

我们这样定义一个相机：

- $Position: \vec e$
- $Gaze~direction :\widehat g$

- $Up~direction:\widehat t$

在以摄像机为原点的坐标系里，默认约定俗成$up ~at~ Y,gaze ~at~-Z,\vec e在Origin点$.

我们通过操作$M_{view}$把坐标系从世界转换到以摄像机为原点的坐标系(视图空间)。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241014204627756.png" alt="image-20241014204627756" style="zoom:50%;" />
$$
M_{view}=R_{view}T_{view}
\\Step ~1.~Translate~\vec e ~ to ~ Origin.
\\T_{view}=\begin{bmatrix}
1&0&0&{-x_e}
\\0& 1& 0& {-y_2}
\\0& 0& 1& {-z_e}
\\0&0&0&1
\end{bmatrix}
\\Step ~2.~Rotate~\widehat g ~ to ~ -Z,\widehat t ~ to ~ X.
\\R_{view}=\begin{bmatrix}
x_{\widehat g \times \widehat t}&y_{\widehat g \times \widehat t}&z_{\widehat g \times \widehat t}&0
\\x_{\widehat t}& y_{\widehat t}&z_{\widehat t}& 0
\\x_{-\widehat e}& y_{-\widehat e}&z_{-\widehat e}& 0
\\0&0&0&1
\end{bmatrix}
\\
$$

## 投影变换 Project Transform

投影变换在视口变换之前，分为两种：正交投影和透视投影。

正交投影一般被应用在工程制图中，特点是平行线永远平行。

透视投影则是近似于人眼成像，近大远小，平行线会相交。($Parallel~ lines ~converge ~at ~single~ point.$)

投影变换可以把视图空间(`view space`)的坐标转换为裁剪空间（`clip space`）坐标.

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241015154134263.png" alt="image-20241015154134263" style="zoom: 67%;" />

### 正交投影 Orthegraphic Projection

通俗理解正交投影的顺序：

- $Step ~ 1.Drop~Z~coordinate$；第一步：将Z坐标轴压扁
- $Step~2.Translate~and~scale~the ~Resulting~Rectangle~to[-1,1]^2$：第二步：将二维长方形挤压且正则化到[-1,1]的二维平面。

严谨一点：$map~a~cuboid([left,right]\times[bottom,top]\times[far,near])~to~a~canonical~cube~[-1,1]^3.$

即把长方体映射到正则立方体上；正则化即为$Normalize.$

`ps`:在坐标数值大小上，$far<near.$因为摄像机看的方向是-Z；这也是为什么OpenGL使用的是左手坐标系。

正交投影的矩阵：
$$
M_{ortho}=
\begin{bmatrix}
\frac{2}{r-l} & 0&0&0
\\0&\frac{2}{t-b}&0&0
\\0&0&\frac{2}{n-f}&0
\\0&0&0&1
\end{bmatrix}
\begin{bmatrix}
1&0&0&-\frac{r+l}{2}\\
0&1&0&-\frac{t+b}{2}\\
0&0&1&-\frac{n+f}{2}\\
0&0&0&1
\end{bmatrix}
$$


### 透视投影 Perspective Projection

- `回忆`：$(x,y,z,1),(kx,ky,kz,k),(xz,yz,z^2,z)$均代表同一个点：$(x,y,z)in~3D.$

- `Step 1. Squish the frustum into a cuboid.` 把锥体压缩成长方体；($M_{persp\rightarrow ortho}$)

- `Step 2. Do orthographic Projection .`对变换后的长方体做正交投影。($M_{ortho},already~known$)

  <img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241015160605560.png" alt="image-20241015160605560" style="zoom:67%;" />

- 这样，经过第一部的变换加上正交投影的结果最终得到的就是透视投影了！

- 如图，$y'=\frac{n}{z}y,x'=\frac{n}{x}x,z'=?$

  <img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241015161200875.png" alt="image-20241015161200875" style="zoom: 50%;" />

- 利用`回忆`的内容，我们知道$(nx/z,ny/z,?,1)==(nx,ny,z*?,z)$,加上近平面、远平面、以及中点三者坐标不会有变化；

- 加以推算得到$M_{persp\rightarrow ortho}$：
  $$
  M_{persp\rightarrow ortho}=\begin{bmatrix}
  n&0&0&0\\
  0&n&0&0\\
  0&0&n+f&-nf\\
  0&0&1&0
  \end{bmatrix}
  $$

- 最后，$M_{persp}=M_{ortho}M_{persp\rightarrow ortho}.$

- ```cs
  // 对象空间 -> 世界空间
  float4 worldPos = mul(unity_ObjectToWorld, objPos);
  
  // 世界空间 -> 视图空间
  float4 viewPos = mul(unity_WorldToView, worldPos);
  
  // 视图空间 -> 裁剪空间（通过投影矩阵）
  float4 clipPos = mul(unity_ViewToProjection, viewPos);
  
  ```



