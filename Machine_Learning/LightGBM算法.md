## LightGBM算法

![1554777456553](C:\Users\MarioCode\AppData\Roaming\Typora\typora-user-images\1554777456553.png)

### 0.XGBoost主要特点

* 使用pre-sorted算法，对所有特征按数值进行预排序，能够更精确的找到数据分隔点
* 在每次的样本分割时，用O(# data)的代价找到每个特征的最优分割点，即对整个数据集进行特征样本分割，代价巨大
* 找到最后的特征以及分割点，将数据分裂成左右两个子节点

* 由于需要对特征进行预排序并且需要保存排序后的索引值（为了后续快速的计算分裂点），因此内存需要训练数据的两倍。
* 在遍历每一个分割点的时候，都需要进行分裂增益的计算，消耗的代价大

### 1.LightGBM主要特点

* 带深度限制的Leaf-wise的叶子生长策略
* 基于Histogram的决策树算法
* Histogram图做差加速
* 直接支持类别特征(Categorical Feature)
* Cache命中率优化
* 基于直方图的稀疏特征优化
* 多线程优化

### 2.树生成原理

#### 2.1 Level (depth)-wise

大多数树的生产原理如下： **level (depth)-wise**，按层生长。同一层的所有节点都做分裂，最后剪枝。

![img](https://lightgbm.readthedocs.io/en/latest/_images/level-wise.png)

**优点**：容易多线程优化，好控制模型复杂度，不容易过拟合

**缺点**：比较低效，很多叶子的分裂增益Gain较低，没必要进行搜索和分裂

#### 2.2 Leaf-wise

LightGBM采用的是**Leaf**-wise tree growth, 每次从当前所有叶子中，找到分裂增益最大的一个叶子，然后分裂，如此循环。

![img](https://lightgbm.readthedocs.io/en/latest/_images/leaf-wise.png)

**优点**：同Level-wise相比，在分裂次数相同的情况下，Leaf-wise可以降低更多的误差，得到更好的精度

**缺点**：可能会长出比较深的决策树，产生过拟合。因此LightGBM在Leaf-wise之上增加了一个最大深度的限制max_depth，在保证高效率的同时防止过拟合

### 3.Histogram直方图算法原理

把连续的浮点特征值离散化成k个整数，同时构造一个宽度为k的直方图(bin)。一个区间的范围内作为一个bin，数据的表示更加简化，减少了内存的适用，而且直方图带来了一定的正则化的效果，不易过拟合。在遍历数据的时候，根据离散化后的值生成索引在直方图中累积统计量，当遍历一次数据后，根据直方图的离散值，遍历寻找最优的分割点。

![img](https://fuhailin.github.io/LightGBM/20181126165335641.png)

Histogram优化原理：

![img](https://fuhailin.github.io/LightGBM/20181126164828201.jpg)

1. **`For all Leaf p in Tc-1(X)`**: 表示循环当前树下所有的叶子节点
2. **`For all f in X.Features`**: 开始遍历某个叶子上所有的特征，比如从100特征中找出最佳分裂特征
3. **`For i in(0, num_of_row)`**: 开始遍历所有的样本来构建直方图（样本装箱），直方图的每个 bin 中包含了一定的样本，在此计算每个 bin 中的样本的梯度之和并对 bin 中的样本记数

注意：其中`f.bins[i]`为特征`f`在样本`i`的对应bin；`H[f.bins[i]]`则表示对bin进行累加，该bin就是特征`f`在样本`i`的对应bin

4. **`For i in(0, len(H))`**：开始遍历所有的 bin，找到适合分裂的最佳 bin

SL是当前分裂 bin 左边所有 bin 的集合，对比理解SR，那么SP其中的 P 就是 parent 的意思，就是父节点，传统的决策树会有分裂前后信息增益的计算，典型的 ID3 或者 C45 之类，在这里我们也会计算，但是 SR 中所有 bin 的梯度之和不需要在额外计算了，直接使用父节点的减去左边的就得到。

公式的中 loss 就是来衡量分裂的好坏的，在遍历完所有的特征之后根据 loss，理论上 loss 最小的特征会被选中作为最佳的分裂节点。

**优点**：

1. 内存消耗的降低，pre-sorted算法需要保存起来每一个特征的排序结构，所以其需要的内存大小是：

    （2 * #data * #feature * 4Bytes），如下图

    ![img](https://fuhailin.github.io/LightGBM/20181126170048718.png)

2. 计算效率提高，pre-sorted算法需要对每个特征下样本都遍历一遍，并计算增益，复杂度为O(#feature * #data)。而直方图算法在建立完直方图后，只需要对每个特征遍历直方图即可，复杂度为O(#feature * #bins)

3. 数据并行时，直方图算法可以**大幅降低通信代价**

**缺点**：

1. 特征被离散化后（装箱），不能精确分割样本
2. 直方图不能对稀疏样本进行优化，只是计算累加值（累加梯度和样本数），但是，LightGBM 对稀疏进行了优化：**只用非零特征构建直方图**

**注意**：

* 虽然直方图分割后精度变差，但是对最后结果的影响不是很大，主要由于基模型是决策树，分割点是不是精确并不是太重要
* 较粗的分割点也有正则化的效果，可以有效地防止过拟合
* 即使单棵树的训练误差比精确分割的算法稍大，但在梯度提升（Gradient Boosting）的框架下没有太大的影响

#### 3.1 Histogram差加速

![1554723182923](C:\Users\MarioCode\AppData\Roaming\Typora\typora-user-images\1554723182923.png)

一个叶子的直方图可以由它的父节点的直方图与它兄弟的直方图做差得到，计算速度可以提升约一倍

#### 3.2 Histogram算法两大改进

Histogram算法在建立直方图时复杂度为O(#feature×#data)，如果能**降低特征数feature**或者**降低样本数data**，训练的时间会大大减少
##### 3.2.1 Gradient-based One-Side Sampling（GOSS）

GOSS: 用于减少训练样本数（行）

**原理**：如果一个样本的**梯度**很小，说明该样本的训练误差很小，或者说该样本已经得到了很好的训练(well-trained)。要减少样本数，一个直接的想法是抛弃那些梯度很小的样本，但是这样训练集的分布会被改变，可能会使得模型准确率下降。但是，GOSS弥补了这个缺点。使用GOSS进行采样，使得训练算法更加的关注没有充分训练(under-trained)的样本，并且只会稍微的改变原有的数据分布。

注意：梯度对于LightGBM来说，跟权重向量w对于AdaBoost一样都很好的反应了样本的重要性。

##### 3.2.1 Exclusive Feature Bundling（EFB）

EFB: 用于减少训练特征（列）

**原理**：一个有高维特征空间的数据往往是稀疏的，而稀疏的特征空间中，许多特征是互斥的。所谓互斥就是他们从来不会同时具有非0值（一个典型的例子是进行One-hot编码后的类别特征）。

EFB算法可以**进行互斥特征的合并，从而减少特征的数目**。做法是先确定哪些互斥的特征可以合并（可以合并的特征放在一起，称为bundle），然后将各个bundle合并为一个特征。

复杂度将从O(#feature * #data)变为O(#bundle * #data)，而#bundle<<#feature，这样GBDT能在精度不损失的情况下进一步提高训练速度。

关于怎么哪些特征可以合并以及怎么合并的细节暂不讨论。

### 4.提高缓存命中率Increase cache hit chance

这个涉及计算机系统原理，不易懂

### 5.支持分类特征

![img](https://fuhailin.github.io/LightGBM/20181126170238801.png)

现有算法需要将类别特征转换成0/1值，才能进行建模，例如，One-Hot编码，特价了数据集维度，时间和空间利用效率大大降低。

LightGBM直接支持类别特征，有实验表明，这种方法比类别特征编码方法训练速度快大约8倍（数据集全为类别特征时）

### 6.并行计算优化

![img](https://fuhailin.github.io/LightGBM/2018112617025697.png)



* Feature Parallelization：适用于小数据且feature比较多的场景
* Data Parallelization：适用于数据量比较大，但feature比较少的场景
* Voting Parallelization：适用于数据量比较大，feature也比较多的场景

#### 6.1 Feature Parallelizaton 特征并行

特征并行主要是并行化决策树中寻找最优分割点的过程，这部分最为耗时

![img](https://www.hrwhisper.me/wp-content/uploads/2018/07/LightGBM-feature-parallelization.png)

**现有算法特征并行过程**：（注意：worker是几个特征的集合，数量根据实际运行时调整）

1. 垂直划分数据（**对特征划分**），不同的worker有**不同的特征集**
2. 每个workers找到局部最佳的切分点
3. workers使用点对点通信，找到全局最佳切分点
4. 具有全局最佳切分点的worker进行节点分裂，然后广播切分后的结果（左右子树的instance indices）
5. 其它worker根据收到的instance indices也进行划分

**现有算法缺点**：

1. 无法加速split的过程，该过程复杂度为O(#data)，当数据量大的时候效率不高
2. 需要全局广播划分的结果（左右子树的instance indices），1条数据1bit的话，大约需要花费O(#data/8)

**LightGBM优化过程**：

每个worker**保存所有的数据集**，这样找到全局最佳切分点后各个worker都可以自行划分，就不用进行广播划分结果，减小了网络通信量。

1. 每个workers找到局部最佳的切分点
2. workers使用点对点通信，找到全局最佳切分点
3. 每个worker根据全局全局最佳切分点进行节点分裂

**缺点**：

1. split过程的复杂度仍是O(#data)，当数据量大的时候效率不高
2. 每个worker保存所有数据，存储代价高

#### 6.2 Data Parallelization 数据并行

![img](https://www.hrwhisper.me/wp-content/uploads/2018/07/LightGBM-data-parallelization.png)

**现有算法数据并行过程**：

1. 水平切分数据，不同的worker拥有部分数据
2. 每个worker根据本地数据构建局部直方图
3. 合并所有的局部直方图得到全部直方图，合并过程通信代价巨大（主要缺点）
4. 根据全局直方图找到最优切分点并进行分裂

**LightGBM优化过程**

1. 使用“Reduce Scatter”将不同worker的不同特征的直方图合并，然后workers在局部合并的直方图中找到局部最优划分，最后同步全局最优划分
2. 直方图做差加速，因此只需要通信一个节点的直方图，通信代价降低一半







