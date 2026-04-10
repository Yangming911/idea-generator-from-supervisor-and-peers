---
name: scout-peers
description: "Discover complementary peer research groups based on a mentor's frontier research directions. | 发现与导师前沿方向互补的优秀课题组，输出结构化 MD 供 /ideas 消费。"
argument-hint: "[<mentor-skill-name>] [--count <number>] [--save <directory>]"
version: "1.0.0"
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, Task
---

> **Language / 语言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout.
>
> 本 Skill 支持中英文。根据用户第一条消息的语言，全程使用同一语言回复。

# Scout Peers — 互补课题组发现

基于导师数字分身 Skill 的最新研究兴趣，通过多源搜索发现与导师前沿方向高度相关但方法互补的优秀课题组，输出结构化 MD 文档。

## 触发条件 / Trigger Conditions

Activate when the user says:
- `/scout-peers` or `/scout-peers <mentor-skill-name>`
- `/scout-peers xiang-yin --count 12`
- "帮我找相关课题组" / "发现互补课题组"
- "Find peer research groups for <mentor>"

## 参数说明 / Parameters

| 参数 | 说明 | 默认值 |
|:---|:---|:---|
| `<mentor-skill-name>` | 导师 skill 名称（如 `xiang-yin`） | 自动检测 `~/.claude/skills/` 中的导师 skill |
| `--count <N>` | 最终输出的课题组数量 | 8-12（自动根据质量调整） |
| `--save <dir>` | 保存目录 | 当前工作目录下的 `peers/` |

## 核心理念 / Core Philosophy

**为什么需要 peer 课题组？**

导师的 idea 生成不应只是"旧专长 A + 新方向 B"的纵向组合。更有价值的创新往往来自：
- 不同课题组在**同一前沿方向**上的**不同方法路径**的碰撞
- 看到别人正在做什么 → 发现自己的独特切入角度
- 两个组的**新工作**之间的横向联结，而非一个组的旧工作与新趋势的拼接

**什么是好的 peer 课题组？**
- ✅ 研究同一前沿方向，但用不同的方法/工具/理论框架（互补性最高）
- ✅ 解决相关但不同的问题，方法上有潜在协同
- ✅ 直接竞争者（方向和方法高度重叠）——了解竞争者的最新进展同样重要
- ✅ 密切合作者——已有合作基础，可能有深度协作的新可能
- ❌ 领域相距太远（无法产生有意义的交叉）

---

## 完整工作流程 / Complete Workflow

本 skill 严格按以下 4 个阶段执行，每个阶段完成后向用户汇报进度。

---

### Phase 1: 提取导师前沿画像 / Extract Mentor Frontier Profile

**目标**：从导师 skill 中提取最新研究兴趣和已知合作者，构建搜索基础。

**步骤**：

1. 定位导师 skill 文件：
   - 若指定 `<mentor-skill-name>`：读取 `~/.claude/skills/<name>/SKILL.md`
   - 若未指定：扫描 `~/.claude/skills/` 中的所有 skill，排除 `scout-peers`、`ideas`、`distill-mentor` 等工具 skill，若只有一个导师 skill 则直接使用，多个则让用户选择
2. 从 SKILL.md 中提取：
   - **最新研究兴趣**：定位 `## 最新研究兴趣` 或类似章节，提取所有方向及其热度排序（关注 🔥 标记）
   - **近期代表性论文**：提取最近 2 年的代表性工作（标题 + 合作者）
   - **核心关键词**：从研究兴趣和论文标题中提取技术关键词
   - **已知合作者**：从代表性论文中收集共同作者名单（用于后续排除）
3. 输出摘要：

```
📋 导师前沿画像提取完成：
- 导师：<name> (<institution>)
- 前沿方向（共 <N> 个）：
  🔥🔥🔥 <direction-1>
  🔥🔥 <direction-2>
  🔥 <direction-3>
  ...
- 核心关键词：<keyword-1>, <keyword-2>, ...
- 已知合作者：<N> 人（将排除）
```

---

### Phase 2: 多源候选发现 / Multi-Source Candidate Discovery

**目标**：通过三条并行线索发现候选课题组 PI，合并去重后取 top 15-20。

**三条线索并行执行**（使用 Task tool 的 subagent）：

#### 线索 A — 前沿方向活跃作者搜索

对导师的每个 🔥 方向：
1. 构造 2-3 个精准搜索查询，搜 arxiv 近 1-2 年论文
   - 查询应使用方向的核心技术关键词（非导师姓名）
   - 示例：`"conformal prediction" "control barrier function" arxiv 2025 2026`
2. 从搜索结果中提取论文作者
3. 统计每个作者在各方向的出现频次
4. 排除导师本人及已知合作者

**输出**：候选人列表 + 每人关联的方向 + 出现频次

#### 线索 B — 引用网络扩展

1. 选取导师 3-5 篇最重要/最新的论文
2. 用 WebSearch 搜索这些论文的引用情况：
   - `"<paper-title>" cited by` 或在 Semantic Scholar/Google Scholar 上查找
3. 也可用 WebFetch 直接调 Semantic Scholar API：
   - `https://api.semanticscholar.org/graph/v1/paper/search?query=<title>&fields=citations.authors`
4. 从引用论文的作者中提取高频非合作者

**输出**：候选人列表 + 引用关系描述

#### 线索 C — 领域综述与 Workshop 搜索

1. 对每个 🔥 方向搜索：
   - `"<direction keywords>" survey review 2024 2025 2026`
   - `"<direction keywords>" workshop tutorial <key-venue>`
2. 从 survey 论文的参考文献中识别被高频引用的课题组
3. 从 workshop 组织者/受邀报告人中识别活跃 PI
4. 搜索导师关注的核心会议（如 IEEE CDC, NeurIPS, ICRA）的近期 proceedings 中相关 session 的作者

**输出**：候选人列表 + 发现来源

#### 合并与去重

1. 合并三条线索的候选人
2. 去除：导师本人
3. 按综合出现频次排序（跨多条线索出现的优先）
4. 取 top 15-20 候选人进入下一阶段

**汇报**：

```
🔍 候选发现完成：
- 线索 A（关键词搜索）：发现 <N> 位候选人
- 线索 B（引用网络）：发现 <M> 位候选人
- 线索 C（综述/Workshop）：发现 <K> 位候选人
- 合并去重后：<T> 位候选人进入画像提取
```

---

### Phase 3: 候选画像提取 / Candidate Profiling

**目标**：对 top 15-20 候选人提取结构化画像。

**并行执行**（使用 Task subagent，每 5 人一组并行）：

对每位候选人：

1. **WebSearch** 搜索：`"<candidate-name>" "<institution>" research` 找到个人主页
2. **WebFetch** 抓取个人主页，提取：
   - 全名、机构、职位
   - 研究方向描述（2-3 句话）
   - 实验室名称和网址（如有）
3. **WebSearch** 搜索其近期论文：`"<candidate-name>" arxiv 2025 2026`
4. 提取近 2 年的 3-5 篇代表性论文（标题 + 一句话摘要）
5. 识别其核心方法/工具/理论框架（与导师的做对比）

**每位候选人的画像格式**：

```yaml
name: <全名>
institution: <机构>
position: <职位>
core_focus: <2-3 句核心研究方向>
key_methods: [<method-1>, <method-2>, ...]
recent_papers:
  - title: <论文标题>
    venue: <发表场所>
    year: <年份>
    summary: <一句话摘要>
  - ...
homepage: <URL>
```

**汇报**：

```
📊 候选画像提取完成：
- 成功提取：<N> / <total> 位
- 失败/信息不足：<M> 位（已排除）
```

---

### Phase 4: 互补性评估与报告生成 / Complementarity Assessment & Report

**目标**：筛选最具互补性的 8-12 个课题组，生成最终报告。

#### 互补性评估

对每位候选人，基于 Phase 1 的导师画像和 Phase 3 的候选画像，评估：

1. **方向相关性**（是否研究相关前沿方向）
   - 高：研究同一 🔥 方向的核心问题
   - 中：研究相关但不完全相同的方向
   - 低：研究领域相距较远 → 排除

2. **近期活跃度**（近 2 年的研究产出）
   - 高：持续高质量产出，在前沿方向有新突破
   - 中：有产出但不算突出
   - 低：近期不太活跃 → 降低优先级

**筛选规则**：
- 方向相关性为"低"的直接排除
- 按导师的 🔥 方向分组，每个方向至少保留 1-2 个课题组
- 最终保留 8-12 个课题组

#### 报告生成

生成最终 MD 文件，保存到 `<save-dir>/peers_<mentor>_<YYYY-MM-DD>.md`：

```markdown
# 互补课题组：<导师姓名>

> 生成日期：<date>
> 基于导师 Skill：<mentor-skill-path>
> 发现课题组：<N> 个
> 搜索范围：arxiv + Semantic Scholar + 个人主页 + 会议/Workshop

## 导师前沿方向概览

<简要列出导师当前 🔥 方向，每个方向 1 句话描述>

---

## 按方向分组的互补课题组

### 方向：<导师 🔥 方向 1 名称>

#### <PI 姓名> — <机构>
- **核心方向**：<2-3 句>
- **研究特色**：<该组的独特方法/视角，1-2 句>
- **近期代表作**：
  1. <论文标题> (<venue>, <year>) — <一句话摘要>
  2. <论文标题> (<venue>, <year>) — <一句话摘要>
  3. <论文标题> (<venue>, <year>) — <一句话摘要>
- **潜在交叉点**：<这个组的工作与导师的工作可能在哪里产生化学反应>

#### <PI 姓名 2> — <机构 2>
...

---

### 方向：<导师 🔥 方向 2 名称>

...

---

## 总览表

| # | PI | 机构 | 对齐方向 | 研究特色 | 近期活跃度 |
|:---:|:---|:---|:---|:---|:---:|
| 1 | <name> | <inst> | <direction> | <specialty> | ⭐⭐⭐ |
| 2 | ... | ... | ... | ... | ... |

---

## 最具潜力的交叉轴线

基于以上课题组的分析，以下是最可能产生高价值研究的交叉点：

1. <描述一个具体的交叉可能性>
2. <描述另一个交叉可能性>
3. ...

---

## 使用建议

本文档可作为 `/ideas` skill 的输入，为 idea 生成提供跨课题组的灵感来源。
运行 `/ideas <mentor-name>` 时，系统会自动检测并加载本文档。
```

---

## 与导师 Skill 和 Ideas Skill 的关系 / Ecosystem

```
distill-mentor → 生成导师 skill (SKILL.md)
                       ↓
scout-peers    → 读取导师画像 → 发现互补课题组 → 输出 peers MD
                       ↓                              ↓
ideas skill    → 读取导师画像 ────────────────── + 读取 peers MD → 生成 idea
```

**前置依赖**：需要先通过 `distill-mentor` 生成至少一个导师 skill。

**下游消费者**：`/ideas` skill 在 Phase 2（前沿调研）中自动检测 `peers/` 目录是否有对应导师的 peers MD，若有则作为额外灵感来源加载。

---

## 注意事项 / Important Notes

1. **合作者排除的局限**：仅排除导师代表性论文中的共同作者，可能遗漏偶尔合作过的人。用户可手动审阅并标注。
2. **作者消歧**：对于常见姓名（如 "Wei Zhang"），需结合机构信息和研究方向交叉验证，避免混淆。
3. **搜索覆盖率**：WebSearch + Semantic Scholar 无法覆盖所有论文，建议用户对感兴趣的方向做补充检索。
4. **时效性**：课题组的研究方向可能快速变化，建议每 3-6 个月重新运行。
5. **隐私**：不上传导师 skill 中的个人信息到外部 API，仅使用公开信息进行搜索。

---

## 使用示例 / Examples

### 基本用法

```
/scout-peers xiang-yin
```

基于殷翔教授的导师 skill，发现 8-12 个互补课题组，保存到 `peers/peers_xiang-yin_<date>.md`。

### 自定义参数

```
/scout-peers xiang-yin --count 15 --save ./my-peers
```

发现 15 个课题组，保存到 `./my-peers/` 目录。

### 无指定导师

```
/scout-peers
```

自动检测 `~/.claude/skills/` 中的导师 skill。

---

## 版本历史 / Version History

- **1.0.0** (2026-04-08): Initial release
  - 4-phase workflow: extract → discover → profile → filter+report
  - Multi-source discovery: keyword search + citation network + survey/workshop
  - Parallel candidate profiling via Task subagents
  - Complementarity-based filtering
  - Structured MD output for /ideas integration
