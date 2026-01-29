# mdmot_bench

本项目面向**分布式动态观测机条件下的被动定位与多目标跟踪研究**，提供一套可复现实验流程的**数据集规范**与**评估工具**实现，旨在支持多视角、多目标、动态平台场景下的数据关联、定位与跟踪算法评测。

项目主要包含两部分内容：
- 动态观测机二对三和三对五的被动观测数据集
- 多目标跟踪评估工具（基于 TrackEval 的定制改版）

---

## 一、 二对三数据集

### 1. 数据集概述

本数据集针对动态观测机二对三（2 Observers → 3 Targets）的被动观测场景构建，模拟或采集观测平台持续运动、多目标同时存在条件下的视觉观测数据，适用于以下研究方向：

- 被动定位（Angle-only Localization）
- 多目标数据关联
- 多目标跟踪（MOT）
- 多平台协同感知与状态估计

具体录制的rosbag的网盘下载链接为：[link](https://pan.baidu.com/s/1dEMe61BcntzswjDKO6Wnkw?pwd=mxmq )

---

### 2. 场景配置

- 观测机数量：2  
- 目标数量：3  
- 观测方式：被动视觉观测 
- 观测机状态：动态运动  
- 目标状态：动态运动

---

### 3. 数据内容

每个数据序列原则上包含以下信息：

- 图像数据：来自两架观测机的图像序列，支持 RGB格式
- 目标位姿数据：目标机在世界坐标系下的位置与姿态

- 观测机位姿数据：观测机在世界坐标系下的位置与姿态

---

### 4. ROS 话题结构

- 图像话题  
  - `/obs_01/image_raw`  
  - `/obs_02/image_raw`
- 观测机位姿话题  
  - `/vrpn_client_node/obs_01/pose`  
  - `/vrpn_client_node/obs_02/pose`
- 目标机位姿话题
  - `/vrpn_client_node/target_01/pose`
  - `/vrpn_client_node/target_02/pose`
  - `/vrpn_client_node/target_03/pose`


---

### 5. 相机参数

存放到`2_3-parameter.txt`





## 二、三维跟踪指标评估工具

### 2.1 工具概述

本项目采用 **TrackEval** 的改版实现，用于多目标跟踪结果的统一评估，并针对**三维点匹配（3D Point Matching）**场景进行了定制扩展，适用于被动定位与多观测机协同感知任务。

评估框架支持多种主流多目标跟踪评价指标（包括 HOTA 系列指标），并提供完整的评估流程、结果统计与可视化功能。同时，通过参数配置可同时兼容 **三维点级匹配** 与 **二维目标框匹配** 两类评估模式。

---

### 2.2 数据格式说明

评估工具采用扩展的 MOTChallenge 风格文本格式，每一行表示一个目标在某一帧的观测或跟踪结果：

```
<帧号>, <目标ID>, -1, -1, -1, -1, 1, x, y, z
```
其中 `<x>, <y>, <z>` 表示目标在世界坐标系或统一参考坐标系下的三维位置。

---

### 2.3 评估运行方式

针对三维点匹配场景，执行如下命令：

```bash
cd TrackEval
python3 scripts/run_mot_challenge.py \
    --BENCHMARK your_dataset_name \
    --BOUNDINGBOX False
```

其中，`--BOUNDINGBOX False` 表示评估过程中不使用二维框重叠度，而是基于三维空间中的欧氏距离进行匹配。如果是仅针对不同方法得到的数据，可以使用如下示例：

```
cd TrackEval
python3 scripts/run_mot_challenge.py \
    --BENCHMARK air2air \
    --BOUNDINGBOX False \
    --SEQ_INFO 3d-target
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



