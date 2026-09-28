---
layout: page
title: "WorldQuant BRAIN — Automated Alpha Research"
description: "<span class='lang-en'>An end-to-end pipeline for alpha factor mining on the WorldQuant BRAIN platform: 1,354 simulations, 15 submitted alphas, best fitness 3.88. The core move was to reverse-engineer the platform's own admission rules so a candidate can be judged before submission rather than after rejection.</span><span class='lang-zh'>WorldQuant BRAIN 平台上的端到端 Alpha 因子挖掘流水线：1354 次仿真、15 条已提交 Alpha、最佳 fitness 3.88。核心做法是先逆向平台自身的准入规则，让候选在提交前就能被判断，而不是被拒之后再猜原因。</span>"
importance: 2
category: projects
toc:
  sidebar: left
---

<div class="lang-en" markdown="1">

<div class="project-overview" markdown="1">

**Personal project · WorldQuant BRAIN · September 2026 · Claude Code driving, human steering**

**In one sentence:** I built a pipeline that turns alpha hunting from one-expression-at-a-time trial and error into a **batch, traceable, self-correcting loop** — **1,354 platform simulations**, **15 alphas submitted and live**, best **fitness 3.88**.

*Stated plainly up front: the AI (Claude Code on a DeepSeek backend) is the primary executor here. My role is setting the objective, choosing which direction to test next, accepting or rejecting results, and performing the platform submissions. Part 3 is explicit about how to read that split.*

</div>

## Part 1 — Background: a platform whose rules are the real obstacle

WorldQuant BRAIN is a crowdsourced platform for quantitative research. You construct a mathematical expression over **4,367 data fields** and **66 operators**; the platform simulates it and returns **Sharpe, fitness, turnover** and eight compliance checks; alphas that clear everything can be submitted to a live pool.

The naive working loop is: *guess an expression → submit → get rejected → guess again.* The rejection tells you **that** you failed, not **why**, and the rules that decide are not published. That is the actual bottleneck — not idea generation.

So the project inverted the order: **understand the gate first, mine factors second.**

## Part 2 — What I built

### 2.1 Reverse-engineering the admission rules

I fit the platform's decision behaviour against my own accumulated results, and recovered six rules. The load-bearing three:

- **The fitness formula.** `fitness = sharpe × √(returns / max(turnover, 0.125))` — validated against **262** of my own simulations, **maximum error 0.005**.
- **The sub-universe check threshold.** It triggers at a **sub-universe / main Sharpe ratio of 0.433** — validated on **182** simulations, and confirmed with **zero misjudgements** on a 21-sample holdout.
- **The self-correlation penetration rule** — **13 of 13** positive confirmations.

Once a candidate's outcome is computable before submission, the loop changes character: instead of spending a slot to learn whether an idea is admissible, you spend slots only on ideas that are.

**The counter-intuitive consequence.** If the sub-universe check fails at a ratio of 0.433, then a *higher* main Sharpe can be what gets you rejected. Acting on that, I deliberately **pushed Sharpe down** — via a shorter smoothing window and `sector` grouping — to clear a check that a stronger-looking alpha would have failed.

### 2.2 The pipeline itself

Four pieces, each solving something the platform does not help with:

1. **Simulation scheduler.** The platform allows **three concurrent simulations**, so the scheduler manages that hard cap, deduplicates by expression (a re-run costs nothing), distinguishes the two different `429` conditions — concurrency limit versus rate limit — and guards against simulations that hang forever.
2. **Self-correlation pre-check.** This one is subtle. The rule is not a fixed threshold: a new alpha correlated above **ρ = 0.70** with an existing one must beat **1.1 × that specific alpha's Sharpe**. Parsing the platform's `/check` response to find *which* alpha you correlate with — and what its Sharpe is — matters, because the bar moves by up to **40%** depending on the counterparty. Pre-checking only ρ would misjudge it.
3. **Co-existence solver.** The obvious reading of the rule is a path problem — keep the longest chain of mutually compatible alphas. It is not: **compatibility is not transitive**, so this is a **maximum-clique problem**. I implemented a greedy colouring bound that outputs the largest simultaneously-submittable set *and* the order to submit in. The gap between "passes the gate" and "can coexist" turned out to be the whole story — see below.
4. **A three-zone knowledge base.** External claims and my own findings are filed as **useful / landmine / uncertain**, each with a source and an evidence grade. External advice enters as a hypothesis, gets a controlled test, and is re-filed based on measurement. Six external claims were validated and **22 directions disproven** this way.

## Part 3 — Results, and how to read them

| Metric | Value |
|---|---|
| Platform simulations run | **1,354** |
| Alphas submitted and live | **15** (all submitted manually, per platform rules) |
| Best fitness | **3.88** — platform rating **SPECTACULAR** |
| Rating distribution | SPECTACULAR ×2, EXCELLENT ×1, GOOD several |
| Platform rules reverse-engineered | **6** |
| Directions disproven and archived | **22** |
| Independent signal source | 1 (option put–call IV divergence, ρ 0.30–0.56 against 200+ existing factors) |

**The result I would actually put first is a negative one.** 22 alphas cleared every admission gate — but the co-existence solver showed only **4 could ever be submitted together**. Any count of "alphas that passed the checks" that stops at the first number is overstating the output by a factor of five. Quantifying that ceiling is what the clique solver is for.

**Two honest caveats:**

- **The AI did the execution; I did the steering.** Claude Code drove experiment design, result interpretation, rule reverse-engineering and code; I set the objective, decided the next direction, accepted or rejected findings, and performed every submission. That division is worth stating plainly rather than letting the numbers imply a solo hand-crafted effort.
- **Originality is in the method, not the expressions.** I deliberately avoided reusing public alpha expressions — copied templates correlate with each other and are self-defeating. What is borrowed is methodology and structure; every expression was constructed and measured here. The one external construction I reproduced (option put–call implied-volatility divergence, from a public paper) measured **fitness 3.88 / Sharpe 3.34** and correlates only **0.30–0.56** with the 200+ factors already in the pool.

**Negative results were kept as assets.** The 22 disproven directions are archived with the *cause* of failure rather than a bare "didn't work" — for example, the analyst-disagreement family failed because those specific fields are weak, not because the idea is wrong. The project also logged **six of my own judgement errors** and turned each into a standing check.

## Certificates

Three certificates of accomplishment from the WorldQuant Challenge, issued by WorldQuant and authorised by its Chief Strategy Officer, recording the levels reached:

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/worldquant/bronze_certificate.png' | relative_url }}" alt="WorldQuant Challenge Bronze Level certificate awarded to Wang YuHeng" style="width:100%; border-radius:8px;">

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/worldquant/silver_certificate.png' | relative_url }}" alt="WorldQuant Challenge Silver Level certificate awarded to Wang YuHeng" style="width:100%; border-radius:8px;">

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/worldquant/gold_certificate.png' | relative_url }}" alt="WorldQuant Challenge Gold Level certificate awarded to Wang YuHeng" style="width:100%; border-radius:8px;">

</div>

<div class="lang-zh" markdown="1">

<div class="project-overview" markdown="1">

**个人项目 · WorldQuant BRAIN · 2026 年 9 月 · Claude Code 主控执行，本人负责决策**

**一句话概括：** 我做了一条把 Alpha 挖掘从"一条表达式一次试错"变成**可批量、可追溯、能自我纠错**的流水线 —— **1354 次平台仿真**、**15 条 Alpha 已提交上线**，最佳 **fitness 3.88**。

*先把话说清楚：这里的主控执行者是 AI（Claude Code + DeepSeek 后端）。我的角色是设定目标、决定下一步测什么方向、验收或否决结果、以及执行平台提交。第三部分会明确讲清楚这个分工该怎么理解。*

</div>

## 一、背景：真正的障碍是平台规则本身

WorldQuant BRAIN 是一个量化研究众包平台。你用平台提供的 **4367 个数据字段**和 **66 个算子**构造数学表达式，平台仿真后返回 **Sharpe、fitness、换手率**与八项合规检查；全部通过的 Alpha 才能提交进实盘池。

朴素的工作循环是：*猜一条表达式 → 提交 → 被拒 → 再猜*。拒绝只告诉你**失败了**，不告诉你**为什么**，而决定成败的规则并未公开。**这才是真正的瓶颈——不是点子不够。**

所以这个项目把顺序倒了过来：**先搞懂闸门，再挖因子。**

## 二、我做了什么

### 2.1 逆向平台的判定规则

我用自己积累的仿真结果去拟合平台的行为，还原出六条规则。其中承重的三条：

- **fitness 公式**：`fitness = sharpe × √(returns / max(turnover, 0.125))` —— 用自有 **262** 条仿真验证，**最大误差 0.005**。
- **子宇宙检查阈值**：触发点是**子宇宙 / 主 Sharpe 比值 0.433** —— 用 **182** 条仿真验证，并在 21 条留出样本上做到**零误判**。
- **自相关穿透规则** —— **13/13** 正面验证。

一旦候选的结局在提交前就能算出来，这个循环的性质就变了：不必再花一个仿真槽去问"这个想法合不合规"，只把槽位花在已经确定合规的想法上。

**反直觉的推论。** 既然子宇宙检查在比值 0.433 处触发，那么**主 Sharpe 更高**反而可能让你被拒。据此我**刻意压低 Sharpe**——用更短的平滑窗配合 `sector` 分组——去过掉一个"看起来更强"的 Alpha 反而过不了的检查。

### 2.2 流水线本身

四块，每一块都在解决平台本身不帮你解决的问题：

1. **仿真调度器。** 平台只允许 **3 个并发仿真**，调度器管理这个硬上限；按表达式去重（重跑不占新槽位）；区分两类 `429`（并发上限与限流）；并对永久卡死的仿真做守卫。
2. **自相关预检。** 这一条很微妙：门槛不是固定值——新 Alpha 与已有 Alpha 相关性超过 **ρ = 0.70** 时，必须打赢 **1.1 × 对方那条 Alpha 的 Sharpe**。所以必须解析平台 `/check` 返回，**定位到"你在跟哪一条相关、它的 Sharpe 是多少"**——因为门槛会随对手浮动最多 **40%**。只看 ρ 会误判。
3. **共存集求解器。** 这条规则的直觉读法是路径问题——找互不冲突的最长链。**并不是**：兼容性**不可传递**，所以这是**最大团问题**。我实现了一个贪心着色上界求解，输出**可同时提交的最大集合**与**提交顺序**。"过闸门"与"能共存"之间的差距后来成了这个项目最重要的发现——见下文。
4. **三区知识库。** 外部主张与自己的发现一律按 **有益 / 排雷 / 不确定**归档，每条带来源与证据等级。外部经验以假设身份进入，做对照实验，再按实测结果重新分区。用这种方式验证了 6 条外部经验，**证伪了 22 个方向**。

## 三、结果，以及该怎么读这些结果

| 指标 | 数值 |
|---|---|
| 平台仿真次数 | **1354** |
| 已提交上线 Alpha | **15** 条（全部按平台规则人工提交） |
| 最佳 fitness | **3.88** —— 平台评级 **SPECTACULAR** |
| 评级分布 | SPECTACULAR ×2、EXCELLENT ×1、GOOD 若干 |
| 逆向出的平台规则 | **6** 条 |
| 已证伪并归档的方向 | **22** 个 |
| 独立信号源 | 1 个（期权 put-call 隐含波动率分歧，与既有 200+ 因子 ρ 仅 0.30–0.56） |

**我真正想放在第一位的是一个负面结果。** 有 22 条 Alpha 通过了全部准入检查——但共存集求解器显示，其中**只有 4 条能同时提交**。任何停在第一个数字上的"通过检查的 Alpha 数"，都把实际产出高估了五倍。量化这个上限，正是最大团求解器的用途。

**两条诚实的保留：**

- **执行是 AI 做的，掌舵是我。** Claude Code 承担实验设计、结果解读、规则逆向与代码编写；我设定目标、决定下一个方向、验收或否决结论、并执行每一次提交。这个分工值得明说，而不是让数字暗示这是一个人手工打磨的成果。
- **原创性在方法，不在表达式。** 我刻意不使用公开 Alpha 表达式——公开模板会被多人照抄而互相高相关，**策略上自败**。借用的是方法论与结构；每一条表达式都在本地构造并实测。唯一复现的外部构造（期权 put-call 隐含波动率分歧，来自公开论文）实测 **fitness 3.88 / Sharpe 3.34**，与池中既有 200+ 因子的相关性只有 **0.30–0.56**。

**负面结果被当作资产保留。** 22 个证伪方向都记录了**失败原因**，而不是一句"试过不行"——例如"分析师分歧度"那一族失败，记录指出**死因是那族字段本身低效，不是"分歧"这个概念**。项目全程还记录并修正了**我自己 6 次判断错误**，每条都写成一条常设检查规则。

## 证书

三张 WorldQuant Challenge 成就证书，由 WorldQuant 颁发、由其首席战略官授权，记录所达到的等级：

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/worldquant/bronze_certificate.png' | relative_url }}" alt="颁发给 Wang YuHeng 的 WorldQuant Challenge 铜牌等级证书" style="width:100%; border-radius:8px;">

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/worldquant/silver_certificate.png' | relative_url }}" alt="颁发给 Wang YuHeng 的 WorldQuant Challenge 银牌等级证书" style="width:100%; border-radius:8px;">

<img loading="lazy" decoding="async" src="{{ '/assets/img/projects/worldquant/gold_certificate.png' | relative_url }}" alt="颁发给 Wang YuHeng 的 WorldQuant Challenge 金牌等级证书" style="width:100%; border-radius:8px;">

</div>
