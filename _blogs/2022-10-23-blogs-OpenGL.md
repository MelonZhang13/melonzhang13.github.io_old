---
layout: archive
title: 'OpenGL练习——搭建一个太阳系'
date: 2022-10-23
permalink: /blogs/2022/10/LearningOpenGL/
tags:
  - Coursework Exercises
  - OpenGL
redirect_from:
  - /blogs
---

<br>

* TOC
{:toc}



## 前言

本学期在上一门《虚拟现实技术Virtual Reality Technology》的选修课程，虽然名为虚拟现实技术，但实际教授内容主要为计算机图形学理论和OpenGL编程技巧。我不止一次怀疑自己是否走错了教室。言归正传，本文将以搭建一个太阳系为例，介绍OpenGL的基础使用方法。

OpenGL（英语：Open Graphics Library，译名：开放图形库或者“开放式图形库”）是用于渲染2D、3D矢量图形的跨语言、跨平台的应用程序编程接口（API）。这个接口由近350个不同的函数调用组成，用来从简单的图形比特绘制复杂的三维景象。而另一种程序接口系统是仅用于Microsoft Windows上的Direct3D。OpenGL常用于CAD、虚拟现实、科学可视化程序和电子游戏开发。

## 环境准备
主要库文件包括OpenGL的窗口管理库```GLFW``` ```GLUT```, 扩展库```gLEW```, 纹理读取```opencv2```。

## 渲染流水线原理
<div style="text-align:center;">
    <img src='/images/CG1.png' style="width: 80%;">
</div>
<div style="text-align:center;">
    <img src='/images/CG2.png' style="width: 80%;">
</div>

## 最终效果
<div class="video-container">
  <iframe src="https://drive.google.com/file/d/1hDoh_hsBXj7V8GbvoPijtXWiKb84H2Y2/preview" frameborder="0" allow="autoplay"></iframe>
</div>
<style>
.video-container {
  position: relative;
  padding-bottom: 56.25%; /* 16:9 aspect ratio */
  height: 0;
  overflow: hidden;
  max-width: 100%;
  background: #fff;
  text-align: left; /* Align video to the left */
}
.video-container iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #fff;
  border: none;
}
/* Media query for larger screens */
@media (min-width: 768px) {
  .video-container {
    max-width: 960px; /* Optional: you can set a max-width for larger screens */
    margin: 0; /* Align to the left */
  }
}
</style>

## 源代码
```c++
  #include <iostream>
  #include <string>
  #include <GL/glut.h>
  #include <opencv2/opencv.hpp>

  using namespace std;
  using namespace cv;

  #pragma region 变量

  //场景信息变量
  static float year = 0;
  static float moon = 0;
  static float day = 0;
  static float n = 0;
  static float camera_pos_x = 0;
  static float camera_pos_y = 3.0f;
  static GLfloat PI = 3.1415926;

  //纹理贴图变量
  GLuint SunID, MoonID, EarthID;
  Mat Sun, Earth, Moon;                                  //太阳、地球、月亮纹理图
  GLubyte* SunPixels, * EarthPixels, * MoonPixels;           //太阳、地球、月亮纹理图数据

  #pragma endregion


  #pragma region 功能函数声明

  void display();                                        //显示函数
  void reshape(int w, int h);                            //窗口变化回调函数
  void myTimer(int t);                                   //定时回调函数
  void keyboard(unsigned char key, int x, int y);        //键盘控制函数
  void drawstring(const char* str);                      //位图字体绘制函数
  void lightInit();                                      //光源初始化函数
  void LoadTexture(GLuint texture_ID, Mat Texture, GLubyte* TexturePixels, string TextureFile);   //纹理加载函数
  void gltDrawSphere(GLfloat fRadius, GLint iSlices, GLint iStacks);                              //绘制球体

  #pragma endregion


  //Main函数
  int main(int argc, char** argv)
  {
    //准备阶段
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_RGB | GLUT_DOUBLE | GLUT_DEPTH);   //初始化显示模式
    glutInitWindowSize(1000, 1000);                             //初始化窗口大小
    glutCreateWindow("OpenGL");                                 //窗口命名

    lightInit();                                                //光照初始化
    glutDisplayFunc(display);                                   //显示
    glutReshapeFunc(reshape);                                   //窗口变化回调
    glutKeyboardFunc(keyboard);                                 //键盘控制
    glutTimerFunc(10, myTimer, 1);                              //定时器
    glutMainLoop();                                             //进入GLUT事件处理循环，永远不返回
    return 0;
  }


  //显示函数
  void display()
  {
    //初始化屏幕或清空屏幕
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    //视图变换
    glLoadIdentity();
    gluLookAt(camera_pos_x, camera_pos_y, 5.0f, 0.0f, 0.0f, 0.0f, 0.0f, 1.0f, 0.0f);
    glPushMatrix();

    //太阳
    glEnable(GL_TEXTURE_2D);
      LoadTexture(SunID, Sun, SunPixels, "D:/220220426GOTOSTUDY/Master-Course/Virtual Reality Technology/OpenGLTest/sun.bmp");
      glRotatef((GLfloat)year, 0.0, 1.0, 0.0);                 //太阳自转
      gltDrawSphere(1.0, 20, 16);
    glDisable(GL_TEXTURE_2D);

    //地球轨道
    glEnable(GL_COLOR_MATERIAL);
      glBegin(GL_LINE_LOOP);
      glColor3f(1.0, 1.0, 0.0);
      for (int i = 0; i < 100; i++)
        glVertex3f(2.0 * cos(2 * i * PI / 100), 0.0f, 2.0 * sin(2 * i * PI / 100));
      glEnd();
      //消除glColor对纹理的影响
      glColor3f(1.0, 1.0, 1.0);
    glDisable(GL_COLOR_MATERIAL);

    //地球
    glEnable(GL_TEXTURE_2D);
      LoadTexture(EarthID, Earth, EarthPixels, "D:/220220426GOTOSTUDY/Master-Course/Virtual Reality Technology/OpenGLTest/earth.bmp");
      glRotatef((GLfloat)year, 0.0, 1.0, 0.0);                 //地球公转
      glTranslatef(2.0, 0.0, 0.0);
      glRotatef((GLfloat)day - year, 0.0, 1.0, 0.0);           //地球自转
      gltDrawSphere(0.2, 10, 8);
    glDisable(GL_TEXTURE_2D);

    //月球轨道
    glEnable(GL_COLOR_MATERIAL);
      glBegin(GL_LINE_LOOP);
      glColor3f(1.0, 1.0, 1.0);
      for (int i = 0; i < 100; i++)
        glVertex3f(0.5 * cos(2 * i * PI / 100), 0.0f, 0.5 * sin(2 * i * PI / 100));
      glEnd();
      //消除glColor对纹理的影响
      glColor3f(1.0, 1.0, 1.0);
    glDisable(GL_COLOR_MATERIAL);

    //月球
    glEnable(GL_TEXTURE_2D);
        LoadTexture(MoonID, Moon, MoonPixels, "D:/220220426GOTOSTUDY/Master-Course/Virtual Reality Technology/OpenGLTest/moon.bmp");
      glRotatef((GLfloat)moon - day, 0.0, 1.0, 0.0);           //月球公转
      glTranslatef(0.5, 0.0, 0.0);
      glRotatef((GLfloat)moon, 0.0, 1.0, 0.0);                 //月球自转
      glutSolidSphere(0.05, 10, 8);
    glDisable(GL_TEXTURE_2D);
    glPopMatrix();

    //输出文字
    glEnable(GL_COLOR_MATERIAL);
      glRasterPos2f(-2.0, 2.0);
      string s1 = "camera_pos_x:" + to_string(camera_pos_x);
      char* c1 = const_cast<char*>(s1.c_str());
      drawstring(c1);
      glRasterPos2f(-2.0, 1.9);
      string s2 = "camera_pos_y:" + to_string(camera_pos_y);
      char* c2 = const_cast<char*>(s2.c_str());
      drawstring(c2);
      glColor3f(1.0, 1.0, 0.0);
      glRasterPos2f(-2.0, 1.8);
      string s3 = "Yellow：The Sun";
      char* c3 = const_cast<char*>(s3.c_str());
      drawstring(c3);
      glColor3f(0.0, 0.0, 1.0);
      glRasterPos2f(-2.0, 1.7);
      string s4 = "Blue:The Earth";
      char* c4 = const_cast<char*>(s4.c_str());
      drawstring(c4);
      glColor3f(1.0, 1.0, 1.0);
      glRasterPos2f(-2.0, 1.6);
      string s5 = "White:The Moon";
      char* c5 = const_cast<char*>(s5.c_str());
      drawstring(c5);
      //消除glColor对纹理的影响
      glColor3f(1.0, 1.0, 1.0);
    glDisable(GL_COLOR_MATERIAL);

    //开启双缓存
    glutSwapBuffers();
  }


  //窗口变化回调函数
  void reshape(int w, int h)
  {
    //投影变换
    glMatrixMode(GL_PROJECTION); //指出当前矩阵用于投影变换
    glLoadIdentity();            //使栈顶矩阵变换为单位矩阵	
    gluPerspective(60.0, 1.0, 1.0, 20.0);
    //视点变换
    glMatrixMode(GL_MODELVIEW); //指出当前矩阵用于视点变换
  }


  //定时回调函数
  void myTimer(int t)
  {
    n += 0.01;
    day = n * 360;
    moon = n * 30;
    year = n;
    glutPostRedisplay();
    glutTimerFunc(10, myTimer, 1);
  }


  //键盘控制函数
  void keyboard(unsigned char key, int x, int y)
  {
    switch (key)
    {
    case 'q':case 'Q':
      std::cout << "按下按钮Q或q退出" << std::endl;
      exit(0);
      break;
    case 'w':case 'W':
      camera_pos_y += 0.5f;
      break;
    case 's':case 'S':
      camera_pos_y -= 0.5f;
      break;
    case 'a':case 'A':
      camera_pos_x += 0.5f;
      break;
    case 'd':case 'D':
      camera_pos_x -= 0.5f;
      break;
    }
  }


  //位图字体绘制函数
  void drawstring(const char* str)
  {
    static bool isfirstcall = true;
    static GLuint lists;
    if (isfirstcall)
    {
      isfirstcall = false;
      lists = glGenLists(128);
      wglUseFontBitmaps(wglGetCurrentDC(), 0, 128, lists);
    }
    for (; *str != '\0'; ++str)
    {
      glCallList(lists + *str);
    }
  }


  //光源初始化函数
  void lightInit()
  {
    GLfloat light_environment[] = { 0.8,0.8,0.8,1.0 }; //环境光
    GLfloat light_diffuse[] = { 1.0,1.0,1.0,1.0 };     //漫反射光
    GLfloat light_mirror[] = { 1.0,1.0,1.0,1.0 };      //镜面反射光
    GLfloat light_position[] = { 0,0,0,1 };            //光源位置

    glLightfv(GL_LIGHT0, GL_AMBIENT, light_environment);
    glLightfv(GL_LIGHT0, GL_DIFFUSE, light_diffuse);
    glLightfv(GL_LIGHT0, GL_SPECULAR, light_mirror);
    glLightfv(GL_LIGHT0, GL_POSITION, light_position);
    glEnable(GL_LIGHTING);//开启光照效果
    glEnable(GL_LIGHT0);
    glEnable(GL_COLOR_MATERIAL);//！！！开启光照下的颜色模式，不开启这个直接没有颜色渲染！！！
    glEnable(GL_DEPTH_TEST);//开启深度测试

  }


  //纹理加载函数
  void LoadTexture(GLuint texture_ID, Mat Texture, GLubyte *TexturePixels, string TextureFile)
  {
    Texture = imread(TextureFile);
    int pixellength = Texture.cols * Texture.rows * 3;
    TexturePixels = new GLubyte[pixellength];
    memcpy(TexturePixels, Texture.data, pixellength * sizeof(char));
    imshow(TextureFile, Texture);
    //创建纹理ID
    glGenTextures(1, &texture_ID);
    //绑定该纹理ID，后续操作是对该纹理操作
    glBindTexture(GL_TEXTURE_2D, texture_ID);
    //纹理放大缩小使用线性插值
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    //纹理水平竖直方向外扩使用重复贴图
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);;

    //将图像内存用作纹理信息
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, Texture.cols, Texture.rows, 0, GL_BGR_EXT, GL_UNSIGNED_BYTE, TexturePixels);

    free(TexturePixels);
  }


  //绘制球体函数
  void gltDrawSphere(GLfloat fRadius, GLint iSlices, GLint iStacks)

  {
    GLfloat drho = (GLfloat)(3.141592653589) / (GLfloat)iStacks;
    GLfloat dtheta = 2.0f * (GLfloat)(3.141592653589) / (GLfloat)iSlices;
    GLfloat ds = 1.0f / (GLfloat)iSlices;
    GLfloat dt = 1.0f / (GLfloat)iStacks;
    GLfloat t = 1.0f;
    GLfloat s = 0.0f;
    GLint i, j;
    for (i = 0; i < iStacks; i++)
    {
      GLfloat rho = (GLfloat)i * drho;
      GLfloat srho = (GLfloat)(sin(rho));
      GLfloat crho = (GLfloat)(cos(rho));
      GLfloat srhodrho = (GLfloat)(sin(rho + drho));
      GLfloat crhodrho = (GLfloat)(cos(rho + drho));
      glBegin(GL_TRIANGLE_STRIP);
      s = 0.0f;
      for (j = 0; j <= iSlices; j++)
      {
        GLfloat theta = (j == iSlices) ? 0.0f : j * dtheta;
        GLfloat stheta = (GLfloat)(-sin(theta));
        GLfloat ctheta = (GLfloat)(cos(theta));
        GLfloat x = stheta * srho;
        GLfloat y = ctheta * srho;
        GLfloat z = crho;
        glTexCoord2f(s, t);
        glNormal3f(x, y, z);
        glVertex3f(x * fRadius, y * fRadius, z * fRadius);
        x = stheta * srhodrho;
        y = ctheta * srhodrho;
        z = crhodrho;
        glTexCoord2f(s, t - dt);
        s += ds;
        glNormal3f(x, y, z);
        glVertex3f(x * fRadius, y * fRadius, z * fRadius);
      }
      glEnd();
      t -= dt;
    }
  }
```

<br><br>

<!-- Add a button to return to the directory -->
<!-- <a href="/blogs" class="back-button">Return to Blogs</a> -->
<a href="/blogs" class="pagination--pager">Return to Blogs</a>

<style>
.back-button {
  display: inline-block;
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 16px;
  color: white;
  background-color: #6AABC5;
  border: none;
  border-radius: 4px;
  text-decoration: none!important;
  cursor: pointer;
}

.back-button:hover {
  background-color: #6AABC5;
}
</style>
