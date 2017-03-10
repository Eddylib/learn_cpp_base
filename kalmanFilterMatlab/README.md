卡尔曼滤波跟踪鼠标位置的演示小程序
蓝色为真实值
红色为后验估计值
绿色为观测值

使用：运行ekfmyappjaccsdf.m即可。

卡尔曼滤波个人理解:
对于一个大致知道其运动模型,且有一些不准确的观察点的情况下。
需要一种方法将这两个信息结合起来组成更可靠的信息。
通常方法自然就是加权求和.
但是加权求和的权重需要考虑,不能人为给出.
而卡尔曼滤波的核心要义就是求出这一个权重,这一权重也叫作增益因子。
其获得的思路是:
使得估计值(先验)和真实值(不存在,无法给出,仅仅是个符号,在求导后被消去)间误差的均方差最小化。
具体推导见：http://blog.csdn.net/heyijia0327/article/details/17487467

这就造成了卡尔曼滤波只能应对线性系统。而这里说的线性系统指的是运动模型的递推公式满足线性(运动模型本身可以是随时间的二次函数)。在求导时需要删去状态自变量,于是观测模型也需要是线性的(即)。

针对这一限制,才有了扩展卡尔曼滤波。
其实也简单,就是直接取运动模型和观测模型泰勒展开。然后省略高阶导数项，把系统直接就线性化了。之后就是普通的卡尔曼滤波。
而这会造成误差较大。写的程序是根据http://blog.csdn.net/lizilpl/article/details/45289541
提供的代码改编而成。
暂时感觉对卡尔曼滤波还是有一些困惑，代码效果也不是很好，结果和标准差参数相关性很大。还是不明白为何是这个样子。。。继续学习

---------------------------------------------------------------------------------------------------
2017.3.9更新：
从state estimation for robotics 上重温了卡尔曼滤波。完全理解了卡尔曼滤波的一个推导来源（也就是说卡尔曼滤波可以从不同的方向去解释它的由来）。
最近几天由于有SEFR这本书，加上老师指导，回顾上个版本写的代码即“refine to a standard project structure”这次提交，以下简称RTA。发现RTA版本代码确实有问题。
一、RTA对运动模型和观测模型的处理不太恰当：
运动模型直接在真实值上加了一个高斯噪声。观测模型模型也是在真实值上加了一个高斯噪声。这样就造成了实际是对真实值的两个高斯噪声观测的滤波。而经过老师指导发现，运动模型不是一个对真实值的观测，而是来自于对自身的判断，如根据自身位置，速度，加速度来判断自己的状态。典型的模型有匀速运动模型，IMU给予信息的状态更新。而观测模型则是对外界信息的观测，典型的就是通过激光、摄像机、GPS等传感器对自身的观测。
当然，我个人觉得，卡尔曼滤波本身就是一个融合多个传感器的信息。这里是否不需要划分如此清晰？由此引出一个疑
-------------->问，多传感器信息又如何融合？-----这个问题我会继续思考，有了结果会在这里跟新。
二、RTA版本将鼠标跟踪问题看成了一个非线性系统：
有了近几日的学习，我发现这是个错误的看法。首先观测模型在模拟程序中就是直接对点的探测，本身就是线性的。而如果运动模型也是线性的话，那所有部分就都是线性的了。所以没必要用所谓的扩展卡尔曼滤波。

-------------->引出问题，到底什么是线性系统，什么是非线性系统？是指观测模型和运动模型都为线性才为线性系统？
2017.3.9晚
完成代码.又出现问题:系统转移矩阵一般采用齐次坐标,引入了一个新的维度,然而新的维度引入,方差该怎么处理?
如果对应那一维度的方差为0,会造成求卡尔曼增益时,协方差会是奇异矩阵.
目前处理方法就是采用齐次坐标,在求卡尔曼增益时把多余的一位割掉求增益...不知这种处理方法是否正确...


结果是:估计值的总误差比观测值的总误差还要大!!!!!!!!
这是个令人发狂的结果...做了半天还不如直接去观测呢!

反思如下:
-------------->对齐次坐标的协方差处理不太正确导致的误差....(换1试试...，效果好一些但又觉得不合理，得从公式的推到上来分析一下)
调参影响还是很大....过程:测量的标准差比为15:30时效果较好
要想使估计结果更好,必须有正确的运动模型和合理的方差设置。

