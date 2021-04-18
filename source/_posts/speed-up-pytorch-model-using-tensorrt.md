---
title: "使用TensorRT加速PyTorch模型"
tags: ["tensorrt", "pytorch"]
categories: ["machine-learning"]
reward: true
copyright: true
date: 2021-04-16 13:55:20
thumbnail:
---



本文将介绍使用TensorRT推理引擎对PyTorch模型进行加速，便于算法的部署。

<!--more-->



# PyTorch模型转换成TensorRT引擎的路径

目前，PyTorch模型转TensorRT需要借助ONNX作为中间的交换格式，先将PyTorch模型转换成ONNX模型，再将ONNX模型转换成TensorRT引擎。

# 环境准备

系统：Ubuntu18.04

1. 从[TensorRT官网](https://developer.nvidia.com/zh-cn/tensorrt)下载对应版本的TensorRT安装包。本例中下载到的是`TensorRT-7.2.3.4.Ubuntu-18.04.x86_64-gnu.cuda-11.1.cudnn8.1.tar.gz`。

2. 将压缩包解压到`/opt`目录下。

   ```shell
   tar xzvf TensorRT-7.2.3.4.Ubuntu-18.04.x86_64-gnu.cuda-11.1.cudnn8.1.tar.gz -C /opt
   ```

3. 设置环境变量

   将以下两条放入`~/.bashrc`中。
   
   ```shell
   export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/TensorRT-7.2.3.4/lib:/usr/local/cuda/lib64
   export PATH=$PATH:/opt/TensorRT-7.2.3.4/bin
   ```
   
   将其生效。
   
   ```shell
   source ~/.bashrc
   ```

4. 安装TensorRT的python包及其他辅助python包

   ```shell
   pip install /opt/TensorRT-7.2.3.4/python/tensorrt-7.2.3.4-cp38-none-linux_x86_64.whl
   pip install /opt/TensorRT-7.2.3.4/uff/uff-0.6.9-py2.py3-none-any.whl
   pip install /opt/TensorRT-7.2.3.4/graphsurgeon/graphsurgeon-0.4.5-py2.py3-none-any.whl
   pip install /opt/TensorRT-7.2.3.4/onnx_graphsurgeon/onnx_graphsurgeon-0.2.6-py2.py3-none-any.whl
   ```

5. 安装其他必要的库

   ```shell
   pip install -i https://mirrors.aliyun.com/pypi/simple pycuda onnx-simplifier
   
   sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list \
   && apt-get update -y \
   && apt-get install -y -f libglib2.0-0 libsm6 libxrender1 libxext6 libgl1-mesa-glx \
   && pip install -i https://mirrors.aliyun.com/pypi/simple opencv-python
   ```

   

# PyTorch转换成ONNX

ONNX格式是本身是由Facebook公司参与开发的，受到PyTorch的原生支持，可用`torch.onnx`模块进行ONNX的导出。

## 准备待转换的PyTorch模型

首先构建好模型并加载上权重，还可以将标准化、阈值化等预处理、后处理步骤也加入到模型中，将尽可能多的步骤放到ONNX中，后续可以一并转换成TensorRT引擎，避免因预处理、后处理步骤放在了CPU上执行或使用PyTorch实现而造成的数据在GPU和CPU上频繁传输导致算法速度降低。

```python
import torch
import torch.onnx
import torch.nn as nn
import ...


class OnnxModel(nn.Module):
    def __init__(self, cfg):
        super().__init__()
        self.model = build_model(cfg)
        self.model.load_state_dict(torch.load(cfg.MODEL.WEIGHTS))
        self.num_classes = 2
        assert self.num_classes == 2, "the OnnxModel implementation is only for num_classes == 2"

    def forward(self, images, images_wh, pixel_mean_std, score_thresholds):
        images = (images - pixel_mean_std[:, :1].view(-1, 1, 1, 1)) / pixel_mean_std[:, 1:].view(-1, 1, 1, 1)
        bboxes, classification = self.model(images)

        batch_size, num_boxes = [int(i) for i in bboxes.shape[:2]]
        h, w = [int(i) for i in images.shape[-2:]]
        standard_wh = torch.from_numpy(np.array([w, h]).reshape(1, 2)).cuda()

        wh_scale = images_wh.max(1, keepdims=True)[0] / (images_wh * standard_wh)

        bboxes *= wh_scale[:, None, :].repeat(1, 1, 2)
        bboxes = torch.clamp(bboxes, min=0, max=1)

        classification_class0 = classification[:, :, 0]
        classification_class1 = classification[:, :, 1]

        is_valid_boxes = (bboxes[:, :, 0] < bboxes[:, :, 2]) & (bboxes[:, :, 1] < bboxes[:, :, 3])

        is_class0_over_threshold = classification_class0 > score_thresholds[0]
        is_class1_over_threshold = classification_class1 > score_thresholds[1]
        is_0_classes = classification_class0 > classification_class1

        is_class0_score_save = is_class0_over_threshold & is_0_classes & is_valid_boxes
        is_class1_score_save = is_class1_over_threshold & ~is_0_classes & is_valid_boxes


        index_score_save = torch.cat([is_class0_score_save.float()[:, :, None], is_class1_score_save.float()[:, :, None]], dim=2)

        classification *= index_score_save

        images_wh_helper = images_wh[:, None, :].repeat(1, 1, 2)

        return bboxes, classification, images_wh_helper
```

受TensorRT的限制，某些操作不能在PyTorch中使用，需要用其他的形式进行代替：

1. 不能够对Boolean类型张量进行索引或切片。TensorRT不允许对Boolean类型的张量进行索引或切片，因此一个替代的方案是在需要进行索引或切片前，先将Boolean类型张量转换成int类型张量或float类型张量，再进行索引或切片。

```python
temp = bboxes[:, :, 0:2] < bboxes[:, :, 2:4]
is_valid_boxes = temp[:, :, 0] & temp[:, :, 1]  # 该操作对布尔类型的张量进行了索引，可以正常转换成ONNX，但ONNX转换成TensorRT将会失败，因尽可能避免

# 可改成
is_valid_boxes = (bboxes[:, :, 0] < bboxes[:, :, 2]) & (bboxes[:, :, 1] < bboxes[:, :, 3])
```

2. 不能够给张量的切片赋值。

```python
bboxes[:, :, 0::2] /= w  # 此操作单独改变了切片中的值，无法转换成ONNX
bboxes[:, :, 1::2] /= h

# 可改成
bboxes /= torch.cat([w, h]).repeat(1, 1, 2)
```

## 转换PyTorch到ONNX

主要使用`torch.onnx.export`接口将PyTorch模型转换成ONNX。

将待转换的模型准备好，放到gpu设备上，并准备好一个输入的样本用于运行生成静态图。

```python
device = "cuda:0"

model = OnnxModel(cfg)
model.to(device)

model.eval()

batch_size = 2

image_shape = (batch_size, cfg.MODEL.INPUT.IN_CHANNELS, *cfg.MODEL.INPUT.SIZE)

images = torch.randn(*image_shape).to(device) * 255
pixel_mean_std = torch.Tensor(batch_size * [0.0, 1.0]).view(batch_size, 2).to(device)
score_thresholds = torch.Tensor([0.2, 0.2]).to(device)
images_wh = torch.Tensor(batch_size * [640, 480]).view(batch_size, 2).to(device)

model(images, images_wh, pixel_mean_std, score_thresholds)

torch.onnx.export(
    model,
    (images, images_wh, pixel_mean_std, score_thresholds),
    out_onnx_path,
    export_params=True,
    input_names=['images', 'images_width_height', 'pixel_mean_std', 'score_thresholds'],
    output_names=['bboxes', 'classification', 'images_width_height_helper'],
    do_constant_folding=True,
    verbose=True,
    opset_version=12,
    dynamic_axes={
        "images": {
            0: 'batch_size',
        },
        "images_width_height": {
            0: 'batch_size',
        },
        "pixel_mean_std": {
            0: 'batch_size',
        },
        "bboxes": {
            0: 'batch_size',
        },
        "classification": {
            0: 'batch_size',
        },
        "images_width_height_helper": {
            0: 'batch_size',
        }
    },
)
```

## 简化静态图

由于PyTorch底层实现上产生的ONNX静态图中还会存在着很多可以合并优化的步骤，其中有些步骤会在ONNX转换成TensorRT的步骤中出错，因此需要将得到的ONNX模型进行简化。简化的工具可使用python包`onnx-simplifier`。

```python
import onnxsim
model_opt, check_ok = onnxsim.simplify(
    out_onnx_path,
    dynamic_input_shape=True,
    input_shapes={
        'images': image_shape,
        'images_width_height': (batch_size, 2),
        'pixel_mean_std': (batch_size, 2),
    }
)
if check_ok:
    onnx.save(model_opt, out_onnx_path)
else:
    raise Exception("Check failed")
```

注意到此时我们生成的静态图仍然是动态batchsize的。

# ONNX转换成TensorRT

上一步中我们得到的onnx模型是动态大小的，直接转换成TensorRT模型需要使用TensorRT的dynamic shape特性，但是该特性下会有很多问题，部分模块会无法使用，因此先介绍如何转换成固定张量维度的TensorRT引擎。

## 添加非极大值抑制插件

由于非极大值抑制操作无法直接从PyTorch转换到ONNX，因此需要手动添加非极大值抑制，可以通过TensorRT提供的插件机制，实现无法转换的部分。由于TensorRT官方提供了[各种插件](https://github.com/NVIDIA/TensorRT/tree/master/plugin)，其中包括了非极大值抑制，因此可以很方便地使用非极大值抑制插件而不必自己编写。

```python
def add_nms(onnx_model, nms_iou_threshold, nms_top_k, nms_keep_top_k):
    import onnx_graphsurgeon as gs
    graph = gs.import_onnx(onnx_model)

    bboxes_tensor, classification_tensor, images_width_height_helper_tensor = graph.outputs

    batch_size = bboxes_tensor.shape[0]

    # add reshape
    bboxes_new_shape = gs.Constant(
        "bboxes_new_shape",
        values=np.array([bboxes_tensor.shape[0], bboxes_tensor.shape[1], 1, bboxes_tensor.shape[2]], dtype=np.int64),
    )
    bboxes_new_tensor = gs.Variable(
        "bboxes_new",
        shape=tuple(int(i) for i in bboxes_new_shape.values),
        dtype=bboxes_tensor.dtype,
    )
    bboxes_reshape_node = gs.Node(
        op="Reshape",
        inputs=[bboxes_tensor, bboxes_new_shape],
        outputs=[bboxes_new_tensor],
    )
    graph.nodes.append(bboxes_reshape_node)

    # add nms
    nms_attrs = {
        "shareLocation": True,
        "backgroundLabelId": -1,
        "numClasses": 2,
        "topK": nms_top_k,
        "keepTopK": nms_keep_top_k,
        "scoreThreshold": 0.01,
        "iouThreshold": nms_iou_threshold,
        "isNormalized": True,
        "clipBoxes": True,
        "plugin_version": "1",
        "plugin_namespace": "",
    }
    num_detections_tensor = gs.Variable(
        "num_detections",
        shape=(batch_size, 1),
        dtype=np.int32,
    )
    nmsed_boxes_norm_tensor = gs.Variable(
        "nmsed_boxes_norm",
        shape=(batch_size, nms_attrs['keepTopK'], 4),
        dtype=np.float32,
    )
    nmsed_scores_tensor = gs.Variable(
        "nmsed_scores",
        shape=(batch_size, nms_attrs['keepTopK']),
        dtype=np.float32,
    )
    nmsed_classes_tensor = gs.Variable(
        "nmsed_classes",
        shape=(batch_size, nms_attrs['keepTopK']),
        dtype=np.float32,
    )
    nms_node = gs.Node(
        op="BatchedNMS_TRT",
        name='NMS',
        attrs=nms_attrs,
        inputs=[bboxes_new_tensor, classification_tensor],
        outputs=[num_detections_tensor, nmsed_boxes_norm_tensor, nmsed_scores_tensor, nmsed_classes_tensor],
    )
    graph.nodes.append(nms_node)

    #rescale box xyxy from (1, 1) to (w, h)
    nmsed_boxes_tensor = gs.Variable(
        "nmsed_boxes",
        shape=(batch_size, nms_attrs['keepTopK'], 4),
        dtype=np.float32,
    )
    scale_node = gs.Node(
        op="Mul",
        name="scale_width_height",
        inputs=[nmsed_boxes_norm_tensor, images_width_height_helper_tensor],
        outputs=[nmsed_boxes_tensor],
    )
    graph.nodes.append(scale_node)

    #nmsed_boxes_tensor = nmsed_boxes_norm_tensor

    # redefine output
    graph.outputs = [num_detections_tensor, nmsed_boxes_tensor, nmsed_scores_tensor, nmsed_classes_tensor]


    graph.cleanup().toposort()
    onnx_model = gs.export_onnx(graph)
    return onnx_model
```



## 将ONNX从动态形状转换成固定形状

```python
import onnxsim
model_opt, check_ok = onnxsim.simplify(
    base_onnx_file_path,
    dynamic_input_shape=False,
    input_shapes={
        'images': image_shape,
        'images_width_height': (image_shape[0], 2),
        'pixel_mean_std': (image_shape[0], 2),
    }
)
if check_ok:
    print("success batchlize")
    model_opt = add_nms(model_opt, nms_iou_threshold, nms_top_k, nms_keep_top_k)
    print("success add nms")
    onnx.save(model_opt, out_onnx_file_path)
else:
    raise Exception("Check failed")
```



## 转换ONNX到TensorRT

```python
TRT_LOGGER = trt.Logger()
trt.init_libnvinfer_plugins(TRT_LOGGER, "")
explicit_batch = 1 << (int)(trt.NetworkDefinitionCreationFlag.EXPLICIT_BATCH)


def build_engine(max_batch_size, onnx_file_path, engine_file_path, fp16_mode=False):
    """ Take an ONNX file and creates a TensorRT engine to run inference with"""
    with trt.Builder(TRT_LOGGER) as builder, \
            builder.create_network(explicit_batch) as network, \
            trt.OnnxParser(network, TRT_LOGGER) as parser:

        builder.max_workspace_size = 1 << 30  # your workspace size 1GiB
        builder.max_batch_size = max_batch_size
        builder.fp16_mode = fp16_mode
        builder.int8_mode = False

        # Parse model file
        if not os.path.exists(onnx_file_path):
            quit("ONNX file {} not found".format(onnx_file_path))

        print("Loading ONNX file from path {} ...".format(onnx_file_path))
        with open(onnx_file_path, 'rb') as model:
            print("Beginning ONNX file parsing")
            parser.parse(model.read())

        print("Completed parsing of ONNX file")
        print("Building an engine from file {}; this may take a while...".format(onnx_file_path))

        engine = builder.build_cuda_engine(network)
        print("Completed creating Engine")

        with open(engine_file_path, 'wb') as f:
            f.write(engine.serialize())


def generate_engine_from_onnx(batch_size, nms_iou_threshold, nms_top_k, nms_keep_top_k):
    cfg = get_cfg('config.yaml')
    base_onnx_file_path = cfg.MODEL.BASE_ONNX_WEIGHTS(cfg)
    if not os.path.exists(base_onnx_file_path):
        raise FileNotFoundError("please generate base onnx file firstly.")

    cfg.MODEL.BATCH_SIZE = batch_size
    image_shape = (batch_size, cfg.MODEL.INPUT.IN_CHANNELS, *cfg.MODEL.INPUT.SIZE)

    if nms_iou_threshold is None:
        nms_iou_threshold = cfg.MODEL.NMS_THRESHOLD
    else:
        cfg.MODEL.NMS_THRESHOLD = nms_iou_threshold
    out_onnx_file_path = cfg.MODEL.ONNX_WEIGHTS(cfg)
    out_engine_file_path = cfg.MODEL.TENSORRT_WEIGHTS(cfg)

    if not os.path.exists(out_onnx_file_path):
        from .modify_onnx import build_onnx_file
        build_onnx_file(base_onnx_file_path, out_onnx_file_path, image_shape, nms_iou_threshold, nms_top_k, nms_keep_top_k)

    if not os.path.exists(out_engine_file_path):
        build_engine(batch_size, out_onnx_file_path, out_engine_file_path, fp16_mode=False)
```



# 使用TensorRT引擎进行推理

```python
import pycuda.driver as cuda
import tensorrt as trt
import numpy as np


TRT_LOGGER = trt.Logger()
trt.init_libnvinfer_plugins(TRT_LOGGER, "")
explicit_batch = 1 << (int)(trt.NetworkDefinitionCreationFlag.EXPLICIT_BATCH)


class HostDeviceMem(object):
    def __init__(self, host_mem, device_men):
        """ Within this context, host_men meas the cpu memory, device_mem means the gpu memory"""
        self.host = host_mem
        self.device = device_men

    def __str__(self):
        return "Host:\n{}\nDevice:{}\n".format(self.host, self.device)

    def __repr__(self):
        return self.__str__()


def allocate_buffers(engine):
    inputs = []
    outputs = []
    bindings = []
    stream = cuda.Stream()
    for binding in engine:
        size = trt.volume(engine.get_binding_shape(binding))
        dtype = trt.nptype(engine.get_binding_dtype(binding))
        # Allocate host and device buffers
        host_mem = cuda.pagelocked_empty(size, dtype)
        device_mem = cuda.mem_alloc(host_mem.nbytes)
        # Append the device buffer to device bindings
        bindings.append(int(device_mem))
        if engine.binding_is_input(binding):
            inputs.append(HostDeviceMem(host_mem, device_mem))
        else:
            outputs.append(HostDeviceMem(host_mem, device_mem))
    return inputs, outputs, bindings, stream


def get_engine(engine_file_path):
    with open(engine_file_path, 'rb') as f, trt.Runtime(TRT_LOGGER) as runtime:
        return runtime.deserialize_cuda_engine(f.read())


def do_inference(context, bindings, inputs, outputs, stream, batch_size=1):
    # Transfer data from CPU to the GPU.
    [cuda.memcpy_htod_async(input.device, input.host, stream) for input in inputs]
    # Run inference
    context.execute_async(batch_size=batch_size, bindings=bindings, stream_handle=stream.handle)
    # Transfer predictions back from the GPU.
    [cuda.memcpy_dtoh_async(output.host, output.device, stream) for output in outputs]
    # Synchronize the stream
    stream.synchronize()
    # Return only the host outputs.
    return [output.host for output in outputs]


class Predictor:
    def __init__(self, cfg):
        self.cfg = cfg.copy()
        self.input_size = cfg.MODEL.INPUT.SIZE

        self.cuda_device = cuda.Device(cfg.MODEL.DEVICE)
        self.cuda_context = self.cuda_device.make_context()

        self.engine = get_engine(cfg.MODEL.TENSORRT_WEIGHTS(cfg))
        self.context = self.engine.create_execution_context()

        self.inputs, self.outputs, self.bindings, self.stream = allocate_buffers(self.engine)

        self.max_batch_size = cfg.MODEL.BATCH_SIZE

        self.thresholds = np.array([cfg.MODEL.PEOPLE_THRESHOLD, cfg.MODEL.CAR_THRESHOLD], dtype=np.float32)
        self.nms_threshold = cfg.MODEL.NMS_THRESHOLD

    def __del__(self):
        self.cuda_context.pop()
        del self.cuda_context

    def __call__(self, images, pixel_mean_std_list):
        self.cuda_context.push()
        batch_size = len(images)
        if batch_size > self.max_batch_size:
            raise Exception("the max_batch_size is {}".format(self.max_batch_size))
        
        images_width_height = np.array([[image.shape[1], image.shape[0]] for image in images], dtype=np.float32)
        pixel_mean_std = np.stack(pixel_mean_std_list, axis=0)

        images, resize_params_list = zip(*[resize_padding(images[i], size=self.input_size, fill_pixel=pixel_mean_std[i, 0]) for i in range(len(images))])
        images = [np.expand_dims(image, -1) if len(image.shape) == 2 else image for image in images]

        images = np.stack(images, axis=0).transpose(0, 3, 1, 2)

        self.inputs[0].host = images.reshape(-1).astype('float32')
        self.inputs[1].host = images_width_height
        self.inputs[2].host = pixel_mean_std
        self.inputs[3].host = self.thresholds

        trt_outputs = do_inference(self.context, self.bindings, self.inputs, self.outputs, self.stream)

        num_detections = trt_outputs[0][:batch_size]
        nmsed_boxes = trt_outputs[3].reshape(self.engine.max_batch_size, -1, 4)[:batch_size]
        nmsed_scores = trt_outputs[1].reshape(self.engine.max_batch_size, -1)[:batch_size]
        nmsed_classes = trt_outputs[2].reshape(self.engine.max_batch_size, -1)[:batch_size]

        prediction_list = [
            {
                'rois': nmsed_boxes[i, :num_detections[i]],
                'class_ids': nmsed_classes[i, :num_detections[i]],
                'scores': nmsed_scores[i, :num_detections[i]],
            }
            for i in range(batch_size)
        ]

        self.cuda_context.pop()

        return prediction_list
```

```python
from pycuda.driver as cuda

cuda.init()

cfg = ...
predictor = Predictor(cfg)

images = ...
pixel_mean_std_list = ...
prediction_list = predictor(images, pixel_mean_std_list)
```

注意：

1. 推理的输出`trt_output`不一定会按照PyTorch输出时的顺序或修改后ONNX模型中的输出顺序，需要先确定根据输出输出大小、类型、数值确定。
2. TensorRT引擎的输入和输出都是一维向量，输入前需要展开成一维向量，输出后将一维向量reshape成预定的大小。
3. 在进行推理前，对于每一个使用TensorRT的进程，需要先导入`pycuda.driver`，并使用`cuda.init()`进行初始化。
4. 使用TensorRT进行一次推理前需要先`cuda.context.push()`，并在推理完成后`cuda.context.pop()`。

