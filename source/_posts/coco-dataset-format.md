---
title: COCO数据集格式
tags: ["dataset"]
categories: ["machine-learning"]
reward: true
copyright: true
date: 2020-01-07 13:33:56
thumbnail: coco-dataset-format/coco-logo.png
---

本文介绍COCO数据集的标注格式。

COCO 全称为Common Objects in Context，是微软团队提供的一个可以用来进行图像识别的数据集。

COCO数据集目前有3种标注类型：

+ object instances (目标实例)
+ object keypoints (目标上的关键点)
+ image captions (看图说话)

COCO 数据集的信息采用JSON文件存储。

<!--more-->



# 基本的JSON结构体类型

object instances (目标实例)，object keypoints (目标上的关键点),，image captions (看图说话) 这3种类型共享这些基本类型：`info`、`image`、`license`。而 `annotation` 类型则呈现出多态。

```json
{
    "info": info,
    "licenses": [license],
    "images": [image],
    "annotations": [annotation],
}
    
info{
    "year": int,
    "version": str,
    "description": str,
    "contributor": str,
    "url": str,
    "date_created": datetime,
}
license{
    "id": int,
    "name": str,
    "url": str,
} 
image{
    "id": int,
    "width": int,
    "height": int,
    "file_name": str,
    "license": int,
    "flickr_url": str,
    "coco_url": str,
    "date_captured": datetime,
}
```

1. `info` 类型，比如一个 `info` 类型的实例：

   ```json
   {
   	"description":"This is stable 1.0 version of the 2014 MS COCO dataset.",
   	"url":"http:\/\/mscoco.org",
   	"version":"1.0","year":2014,
   	"contributor":"Microsoft COCO group",
   	"date_created":"2015-01-27 09:11:52.357475"
   }
   ```

2. `images` 是包含多个 `image` 实例的数组，对于一个 `image` 类型的实例

   ```json
   {
   	"license":3,
   	"file_name":"COCO_val2014_000000391895.jpg",
   	"coco_url":"http:\/\/mscoco.org\/images\/391895",
   	"height":360,"width":640,"date_captured":"2013-11-14 11:18:45",
   	"flickr_url":"http:\/\/farm9.staticflickr.com\/8186\/8119368305_4e622c8349_z.jpg",
   	"id":391895
   }
   ```

3. `licenses` 是包含多个 `license` 实例的数组，对于一个 `license` 类型的实例

   ```json
   {
   	"url":"http:\/\/creativecommons.org\/licenses\/by-nc-sa\/2.0\/",
   	"id":1,
   	"name":"Attribution-NonCommercial-ShareAlike License"
   }
   ```

   

# Object Instance 类型的标注格式

## 整体JSON文件格式

```json
{
    "info": info,
    "licenses": [license],
    "images": [image],
    "annotations": [annotation],
    "categories": [category]
}
```

`info`, `licenses`, `image` 这三个结构体在不同的标注类型是统一的，只有 `annotation` 和 `category` 这两个结构体在不同类型的JSON文件中是不同的。

+ `images` 数组元素的数量等同于划入训练集（或者测试集）的图片的数量
+ `annotations` 数组元素的数量等同于训练集（或者测试集）中bounding box的数量
+ `categories` 数组元素的数量为分类类别的数量



## `annotations` 字段

`annotations` 字段是包含多个 `annotation` 实例的一个数组，`annotation` 类型本身又包含了一系列的字段，如这个目标的 `category id` 和 `segmentation mask` 。`segmentation` 格式取决于这个实例是一个单个的对象（即 `iscrowd=0`，将使用polygons格式）还是一组对象（即 `iscrowd=1`，将使用RLE格式）。如下所示：

```json
annotation{
    "id": int,    
    "image_id": int,
    "category_id": int,
    "segmentation": RLE or [polygon],
    "area": float,
    "bbox": [x,y,width,height],
    "iscrowd": 0 or 1,
}
```

+ `iscrowd=0` (单个对象) 时, `segmentation` 就是 `polygon` 格式，单个的对象可能需要多个`polygon`来表示，比如这个对象在图像中被挡住了。而`iscrowd=1`（将标注一组对象，比如一群人）时，`segmentation`使用的就是RLE格式。
+ `area` 是area of encoded masks，是标注区域的面积。如果是矩形框，那就是高乘宽；如果是`polygon`或者`RLE`，那就复杂点。
+ `categories` 字段存储的是当前对象所属的category的id，以及所属的supercategory的name。

**`polygon` 格式 `annotation` 示例：**

```json
{
	"segmentation": [[510.66,423.01,511.72,420.03,510.45......]],
	"area": 702.1057499999998,
	"iscrowd": 0,
	"image_id": 289343,
	"bbox": [473.07,395.93,38.65,28.67],
	"category_id": 18,
	"id": 1768
}
```

+ `polygon` 格式比较简单，这些数按照相邻的顺序两两组成一个点的xy坐标，如果有n个数（必定是偶数），那么就是n/2个点坐标。

**`RLE` 格式 `annotation` 示例：**

```json
"segmentation" : 
{
    u'counts': [272, 2, 4, 4, 4, 4, 2, 9, 1, 2, 16, 43, 143, 24......], 
    u'size': [240, 320]
}
```

+ `RLE` (Run-length encoding)格式：对于一张图240x320的图片，共有76800个像素点，根据每个像素点在不在目标区域中，共有76800个bit；对于00000111100111110...，记录连续0和1的个数得到 [5,4,2,5,...]，这就是上文的 `counts` 数组。
+ `size` 为这个图片的宽和高



## `categories` 字段

`categories` 是一个包含多个 `category` 实例的数组，而 `category` 结构体描述如下：

```json
category{
    "id": int,
    "name": str,
    "supercategory": str,
}
```

**示例：**

```json
{
	"supercategory": "person",
	"id": 1,
	"name": "person"
},
{
	"supercategory": "vehicle",
	"id": 2,
	"name": "bicycle"
},
```



# Object Keypoint 类型的标注格式

## 整体JSON文件格式

```json
{
    "info": info,
    "licenses": [license],
    "images": [image],
    "annotations": [annotation],
    "categories": [category]
}
```

+ `images` 数组元素数量是划入训练集（测试集）的图片的数量
+ `annotations` 是bounding box的数量，在这里只有人这个类别的bounding box
+ `categories` 数组元素的数量为1，只有一个：person（2017年）

## `annotations` 字段

这个类型中的 `annotation` 结构体包含了Object Instance中 `annotation` 结构体的所有字段，再加上2个额外的字段：`keypoints` 和 `num_keypoints`。

```json
annotation{
    "keypoints": [x1,y1,v1,...],
    "num_keypoints": int,
    "id": int,
    "image_id": int,
    "category_id": int,
    "segmentation": RLE or [polygon],
    "area": float,
    "bbox": [x,y,width,height],
    "iscrowd": 0 or 1,
}
```

+ `keypoints` 是一个长度为3*k的数组，其中k是 `category` 中 `keypoints` 的总数量。每一个 `keypoint` 是一个长度为3的数组，第一和第二个元素分别是x和y坐标值，第三个元素是个标志位v，v为0时表示这个关键点没有标注（这种情况下x=y=v=0），v为1时表示这个关键点标注了但是不可见（被遮挡了），v为2时表示这个关键点标注了同时也可见
+ `num_keypoints` 表示这个目标上被标注的关键点的数量（v>0），比较小的目标上可能就无法标注关键点

**示例：**

```json
{
	"segmentation": [[125.12,539.69,140.94,522.43...]],
	"num_keypoints": 10,
	"area": 47803.27955,
	"iscrowd": 0,
	"keypoints": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,142,309,1,177,320,2,191,398...],
	"image_id": 425226,"bbox": [73.35,206.02,300.58,372.5],"category_id": 1,
	"id": 183126
}
```



## `categories` 字段

对于每一个 `category` 结构体，相比Object Instance中的 `category` 新增了2个额外的字段。

```json
category{
    "id": int,
    "name": str,
    "supercategory": str,
    "keypoints": [str],
    "skeleton": [edge]
}
```

+ `keypoints` 是一个长度为k的数组，包含了每个关键点的名字
+ `skeleton` 定义了各个关键点之间的连接性（比如人的左手腕和左肘就是连接的，但是左手腕和右手腕就不是）。目前，COCO的keypoints只标注了person category （分类为人）。

**示例：**

```json
{
	"supercategory": "person",
	"id": 1,
	"name": "person",
	"keypoints": ["nose","left_eye","right_eye","left_ear","right_ear","left_shoulder","right_shoulder","left_elbow","right_elbow","left_wrist","right_wrist","left_hip","right_hip","left_knee","right_knee","left_ankle","right_ankle"],
	"skeleton": [[16,14],[14,12],[17,15],[15,13],[12,13],[6,12],[7,13],[6,7],[6,8],[7,9],[8,10],[9,11],[2,3],[1,2],[1,3],[2,4],[3,5],[4,6],[5,7]]
}
```



# Image Caption 类型的标注格式

## 整体JSON文件格式

```json
{
    "info": info,
    "licenses": [license],
    "images": [image],
    "annotations": [annotation]
}
```

+ `images` 数组的元素数量等于划入训练集（或者测试集）的图片的数量
+ `annotations` 的数量要多于图片的数量，这是因为一个图片可以有多个场景描述；

## `annotations` 字段

```json
annotation{
    "id": int,
    "image_id": int,
    "caption": str
}
```

**示例：**

```json
{
	"image_id": 179765,
	"id": 38,
	"caption": "A black Honda motorcycle parked in front of a garage."
}
```

# 参考资料

+ [COCO数据集的标注格式](https://zhuanlan.zhihu.com/p/29393415)

