---
layout: post
title: 自动驾驶之车道检测
date: 2018-06-24 15:42
status: public
tags: '自动驾驶, 机器视觉, 车道检测, Canny, Hough, OpenCV'
categories: [Machine-Learning]
---

![lane]({{ "/assets/img/machine-learning/lane.jpg" | absolute_url }})

车道检测是自动驾驶的基础工作。本文将简要讨论，如何一步一步实现路面上车道线的检测。

## 从最基础开始

在开始尝试各种复杂的算法之前，我们不妨先来想想，车道线通常会有哪些可以用来辨识的信息：

1. 颜色：通常是白色或者黄色，但是这两种颜色的物体并不一定是车道。
2. 位置：出现在路面上。而不是在天上。假设我们的摄像头放在车前方，得到上图的图像。那么车道线只能出现在下半区域。
3. 形状：是一条有一定宽度的直线或者稍带弧度的线。虽然可能是虚线，但不大会是三角形或者其他形状。同时斜率也是在一定范围内的。

如何利用这些信息呢：

1. 颜色：直接根据不同的颜色阈值筛选像素即可，也就是说把不可能是车道线颜色的像素都过滤除去。
2. 位置：同样的道理，首先划出感兴趣的区域 （Region of interest, ROI), 然后过滤掉之外的像素。
3. 形状：这里需要一些计算机视觉的算法了，下面会展开讲述

### Canny 边缘检测 

首先我们要找到图形的边缘地区，也就是梯度比较大（相对于图像内部）的区域。通常需要使用 Sobel 算子进行一个卷积，也就是

$$
G_x = \begin{bmatrix} -1 & 0 & +1 \\\ -1 & 0 & +1 \\\ -1 & 0 & +1 \end{bmatrix} \\\
G_y = \begin{bmatrix} -1 & -2 & -1 \\\ 0 & 0 & 0 \\\ -1 & +2 & +1 \end{bmatrix}
$$


$G_x$ 和 $G_y$ 分别 来计算 $x$ 和 $y$ 方向的梯度。之后便可以根据梯度的大小和方向进行过滤。为了避免噪声的影响，通常还需要线进行高斯滤波。

更多原理可以参考 [Wiki](https://en.wikipedia.org/wiki/Canny_edge_detector), 实现的话则有 [OpenCV](https://docs.opencv.org/3.1.0/da/d22/tutorial_py_canny.html)

### Hough 变换

现在我们有了很多可能是车道的点，下一个问题是，哪些才是真正的车道线呢？

在二维坐标中，一条不与 $y$ 轴平行的直线总可以表示为 $y = k x + b$ 的形式，而给定一个点 $(x_0, y_0)$, 也就确定了一组直线参数$(k, b)$。为了处理与 $y$ 轴平行的情况，通常我们会使用极坐标系。

在参数空间内，将所有点都表示出来，得到 (图来自 [Wiki](https://en.wikipedia.org/wiki/Hough_transform))

![Hough_space_plot_example]({{ "/assets/img/machine-learning/Hough_space_plot_example.png" | absolute_url }})

相较于同一个点的图就是处于同一条直线的。

使用时可以参考 [OpenCV](https://docs.opencv.org/2.4.13.4/doc/tutorials/imgproc/imgtrans/hough_lines/hough_lines.html ) 的文档。



## 更复杂的情况

### 相机的校准 （Camera Calibration ）

相机成像总会产生种种的畸变，如果不加以校准，就会对检测的结果产生影响。通常，可以用棋盘图对相机进行校准的操作。

用到的方法有

* cv2.findChessboardCorners：找到棋盘的格点
* cv2.calibrateCamera：对相机进行矫正
* cv2.undistort 使用矫正结果修复图像

### 视角变换

由于摄像头时向前方向的，根据透视的远离，原本平行的左右两条车道线在图像里变得不平行了。这也给图像的处理带了了难度。由于相机时处在固定位置，我们可以所有输入的图像先做一个伸展变换，让视角变成从上而下俯视的样子. Open CV 提供了 `cv2.warpPerspective ` 来实现这一变换。

### 弯道的处理

Hough 变换只对直线的情况做了处理。如果是一个曲率未知的弯道线呢？好在我们已经是俯视的视角了，可以很简单的把左右两条线划分出来。然后，对每一条都做一次多项式的拟合，就呢更加稳定计算出所需要的车道了。具体可以用 `cv2.fillPoly `



## 从图像到视频

在实际的应用中，输入的往往是一段视频。这就提供了更多的信息来处理。例如，我们可以假设，大多数情况，车道线不会突然的发生变化。因此，一旦某一帧的检测和之前大不相同，就说明出现了错误，需要进行额外的修正。

<video controls preload="auto" width="100%">
  <source src="{{ "/assets/video/project_video_result.mp4" | absolute_url }}" type="video/mp4">
</video>
