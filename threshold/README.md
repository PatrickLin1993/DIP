## 阈值处理

### 1. 基本意义:
* 设置一个阈值，将图像中的每一个像素点与阈值进行比较，提取出我们想要的部分。

### 2. 阈值类型:

#### 2.1 二进制阈值化

\begin{eqnarray}f(x)=
\begin{cases}
maxValue, &x>threshold\cr 0, &x<=threshold\end{cases}
\end{eqnarray}

* 高于该阈值则为 `maxValue`，否则为 `0`。

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_type1.png)

#### 2.2 反二进制阈值化

\begin{eqnarray}f(x)=
\begin{cases}
0, &x>threshold\cr maxValue, &x<=threshold\end{cases}
\end{eqnarray}

* 高于该阈值则为 `0`，否则为 `maxValue`。

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_type2.png)

#### 2.3 截断阈值化

\begin{eqnarray}f(x)=
\begin{cases}
threshold, &x>=threshold\cr x, &x<threshold\end{cases}
\end{eqnarray}

* 高于该阈值则为阈值 `threshold`，否则为原值。

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_type3.png)

#### 2.4 阈值化为 0

\begin{eqnarray}f(x)=
\begin{cases}
x, &x>=threshold\cr 0, &x<threshold\end{cases}
\end{eqnarray}

* 高于该阈值则为原值，否则为 `0`。

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_type4.png)

#### 2.5 反阈值化为 0

\begin{eqnarray}f(x)=
\begin{cases}
0, &x>threshold\cr x, &x<=threshold\end{cases}
\end{eqnarray}

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

![](https://github.com/PatrickLin1993/DIP/blob/master/threshold/pics/threshold_result1.png)

### 参考
[1] [http://www.opencv.org.cn/opencvdoc/2.3.2/html/doc/tutorials/imgproc/threshold/threshold.html](http://www.opencv.org.cn/opencvdoc/2.3.2/html/doc/tutorials/imgproc/threshold/threshold.html)