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

  在test的时候对每张图像单独学习，使用[41]中的方法。首先color clusters通过K-means来学习，然后用一个迭代的算法:reminiscent of EM。没怎么看懂

* location potential parameters:
	$$
  {\pmb {\theta}}_{\lambda}(c, \hat{i}) = \left(\frac{N_{c,\hat{i}}+\alpha_{\lambda}}{N_{\hat{i}}+\alpha_{\lambda}} \right)^{w_{\lambda}}
  $$
  
  其中$N_{c,\hat{i}}$是在normalized location $\hat{i}$上的标签为$c$类的像素个数，$N_{\hat{i}}$是总像素个数，该式可以看作是在位置$\hat{i}$上像素类别为$c$的概率。
  
* Edge potential parameters: manually selected 来减小在validation set上的error

## 四. Boosted Learning of Texture, Layout and Context

上述四个中最重要的term就是unary texture-layout potential $\psi$，它基于一组新的features，叫做texture-layout filters。

### 4.1 Textons

textons被证明在categorize material 和 generic class objects上是高效的。



![image-20200910104305361](/Users/lovelyfrog/Library/Application Support/typora-user-images/image-20200910104305361.png)

textonization的过程：每张图像都被一个17D的filter-bank卷积，得到的17D responses然后被whitened，然后使用在Euclidean-distance上的K-means 聚类（这一步具体怎么聚类的？），最后图像中的每个像素都会被分配给最近的cluster center，得到texton map。

### 4.2 Texture-Layout filters

每个texture-layout filter都是一个对于一个图像区域$r$和一个texton $t$的一个pair $(r, t)$，其中区域$r$是定义在像素$i$附近的区域，这里只考虑长方形区域。一组candidate rectangles被随机选择，使得他们能够cover图像的一半区域，在像素$i$的feature response是在offset region $r+i$上texton为t的像素的概率
$$
v_{[r,t]}(i) = \frac{1}{area(r)} \sum_{j \in (r+i)} [T_j = t]
$$
filter responses 可以用integral images[48]来高效的计算，$\hat{T}^{(t)}$是$T$对于texton channel $t$的integral image，那么feature response可以这样计算：
$$
v_{[r,t]}(i) = \hat{T}^{(t)}_{r_{br}} - \hat{T}^{(t)}_{r_{bl}} - \hat{T}^{(t)}_{r_{tr}} + \hat{T}^{(t)}_{r_{tl}}
$$
其中$r_{br}, r_{bl}, r_{tr}, r_{tl}$指得是bottom right, bottom left, top right, top left corners of rectangle $r$。

它可以让我们自动学到layout和context 信息。

![image-20200910145044304](/Users/lovelyfrog/Library/Application Support/typora-user-images/image-20200910145044304.png)

### 4.3 variations on texture-layout filters

作者实验了不同的region shape $r$，但是发现效果都并没有怎么样的好。

#### 4.3.1 texton-dependent layout filters

Texton-dependent layout filter与standard texture-layout filter一样，除了它用的是在要被分类的像素$i$处的sexton $T_i$，而不是一个particular learned sexton。feature response 就是 $v_{[r, T_i]}(i)$

#### 4.3.2 separable texture-layout filters

对于特定的情况，比如处理视频序列或大图像，使用一个新的filter。使用spans来替代rectangles：horizontal spans计算离要分类的像素的y坐标相邻的horizontal strip中，与texton index一致的比例。vertical spans 也是类似的。

![image-20200910162650344](/Users/lovelyfrog/Library/Application Support/typora-user-images/image-20200910162650344.png)

### 4.4 Learning texture-layout filters using Joint Boost

使用了Joint Boost算法的变式[45]，