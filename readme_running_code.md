### 1. Run the demo
ViTPose github：[GitHub - ViTAE-Transformer/ViTPose"](https://github.com/ViTAE-Transformer/ViTPose/tree/main/)
代码内集成了当前在HPE中表现较好的模型，详细见demo文件夹。

see the doc files in ./demo/docs
e.g. for 2d human pose demo
- 2D HUman Pose Top-down Image demo
``` 
python demo/top_down_img_demo.py 
configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/hrnet_w48_coco_256x192.py 
https://download.openmmlab.com/mmpose/top_down/hrnet/hrnet_w48_coco_256x192-b9e0b3ab_20200708.pth 
--img-root tests/data/coco/ 
--json-file tests/data/coco/test_coco.json
--out-img-root vis_results
```

- 2D Human Pose Bottom-Up Image Demo
```shell  
python demo/bottom_up_img_demo.py configs/body/2d_kpt_sview_rgb_img/associative_embedding/coco/hrnet_w32_coco_512x512.py  https://download.openmmlab.com/mmpose/bottom_up/hrnet_w32_coco_512x512-bcb8c247_20200816.pth 
--img-path tests/data/coco/ 
--out-img-root vis_results
```

- ViT image demo
1. 下载vitpose模型，如下载[VitPose-B with classic decoder](https://onedrive.live.com/?authkey=%21ACOnX82tXdVFKYo&id=E534267B85818129%21163&cid=E534267B85818129&parId=root&parQt=sharedby&o=OneUp),防至对应目录，如：
2. 注意vit为top-down模型，需要使用`top_down_img_demo.py`的参数`--image-root` 
```shell  
python demo/top_down_img_demo.py configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/ViTPose_base_coco_256x192.py  
models/vitpose-b.pth
--img-root tests/data/coco/
--json-file tests/data/coco/test_coco.json
--out-img-root vis_results
```

Note: 
1. 运行后本地没有模型会自动下载对应模型文件，至相关dir
`Downloading: "https://download.openmmlab.com/mmpose/bottom_up/hrnet_w32_coco_512x512-bcb8c247_20200816.pth" to /home/mengqi/.cache/torch/hub/checkpoints/hrnet_w32_coco_512x512-bcb8c247_20200816.pth`
2. 如果使用自定义数据集，需要建立test文件夹来存放coco数据集，并修改相应数据集名称或参数名称以及jason文件路径
3. 运行时注意work dir路径的设置
4. 运行结果在vis_results内
###### 问题记录
- AttributeError: module 'numpy' has no attribute 'int'.
`np.int` was a deprecated alias for the builtin `int`. To avoid this error in existing code, use `int` by itself. Doing this will not modify any behavior and is safe. When replacing `np.int`, you may wish to use e.g. `np.int64` or `np.int32` to specify the precision. If you wish to review your current use, check the release note link for additional information.

Solution: 将代码中的`np.int`修改为：`np.int_`,`np.int32`或者`np.int64`


- `qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "/home/mengqi/work/Env/anaconda3/lib/python3.10/site-packages/cv2/qt/plugins" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.
Available platform plugins are: xcb, eglfs, minimal, minimalegl, offscreen, vnc, webgl.`
  
Solution: delete `libxcb.so` works

### 2. Train the model
- 注意:对于自定义数据集，需在configs文件下指定dataset相关路径，如`ViTPose_base_coco_256x192.py`中需要指定`bbox_file`、`ann_file``、img_prefix`
- data preparation可参考[2d_body_keypoint.md](https://github.com/ViTAE-Transformer/ViTPose/blob/d5216452796c90c6bc29f5c5ec0bdba94366768a/docs/en/tasks/2d_body_keypoint.md)，需使用HRNet提供的people detection的bbox结果jason文件，下载见上述链接内。
```
python tools/train.py
configs/body/2d_kpt_sview_rgb_img/topdown_heatmap/coco/ViTPose_base_coco_256x192.py
--gpus 1
--cfg-options model.pretrained=models/vitpose-b.pth
--seed 0
```

### 3D baseline 
模型部分代码的实现在mmpose/models/detectors/pose_lifter.py-class PoseLifter(BasePose)

- 运行demo测试
```commandline
python demo/body3d_two_stage_img_demo.py \
    configs/body/3d_kpt_sview_rgb_img/pose_lift/h36m/simplebaseline3d_h36m.py \
    https://download.openmmlab.com/mmpose/body3d/simple_baseline/simple3Dbaseline_h36m-f0ad73a4_20210419.pth \
    --json-file tests/data/h36m/h36m_coco.json \
    --img-root tests/data/h36m \
    --camera-param-file tests/data/h36m/cameras.pkl \
    --only-second-stage \
    --out-img-root vis_results \
    --rebase-keypoint-height \
    --show-ground-truth
```