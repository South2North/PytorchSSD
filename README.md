# Pytorch SSD Series
## Pytorch 4.1 is suppoted on branch 0.4 now.
## Support Arc:
* SSD [SSD: Single Shot Multibox  Detector](https://arxiv.org/abs/1512.02325)
* FSSD [FSSD: Feature Fusion Single Shot Multibox Detector](https://arxiv.org/abs/1712.00960)
* RFB-SSD[Receptive Field Block Net for Accurate and Fast Object Detection](https://arxiv.org/abs/1711.07767)
* RefineDet[Single-Shot Refinement Neural Network for Object Detection](https://arxiv.org/pdf/1711.06897.pdf)

### VOC2007 Test
| System                                   |  *mAP*   | **FPS** (Titan X Maxwell) |
| :--------------------------------------- | :------: | :-----------------------: |
| [Faster R-CNN (VGG16)](https://github.com/ShaoqingRen/faster_rcnn) |   73.2   |             7             |
| [YOLOv2 (Darknet-19)](http://pjreddie.com/darknet/yolo/) |   78.6   |            40             |
| [R-FCN (ResNet-101)](https://github.com/daijifeng001/R-FCN) |   80.5   |             9             |
| [SSD300* (VGG16)](https://github.com/weiliu89/caffe/tree/ssd) |   77.2   |            46             |
| [SSD512* (VGG16)](https://github.com/weiliu89/caffe/tree/ssd) |   79.8   |            19             |
| RFBNet300 (VGG16)                        | **80.5** |            83             |
| RFBNet512 (VGG16)                        | **82.2** |            38             |
| SSD300 (VGG)                             |   77.8   |     **150 (1080Ti)**      |
| FSSD300 (VGG)                            |   78.8   |       120 (1080Ti)        |

### COCO 
| System                                   | *test-dev mAP* | **Time** (Titan X Maxwell) |
| :--------------------------------------- | :------------: | :------------------------: |
| [Faster R-CNN++ (ResNet-101)](https://github.com/KaimingHe/deep-residual-networks) |      34.9      |           3.36s            |
| [YOLOv2 (Darknet-19)](http://pjreddie.com/darknet/yolo/) |      21.6      |            25ms            |
| [SSD300* (VGG16)](https://github.com/weiliu89/caffe/tree/ssd) |      25.1      |            22ms            |
| [SSD512* (VGG16)](https://github.com/weiliu89/caffe/tree/ssd) |      28.8      |            53ms            |
| [RetinaNet500 (ResNet-101-FPN)](https://arxiv.org/pdf/1708.02002.pdf) |      34.4      |            90ms            |
| RFBNet300 (VGG16)                        |    **29.9**    |         **15ms\***         |
| RFBNet512 (VGG16)                        |    **33.8**    |         **30ms\***         |
| RFBNet512-E (VGG16)                      |    **34.4**    |         **33ms\***         |
| [SSD512 (HarDNet68)](https://github.com/PingoLH/PytorchSSD-HarDNet) |      31.7      |          TBD (12.9ms\*\*)  |
| [SSD512 (HarDNet85)](https://github.com/PingoLH/PytorchSSD-HarDNet) |      35.1      |          TBD (15.9ms\*\*)  |
| RFBNet512 (HarDNet68)                    |      33.9      |          TBD (16.7ms\*\*)  |
| RFBNet512 (HarDNet85)                    |      36.8      |          TBD (19.3ms\*\*)  |

*Note*: **\*** The speed here is tested on the newest pytorch and cudnn version (0.2.0 and cudnnV6), which is obviously faster than the speed reported in the paper (using pytorch-0.1.12 and cudnnV5).

*Note*: **\*\*** HarDNet results are measured on Titan V with pytorch 1.0.1
for detection only (NMS is NOT included, which is 13~18ms in general cases).
For reference, the measurement of SSD-vgg on the same environment is 15.7ms
(also detection only).

### MobileNet
| System                                   | COCO *minival mAP* | **\#parameters** |
| :--------------------------------------- | :----------------: | :--------------: |
| [SSD MobileNet](https://arxiv.org/abs/1704.04861) |        19.3        |       6.8M       |
| RFB MobileNet                            |       20.7\*       |       7.4M       |

\*: slightly better than the original ones in the paper (20.5).

### Contents
1. [Installation](#installation)
2. [Datasets](#datasets)
3. [Training](#training)
4. [Evaluation](#evaluation)
5. [Models](#models)

## Installation
- Install [PyTorch-0.2.0-0.3.1](http://pytorch.org/) by selecting your environment on the website and running the appropriate command.
- Clone this repository. This repository is mainly based on[RFBNet](https://github.com/ruinmessi/RFBNet), [ssd.pytorch](https://github.com/amdegroot/ssd.pytorch) and [Chainer-ssd](https://github.com/Hakuyume/chainer-ssd), a huge thank to them.
  * Note: We currently only support Python 3+.
- Compile the nms and coco tools:
```Shell
./make.sh
```
Note*: Check you GPU architecture support in utils/build.py, line 131. Default is:

``` 
'nvcc': ['-arch=sm_52',
```
- Install [pyinn](https://github.com/szagoruyko/pyinn) for MobileNet backbone:
```Shell
pip install git+https://github.com/szagoruyko/pyinn.git@master
```
- Then download the dataset by following the [instructions](#download-voc2007-trainval--test) below and install opencv. 
```Shell
conda install opencv
```
Note: For training, we currently  support [VOC](http://host.robots.ox.ac.uk/pascal/VOC/) and [COCO](http://mscoco.org/). 

## Datasets
To make things easy, we provide simple VOC and COCO dataset loader that inherits `torch.utils.data.Dataset` making it fully compatible with the `torchvision.datasets` [API](http://pytorch.org/docs/torchvision/datasets.html).

### VOC Dataset
##### Download VOC2007 trainval & test

```Shell
# specify a directory for dataset to be downloaded into, else default is ~/data/
sh data/scripts/VOC2007.sh # <directory>
```

##### Download VOC2012 trainval

```Shell
# specify a directory for dataset to be downloaded into, else default is ~/data/
sh data/scripts/VOC2012.sh # <directory>
```
### COCO Dataset
Install the MS COCO dataset at /path/to/coco from [official website](http://mscoco.org/), default is ~/data/COCO. Following the [instructions](https://github.com/rbgirshick/py-faster-rcnn/blob/77b773655505599b94fd8f3f9928dbf1a9a776c7/data/README.md) to prepare *minival2014* and *valminusminival2014* annotations. All label files (.json) should be under the COCO/annotations/ folder. It should have this basic structure
```Shell
$COCO/
$COCO/cache/
$COCO/annotations/
$COCO/images/
$COCO/images/test2015/
$COCO/images/train2014/
$COCO/images/val2014/
```
*UPDATE*: The current COCO dataset has released new *train2017* and *val2017* sets which are just new splits of the same image sets. 

## Training
- First download the fc-reduced [VGG-16](https://arxiv.org/abs/1409.1556) PyTorch base network weights at:    https://s3.amazonaws.com/amdegroot-models/vgg16_reducedfc.pth
  or from our [BaiduYun Driver](https://pan.baidu.com/s/1jIP86jW) 
- MobileNet pre-trained basenet is ported from [MobileNet-Caffe](https://github.com/shicai/MobileNet-Caffe), which achieves slightly better accuracy rates than the original one reported in the [paper](https://arxiv.org/abs/1704.04861), weight file is available at: https://drive.google.com/open?id=13aZSApybBDjzfGIdqN1INBlPsddxCK14 or [BaiduYun Driver](https://pan.baidu.com/s/1dFKZhdv).

- By default, we assume you have downloaded the file in the `RFBNet/weights` dir:
```Shell
mkdir weights
cd weights
wget https://s3.amazonaws.com/amdegroot-models/vgg16_reducedfc.pth
```

- To train RFBNet using the train script simply specify the parameters listed in `train_RFB.py` as a flag or manually change them.
```Shell
python train_test.py -d VOC -v RFB_vgg -s 300 
```
- Note:
  * -d: choose datasets, VOC or COCO.
  * -v: choose backbone version, RFB_VGG, RFB_E_VGG or RFB_mobile.
  * -s: image size, 300 or 512.
  * You can pick-up training from a checkpoint by specifying the path as one of the training parameters (again, see `train_RFB.py` for options)

## Evaluation
The test frequency can be found in the train_test.py
By default, it will directly output the mAP results on VOC2007 *test* or COCO *minival2014*. For VOC2012 *test* and COCO *test-dev* results, you can manually change the datasets in the `test_RFB.py` file, then save the detection results and submitted to the server. 

## Models
* ImageNet [mobilenet](https://drive.google.com/open?id=11VqerLerDkFzN_fkwXG4Vm1CIU2G5Gtm)
* 07+12 [RFB_Net300](https://drive.google.com/open?id=1V3DjLw1ob89G8XOuUn7Jmg_o-8k_WM3L), [BaiduYun Driver](https://pan.baidu.com/s/1bplRosf),[FSSD300](https://drive.google.com/open?id=1xhgdxCF_HuC3SP6ALhhTeC5RTmuoLzgC),[SSD300](https://drive.google.com/open?id=10sM_yWSN8vRZdh6Sf0CILyMfcoJiCNtn)
* COCO [RFB_Net512_E](https://drive.google.com/open?id=1pHDc6Xg9im3affOr7xaimXaRNOHtbaPM), [BaiduYun Driver](https://pan.baidu.com/s/1o8dxrom)
* COCO [RFB_Mobile Net300](https://drive.google.com/open?id=1vmbTWWgeMN_qKVWOeDfl1EN9c7yHPmOe), [BaiduYun Driver](https://pan.baidu.com/s/1bp4ik1L)

## Update (Sep 29, 2019)
* Add SSD and RFBNet with [Harmonic DenseNet (HarDNet)](https://github.com/PingoLH/Pytorch-HarDNet) as backbone models.
* Pretrained backbone models: 
[hardnet68_base_bridge.pth](https://ping-chao.com/hardnet/hardnet68_base_bridge.pth) | 
[hardnet85_base.pth](https://ping-chao.com/hardnet/hardnet85_base.pth) 
* Pretrained models for COCO dataset:
[SSD512-HarDNet68](https://ping-chao.com/hardnet/SSD512_HarDNet68_COCO.pth) | 
[SSD512-HarDNet85](https://ping-chao.com/hardnet/SSD512_HarDNet85_COCO.pth) | 
[RFBNet512-HarDNet68](https://ping-chao.com/hardnet/RFB512_HarDNet68_COCO.pth) | 
[RFBNet512-HarDNet85](https://ping-chao.com/hardnet/RFB512_HarDNet85_COCO.pth)


