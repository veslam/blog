---
title: 计算几何入门
date: 2016-11-19 09:24:27
tags: [computational geometry, convex hull, coding, Processing]
---
公司每周举办技术沙龙，我惆怅了许久演讲主题，最终选定__计算几何__。研究生时期旁听了邓老师开设的这门课程，体验到思考的乐趣。
惊喜的发现，学堂在线有邓老师的计算几何课程，有讲义和视频。已经与去教室听讲几乎无异啦。于是演讲前两天刷了刷，温习了__*Convex Hull(凸包)*__的生成算法和__*Voronoi Diagram*__的基本知识，现学现卖😁

---
把链接和参考资料记录在这里

[邓老师《计算几何》课程 - 学堂在线](http://www.xuetangx.com/courses/course-v1:TsinghuaX+70240183x+2016_2/info)
邓老师的电子讲义[电子阅读版（全幅）](http://pan.baidu.com/s/1eQrYn9w) [打印版（八拼）](http://pan.baidu.com/s/1dDAXjFz)
[Demo演示（需要使用支持Java Applet的旧版浏览器）](http://dsa.cs.tsinghua.edu.cn/~deng/cg/demo/index.htm)

[Mesh – A Processing Library](http://leebyron.com/mesh/)

---
⬆️最后一个链接是一个小巧的Processing库，提供生成从平面点集生成凸包／Voronoi图的功能。
Processing确实是快速做演示的好工具，把我用Mesh库做的两个小Demo放上来。

接受鼠标点击，生成当前点集的凸包。
![showConvexHull](https://github.com/veslam/ImagesForBlog/raw/master/res/20161119_01_ConvexHull.png)
``` C
import megamu.mesh.*;

int arraySize = 64, numOfPoints = 0;
float[][] points = new float[arraySize][2];
Hull myHull;

static int pointRadius = 4;

void setup() {
  size(640, 360);
  
  addPoint(120, 230);
  addPoint(150, 105);
  addPoint(320, 113);
  addPoint(250, 250);
  addPoint(240, 150);
  addPoint(200, 190);
  addPoint(236, 75);
  addPoint(310, 70);
  addPoint(373, 134);
  addPoint(193, 126);
  addPoint(269, 96);
  addPoint(301, 171);
  addPoint(258, 193);
  addPoint(285, 135);
  
  //rebuild();
}

void draw() {
   // show all points
   fill(0,255,0);
   for (int i = 0; i < numOfPoints; i ++) {
     ellipse(points[i][0], points[i][1], pointRadius + 2, pointRadius + 2);
   }
  
   if (null != myHull) {
    
       // fill polygon
       MPolygon myRegion = myHull.getRegion();
       fill(255,0,0, 20);
       myRegion.draw(this);
             
       // reshow extreme points
       int[] extrema = myHull.getExtrema();
       fill(0,0,255);     
       for (int i = 0; i < extrema.length; i ++) {      
          ellipse(points[extrema[i]][0], points[extrema[i]][1], pointRadius, pointRadius);
       }
  }
}

void mouseClicked() {
  //println("mousePosition (", mouseX, ", ", mouseY, ")");
  
  addPoint(mouseX, mouseY);
  rebuild();
}

void addPoint(int x, int y) {
  points[numOfPoints][0] = x;
  points[numOfPoints][1] = y;
  
  // trick
  if (0 == numOfPoints) {
    for (int i = 0; i < arraySize; i ++) {
      points[i][0] = x;
      points[i][1] = y;
    }
  }
  
  numOfPoints ++;
}

void keyPressed() {
  rebuild();
}
  

void rebuild() {
  myHull = new Hull( points );
}
```

接受鼠标点击，生成当前点集的Voronoi Diagram。
![showVoronoiDiagram](https://github.com/veslam/ImagesForBlog/raw/master/res/20161119_02_VoronoiDiagram.png)
``` C
import megamu.mesh.*;

float[][] points;
Voronoi myVoronoi;

static int pointRadius = 4;

void setup() {
  size(640, 360);
  
  addPoint(120, 230);
  addPoint(150, 105);
  addPoint(320, 113);
  addPoint(250, 250);
  addPoint(240, 150);
  addPoint(200, 190);
  addPoint(236, 75);
  addPoint(310, 70);
  addPoint(373, 134);
  addPoint(193, 126);
  addPoint(269, 96);
  addPoint(301, 171);
  addPoint(258, 193);
  addPoint(285, 135);
  
  //rebuild();
}

void draw() {
   // show all points
   fill(0,255,0);
   for (int i = 0; i < points.length; i ++) {
     ellipse(points[i][0], points[i][1], pointRadius + 2, pointRadius + 2);
   }
  
   if (null != myVoronoi) {
    
      // fill polygon
      MPolygon[] myRegions = myVoronoi.getRegions();
       
      for(int i=0; i<myRegions.length; i++)
      {
        randomColorFill();
        myRegions[i].draw(this); // draw this shape
      }
       
  
      float[][] myEdges = myVoronoi.getEdges();
      
      for(int i=0; i<myEdges.length; i++)
      {
        float startX = myEdges[i][0];
        float startY = myEdges[i][1];
        float endX = myEdges[i][2];
        float endY = myEdges[i][3];
        line( startX, startY, endX, endY );
      }
   }
}

void mouseClicked() {
  //println("mousePosition (", mouseX, ", ", mouseY, ")");
  
  addPoint(mouseX, mouseY);
  rebuild();
}

void addPoint(int x, int y) {
  int oldLength;
  if (null == points) {
    oldLength = 0;
  } else {
    oldLength = points.length;
  }
  
  
  float[][] newPoints = new float[oldLength + 1][2];
  for (int i = 0; i < oldLength; i ++) {
    newPoints[i][0] = points[i][0];
    newPoints[i][1] = points[i][1];
  }
  newPoints[oldLength][0] = x;
  newPoints[oldLength][1] = y;
  
  points = newPoints;
}

void keyPressed() {
  rebuild();
}
  

void rebuild() {
  myVoronoi = new Voronoi( points );
}

void randomColorFill() {
  fill(random(256), random(256), random(256), 20);
  return;
  // TODO: '''Generate a random hsl color.'''
}
```