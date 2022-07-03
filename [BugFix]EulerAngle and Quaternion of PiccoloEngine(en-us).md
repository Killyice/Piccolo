# Forward
This markdown file needs the support of `mathjax` to show math formulas.

All code references in this article refer to Piccolo updates before 31 June 2022.


<br/><br/>

# Information and Definitions
## Lecture of Piccolo `Lecture_08`

- [3D Orientation Math](https://www.bilibili.com/video/BV1jr4y1t7WR/?spm_id_from=333.788&vd_source=23c9b65234255c26883eb1b9d4d9745a&t=2907.0)

- [Euler Angle](https://www.bilibili.com/video/BV1jr4y1t7WR/?spm_id_from=333.788&vd_source=23c9b65234255c26883eb1b9d4d9745a&t=3133.0)

- [Euler Angle to Quaternion](https://www.bilibili.com/video/BV1jr4y1t7WR/?spm_id_from=333.788&vd_source=23c9b65234255c26883eb1b9d4d9745a&t=4014.7)

## Coordinate System Definition

1. In the [Slideshow](https://games-1312234642.cos.ap-guangzhou.myqcloud.com/course/GAMES104/GAMES104_Lecture08.pdf), the coordinate system defination of `Euler Angle` is $\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{red}X^+(forward)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{green}Y^+(right)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}Z^+(bottom)\end{array}\right.$，belongs to the `Right-handed Coordinate System`

2. But the coordinate system defination of `Piccolo Engine`(subject to UI display) is $\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{green}Y^+(forward)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{red}X^+(right)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}Z^+(top)\end{array}\right.$ . It's `Right-handed Coordinate System` too.

3. Also in the [Slideshow](https://games-1312234642.cos.ap-guangzhou.myqcloud.com/course/GAMES104/GAMES104_Lecture08.pdf), for `Euler Angle to Quaternion`：is as same as the `Piccolo Engine`.

P.S. The direction of Piccolo-Robot is along the negative Y axis. It's uncommon but not the point.

<br/><br/>

# How to test
1. Select the 'Piccolo-Robot' or the 'Cube' behind it (anything without rotation is ok, because we need to exclude the influence of Euler Angle rotation order).
2. Click the `Rotate` in the menubar `Game Engine` to switch to the 'Rotate Mode'
3. Rotate the object along each axis, and you will see that the `ring` is perpendicular to the `axis`.
4. Then rotate the `X/Y/Z Slider of <TransformComponent>`, and you'll find out what went wrong(for example, you rotate the `X slider` and the object should rotate around the `X axis`, but ... the result is `Z axis`).

<br/><br/>

# Search for the problems
I find the defination of slider in the `m_editor_ui_creator["Transform"]` of [editor_ui](https://github.com/BoomingTech/Piccolo/blob/64c422eff6cba7af7292f567f6bdef3d83d1e55b/engine/source/editor/source/editor_ui.cpp#L84-L115). The code logic is "Quaternion → Euler Angle → Quaternion"。

In the "Quaternion → Euler Angle" code segment (L84~L86), The angle that was converted from the quaternion was stored in the `degrees_val`. The corresponding relationship is
$\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{red}R_x(\alpha)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{green}R_y(\beta)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}R_z(\gamma)\end{array}\right.$

Thus, the code is according to the defination of coordinate system of the `Euler Angle` on page 47 of the Slideshow, but is not consistent with the coordinate system of UI in this engine.


<br/><br/>

# Fix

We know that the order of Euler Angle of Piccolo is `Z,Y,X` (P55 of Slideshow)，so the quaternion $q=\color{red}q_x \color{green}q_y \color{blue}q_z$

In the engine, the defination of Euler Angle should be:
$\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{green}R_y(\beta)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{red}R_x(\alpha)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}R_z(\gamma)\end{array}\right.$ . It not matches the code from line 84 to line 86.

<hr>

Now we change the `degrees_val` in [editor_ui](https://github.com/BoomingTech/Piccolo/blob/64c422eff6cba7af7292f567f6bdef3d83d1e55b/engine/source/editor/source/editor_ui.cpp#L84-L115), and make sure the relationship of "Quaternion" and "Euler Angle" is related to the coordinate system：

$\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{red}R_x(\alpha)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{green}R_y(\beta)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}R_z(\gamma)\end{array}\right.$ $\Rightarrow$ $\left\{\begin{array}{lc}\color{pink}\phi(Roll)&\Rightarrow&\color{green}R_y(\beta)\\\\\color{lime}\theta(Pitch)&\Rightarrow&\color{red}R_x(\alpha)\\\\\color{teal}\Psi(Yaw)&\Rightarrow&\color{blue}R_z(\gamma)\end{array}\right.$，so we just swap `x` and `y`, and the `degrees_val` is all right.

we also need to swap the `x` and `y` in the trigonometric function of code segment(L92-L115), otherwise the values of the two will be confused.

<hr>

Ok! Let's run the engine. Oh! We find a new BUG: `y` and `z` are reversed.

So the problem is due to the "Euler Angles → Quaternion" code segment because we have fix the `degrees_val` before.

Just do as below：

1. Swap `Quaternion::getYaw(bool)` and `Quaternion::getRoll(bool)` in `PiccoloRuntime → quaternion.cpp`
2. Swap `y` and `z` in the trigonometric function of code segment(L92-L115) of [PiccoloEditor → editor_ui.cpp](https://github.com/BoomingTech/Piccolo/blob/64c422eff6cba7af7292f567f6bdef3d83d1e55b/engine/source/editor/source/editor_ui.cpp#L84-L115)

Let's run the engine again.

The BUG has been fixed.

<hr>

BUT! There is a new little feature: 

In this engine, with the increase of $R_y$ , the object is rotating counterclockwise if we view from the positive half axis of the Y axis to the negative half one. That not matches the `illustration` in the Slideshow.

Such as the `3DOrientation Math`(P46), especially the `Euler Angle to Quaternion`(P55). They are all the opposite of the phenomenon above.

If we check out the `Illustration` and the `Rotation Matrix`, we'll see that all the illustration do not match the rotation matrix $R_y(\beta)=\begin{bmatrix}\cos(\beta)&0&\sin(\beta)\\\\0&1&0\\\\-\sin(\beta)&0&\cos(\beta)\end{bmatrix}$. But the code follows the rotation matrix.
- Test：Take any `vector v` from `the positive half-axis of the X-axis`, and rotate 90° along the Y axis with this rotation matrix. So we get $R_y(\beta)\cdot v =\begin{bmatrix}\cos(\beta)&0&\sin(\beta)\\\\0&1&0\\\\-\sin(\beta)&0&\cos(\beta)\end{bmatrix}\begin{bmatrix}x\\\\0\\\\0\end{bmatrix}=\begin{bmatrix}0\\\\0\\\\-x\end{bmatrix}，(x>0)$. We find out that the `v` goes from `X+` to `Z-`，and that not match one of the `v is on Z+` and `the rotation of v matches the illustration`.
- And then we test the `X` and `Z`，they are both right.

Thus we have a conclusion, the direction of $R_y$ is reversed，should be $R_y(\beta)=\begin{bmatrix}\cos(\beta)&0&-\sin(\beta)\\\\0&1&0\\\\\sin(\beta)&0&\cos(\beta)\end{bmatrix}$，so...

we have TWO solutions，
1. We simply think that the Slideshow illustration is wrong, and the actual rotation direction should be according to the rotation matrix of Y-axis(I prefer this one...🥳 We don't have to change any other code)
2. Modify ALL the CODE associated with $R_y$ 😵
