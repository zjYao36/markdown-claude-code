---
name: writing-cc
description: 'Automated document writing assistant with iterative GPT review. Use when the user says "写文档", "写总结", "帮我写", "summarize", "write a guide", "document", "/writing-cc", or wants to produce a high-quality written document on any topic through research, structured writing, and multi-round external review. The skill analyzes requirements, derives evaluation metrics, researches and writes the document, then iteratively improves it via Codex/GPT scoring and feedback for 4-5 rounds.'
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, WebSearch, WebFetch, Agent, mcp__codex__codex, mcp__codex__codex-reply
---

# Writing-CC: Automated Document Writing with Iterative Review

Write and refine: **$ARGUMENTS**

## Overview

Use this skill when the user wants a polished, well-researched written document on any topic. The skill does not simply write and hand off. It first understands what makes the document valuable, derives measurable evaluation criteria, writes a grounded first draft, then runs it through an external GPT reviewer using those exact criteria — iterating until the document meets the bar or MAX_ROUNDS is reached.

Three principles dominate:

1. **Requirements first, writing second.** Extract what the user actually needs before producing a single word of content.
2. **Criteria-driven quality.** Every evaluation round uses the same rubric derived from the user's requirements — not generic writing advice.
3. **Grounded writing.** All claims and examples must come from real sources: web search, local materials, or cited papers. Never fabricate examples.

```
User input (document request)
  -> Phase 0 (Claude): Analyze requirements — purpose, audience, key points, constraints
  -> Phase 1 (Claude): Derive evaluation rubric (5-7 measurable criteria) from requirements
  -> Phase 2 (Claude): Research (web search + local materials) + write first draft
  -> Phase 3 (Codex/GPT): Score draft on rubric, provide structured feedback
  -> Phase 4 (Claude): Parse feedback, revise document
  -> Phase 5 (Codex, same thread): Re-score revised document
  -> Repeat Phase 4-5 until OVERALL SCORE >= SCORE_THRESHOLD or MAX_ROUNDS reached
  -> Phase 6: Save full history to writing-logs/
```

## Constants

- **REVIEWER_MODEL = `gpt-5.4`** — Reviewer model used via Codex MCP.
- **MAX_ROUNDS = 5** — Maximum review-revise cycles.
- **SCORE_THRESHOLD = 8.5** — Minimum overall score to stop early.
- **OUTPUT_DIR = `writing-logs/`** — Directory for all output files.
- **MAX_SEARCH_QUERIES = 10** — Maximum web search queries during research.
- **MAX_LOCAL_FILES = 10** — Maximum local files to scan for grounding.

> Override via argument if needed, e.g. `/writing-cc "topic" -- max rounds: 3, threshold: 8`.

## Output Structure

```
writing-logs/
├── phase0-requirements.md        # Analyzed requirements and key points
├── phase1-rubric.md              # Derived evaluation rubric
├── round-0-draft.md              # Initial draft
├── round-1-review.md             # Round 1 external review
├── round-1-revision.md           # Round 1 revised document
├── round-2-review.md
├── round-2-revision.md
├── ...
├── score-history.md              # Score evolution table
├── FINAL_DOCUMENT.md             # Clean final version
└── WRITING_REPORT.md             # Full run report with all reviews
```

---

## Workflow

### Phase 0: Analyze Requirements

Before writing anything, understand precisely what the user needs. This analysis is the foundation for the rubric and the writing itself.

Extract and save to `writing-logs/phase0-requirements.md`:

```markdown
# Requirements Analysis

## User Request
[Verbatim user input]

## Document Purpose
What is this document for? (tutorial, reference, persuasion, summary, guide, analysis, etc.)

## Target Audience
Who will read this? What do they already know? What do they need?

## Core Goal
In one sentence: what must a reader be able to do, know, or feel after reading this?

## Key Topics / Coverage Areas
List the 5-10 most important things the document must cover to fulfill the goal.
(These become the basis for rubric criteria.)

## Sources & Materials to Use
- Specific papers, authors, venues, or datasets the user named
- Local files to check: papers/, literature/, any other local dirs
- Web searches to run

## Constraints
- Length / format requirements
- Tone (technical, accessible, formal, etc.)
- Any explicit exclusions the user mentioned
- Deadline or urgency

## Success Condition
What would make the user say "yes, this is exactly what I needed"?
```

**CRITICAL**: The Core Goal, Key Topics, and Success Condition are immutable anchors. Carry them into every round.

---

### Phase 1: Derive Evaluation Rubric

Based on Phase 0, derive 5-7 measurable evaluation criteria tailored to this specific document request. Generic criteria like "grammar" or "formatting" are only included if the user explicitly cares about them. The rubric must test whether the document fulfills the user's actual goal.

Save to `writing-logs/phase1-rubric.md`:

```markdown
# Evaluation Rubric

## Document Goal (Anchor)
[One-sentence goal from Phase 0]

## Evaluation Criteria

| # | Criterion | Description | Weight |
|---|-----------|-------------|--------|
| 1 | [Name] | [What a high-scoring document does for this criterion] | [e.g., 20%] |
| 2 | [Name] | [Description] | [%] |
| ...| ...       | ...         | ...    |

## Scoring Guide
- 9-10: Fully addresses criterion with concrete, accurate, well-organized content
- 7-8: Mostly addresses criterion with minor gaps or imprecision
- 5-6: Partially addresses criterion; notable gaps or errors
- 3-4: Criterion barely met; significant missing content
- 1-2: Criterion not met

## Overall Score Formula
OVERALL = weighted average of all criteria scores
```

**Example rubric criteria for the use case `/writing-cc 总结CV/ML论文Introduction写作秘诀`:**
- Coverage of structural patterns (narrative arc, problem motivation, gap, contribution)
- Quality and specificity of examples from real top papers
- Actionability (can a writer follow this as a guide?)
- Accuracy of paper references and claims
- Depth vs breadth balance (insightful, not a surface-level checklist)
- Audience fit (for ML researchers writing papers)

---

### Phase 2: Research and Write First Draft

#### Step 2.1: Gather Source Materials

**Local materials first**: Check `papers/`, `literature/`, and any path the user mentioned. Read relevant sections.

**Web search**: Run targeted searches based on the Key Topics and Sources from Phase 0. For each search:
- Extract specific, citable evidence (paper titles, author names, specific quotes or structures)
- Note the source URL and content

**Minimum bar**: Each major claim or example in the draft must be traceable to a real source found in this step. Never invent a paper title, author, or example.

#### Step 2.2: Write the First Draft

Write a complete, well-structured document. Not an outline. Not a skeleton. A full draft a reader could actually use.

The draft must:
- Open with a clear purpose statement (what this document is and why it exists)
- Cover all Key Topics from Phase 0 in logical order
- Include specific, concrete examples from real sources found in Step 2.1
- Be written at the appropriate level for the Target Audience
- End with actionable takeaways or a summary appropriate to the document type

Save to `writing-logs/round-0-draft.md`:

```markdown
# [Document Title]

> **Purpose**: [one-line purpose statement]
> **Audience**: [target audience]
> **Sources**: [key sources consulted]

---

[Full document content]

---

## Source References
- [Source 1]: [URL or citation]
- [Source 2]: [URL or citation]
...
```

---

### Phase 3: External Review (Round 1)

Send the rubric + draft to GPT for evaluation. The reviewer must score using the exact rubric derived in Phase 1 — not generic writing quality.

```
mcp__codex__codex:
  model: REVIEWER_MODEL
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    You are an expert reviewer evaluating a written document against a specific rubric.
    Your goal is to give actionable, evidence-based feedback that helps improve the document
    toward its stated purpose — not to rewrite it in a different style.

    Do NOT penalize the document for not covering things the user did not ask for.
    Do NOT reward length or formal-sounding language.
    DO reward specificity, accuracy of examples, and genuine usefulness to the target reader.

    === DOCUMENT GOAL ===
    [Paste Core Goal from Phase 0]
    === END GOAL ===

    === EVALUATION RUBRIC ===
    [Paste full rubric from Phase 1]
    === END RUBRIC ===

    === DOCUMENT TO REVIEW ===
    [Paste the FULL draft]
    === END DOCUMENT ===

    For each rubric criterion:
    - Score: X/10
    - Strengths: what the document does well on this criterion
    - Weaknesses: specific gaps, errors, or missing content
    - Concrete fix: one actionable improvement instruction (not a rewrite, an instruction)

    Then provide:
    - **OVERALL SCORE**: X/10 (weighted average per rubric)
    - **Top 3 Priority Fixes** (ranked by impact on overall score):
      1. [Highest impact fix]
      2. [Second fix]
      3. [Third fix]
    - **Content Accuracy Issues** (if any): specific factual errors or fabricated examples to correct
    - **Verdict**: READY / REVISE / MAJOR_REVISION
      - READY: overall >= 8.5, no accuracy issues, all criteria >= 7
      - REVISE: good direction, targeted improvements needed
      - MAJOR_REVISION: structural or accuracy problems require significant rework
```

**CRITICAL: Save the `threadId`** from this call for all later rounds.
**CRITICAL: Save the FULL raw response** verbatim.

Save to `writing-logs/round-1-review.md` with raw response in a `<details>` block.

---

### Phase 4: Parse Feedback and Revise

#### Step 4.1: Parse the Review

Extract from the review:
- Per-criterion scores and specific feedback
- Overall score
- Verdict
- Top 3 priority fixes
- Any content accuracy issues
- Round number

Update `writing-logs/score-history.md`:

```markdown
# Score History

| Round | [Criterion 1] | [Criterion 2] | ... | Overall | Verdict |
|-------|---------------|---------------|-----|---------|---------|
| 1     | X             | X             | ... | X       | REVISE  |
```

**STOP CONDITION**: If overall score >= SCORE_THRESHOLD and verdict is READY and no accuracy issues, skip to Phase 6.

#### Step 4.2: Apply Revisions

Before changing anything, do a quick **Goal Check**:
- Does the current draft still serve the Core Goal?
- Which reviewer suggestions would improve the document toward the goal?
- Which suggestions would pull the document off-topic or change its scope?

Then apply revisions:
- **Content gaps**: add the missing coverage the reviewer identified
- **Accuracy issues**: fix or remove any fabricated/incorrect examples; do a targeted search if needed
- **Structural issues**: reorganize sections if the reviewer identified a logical flow problem
- **Pushback**: if a reviewer suggestion would make the document worse for the Target Audience or deviate from the goal, note it and decline with a brief reason

Save to `writing-logs/round-N-revision.md`:

```markdown
# Round N Revision

## Goal Anchor
[Copy Core Goal verbatim from Phase 0]

## Reviewer Scores This Round
| Criterion | Score | Key Feedback |
|-----------|-------|--------------|
| ...       | ...   | ...          |
| **Overall** | **X** | **Verdict: ...** |

## Changes Made
### Fix 1: [Criterion / Issue]
- Reviewer said: [specific feedback]
- Action taken: [what was changed and why]

### Fix 2: ...

### Declined Suggestions
- [Suggestion]: [reason for declining]

## Revised Document
[Full revised document — complete, not incremental patches]
```

---

### Phase 5: Re-evaluation (Round 2+)

Send the revised document back to GPT **in the same thread**:

```
mcp__codex__codex-reply:
  threadId: [saved from Phase 3]
  model: REVIEWER_MODEL
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Round N re-evaluation]

    I revised the document based on your feedback. Here are the key changes:
    1. [Change 1 — criterion addressed + what was done]
    2. [Change 2]
    3. [Change 3]
    [If any suggestions were declined, briefly note: "Declined [X] because [Y]"]

    === REVISED DOCUMENT ===
    [Paste the FULL revised document]
    === END REVISED DOCUMENT ===

    Please re-score against the same rubric:
    - Score each criterion again (X/10) with brief note on what improved or still needs work
    - OVERALL SCORE
    - Updated Top 3 Priority Fixes (or "NONE" if READY)
    - Any remaining content accuracy issues
    - Verdict: READY / REVISE / MAJOR_REVISION
```

Save review to `writing-logs/round-N-review.md`.

Return to Phase 4 until:
- **Overall score >= SCORE_THRESHOLD** and verdict is READY
- or **MAX_ROUNDS reached**

---

### Phase 6: Final Report and Clean Output

#### Step 6.1: Write `writing-logs/FINAL_DOCUMENT.md`

The clean, final version of the document. No review history, no round markers, no scaffolding — just the final document a reader would actually use.

```markdown
# [Document Title]

[Final complete document content]

---

## Sources
[All cited sources from the research phase]
```

If the final verdict is not READY, still write the best current version here.

#### Step 6.2: Write `writing-logs/WRITING_REPORT.md`

```markdown
# Writing Report

**Topic**: [user's request]
**Date**: [today]
**Rounds**: N / MAX_ROUNDS
**Final Score**: X / 10
**Final Verdict**: [READY / REVISE / MAJOR_REVISION]

## Document Goal (Anchor)
[Verbatim from Phase 0]

## Evaluation Rubric Summary
[Criteria names and weights]

## Score Evolution

| Round | [Crit 1] | [Crit 2] | ... | Overall | Verdict |
|-------|----------|----------|-----|---------|---------|
| 1     | ...      | ...      | ... | ...     | ...     |
| N     | ...      | ...      | ... | ...     | ...     |

## Key Improvements Per Round

| Round | Main Issues Found | Changes Made | Impact |
|-------|-------------------|--------------|--------|
| 1     | [top issues]      | [fixes]      | [score delta] |

## Content Accuracy Log
- [Any fabricated/incorrect examples that were fixed, and what replaced them]
- NONE if no accuracy issues arose

## Declined Suggestions
| Round | Suggestion | Reason Declined |
|-------|-----------|-----------------|

## Remaining Weaknesses
[Honest unresolved issues, if any]

## Raw Reviewer Responses

<details>
<summary>Round 1 Review</summary>

[Full verbatim GPT response]

</details>

<details>
<summary>Round 2 Review</summary>

[Full verbatim GPT response]

</details>

...

## Output Files
- Final document: `writing-logs/FINAL_DOCUMENT.md`
- Score history: `writing-logs/score-history.md`
- Phase 0 requirements: `writing-logs/phase0-requirements.md`
- Rubric: `writing-logs/phase1-rubric.md`
```

#### Step 6.3: Present Summary to User

```
Writing complete after N rounds.

Final score: X/10 (Verdict: READY / REVISE / MAJOR_REVISION)

Document: writing-logs/FINAL_DOCUMENT.md

Score evolution:
  Round 1: X/10 → Round N: X/10

Key improvements made:
- [improvement 1]
- [improvement 2]

Remaining concerns:
- [if any, else "None"]

Full report: writing-logs/WRITING_REPORT.md
```

---

## Key Rules

- **Goal anchor first, every round.** The Core Goal from Phase 0 is copied verbatim into every revision.
- **Rubric-driven review.** Every reviewer call uses the specific rubric from Phase 1. Do not let the reviewer improvise generic writing quality scores.
- **Grounded writing only.** Every example and claim must come from a real source found during research. If you cannot find a real example, say so and omit rather than fabricate.
- **Full document every round.** Each round-N-revision.md contains the complete revised document, not a patch list.
- **Save `threadId` from Phase 3** and use `mcp__codex__codex-reply` for all subsequent rounds.
- **ALWAYS use `config: {"model_reasoning_effort": "xhigh"}`** for all Codex review calls.
- **Pushback is allowed.** If the reviewer suggests something that conflicts with the goal or audience, decline it and explain why.
- **Content accuracy is non-negotiable.** Any fabricated paper title, author, or example found by the reviewer must be fixed before the next round — even if it means doing another web search.
- **Do not pad for length.** A shorter, accurate, actionable document beats a longer padded one.

---

## Composing with Other Skills

This skill produces polished documents. It works best when combined with:

```
/writing-cc "[document request]"    <- you are here
  -> writing-logs/FINAL_DOCUMENT.md  (ready to share, publish, or include in a paper)

Related skills:
  /research-lit   -> find and summarize papers before writing
  /paper-write    -> write LaTeX sections for academic papers
  /paper-plan     -> plan a full academic paper structure
```

Typical flow:

1. User has a document to write (summary, guide, tutorial, analysis, etc.)
2. `/writing-cc` turns the request into a research-backed, rubric-validated final document
3. If the output is for a paper, hand off to `/paper-write` for LaTeX formatting
4. If the topic needs deeper literature grounding first, run `/research-lit` beforehand

---

## Example Invocations

```bash
# Summarize writing tips for ML paper introductions
/writing-cc 我现在需要总结一下计算机视觉、机器学习顶会论文Introduction的写作秘诀，请你参考近几年CVPR、NeurIPS、ICLR的Best Paper，以及Kaiming He的过往优秀论文，细致总结出来好的Introduction应该怎么写

# Write a technical tutorial
/writing-cc Write a practical guide to using LoRA fine-tuning for image generation models, targeting ML engineers who know transformers but are new to diffusion models

# Produce a project summary document
/writing-cc 帮我整理总结我们项目的技术方案，重点说清楚创新点和与现有方法的区别，读者是组内成员

# Write a literature review section
/writing-cc Summarize the state of video generation models as of 2024-2025, focusing on autoregressive vs diffusion approaches and key benchmark results
```
