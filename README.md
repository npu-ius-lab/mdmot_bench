# mdmot_bench

本项目面向**分布式动态观测机条件下的被动定位与多目标跟踪研究**，提供一套可复现实验流程的**数据集规范**与**评估工具**实现，旨在支持多视角、多目标、动态平台场景下的数据关联、定位与跟踪算法评测。

项目主要包含两部分内容：
- 动态观测机二对三和三对五的被动观测数据集
- 多目标跟踪评估工具（基于 TrackEval 的定制改版）

---

## 一、数据集

### 1. 数据集概述

本数据集针对**动态观测机二对三（2 Observers → 3 Targets）**的被动观测场景构建，模拟或采集观测平台持续运动、多目标同时存在条件下的视觉观测数据，适用于以下研究方向：

- 被动定位（Angle-only Localization）
- 多目标数据关联
- 多目标跟踪（MOT）
- 多平台协同感知与状态估计

数据以**在线观测序列（streaming / time-sequential）**形式组织，强调时间连续性与跨平台一致性。

---

### 2. 场景配置

- 观测机数量：2  
- 目标数量：3  
- 观测方式：被动视觉观测（角度信息）  
- 观测机状态：动态运动  
- 目标状态：动态运动（可包含机动）

---

### 3. 数据内容

每个数据序列原则上包含以下信息：

#### （1）图像数据
- 来自两架观测机的同步或近同步图像序列
- 支持 RGB 或灰度格式

#### （2）目标检测 / 观测结果
- 每帧图像中目标的二维观测信息（如检测框或像素点）
- 支持多目标同时出现与遮挡情况

#### （3）观测机位姿信息
- 观测机在世界坐标系下的位置与姿态
- 可来自仿真、动捕系统或高精度定位系统

---

### 4. ROS 话题结构（示例，占位）

> **说明：以下话题名称仅作为示例，具体名称可根据实际系统配置填写**

- 图像话题  
  - `/obs_01/image_raw`  
  - `/obs_02/image_raw`

- 目标检测 / 观测话题  
  - `/obs_01/detections`  
  - `/obs_02/detections`

- 观测机位姿话题  
  - `/obs_01/pose`  
  - `/obs_02/pose`

---

### 5. 相机参数（占位）

#### （1）相机内参（Camera Intrinsics）
```text
fx:
fy:
cx:
cy:
```

#### （2）相机外参（Camera Extrinsics）

```
T_body_camera:
R_body_camera:
```

> 参数格式与单位需与具体实验系统保持一致，建议采用 ROS 或 OpenCV 约定。



## 二、评估工具（Evaluation Tool）

### 2.1 工具概述

本项目采用 **TrackEval** 的改版实现，用于多目标跟踪结果的统一评估，并针对**三维点匹配（3D Point Matching）**场景进行了定制扩展，适用于被动定位与多观测机协同感知任务。

评估框架支持多种主流多目标跟踪评价指标（包括 HOTA 系列指标），并提供完整的评估流程、结果统计与可视化功能。同时，通过参数配置可同时兼容 **三维点级匹配** 与 **二维目标框匹配** 两类评估模式。

---

### 2.2 数据格式说明

评估工具采用扩展的 MOTChallenge 风格文本格式，每一行表示一个目标在某一帧的观测或跟踪结果：
```

<帧号>, <目标ID>, <bb_left>, <bb_top>, <bb_width>, <bb_height>, , , ,

```
#### （1）三维点匹配格式（3D Points）

在仅进行三维点匹配、不使用二维检测框的场景下，二维框字段置为无效值：
```

<帧号>, <目标ID>, -1, -1, -1, -1, 1, , ,

```
其中 `<x>, <y>, <z>` 表示目标在世界坐标系或统一参考坐标系下的三维位置。

---

#### （2）二维目标框匹配格式（2D Bounding Boxes）

在传统二维多目标跟踪评估场景下，采用标准检测框信息：
```

<帧号>, <目标ID>, <bb_left>, <bb_top>, <bb_width>, <bb_height>, -1, 1, -1, -1

```
此模式下三维位置信息不参与评估。

---

### 2.3 评估运行方式

#### （1）三维点匹配评估

针对三维点匹配场景，执行如下命令：

```bash
cd TrackEval
python3 scripts/run_mot_challenge.py \
    --BENCHMARK your_dataset_name \
    --BOUNDINGBOX False
```

其中，`--BOUNDINGBOX False` 表示评估过程中不使用二维框重叠度，而基于三维空间中的欧氏距离进行匹配。

------

#### （2）二维目标框匹配评估

针对二维目标框匹配场景，执行如下命令：

```bash
cd TrackEval
python3 scripts/run_mot_challenge.py \
    --BENCHMARK your_dataset_name \
    --BOUNDINGBOX True
```

------

### 2.4 数据集目录结构与准备说明

#### （1）真值数据（Ground Truth）

在如下路径下新建数据集文件夹：

```
/data/gt/mot_challenge/
```

例如：

```
/data/gt/mot_challenge/mydata-train/
```

每个数据集文件夹中包含多个序列子文件夹，每个序列文件夹结构如下：

```
my_sequence/
├── gt/
│   └── gt.txt
└── seqinfo.ini
```

- `gt.txt`：对应序列的真实轨迹数据
- `seqinfo.ini`：序列配置信息文件

其中 `seqinfo.ini` 需至少包含以下字段：

- `name`：数据集名称
- `frameRate`：序列帧率
- `seqLength`：序列总帧数
- `imWidth`：图像宽度（如不使用可占位）

------

#### （2）跟踪结果数据（Trackers）

在如下路径中新建与真值数据集同名的文件夹：

```
/data/trackers/mot_challenge/mydata-train/
```

该目录下每个子文件夹对应一种待评估的跟踪方法，结构示例如下：

```
my_tracker/
└── data/
    └── your_dataset_name.txt
```

- `your_dataset_name.txt`：该方法在指定数据集上的跟踪结果文件
- 评估完成后，指标统计结果与可视化图表将自动生成于对应子文件夹中

------

#### （3）序列映射文件（SeqMaps）

在如下路径下新建序列映射文件：

```
/data/gt/mot_challenge/seqmaps/
```

该文件需按照 TrackEval 官方格式列出当前评估所包含的全部序列名称。

------

### 2.5 Python / Conda 环境依赖

评估工具依赖以下 Python 库：

```bash
pip install pycocotools
pip install scipy
pip install tabulate
```

建议使用 Python 3 运行环境。



