---
layout: page
title: Medical Image Segmentation with U-Net
description: "<span class='lang-en'>Automatic heart detection, location prediction, and segmentation in X-ray images — trained entirely on self-built synthetic data; 97% accuracy, 0.7992 F1 on real hand-labeled X-rays.</span><span class='lang-zh'>X 光图像中的自动心脏检测、位置预测与分割——完全用自建合成数据训练；在真实人工标注 X 光上达 97% 准确率、0.7992 F1 分数。</span>"
importance: 2
category: projects
toc:
  sidebar: left
---

<div class="lang-en" markdown="1">

**Research Exchange Student · Seoul National University · HKUST–SNU SPIA Program · Summer 2025 (49 days)**

**In one sentence:** with medical X-ray data almost impossible to obtain, we **manufactured our own training data** — assembling a human torso in Blender and simulating X-ray imaging — then trained a U-Net that segments the heart in **real** chest X-rays it had never seen, reaching **~97% accuracy and 0.7992 F1** against hand-labeled masks.

> The exchange project was scoped for **4 people**; in the end **2 of us** carried it, and in 49 days we completed the first two of the three planned stages. What the third stage would have been — and why we stopped — is at the end of this page.

## The goal: from one X-ray to the heart's 3D pose

A heart's shape and orientation carry diagnostic information — a thickened left ventricle (hypertrophy), for example, shows up as an abnormally shaped silhouette on a chest X-ray. The program's research question was ambitious: **from a single chest X-ray, determine whether the heart's shape, position, and axial orientation are normal** — which ultimately means reasoning about 3D structure from a 2D projection.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/xray_features.jpg' | relative_url }}" alt="Typical chest X-ray features: left ventricle hypertrophy and shape variety" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Why this is hard even for humans: hearts and lungs vary widely across patients (right), and pathologies like left-ventricle hypertrophy change the silhouette (left).</p>

## The three-stage plan

1. **Make the data.** Real annotated medical data is scarce and confidential — so we built a synthetic factory: assemble a human torso from **2,746 anatomical OBJ models** in Blender, simulate the X-ray imaging physics, and get unlimited images **with perfect ground-truth masks for free**.
2. **Learn 2D segmentation.** Train a U-Net on the synthetic chest X-rays, then test on **real, hand-labeled** chest X-rays: does the model correctly locate the heart?
3. **Lift to 3D (not reached).** Extend the 2D position and axis information back into 3D body space — one X-ray in, a verdict on the heart's 3D shape/position/orientation out.

## Stage 1 — a synthetic X-ray factory

Two problems had to be solved: X-ray imaging is a physics process (not visible light), and one fixed body would give one fixed image.

**Simulating the physics.** We assembled the torso in Blender and drove **gvxr (GVXR)** — a library that simulates a monochromatic X-ray source and detector — to render physically plausible radiographs through the bone/tissue model.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/blender_gvxr.jpg' | relative_url }}" alt="Blender torso model and the gvxr simulation scene" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Left: the assembled torso in Blender. Right: the gvxr scene — source, torso, and detector plane with a live radiograph.</p>

**Making every image different.** We automated two Blender deformations so each generated sample is a new "patient": **armatures** change the body pose, **lattice deformations** randomly reshape the heart and lungs.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/deformation.jpg' | relative_url }}" alt="Armature and lattice deformations in Blender" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Armature (left) re-poses the skeleton; lattice cages (right) deform the organs themselves.</p>

**Closing the realism gap.** Raw simulated images look too clean, so we passed them through **CycleGAN** to adopt the texture of real X-rays, and applied **CLAHE** histogram augmentation — real X-rays come in wildly different brightness and contrast, and the model must not learn just one shade.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/synth_cyclegan.jpg' | relative_url }}" alt="Raw gvxr simulation vs CycleGAN-enhanced result" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Left: raw gvxr output. Right: after CycleGAN — noticeably closer to a real radiograph's texture.</p>

## Stage 2 — U-Net from synthetic to real

We built the segmentation model after the original U-Net paper (Ronneberger et al.): a downsampling path captures features, an upsampling path regenerates the image at full resolution, and skip connections preserve fine detail.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/unet_arch.jpg' | relative_url }}" alt="U-Net architecture: downsampling, upsampling, skip connections" style="width:100%; border-radius:8px;">

Three training details mattered:

- **Combined loss (Dice + BCE).** Dice loss separates regions well but is weak at edges; BCE supplies the edge detail.
- **Dynamic loss weighting**, adjusted automatically from each term's contribution during training.
- **Automatic threshold selection**: 20 candidate thresholds are sampled uniformly, 5 are picked at random and evaluated each run, and the best is kept — so training can run unattended on the server and keep the best checkpoint.

**A proxy exam first.** No public dataset labels *hearts* in X-rays — so before trusting the heart model, we validated the exact same training setup on **lung** segmentation, where labels do exist: **accuracy 0.918, F1 0.817**.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/lung_seg_test.jpg' | relative_url }}" alt="Lung segmentation proxy test: confusion matrix and metrics" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">The pipeline's "mock exam" on lung segmentation, where ground-truth labels exist.</p>

**A real exam needs a real answer sheet.** For the heart there is no labeled data at all — without a metric, automated training can't know which checkpoint is best. So we **hand-drew accurate heart masks** (guided by 3D anatomy references) to build a small real test set.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/manual_mask.jpg' | relative_url }}" alt="A real chest X-ray and its hand-drawn heart mask" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">A real chest X-ray (left) and its hand-drawn ground-truth heart mask (right).</p>

## Results on real X-rays

The model — which had only ever trained on synthetic data — was evaluated on real chest X-rays against our hand-drawn masks:

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/predictions.jpg' | relative_url }}" alt="Predictions on real chest X-rays: original, probability map, post-processing, overlay" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Left to right: original real X-ray → raw probability map → after post-processing (stray false-positive blobs removed) → segmentation overlay. The segmented region sits exactly where the heart is.</p>

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/metrics_report.jpg' | relative_url }}" alt="Training summary: metric trends, best vs worst run, F1 distribution" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">30 training iterations, 100% completed successfully. Best checkpoint (iteration 17): <strong>accuracy ≈ 0.97, F1 = 0.7992</strong>, precision 0.84, IoU 0.67.</p>

**Honest caveats:** the heart metrics are measured against masks we drew ourselves — a small, self-labeled test set, not clinical ground truth. And the model was tested on the same X-ray style it saw during CycleGAN adaptation; truly out-of-distribution hospital scans remain an open question.

## What stage 3 would have taken

The final stage — lifting the 2D detection into a 3D estimate of heart position, axis, and shape for abnormality diagnosis — was designed but not built: 49 days and 2 people were enough for stages 1–2 only. What it would need next:

- **Real annotated data** — the single biggest blocker; everything here routes around its absence.
- More body diversity in the factory (a female body model, more tissue and bone variety, hearts at different locations).
- The 3D lifting module itself: from a 2D mask + axis to a 3D pose estimate.

</div>

<div class="lang-zh" markdown="1">

**科研交换生 · 首尔国立大学 · 港科大–首尔大 SPIA 交换项目 · 2025 年夏（49 天）**

**一句话概括：** 医疗 X 光数据几乎无法获得，于是我们**自己制造训练数据**——在 Blender 里拼装人体躯干、模拟 X 光成像物理——再训练 U-Net，让它在**从未见过的真实**胸腔 X 光中分割心脏，对人工标注的掩膜达到 **约 97% 准确率、0.7992 F1 分数**。

> 这个交换项目原本按 **4 人**规模规划，最后实际由 **2 人**完成；49 天内我们完成了原计划三个阶段中的前两个。第三阶段是什么、为什么停下来，见本页末尾。

## 目标：从一张 X 光到心脏的 3D 位姿

心脏的形状与朝向携带诊断信息——例如左心室增厚（肥大）会在胸片上表现为异常轮廓。项目的研究问题相当有野心：**从一张胸腔 X 光，判断心脏的形状、位置、轴向是否正常**——这本质上是从 2D 投影推理 3D 结构。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/xray_features.jpg' | relative_url }}" alt="典型胸片特征：左心室肥大与形态多样性" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">这件事对人类都难：不同病人的心肺形态差异很大（右），左心室肥大等病变会改变轮廓（左）。</p>

## 三阶段计划

1. **造数据。** 真实标注的医疗数据稀缺且涉密——所以我们搭了一座合成工厂：在 Blender 里用 **2746 个人体解剖 OBJ 模型**拼装躯干，模拟 X 光成像物理，得到无限多的图像，并且**免费附带完美的真值掩膜**。
2. **学 2D 分割。** 在合成胸片上训练 U-Net，然后在**真实、人工标注**的胸片上测试：模型能正确定位心脏吗？
3. **升到 3D（未完成）。** 把 2D 的位置与轴向信息反推回三维人体空间——一张 X 光输入，输出心脏 3D 形状/位置/朝向是否正常的判断。

## 第一阶段——合成 X 光工厂

要解决两个问题：X 光成像是物理过程（不是可见光渲染）；而且一个固定的身体只能生成一张固定的图。

**模拟物理。** 我们在 Blender 中拼装躯干，并接入 **gvxr（GVXR）**——一个模拟单能 X 光源与探测器的库——对骨骼/组织模型渲染出物理上可信的 X 光片。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/blender_gvxr.jpg' | relative_url }}" alt="Blender 躯干模型与 gvxr 仿真场景" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">左：Blender 中拼装的躯干。右：gvxr 场景——射线源、躯干、探测器平面与实时成像。</p>

**让每张图都不一样。** 我们自动化了两种 Blender 形变，让每个样本都是一个新"病人"：**骨骼绑定（armature）**改变体态，**晶格形变（lattice）**随机改变心脏与肺的形状。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/deformation.jpg' | relative_url }}" alt="Blender 中的骨骼绑定与晶格形变" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">骨骼绑定（左）改变骨架姿态；晶格笼（右）直接让器官变形。</p>

**弥合真实感差距。** 直接仿真的图像太"干净"，所以我们用 **CycleGAN** 让它学真实 X 光的质感，并施加 **CLAHE** 直方图增强——真实 X 光明暗对比千差万别，模型不能只认得一种。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/synth_cyclegan.jpg' | relative_url }}" alt="gvxr 原始仿真图 vs CycleGAN 增强结果" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">左：gvxr 原始输出。右：经过 CycleGAN——质感明显更接近真实 X 光。</p>

## 第二阶段——U-Net 从合成走向真实

分割模型按 U-Net 原论文（Ronneberger 等）搭建：下采样路径提取特征，上采样路径重建全分辨率图像，跳跃连接保留细节。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/unet_arch.jpg' | relative_url }}" alt="U-Net 结构：下采样、上采样、跳跃连接" style="width:100%; border-radius:8px;">

三个训练细节很关键：

- **组合损失（Dice + BCE）。** Dice 损失擅长区域分离但在边缘处较弱；BCE 补充边缘细节。
- **动态损失加权**，按各损失项的贡献在训练中自动调整。
- **自动阈值选择**：均匀生成 20 个候选阈值，每次随机抽 5 个评估并保留最优——这样训练可以在服务器上无人值守地跑并自动保留最佳检查点。

**先考一场模拟考。** 公开数据集没有标注 X 光里的*心脏*——所以在信任心脏模型之前，我们先用完全相同的训练配置在**肺**分割上做了验证（肺有现成标注）：**准确率 0.918，F1 0.817**。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/lung_seg_test.jpg' | relative_url }}" alt="肺分割代理测试：混淆矩阵与指标" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">流水线在有真值标注的肺分割上的"模拟考"。</p>

**真正的考试需要真正的答案。** 心脏没有任何现成标注——没有评估指标，自动化训练就不知道哪个检查点最好。于是我们（借助 3D 解剖参考）**手工绘制了精确的心脏掩膜**，构建了一个小规模真实测试集。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/manual_mask.jpg' | relative_url }}" alt="真实胸片与手工绘制的心脏掩膜" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">一张真实胸片（左）与手工绘制的真值心脏掩膜（右）。</p>

## 在真实 X 光上的结果

这个只在合成数据上训练过的模型，在真实胸片上对照我们手绘的掩膜做了评估：

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/predictions.jpg' | relative_url }}" alt="真实胸片上的预测：原图、概率图、后处理、叠加" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">从左到右：真实 X 光原图 → 原始概率图 → 后处理（去除零散的假阳性斑块）→ 分割叠加。分割区域正好落在心脏位置。</p>

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/unet/metrics_report.jpg' | relative_url }}" alt="训练汇总：指标趋势、最佳与最差对比、F1 分布" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">共训练 30 轮，100% 成功完成。最佳检查点（第 17 轮）：<strong>准确率约 0.97，F1 = 0.7992</strong>，精确率 0.84，IoU 0.67。</p>

**诚实的提醒：** 心脏指标是对照我们自己手绘的掩膜测的——这是一个小规模、自标注的测试集，不是临床金标准。而且模型测试时所见的是它经 CycleGAN 适配过的同类 X 光风格；真正分布外的医院影像仍是未知数。

## 第三阶段还差什么

最后一个阶段——把 2D 检测结果提升为心脏 3D 位置、轴向与形状的估计，用于异常诊断——完成了设计但没有实现：49 天、2 个人只够完成前两阶段。接下来需要：

- **真实标注数据**——最大的障碍；本项目的所有设计都是在绕开它的缺席。
- 工厂里更多的人体多样性（女性人体模型、更多组织与骨骼变化、不同位置的心脏）。
- 3D 提升模块本身：从 2D 掩膜与轴向反推 3D 位姿。

</div>
