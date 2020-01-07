---
title: PASCAL VOC 数据集格式
tags: ["dataset"]
categories: ["machine-learning"]
reward: true
copyright: true
date: 2020-01-07 15:18:30
thumbnail: pascal-voc-dataset-format/pascal-logo.png
---



本文介绍 PASCAL VOC 数据集的标注格式。

+ PASCAL的全称是Pattern Analysis, Statistical Modelling and Computational Learning

+ VOC的全称是Visual Object Classes

详细的官方说明见[此处](http://host.robots.ox.ac.uk/pascal/VOC/)。

<!--more-->



# 数据集目录结构

```
.
└── VOCdevkit     #根目录
    └── VOC2012   #不同年份的数据集，目前最新是2012
        ├── Annotations        #存放xml文件，与JPEGImages中的图片一一对应，解释图片的内容等等
        ├── ImageSets          #该目录下存放的都是txt文件，txt文件中每一行包含一个图片的名称，末尾会加上±1表示正负样本
        │   ├── Action
        │   ├── Layout
        │   ├── Main
        │   └── Segmentation
        ├── JPEGImages         #存放源图片
        ├── SegmentationClass  #存放的是图片，语义分割相关
        └── SegmentationObject #存放的是图片，实例分割相关
```

+ 必备的目录只有Annotations, JPEGImages 和 ImageSets/Main。
+ Annotations 文件夹存放的是xml文件，该文件是对图片的解释，每张图片都对于一个同名的xml文件。
+ ImageSets文件夹存放的是txt文件，这些txt将数据集的图片分成了各种集合。如Main下的train.txt中记录的是用于训练的图片集合
+ JPEGImages文件夹存放的是数据集的原图片
+ SegmentationClass以及SegmentationObject文件夹存放的都是图片，且都是图像分割结果图
  

## Annotations 目录

```
Annotations/
├── 2007_000027.xml
├── 2007_000032.xml
├── 2007_000033.xml
├── 2007_000039.xml
├── 2007_000042.xml
...
└── 2012_004331.xml
```

Annotations包含一系列标注文件。对于目标检测来说，每一张图片对应一个xml格式的标注文件。

下面是其中一个**xml文件的示例**：

```xml Annotations/2007_000027.xml
<annotation>
	<folder>VOC2012</folder>  #表明图片来源
	<filename>2007_000027.jpg</filename> #图片名称
	<source>                  #图片来源相关信息
		<database>The VOC2007 Database</database>
		<annotation>PASCAL VOC2007</annotation>
		<image>flickr</image>
	</source>
	<size>     #图像尺寸
		<width>486</width>
		<height>500</height>
		<depth>3</depth>
	</size>
	<segmented>0</segmented> #是否用于分割
	<object>  #包含的物体
		<name>person</name> #物体类别
		<pose>Unspecified</pose>
		<truncated>0</truncated>
		<difficult>0</difficult>
		<bndbox>  #物体的bbox
			<xmin>174</xmin>
			<ymin>101</ymin>
			<xmax>349</xmax>
			<ymax>351</ymax>
		</bndbox>
		<part> #物体的头
			<name>head</name>
			<bndbox>
				<xmin>169</xmin>
				<ymin>104</ymin>
				<xmax>209</xmax>
				<ymax>146</ymax>
			</bndbox>
		</part>
		<part>   #物体的手
			<name>hand</name>
			<bndbox>
				<xmin>278</xmin>
				<ymin>210</ymin>
				<xmax>297</xmax>
				<ymax>233</ymax>
			</bndbox>
		</part>
		<part>
			<name>foot</name>
			<bndbox>
				<xmin>273</xmin>
				<ymin>333</ymin>
				<xmax>297</xmax>
				<ymax>354</ymax>
			</bndbox>
		</part>
		<part>
			<name>foot</name>
			<bndbox>
				<xmin>319</xmin>
				<ymin>307</ymin>
				<xmax>340</xmax>
				<ymax>326</ymax>
			</bndbox>
		</part>
	</object>
</annotation>
```

+ `bndbox` 是一个轴对齐的矩形，它框住的是目标在照片中的可见部分
+ `truncated` 表明这个目标因为各种原因没有被框完整（被截断了），比如说一辆车有一部分在画面外
+ `occluded` 是说一个目标的重要部分被遮挡了（不管是被背景的什么东西，还是被另一个待检测目标遮挡）
+ `difficult` 表明这个待检测目标很难识别，有可能是虽然视觉上很清楚，但是没有上下文的话还是很难确认它属于哪个分类；标为`difficult`的目标在测试成绩的评估中一般会被忽略。

**注意：在一个`<object />`中，`<name />` 标签要放在前面，否则的话，目标检测的一个重要工程会出现解析数据集错误**



## ImageSets 目录

```
ImageSets/
├── Action
├── Layout
├── Main
│   ├── person_train.txt
│   ├── person_trainval.txt
│   ├── person_val.txt
│   ...
│   ├── train.txt
│   ├── trainval.txt
│   └── val.txt    
└── Segmentation
```

各文件夹存放着各种用途的 TXT 文件。

我们着重介绍Main子目录下的 TXT 文件格式，该文件下的数据分为两种：

1. XXX_train.txt,  XXX_trainval.txt,  XXX_val.txt 为类别XXX的数据集划分文件

   ``` txt ImageSets/Main/person_val.txt
   2008_000002 -1
   2008_000003  1
   2008_000007 -1
   2008_000009 -1
   2008_000016 -1
   2008_000021 -1
   2008_000026  1
   2008_000027 -1
   2008_000032  1
   ...
   ```

   + 每一行有两项，
     + 第一项为数据实例的名称，如 2008_000002 对应着 Annotations/2008_000002.xml 和 JPEGImages/2008_000002.jpg
     + 第二项为1或者-1，表示正类或者负类

2. train.txt，trainval.txt，val.txt 为所有类别的数据集划分文件

   ``` txt ImageSets/Main/val.txt
   2008_000002
   2008_000003
   2008_000007
   2008_000009
   2008_000016
   2008_000021
   2008_000026
   ...
   ```

   + 每一行为数据实例的名称，如 2008_000002 对应着 Annotations/2008_000002.xml 和 JPEGImages/2008_000002.jpg



## JPEGImages 目录

该文件夹存放数据集的所有源图片。

```
JPEGImages/
├── 2007_000027.jpg
├── 2007_000032.jpg
├── 2007_000033.jpg
...
├── 2012_004330.jpg
└── 2012_004331.jpg
```

**示例 (2007_000032)：**

![2007_000032-image](pascal-voc-dataset-format/2007_000032-image.jpg)

## SegmentationClass 目录

该目录包含语义风格相关的图片

```
SegmentationClass/
├── 2007_000032.jpg
├── 2007_000033.jpg
├── 2007_000039.jpg
...
├── 2011_003256.jpg
└── 2011_003271.jpg
```

**示例 (2007_000032)：**

![2007_000032-class](pascal-voc-dataset-format/2007_000032-class.png)

## SegmentationObject 目录

该目录包含实例分割相关的图片

```
SegmentationObject/
├── 2007_000032.jpg
├── 2007_000033.jpg
├── 2007_000039.jpg
...
├── 2011_003256.jpg
└── 2011_003271.jpg
```

**示例 (2007_000032)：**

![2007_000032-object](pascal-voc-dataset-format/2007_000032-object.png)

# 参考资料

+ [PASCAL VOC 数据集详细分析](https://blog.csdn.net/u013832707/article/details/80060327)
+ [PASCAL VOC 数据集的标注格式](https://zhuanlan.zhihu.com/p/33405410)