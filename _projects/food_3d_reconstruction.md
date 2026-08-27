---
layout: page
title: Metric-Scale 3D Food Reconstruction for Nutrition Estimation
description: "<span class='lang-en'>A multi-view 3D reconstruction pipeline for tabletop food — volume MAPE 12.36%, improving on the company's single-view baseline of 25%.</span><span class='lang-zh'>面向桌面食物的多视角 3D 重建流水线——体积 MAPE 12.36%，优于公司单视角基线的 25%。</span>"
importance: 1
category: projects
---

<div class="lang-en" markdown="1">

**Computer Vision Intern · Zhijie Exploration Technology (深圳智界探索), Shenzhen · Summer 2026**

Computer vision work supporting a nutrition-estimation pipeline — a CV / bioinformatics crossover.

- **Dataset construction:**
  - Researched and validated construction schemes for food ground-truth volume datasets.
  - Built a small-sample dataset used to benchmark the company's single-view and multi-view food volume reconstruction algorithms on iOS and Android devices.
- **3D reconstruction pipeline:**
  - Researched, tested, and built a complete metric-scale, multi-view 3D reconstruction pipeline for tabletop food — essentially a sparse-feature, weakly-textured, small-to-medium-object reconstruction task.
  - Pipeline: VGGT-Omega multi-view point cloud reconstruction + container reconstruction.
  - Achieved **volume MAPE = 12.36%** on the test dataset, improving on the company's previous single-view pipeline (**MAPE ≈ 25%**).
- **iOS RGBD data-collection app:**
  - Added post-capture display of key quality metrics to the existing product — e.g., overall photo confidence score and server upload ID.
  - Made the data collection process more controllable and the collected data quality more consistent.

> Detailed write-up coming soon — this page will cover the pipeline design, experiments, and visual results in depth.

</div>

<div class="lang-zh" markdown="1">

**计算机视觉实习生 · 智界探索科技（深圳）· 2026 年夏**

为营养估计流水线提供计算机视觉支持——CV 与生物信息交叉的工作。

- **数据集构建：**
  - 研究并验证了食物体积真值数据集的构建方案。
  - 构建了小样本数据集，用于在 iOS 和 Android 设备上基准测试公司的单视角与多视角食物体积重建算法。
- **3D 重建流水线：**
  - 研究、测试并搭建了完整的 metric-scale 多视角 3D 重建流水线，面向桌面食物——本质是稀疏特征、弱纹理、中小型物体的重建任务。
  - 流水线：VGGT-Omega 多视角点云重建 + 容器重建。
  - 在测试集上达到**体积 MAPE = 12.36%**，优于公司此前的单视角流水线（**MAPE ≈ 25%**）。
- **iOS RGBD 数据采集应用：**
  - 在现有产品中新增了拍摄后关键质量指标的展示——如整体照片置信度分数和服务器上传 ID。
  - 使数据采集过程更可控，数据质量更稳定。

> 详细的技术文章即将发布——本页将深入介绍流水线设计、实验与可视化结果。

</div>
