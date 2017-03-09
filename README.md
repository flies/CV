//连通域寻找
#include <opencv2/core/core.hpp>
#include<opencv2/highgui/highgui.hpp>
#include<opencv2/imgproc/imgproc.hpp>
#include<iostream>
#include<stack>



using namespace cv;
using namespace std;


const int k_max = 200;//连通域最多个数
int k =150;              //滑动条对应的连通域个数

stack<pair<int, int>>area;
pair<int, int>place;

//另一种方式灰度图转RGB，原理参考他人+一定修改
void color(Mat & M_erode1, Mat &M_end)
{
	uchar temp = 0;
	for (int i = 0; i < M_erode1.rows; i++){
		for (int j = 0; j < M_erode1.cols; j++){
			temp = M_erode1.at<uchar>(i, j);
			if (temp <= 51)
			{
				M_end.at<Vec3b>(i, j)[0] = 255;
				M_end.at<Vec3b>(i, j)[1] = temp * 5;
				M_end.at<Vec3b>(i, j)[2] = 0;
			}
			else if (temp <= 102)
			{
				temp -= 51;

				M_end.at<Vec3b>(i, j)[0] = 255 - temp * 5;
				M_end.at<Vec3b>(i, j)[1] = 255;
				M_end.at<Vec3b>(i, j)[2] = 0;
			}
			else if (temp <= 153)
			{
				temp -= 102;

				M_end.at<Vec3b>(i, j)[0] = 0;
				M_end.at<Vec3b>(i, j)[1] = 255;
				M_end.at<Vec3b>(i, j)[2] = temp * 5;
			}
			else if (temp <= 204)
			{
				temp -= 153;

				M_end.at<Vec3b>(i, j)[0] = 0;
				M_end.at<Vec3b>(i, j)[1] = 255 - uchar(128.0*temp / 51.0 + 0.5);
				M_end.at<Vec3b>(i, j)[2] = 255;
			}
			else if (temp < 255)
			{
				temp -= 204;

				M_end.at<Vec3b>(i, j)[0] = 0;
				M_end.at<Vec3b>(i, j)[1] = 127 - uchar(127.0*temp / 51.0 + 0.5);
				M_end.at<Vec3b>(i, j)[2] = 255;
			}
			else
			{
				M_end.at<Vec3b>(i, j)[0] = 255;
				M_end.at<Vec3b>(i, j)[1] = 255;
				M_end.at<Vec3b>(i, j)[2] = 255;
			}
		}
	}
}

//自己写的腐蚀+二值化
void erode_me(Mat &M,Mat &M_erode1){
	for (int i = 1; i < M.rows - 1; i++){
		for (int j = 1; j < M.cols - 1; j++){
			if (M.at<uchar>(i, j)>0 && M.at<uchar>(i, j + 1)>0 && M.at<uchar>(i, j - 1) > 0
				&& M.at<uchar>(i - 1, j) > 0 && M.at<uchar>(i + 1, j) > 0){
				M_erode1.at<uchar>(i, j) = 255;

			}
			else
			{
				M_erode1.at<uchar>(i, j) = 0;
			}
		}
	}
}

void ROI_me(Mat &M, Mat &M_erode1, Mat &M_erode){
	int maxx = 0, maxy = 0, minx = 100000, miny = 100000;
	for (int i = 1; i < M.rows - 1; i++){
		for (int j = 1; j < M.cols - 1; j++){
			if (M.at<uchar>(i, j) == 0 && i >= maxx){
				maxx = i;
			}
			if (M.at<uchar>(i, j) == 0 && j >= maxy){
				maxy = j;
			}
			if (M.at<uchar>(i, j) == 0 && i <= minx){
				minx = i;
			}
			if (M.at<uchar>(i, j) == 0 && j <= miny){
				miny = j;
			}
		}
	}

	//防止溢出
	if (miny - 3 >= 0 && minx - 3 >= 0 && maxx + 3 <= M.rows&&maxy + 3 <= M.cols){
		M_erode = M_erode1(Rect(miny - 3, minx - 3, (maxy - miny + 7), (maxx - minx + 7)));
	}
	else
		M_erode = M_erode1;
}

void tab(int, void*){
	int p = 255 / k;
	Mat M;
	Mat M1 = imread("1.bmp", 0);
	color(M1, M);
	imwrite("2.bmp", M);
	/*namedWindow("原图");
	imshow("原图", M);
	Mat M_erode1 = Mat::zeros(M.rows, M.cols, CV_8UC1);

	
	threshold(M, M_erode1, 10, 255, CV_THRESH_BINARY);

	//针对白色背景的ROI
	Mat M_erode;
	ROI_me(M, M_erode1, M_erode);
	imshow("测试", M_erode);

	//连通域标记（8通路）
	int flag = 1;
	int label = 0;
	
	if (label <= k){
		for (int i = 1; i < M_erode.rows - 1; i++){
			for (int j = 1; j < M_erode.cols - 1; j++){
				if (M_erode.at<uchar>(i, j) == 0){
					label++;
					area.push(pair<int, int>(i, j));
				}
				while (!area.empty()){
					place = area.top();
					int tempx = place.first;
					int tempy = place.second;
					M_erode.at<uchar>(tempx, tempy) = p*label;
					area.pop();
					if (tempx - 1 >= 0){
						if (M_erode.at<uchar>(tempx - 1, tempy) == 0) {
							area.push(pair<int, int>(tempx - 1, tempy));
						}
					}
					if (tempx + 1 <= M_erode.rows - 1){
						if (M_erode.at<uchar>(tempx + 1, tempy) == 0) {
							area.push(pair<int, int>(tempx + 1, tempy));
						}
					}
					if (tempy - 1 >= 0){
						if (M_erode.at<uchar>(tempx, tempy - 1) == 0) {
							area.push(pair<int, int>(tempx, tempy - 1));
						}
					}
					if (tempy + 1 <= M_erode.cols - 1){
						if (M_erode.at<uchar>(tempx, tempy + 1) == 0) {
							area.push(pair<int, int>(tempx, tempy - 1));
						}
					}
					if (tempx + 1 <= M_erode.rows - 1 && tempy + 1 <= M_erode.cols - 1){
						if (M_erode.at<uchar>(tempx + 1, tempy + 1) == 0) {
							area.push(pair<int, int>(tempx + 1, tempy + 1));
						}
					}
					if (tempx - 1 >= 0 && tempy + 1 <= M_erode.cols - 1){
						if (M_erode.at<uchar>(tempx - 1, tempy + 1) == 0) {
							area.push(pair<int, int>(tempx - 1, tempy + 1));
						}
					}
					if (tempx - 1 >= 0 && tempy - 1 >= 0){
						if (M_erode.at<uchar>(tempx - 1, tempy - 1) == 0) {
							area.push(pair<int, int>(tempx - 1, tempy - 1));
						}
					}
					if (tempx + 1 <= M_erode.rows - 1 && tempy - 1 >= 0){
						if (M_erode.at<uchar>(tempx + 1, tempy - 1) == 0) {
							area.push(pair<int, int>(tempx + 1, tempy - 1));
						}
					}
				}
			}
		}
	}
	cout << label;

	namedWindow("3");
	imshow("3", M_erode);




	Mat M_end(M_erode1.rows, M_erode1.cols, CV_8UC3);
	color(M_erode1, M_end);

    //滑动条的创建和使用
	namedWindow("最终");
	createTrackbar("色彩对比度", "最终", &k, k_max, tab);

	imshow("最终", M_end);
	imwrite("最终.bmp", M_end);*/
}

int main(){
	tab(k, 0);
	waitKey();
	return 0;
}
