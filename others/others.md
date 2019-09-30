```
#include "yx_test.h"
#include "GeometryTool.h"

YXTest::YXTest() {
}

YXTest::~YXTest() {
}

bool Pline::isPointInDisYx(const Point3D& p) {
	if (this->lineSegment.size() != 2 || this->lineSegment.at(0).isTheSame(this->lineSegment.at(1)))
		// 该线段的端点数不为2，或2个点相同
		return false;

	// 获取点到当前多线段的距离
	if (getDistanceOfPointToLine(p) < error_equal_point_coord)
		return true;
	return false;
}

bool Pline::isPointInLineYx(const Point3D& p) {
	return isPointInRangeOfPline(p);
}

// 4. 获取重叠端点 from yx
int * GeometryTool::dealPlineCrossPlineYx(Point3D & pointIS, Point3D & pointIE, Point3D & pointJS, Point3D & pointJE) {
	// 新建局部变量
	Pline pline;
	pline.lineSegment.push_back(pointIS);
	pline.lineSegment.push_back(pointIE);

	// 返回值数组, 0是被替换位置, 1是替换位置
	int * ret;
	ret = new int[2];
	ret[0] = -1;
	ret[1] = -1;

	// 先确保点到线段的距离在差值之内
	if (pline.isPointInDisYx(pointJS) && pline.isPointInDisYx(pointJE)) {
		// 获取js, je位置信息
		bool sInLine = pline.isPointInLineYx(pointJS);
		bool eInLine = pline.isPointInLineYx(pointJE);

		if (sInLine && eInLine) {
			ret[0] = 3;
			ret[1] = 4;
			return ret;
		}

		Pline pline2;
		pline2.lineSegment.push_back(pointJS);
		pline2.lineSegment.push_back(pointJE);

		if (sInLine) {
			// 判断ie在不在线段js-je上面, 在, 返回js-ie, 不在, 返回js-is
			if (pline2.isPointInLineYx(pointIE)) {
				ret[0] = 3;
				ret[1] = 2;
			}
			else {
				ret[0] = 3;
				ret[1] = 1;
			}
			return ret;
		}

		if (eInLine) {
			// 判断ie在不在线段js-je上面, 在, 返回je-ie, 不在, 返回je-is
			if (pline2.isPointInLineYx(pointIE)) {
				ret[0] = 4;
				ret[1] = 2;
			}
			else {
				ret[0] = 4;
				ret[1] = 1;
			}
			return ret;
		}
	}

	return ret;
}

// 3. 处理重叠线 from yx
bool Pline::dealCrossPlineYx(Pline & pline, vector<pline>& polyLineList){
	int paramPlineSize = pline.lineSegment.size();
	int plineSize = lineSegment.size();

	if (paramPlineSize < 2 || plineSize < 2) {
		return false;
	}

	// 检测子线段是否重叠
	GeometryTool gt;
	// 跳出标志
	bool flag = false;

	for (int i = 1; i < plineSize; ++i) {
		// 交叉结果返回值
		int * crossRet = new int[2];
		crossRet[0] = -1;

		// 获取子线段
		Point3D& pointIS = lineSegment[i - 1];
		Point3D& pointIE = lineSegment[i];

		for (int j = 1; j < paramPlineSize; ++j) {
			Point3D& pointJS = pline.lineSegment[j - 1];
			Point3D& pointJE = pline.lineSegment[j];

			// i线段是不变线段, j是处理线段

			// 线段重叠判断
			crossRet = gt.dealPlineCrossPlineYx(pointIS, pointIE, pointJS, pointJE);
			
			if (crossRet[0] != -1) {
				// 存在重叠
				flag = true;

				// 头部删除, 只需要新增尾段
				// 中间删除, 需要新增头段和尾段
				if (j - 1 != 0){
					// 新增头段
					Pline a = new Pline ();
					// 新建内容需要依据crossRet
					// 3, 4 0-js je-(size-1)
					// 3, 1 0-js is-(size-1)
					// 3, 2 0-js ie-(size-1)
					// 4, 1 0-is je-(size-1)
					// 4, 2 0-ie je-(size-1)
					// 0-js
					for (int k = 0; k <= j - 1; ++k){
						// 循环填充内容
					}
					if (crossRet[0] == 4 && crossRet[1] == 1){
						// 0-is 0-js + js-is
					}
					if (crossRet[0] == 4 && crossRet[1] == 2){
						// 0-ie 0-js+js-ie
					}
					polyLineList.add (a);
				}
				// 新增尾段
				Pline b = new Pline ();
				// je-(size-1)
				for (int l = j; l < pline.lineSegment.size (); ++l){
					// 循环填充内容
				}
				if (crossRet[0] == 3 && crossRet[1] == 1){
					// is-(size-1) is-je + je-(size-1)
				}
				if (crossRet[0] == 3 && crossRet[1] == 2){
					// ie-(size-1) ie-je + je-(size-1)
				}
				polyLineList.add (b);

				break;
			}
		}
		// 新增梁线
		if (flag){
			// 跳出之后返回, 删除当前被拆分梁线
			break;
		}
	}
	return flag;
}

// 2. 矩形重叠检测 from yx
bool GeometryTool::isRectCrossRectYx(Point3D & pointIS, Point3D & pointIE, Point3D & pointJS, Point3D & pointJE){
	double minX = pointIS.getX() - error_the_same_point_coord;
	double maxX = pointIE.getX() + error_the_same_point_coord;
	double minY = pointIS.getY() - error_the_same_point_coord;
	double maxY = pointIE.getY() + error_the_same_point_coord;

	double minX2 = pointJS.getX() - error_the_same_point_coord;
	double maxX2 = pointJE.getX() + error_the_same_point_coord;
	double minY2 = pointJS.getY() - error_the_same_point_coord;
	double maxY2 = pointJE.getY() + error_the_same_point_coord;

	return !(maxx <= minx2 ||		// left
			 maxy <= miny2 ||		// bottom
			 minx >= maxx2 ||		// right
			 miny >= maxy2);		// top
}

// 1. 删除多余的线
void yxtest::deleteoverlappline(vector<pline>& polylinelist) {
	geometrytool gt;

	// 获取多段线的个数
	//int size = polylinelist.size();

	int i = 0;
	while (i < polylinelist.size() - 1){
		// i层是不动层, 
		// 由i对j发起比较, j处理之后可能会变化, 而i不变
		// 获取多段线i
		pline& plineI = polylinelist[i];

		// 获取i包围矩形
		point3d plineImin, plineImax;
		plineI.getenvelopepoints(plineImin, plineImax);

		int j = i + 1;
		// 新增标志
		bool addNew = false;
		while (j < polylinelist.size()){
			addNew = false;
			// 获取多段线j
			pline& plineJ = polylinelist[j];

			// 获取j包围矩形
			point3d plineJmin, plineJmax;
			plineJ.getenvelopepoints(plineJmin, plineJmax);

			// 包围矩形相交检测
			if (gt.isRectCrossRectYx(plineImin, plineImax, plineJmin, plineJmax)){
				// 重叠处理
				if (plineI.dealcrossplineyx(plineJ, polylinelist)){
					// 删除了重叠子段
					polylinelist.erase (j);
					addNew = true;
				}
			}

			if (!addNew)
				j++;	
		}
		i++;
	}
}

```
