# DataMate — Natural Language Question Answering over Structured Data

Zalo AI Challenge 2026 — **DataMate**.

An AI agent that answers everyday Vietnamese questions directly on a business's structured data (`.csv`, `.xlsx`, `.sqlite`), returns a typed answer, and declines when the data cannot support the question.

```json
{"id": "q_06de33fa21cc", "business_id": "biz_118", "question": "sp bán dc nhiều nhất tuần rồi là gì", "as_of_date": "2026-03-04"}
→
{"id": "q_06de33fa21cc", "type": "string", "answer": "Xi măng Hà Tiên PCB40 bao 50kg"}
```

## Documentation

| Document | What it covers |
|---|---|
| [description.md](description.md) | The task: motivation, input, output, worked example |
| [data.md](data.md) | Data packages, splits, the rulebook, submission format, Docker image contract, dataset downloads |
| [evaluation.md](evaluation.md) | Answer types and matching rules, ranking questions, Penalized Accuracy |
| [rule.md](rule.md) | Competition rounds and the 8 technical requirements (models, offline, compute, reproducibility) |
| [submission_guidance.md](submission_guidance.md) | Timeline, submission quotas, deadlines |
| [Docker guideline (PDF)](zac2026_DataMate_docker_guideline_final.pdf) | Official image/base-image and execution guideline |

## Repository Layout

```text
Zalo_AI/
├── data_mate/                  # docs for the DataMate track
│   ├── description.md          # task description
│   ├── data.md                 # data, splits, submission format
│   ├── evaluation.md           # scoring
│   ├── rule.md                 # rules and constraints
│   ├── submission_guidance.md  # timeline and submission process
│   └── notebooks/              # exploration / audit notebooks
└── data/                       # datasets — git-ignored, not committed
    └── data_mate/
        ├── train/train/        # 30 businesses, 3,000 questions + answers.jsonl
        └── public_test/public_test/   # 14 businesses, 1,000 questions
```

Each split folder holds `data/<business_id>/`, `rulebook.md`, `calendar.csv`, `questions.jsonl` (and `answers.jsonl` for train). The whole `data/` tree, including the zipped downloads, is git-ignored — grab it from [Dataset Downloads](data.md#7-dataset-downloads).

## The Constraints That Shape the Solution

- **Penalized Accuracy**: correct `+1`, abstain `0`, wrong `−1`. A wrong answer costs as much as two abstains — be conservative. See [evaluation.md](evaluation.md).
- **Fully offline**: no network, no API calls; ≤ **15B total parameters** across every model used at inference. See [rule.md](rule.md#2-models).
- **6 hours** wall-clock for the whole private test set on 2 × RTX 5090, model loading included.
- **Contract**: read `/data` (read-only), write `/result/submission.jsonl` and per-question logs to `/result/logs/`.
- **Rulebook per split**: read the split's own `rulebook.md` at run time — defaults for revenue, "bán chạy", week boundaries, "tháng này", cost, and the Tết window may differ between splits.
