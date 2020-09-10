## 一. Main Contributions

结合三类信息：

* Textural appearance
* layout
* context

这样可以建模长距离的上下文关系。

基于dense features，能够处理textured 和 untextured objects, 以及multiple objects which inter- or self-occlude。

三个贡献：

* 一个新的feature: texture-layout filter，它记录了patterns of textons，利用the textural appearance of the object, layout和textural context.
* a new discriminative model，结合了texture-layout filters和low-level image features
* 利用boosting 和 piecewise training methods 使得在大数据上的模型训练更加高效

## 二. Image Databases

4个不同的图像数据库：

* MSRC: 591个图片，21个物体类别。45%训练，10%验证，45%测试。包含void labels
* 7-class Corel database
* 7-class Sowerby database
* a set of 9 20-minute video sequences of tv programs

## 三. CRF model

4个potentials:

* Texture-layout potentials
* Color potentials
* location potentials
* edge potentials

### 3.1 CRF上的推断

给定CRF模型和学得的参数，我们想要求出使得概率最大的label ${\bold c^*}$, 通过应用alpha-expansion graph cut算法可以得到the optimal labeling。

该算法的核心在于，将最大化$f({\bold c})$的问题转化成a sequence of binary maximization问题，这些子问题被称作：alpha-expansions。

### 3.2 Learning the CRF

先用了MAP training，发现效果不好，然后引入了piecewise training。

#### 3.2.1 MAP Training

$$
L({\pmb \theta}) = \sum_n \log P({\bf c_n}|{\bold x_n}, {\bold \theta}) + \log P({\pmb \theta})
$$

引入$P(\pmb \theta)$是为了防止过拟合。然后使用梯度上升来更新$\pmb \theta$，但是这样精确推断计算量过大，所以使用alpha-expansion的技巧来近似，也可以使用loopy belief propagation (BP)和variational methods。

但是这样得到的结果不是很好，具体我还没怎么看懂。

### 3.2.2 Piecewise Training

使用piecewise training，CRF中的每一项被单独训练然后用weighting functions重新结合起来。

![image-20200909184919624](/Users/lovelyfrog/Library/Application Support/typora-user-images/image-20200909184919624.png)

根据[44]中，这个训练方法最小化了log partition function的upper bound。定义$z({\pmb \theta}, {\bf x}) = \log Z({\pmb \theta}, {\bf x})$ ，则：
$$
z({\pmb \theta}, {\bf x}) \leq \sum_r z_r({\pmb \theta_r}, {\bf x})
$$
其中${\pmb \theta_r}$是第r个term的参数，$z_r({\pmb \theta_r, {\bf x}})$是只包含第r个term的partition function。在piecewise training中训练的其实就是这个upper bound, 但是让它变大并不一定会让原来的partition function变大，尤其是当这些term都比较相关的时候。

### 3.3 Learning the potentials

* Texture-layout potential parameters: 在boosting时学习

* color potential parameters:

  在test的时候对每张图像单独学习，使用[41]中的方法。首先color clusters通过K-means来学习，然后用一个迭代的算法:reminiscent of EM。