---
layout: page
title: "Kaggriculture — Building a Competition-Grade Game-Playing Agent"
description: "<span class='lang-en'>A 720-turn two-player farming-economy agent for the Kaggle Kaggriculture ladder: ladder Elo 2452.2, about 850th of ~8,000 teams. The work was less about writing a policy than about building the measurement rig that could tell a real improvement from ladder noise.</span><span class='lang-zh'>Kaggle Kaggriculture 天梯上的 720 回合双人农业经营 agent：天梯 Elo 2452.2，约 8000 支队伍中排名 850 名左右。这项工作的重点不在写策略，而在于搭出一套能把真实改进与天梯噪声区分开的测量装置。</span>"
importance: 1
category: projects
toc:
  sidebar: left
---

<div class="lang-en" markdown="1">

<div class="project-overview" markdown="1">

**Kaggle Competition · Kaggriculture · September 2026 · Solo**

**In one sentence:** I built an agent that plays a 720-turn two-player farming economy, and — more importantly — the **tournament harness that could prove a change was real**, which took it from a ladder Elo of 398 to **2452.2 (≈ 850th of ~8,000 teams)**.

*Honest framing up front: the agent descends from public work that is itself the meta — the whole public frontier turns out to be one architecture. What is mine is the evaluation rig, the ablations, and the discipline about what the numbers can actually support. Part 3 is explicit about this.*

</div>

## Part 1 — Background: a game that punishes intuition

[Kaggriculture](https://www.kaggle.com/competitions/kaggriculture) is a two-player farming economy. Each player runs a **10×10 farm split into four 5×5 quadrants**, starting with **$3,000**. Over **720 turns — 30 in-game days of 24 turns** — you plant and harvest crops, raise animals, hire farmhands on a **Fibonacci cost ladder** (1, 1, 2, 3, 5, 8, 13…), and trade into a market whose prices move with **everyone's** supply. Most money at the end of the season wins.

Three properties make it hard to reason about by hand:

- **The market is an opponent, not a backdrop.** Prices are set by accumulated inventory, so a big harvest depresses its own price. Selling *when* your rival sells is often worth more than producing more.
- **The ladder scores by Elo, and only win/loss/draw counts.** Margin is irrelevant to the rating. A strategy that wins by $50 and one that wins by $50,000 are identical on the leaderboard.
- **Time is a hard resource.** With 720 turns and a fixed action budget per turn, a plan that is one turn late is simply worse, and you cannot recover the turn.

Roughly **8,000 teams** were on the ladder, with the top at 3110.7 and a median of about **1010**.

## Part 2 — What I built

### 2.1 First: figure out what the frontier actually is

Before writing any policy, I decoded the public high-scoring agents to see how they worked. The answer was uniform and slightly deflating: **all ten agents I decoded are the same kind of object — a fixed 720-step action tape with a thin reflex layer that patches execution.** None is a rule-based policy that reasons about the board. The differences between the public leaders are not search or strategy; they are *which tape* gets played, *when sales are sent*, and *how execution errors are repaired*.

A lineage diff made this concrete. My file and the two agents I could not beat sit on **one iteration chain**: ours and one rival's differ in **5 constants and a 186-line tail**, and the rival is itself a superset of a third.

This reframed the problem. The frontier is not a search problem, it is a **measurement** problem: with a near-identical agent, progress means knowing which of a handful of tiny constants actually helps.

### 2.2 The tournament harness — the part that mattered

A single game of Kaggriculture is noisy, and the ladder is worse: two different ratings for *identical* code can differ by hundreds of points (I measured this — see Part 3). So I built the evaluation rig first:

- **Paired, dual-seat, multi-seed.** Every match is played from both seats and across a fixed block of seeds, so each configuration is compared on the same worlds rather than on a lucky draw.
- **Mean *and* worst-case margin.** Win rate alone can be fooled by luck; reporting the worst single game catches a change that wins on average by taking catastrophic losses.
- **A resolution floor, measured rather than assumed.** Every A/B is run at **≥500 games per arm on two independent seed blocks**, and a change counts only if both blocks agree. This rule was not free — it came from getting it wrong.

The harness made the difference between "I think this is better" and "this is better, and here is the sample size".

### 2.3 Then: localize one constant properly

The clearest result came from a single parameter — the **sale-advance lookahead**, i.e. how many turns ahead the agent races a planned sale to beat its rival's sale to the same buyer. Instead of guessing, I swept it across **7 values × 2 independent seed blocks × both seats × 500 games per arm** — 7,000 games.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/kaggriculture/adv_look_ablation.png' | relative_url }}" alt="Win rate against the strongest local peer versus the sale-advance lookahead, for two independent seed blocks" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">One constant, swept properly. The two lines are <strong>independent seed blocks</strong> — they land on top of each other, which is what makes the effect credible rather than a lucky draw.</p>

The curve is monotone to a peak at lookahead 12 — **37.5% → 93.1%**, paired by seed, p &lt; 1e-4 — and then flattens and slightly falls. A control run against opponents that carry no such layer showed the gain is concentrated exactly where it should be: worth roughly **+37 points of win rate against a rival that races**, and about **zero against one that does not**.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/kaggriculture/panel_winrate.png' | relative_url }}" alt="Per-opponent win rate on the local panel before and after the change" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">A six-opponent panel before and after: the two agents that used to beat us 20–0 flip, and the four that we already beat stay untouched — the “nothing regressed” half of the result.</p>

## Part 3 — Results, and what they do not prove

**The ladder.** Across the competition the agent went from a rule-engine era stuck around **398–566 Elo** (six successive variants failed to beat their predecessor) to **2162** once it moved to the tape architecture, and reached **2452.2 ≈ 850th of ~8,000 teams**.

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/kaggriculture/ladder_standing.png' | relative_url }}" alt="Distribution of ladder Elo across roughly 8,000 teams, with our best score marked" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">Ladder snapshot, 2026-09-22. The ladder is long and shallow — most of the field sits under 1500, and 2452 is only ~850th.</p>

**Three things I want to be precise about:**

- **The agent is not from scratch, and that caps what the score means.** It is a descendant of public work, under Apache-2.0, with the upstream attribution preserved in the file header. A near-clone of the strongest public file sits near the same rating, so ~2450 is closer to *the equilibrium of that public file* than to a measure of original modelling. My contribution is the harness, the ablations, and the lineage analysis.
- **The ladder cannot resolve small differences, and I measured that instead of pretending otherwise.** Re-submitting **byte-identical** code produced score spreads of **62 / 152 / 609 / 675** Elo across separate draws. Any claimed improvement smaller than that is unfalsifiable on the ladder alone — which is exactly why the local harness exists, and why several confident-looking ladder readings had to be thrown out.
- **The ranking is live.** The competition closes on 2026-09-30 and the final placement depends on the last submitted window; 2452.2 is a snapshot, not a final result.

The most transferable output of this project is not the agent. It is the habit of asking *what would this number look like if nothing had changed* — and building the thing that answers it before trusting the result.

</div>

<div class="lang-zh" markdown="1">

<div class="project-overview" markdown="1">

**Kaggle 竞赛 · Kaggriculture · 2026 年 9 月 · 个人参赛**

**一句话概括：** 我做出了一个能玩 720 回合双人农业经营对局的 agent，但更重要的是**那套能证明"改动是真的"的巡回赛评测装置**——它把天梯 Elo 从 398 推到 **2452.2（约 8000 支队伍中第 850 名）**。

*先把话说清楚：这个 agent 源自公开工作，而公开前沿本身就是"一个架构"。属于我的是评测装置、消融实验，以及对"这些数字到底能支撑什么结论"的克制。第三部分会明确讲这一点。*

</div>

## 一、背景：一个惩罚直觉的游戏

[Kaggriculture](https://www.kaggle.com/competitions/kaggriculture) 是一个双人农业经济模拟。每位玩家经营一座 **10×10 的农场，分成四个 5×5 象限**，起始资金 **3000 美元**。在 **720 个回合——30 个游戏日、每天 24 回合**里，你要种植与收获、养殖牲畜、按**斐波那契成本阶梯**（1、1、2、3、5、8、13……）雇佣工人，并在一个价格随**所有人**供给变动而变动的市场里交易。季末资金最多者获胜。

有三个性质让它很难靠直觉推演：

- **市场是对手，不是背景。** 价格由累计库存决定，所以一次大丰收会压低自己的售价。**在什么时候卖**，往往比**多生产多少**更值钱。
- **天梯按 Elo 计分，而只有胜负平算数。** 资金差距完全不影响评分——赢 50 美元和赢 5 万美元在排行榜上完全等价。
- **时间是硬资源。** 720 个回合、每回合固定动作预算，一个晚了一回合的计划就是更差，而且那一回合拿不回来。

天梯上约有 **8000 支队伍**，榜首 3110.7 分，中位数约 **1010**。

## 二、我做了什么

### 2.1 第一步：先搞清楚前沿到底是什么

在写任何策略之前，我先把公开的高分 agent 解码看了一遍。结论高度一致、也略微让人泄气：**我解码的十个 agent 是同一类东西——一张固定的 720 步动作磁带，外加一层薄薄的反射修补逻辑。** 没有一个是从棋盘状态出发推理的规则策略。公开头部之间的差距不在搜索或策略，而在于**放哪盘磁带**、**什么时候发卖单**、以及**怎么修补执行失误**。

一次血统比对把这一点坐实了。我的文件和两个我打不过的 agent 处在**同一条迭代链上**：我们与其中一个对手的差异只有 **5 个常量加一段 186 行的尾部**，而那个对手本身又是第三个的子集。

这重新定义了问题：前沿不是搜索问题，而是**测量**问题——当双方 agent 几乎相同时，进步就意味着要判断这几个常量里到底哪个真的有用。

### 2.2 巡回赛评测装置——真正的关键部分

单局 Kaggriculture 噪声很大，天梯更糟：**完全相同的代码**两次提交可以差出几百分（我实测过，见第三部分）。所以我先把评测装置搭起来：

- **配对、双座位、多种子。** 每一组配置都从两个座位、在一组固定种子上对打，比较的是**同一批世界**，而不是一次运气抽签。
- **同时报平均边际与最差边际。** 只看胜率会被运气骗；报最差单局才能抓住"平均占优、但偶尔巨亏"的改动。
- **分辨率下限是量出来的，不是假设的。** 每个 A/B 要求**每臂 ≥500 局、两个独立种子块**，两块方向一致才算数。这条纪律不是白来的——是踩坑换的。

有了这套装置，"我觉得这个更好"才变成"这个确实更好，而且样本量在这里"。

### 2.3 然后：把一个常量定位清楚

最干净的结果来自一个参数——**卖出前视长度**，也就是 agent 提前多少个回合去抢在对手之前把货卖给同一个买家。我没有靠猜，而是扫了 **7 个取值 × 2 个独立种子块 × 双座位 × 每臂 500 局**，共 7000 局。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/kaggriculture/adv_look_ablation.png' | relative_url }}" alt="对本地最强对手的胜率随卖出前视长度的变化，两个独立种子块" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">把一个常量扫透。两条线是<strong>两个独立种子块</strong>——它们几乎完全重合，这正是这个效应可信、而非一次幸运抽签的原因。</p>

曲线单调上升到前视 12 的峰值——**37.5% → 93.1%**，按种子配对检验 p &lt; 1e-4——之后走平并略微回落。对**不带该层**的对手做的对照实验显示，增益恰好落在该落的地方：对**会抢卖的对手约值 +37 个百分点胜率**，对**不抢卖的对手约为零**。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/kaggriculture/panel_winrate.png' | relative_url }}" alt="改动前后在本地对手面板上的逐对手胜率" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">六对手面板的前后对比：原本 20–0 碾压我们的两个对手被翻转，原本就打过的四个保持不变——这就是"没有退化"的那一半结论。</p>

## 三、结果，以及这些结果**不能**证明什么

**天梯。** 整个比赛期间，agent 从规则引擎时代卡在 **398–566 Elo**（连续六个变体都没能打败自己的前一版），在换成磁带架构后跃升到 **2162**，并达到 **2452.2，约为 8000 支队伍中的第 850 名**。

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/kaggriculture/ladder_standing.png' | relative_url }}" alt="约 8000 支队伍的天梯 Elo 分布，标出我们的最高分" style="width:100%; border-radius:8px;">
<p style="text-align:center; color:#666; font-size:0.9em;">天梯快照（2026-09-22）。天梯又长又平——绝大多数队伍在 1500 分以下，2452 分也只是第 850 名左右。</p>

**有三件事我想说准确：**

- **这个 agent 不是从零写的，这限定了分数的含义。** 它是公开工作的后代（Apache-2.0，上游署名完整保留在文件头）。公开最强文件的近似克隆大致也在这个分段，所以约 2450 分更接近**那份公开文件的均衡点**，而不是对原创建模能力的度量。我的贡献是评测装置、消融实验和血统分析。
- **天梯分辨不了小差异，而我把这件事量出来了，而不是假装没看见。** 重复提交**逐字节相同**的代码，在不同抽签下分数散布达 **62 / 152 / 609 / 675** Elo。任何小于这个幅度的"改进"在天梯上都无法被证伪——这正是本地评测装置存在的理由，也是若干看起来很有把握的天梯读数最后必须被推翻的原因。
- **排名是活的。** 比赛 2026-09-30 截止，最终名次取决于最后一次提交窗口；2452.2 是一个快照，不是最终结果。

这个项目最可迁移的产出不是那个 agent，而是一个习惯：先问**"如果什么都没变，这个数字会长什么样"**，并在相信结果之前，先把能回答这个问题的东西造出来。

</div>
