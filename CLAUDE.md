# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## What This Is

A **Claude Code skill** (`/writing-cc`) that produces high-quality, research-backed written documents through an automated pipeline: requirements analysis → rubric derivation → research + drafting → iterative GPT review and revision (4-5 rounds). The core logic lives entirely in `writing-cc/SKILL.md`.

## Installation

```bash
# Install skill
cp -r writing-cc ~/.claude/skills/

# Set up Codex MCP for external reviewer (GPT-4o)
npm install -g @openai/codex
claude mcp add codex -s user -- codex mcp-server
```

Place local reference materials (papers, notes) in `papers/` and `literature/` for grounding during research.

## Usage

```bash
/writing-cc "document request"
```

**Example:**
```bash
/writing-cc 我现在需要总结一下计算机视觉、机器学习顶会论文Introduction的写作秘诀，请你参考近几年CVPR、NeurIPS、ICLR的Best Paper，以及Kaiming He的过往优秀论文，细致总结出来好的Introduction应该怎么写,有些失效链接打不开就不用强行打开，最终目的是总结规律，不要为了找论文而浪费时间
```

## Architecture

**6-phase iterative pipeline** (max 5 review rounds, exits early if overall score >= 8.5):

1. **Phase 0** — Analyze requirements: extract goal, audience, key topics, sources, constraints → `writing-logs/phase0-requirements.md`
2. **Phase 1** — Derive evaluation rubric: 5-7 measurable criteria tailored to the specific document request → `writing-logs/phase1-rubric.md`
3. **Phase 2** — Research (web search + local files) + write first draft → `writing-logs/round-0-draft.md`
4. **Phase 3** — Send rubric + draft to GPT-4o via Codex MCP (`mcp__codex__codex`) for structured scoring and feedback; save `threadId`
5. **Phase 4** — Parse review, apply revisions, Goal Check before each round → `writing-logs/round-N-revision.md`
6. **Phase 5** — Re-evaluate via same Codex thread (`mcp__codex__codex-reply`) → `writing-logs/round-N-review.md`
7. **Phase 6** — Generate clean `FINAL_DOCUMENT.md` and full `WRITING_REPORT.md`

Key config: `REVIEWER_MODEL = gpt-4o`, `MAX_ROUNDS = 5`, `SCORE_THRESHOLD = 8.5`.

## Output Files

All outputs go to `writing-logs/` (created at runtime in the working directory):

| File | Content |
|------|---------|
| `phase0-requirements.md` | Requirements analysis: goal, audience, key topics, sources |
| `phase1-rubric.md` | Derived evaluation rubric with criteria and weights |
| `round-0-draft.md` | Initial research-backed draft |
| `round-N-review.md` | GPT review for round N (with raw response) |
| `round-N-revision.md` | Revised document for round N + change log |
| `score-history.md` | Score evolution table across all rounds |
| `FINAL_DOCUMENT.md` | Clean final document ready to share |
| `WRITING_REPORT.md` | Full run report: scores, changes, raw reviews |

## Key Design Decisions

- **Rubric derived from requirements**: the evaluation criteria are custom-generated from what the user actually asked for — not a generic quality checklist
- **Grounded writing**: every example and claim must be traceable to a real source; fabricated examples are a blocker for READY verdict
- **Full document each round**: entire document is re-sent to GPT each round (not diffs) since GPT cannot read local files
- **Same Codex thread**: reused across all rounds so the reviewer has context continuity
- **Goal anchor**: the Core Goal from Phase 0 is copied verbatim into every revision — the reviewer is explicitly prevented from pulling the document off-topic
- **Pushback logic**: the skill may decline reviewer suggestions that deviate from the goal or audience

## Sibling Skills

```
/research-lit  →  /writing-cc (this)  →  /paper-write
                                       →  /paper-plan
```
