# Writing-CC

[中文](#中文) | [English](#english)

---

## 中文

### 简介

Writing-CC 是一个 Claude Code 技能（skill），通过自动化流水线生成高质量的研究型文档：需求分析 → 评分标准制定 → 研究与初稿撰写 → 多轮 GPT 迭代审阅与修订。

### 核心特性

- **需求驱动**：先理解用户真正需要什么，再开始写作
- **标准评估**：基于具体需求定制5-7个可衡量的评估标准
- **研究支撑**：所有例子和论断必须来源于真实来源（网页搜索、本地文件、论文）
- **迭代优化**：通过 GPT-5.4 xHigh 进行结构化评分和反馈，循环修订直到达标
- **全程追踪**：保留完整的需求分析、评分标准、每轮修订记录

### 安装

```bash
# 安装 skill
cp -r writing-cc ~/.claude/skills/

# 设置 Codex MCP 用于外部审阅（GPT-5.4）
npm install -g @openai/codex
claude mcp add codex -s user -- codex mcp-server
```

### 使用

```bash
/writing-cc "你的文档需求"
```

**示例：**
```bash
/writing-cc 我现在需要总结一下计算机视觉、机器学习顶会论文Introduction的写作秘诀，请你参考近几年CVPR、NeurIPS、ICLR的Best Paper，以及Kaiming He的过往优秀论文，细致总结出来好的Introduction应该怎么写
```

### 工作流程

```
用户输入（文档需求）
  -> Phase 0: 需求分析
  -> Phase 1: 制定评估标准
  -> Phase 2: 研究 + 撰写初稿
  -> Phase 3: GPT 评分 + 反馈
  -> Phase 4: 解析反馈 + 修订
  -> Phase 5: 重新评估
  -> 重复 Phase 4-5 直到分数 >= 8.5 或达到最大轮次
  -> Phase 6: 生成最终文档
```

### 输出文件

所有输出保存在 `writing-logs/` 目录：

| 文件 | 内容 |
|------|------|
| `phase0-requirements.md` | 需求分析：目标、受众、关键点、来源 |
| `phase1-rubric.md` | 评估标准（含权重） |
| `round-0-draft.md` | 初始初稿 |
| `round-N-review.md` | 第 N 轮 GPT 审阅 |
| `round-N-revision.md` | 第 N 轮修订文档 + 变更日志 |
| `score-history.md` | 分数演变表 |
| `FINAL_DOCUMENT.md` | 干净的最终文档 |
| `WRITING_REPORT.md` | 完整运行报告 |

### 配置参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `REVIEWER_MODEL` | `gpt-5.4` | 审阅模型 |
| `model_reasoning_effort` | `xhigh` | 推理强度 |
| `MAX_ROUNDS` | `5` | 最大迭代轮次 |
| `SCORE_THRESHOLD` | `8.5` | 提前停止分数阈值 |

### 配套技能

```
/research-lit  ->  /writing-cc  ->  /paper-write
                                ->  /paper-plan
```

---

## English

### Overview

Writing-CC is a Claude Code skill that produces high-quality, research-backed written documents through an automated pipeline: requirements analysis → rubric derivation → research + drafting → iterative GPT review and revision.

### Key Features

- **Requirements First**: Understand what the user actually needs before writing
- **Criteria-Driven Quality**: 5-7 measurable evaluation criteria tailored to specific needs
- **Grounded Writing**: Every claim and example must trace to real sources (web search, local files, papers)
- **Iterative Refinement**: Structured scoring and feedback via GPT-5.4 xHigh, iterating until quality threshold is met
- **Full Traceability**: Complete records of requirements, rubrics, and revisions across all rounds

### Installation

```bash
# Install skill
cp -r writing-cc ~/.claude/skills/

# Set up Codex MCP for external reviewer (GPT-5.4)
npm install -g @openai/codex
claude mcp add codex -s user -- codex mcp-server
```

### Usage

```bash
/writing-cc "your document request"
```

**Example:**
```bash
/writing-cc Summarize the writing secrets of Introduction sections in CV/ML top conference papers, referencing recent Best Papers from CVPR, NeurIPS, ICLR and outstanding papers by Kaiming He
```

### Workflow

```
User Input (Document Request)
  -> Phase 0: Analyze Requirements
  -> Phase 1: Derive Evaluation Rubric
  -> Phase 2: Research + Write First Draft
  -> Phase 3: GPT Scoring + Feedback
  -> Phase 4: Parse Feedback + Revise
  -> Phase 5: Re-evaluate
  -> Repeat Phase 4-5 until score >= 8.5 or MAX_ROUNDS reached
  -> Phase 6: Generate Final Document
```

### Output Files

All outputs go to `writing-logs/`:

| File | Content |
|------|---------|
| `phase0-requirements.md` | Requirements analysis |
| `phase1-rubric.md` | Evaluation rubric with weights |
| `round-0-draft.md` | Initial draft |
| `round-N-review.md` | Round N GPT review |
| `round-N-revision.md` | Round N revision + change log |
| `score-history.md` | Score evolution table |
| `FINAL_DOCUMENT.md` | Clean final document |
| `WRITING_REPORT.md` | Full run report |

### Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `REVIEWER_MODEL` | `gpt-5.4` | Reviewer model |
| `model_reasoning_effort` | `xhigh` | Reasoning effort level |
| `MAX_ROUNDS` | `5` | Maximum review iterations |
| `SCORE_THRESHOLD` | `8.5` | Early exit score threshold |

### Related Skills

```
/research-lit  ->  /writing-cc  ->  /paper-write
                                ->  /paper-plan
```

---

## License

MIT License

---

*This README is bilingual (Chinese & English).*
