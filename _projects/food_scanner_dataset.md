---
layout: page
title: Building a Metric-Scale Food Dataset with a 3D Scanner
description: "<span class='lang-en'>From scanner survey to a validated choice: a head-to-head between two consumer 3D scanners, then an automated capture pipeline producing a 90-dish ground-truth dataset.</span><span class='lang-zh'>从扫描仪调研到验证选型：两款消费级 3D 扫描仪的正面对比，再用自动化采集链路产出 90 个样本的真值数据集。</span>"
importance: 2
category: projects
---

<div class="lang-en" markdown="1">

**Computer Vision Intern · Zhijie Exploration Technology (深圳智界探索), Shenzhen · Summer 2026**

> This page covers the **ground-truth dataset**. The companion page [Metric-Scale 3D Food Reconstruction]({{ '/projects/food_3d_reconstruction/' | relative_url }}) covers the volume-estimation pipeline this dataset was built to validate.

**In one sentence:** the company needed trustworthy, true-scale food volumes as ground truth — I surveyed the scanner market, ran a head-to-head experiment between the two best-fit consumer scanners, and built an automated pipeline that turned the winner into a 90-sample dataset.

## Why a scanner at all

Estimating food volume from photos needs something to be validated **against** — and the company had no accurate, true-scale food point clouds to serve as ground truth. Following the approach of **MetaFood3D** (a reference food 3D dataset), I established that a consumer-grade 3D scanner could play this role, then set out to pick the right one.

## Survey: how depth sensors see

I surveyed the four main depth-sensing principles — **passive stereo, structured light, ToF, and dToF** — and, more importantly for us, the practical difference between **infrared and blue-light** scanning: published and measured volume errors run about **5% for infrared vs. ≤3% for blue light**.

To check the claims against reality, I ran a hands-on test at a Creality store: a calibration block with a true volume of 96 cm³ was scanned at 93.2 cm³ (**2.8% error**), and a sample model (true 100 cm³) came out at 94.6 cm³ with infrared (**5.5%**) — matching the advertised IR-vs-blue-light gap. Length measurements were accurate at the millimeter level.

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/food_dataset/owl_photo.jpg' | relative_url }}" alt="Owl figurine used in the store test" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">The test object: an owl figurine.</figcaption>
  </figure>
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/food_dataset/owl_bluelight.jpg' | relative_url }}" alt="Blue-light scan of the owl figurine" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">Its blue-light scan — fine feather texture preserved.</figcaption>
  </figure>
</div>

## Head-to-head: MIRACO PLUS vs. POP 4

The shortlist came down to two scanners: **MIRACO PLUS** and **POP 4**. I ran a controlled comparison on the same set of objects.

**Mesh detail.** On the same dragon fruit, POP 4 reconstructed **129,587 vertices** against MIRACO PLUS's **11,461** — an order-of-magnitude difference in surface fidelity:

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:280px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/food_dataset/mesh_pop4.png' | relative_url }}" alt="POP 4 mesh of a dragon fruit, 129587 vertices" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">POP 4 — 129,587 vertices (highlighted bottom-left).</figcaption>
  </figure>
  <figure style="flex:1; min-width:280px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/food_dataset/mesh_miraco.png' | relative_url }}" alt="MIRACO PLUS mesh of the same dragon fruit, 11461 vertices" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">MIRACO PLUS — 11,461 vertices, same object.</figcaption>
  </figure>
</div>

**Volume accuracy.** Over repeated scans with known true volumes, POP 4 was both less biased and far more consistent: MIRACO PLUS averaged **+0.80%** error over 48 scans (95% CI [−0.46, 1.97], std 4.37), while POP 4 averaged **−0.09%** over 21 scans (95% CI [−0.77, 0.67], std 1.72; worst case −2.09% / +4.29%).

<img src="{{ '/assets/img/projects/food_dataset/error_boxplot.png' | relative_url }}" alt="Volume error spread: MIRACO PLUS vs POP 4" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Every dot is one scan. POP 4 (right, unplated objects) clusters tightly around zero; MIRACO PLUS spreads from −13% to +10%.</p>

**Practical factors** sealed the decision: POP 4 scans a single object in **2–3 minutes** (MIRACO needs at least 5), costs about **a third** as much (~¥6.3k vs. ~¥17.9k), and handles **dark and reflective** foods better. **Decision: POP 4.**

## The trade-offs we accepted

No choice is free, and this one came with two known costs, which I quantified rather than hand-waved:

- **Watertightness dropped slightly** — trimesh reports the POP 4 meshes as non-watertight, though this does not affect volume computation for our purposes.
- **Plated scans occasionally ghost** (double images from food shifting between passes) and need a rescan; and plated items carry a small **systematic underestimate of about −3.2%**, visible when splitting the stats by condition:

<img src="{{ '/assets/img/projects/food_dataset/pop4_plated_table.png' | relative_url }}" alt="POP 4 statistics split by plating condition" style="width:100%; border-radius:8px;">

<img src="{{ '/assets/img/projects/food_dataset/plated_mesh.png' | relative_url }}" alt="POP 4 scan of a plated dish" style="width:70%; display:block; margin:0 auto; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">A plated dish scanned with POP 4 — food and container captured together, which is exactly the geometry the volume pipeline must reason about.</p>

## Automating capture: a three-script chain

Hand-driving the scanner software for ~100 objects does not scale, so I wrapped the whole workflow in **three automation scripts**, with every step logged to a `manifest.csv`:

<img src="{{ '/assets/img/projects/food_dataset/automation_flow.png' | relative_url }}" alt="Automated capture pipeline: scan, denoise, fuse" style="width:100%; border-radius:8px;">

1. **Script 1 — GUI auto-scan**: drives the Revo Scan interface to capture each object into `raw/`.
2. **Script 0 — auto denoise**: RANSAC plane fitting plus statistical outlier removal, with a large model as the fallback judge for ambiguous cases, writing to `processed/`.
3. **Script 2 — GUI auto-fuse**: fuses the cleaned point clouds into meshes in `fused/`.

## The dataset

The pipeline produced the final ground-truth dataset:

- **70 tabletop single food items** (simple geometry, unplated)
- **20 plated high-oil / high-sugar dishes** — the hard, realistic case

This dataset became the ground truth for validating the five-view volume-estimation pipeline — see the [companion page]({{ '/projects/food_3d_reconstruction/' | relative_url }}) for how the 20 plated dishes were used to measure its accuracy (final MAPE 12.36%).

## Limitations

- The **−3.2% plated bias** is characterized but not eliminated — the container's reflective surface and food shifting are the suspected causes.
- Meshes are **not watertight**; fine for volume ground truth, but they would need repair for other uses.
- Plated scans with ghosting must be **detected and redone manually** — the automation chain does not yet catch them on its own.

</div>

<div class="lang-zh" markdown="1">

**计算机视觉实习生 · 智界探索科技（深圳）· 2026 年夏**

> 本页讲**真值数据集**。配套页面[真实尺度 3D 食物重建]({{ '/projects/food_3d_reconstruction/' | relative_url }})讲这个数据集所要验证的体积估算流水线。

**一句话概括：** 公司需要可信的、真实尺度的食物体积作为真值——我调研了扫描仪市场，对两款最符合需求的消费级扫描仪做了正面对比实验，并为胜出者搭建了自动化采集链路，产出 90 个样本的数据集。

## 为什么需要扫描仪

从照片估计食物体积，总得有个东西可以拿来**对答案**——而公司手里没有精确的、真实尺度的食物点云可以当真值。参照 **MetaFood3D**（一个参考性的食物 3D 数据集）的建库方式，我论证了消费级 3D 扫描仪可以承担这个角色，接下来就是挑一台合适的。

## 调研：深度传感器怎么"看"东西

我调研了四大深度感知原理——**被动双目、结构光、ToF、dToF**——以及对我们更关键的实际差异：**红外 vs 蓝光**扫描：公开资料与实测的体积误差大约是**红外 5% vs 蓝光 ≤3%**。

为了验证这些说法，我在创想三维门店做了实测：一个真值 96 cm³ 的标定积木扫出 93.2 cm³（**误差 2.8%**），一个样例模型（真值 100 cm³）用红外扫出 94.6 cm³（**5.5%**）——与宣传的"红外 vs 蓝光"差距吻合。长度测量达到毫米级精度。

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/food_dataset/owl_photo.jpg' | relative_url }}" alt="门店实测用的猫头鹰摆件" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">测试对象：猫头鹰摆件。</figcaption>
  </figure>
  <figure style="flex:1; min-width:220px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/food_dataset/owl_bluelight.jpg' | relative_url }}" alt="猫头鹰摆件的蓝光扫描" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">蓝光扫描结果——羽毛细节保留完好。</figcaption>
  </figure>
</div>

## 正面对比：MIRACO PLUS vs. POP 4

候选最后落在两款扫描仪上：**MIRACO PLUS** 和 **POP 4**。我用同一批物体做了对照实验。

**网格精细度。** 同一个火龙果，POP 4 重建出 **129,587 个顶点**，MIRACO PLUS 只有 **11,461 个**——表面保真度差了一个数量级：

<div style="display:flex; gap:1.5%; flex-wrap:wrap;">
  <figure style="flex:1; min-width:280px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/food_dataset/mesh_pop4.png' | relative_url }}" alt="POP 4 的火龙果网格，129587 顶点" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">POP 4——129,587 顶点（左下角红框）。</figcaption>
  </figure>
  <figure style="flex:1; min-width:280px; margin:0 0 8px 0;">
    <img src="{{ '/assets/img/projects/food_dataset/mesh_miraco.png' | relative_url }}" alt="MIRACO PLUS 的同一火龙果网格，11461 顶点" style="width:100%; border-radius:8px;">
    <figcaption style="text-align:center; color:#666; font-size:0.85em;">MIRACO PLUS——11,461 顶点，同一物体。</figcaption>
  </figure>
</div>

**体积精度。** 在已知真值的重复扫描中，POP 4 偏差更小、稳定性也好得多：MIRACO PLUS 48 次扫描平均误差 **+0.80%**（95% CI [−0.46, 1.97]，标准差 4.37）；POP 4 21 次扫描平均 **−0.09%**（95% CI [−0.77, 0.67]，标准差 1.72，最差 −2.09% / +4.29%）。

<img src="{{ '/assets/img/projects/food_dataset/error_boxplot.png' | relative_url }}" alt="体积误差分布：MIRACO PLUS vs POP 4" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">每个点是一次扫描。POP 4（右，不装盘物体）紧紧收在零附近；MIRACO PLUS 则从 −13% 散布到 +10%。</p>

**实用因素**最终拍板：POP 4 单扫一个物体只需 **2–3 分钟**（MIRACO 至少 5 分钟），价格约为**三分之一**（~¥6.3k vs ~¥17.9k），对**深色、反光**食物的适应性也更好。**结论：选 POP 4。**

## 我们接受的代价

任何选择都有成本，这一个有两项——我选择量化它们而不是含糊带过：

- **水密性略降**——trimesh 报告 POP 4 的网格未闭合，但对我们的体积计算用途没有影响。
- **装盘扫描偶尔出现重影**（两次扫描之间食物挪动导致），需要重扫；并且装盘样本存在约 **−3.2% 的系统性低估**，按条件拆开统计就能看到：

<img src="{{ '/assets/img/projects/food_dataset/pop4_plated_table.png' | relative_url }}" alt="POP 4 按装盘条件拆分的统计" style="width:100%; border-radius:8px;">

<img src="{{ '/assets/img/projects/food_dataset/plated_mesh.png' | relative_url }}" alt="POP 4 扫描的装盘食物" style="width:70%; display:block; margin:0 auto; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">POP 4 扫描的装盘食物——食物与容器一起入模，这正是体积流水线需要推理的几何形态。</p>

## 自动化采集：三脚本链路

手动开扫描软件处理上百个物体是行不通的，于是我把整个流程包进**三个自动化脚本**，每一步都写入 `manifest.csv` 留痕：

<img src="{{ '/assets/img/projects/food_dataset/automation_flow.png' | relative_url }}" alt="自动化采集流水线：扫描、去噪、融合" style="width:100%; border-radius:8px;">

1. **脚本 1——GUI 自动扫描**：驱动 Revo Scan 界面逐物体采集，存入 `raw/`。
2. **脚本 0——自动去噪**：RANSAC 平面拟合加统计离群点剔除，模糊个案由大模型兜底判断，输出到 `processed/`。
3. **脚本 2——GUI 自动融合**：把干净的点云融合成网格，存入 `fused/`。

## 数据集成果

这条链路产出了最终的真值数据集：

- **70 个桌面单体简单食物**（几何简单、不装盘）
- **20 个带盘高油高糖食物**——困难但真实的场景

这个数据集成为五视角体积估算流水线的验证真值——20 个装盘样本如何用来测量流水线精度（最终 MAPE 12.36%），见[配套页面]({{ '/projects/food_3d_reconstruction/' | relative_url }})。

## 局限

- **−3.2% 的装盘偏差**已被刻画但尚未消除——容器表面反光和食物挪动是可疑原因。
- 网格**非水密**；做体积真值没问题，但其他用途需要先修复。
- 出现重影的装盘扫描仍需**人工发现并重扫**——自动化链路还不能自己识别。

</div>
