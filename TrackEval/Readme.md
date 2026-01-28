
# TrackEval (For 3D Points Matching Scenarios)
*Code for evaluating object tracking.*

This codebase provides code for a number of different tracking evaluation metrics (including the [HOTA metrics](https://link.springer.com/article/10.1007/s11263-020-01375-2)), as well as supporting running all of these metrics on a number of different tracking benchmarks. Plus plotting of results and other things one may want to do for tracking evaluation.

## Usage
### Data Format
```bash
<frame number>, <object id>, <bb_left>, <bb_top>, <bb_width>, <bb_height>, <confidence>, <x>, <y>, <z>
## For 3D points
<frame number>, <object id>, -1, -1, -1, -1, 1, <x>, <y>, <z>
## For 2D boundingbox
<frame number>, <object id>, <bb_left>, <bb_top>, <bb_width>, <bb_height>, -1, 1, -1, -1
```
### Run
Follow the commands below to process 3D point matching scenarios:
```bash
cd TrackEval
python3 scripts/run_mot_challenge.py --BENCHMARK your_dataset_name --BOUNDINGBOX False 
```
Follow the commands below to process 2D boundingboxes matching scenarios:
```bash
cd TrackEval
python3 scripts/run_mot_challenge.py --BENCHMARK your_dataset_name --BOUNDINGBOX True 
```

## Conda Environment
```bash
pip install pycocotools
pip install scipy
pip install tabulate
```

## Memo
在`/data/gt/mot_challenge`中新建数据集文件夹"`mydata-train`"，其中包含多个数据集子文件夹，每个子文件夹由`gt`文件夹和`seqinfo.ini`文件构成。将真值文件放在`gt`文件夹下，并重命名为`gt.txt`；更改`seqinfo.ini`文件参数，`name`为数据集名，`frameRate`为该数据集帧率，`seqLength`为该数据集总帧数，`imWidth`为图像宽。

在`/data/trackers/mot_challenge`中新建相同文件夹"`mydata-train`"，其中包含多个子文件夹（每个子文件夹对应一种需要评估的方法），每个子文件中均含有一个`data`文件，其中包含该方法跟踪结果文件`your_dataset_name.txt`。后续评估结果图会生成在子文件夹中。

在`/data/gt/mot_challenge/seqmaps`中新建文件，对照官方格式，罗列所有数据集名称。

## Citing TrackEval

If you use this code in your research, please use the following BibTeX entry:

```BibTeX
@misc{luiten2020trackeval,
  author =       {Jonathon Luiten, Arne Hoffhues},
  title =        {TrackEval},
  howpublished = {\url{https://github.com/JonathonLuiten/TrackEval}},
  year =         {2020}
}
```

Furthermore, if you use the HOTA metrics, please cite the following paper:

```
@article{luiten2020IJCV,
  title={HOTA: A Higher Order Metric for Evaluating Multi-Object Tracking},
  author={Luiten, Jonathon and Osep, Aljosa and Dendorfer, Patrick and Torr, Philip and Geiger, Andreas and Leal-Taix{\'e}, Laura and Leibe, Bastian},
  journal={International Journal of Computer Vision},
  pages={1--31},
  year={2020},
  publisher={Springer}
}
```

If you use any other metrics please also cite the relevant papers, and don't forget to cite each of the benchmarks you evaluate on.
