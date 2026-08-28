---
layout: page
title: Visualizing AI's Impact on the Job Market
description: "<span class='lang-en'>An interactive scrollytelling website that turns job-market data into a job-seeker's guide to the AI era — world map, industry trends, job volumes, and a personalized scorecard. Built with React + D3.js.</span><span class='lang-zh'>一个交互式滚动叙事网站，把就业市场数据变成一份 AI 时代的求职指南——世界地图、行业趋势、岗位数量、个性化记分卡。React + D3.js 构建。</span>"
importance: 5
category: projects
toc:
  sidebar: left
---

<div class="lang-en" markdown="1">

<div class="project-overview" markdown="1">

**Course Project · Data Visualization, HKUST · 2025 · Team of two**

**In one sentence:** an interactive **scrollytelling website** that walks a visitor from "where in the world should I work?" to "what should I personally optimize for?" — a data-driven job-seeker's guide to the AI era, built with **React + D3.js**.

*Source code, data, and run instructions: [github.com/TheLogarhythm/Career-a-vis-AI](https://github.com/TheLogarhythm/Career-a-vis-AI) (`npm install` → `npm run dev`).*

</div>

## Part 1 — Background: a course project, a free topic

For the final project of our data visualization course, each team picked its own theme and built a visualization website around it. My teammate and I chose a question we both cared about as students about to enter the job market: **how is AI reshaping jobs and industries — and what should a job seeker do about it?**

We assembled the data from public sources, mostly Kaggle:

- **JobStreet job postings** (~59k real listings) — job volumes by category and subcategory.
- **Tortoise Global AI Index** (62 countries) — national AI readiness scores.
- **AI-impact job datasets** (~5k postings 2010–2025 + ~30k trend rows) — curated/AI-generated datasets that provide the per-job AI intensity, salary, and automation-risk attributes. We labeled them as AI-generated on the site itself — more on that honesty point in Part 3.

The design goal was not a dashboard but a **story**: each scroll stop answers one question a job seeker would actually ask, in order.

## Part 2 — What we built

The site is a guided scroll narrative: a left panel explains the current stage while the right side presents the interactive chart. Four stops:

### 2.1 Where to work? — the world map

A choropleth of average salary by country (2010–2025), with a hover breakdown and a live top-10 ranking — plus a second view coloring countries by the Global AI Index, so "high pay" and "AI readiness" can be weighed side by side.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/dataviz/map.jpg' | relative_url }}" alt="World map of average salary by country with top-10 ranking" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">The first stop: average salary by country on the map, top-10 leaderboard on the right. The left panel narrates the current stage.</p>

### 2.2 Which industry? — 15 years of trends

Four synchronized trend lines (2010–2025) — **AI adoption %, AI intensity, average salary, automation risk** — selectable per industry, with any second industry overlaid for comparison. The same stop includes an industry × salary-bracket density heatmap with an AI-intensity threshold slider.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/dataviz/trends.jpg' | relative_url }}" alt="Trend analysis: AI adoption, AI intensity, salary, risk over 2010-2025" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">AI adoption climbs sharply after 2018 while average automation risk declines — pick any industry to see whether it beats the market average.</p>

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/dataviz/heatmap.jpg' | relative_url }}" alt="Heatmap of job density by industry and salary bracket" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Job density by industry and salary bracket; the slider highlights only brackets whose average AI intensity clears the chosen threshold.</p>

### 2.3 How many jobs are out there? — risk-weighted demand

Raw job counts mislead: a huge category with a huge automation risk is not "demand". So the circle-packing view multiplies each subcategory's job count by its automation risk — circle area = **risk-weighted demand** — across the 9 industry categories.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/dataviz/jobcount.jpg' | relative_url }}" alt="Circle packing of risk-weighted job count by subcategory" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Circle area encodes risk-weighted job count (raw count × average automation risk). Pick a category to compare its subcategories.</p>

### 2.4 What should you optimize for? — a personalized scorecard

The final stop hands the analysis to the visitor: a **draggable pie chart** sets weights over six factors — salary, AI intensity, automation risk, reskilling need, displacement risk, skill complexity — and the industry and regional rankings re-compute live. Everyone's "best industry" is different; this lets the chart reflect what matters to *you*.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/dataviz/scorecard.jpg' | relative_url }}" alt="Personalized scorecard: draggable weights, live industry and regional rankings" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Drag the pie segments to re-weight the six factors; the industry (blue) and regional (green) rankings update instantly.</p>

**Under the hood:** React 19 + TypeScript + Vite, with all charts hand-built in **D3.js** and data loaded from CSV at runtime. The aggregation and cleaning behind the charts lives in a pandas notebook in the same repo; earlier prototypes were sketched in Datawrapper, Tableau, iNZight, and RAWGraphs.

## Part 3 — Reflections and next steps

- **Data honesty.** Two of the four datasets are AI-generated/curated rather than measured labor statistics — the site says so with an explicit "AI-Generated Data" badge. The real deliverable of this project is the **narrative and interaction design**; the numbers illustrate it, but they are not labor-market truth.
- **Scrollytelling works.** Tying each chart to a narrated question made the site far more approachable than a dashboard of widgets — the most-cited compliment in the course showcase.
- **Next step** would be swapping the synthetic data for real labor statistics (e.g., BLS or LinkedIn Economic Graph) and deploying the site publicly; the architecture (CSV-driven D3 components) makes that a data swap, not a rewrite.

</div>

<div class="lang-zh" markdown="1">

<div class="project-overview" markdown="1">

**课程项目 · 数据可视化（香港科技大学）· 2025 · 两人小组**

**一句话概括：** 一个**滚动叙事**交互网站，带访问者从"该去世界上哪里工作？"一路走到"我个人最该看重什么？"——一份数据驱动的 AI 时代求职指南，用 **React + D3.js** 构建。

*源码、数据与运行方式：[github.com/TheLogarhythm/Career-a-vis-AI](https://github.com/TheLogarhythm/Career-a-vis-AI)（`npm install` → `npm run dev`）。*

</div>

## 一、项目背景：课程项目，题目自选

数据可视化课的期末项目要求每组自定主题、做一个可视化网站。我和搭档选了一个我们都关心的问题——作为即将进入就业市场的学生：**AI 正在如何重塑岗位与行业？求职者该怎么办？**

数据来自公开渠道，主要是 Kaggle：

- **JobStreet 招聘数据**（约 5.9 万条真实招聘信息）——按类别/子类别统计岗位数量。
- **Tortoise 全球 AI 指数**（62 个国家）——各国 AI 准备度评分。
- **AI 岗位影响数据集**（约 5 千条 2010–2025 岗位 + 3 万条趋势数据）——整理/AI 生成的数据集，提供每个岗位的 AI 强度、薪资、自动化风险等属性。我们在网站上也明确标注了它是 AI 生成的——这一点在第三部分细说。

设计目标不是 dashboard，而是一个**故事**：每一次滚动停留都按顺序回答一个求职者真正会问的问题。

## 二、我们做了什么

网站是引导式滚动叙事：左侧面板讲解当前环节，右侧展示对应的交互图表。共四站：

### 2.1 去哪里工作？——世界地图

按国家统计的平均薪资 choropleth 地图（2010–2025），悬停可逐年分解，右侧实时 Top-10 榜单——另有按全球 AI 指数着色的第二视图，把"高薪"和"AI 准备度"放在一起权衡。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/dataviz/map.jpg' | relative_url }}" alt="按国家的平均薪资世界地图与 Top-10 榜单" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">第一站：地图上是各国平均薪资，右侧是 Top-10 榜单，左侧面板负责讲解当前环节。</p>

### 2.2 选哪个行业？——十五年趋势

四条联动的趋势线（2010–2025）——**AI 采用率、AI 强度、平均薪资、自动化风险**——可按行业切换，并可叠加第二个行业做对比。同一站还有一张"行业 × 薪资区间"的岗位密度热力图，配 AI 强度阈值滑块。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/dataviz/trends.jpg' | relative_url }}" alt="趋势分析：2010–2025 的 AI 采用率、强度、薪资与风险" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">2018 年后 AI 采用率陡增，而平均自动化风险在下降——任选行业即可对比它是否跑赢市场均值。</p>

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/dataviz/heatmap.jpg' | relative_url }}" alt="行业 × 薪资区间的岗位密度热力图" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">行业 × 薪资区间的岗位密度；拖动滑块可只高亮平均 AI 强度超过阈值的区间。</p>

### 2.3 到底有多少岗位？——风险加权需求

原始岗位数量会说谎：一个庞大但自动化风险同样高的类别算不上"需求"。所以圆形堆积图把每个子类别的岗位数乘以其自动化风险——圆面积 = **风险加权需求**——覆盖 9 个行业大类。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/dataviz/jobcount.jpg' | relative_url }}" alt="按子类别的风险加权岗位数圆形堆积图" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">圆面积编码风险加权岗位数（原始数量 × 平均自动化风险）。选择大类即可对比其子类别。</p>

### 2.4 你最该看重什么？——个性化记分卡

最后一站把分析权交给访问者：一个**可拖动的饼图**用来设置六个因素的权重——薪资、AI 强度、自动化风险、再培训需求、替代风险、技能复杂度——行业榜和地区榜随之实时重算。每个人的"最佳行业"都不同，这张图让答案反映你自己的偏好。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/dataviz/scorecard.jpg' | relative_url }}" alt="个性化记分卡：可拖权重，实时行业与地区榜单" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">拖动饼图扇区即可调整六个因素的权重；行业榜（蓝）与地区榜（绿）即时更新。</p>

**技术实现：** React 19 + TypeScript + Vite，所有图表用 **D3.js** 手写，数据在运行时从 CSV 加载。图表背后的清洗与聚合在同一仓库的 pandas notebook 里；早期原型用 Datawrapper、Tableau、iNZight、RAWGraphs 打过草稿。

## 三、反思与下一步

- **数据诚实。** 四个数据集里有两个是 AI 生成/整理的，而不是真实测量的劳动力统计——网站上用"AI-Generated Data"徽章明确标出。这个项目真正的交付物是**叙事与交互设计**：数字是为它服务的插图，不是劳动力市场的结论。
- **滚动叙事是有效的。** 把每张图绑在一个被讲述的问题上，比一堆 widget 拼成的 dashboard 友好得多——这是课程展示中被夸得最多的点。
- **下一步**是把合成数据换成真实劳动力统计（如 BLS 或 LinkedIn Economic Graph）并把网站公开部署；现有架构（CSV 驱动的 D3 组件）让这件事只是换数据，不用重写。

</div>
