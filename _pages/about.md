---
layout: about
title: HOME
permalink: /
subtitle: "<span class='lang-en'>Final-year Data Science undergraduate at <a href='https://hkust.edu.hk/'>HKUST</a></span><span class='lang-zh'><a href='https://hkust.edu.hk/'>香港科技大学</a>数据科学专业大四本科生</span>"

profile:
  align: right
  image: prof_pic.jpg # replace assets/img/prof_pic.jpg to change the photo
  image_circular: false # crops the image to make it circular
  more_info: >
    <p>Hong Kong SAR</p>
    <p>ywangrg@connect.ust.hk</p>

selected_papers: false # includes a list of papers marked as "selected={true}"
social: true # includes social icons at the bottom of the page

announcements:
  enabled: false # includes a list of news items
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: false
  scrollable: true # adds a vertical scroll bar if there are more than 3 new posts items
  limit: 3 # leave blank to include all the blog posts
---

## <span class="lang-en">BACKGROUND</span><span class="lang-zh">背景</span>

<div class="lang-en" markdown="1">

I am a final-year Data Science student at the Hong Kong University of Science and Technology (HKUST), with a strong foundation in **machine learning, computer vision, statistical modeling, and programming**.

This summer I worked as a Computer Vision intern at Zhijie Exploration Technology (Shenzhen), where I built a metric-scale multi-view 3D reconstruction pipeline for tabletop food volume estimation from scratch, reducing MAPE from *25.4% to 12.36%*. Previously, I trained a U-Net model for heart detection and segmentation in X-ray images at Seoul National University (over 90% accuracy, 0.7992 F1-score), and I have conducted research with Prof. Gary Chan at HKUST on making 3D reconstruction more efficient by using 2D floorplans as structural priors. My final-year project explores a personal embodied AI agent driven by reinforcement learning.

</div>

<div class="lang-zh" markdown="1">

我是香港科技大学（HKUST）数据科学专业的大四学生，在**机器学习、计算机视觉、统计建模与编程**方面有扎实的基础。

今年夏天，我在智界探索科技（深圳）担任计算机视觉实习生，从零搭建了一套面向桌面食物体积估计的 metric-scale 多视角 3D 重建流水线，将 MAPE 从 *25.4% 降至 12.36%*。此前，我在首尔国立大学训练了用于 X 光图像中心脏检测与分割的 U-Net 模型（准确率超过 90%，F1 分数 0.7992），并在香港科技大学 Gary Chan 教授的指导下开展研究，利用 2D 户型图作为结构先验提升 3D 重建的效率。我的毕业设计探索的是一个由强化学习驱动的个人具身智能体。

</div>

## <span class="lang-en">SKILLS</span><span class="lang-zh">技能</span>

<div class="lang-en" markdown="1">

- **Programming:** Python, C++, Java, R
- **Machine Learning & Data:** PyTorch, NumPy, pandas
- **Visualization:** Tableau, Datawrapper, iNZight, RAWGraphs
- **Languages:** Mandarin (native), English

</div>

<div class="lang-zh" markdown="1">

- **编程语言：** Python、C++、Java、R
- **机器学习与数据：** PyTorch、NumPy、pandas
- **可视化：** Tableau、Datawrapper、iNZight、RAWGraphs
- **语言：** 中文（母语）、英语

</div>

## <span class="lang-en">PROJECTS</span><span class="lang-zh">项目</span>

<div class="lang-en" markdown="1">

A quick overview — click any title for the dedicated detail page, or browse the full [projects]({{ '/projects/' | relative_url }}) page.

</div>

<div class="lang-zh" markdown="1">

快速概览——点击标题进入详情页，或浏览完整的[项目页]({{ '/projects/' | relative_url }})。

</div>

{% assign categories = "research,projects" | split: "," %}
{% for category in categories %}

### {{ category | capitalize }}

{% assign projs = site.projects | where: "category", category | sort: "importance" %}
{% for p in projs %}

- **[{{ p.title }}]({{ p.url | relative_url }})** — {{ p.description }}
{% endfor %}
{% endfor %}

## <span class="lang-en">INTERESTS & DIRECTIONS</span><span class="lang-zh">兴趣与方向</span>

<div class="lang-en" markdown="1">

I am seeking opportunities in **data analysis** and **AI/CV research**, for both graduate study and industry roles. My current interests include computer vision and 3D reconstruction, embodied AI, and applied machine learning on real-world problems. The [CV]({{ '/cv/' | relative_url }}) page has my full background.

</div>

<div class="lang-zh" markdown="1">

我正在寻找**数据分析**和 **AI/CV 研究**方向的机会，包括研究生深造和工业界职位。我目前的兴趣包括计算机视觉与 3D 重建、具身智能，以及面向真实问题的应用机器学习。我的完整背景见 [CV]({{ '/cv/' | relative_url }}) 页面。

</div>
