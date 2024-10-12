# 图形学基础-线性代数知识

## 向量

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

- 点乘用来计算投影、且有助于用点乘来判断两个向量的方向有多靠近。（角度离的越大，点乘积从正变为0再到负值）
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

  

## 矩阵



- 转置矩阵($Transpose$):$A^T;(AB)^T=A^T B^T$
- 单位矩阵($Identity$):$I=\begin{pmatrix} 1 ~ 0 ~ 0 ~ ...\\0~ 1~0~...\\0~0~1~...\\....... \end{pmatrix}$
- 逆矩阵:$AA^{-1}=A^{-1}A=I;(AB)^{-1}=B^{-1}A^{-1}$

### 运算：乘法

- 不符合交换律：$AB≠BA$
- 符合结合律、分配律：$(AB)C=A(BC);A(B+C)=AB+AC;(A+B)C=AC+BC$
