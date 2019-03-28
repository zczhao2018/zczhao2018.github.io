---
layout:     post                    # 使用的布局（不需要改）
title:      OpenCV下的Lucas-Kanade算法:运动分析和目标跟踪               # 标题 
subtitle:   Hello World, Hello Blog #副标题
date:       2017-03-30              # 时间
author:     zc                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - OpenCV
---
在本篇博客中一并使用了OpenCV读取摄像头,读取视频等操作,通过光流法,实现目标检测.含义有自己写的代码和官方的代码,实现的方法不尽相同
主要使用如下几个函数:具体使用和注解,看下面代码,
[goodFeaturesToTrack](http://www.opencv.org.cn/opencvdoc/2.3.2/html/modules/imgproc/doc/feature_detection.html?highlight=cvgoodfeaturestotrack#void%20cvGoodFeaturesToTrack%28const%20CvArr*%20image,%20CvArr*%20eigImage,%20CvArr*%20tempImage,%20CvPoint2D32f*%20corners,%20int*%20cornerCount,%20double%20qualityLevel,%20double%20minDistance,%20const%20CvArr*%20mask,%20int%20blockSize,%20int%20useHarris,%20double%20k%29)
确定图像上的强角点。
[cornerSubPix](http://www.opencv.org.cn/opencvdoc/2.3.2/html/modules/imgproc/doc/feature_detection.html?highlight=cvgoodfeaturestotrack#void%20cvGoodFeaturesToTrack%28const%20CvArr*%20image,%20CvArr*%20eigImage,%20CvArr*%20tempImage,%20CvPoint2D32f*%20corners,%20int*%20cornerCount,%20double%20qualityLevel,%20double%20minDistance,%20const%20CvArr*%20mask,%20int%20blockSize,%20int%20useHarris,%20double%20k%29)
精确角点的位置。
[calcOpticalFlowPyrLK](https://docs.opencv.org/2.4.13/modules/video/doc/motion_analysis_and_object_tracking.html?highlight=cvcalcoptical#void%20cvCalcOpticalFlowPyrLK%28const%20CvArr*%20prev,%20const%20CvArr*%20curr,%20CvArr*%20prev_pyr,%20CvArr*%20curr_pyr,%20const%20CvPoint2D32f*%20prev_features,%20CvPoint2D32f*%20curr_features,%20int%20count,%20CvSize%20win_size,%20int%20level,%20char*%20status,%20float*%20track_error,%20CvTermCriteria%20criteria,%20int%20flags%29)
使用具有金字塔的迭代Lucas-Kanade方法计算稀疏特征集的光流。

```
#include "opencv/cv.h"
#include "opencv/cxcore.h"
#include "opencv/highgui.h"
#include <iostream>
using namespace std;
using namespace cv;

/* 全局变量 */
namespace{
    const int MAX_CORNERS = 1000;
    int WIN_SIZE = 5;
}

/* 文件路径 */
namespace{
//	const char* InputVideoPath = "../image/test.mp4";
}

int main()
{ 
    // 读取视频数据
//    CvCapture* capture = cvCaptureFromAVI(InputVideoPath);

    // 摄像头数据
    CvCapture* capture = cvCreateCameraCapture(0);
   
    // 放置视频当前帧的图片
    IplImage* imgSrc = cvQueryFrame(capture);
    if(!capture){
        fprintf(stderr, "Cannot open video!\n");
        return -1;
    }

    // 设置帧的大小 
    CvSize sizeImg = cvGetSize(imgSrc);
    
    // 灰度图
    IplImage* imgGray = cvCreateImage( sizeImg, IPL_DEPTH_8U, 1 );
    IplImage* imgCurr = cvCreateImage( sizeImg, IPL_DEPTH_8U, 1 );

    // 光流显示
    IplImage* imgDisplay = cvCreateImage( sizeImg, IPL_DEPTH_8U, 3 );
    IplImage* imgDisplay1 = cvCreateImage( sizeImg, IPL_DEPTH_8U, 3 );
    IplImage* imgDisplay2 = cvCreateImage( sizeImg, IPL_DEPTH_8U, 3 );

    // 金字塔图像的缓冲区
    CvSize sizePyr = cvSize( imgSrc->width + 8, imgSrc->height / 3 );
    IplImage* pyrPrev = cvCreateImage( sizePyr, IPL_DEPTH_32F, 1 );
    IplImage* pyrCurr = cvCreateImage( sizePyr, IPL_DEPTH_32F, 1 );
    CvPoint2D32f* featuresPrev = new CvPoint2D32f[ MAX_CORNERS ];
    CvPoint2D32f* featuresCurr = new CvPoint2D32f[ MAX_CORNERS ];
    CvSize sizeWin = cvSize( WIN_SIZE, WIN_SIZE );
    IplImage* imgEig = cvCreateImage( sizeImg, IPL_DEPTH_32F, 1 );
    IplImage* imgTemp = cvCreateImage( sizeImg, IPL_DEPTH_32F, 1 );

    // 金字塔卢卡斯-卡纳德 Lucas-Kanade
    char featureFound[ MAX_CORNERS ];
    float featureErrors[ MAX_CORNERS ];
    int cornerCount = MAX_CORNERS;

    while(true){
        // 得到第一帧
        imgSrc = cvQueryFrame(capture);
        if(!imgSrc){
            break;   
        }
        // 将rgb转换为gray
        cvCvtColor(imgSrc, imgGray, CV_BGR2GRAY );
        // 显示
        cvCopy( imgSrc, imgDisplay );
        cvCopy( imgSrc, imgDisplay1 );
        cvCopy( imgSrc, imgDisplay2 );
        // 得到第二帧,也就是当前帧
        imgSrc = cvQueryFrame(capture);// get the second frame
        if(!imgSrc){
            break;
        }
        // 得到第二帧(灰度)
        cvCvtColor(imgSrc, imgCurr, CV_BGR2GRAY );

        // 我需要做的第一件事就是得到特征点,检验第一帧的特征点
        cvGoodFeaturesToTrack(
            imgGray,//8位或32为浮点型输入图像,单通道
            imgEig,//缓存
            imgTemp,//缓存
            featuresPrev,//保存检测出的角点
            &cornerCount,//角点数目最大值,如果实际检测的角点超过此值,则只返回前cornerCount个强角点
            0.01,
            5.0,
            0,
            3,
            0,
            0.04
        );
        // 绘制特征点
        for( int i=0; i < cornerCount; i++ )
        {
            cvLine(
                imgDisplay,
                cvPoint(featuresPrev[i].x, featuresPrev[i].y),
                cvPoint(featuresPrev[i].x, featuresPrev[i].y),
                CV_RGB(255,0,0),
                5
            );
        }
        //优化角点的位置,精确角点位置
        cvFindCornerSubPix(
            imgGray,//输入图像
            featuresPrev,//输入角的初始坐标和为输出提供的精细坐标
            cornerCount,
            sizeWin,
            CvSize(-1,-1),//无死区
            cvTermCriteria(CV_TERMCRIT_ITER|CV_TERMCRIT_EPS,20,0.03)//终止角点细化迭代过程的标准
            );

        for( int i=0; i < cornerCount; i++ )
        {
            cvLine(
                imgDisplay1,
                cvPoint(featuresPrev[i].x, featuresPrev[i].y),
                cvPoint(featuresPrev[i].x, featuresPrev[i].y),
                CV_RGB(255,0,0),
                5
            );
        }
        
        cvCalcOpticalFlowPyrLK(
            imgGray,//需要计算光流的前一帧图像
            imgCurr,//需要计算的当前帧图像
            pyrPrev,//缓存
            pyrCurr,//缓存
            featuresPrev,
//是前一帧图像中的特征点,这个特征点必须自己去找,
//所以在使用calcOpticalFlowPyrLK()函数时,前面需要有一个找特征点的操作,
//那么一般就是找图像的角点,就是一个像素点与周围像素点都不同的那个点,
//这个角点特征点的寻找,opencv也提供了一个函数:goodFeatureToTrack(),就是前面用到的,也有其他寻找特征点的方式
            featuresCurr,
//计算出来的特征点在第二帧中的新位置,然后输出,特征点的新位置可能变化了,
//也可能没有变化,然后这个状态存放在featureFound
            cornerCount,
            sizeWin,
            5,
            featureFound,//变化了则为1,没有变化则为0
            featureErrors,//这是一个输出矩阵,存的是新旧两个特征点位置的误差
            cvTermCriteria(CV_TERMCRIT_ITER|CV_TERMCRIT_EPS,20,0.3),
            0
        );

        // 显示
        for(int i=0; i<cornerCount; i++){
            if( featureFound == 0 || featureErrors[i] > 550 ){
                continue;
            }       
            CvPoint p0 = cvPoint(
                cvRound( featuresPrev[i].x ),
                cvRound( featuresPrev[i].y )
            );
            CvPoint p1 = cvPoint(
                cvRound( featuresCurr[i].x ),
                cvRound( featuresCurr[i].y )
            );
            cvLine(imgDisplay2, p0, p1, CV_RGB(255,0,0), 2 );
        }
        
        cvNamedWindow("goodFeatureToTrack", 0 );
        cvShowImage("goodFeatureToTrack",imgDisplay);
        cvNamedWindow("FindCornerSubPix", 0 );
        cvShowImage("FindCornerSubPix",imgDisplay1);
        cvNamedWindow("CalcOpticalFlowPyrLK", 0 );
        cvShowImage("CalcOpticalFlowPyrLK",imgDisplay2);
        cvWaitKey(5);
	}

    cvWaitKey(0);
    return 0;
}
```
goodFeatureToTrack效果图
![这里写图片描述](https://img-blog.csdn.net/20180802214249582?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW96aGljaGVuZ2hwdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
FindCornerSubPix效果图
![这里写图片描述](https://img-blog.csdn.net/20180802214311918?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW96aGljaGVuZ2hwdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
CalcOpticalFlowPyrLK效果图
![这里写图片描述](https://img-blog.csdn.net/20180802214405968?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW96aGljaGVuZ2hwdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**官方有一个LKdemo效果一致,挺有趣的**
运行时会有帮助信息,按键ESC退出程序,按键r自动初始化特征点,按键c删除所有点,按键n切换到夜间模式
![这里写图片描述](https://img-blog.csdn.net/20180802214517851?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW96aGljaGVuZ2hwdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
初始化特征点效果
![这里写图片描述](https://img-blog.csdn.net/20180802214647660?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW96aGljaGVuZ2hwdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
夜间模式
![这里写图片描述](https://img-blog.csdn.net/20180802214710831?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW96aGljaGVuZ2hwdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
[官方链接LKdemo](https://github.com/opencv/opencv/blob/master/samples/cpp/lkdemo.cpp)

```
#include "opencv2/video/tracking.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/videoio.hpp"
#include "opencv2/highgui.hpp"

#include <iostream>
#include <ctype.h>

using namespace cv;
using namespace std;

static void help()
{
    // print a welcome message, and the OpenCV version
    cout << "\nThis is a demo of Lukas-Kanade optical flow lkdemo(),\n"
            "Using OpenCV version " << CV_VERSION << endl;
    cout << "\nIt uses camera by default, but you can provide a path to video as an argument.\n";
    cout << "\nHot keys: \n"
            "\tESC - quit the program\n"
            "\tr - auto-initialize tracking\n"
            "\tc - delete all the points\n"
            "\tn - switch the \"night\" mode on/off\n"
            "To add/remove a feature point click it\n" << endl;
}

Point2f point;
bool addRemovePt = false;

static void onMouse( int event, int x, int y, int /*flags*/, void* /*param*/ )
{
    if( event == EVENT_LBUTTONDOWN )
    {
        point = Point2f((float)x, (float)y);
        addRemovePt = true;
    }
}

int main( int argc, char** argv )
{
    VideoCapture cap;
    TermCriteria termcrit(TermCriteria::COUNT|TermCriteria::EPS,20,0.03);
    Size subPixWinSize(10,10), winSize(31,31);

    const int MAX_COUNT = 500;
    bool needToInit = false;
    bool nightMode = false;

    help();
    cv::CommandLineParser parser(argc, argv, "{@input|0|}");
    string input = parser.get<string>("@input");

    if( input.size() == 1 && isdigit(input[0]) )
        cap.open(input[0] - '0');
    else
        cap.open(input);

    if( !cap.isOpened() )
    {
        cout << "Could not initialize capturing...\n";
        return 0;
    }

    namedWindow( "LK Demo", 1 );
    setMouseCallback( "LK Demo", onMouse, 0 );

    Mat gray, prevGray, image, frame;
    vector<Point2f> points[2];

    for(;;)
    {
        cap >> frame;
        if( frame.empty() )
            break;

        frame.copyTo(image);
        cvtColor(image, gray, COLOR_BGR2GRAY);

        if( nightMode )
            image = Scalar::all(0);

        if( needToInit )
        {
            // automatic initialization
            goodFeaturesToTrack(gray, points[1], MAX_COUNT, 0.01, 10, Mat(), 3, 3, 0, 0.04);
            cornerSubPix(gray, points[1], subPixWinSize, Size(-1,-1), termcrit);
            addRemovePt = false;
        }
        else if( !points[0].empty() )
        {
            vector<uchar> status;
            vector<float> err;
            if(prevGray.empty())
                gray.copyTo(prevGray);
            calcOpticalFlowPyrLK(prevGray, gray, points[0], points[1], status, err, winSize,
                                 3, termcrit, 0, 0.001);
            size_t i, k;
            for( i = k = 0; i < points[1].size(); i++ )
            {
                if( addRemovePt )
                {
                    if( norm(point - points[1][i]) <= 5 )
                    {
                        addRemovePt = false;
                        continue;
                    }
                }

                if( !status[i] )
                    continue;

                points[1][k++] = points[1][i];
                circle( image, points[1][i], 3, Scalar(0,255,0), -1, 8);
            }
            points[1].resize(k);
        }

        if( addRemovePt && points[1].size() < (size_t)MAX_COUNT )
        {
            vector<Point2f> tmp;
            tmp.push_back(point);
            cornerSubPix( gray, tmp, winSize, Size(-1,-1), termcrit);
            points[1].push_back(tmp[0]);
            addRemovePt = false;
        }

        needToInit = false;
        imshow("LK Demo", image);

        char c = (char)waitKey(10);
        if( c == 27 )
            break;
        switch( c )
        {
        case 'r':
            needToInit = true;
            break;
        case 'c':
            points[0].clear();
            points[1].clear();
            break;
        case 'n':
            nightMode = !nightMode;
            break;
        }

        std::swap(points[1], points[0]);
        cv::swap(prevGray, gray);
    }

    return 0;
}
```
参考链接:
https://blog.csdn.net/leixiaohua1020/article/details/15029187
https://blog.csdn.net/on2way/article/details/48954159
https://github.com/opencv/opencv/blob/master/samples/cpp/lkdemo.cpp
https://docs.opencv.org/2.4.13/index.html
https://blog.csdn.net/crzy_sparrow/article/details/7407604
