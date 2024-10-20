# 图形学基础-线性代数知识

## 向量(Vector)

在图形学中，向量一般表示为列矩阵，即
$$
\vec{a}=\begin{pmatrix}
x_a \\ y_a \\ z_a
\end{pmatrix}
$$

这样更有利于和矩阵做乘法进行线性变换。

### 运算：点乘(Dot Product)

-  $\vec{a}·\vec{b}= x_ax_b + y_ay_b+z_a z_b$

- $Vector ~ Multiply ~ in ~ Matrix ~ form:$

- $$
  \vec{a}·\vec{b}=\begin{pmatrix}x_a~y_a~z_a\end{pmatrix}\begin{pmatrix}
  x_b\\y_b\\z_b
  \end{pmatrix}
  $$

- 点乘用来计算投影、且有助于用点乘来判断两个向量的方向有多靠近、判断前后。（角度离的越大，点乘积从正变为0再到负值）
- $Projection: \vec{p} = (\vec{p}·\vec{u})\vec{u}+ (\vec{p}·\vec{v})\vec{v}+ (\vec{p}·\vec{w})\vec{w}$

### 运算：叉乘(Cross Product)

- $\vec{a}\times\vec{b}=-\vec{b}\times\vec{a}~;~||\vec{a}\times\vec{b}||=||\vec{a}||||\vec{b}||sin\theta$

- $Vector ~ Multiply ~ in ~ Matrix ~ form:$

- $$
  \vec{a}\times\vec{b}=\begin{pmatrix}
  0 ~ {-z_a} ~ y_a \\
  z_a ~ 0 ~ {-x_a} \\
  {-y_a} ~ x_a ~ 0
  \end{pmatrix}\begin{pmatrix}
  x_b\\y_b\\z_b
  \end{pmatrix}
  $$

- 叉乘有助于根据正负判断：

  - 一个向量在另一个向量的左侧还是右侧；
  - 一个点在一个多边形的里面还是外面；（eg.如果点$P$在多边形$A_1...A_N$的内部，则所有$\vec{A_i A_{i+1}\times\vec{A_iP}}$正负均一致，反之则在外部。）
  - ↑这一例子的一个使用场景就是着色时判断三角面覆盖了屏幕的哪些像素。

  

## 矩阵(Matrix)



- 转置矩阵($Transpose$):$A^T;(AB)^T=A^T B^T$
- 单位矩阵($Identity$):$I=\begin{pmatrix} 1 ~ 0 ~ 0 ~ ...\\0~ 1~0~...\\0~0~1~...\\....... \end{pmatrix}$
- 逆矩阵:$AA^{-1}=A^{-1}A=I;(AB)^{-1}=B^{-1}A^{-1}$

### 运算：乘法

- 不符合交换律：$AB≠BA$
- 符合结合律、分配律：$(AB)C=A(BC);A(B+C)=AB+AC;(A+B)C=AC+BC$

## 2D变换(Transform)

变换分为**线性变换**和**非线性变换**，我i们可以利用矩阵相乘将线性变换施加到点$(x,y)^T$上：
$$
线性变换\begin{cases}Rotation 旋转 
\begin{pmatrix} x' \\ y' \end{pmatrix} = \begin{pmatrix} cos\theta ~ {-}sin\theta \\ sin\theta ~~~ cos\theta \end{pmatrix}\begin{pmatrix}x \\ y \end{pmatrix}
\\(ps.旋转永远绕圆点逆时针旋转)
\\Scale 缩放
\begin{pmatrix} x' \\ y' \end{pmatrix} = \begin{pmatrix} s_x~~ 0 \\ 0 ~~ x_y \end{pmatrix}\begin{pmatrix}x \\ y \end{pmatrix}
\\Shear 切变
\end{cases}
$$
`ps:`旋转矩阵是一个**正交矩阵**，即：
$$
R_{\theta}=\begin{pmatrix} cos\theta ~ {-}sin\theta \\ sin\theta ~~~ cos\theta \end{pmatrix}
\\R_{-\theta}=\begin{pmatrix} cos\theta ~~~ sin\theta \\ {-}sin\theta ~~ cos\theta \end{pmatrix}
\\有:R_\theta ^{-1}=R_{-\theta}=R_\theta ^{T}; 转置矩阵=逆矩阵
$$


其中线性变换可以组合相乘，达到一样的效果。
$$
A_n(...(A_2(A_1(X))))=A_n...A_2·A_1·\begin{pmatrix}x \\ y\end{pmatrix}
$$
而非线性变换只有**平移**：
$$
\begin{pmatrix} x' \\ y' \end{pmatrix} = \begin{pmatrix}x \\ y \end{pmatrix} +\begin{pmatrix}t_x \\ t_y \end{pmatrix}
$$
二者可以组合为**仿射变换**：
$$
AffineMap=LinearMap+Translation
\\
\begin{pmatrix} x' \\ y' \end{pmatrix} = \begin{pmatrix} a~~b \\ c~~d \end{pmatrix} +\begin{pmatrix}t_x \\ t_y \end{pmatrix}
$$
为了统一化这些变换，选择升维增加一个坐标$w$:

- 对于二维的点表示为：$(x,y,1)^T$ 二维的向量：$(x,y,0)^T$.

  - $(x,y,w)^T$代表2维的点$(x/w,y/w,1)^T.$

  - 为什么向量的$w=0$？因为向量满足平移不变性。且如果用w来代表对应的点P、向量V，他们满足以下对应：
    $$
    V+V=V ~~0+0=0
    \\P-P=V ~~ 1-1=0
    \\P+V=P ~~ 1+0=1
    $$

- 这样，非线性平移就可以表示为$Matrix~ Representation$：

$$
\begin{pmatrix} x' \\ y'\\1 \end{pmatrix} = \begin{pmatrix}1~~0~~t_x
\\0~~1~~t_y
\\0~~0~~1
\end{pmatrix} \begin{pmatrix}x\\y\\1 \end{pmatrix}=\begin{pmatrix}x+t_x\\y+t_y\\1 \end{pmatrix}
$$

- 故仿射变换为：
  $$
  \begin{pmatrix} x' \\ y'\\1 \end{pmatrix} 
  \longleftrightarrows^{M}_{M^{-1}}
  \begin{pmatrix}s_xcos\theta~~{-sin\theta}~~t_x
  \\sin\theta~~~~s_ycos\theta~~t_y
  \\0~~~~~~~~~0~~~~~~~~1
  \end{pmatrix} \begin{pmatrix}x\\y\\1 \end{pmatrix}
  $$
  遵循的顺序为：先线性变换、再平移变换。

  先旋转后平移和先平移后旋转结果不一样，即不满足交换律。$Order ~Matters!$

  同理，有：
  $$
  A_n(...(A_2(A_1(X))))=A_n...A_2·A_1·\begin{pmatrix}x \\ y\\1\end{pmatrix}
  $$

- 

![image-20241013163518502](C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241013163518502.png)

## 3D变换(Transform)

- 和2D类似，我们有：

- 对于三维的点表示为：$(x,y,z,1)^T$ 三维的向量：$(x,y,z,0)^T$.

  - $(x,y,z,w)^T$代表3维的点$(x/w,y/w,z/w,1)^T.$

- 仿射变换为：
  $$
  \begin{pmatrix} x' \\ y'\\z'\\1 \end{pmatrix} 
  \longleftrightarrows^{M}_{M^{-1}}
  \begin{pmatrix}a~~b~~c~~t_x
  \\d~~e~~f~~t_y
  \\g~~h~~i~~y_z
  \\0~~0~~0~~1
  \end{pmatrix} \begin{pmatrix}x\\y\\z\\1 \end{pmatrix}
  $$

- 其中，缩放矩阵为：
  $$
  \begin{pmatrix}a~~b~~c\\d~~e~~f\\g~~h~~i\end{pmatrix}=\begin{pmatrix}s_x ~~0~~0
  \\0 ~~s_y~~0
  \\0~~0~~s_z\end{pmatrix}
  $$
  而变换矩阵则为那三个$t_x,t_y,t_z.$

- 旋转比较复杂，可以分为三种：
  $$
  旋转\begin{cases}绕x轴旋转:\begin{pmatrix}a~~b~~c\\d~~e~~f\\g~~h~~i\end{pmatrix}=
  \begin{pmatrix}
  1~~~~~~~~0~~~~~~~~0
  \\0~\cos\theta~{-}\sin\theta
  \\0 ~~\sin\theta~~\cos\theta
  \end{pmatrix}
  \\
  \\绕y轴旋转:\begin{pmatrix}a~~b~~c\\d~~e~~f\\g~~h~~i\end{pmatrix}=
  \begin{pmatrix}
  \cos\theta~~0~~\sin\theta
  \\0~~~~~~~~1~~~~~~~~0
  \\{-}\sin\theta~~0~~\cos\theta
  \end{pmatrix}
  \\
  \\绕z轴旋转:\begin{pmatrix}a~~b~~c\\d~~e~~f\\g~~h~~i\end{pmatrix}=
  \begin{pmatrix}
  \cos\theta ~~ {-}\sin\theta ~~ 0
  \\\sin\theta ~~ \cos\theta ~~ 0
  \\0~~~~~~~~0~~~~~~~~1
  \end{pmatrix}
  
  \end{cases}
  $$

- 为什么绕y轴与众不同？因为$\vec{x}=\vec{y}\times \vec{z},\vec{z}=\vec{x}\times \vec{y},但\vec{y}=\vec{z}\times \vec{x}$.

- 定义欧拉角$R_{xyz}(\alpha,\beta,\gamma)=R_x(\alpha)R_y(\beta)R_z(\gamma)$,三个旋转轴俗称$roll,pitch和yaw.$

- $Rodrigue's$旋转公式：绕坐标轴n旋转α度：
  $$
  R(n,\alpha)=cos(\alpha)I+(1-cos(\alpha))nn^T+sin\alpha\begin{pmatrix}
  0~{-n_z}~~n_y
  \\n_z~~0~~{-}n_x
  \\{-}n_y~n_x~~0
  \end{pmatrix}
  $$

- 假设我们有一个单位向量 $(\mathbf{u} = (u_x, u_y, u_z))$ 表示旋转轴，以及一个旋转角度$ (\theta)$。Rodrigues公式可以用来计算一个向量$ \(\mathbf{v}\) $绕旋转轴 $(\mathbf{u}) $旋转 $(\theta) $角度后的新向量 \($\mathbf{v}'$\)。

  Rodrigues公式的具体形式如下：
  $$
  [ \mathbf{v}' = \mathbf{v} \cos\theta + (\mathbf{u} \times \mathbf{v}) \sin\theta + \mathbf{u} (\mathbf{u} \cdot \mathbf{v}) (1 - \cos\theta) ]
  $$
  其中：
  \- \($\mathbf{v}$\) 是原始向量。
  \- \($\mathbf{v}'$\) 是旋转后的向量。
  \- \($\mathbf{u}$\) 是旋转轴的单位向量。
  \- \($\theta$\) 是旋转角度。
  \- \($\times$\) 表示向量叉积。
  \- \($\cdot$\) 表示向量点积。

  这个公式的推导基于向量的旋转和线性代数的基本原理。它将旋转分解为三个部分：

  1. **原始向量的投影**：\($\mathbf{v} \cos\theta$\) 是原始向量在旋转平面上的投影。
  2. **垂直于旋转轴的分量**：\($(\mathbf{u} \times \mathbf{v}) \sin\theta$\) 是原始向量垂直于旋转轴的分量，经过旋转后产生的分量。
  3. **沿旋转轴的分量**：\($\mathbf{u} (\mathbf{u} \cdot \mathbf{v}) (1 - \cos\theta)$\) 是原始向量沿旋转轴的分量，经过旋转后保持不变。

- 四元数：待补充
