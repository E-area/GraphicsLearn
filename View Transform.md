# Viewing Transform

我们这样定义一个相机：

- $Position: \vec e$
- $Gaze~direction :\widehat g$

- $Up~direction:\widehat t$

在以摄像机为原点的坐标系里，默认约定俗成$up ~at~ Y,gaze ~at~-Z,\vec e在Origin点$.

我们通过操作$M_{view}$把坐标系从世界转换到以摄像机为原点的坐标系。

<img src="C:\Users\Terra233\Desktop\ComputerGraphicsLearn\Images\image-20241014204627756.png" alt="image-20241014204627756" style="zoom:50%;" />
$$
M_{view}=R_{view}T_{view}
\\Step ~1.~Translate~\vec e ~ to ~ Origin.
\\T_{view}=\begin{bmatrix}
1~0~0~{-x_e}
\\0~ 1~ 0~ {-y_2}
\\0~ 0~ 1~ {-z_e}
\\0~0~0~~~~~1
\end{bmatrix}
\\Step ~2.~Rotate~\widehat g ~ to ~ -Z,\widehat t ~ to ~ X.
\\R_{view}=\begin{bmatrix}
x_{\widehat g \times \widehat t}~y_{\widehat g \times \widehat t}~z_{\widehat g \times \widehat t}~0
\\x_{\widehat t}~~~ y_{\widehat t}~~~z_{\widehat t}~~~~~~~ 0
\\x_{-\widehat e}~~ y_{-\widehat e}~~z_{-\widehat e}~~ 0
\\0~~~~~0~~~~~0~~~~~~~1
\end{bmatrix}
\\
$$
