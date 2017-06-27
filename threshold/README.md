# 阈值处理

## 一. 阈值处理

### 1 基本意义:
* 设置一个阈值，将图像中的每一个像素点与阈值进行比较，提取出我们想要的部分。

### 2 阈值类型:

#### 2.1 二进制阈值化

```
f(x) =  maxValue,  if x >  threshold
        0,         if x <= threshold
```

* 高于该阈值则为 `maxValue`，否则为 `0`。

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_type1.png)

#### 2.2 反二进制阈值化

```
f(x) =  0,         if x >  threshold
        maxValue,  if x <= threshold
```

* 高于该阈值则为 `0`，否则为 `maxValue`。

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_type2.png)

#### 2.3 截断阈值化

```
f(x) =  threshold,  if x >  threshold
        x,          if x <= threshold
```

* 高于该阈值则为阈值 `threshold`，否则为原值。

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_type3.png)

#### 2.4 阈值化为 0

```
f(x) =  0,  if x >  threshold
        x,  if x <= threshold
```

* 高于该阈值则为原值，否则为 `0`。

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_type4.png)

#### 2.5 反阈值化为 0

```
f(x) =  x,  if x >  threshold
        0,  if x <= threshold
```

* 高于该阈值则为 `0`，否则为原值。

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_type5.png)

### 3. 代码及结果

```cpp
/**
@version: opencv-3.2.0-vc14
@patrick 2017-6-26
**/

#include <opencv.hpp>

using namespace std;
using namespace cv;

int threshold_type = 0;
int max_type = 4;
int threshold_val = 0;
int max_val = 255;

char* window_name = "Treshold Demo";
char* trackbar_type = "Type ";
char* trackbar_value = "Value ";

Mat src, src_gray, dst;

void ThresholdUpdate(int, void*);

int main(int argc, char* argv[])
{
	if (argv[1] != NULL) {
		src = imread(argv[1], IMREAD_COLOR);
		if (src.empty()){
			fprintf(stderr, "Open Failed %s\n", argv[1]);
			return -1;
		}
	}

	// 转化为灰度图片
	cvtColor(src, src_gray, CV_RGB2GRAY);

	// 创建一个窗口
	namedWindow(window_name, CV_WINDOW_NORMAL);
	
	// 创建滑动条
	createTrackbar(trackbar_type, window_name, &threshold_type, max_type, ThresholdUpdate);
	createTrackbar(trackbar_value, window_name, &threshold_val, max_val, ThresholdUpdate);

	waitKey();
	return 0;
}

void ThresholdUpdate(int, void*)
{
	threshold(src_gray, dst, threshold_val, max_val, threshold_type);
	imshow(window_name, dst);
}
```

效果：

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_res.png)

## 二. 灰度图像分割之阈值处理

### 1. 平均阈值

#### 1.1 原理

平均阈值属于阈值处理中最简单的一种，使用图像中阈值的均值作为阈值，分割图像。 根据统计学，可以使用概率方法如以下步骤：

>```
>步骤：
>1) 计算图像灰度直方图
>2) 直方图归一化，得出每项概率（归一类型为 NORM_L1）
>3) 均值为横坐标值乘于概率的总和
>    threshold = mean = sum(p(i) * i)  (i = bins[1], bins[2], ...)
>```

#### 1.2 代码及效果

```cpp
double GetMeanThreshold(Mat img, int graylvl)
{
	if (img.empty()) {
		return 0.0;
	}
	
	// 1. 计算图像灰度直方图
	if (img.channels() == 3) {
		cvtColor(img, img, CV_RGB2GRAY);
	}
	Mat hist_gray;
	float range[] = { 0, 255 };
	const float* hist_range = range;
	calcHist(&img, 1, 0, Mat(), hist_gray, 1, &graylvl, &hist_range);

	// 2. 归一化直方图，得出直方图每一项的概率
	normalize(hist_gray, hist_gray, 1, 0, NORM_L1);

	// 3. 直方图的横坐标乘于概率，然后求和得出阈值
	double bin_width = (range[1] - range[0]) / graylvl;
	double thres = 0.0;
	for (int i = 0; i < graylvl; ++i) {
		thres += hist_gray.at<float>(i) * ((i + 0.5) * bin_width);
	}

	return thres;
}
```

效果：

![meanthreshold_res1](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/getmeanthreshold_res.png)

### 2. P-tile 阈值

#### 2.1 原理

P-tile 阈值即 **p 分位**的阈值， 当 `p = 0.5` 即以中位数为阈值。也就是说，将所有像素值排序，第 `p * totalPixels` 个像素点的值，`p 取 (0, 1]`。 

>```
>步骤：
>1) 计算图像灰度直方图
>2) 取出直方图中第 `p * totalPixels` 个像素点的值
>```

#### 2.2 代码及效果

```cpp
double GetPtileThreshold(Mat img, double ptile, int graylvl)
{
	if (img.empty()) {
		return 0.0;
	}

	// 1. 计算图像灰度直方图
	if (img.channels() == 3) {
		cvtColor(img, img, CV_RGB2GRAY);
	}
	Mat hist_gray;
	float range[] = { 0, 255 };
	const float* hist_range = range;
	calcHist(&img, 1, 0, Mat(), hist_gray, 1, &graylvl, &hist_range);
	
	// 2. 选取 p 分位
	double count = 0.0;
	double p_count = img.rows * img.cols * ptile;
	double bin_width = (range[1] - range[0]) / graylvl;
	double thres = graylvl;
	for (int i = 0; i < graylvl; ++i) {
		count += hist_gray.at<float>(i);
		if (count >= p_count) {
			thres = (i + 0.5) * bin_width;
			break;
		}
	}
	return thres;
}
```


效果：

![ptilethreshold_res](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/getptilethreshold_res.png)

### 参考
[1] [http://www.opencv.org.cn/opencvdoc/2.3.2/html/doc/tutorials/imgproc/threshold/threshold.html](http://www.opencv.org.cn/opencvdoc/2.3.2/html/doc/tutorials/imgproc/threshold/threshold.html)
