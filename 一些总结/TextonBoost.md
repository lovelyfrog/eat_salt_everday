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



