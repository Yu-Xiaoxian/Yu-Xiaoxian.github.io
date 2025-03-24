---
title: 从 2D 到 BEV，LSS 技术如何重塑自动驾驶感知？
tags:
  - BEV
  - LSS
categories:
  - Algorithms
date: 2025-03-21 19:20:16
---

## 1. BEV 与 LSS

**BEV** (Bird-Eye-View)，鸟瞰视角是一种比相机视角更直观、信息更丰富的视角，非常适合用来做多传感器信息融合，特别是在跨相机物体跟踪方面，能够提供一致性更强的信息。

**LSS**（Lift-Splat-Shoot），是一种能够将多视角图像转换为 BEV 表示的技术。作为 BEV 领域的一个经典方法，LSS 兼顾了效率和准确性，在自动驾驶、机器人感知等任务中都有广泛应用。

论文：[Lift, Splat, Shoot: Encoding Images From Arbitrary Camera Rigs by Implicitly Unprojecting to 3D](https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123590188.pdf)

代码：[https://github.com/nv-tlabs/lift-splat-shoot](https://github.com/nv-tlabs/lift-splat-shoot#)

## 2. Motivation

- 先做深度估计和特征融合，然后投影到 BEV 视图中
- 在 BEV 视图中做特征融合，在融合后的特征图上做检测、规划等任务

## 3. Method

### 3.1 Lift（升维）

将 2D 图像（WxHx3）增加深度信息，升维到 3D（WxHxD），其中 D 表示不同的深度，为不同的深度学习出C维的特征，就得到了 WxHxDxC 维度的视锥点云

![LSS-Lift](https://s21.ax1x.com/2025/03/17/pEdFjnx.md.png)

### 3.2 Splat（投影）

将图像特征投影到BEV空间中，采用 cumsum trick 的方法将特征求和，得到 CxXxY 维度的 BEV 特征，至此 BEV 特征已提取完成

![LSS-Splat](https://s21.ax1x.com/2025/03/17/pEdkP9H.md.png)

之所以要做求和操作，是因为会出现多个视锥点云落在同一BEV栅格的情况，有两种情况会出现

- 不同高度的视锥点云会落在同一个栅格内，比如电线杆上的不同像素点
- 不同相机之间存在 overlap，两个相机观测到的同一个物体，会落在同一个BEV栅格

### 3.3 Shoot

将轨迹通过模板投影到 BEV 空间上，并计算轨迹的 cost，实现端到端的轨迹规划，这一步是锦上添花的工作

![LSS-Shoot](https://s21.ax1x.com/2025/03/17/pEdkpND.md.png)

## 4. 训练

### 4.1 Cost

- 图像 backbone 采用 EfficientNet，通过预训练得到深度估计，需要标记检测出的物体在BEV视角下的投影
- 监督真值是实例分割结果、可行驶区域，Loss 定义为预测结果与 groud truth 的交叉熵

## 5. 代码解析

首先看模型的初始化函数，包含以下三个模块：
- camencode：图像特征提取
- bevencode：BEV检测backbone
- frustum：视锥，用于图像点云坐标和BEV栅格间的坐标转换

其它变量是模型的参数，看名称就可以知道是什么含义，这里不做展开

```python
class LiftSplatShoot(nn.Module):
    def __init__(self, grid_conf, data_aug_conf, outC):
        super(LiftSplatShoot, self).__init__()
        self.grid_conf = grid_conf
        self.data_aug_conf = data_aug_conf

        dx, bx, nx = gen_dx_bx(self.grid_conf['xbound'],
                                              self.grid_conf['ybound'],
                                              self.grid_conf['zbound'],
                                              )
        self.dx = nn.Parameter(dx, requires_grad=False)
        self.bx = nn.Parameter(bx, requires_grad=False)
        self.nx = nn.Parameter(nx, requires_grad=False)

        self.downsample = 16
        self.camC = 64
        self.frustum = self.create_frustum()
        self.D, _, _, _ = self.frustum.shape
        self.camencode = CamEncode(self.D, self.camC, self.downsample)
        self.bevencode = BevEncode(inC=self.camC, outC=outC)

        # toggle using QuickCumsum vs. autograd
        self.use_quickcumsum = True
```

模型的推理 pipeline 如下，forward() 调用了 get_voxels() 方法，get_voxels() 中：
- get_geometry() 属于 Lift 操作
- voxel_polling() 属于 Splat 操作

下面小结我们详细介绍这两步操作，这里先介绍一下 forward 的函数的参数：
- x：BxN 张图像
- rots, trans：相机的外参，以旋转矩阵和平移矩阵形式表示
- intrins：相机内参
- post_rots，post_trans：图像增强时使用的旋转矩阵和平移矩阵，用于在模型训练时撤销图像增强引入的位姿变化

```python
def get_voxels(self, x, rots, trans, intrins, post_rots, post_trans):
    geom = self.get_geometry(rots, trans, intrins, post_rots, post_trans)
    x = self.get_cam_feats(x)

    x = self.voxel_pooling(geom, x)

    return x

def forward(self, x, rots, trans, intrins, post_rots, post_trans):
    x = self.get_voxels(x, rots, trans, intrins, post_rots, post_trans)
    x = self.bevencode(x)
    return x
```

### 5.1 Lift

Lift 操作分为两步：

- 生成图像坐标->视锥坐标的变换
- 计算视锥坐标->投影坐标的变换

第一步比较简单，需要了解普通相机模型，得到的 xy 是单位深度下的像素坐标

```python
def create_frustum(self):
    # make grid in image plane
    ogfH, ogfW = self.data_aug_conf['final_dim']
    fH, fW = ogfH // self.downsample, ogfW // self.downsample
    # ds: DxWxH 表示每个点的深度
    ds = torch.arange(*self.grid_conf['dbound'], dtype=torch.float).view(-1, 1, 1).expand(-1, fH, fW)
    D, _, _ = ds.shape
    # xs：DxWxH 表示每个点的x坐标
    xs = torch.linspace(0, ogfW - 1, fW, dtype=torch.float).view(1, 1, fW).expand(D, fH, fW)
    # ys：DxWxH 表示每个点的y坐标
    ys = torch.linspace(0, ogfH - 1, fH, dtype=torch.float).view(1, fH, 1).expand(D, fH, fW)

    # D x H x W x 3
    frustum = torch.stack((xs, ys, ds), -1)
    return nn.Parameter(frustum, requires_grad=False)
```

主要分为三步：
- 去除相机增强的位姿变换影响
- 转换到真实坐标系再乘以内存去畸变，需要注意的是，上一步得到的 xy 是单位深度下的相机坐标，不同深度对应的 xy 是一样的，因此需要乘以深度 d 才能得到真实世界的坐标
- 通过外参转换到 BEV 坐标系下

```python
def get_geometry(self, rots, trans, intrins, post_rots, post_trans):
    """Determine the (x,y,z) locations (in the ego frame)
    of the points in the point cloud.
    Returns B x N x D x H/downsample x W/downsample x 3
    """
    B, N, _ = trans.shape

    # 撤销图像增强的图像变换
    # B x N x D x H x W x 3
    points = self.frustum - post_trans.view(B, N, 1, 1, 1, 3)
    points = torch.inverse(post_rots).view(B, N, 1, 1, 1, 3, 3).matmul(points.unsqueeze(-1))

    # cam_to_ego
    points = torch.cat((points[:, :, :, :, :, :2] * points[:, :, :, :, :, 2:3],
                        points[:, :, :, :, :, 2:3]
                        ), 5)
    combine = rots.matmul(torch.inverse(intrins))
    points = combine.view(B, N, 1, 1, 1, 3, 3).matmul(points).squeeze(-1)
    points += trans.view(B, N, 1, 1, 1, 3)

    return points
```

### 5.2 Splat

- 首先，将输入的矩阵一维化，得到了一维索引
- 由于一维索引是在 BEV 空间下按照顾 X->Y->Z->B 的顺序生成的，因此相邻体素的索引也是相邻的
- 用排序后的索引对相机特征进行重排，这样相邻的体素对应的相机特征也是相邻的
- 对排序后的特征进行 acumsum，方便后续特征计算与求导

注：这里采用的是 sum pooling，而不是 max pooling 或者 average pooling，是为了保存特征响应比较强的体素，定性来讲，被观测比较多的特征，在 BEV 空间上会有更强的 feature

```python
def voxel_pooling(self, geom_feats, x):
    B, N, D, H, W, C = x.shape
    Nprime = B*N*D*H*W

    # flatten x
    x = x.reshape(Nprime, C)

    # flatten indices
    geom_feats = ((geom_feats - (self.bx - self.dx/2.)) / self.dx).long()
    geom_feats = geom_feats.view(Nprime, 3)
    batch_ix = torch.cat([torch.full([Nprime//B, 1], ix,
                         device=x.device, dtype=torch.long) for ix in range(B)])
    geom_feats = torch.cat((geom_feats, batch_ix), 1)

    # filter out points that are outside box
    kept = (geom_feats[:, 0] >= 0) & (geom_feats[:, 0] < self.nx[0])\
        & (geom_feats[:, 1] >= 0) & (geom_feats[:, 1] < self.nx[1])\
        & (geom_feats[:, 2] >= 0) & (geom_feats[:, 2] < self.nx[2])
    x = x[kept]
    geom_feats = geom_feats[kept]

    # get tensors from the same voxel next to each other
    ranks = geom_feats[:, 0] * (self.nx[1] * self.nx[2] * B)\
        + geom_feats[:, 1] * (self.nx[2] * B)\
        + geom_feats[:, 2] * B\
        + geom_feats[:, 3]
    sorts = ranks.argsort()
    x, geom_feats, ranks = x[sorts], geom_feats[sorts], ranks[sorts]

    # cumsum trick
    if not self.use_quickcumsum:
        x, geom_feats = cumsum_trick(x, geom_feats, ranks)
    else:
        x, geom_feats = QuickCumsum.apply(x, geom_feats, ranks)

    # griddify (B x C x Z x X x Y)
    final = torch.zeros((B, C, self.nx[2], self.nx[0], self.nx[1]), device=x.device)
    final[geom_feats[:, 3], :, geom_feats[:, 2], geom_feats[:, 0], geom_feats[:, 1]] = x

    # collapse Z
    final = torch.cat(final.unbind(dim=2), 1)

    return final
```

## 6. 总结

- LSS 总体思路蛮直观的，图像特征按照深度投影到 BEV 平面上，经过 pooling 操作得到 BEV feature
- 先学习得到深度分布D 和 特征C，然后两者点积得到 DxC 维向量，比较巧妙
- cumsum_trick 虽然很巧妙，但是容易增加理解复杂度，这个技巧只是为了加速，本质上还是 sum pooling，不用这个技巧应该也能得到相同的效果

下一篇，学习 BEVFormer