---
layout: page
title: Floorplan-Guided 3D Point Cloud Reconstruction
description: "<span class='lang-en'>Aligning and fusing indoor point clouds with 2D floorplan priors for digital twins, robotics, and BIM.</span><span class='lang-zh'>利用 2D 户型图先验对齐与融合室内点云，应用于数字孪生、机器人与 BIM。</span>"
importance: 3
category: research
---

<div class="lang-en" markdown="1">

**HKUST UROP · Supervisor: Prof. Gary Chan · Sep 2025 – Dec 2025**

> **Status: work in progress.** The full pipeline has been assembled and the core 2D registration module is trained and tested; the final end-to-end evaluation on real scans has not been carried out yet. This page documents the idea, the design, and the intermediate results honestly.

**In one sentence:** given a set of photos of a building's interior, plus one floorplan of that floor, this pipeline reconstructs a 3D point cloud of the space **at real-world scale**.

## Motivation: point clouds without a ruler

Point clouds are the standard 3D representation in industry — digital twins, robotics, BIM — and their usefulness depends directly on geometric fidelity. Photos-to-3D reconstruction (COLMAP, or feed-forward models such as VGGT) produces point clouds whose geometry looks right, but whose **absolute scale is unknown**: the model cannot tell whether a room is 5 meters or 50 centimeters wide. Meanwhile, 2D floorplans are easy to obtain and carry exactly the missing information — true xy dimensions. The idea of this project is to use the floorplan as a **ruler and structural prior** for the point cloud.

## The key idea: flatten 3D into 2D

Registering a noisy 3D point cloud directly against a floorplan is hard. Instead, we **project the point cloud top-down into a 2D density map** — walls, which contain most of the points, show up as bright ridges. The 3D-to-floorplan problem then becomes a much friendlier **2D-to-2D image registration** problem: find the rotation, translation, and scale that aligns the density map with the floorplan, then apply that transformation back to the 3D cloud.

<img src="{{ '/assets/img/projects/floorplan/density_map.png' | relative_url }}" alt="From a 3D point cloud to a 2D density map, overlaid with the floorplan" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">A synthetic example of the core intuition: (a) a 3D point cloud of walls, (b) its top-down density map, (c) the density map aligned with the floorplan outline.</p>

## Why existing methods fall short

**Feed-forward reconstruction models (e.g. VGGT).** Under GPU memory limits, only a few views fit in a batch, and the model loses the multi-view constraints it needs. The result is *structural ghosting* (double-vision artifacts) and *depth-scale ambiguity* — exactly the failure modes this project must fix downstream.

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/vggt_bs3.png' | relative_url }}" alt="VGGT point cloud, batch size 3" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">VGGT, batch size 3</figcaption>
  </figure>
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/vggt_bs5.png' | relative_url }}" alt="VGGT point cloud, batch size 5" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">VGGT, batch size 5</figcaption>
  </figure>
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/vggt_gt.png' | relative_url }}" alt="DL3DV-10K ground-truth point cloud" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Ground truth (DL3DV-10K)</figcaption>
  </figure>
</div>
<p style="color:#666; font-size:0.9em;">Same scene, three reconstructions: with small batches the VGGT point clouds are sparse, noisy, and show ghosting; the ground truth on the right is what a clean scan should look like.</p>

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/depth_bs3.png' | relative_url }}" alt="VGGT depth, batch size 3" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">VGGT depth, batch 3</figcaption>
  </figure>
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/depth_bs5.png' | relative_url }}" alt="VGGT depth, batch size 5" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">VGGT depth, batch 5</figcaption>
  </figure>
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/depth_gt.png' | relative_url }}" alt="Ground-truth depth" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Ground-truth depth</figcaption>
  </figure>
</div>
<p style="color:#666; font-size:0.9em;">The depth-scale problem up close: inconsistent depth estimates across small batches visibly distort the geometry.</p>

**Traditional geometric registration (FPFH + TEASER++).** Handcrafted local descriptors break when scans are incomplete, and architecturally symmetric structures (identical corridors, repeated pillars) produce indistinguishable descriptors, driving the inlier ratio below what even robust solvers can handle. A case study on the HKUST Atrium — merging two partial COLMAP reconstructions — fails visibly:

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/atrium_pc1.png' | relative_url }}" alt="Incomplete atrium point cloud 1" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Input 1: partial COLMAP cloud</figcaption>
  </figure>
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/atrium_pc2.png' | relative_url }}" alt="Incomplete atrium point cloud 2" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Input 2: partial COLMAP cloud</figcaption>
  </figure>
</div>
<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/atrium_merge_normal.png' | relative_url }}" alt="Failed merge, normal view" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Merged by FPFH+TEASER++, side view</figcaption>
  </figure>
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/atrium_merge_bird.png' | relative_url }}" alt="Failed merge, top view" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Merged result, top view — clearly misaligned</figcaption>
  </figure>
</div>

## Pipeline

<img src="{{ '/assets/img/projects/floorplan/pipeline.png' | relative_url }}" alt="Pipeline: photos and floorplan to metric-scale point cloud" style="width:100%; border-radius:8px;">

The numbered stages in the figure correspond to:

1. **Reconstruct** an unscaled point cloud from the photos (COLMAP / VGGT).
2. **Project** the cloud top-down into a 2D density map (the vertical axis is recovered separately by ground-plane fitting).
3. **Register** the density map against the floorplan with a learnable 2D registration module: a lightweight **SuperPoint-Lite** network (VGG-style encoder, supervised by Harris-corner pseudo ground truth with a repulsion loss) predicts keypoint heatmaps on both images, and a **Sinkhorn matcher** establishes correspondences.
4. **Solve** for the similarity transform {R, t, s} with **RANSAC** and apply it to the original 3D cloud — the floorplan's dimensions become the cloud's dimensions.

The learnable core is deliberately small: keypoints are learned on 2D floorplan-like images only, which is far more memory-efficient than high-resolution 3D transformers, and — unlike FPFH-style matching — it does not require the two inputs to overlap in 3D at all.

## Preliminary results

The 2D registration module was trained on an augmented dataset built from the 4th-floor plan of the HKUST Academic Building: one sample serves as the **template** (the floorplan), the others as **sources** (density maps with random rotations and translations applied).

**How to read these figures:** the network does not output a transform directly — it outputs a *heatmap* per image, where each bright blob means "the network believes there is a keypoint here". Sparse keypoints are then extracted from the heatmaps and matched across the two images. So below you see, left to right: detected keypoints on the template → its predicted heatmap → keypoints on the source → its predicted heatmap → the final cross-image matches.

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:150px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/train_p1.png' | relative_url }}" alt="Template keypoints" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Template keypoints (floorplan)</figcaption>
  </figure>
  <figure style="flex:1; min-width:150px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/train_p2.png' | relative_url }}" alt="Predicted heatmap on template" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Predicted heatmap, template</figcaption>
  </figure>
  <figure style="flex:1; min-width:150px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/train_p3.png' | relative_url }}" alt="Source keypoints" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Source keypoints (density map)</figcaption>
  </figure>
  <figure style="flex:1; min-width:150px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/train_p4.png' | relative_url }}" alt="Predicted heatmap on source" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Predicted heatmap, source</figcaption>
  </figure>
  <figure style="flex:1; min-width:150px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/train_p5.png' | relative_url }}" alt="Keypoint matches" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Keypoint matches</figcaption>
  </figure>
</div>

<div style="display:flex; gap:2%; flex-wrap:wrap; align-items:flex-start;">
  <figure style="flex:1; min-width:260px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/training_loss.png' | relative_url }}" alt="Training loss curve" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Training loss over 50 epochs — noisy but clearly descending.</figcaption>
  </figure>
  <div style="flex:2; min-width:320px;">
    <div style="display:flex; gap:1.5%; flex-wrap:wrap;">
      <figure style="flex:1; min-width:140px; margin:0 0 8px 0;">
        <img src="{{ '/assets/img/projects/floorplan/reg_p1.png' | relative_url }}" alt="Template with predicted heatmap" style="width:100%; border-radius:8px;">
        <figcaption style="text-align:center; color:#666; font-size:0.85em;">Template + predicted heatmap</figcaption>
      </figure>
      <figure style="flex:1; min-width:140px; margin:0 0 8px 0;">
        <img src="{{ '/assets/img/projects/floorplan/reg_p2.png' | relative_url }}" alt="Source with predicted heatmap" style="width:100%; border-radius:8px;">
        <figcaption style="text-align:center; color:#666; font-size:0.85em;">Source + predicted heatmap</figcaption>
      </figure>
      <figure style="flex:1; min-width:140px; margin:0 0 8px 0;">
        <img src="{{ '/assets/img/projects/floorplan/reg_p3.png' | relative_url }}" alt="Warped source" style="width:100%; border-radius:8px;">
        <figcaption style="text-align:center; color:#666; font-size:0.85em;">Source after applying the predicted transform</figcaption>
      </figure>
    </div>
    <p style="color:#666; font-size:0.9em;">Test-time check: the source density map was randomly rotated and translated; after applying the transform predicted from the matched keypoints, the warped source (right) lines up with the template layout — the alignment is visually satisfactory.</p>
  </div>
</div>

## Interactive point cloud

A real reconstructed point cloud from this project (voxel-downsampled from ~8.2M to ~65k points for the browser; colors are the true RGB values per point — no mesh, no rendering trickery). Drag to rotate, scroll to zoom. If the interactive view fails to load, a static render is shown instead.

<object data="{{ '/assets/plotly/floorplan_pointcloud.html' | relative_url }}" type="text/html" style="width:100%; height:540px; border:1px solid #ddd; border-radius:8px;">
  <img src="{{ '/assets/img/projects/floorplan/dense1.png' | relative_url }}" alt="Static render of the reconstructed point cloud" style="width:100%; border-radius:8px;">
</object>
<p style="text-align:center; color:#666; font-size:0.9em;">
  <a href="{{ '/assets/plotly/floorplan_pointcloud.html' | relative_url }}" target="_blank">Open the interactive point cloud full-screen ↗</a>
</p>

## Limitations and next steps

- **Symmetric or feature-poor geometry** remains hard: repeated structures can still confuse the learned matcher, just as they confuse FPFH.
- **The floorplan is assumed static and accurate** — furniture rearrangements or drawing errors propagate into the alignment.
- **End-to-end evaluation is pending:** the remaining work is to run the full pipeline on real scans and quantify the recovered scale against ground-truth measurements.

</div>

<div class="lang-zh" markdown="1">

**香港科技大学 UROP · 导师：Gary Chan 教授 · 2025 年 9 月 – 12 月**

> **状态：进行中。** 完整流水线已经搭建完毕，核心的 2D 配准模块已完成训练与测试；但针对真实扫描数据的端到端最终评估尚未进行。本页如实地记录研究动机、方案设计与阶段性结果。

**一句话概括：** 输入一组建筑内部照片，加上一张该楼层的平面图，输出一个**具有真实尺度**的 3D 点云。

## 动机：没有尺子的点云

点云是工业界最常用的 3D 表示——数字孪生、机器人、BIM 都依赖它，而其价值直接取决于几何保真度。然而，从照片重建 3D（COLMAP，或 VGGT 等前馈模型）得到的点云，几何形状虽然看起来正确，**绝对尺度却是未知的**：模型无法分辨一个房间是 5 米宽还是 50 厘米宽。与此同时，2D 平面图很容易获得，并且恰好携带了点云缺失的信息——真实的 xy 尺寸。本项目的核心想法，就是把平面图当作点云的**尺子与结构先验**。

## 核心直觉：把 3D 压成 2D

把噪声很大的 3D 点云直接和平面图做配准是很困难的。我们的做法是：**把点云俯视投影成一张 2D 密度图**——墙体集中了大部分点，在密度图上呈现为明亮的脊线。于是"3D 点云 vs 平面图"的难题就转化为友好得多的**2D 图像配准**问题：找到把密度图对齐到平面图的旋转、平移与缩放，再把这个变换施加回 3D 点云。

<img src="{{ '/assets/img/projects/floorplan/density_map.png' | relative_url }}" alt="从 3D 点云到 2D 密度图，再与平面图叠加" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">核心直觉的合成示例：(a) 墙体组成的 3D 点云，(b) 其俯视密度图，(c) 密度图与平面图轮廓对齐叠加。</p>

## 为什么现有方法不够用

**前馈重建模型（如 VGGT）。** 受 GPU 显存限制，一个批次只能容纳少量视角，模型因此丢失所需的多视角约束。结果是*结构鬼影*（类似"重影"的伪影）和*深度尺度歧义*——这正是本项目要在下游修复的两类失效模式。

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/vggt_bs3.png' | relative_url }}" alt="VGGT 点云，batch size 3" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">VGGT，batch size 3</figcaption>
  </figure>
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/vggt_bs5.png' | relative_url }}" alt="VGGT 点云，batch size 5" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">VGGT，batch size 5</figcaption>
  </figure>
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/vggt_gt.png' | relative_url }}" alt="DL3DV-10K 真值点云" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">真值（DL3DV-10K）</figcaption>
  </figure>
</div>
<p style="color:#666; font-size:0.9em;">同一场景的三种重建：小批次下 VGGT 的点云稀疏、有噪声且出现鬼影；右侧真值展示了干净扫描应有的样子。</p>

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/depth_bs3.png' | relative_url }}" alt="VGGT 深度，batch size 3" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">VGGT 深度，batch 3</figcaption>
  </figure>
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/depth_bs5.png' | relative_url }}" alt="VGGT 深度，batch size 5" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">VGGT 深度，batch 5</figcaption>
  </figure>
  <figure style="flex:1; min-width:180px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/depth_gt.png' | relative_url }}" alt="真值深度" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">真值深度</figcaption>
  </figure>
</div>
<p style="color:#666; font-size:0.9em;">近距离看深度尺度问题：小批次之间深度估计不一致，几何结构被明显扭曲。</p>

**传统几何配准（FPFH + TEASER++）。** 手工局部描述子在扫描不完整时会失效；而建筑中常见的对称结构（相同的走廊、重复的柱子）会产生无法区分的描述子，使内点比例低到连鲁棒求解器也无法承受。在香港科技大学中庭（Atrium）上的案例研究——合并两段 COLMAP 重建的残缺点云——失败得非常直观：

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/atrium_pc1.png' | relative_url }}" alt="中庭残缺点云 1" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">输入 1：COLMAP 残缺点云</figcaption>
  </figure>
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/atrium_pc2.png' | relative_url }}" alt="中庭残缺点云 2" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">输入 2：COLMAP 残缺点云</figcaption>
  </figure>
</div>
<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/atrium_merge_normal.png' | relative_url }}" alt="合并失败，侧视" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">FPFH+TEASER++ 合并结果，侧视</figcaption>
  </figure>
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/atrium_merge_bird.png' | relative_url }}" alt="合并失败，俯视" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">合并结果，俯视——错位清晰可见</figcaption>
  </figure>
</div>

## 流水线

<img src="{{ '/assets/img/projects/floorplan/pipeline.png' | relative_url }}" alt="流水线：照片与平面图到真实尺度点云" style="width:100%; border-radius:8px;">

图中的编号阶段与下面四步一一对应：

1. **重建**：从照片得到无尺度的点云（COLMAP / VGGT）。
2. **投影**：把点云俯视投影为 2D 密度图（竖直方向另由地面拟合恢复）。
3. **配准**：用可学习的 2D 配准模块把密度图对齐到平面图——轻量级的 **SuperPoint-Lite** 网络（VGG 式编码器，用 Harris 角点伪真值加 repulsion loss 监督）分别在两张图上预测关键点热力图，再用 **Sinkhorn 匹配器**建立对应关系。
4. **求解**：用 **RANSAC** 求解相似变换 {R, t, s} 并施加到原始 3D 点云——平面图的尺寸从此成为点云的尺寸。

可学习核心被刻意设计得很小：关键点只在 2D 的平面图类图像上学习，显存消耗远低于高分辨率 3D transformer；而且与 FPFH 类匹配不同，它**不要求两个输入在 3D 中存在重叠区域**。

## 阶段性结果

2D 配准模块在由香港科技大学主楼 4 层平面图构建的数据增广数据集上训练：一个样本作为**模板**（平面图），其余作为**源**（施加了随机旋转与平移的密度图）。

**这些图怎么看：** 网络并不直接输出变换，而是对每张图输出一张*热力图*——每个亮斑表示"网络认为这里有一个关键点"。随后从热力图中提取稀疏关键点，并在两张图之间做匹配。所以下面从左到右依次是：模板上的关键点 → 模板的预测热力图 → 源上的关键点 → 源的预测热力图 → 最终的跨图匹配。

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:150px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/train_p1.png' | relative_url }}" alt="模板关键点" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">模板关键点（平面图）</figcaption>
  </figure>
  <figure style="flex:1; min-width:150px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/train_p2.png' | relative_url }}" alt="模板预测热力图" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">模板预测热力图</figcaption>
  </figure>
  <figure style="flex:1; min-width:150px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/train_p3.png' | relative_url }}" alt="源关键点" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">源关键点（密度图）</figcaption>
  </figure>
  <figure style="flex:1; min-width:150px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/train_p4.png' | relative_url }}" alt="源预测热力图" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">源预测热力图</figcaption>
  </figure>
  <figure style="flex:1; min-width:150px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/train_p5.png' | relative_url }}" alt="关键点匹配" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">关键点匹配</figcaption>
  </figure>
</div>

<div style="display:flex; gap:2%; flex-wrap:wrap; align-items:flex-start;">
  <figure style="flex:1; min-width:260px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/floorplan/training_loss.png' | relative_url }}" alt="训练损失曲线" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">50 个 epoch 的训练损失——有波动但整体明显下降。</figcaption>
  </figure>
  <div style="flex:2; min-width:320px;">
    <div style="display:flex; gap:1.5%; flex-wrap:wrap;">
      <figure style="flex:1; min-width:140px; margin:0 0 8px 0;">
        <img src="{{ '/assets/img/projects/floorplan/reg_p1.png' | relative_url }}" alt="模板与预测热力图" style="width:100%; border-radius:8px;">
        <figcaption style="text-align:center; color:#666; font-size:0.85em;">模板 + 预测热力图</figcaption>
      </figure>
      <figure style="flex:1; min-width:140px; margin:0 0 8px 0;">
        <img src="{{ '/assets/img/projects/floorplan/reg_p2.png' | relative_url }}" alt="源与预测热力图" style="width:100%; border-radius:8px;">
        <figcaption style="text-align:center; color:#666; font-size:0.85em;">源 + 预测热力图</figcaption>
      </figure>
      <figure style="flex:1; min-width:140px; margin:0 0 8px 0;">
        <img src="{{ '/assets/img/projects/floorplan/reg_p3.png' | relative_url }}" alt="变换后的源" style="width:100%; border-radius:8px;">
        <figcaption style="text-align:center; color:#666; font-size:0.85em;">施加预测变换后的源</figcaption>
      </figure>
    </div>
    <p style="color:#666; font-size:0.9em;">测试时的检验：源密度图被随机旋转和平移过；施加由匹配关键点预测出的变换后，右图的配准结果与模板布局对得上——对齐效果在视觉上令人满意。</p>
  </div>
</div>

## 交互式点云

本项目真实重建的点云（为浏览器展示用体素降采样从约 817 万点压到约 6.5 万点；颜色是每个点的真实 RGB——没有网格、没有渲染修饰）。拖动旋转，滚轮缩放。若交互视图加载失败，会自动显示一张静态渲染图。

<object data="{{ '/assets/plotly/floorplan_pointcloud.html' | relative_url }}" type="text/html" style="width:100%; height:540px; border:1px solid #ddd; border-radius:8px;">
  <img src="{{ '/assets/img/projects/floorplan/dense1.png' | relative_url }}" alt="重建点云的静态渲染" style="width:100%; border-radius:8px;">
</object>
<p style="text-align:center; color:#666; font-size:0.9em;">
  <a href="{{ '/assets/plotly/floorplan_pointcloud.html' | relative_url }}" target="_blank">全屏打开交互式点云 ↗</a>
</p>

## 局限与下一步

- **对称或弱特征几何**依然是难点：重复结构仍可能迷惑学习匹配器，正如它们迷惑 FPFH 一样。
- **假设平面图静态且准确**——家具移动或图纸误差会传播到配准结果中。
- **端到端评估尚待完成：** 接下来的工作是在真实扫描数据上运行完整流水线，并用实测真值量化恢复出的尺度精度。

</div>
