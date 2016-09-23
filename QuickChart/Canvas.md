# Canvas常用操作速查表

| 操作分类       | 相关API                                    | 备注                                       |
| ---------- | ---------------------------------------- | ---------------------------------------- |
| 绘制颜色       | drawColor, drawRGB, drawARGB             | 使用单一颜色填充整个画布<br/> **相关链接:**<br/> [【基础－颜色】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Base/%5B3%5DColor.md) <br/>[【Canvas－颜色与基本形状】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B2%5DCanvas_BasicGraphics.md) |
| 绘制基本形状     | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧 <br/> **相关链接 :** <br/>[【Canvas－颜色与基本形状】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B2%5DCanvas_BasicGraphics.md) |
| 绘制图片       | drawBitmap, drawPicture                  | 绘制位图和图片<br/>**相关链接:** <br/>[【Canvas－图片文字】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) |
| 绘制文本       | drawText,    drawPosText, drawTextOnPath | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字<br/>**相关链接:** <br/>[【Canvas－图片文字】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) |
| 绘制路径       | drawPath                                 | 绘制路径，绘制贝塞尔曲线时也需要用到该函数 <br/>**相关链接:** <br/> [【Path－基本操作】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B5%5DPath_Basic.md) <br/> [【Path－贝塞尔曲线】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B6%5DPath_Bezier.md) <br/> [【Path－完结篇(伪)】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B7%5DPath_Over.md) <br/> [【Path－Path玩出花样(PathMeasure)】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B08%5DPath_Play.md) |
| 顶点操作       | drawVertices, drawBitmapMesh             | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用 |
| 画布剪裁       | clipPath,    clipRect                    | 设置画布的显示区域                                |
| 画布快照       | save, restore, saveLayerXxx, restoreToCount, getSaveCount | 依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 会滚到指定状态、 获取保存次数<br/>**相关链接:** <br/>[【Canvas－画布操作】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B3%5DCanvas_Convert.md) |
| 画布变换       | translate, scale, rotate, skew           | 依次为 位移、缩放、 旋转、错切<br/> **相关链接:** <br/>[【基础－坐标系】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Base/%5B1%5DCoordinateSystem.md)    <br/>[【基础－角度弧度】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Base/%5B2%5DAngleAndRadian.md)  <br/>[【Canvas－画布操作】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B3%5DCanvas_Convert.md) |
| Matrix(矩阵) | getMatrix, setMatrix, concat             | 实际画布的位移，缩放等操作的都是图像矩阵Matrix，只不过Matrix比较难以理解和使用，故封装了一些常用的方法。<br/>[【Matrix－原理】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B09%5DMatrix_Basic.md) |

| 操作类型       | 相关API                                    | 备注                                       |
| ---------- | ---------------------------------------- | ---------------------------------------- |
| 基础方法       | getDensity, getWidth, getHeight，getDrawFilter，isHardwareAccelerated(API 11)，getMaximumBitmapWidth，getMaximumBitmapHeight，getDensity，quickReject，isOpaque，setBitmap，setDrawFilter | 使用单一颜色填充画布                               |
| 绘制颜色       | drawColor, drawRGB, drawARGB，drawPaint   | 使用单一颜色填充画布                               |
| 绘制基本形状     | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧                  |
| 绘制图片       | drawBitmap, drawPicture                  | 绘制位图和图片                                  |
| 绘制文本       | drawText,    drawPosText, drawTextOnPath | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字          |
| 绘制路径       | drawPath                                 | 绘制路径，绘制贝塞尔曲线时也需要用到该函数                    |
| 顶点操作       | drawVertices, drawBitmapMesh             | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用 |
| 画布剪裁       | clipPath,    clipRect， clipRegion，getClipBounds | 画布剪裁相关方法                                 |
| 画布快照       | save, restore, saveLayer, saveLayerXxx, restoreToCount, getSaveCount | 依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 回滚到指定状态、 获取保存次数 |
| 画布变换       | translate, scale, rotate, skew           | 依次为 位移、缩放、 旋转、错切                         |
| Matrix(矩阵) | getMatrix, setMatrix, concat             | 实际画布的位移，缩放等操作的都是图像矩阵Matrix，只不过Matrix比较难以理解和使用，故封装了一些常用的方法。 |
