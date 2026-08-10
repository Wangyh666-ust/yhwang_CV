---
layout: page
title: Metric-Scale 3D Food Reconstruction for Nutrition Estimation
description: A multi-view 3D reconstruction pipeline for tabletop food — volume MAPE 19.5%, improving on the company's single-view baseline of 25%.
importance: 1
category: research
---

**Computer Vision Intern · Zhijie Exploration Technology (深圳智界探索), Shenzhen · Summer 2026**

Computer vision work supporting a nutrition-estimation pipeline — a CV / bioinformatics crossover.

- **Dataset construction:**
  - Researched and validated construction schemes for food ground-truth volume datasets.
  - Built a small-sample dataset used to benchmark the company's single-view and multi-view food volume reconstruction algorithms on iOS and Android devices.
- **3D reconstruction pipeline:**
  - Researched, tested, and built a complete metric-scale, multi-view 3D reconstruction pipeline for tabletop food — essentially a sparse-feature, weakly-textured, small-to-medium-object reconstruction task.
  - Pipeline: VGGT-Omega multi-view point cloud reconstruction + container reconstruction.
  - Achieved **volume MAPE = 19.5%** on the test dataset, improving on the company's previous single-view pipeline (**MAPE ≈ 25%**).
- **iOS RGBD data-collection app:**
  - Added post-capture display of key quality metrics to the existing product — e.g., overall photo confidence score and server upload ID.
  - Made the data collection process more controllable and the collected data quality more consistent.

> Detailed write-up coming soon — this page will cover the pipeline design, experiments, and visual results in depth.
