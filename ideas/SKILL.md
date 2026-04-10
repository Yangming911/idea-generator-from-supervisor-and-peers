---
name: ideas
description: "Research Idea Generator - Generate novel research ideas based on mentor skills + latest arxiv papers, with automatic novelty verification. | 科研 Idea 生成器 - 基于导师 Skill + arxiv 最新论文，自动生成并查重验证新颖研究 idea。"
argument-hint: "[<mentor-skill-name>] [--count <number>] [--months <number>] [--save <directory>]"
version: "1.1.0"
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, Task
---

> **Language / 语言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout.
>
> 本 Skill 支持中英文。根据用户第一条消息的语言，全程使用同一语言回复。

# Research Idea Generator / 科研 Idea 生成器

基于导师数字分身 Skill 的研究兴趣，结合 arxiv 最新论文，自动生成具有新颖性的研究 idea，并通过全时段 arxiv 查重确保独创性。

## 触发条件 / Trigger Conditions

Activate when the user says:
- `/ideas` or `/ideas <mentor-skill-name>`
- `/ideas xiang-yin --count 20 --months 3`
- "帮我生成研究 idea"
- "Generate research ideas based on <mentor>"

## 参数说明 / Parameters

| 参数 | 说明 | 默认值 |
|:---|:---|:---|
| `<mentor-skill-name>` | 导师 skill 名称（如 `xiang-yin`） | 自动检测 `~/.claude/skills/` 中的导师 skill |
| `--count <N>` | 生成 idea 数量 | 20 |
| `--months <N>` | arxiv 论文检索时间范围（最近 N 个月） | 3 |
| `--save <dir>` | 保存目录 | 当前工作目录下的 `ideas/` |
| `--skip-verify` | 跳过全时段查重步骤 | false |
| `--verify-only` | 仅对已有 idea 文件执行查重 | false |

## 完整工作流程 / Complete Workflow

本 skill 严格按以下 5 个阶段执行，每个阶段完成后向用户汇报进度。

---

### Phase 1: 加载导师研究画像 / Load Mentor Research Profile

**目标**：从导师 skill 中提取最新研究兴趣、核心专长和研究风格。

**步骤**：

1. 读取导师 skill 文件：`~/.claude/skills/<mentor-name>/SKILL.md`
2. 提取以下关键信息：
   - **核心研究领域**（按重要度排序）
   - **最新研究兴趣**（热度排序，关注 🔥 标记的方向）
   - **代表性贡献**（用于定位独特优势）
   - **研究风格**（理论型/应用型/混合型）
   - **审阅与评判偏好**（用于后续打分）
3. 输出摘要：

```
📋 导师研究画像加载完成：
- 导师：<name> (<institution>)
- 核心领域：<fields>
- 最热方向：<top-3-directions>
- 研究风格：<style>
```

---

### Phase 2: arxiv 前沿论文调研 / Frontier Paper Survey

**目标**：检索 arxiv 上最近 N 个月的论文，覆盖导师所有研究兴趣方向。

**步骤**：

1. 根据 Phase 1 提取的研究兴趣，构造搜索查询组（每个热门方向 2-3 个查询）
2. 使用 `WebSearch` 并行搜索 arxiv（按方向分组并行）
3. 对每个方向，收集 5-10 篇最相关的最新论文
4. 提取每篇论文的：标题、arXiv ID、日期、核心贡献（1-2句话）
5. 识别跨方向的 **交叉趋势** 和 **研究空白**
6. **加载互补课题组数据**：检查 `peers/` 目录是否存在该导师的 peers MD 文件（如 `peers/peers_<mentor>_*.md`），若存在则读取并提取各课题组的近期代表作和潜在交叉点，作为额外灵感来源

**搜索策略**：

```
对每个研究兴趣方向：
  - 查询 1：方向关键词 + "arxiv <year>"
  - 查询 2：方向关键词 + 导师核心专长交叉
  - 查询 3：方向关键词 + 最新技术热点（如 LLM、diffusion）

并行执行所有查询组（使用 Task tool 的 subagent 并行搜索）
```

**输出摘要**：

```
📚 前沿论文调研完成：
- 检索方向：<N> 个
- 收集论文：<M> 篇
- 时间范围：<start-date> ~ <end-date>
- 关键趋势：<trend-1>, <trend-2>, <trend-3>
```

---

### Phase 3: Idea 生成 / Idea Generation

**目标**：以导师视角，结合前沿调研，生成 N 个新研究 idea。

**生成原则**：

1. **前沿灵感优先**：互补课题组的近期工作是重要的灵感来源，优先级不低于导师自身的经典工作。鼓励从 peer 近期成果中自由获取启发，不限于简单的"导师方法 A + 外部方法 B"的机械组合
2. **导师专长优先**：每个 idea 至少涉及导师的一个核心专长领域
2. **交叉创新**：优先生成跨方向的交叉 idea（如导师核心专长 × 热门新方向）
3. **分类覆盖**：确保 idea 分布在不同方向，避免过度集中
4. **理论+应用平衡**：根据导师研究风格调整理论/应用比例
5. **难度梯度**：包含不同难度级别的 idea（快速发表型、博士论文型、高风险高回报型）

**每个 Idea 包含**：

```markdown
### Idea N: <标题（英文）>

**思路**：<详细描述，3-5 句话，包括>
- 问题动机（为什么这个问题重要）
- 核心方法（用什么工具/框架/技术解决）
- 关键创新点（与现有工作的本质区别）
- 预期贡献（理论结果 or 系统/方法）

- **科研价值**：X/10 — <简要理由>
- **可行性**：X/10 — <简要理由>
```

**分类策略**（以 20 个 idea 为例）：

```
- 最热方向 × 核心专长 交叉：5-6 个
- 热门方向 × 核心专长 交叉：4-5 个
- 核心专长内部深化：3-4 个
- 跨领域融合创新：3-4 个
- 探索性/高风险 idea：2-3 个
```

---

### Phase 4: 全时段 arxiv 查重 / Full-Period Novelty Verification

**目标**：对每个生成的 idea 在 arxiv 全时段进行查重，确保没有已发表的重复工作。

**步骤**：

1. 对每个 idea 构造 3-4 个精准搜索查询（关键词组合）
2. 使用 `Task` tool 并行启动多个搜索 agent（每 5 个 idea 一组并行）
3. 每个 agent 搜索并评估：
   - 是否有 **直接匹配** 的已发表工作（ALREADY EXISTS）
   - 最接近的 **相关工作** 是什么（列出论文）
   - 相关工作与该 idea 的 **关键区别** 在哪里
4. 给出查重判定：

```
NOVEL — 确认新颖，无直接匹配工作
PARTIALLY NOVEL — 部分组件已有工作，但特定组合/角度新颖
ALREADY EXISTS — 核心思想已被发表
```

**查重 Agent Prompt 模板**：

```
Search arxiv (ALL time periods) to check if the following research idea
already exists. Search thoroughly and report:
(1) Whether there are existing papers doing the same thing
(2) The closest related work found (with arxiv IDs)
(3) Any new inspiration from related papers

Idea: <title>
Description: <description>

Search queries to try:
- <query 1>
- <query 2>
- <query 3>

Provide verdict: NOVEL / PARTIALLY NOVEL / ALREADY EXISTS, with evidence.
```

---

### Phase 5: 修订与输出 / Revision and Output

**目标**：根据查重结果修订 idea 列表，生成最终报告。

**修订规则**：

1. **ALREADY EXISTS** → 替换为新 idea（重新生成 + 重新查重）
2. **PARTIALLY NOVEL** → 增强描述，将与已有工作的区分点自然融入思路中
3. **NOVEL** → 保留原样
4. 在报告末尾挑选 3-5 个最适合 LLM agent 自迭代实现的 idea

**输出风格要求**：
- 不使用 emoji
- idea 标题不附加任何查重状态标签（如 [NOVEL]、[ENHANCED] 等）
- 不强调查重前后的差异（用户看不到查重前版本），不需要"查重总结"对比表
- 对于 PARTIALLY NOVEL 的 idea，将与已有工作的区分自然写入思路描述，不单独设"查重结论"或"增强方向"段落
- 整体呈现为一份干净的最终报告

**最终输出格式**：

```markdown
# <N> 个研究 Idea：<主题>

> 基于 <mentor-name> 教授最新研究兴趣，结合 arxiv <date-range> 最新论文生成
> 生成日期：<date>
> 评分标准：科研价值（Novelty/Impact）和可行性（Feasibility）各 1-10 分
> 全部 idea 已通过 arxiv 全时段查重

---

## [分类标题]

### Idea N: <标题>

**思路**：<详细描述，包括与最相近已有工作的区分>

- **科研价值**：X/10 -- <简要理由>
- **可行性**：X/10 -- <简要理由>

---

## 总结排名

| 排名 | Idea # | 标题 | 科研价值 | 可行性 | 综合推荐 |
|:---:|:---:|:---|:---:|:---:|:---:|
| ... | ... | ... | ... | ... | ... |

---

## 建议

### 第一梯队：最推荐立即启动
### 第二梯队：高新颖度
### 第三梯队：适合快速发表
### 第四梯队：高风险高回报

---

## LLM Agent 自迭代实现推荐

从以上 idea 中，挑选 3-5 个最适合交给 LLM agent（如 Claude Code）通过"写代码->运行->看结果->迭代"方式自主推进的 idea。

评判维度：代码闭环性、工具链成熟度、LLM 自反性、评估可量化性。

| 推荐 | Idea # | 标题 | 推荐理由（1句话） |
|:---:|:---:|:---|:---|
| 1 | #X | ... | ... |
| 2 | #X | ... | ... |
| 3 | #X | ... | ... |

---

## 参考文献
### 最相近的已有工作
### 查重中发现的启发性工作
```

**文件保存**：

```
<save-dir>/ideas_<YYYY-MM-DD>.md
```

---

## 评分标准 / Scoring Criteria

以导师的学术审美和研究偏好为基准打分：

### 科研价值 (Novelty/Impact) 1-10

| 分数 | 标准 |
|:---:|:---|
| 9-10 | 开创性工作：定义新问题/新概念/桥接两个独立社区，有望发 top venue |
| 7-8 | 重要贡献：在已知方向上提出新颖方法或理论结果，有明确的理论/实际意义 |
| 5-6 | 增量贡献：已有框架的合理扩展或新应用场景 |
| 3-4 | 边际贡献：技术组合式创新，新颖度有限 |

### 可行性 (Feasibility) 1-10

| 分数 | 标准 |
|:---:|:---|
| 9-10 | 工具链完整、理论框架清晰，6个月内可出初步结果 |
| 7-8 | 技术路线明确，有一定挑战但可预见解决方案，1年内可完成 |
| 5-6 | 核心技术挑战明确但解决路径不完全清晰，需要探索性研究 |
| 3-4 | 有重大技术障碍（如可扩展性、可计算性），需要突破性进展 |

### 综合推荐

```
S = 科研价值 >= 9 且 可行性 >= 7，或两者之和 >= 16
A = 科研价值 >= 8 且 可行性 >= 6，或两者之和 >= 14
B = 其他
```

---

## 使用示例 / Examples

### 基本用法

```
/ideas xiang-yin
```

生成 20 个 idea，基于殷翔教授的研究画像，检索最近 3 个月 arxiv 论文，全时段查重后输出到 `ideas/` 目录。

### 自定义参数

```
/ideas xiang-yin --count 10 --months 2 --save ./my-ideas
```

生成 10 个 idea，检索最近 2 个月论文，保存到 `./my-ideas/` 目录。

### 仅查重已有 idea

```
/ideas xiang-yin --verify-only
```

对 `ideas/` 目录中最新的 idea 文件执行全时段查重。

### 无指定导师（自动检测）

```
/ideas
```

自动搜索 `~/.claude/skills/` 中的导师 skill，若只有一个则直接使用，多个则让用户选择。

---

## 与导师 Skill 的关系 / Relationship with Mentor Skills

本 skill 是导师 skill 的 **下游消费者**：

```
distill-mentor → 生成导师 skill (xiang-yin/SKILL.md)
                       ↓
ideas skill   → 读取导师研究画像 → 生成 idea → 查重 → 输出
```

**前置依赖**：需要先通过 `distill-mentor` 生成至少一个导师 skill。

**信息提取**：从导师 SKILL.md 中提取以下结构化信息：
- `## 最新研究兴趣` — 热门方向列表
- `## 代表性贡献` — 核心优势领域
- `## 研究风格` / `## 你的研究风格` — 理论/应用偏好
- `## 典型回复模式` — 审阅和讨论 idea 的模式

---

## 注意事项 / Important Notes

1. **查重的局限性**：arxiv 搜索无法覆盖所有期刊/会议论文，建议在正式启动研究前再做一轮针对性 literature search
2. **时效性**：research landscape 变化快，生成的 idea 建议在 1-2 个月内评估启动
3. **导师视角**：idea 的评分基于导师的研究偏好和审美，不代表绝对的学术价值评判
4. **隐私**：不上传导师 skill 中的个人信息到外部 API，仅用于本地 idea 生成

---

## 版本历史 / Version History

- **1.0.0** (2026-04-06): Initial release
  - 5-phase workflow: load → survey → generate → verify → revise
  - Parallel arxiv search via Task subagents
  - Full-period novelty verification
  - Structured output with scoring and ranking
