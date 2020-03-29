---
title: 点在凸包内检测
date: 2016-07-27 18:00:06
tags: [coding, computational geometry, convex hull]
---
不久遇到个具体问题，是如何判断一个point是否在一个rotatedRect内部。
找到两个网页给出了解答。

[第一个网页](http://www.gamedev.net/topic/142526-checking-if-a-point-is-inside-a-rotated-rectangle/)提供了多种解决方法。

方法一：把矩形转正，点的坐标也要变换从而保持相对位置不变，就转化为简单的x/y坐标是否在矩形内判断了。
>You can rotate the point by an inverse angle about the center of the rectangle given that you know by how much the rectangle was rotated. Then the test to see if the point is inside the axis-aligned rectangle becomes a trivial thing.

方法二：用目标点分别与矩形两个相邻节点组成三角形，比较这4个三角形的面积之和是否与矩形总面积（几乎）相等。不过这个方法的误差限epsilon可能要手调。
>If you are testing a rectangle ABCD with a point x, check if
area(ABx) + area(BCx) + area(CDx) + area(DAx) < area(ABCD) + epsilon
If so, you are inside. Otherwise you are outside. There are probably faster checks, but this is pretty simple conceptually and programming-wise. 

方法三，先不说是什么，看到这个简洁的代码我立马震惊。
>try this:

``` c
float ex,ey,fx,fy;

ex=bx-ax; ey=by-ay;
fx=dx-ax; fy=dy-ay;

if ((x-ax)*ex+(y-ay)*ey<0.0) return false;
if ((x-bx)*ex+(y-by)*ey>0.0) return false;
if ((x-ax)*fx+(y-ay)*fy<0.0) return false;
if ((x-dx)*fx+(y-dy)*fy>0.0) return false;

return true;
```
>Easy huh? Without rotation or drawing rectangles... I assume that a,b,c and d are really the four corners of a rectangle.
The test is just checking whether the point lies on the right side of each rectangle edge.

原来是Right Test！脑中远古的记忆又依稀了起来。什么是**Right Test**呢？它是计算几何里非常非常非常基本的一个方法。知名度就像考拉之于澳大利亚一样。

那么下面就结合[第二个网页](http://www.blackpawn.com/texts/pointinpoly/)理解Right Test吧！它讲的很详细。
例子是三角形ABC，但这个方法适用于所有凸多边形。
![p是否在ABC内](https://raw.githubusercontent.com/veslam/blog/master/res/20160727_01_RightTest.png)

怎么判断点p是否在三角形内呢？既然这个三角形是由三条边围成的，那如果相对于每条边，点p与第三个点在同侧，就能确定p在三角形内。

确定问题：怎么判断点p, C在线段AB同侧？用叉乘： [B-A] cross [p-A] 应该与 [B-A] cross [C-A] 同号。
``` python
function SameSide(p1,p2, a,b)
    cp1 = CrossProduct(b-a, p1-a)
    cp2 = CrossProduct(b-a, p2-a)
    if DotProduct(cp1, cp2) >= 0 then return true
    else return false

function PointInTriangle(p, a,b,c)
    if SameSide(p,a, b,c) and SameSide(p,b, a,c)
        and SameSide(p,c, a,b) then return true
    else return false
```

p.s. [该网页还介绍了针对三角形的另一种方法](http://www.blackpawn.com/texts/pointinpoly/)