# AF
AF用于实现自动对焦的模块，本文介绍的主要是是展锐平台的CDAF以及PDAF

PDAF PDPC 原理作用解释
PDAF & PD-PC (Phase Detection Auto Focus & Phase Pixel Correction) - 相位检测自动对焦与相位像素矫正

作用：PDAF用于实现快速、准确的自动对焦；PD-PC（或PPE） 用于矫正用于PDAF的专用像素（相位检测像素）对成像造成的干扰。
原理：

PDAF：Sensor上集成了一些被遮蔽的像素对（左/右或上/下）。通过比较这对像素接收光信号的相位差，可以直接计算出离焦方向和距离，驱动镜头马达快速移动到合焦位置。
PD-PC (PPE)：用于PDAF的遮蔽像素（PD Pixel）本身不参与成像，其输出值需要被邻近的正常成像像素值替换或插值，否则会在图像上形成规则排列的白点或黑点（如文档案例中的shield pd—B点）。PPE模块就是专门负责抽取和矫正这些相位点的算法。

调试影响：PDAF决定对焦速度和准确性；PD-PC影响画质，未矫正的PD点会形成固定图案噪声。


## CDAF
CDAF 的原理图如图下图所示，相机对焦系统搜寻被摄物体主体边缘的对比度，驱动镜头沿着指向被摄主
体轴线改变对焦点，并在每个对焦点上获取影像，先将在每个对焦点上获得的影像数字化，求出图像的
反差值，再将每个对焦点上得到的反差值进行比较，得到最大值，驱动镜头，将焦点放置于反差值最大
的对焦点上，即得到正确的焦点完成精确对焦。

<img src="./phone/AF_1.png " width="500" height="300">

## PDAF
PDAF 是把 CMOS 两个感应器的讯号（L和 R 两组像素）进行比较，计算出相位差，通过 DCC（Defocus Conversion Coefficient，离焦转换系数）换算成对应的马达位置，找到对应的马达位置，推动马达到目标位置进行精确对焦。

当被摄物体合焦时，从该物体发出的光线通过镜头后，会精确汇聚在传感器成像面上。此时，成对的L像素和R像素接收到的光信号完全一致，相位差为0。
当被摄物体失焦时（例如前景或后景），光线无法精确汇聚。对于前景失焦（镜头焦点在被摄物体后方），光线会在传感器前交叉，导致L像素和R像素接收到的图像存在位置偏移。这个偏移量就是相位差。
## AF调试
### 标定
针对AF的模块首先需要进行标定，准备4台机器进行拍摄标定。开启全扫模式拍摄10cm的标定图按照水平，俯拍，仰拍进行拍摄，总计12张，同理拍摄室外远景合计12张准备完后

<img src="./phone/AF_2.png " width="500" height="300">

点击"OpenAll" 导入图片后工具将会自动生成所有图片的准焦Position，再点击“Calculate”,工具将自动计算出 Scan Settings 中 downward_ratio 和 upward_ratio 的值
图中所有参数需满足下图

<img src="./phone/AF_3.png " width="500" height="300">

修改标定完的值可以直接的修改马达的行程

### CDAF turing
接下来会重点介绍其中个别几个的控制功能

首先是OVERvidew 功能
<img src="./phone/AF_4.png " width="500" height="300">
```
flat_no，falling_rto,rising_rto.这三个可以对pos值曲线是平坦还是下降，上升进行阈值判断，调整判定曲线的形态
可以通过调整break_count 以及turnback_count 来调节判定峰值，当下降的点数大于这个阈值的时候会被判定为峰值点。
```

Ratio_turing

<img src="./phone/AF_5.png " width="500" height="300">
···
通过修改search_interval 增加进入细扫的idx阈值，可以让对焦更加精准
search_ratio 会改变进入细扫的曲线两边FV值变化率阈值。当曲线两边的FV值下降率大于这个值的时候会进入细扫
break_rto和break_ent则是修正进入细扫后判断峰值点
···