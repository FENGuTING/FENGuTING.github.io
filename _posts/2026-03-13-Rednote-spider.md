---
title: "Mining RedNote QR Problems: Building a Structured Community-Sourced Question Bank"
date: 2026-03-13 13:00:00 +0800
categories: [QR Interview]
tags: [rednote, quant-research, question-bank, ocr, nlp]
math: false
toc: true
---

## Objective

This project explores how to systematically collect, clean, and organize QR-related interview and written-test problems shared across RedNote/Xiaohongshu posts into a searchable, structured question bank.

The goal is not simply to scrape posts, but to transform scattered community-generated content into a reusable database of quant interview problems with support for deduplication, topic tagging, and future frequency analysis.

---

## Motivation

A large amount of QR interview information is now shared informally on community platforms. In particular, many RedNote posts contain:

- interview recollections,
- written-test summaries,
- screenshots of handwritten or typed problems,
- comment-section follow-ups with additional questions.

However, this information is highly fragmented. Problems are often embedded in screenshots, mixed with personal commentary, repeated across multiple posts, and rarely organized in a way that supports systematic preparation.

This makes RedNote an interesting data source: noisy, incomplete, and unstructured, but potentially rich in high-frequency interview content.

The idea of this project is therefore to treat these posts not as isolated notes, but as raw observations of the QR interview ecosystem.

---

## Problem Setting

The practical challenge is that a RedNote post is not equivalent to a question.

A single post may contain multiple problems, for example:

1. a probability puzzle,
2. a regression intuition question,
3. a coding problem,
4. a follow-up in comments.

Moreover, many of these questions appear in slightly different wordings across different posts. Some are written in Chinese, some in English, and some in mixed shorthand.

A meaningful system therefore needs to solve at least four subproblems:

1. collect post-level raw content,
2. recover text from screenshots via OCR,
3. split post content into question-level records,
4. merge semantically equivalent questions.

This makes the task closer to a lightweight information extraction pipeline than to a simple scraper.

---

## Proposed Pipeline

The full system can be viewed as a five-stage pipeline:

    collect -> OCR -> split -> normalize -> store

### 1. Collection Layer

The collection layer searches RedNote using targeted keywords such as:

- QR 笔试
- 量化笔试
- 九坤 笔试
- 宽德 笔试
- HRT OA
- Citadel interview

For each matched post, the system stores raw metadata including:

- title,
- author,
- post URL,
- text body,
- image links,
- comment snippets.

At this stage, recall matters more than precision. Some noise is acceptable as long as potentially useful posts are retained.

### 2. OCR Layer

A large fraction of useful content is embedded in screenshots rather than in main text. OCR therefore becomes a core component rather than an optional extension.

The OCR layer extracts text from images and merges it back with the original post text.

### 3. Question Splitting

After OCR, the combined text still exists at the post level. The next step is to split one post into multiple question candidates.

Typical splitting signals include:

- numbered lists such as `1.` `2.` `3.`,
- markers like `题1`, `题2`, `follow-up`,
- company labels such as `九坤:` or `HRT:`.

The output of this stage is a set of question-level records.

### 4. Normalization and Deduplication

This is the most important stage for long-term usefulness.

Many community-shared questions are repeated with slightly different phrasing. For example:

- duplicate sample 对 OLS 的影响
- 复制样本一遍后 beta 和 t-value 怎么变
- what happens to OLS estimates if the sample is duplicated

A useful question bank should map these variants into a canonical representation.

Possible approaches include:

- text cleaning,
- fuzzy matching,
- embedding similarity,
- manual review for ambiguous cases.

### 5. Storage and Export

The final structured dataset can be stored in SQLite or JSONL.

A minimal question-level schema may look like:
```json
    {
      "company": "九坤",
      "round_type": "written_test",
      "topic": "statistics",
      "question_raw": "duplicate sample 对 OLS beta 和 t-value 有何影响",
      "question_clean": "Effect of duplicating the full sample on OLS coefficients and t-statistics",
      "source_platform": "rednote",
      "source_url": "...",
      "language": "zh",
      "confidence": 0.92,
      "dedup_group": "ols_duplicate_sample"
    }
```

This schema is intentionally simple. The purpose is to support future scaling rather than to over-engineer the first version.

---

## Why This Is Interesting

What makes this project interesting is not only the engineering pipeline, but also the downstream analysis it enables.

Once a sufficiently large number of questions is collected and deduplicated, the database can answer more meaningful questions such as:

- Which problem types appear most frequently across community reports?
- Do domestic quant firms emphasize different topics from foreign firms?
- Are certain probability or statistics questions repeatedly recycled?

This turns scattered interview anecdotes into a rough observational dataset of QR hiring patterns.

The framing therefore changes from simple collection into latent structure extraction from community-generated data.

---

## Expected Challenges

Several issues make the project nontrivial.

### Noisy OCR

Screenshots are often low-resolution, annotated, or partially cropped. OCR errors can distort formulas and variable names.

### Weak Structure

Posts do not follow a consistent template. Some are highly organized, others are fragmented.

### Cross-Post Redundancy

The same classic interview questions may appear many times, often with slightly different wording.

### Source Bias

Community-shared posts are not a random sample of all interview questions. They contain reporting bias, memory bias, and popularity bias.

This means the resulting dataset should be interpreted as an observational approximation rather than a complete market picture.

---

## Initial Design Decision

For the first version, I plan to prioritize a lightweight and controllable pipeline over full automation.

The first usable version will focus on:

- keyword-based collection from a small set of targeted firms,
- OCR on saved post images,
- rule-based question splitting,
- lightweight fuzzy deduplication,
- SQLite-based storage.

This keeps the system simple enough to build quickly while making it immediately useful.

A more polished version can later add:

- semantic clustering,
- topic classification,
- frequency statistics,
- Markdown export for a public repository.

---

## Potential Outputs

If the system works as intended, it should eventually support outputs such as:

### 1. A searchable question bank

Useful for structured QR interview preparation.

### 2. A frequency-ranked list of recurring questions

For example:

- expectation recursion,
- conditional probability,
- OLS intuition,
- combinatorics,
- linear algebra basics.

### 3. Company-level topic comparison

This may reveal different emphasis across firms.

### 4. A compact research-engineering side project

Beyond interview utility, the project also functions as an exercise in OCR, normalization, weakly structured parsing, and knowledge organization.

---

## Future Work

Several extensions are natural.

First, the same framework can later be applied to additional community sources such as forums, GitHub repositories, and interview discussion boards.

Second, the normalized question bank can be linked to a solution library, allowing structured practice rather than only collection.

Third, the deduplicated dataset may support topic heatmaps, firm-level summaries, and longitudinal observations about interview trends.

---

## Closing Thoughts

This project starts from a practical preparation need, but naturally develops into a broader knowledge-mining problem.

The most interesting part is not the scraping itself. The real value lies in transforming noisy, repeated, semi-structured community content into a small layer of usable infrastructure for quant interview intelligence.
