# 前言
此Markdown需要`mathjax`支持才能显示公式

本文代码数据均指2022年6月底前的Piccolo更新

<br/><br/>

# 资料与相关定义
## Piccolo的教学视频`Lecture_08`

- [3D旋转的数学原理](https://www.bilibili.com/video/BV1jr4y1t7WR/?spm_id_from=333.788&vd_source=23c9b65234255c26883eb1b9d4d9745a&t=2907.0)

- [欧拉角](https://www.bilibili.com/video/BV1jr4y1t7WR/?spm_id_from=333.788&vd_source=23c9b65234255c26883eb1b9d4d9745a&t=3133.0)

- [欧拉角转四元数](https://www.bilibili.com/video/BV1jr4y1t7WR/?spm_id_from=333.788&vd_source=23c9b65234255c26883eb1b9d4d9745a&t=4014.7)

## 坐标系定义

1. [PPT](https://games-1312234642.cos.ap-guangzhou.myqcloud.com/course/GAMES104/GAMES104_Lecture08.pdf)中`欧拉角`的坐标系定义是按照：$\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{red}X^+(面朝方向)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{green}Y^+(物体右侧)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}Z^+(物体下方)\end{array}\right.$，属于`右手坐标系`

2. `Piccolo引擎`的坐标系定义(以UI中显示的为准)：$\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{green}Y^+(面朝方向)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{red}X^+(物体右侧)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}Z^+(物体上方)\end{array}\right.$，同属`右手坐标系`。

3. [PPT](https://games-1312234642.cos.ap-guangzhou.myqcloud.com/course/GAMES104/GAMES104_Lecture08.pdf)中`欧拉角转四元组`的坐标系定义：与`Piccolo引擎`一致；

注：小机器人的朝向沿Y轴负方向，这很奇怪，但这不是重点

<br/><br/>

# 测试方式
1. 选择“Piccolo小机器人”，或“立方体”(只要`Transform → Rotate`归零的都行，要排除欧拉角顺序影响)；
2. 点击`Game Engine`菜单中的`Rotate`标签切换至旋转模式；
3. 分别沿`X(红)`,`Y(绿)`,`Z(蓝)`旋转`旋转环`，可以看到`旋转环`与`旋转轴`是垂直关系，没有问题；
4. 同样分别沿`X(红)`,`Y(绿)`,`Z(蓝)`的顺序依次拖动`<TransformComponent>的Rotation滑块`，可以看到其旋转轴与“步骤3”的不匹配，旋转轴均是前一个轴(如：本应沿X轴，却沿Z轴)。

<br/><br/>

# 查找问题
通过翻看代码发现在[editor_ui](https://github.com/BoomingTech/Piccolo/blob/64c422eff6cba7af7292f567f6bdef3d83d1e55b/engine/source/editor/source/editor_ui.cpp#L84-L115)中的`m_editor_ui_creator["Transform"]`进行了滑块相关的定义，其中用到了“四元数 → 欧拉角 → 四元数”。

在“四元数 → 欧拉角”代码片段(L84~L86)，使用`degrees_val`来接收转化后的欧拉角(角度制)，对应关系为：
$\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{red}R_x(\alpha)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{green}R_y(\beta)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}R_z(\gamma)\end{array}\right.$

即，引擎代码是以`PPT中欧拉角的坐标系`(P47)定义为准，但这和引擎`UI中显示的坐标系`相违背


<br/><br/>

# BUG修复

已知：Piccolo的欧拉角是按照`Z,Y,X`顺序旋转(参考PPT的P55)，即四元数$q=\color{red}q_x \color{green}q_y \color{blue}q_z$

在引擎的坐标系中，欧拉角的定义应为：
$\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{green}R_y(\beta)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{red}R_x(\alpha)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}R_z(\gamma)\end{array}\right.$，与代码不符

<hr>

现在在[editor_ui](https://github.com/BoomingTech/Piccolo/blob/64c422eff6cba7af7292f567f6bdef3d83d1e55b/engine/source/editor/source/editor_ui.cpp#L84-L115)中修改`degrees_val`，将“四元数—欧拉角”对应关系变成`坐标系相关`的：

$\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{red}R_x(\alpha)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{green}R_y(\beta)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}R_z(\gamma)\end{array}\right.$ $\Rightarrow$ $\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{green}R_y(\beta)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{red}R_x(\alpha)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}R_z(\gamma)\end{array}\right.$，也就是交换`x`和`y`，这样就排除了`degrees_val`的问题。

这时需要一并交换下方“欧拉角 → 四元数”代码片段(L92-L115)三角函数里的`x`和`y`，否则两者数值会不匹配

<hr>

运行引擎，会发现一个新BUG：`y`和`z`反了。

但是`degrees_val`我们已经确保没问题了，因此问题出现在“欧拉角 → 四元数”代码片段，

于是交换一下：

1. 在`PiccoloRuntime → quaternion.cpp`中，交换`Quaternion::getYaw(bool)`和`Quaternion::getRoll(bool)`
2. 在[PiccoloEditor → editor_ui.cpp](https://github.com/BoomingTech/Piccolo/blob/64c422eff6cba7af7292f567f6bdef3d83d1e55b/engine/source/editor/source/editor_ui.cpp#L84-L115)的`trans_ptr->m_rotation.?`四元数赋值语句中，交换三角函数里`degrees_val`的`y`和`z`

再次运行引擎，会发现旋转轴错位的BUG就修复了。

<hr>

但仔细看，实际上又出现了一个新的“小”Feature：在引擎里，当 $R_y$ 增长时，从Y轴由正向负看去是逆时针旋转的，这与PPT中的`图示`的旋转方向是不一样的

在PPT`3DOrientation Math`(P46)，尤其在`Euler Angle to Quaternion`(P55)中，图示中沿`Y`轴的旋转是由负向正方向看去为逆时针，
但是与旋转矩阵 $R_y(\beta)=\begin{bmatrix}\cos(\beta)&0&\sin(\beta)\\\\0&1&0\\\\-\sin(\beta)&0&\cos(\beta)\end{bmatrix}$ 不符，而引擎代码实现与PPT(P55)中一致。
- 验证：取任意`X轴正向`上`向量v`，用矩阵沿`Y轴`旋转90°，$R_y(\beta)\cdot v =\begin{bmatrix}\cos(\beta)&0&\sin(\beta)\\\\0&1&0\\\\-\sin(\beta)&0&\cos(\beta)\end{bmatrix}\begin{bmatrix}x\\\\0\\\\0\end{bmatrix}=\begin{bmatrix}0\\\\0\\\\-x\end{bmatrix}，(x>0)$，发现`v`从`x轴正向`变成了`z轴负向`，与图示中`z轴正向`和`沿Y轴旋转方向`之一冲突。
- 于是再分别验证沿`X`和`Z`旋转，发现都没问题(旋转矩阵均与图示一致)

因此得出结论，沿`Y轴`的旋转方向反了，应为$R_y(\beta)=\begin{bmatrix}\cos(\beta)&0&-\sin(\beta)\\\\0&1&0\\\\\sin(\beta)&0&\cos(\beta)\end{bmatrix}$，所以……

有两种解决方式，
1. 我们就单纯的认为PPT画错了，实际旋转方向应该是按旋转矩阵那样(反正我选这个……🥳)
2. 重新把四元数和 $R_y$ 相关的代码全改了😵
