---
layout: page
title: Metric-Scale 3D Food Reconstruction for Nutrition Estimation
description: "<span class='lang-en'>An end-to-end pipeline that turns five RGB-D views of a plated dish into a metric volume estimate — MAPE 12.36% against scanner ground truth, beating the company's single-view baseline of ~25%.</span><span class='lang-zh'>端到端流水线：输入装盘食物的五视角 RGB-D，输出真实尺度体积——对扫描仪真值 MAPE 12.36%，优于公司单视角基线的约 25%。</span>"
importance: 1
category: projects
toc:
  sidebar: left
---

<div class="lang-en" markdown="1">

**Computer Vision Intern · Zhijie Exploration Technology (深圳智界探索), Shenzhen · Summer 2026**

> This page covers the **volume-estimation pipeline**. The companion page [Building a Metric-Scale Food Dataset with a 3D Scanner]({{ '/projects/food_scanner_dataset/' | relative_url }}) covers how the ground-truth dataset used to validate it was built.

**In one sentence:** given five RGB-D views of a plated dish, this pipeline outputs the food's volume in cubic centimeters — end to end, no human in the loop.

## Background: nutrition estimation needs a volume

The company maintains **CN5K**, a dataset of plated Chinese dishes, each captured from **five fixed views** (front-left, front-right, back-left, back-right, top-down). Every view provides an RGB image, a raw depth map, a confidence map, and camera intrinsics.

The nutrition pipeline works as **volume × density → weight → nutrition**. Density comes from food-category priors; the missing piece was an accurate, automated way to get the **volume** of the food on a plate. That was my task: five images in, volume out.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/input_grid.jpg' | relative_url }}" alt="Five-view capture setup: grid of masked food photos" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">What the input looks like: a grid of five-view captures (validation set with fiducial dice markers; red overlay = food mask). Each capture is RGB + depth + confidence + intrinsics per view.</p>

## Choosing the reconstruction front-end

I benchmarked the recent feed-forward 3D reconstruction models on our data — **VGGT-Omega, Pi3X, MASt3R, MapAnything, DA3** — fusing their per-view depth with TSDF and filtering to the support region (connected components within 15 mm of the table plane center).

The result was a classic accuracy-vs-bias trade-off:

- On **single food items**, VGGT-Omega was the most accurate (MAPE 8.32% after TSDF fusion and support filtering).
- On **plated dishes** — the real use case — the picture changed. MASt3R was systematically conservative (footprint too small), MapAnything mis-estimated heights despite a low MAPE on paper (10.2%) and produced poor-quality point clouds, and DA3 suffered from misalignment. After aligning the reconstructions with **RoMa**, **Pi3X was essentially unbiased** with the cleanest geometry.

**Final choice: Pi3X + RoMa** — not the lowest raw error, but the most trustworthy geometry for downstream container reasoning.

## Pipeline

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/pipeline.png' | relative_url }}" alt="End-to-end pipeline: five-view RGB-D to volume" style="width:100%; border-radius:8px;">

The numbered stages correspond to:

1. **VLM gate**: a vision-language model checks the container type, food type, and their spatial relationship — unsuitable cases are rejected outright rather than silently producing garbage.
2. **SAM 3 segmentation**: food / container / table masks, guided by the VLM's bounding boxes with an IoU cross-check.
3. **Pi3X joint inference**: all five views are reconstructed in one shared coordinate frame — point maps, camera poses, and per-point confidence.
4. **Scale recovery**: raw metric depth is scaled by an empirical factor (×1.09, see below), then low-confidence regions are fused by taking the per-region q50 depth and aggregating across views with a five-view median.
5. **Container type**: the VLM distinguishes plate vs. bowl, which determines the inner-bottom height prior (plates: fixed ~13 mm; bowls: a linear function of rim diameter, ~14±4 mm).
6. **Rim fitting**: the container rim is fit with a **superellipse** (circle / ellipse / rounded rectangle) under a battery of priors — height bands, rim width, ring constraints — and the side walls are interpolated (xy linear, z quadratic) to complete the hidden inner surface.
7. **Height-field integration**: food surface minus container support surface, integrated on a 2 mm grid.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/container_priors.png' | relative_url }}" alt="Bowl cross-section priors: visible rim, inner wall, inferred inner bottom" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">The container priors in one cross-section: the visible rim (inner edge R), the inner wall (q² curve), the inferred inner-bottom edge B, the hidden support surface, and the table plane (Z = 0). The food volume is what sits between the food surface and this reconstructed support surface.</p>

**How well does the container reconstruction work?** Against POP 4 scanner ground truth, the reconstructed plates and bowls line up closely (blue = VGGT + RoMa, orange = Pi3X + RoMa, with per-container MAE):

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/container_vs_gt.jpg' | relative_url }}" alt="Reconstructed containers vs POP 4 scanner ground truth, six plates and bowls" style="width:60%; display:block; margin:0 auto; border-radius:8px;">

## Case studies

**How to read these panels:** each case shows, left to right, the representative overhead RGB → the VLM + SAM 3 semantic overlay (food in orange, container in blue) → the final 2 mm height grid from which the volume is integrated.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/case_shrimp.jpg' | relative_url }}" alt="Case study: braised shrimp — RGB, semantic overlay, height grid" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Braised shrimp on a plate: estimated volume 425.97 cm³. The height grid correctly captures the uneven piling of the shrimp.</p>

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/case_rice.jpg' | relative_url }}" alt="Case study: rice bowl — RGB, semantic overlay, height grid" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">A bowl of rice: estimated volume 472.27 cm³ — see the density sanity check below for why this number is believable.</p>

## The ×1.09 correction — honestly

The phone's **raw metric depth systematically underestimates**. I swept a global multiplier from 1.05 to 1.11 over the 20-case validation set (split into first-10 / last-10 to check stability):

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/multiplier_sweep.png' | relative_url }}" alt="Sweep of the raw-depth multiplier from 1.05 to 1.11, MAPE per split" style="width:100%; border-radius:8px;">

MAPE bottoms out around **×1.08–1.09** and the minimum is consistent across both halves of the data, so I adopted **×1.09**. Two honest caveats: this factor carries an **overfitting risk** (it is fit on 20 samples), and the remaining error concentrates in **small volumes**, which are still underestimated.

## Gating: not every case deserves an estimate

Two gates keep unreliable data out of the pipeline: a **data gate** on tags and depth quality, and the **VLM gate**, which actively refused 40% of the cases it was shown — a rejected case is far cheaper than a wrong volume.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/gating_funnel.png' | relative_url }}" alt="Gating funnel: 5859 cases to 348 valid volumes" style="width:100%; border-radius:8px;">

## Validation against scanner ground truth

The final pipeline was validated on **20 plated dishes scanned with the POP 4** (see the [companion dataset page]({{ '/projects/food_scanner_dataset/' | relative_url }})):

- **Final pipeline: volume MAPE 12.36%**
- Without the ×1.09 depth correction: MAPE 25.77%
- Company single-view baseline for reference: MAPE ≈ 20–25%

## Sanity check via density

Volume alone is hard to eyeball, so I cross-checked it against weight: for ten rice sessions, the implied bulk density averages **0.714 g/cm³** (pooled 0.721) — comfortably inside the plausible range for cooked rice (~0.6–1.06 g/cm³). The volumes are not just self-consistent; they make physical sense.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/rice_density.png' | relative_url }}" alt="Implied bulk density of rice across ten sessions" style="width:70%; display:block; margin:0 auto; border-radius:8px;">

## A bad case, shown honestly

**Guilinggao (tortoise jelly) in a straight-walled bowl.** The food is dark, glossy, and flush with the container; the bowl walls are vertical, so the rim priors have little to grab. The semantic overlay and height grid look plausible, but this category is exactly where the container assumptions bend.

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:280px; margin:0 0 8px 0;">
    <img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/badcase_panel.jpg' | relative_url }}" alt="Bad case panel: guilinggao" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">The case panel: dark, glossy food flush with the bowl.</figcaption>
  </figure>
  <figure style="flex:1; min-width:280px; margin:0 0 8px 0;">
    <img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/badcase_bowl_pc.jpg' | relative_url }}" alt="Bad case point cloud: straight-walled bowl" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">The reconstructed bowl point cloud, side view — visible layering near the rim.</figcaption>
  </figure>
</div>

## Limitations and next steps

- **The systematic depth underestimation** is corrected empirically but not yet explained — the root cause (sensor calibration? material response?) is still open.
- **Mid-to-small volumes (100–350 cm³) are under-sampled** in the validation set, and the residual error concentrates there.
- **Container assumptions cover only bowls and plates**; more categories (cups, boxes, irregular containers) need new priors.
- **Filament-like foods** (noodles, shredded dishes) are smoothed and simplified by Pi3X, losing fine structure.

</div>

<div class="lang-zh" markdown="1">

**计算机视觉实习生 · 智界探索科技（深圳）· 2026 年夏**

> 本页讲**体积估算流水线**。配套页面[用 3D 扫描仪构建真实尺度食物数据集]({{ '/projects/food_scanner_dataset/' | relative_url }})讲验证所用的真值数据集是怎么建出来的。

**一句话概括：** 输入一盘菜的五视角 RGB-D，端到端输出这份食物的体积（立方厘米），全程无需人工干预。

## 背景：营养估计缺一个"体积"

公司维护着 **CN5K** 数据集：装盘中式菜品，每份从**五个固定视角**（前左、前右、后左、后右、俯视）拍摄。每个视角都提供 RGB 图像、原始深度图、置信度图和相机内参。

营养估计的链路是**体积 × 密度 → 重量 → 营养**。密度由食物类别先验给出，缺的正是一个准确、自动获取**盘中食物体积**的方法。这就是我的任务：五张图进，体积出。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/input_grid.jpg' | relative_url }}" alt="五视角采集：带 mask 的食物照片网格" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">输入数据长这样：五视角采集网格（验证集带骰子标志点；红色叠加 = 食物 mask）。每次采集每个视角都有 RGB + 深度 + 置信度 + 内参。</p>

## 前端重建模型选型

我在公司数据上基准测试了近期的一批前馈式 3D 重建模型——**VGGT-Omega、Pi3X、MASt3R、MapAnything、DA3**——用 TSDF 融合各视角深度，并筛选支撑区域（距桌面平面中心 15 mm 内的连通组件）。

结果是典型的"精度—偏差"权衡：

- 在**单体食物**上，VGGT-Omega 最准（TSDF 融合 + 支撑筛选后 MAPE 8.32%）。
- 但在**装盘食物**——也就是真实使用场景——情况变了：MASt3R 系统性偏保守（footprint 偏小）；MapAnything 纸面 MAPE 虽低（10.2%）但高度估计有问题、点云质量差；DA3 存在配准错位。而在用 **RoMa** 做对齐之后，**Pi3X 基本无偏**，几何也最干净。

**最终选择：Pi3X + RoMa**——不是纸面误差最低的，但几何最可信，能支撑下游的容器推理。

## 流水线

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/pipeline.png' | relative_url }}" alt="端到端流水线：五视角 RGB-D 到体积" style="width:100%; border-radius:8px;">

图中编号对应以下步骤：

1. **VLM 门控**：视觉语言模型检查容器类型、食物类型及两者的摆放关系——不合格的样本直接拒识，而不是默默输出垃圾结果。
2. **SAM 3 分割**：在 VLM 给出的 bbox 引导下分割食物 / 容器 / 桌面，并用 IoU 交叉校验。
3. **Pi3X 联合推理**：五个视角在同一共享坐标系下重建——point maps、相机位姿、逐点置信度。
4. **尺度恢复**：原始公制深度先乘经验系数（×1.09，见下文），低置信区域取区域 q50 深度，再跨视角取五视角中位数完成融合。
5. **容器类型判断**：VLM 区分盘 / 碗，由此确定内底高度先验（盘：固定约 13 mm；碗：与口径线性相关，约 14±4 mm）。
6. **沿口拟合**：容器沿口用**超椭圆**（圆 / 椭圆 / 圆角矩形）拟合，配合一系列先验（高度带、沿口宽度、环带约束），侧壁用插值补全（xy 线性、z 二次），还原出被食物遮住的容器内表面。
7. **高度场积分**：食物表面减去容器支撑面，在 2 mm 网格上积分得到体积。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/container_priors.png' | relative_url }}" alt="碗的剖面先验：可见上口、内壁、推断内底" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">一张剖面图看懂容器先验：可见上口（内沿 R）、内壁（q² 曲线）、推断的内底边沿 B、隐藏的支撑面、桌面（Z = 0）。食物体积就是食物表面与这个重建出的支撑面之间的部分。</p>

**容器重建的效果如何？** 与 POP 4 扫描真值对比，重建出的盘和碗贴合得很好（蓝 = VGGT + RoMa，橙 = Pi3X + RoMa，附各容器 MAE）：

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/container_vs_gt.jpg' | relative_url }}" alt="容器重建 vs POP 4 扫描真值，六组盘碗" style="width:60%; display:block; margin:0 auto; border-radius:8px;">

## 案例研究

**这些面板怎么看：** 每个案例从左到右依次是：代表性俯视 RGB → VLM + SAM 3 语义叠加图（橙色 = 食物，蓝色 = 容器）→ 用于积分体积的 2 mm 高度场网格。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/case_shrimp.jpg' | relative_url }}" alt="案例：油焖大虾——RGB、语义叠加、高度场" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">油焖大虾（装盘）：估计体积 425.97 cm³。高度场正确刻画了虾堆叠的起伏。</p>

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/case_rice.jpg' | relative_url }}" alt="案例：米饭——RGB、语义叠加、高度场" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">一碗米饭：估计体积 472.27 cm³——这个数字为什么可信，见下面的密度合理性检查。</p>

## ×1.09 修正——如实地讲

手机的**原始公制深度存在系统性低估**。我在 20 个验证样本上对全局乘子从 1.05 到 1.11 做了扫描（按先后拆成前 10 / 后 10 两组检验稳定性）：

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/multiplier_sweep.png' | relative_url }}" alt="深度乘子 1.05–1.11 扫描，各组 MAPE" style="width:100%; border-radius:8px;">

MAPE 在 **×1.08–1.09** 处见底，且最小值在两半数据上一致，因此采用 **×1.09**。两个诚实的提醒：这个系数有**过拟合风险**（只在 20 个样本上拟合）；残余误差集中在**小体积**区间，那里仍然低估。

## 门控：不是每个样本都值得估

两道门控把不可靠数据挡在流水线之外：按标签与深度质量的**数据门控**，以及 **VLM 门控**——后者主动拒绝了 40% 的送检样本。拒识一个样本的代价远小于输出一个错误体积。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/gating_funnel.png' | relative_url }}" alt="门控漏斗：5859 个 case 到 348 个有效体积" style="width:100%; border-radius:8px;">

## 用扫描仪真值做验证

最终流水线在 **20 个经 POP 4 扫描的装盘食物**上验证（见[配套数据集页面]({{ '/projects/food_scanner_dataset/' | relative_url }})）：

- **最终流水线：体积 MAPE 12.36%**
- 不做 ×1.09 深度修正：MAPE 25.77%
- 参照：公司单视角基线 MAPE ≈ 20–25%

## 用密度做合理性检查

单看体积数字很难直觉判断对错，于是我用重量交叉验证：十个米饭 session 推算出的堆积密度均值为 **0.714 g/cm³**（总体重除以总体积为 0.721）——落在熟米饭的合理区间（约 0.6–1.06 g/cm³）内。体积不仅自洽，物理上也讲得通。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/rice_density.png' | relative_url }}" alt="十个 session 的米饭堆积密度" style="width:70%; display:block; margin:0 auto; border-radius:8px;">

## 如实展示一个 bad case

**直上直下碗里的龟苓膏。** 食物颜色深、表面反光、与容器齐平；碗壁直上直下，沿口先验无从着力。语义叠加图和高度场看上去都合理，但这一类样本正是容器假设最容易弯折的地方。

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:280px; margin:0 0 8px 0;">
    <img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/badcase_panel.jpg' | relative_url }}" alt="bad case 面板：龟苓膏" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">案例面板：深色、反光、与碗齐平的食物。</figcaption>
  </figure>
  <figure style="flex:1; min-width:280px; margin:0 0 8px 0;">
    <img loading="lazy" decoding="async" src="{{ '/assets/img/projects/food_volume/badcase_bowl_pc.jpg' | relative_url }}" alt="bad case 点云：直上直下的碗" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">重建碗点云侧视——沿口附近可见分层。</figcaption>
  </figure>
</div>

## 局限与下一步

- **深度系统性低估**目前是经验性修正，尚未找到根因（传感器标定？材质响应？）。
- 验证集中**中小体积样本（100–350 cm³）过少**，而残余误差正集中在这个区间。
- **容器假设只覆盖碗和盘**；杯子、餐盒、异形容器需要新的先验。
- **丝状食物**（面条、切丝类菜品）会被 Pi3X 平滑简化，丢失细节结构。

</div>
