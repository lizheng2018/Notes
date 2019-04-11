## 其它

### [GBDT](https://www.zybuluo.com/yxd/note/611571#fn:2)



梯度提升树(Gradient Boosting Decison Tree, 简称GBDT)

对GBDT进行正则化，防止过拟合。GBDT的正则化主要有三种方式。

第一种是和Adaboost类似的正则化项，即**步长**(learning rate)。

定义为ν,对于前面的弱学习器的迭代

![img](https://pic1.zhimg.com/80/v2-22064da8da99b6b7936e733110028210_hd.jpg)

如果加上正则化项，则有

![img](https://pic3.zhimg.com/80/v2-5c199f6884323dbb5dca059505ac6282_hd.jpg)

v的取值范围为 ![0<v\leq 1](https://www.zhihu.com/equation?tex=0%3Cv%5Cleq+1) .对于同样的训练集学习效果，较小的v意味着需要更多的弱学习器的迭代次数。通常用步长和迭代最大次数一起来决定算法的拟合效果。

第二种正则化的方式是通过**子采样比例**（subsample）。取值为(0,1]。

注意这里的子采样和随机森林不一样，随机森林使用的是放回抽样，而这里是不放回抽样。如果取值为1，则全部样本都使用，等于没有使用子采样。如果取值小于1，则只有一部分样本会去做GBDT的决策树拟合。选择小于1的比例可以减少方差，即防止过拟合，但是会增加样本拟合的偏差，因此取值不能太低。推荐在[0.5, 0.8]之间。

使用了子采样的GBDT有时也称作**随机梯度提升树(Stochastic Gradient Boosting Tree, SGBT)**。由于使用了子采样，程序可以通过采样分发到不同的任务去做boosting的迭代过程，最后形成新树，从而减少弱学习器难以并行学习的弱点。

第三种是对于弱学习器即CART回归树进行**正则化剪枝**。












