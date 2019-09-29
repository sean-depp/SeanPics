# Others

```
#include "yx_test.h"
#include "GeometryTool.h"

YXTest::YXTest()
{
}

YXTest::~YXTest()
{
}

// 矩形重叠检测来自yx
bool GeometryTool::isRectCrossRectYx(Point3D & pointIS, Point3D & pointIE, Point3D & pointJS, Point3D & pointJE){
	// 0 1 2 3 minX minY maxX maxY
	double minX = pointIS.getX() - error_the_same_point_coord;
	double maxX = pointIE.getX() + error_the_same_point_coord;
	double minY = pointIS.getY() - error_the_same_point_coord;
	double maxY = pointIE.getY() + error_the_same_point_coord;

	// 0 1 2 3 minX2 minY2 maxX2 maxY2
	double minX2 = pointJS.getX() - error_the_same_point_coord;
	double maxX2 = pointJE.getX() + error_the_same_point_coord;
	double minY2 = pointJS.getY() - error_the_same_point_coord;
	double maxY2 = pointJE.getY() + error_the_same_point_coord;

	return !(maxX <= minX2 ||		// left
		maxY <= minY2 ||		// bottom
		minX >= maxX2 ||		// right
		minY >= maxY2);		// top
}

bool Pline::isPointInDisYx(const Point3D& p)
{
	if (this->lineSegment.size() != 2 || this->lineSegment.at(0).isTheSame(this->lineSegment.at(1))){
		// 该线段的端点数不为2，或2个点相同
		return false;
	}

	// 获取点到当前多线段的距离
	if (getDistanceOfPointToLine(p) < error_equal_point_coord)
		return true;
	return false;
}

bool Pline::isPointInLineYx(const Point3D& p)
{
	return isPointInRangeOfPline(p);
}


int * GeometryTool::isPlineCrossPlineYx(Point3D & pointIS, Point3D & pointIE, Point3D & pointJS, Point3D & pointJE) {
	Pline pline;
	pline.lineSegment.push_back(pointIS);
	pline.lineSegment.push_back(pointIE);

	int * ret;
	ret = new int[2];

	// 先确保点到线段的距离在差值之内
	if (pline.isPointInDisYx(pointJS) && pline.isPointInDisYx(pointJE)) {
		// 返回重叠部分

		bool sInLine = pline.isPointInLineYx(pointJS);
		bool eInLine = pline.isPointInLineYx(pointJE);

		if (sInLine && eInLine) {
			// 直接return 
			ret[0] = 3;
			ret[1] = 4;
			return ret;
		}

		Pline pline2;
		pline2.lineSegment.push_back(pointJS);
		pline2.lineSegment.push_back(pointJE);

		if (sInLine) {
			// 判断ie在不在线段js-je上面, 在返回, js-ie, 不在, 返回is-js
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
			// 判断ie在不在线段js-je上面, 在返回, je-ie, 不在, 返回is-je
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
	else {
		ret[0] = -1;
		ret[1] = -1;
		return ret;
	}
}

bool Pline::isCrossPlineYx(Pline & pline) {
	// 根据多段线个数过滤 多段线内不存在3点一线的情况
	int paramPlineSize = pline.lineSegment.size();
	int plineSize = lineSegment.size();

	if (plineSize < paramPlineSize || plineSize < 2) {
		// 少于2个没有检测意义
		return false;
	}
	// 确保传入的梁线子线段少
	/*if (plineSize < paramPlineSize || paramPlineSize < 2)
		return false;*/

	// 判断子线段是否重叠
	GeometryTool gt;

	for (int j = 1; j < plineSize; ++j)
	{
		bool flag = false;
		// 交叉结果返回值
		int * crossRet = new int[2];
		crossRet[0] = -1;

		Point3D& pointJS = lineSegment[j - 1];
		Point3D& pointJE = lineSegment[j];

		for (int i = 1; i < paramPlineSize; ++i)
		{
			Point3D& pointIS = pline.lineSegment[i - 1];
			Point3D& pointIE = pline.lineSegment[i];

			// 线段重叠判断
			crossRet = gt.isPlineCrossPlineYx(pointJS, pointJE, pointIS, pointIE);
			
			if (crossRet[0] != -1) {
				// 存在重叠
				// 进行删除操作之前, 要确保头尾信息不丢失
				if (j - 1 == 0) {
					// 如果j-1是0, 可以直接移动
					if (crossRet[0] == 3) {
						// 3
						if (crossRet[1] == 4) 
							pline.lineSegment[i - 1] = pointIE;
						if (crossRet[1] == 1) 
							pline.lineSegment[i - 1] = pointJS;
						if (crossRet[1] == 2) 
							pline.lineSegment[i - 1] = pointJE;
					}
					else {
						// 4
						if (crossRet[1] == 4)
							pline.lineSegment[i] = pointIE;
						if (crossRet[1] == 1)
							pline.lineSegment[i] = pointJS;
						if (crossRet[1] == 2)
							pline.lineSegment[i] = pointJE;
					}
				}
				else {
					flag = true;
					// 如果j-1是中间段, 要新建梁线, 添加到vector
					// 起点之前, 终点之后建立两个梁线
					// 起点之前加入vector, 终点之后替换当前
					if (crossRet[1] == 3) {
						// 3

					}
					else {
						// 4
					}
				}
				break;
			}
		}
		// 新增梁线跳过
		if (flag)
			break;
	}
}

// 1.
void YXTest::deleteOverlapPline(vector<Pline>& polyLineList) {
	GeometryTool gt;

	// 获取多段线的个数
	int size = polyLineList.size();

	for (int i = 0; i < size; ++i)
	{
		Pline& plineI = polyLineList[i];

		// 获取i包围矩形的对角点
		Point3D plineIMin, plineIMax;
		plineI.getEnvelopePoints(plineIMin, plineIMax);

		for (int j = i + 1; j < size; ++j)
		{
			Pline& plineJ = polyLineList[j];

			// 获取j包围矩形的对角点
			Point3D plineJMin, plineJMax;
			plineJ.getEnvelopePoints(plineJMin, plineJMax);

			// 包围矩形相交检测
			if (gt.isRectCrossRectYx(plineIMin, plineIMax, plineJMin, plineJMax)){
				// 线段重叠处理
				plineI.isCrossPlineYx(plineJ);
				//plineJ.isCrossPlineYx(plineI);
				/*if (ret == 0) {
					delPline(plineI);
				}
				if (ret == 3) {
					delPline(plineJ);
				}
				if (ret == 1) {
					fixPline(s);
				}
				if (ret == 2) {
					fixPline(e);
				}*/
				//if (plineI.isCrossPlineYx(plineJ) == 1) {
				//	// 1.如果两点都在线段内部, 可以直接删除这个段
				//	delPline();
				//}
				//else if (plineI.isCrossPlineYx(plineJ) == 2) {
				//	// 2.一个在里面, 执行替换值操作
				//	fixPline();
				//}
				
				//if (plineJ.isCrossPlineYx(plineI)) {
				//	// 修正在线上点的坐标值
				//	fixPoint();
				//}
			}

			//if (gt.isRectInRect(plineIMin, plineIMax, plineJMin, plineJMax))
			//{
			//	if (plineI.isOverlapPline(plineJ))
			//	{
			//		// 删除多段线J
			//		plineJ = polyLineList[size - 1];
			//		polyLineList.pop_back();
			//		// 调整索引
			//		size -= 1;
			//		j -= 1;
			//	}
			//	break;
			//}
		}
	}
}
```
