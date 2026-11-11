# Data

## 1. Business Types

The data covers a wide range of Vietnamese small businesses:

- Grocery stores
- Coffee and bubble-tea shops
- Restaurants
- Pharmacies
- Building-material stores
- Motorbike repair shops
- Mother-and-baby stores
- And more.

**The private test includes business types that do not appear in the training or public test data.**

Each business covers **12–30 months** of activity with seasonality (Tết, Mid-Autumn, public holidays, paydays, weekends) and ranges from a few hundred to about **300,000 rows**.

## 2. Package Structure

```text
<split>/
├── data/<business_id>/
│   ├── *.csv | *.xlsx | *.sqlite   # one or more data files
│   └── README.md                   # business description and notes
├── rulebook.md                     # default rules and vocabulary
├── calendar.csv                    # solar–lunar calendar (Tết)
├── questions.jsonl                 # one question per line
└── answers.jsonl                   # training only
```

### Sample Record in `questions.jsonl`

```json
{
  "id": "q_01234",
  "business_id": "biz_017",
  "question": "thang nay hang nao ban cham nhat, ton nhieu ma ban ko dc",
  "as_of_date": "2026-08-27"
}
```

### Sample Record in `answers.jsonl`

Training businesses also come with `answers.jsonl`, which contains the reference answer for every training question, linked by `id`.

Each line follows the [submission format](#5-submission-format), plus an `alias` field listing other accepted forms of a name answer (alternative spellings found in the data, and the item code). Unanswerable questions have `"answer": null`. **Submissions do not need to include the `alias` field.**

The reference answer of the example question in [description.md](description.md):

```json
{
  "id": "q_06de33fa21cc",
  "type": "string",
  "answer": "Xi măng Hà Tiên PCB40 bao 50kg",
  "alias": ["SP0001"]
}
```

The quality of each business `README.md` varies: some come with a full data dictionary, others with a few informal notes, some with almost nothing.

## 3. Splits

| Split | Businesses | Questions | Reference answers |
|---|---|---|---|
| **Training** | 80 | ~3,000 | Yes (`answers.jsonl`) |
| **Public test** | 10–20 | ~1,000 | Hidden; scored on the public leaderboard |
| **Private test** | 10–20 | ~500 | Hidden; released at the private test phase; **includes new business types** |

As released locally under `data/data_mate/` (git-ignored): `train/train/` has 30 businesses (`biz_101` …) with 3,000 questions and 3,000 answers; `public_test/public_test/` has 14 businesses (`dev_001` …) with 1,000 questions and no answers.

## 4. Business Rules (Default Interpretations)

Each split ships its own `rulebook.md`, which fixes the default meaning of common terms whenever the question does not say otherwise.

> The rulebook **may differ between splits**, so the agent must read and apply the rulebook of the split it is answering, not a copy of the training rulebook.

Typical rules:

| Term / Concept | Default Interpretation |
|---|---|
| Revenue | Sum of line amounts after line-level discounts (order-level vouchers are **not** deducted), excluding VAT and delivery fees, excluding cancelled orders and orders with any returned item. |
| "Bán chạy" (best-selling) | By quantity. |
| A week | Monday–Sunday. |
| "Tháng này" (this month) | The month containing the as-of date, up to and including that date. |
| Default cost | Weighted-average purchase cost by lot. |
| Tết window | 23 tháng Chạp to mùng 10 (lunar calendar — see `calendar.csv`). |

`calendar.csv` columns: `date, weekday, lunar_day, lunar_month, lunar_year, lunar_leap, holiday`.

## 5. Submission Format

In both rounds you upload a `submission.jsonl` file, one JSON object per line:

```jsonl
{"id": "q_01234", "type": "string", "answer": "Sting dâu 330ml"}
{"id": "q_01235", "type": "number", "answer": 48250000}
{"id": "q_01236", "type": "percent", "answer": -12.45}
{"id": "q_01237", "type": "date", "answer": "2026-08-15"}
{"id": "q_01238", "type": "boolean", "answer": true}
{"id": "q_01239", "type": "list", "answer": ["Cà phê sữa", "Bạc xỉu", "Trà đào"]}
{"id": "q_01240", "type": "table", "answer": {"columns": ["khach_hang", "so_tien_no"], "rows": [["Chị Lan", 3200000], ["Anh Tú", 1500000]]}}
{"id": "q_01241", "type": "unanswerable", "reason": "Dữ liệu không có giá vốn nên không tính được lợi nhuận"}
{"id": "q_01242", "type": "abstain"}
```

- Questions missing from the file are scored as `abstain`.
- The `reason` field is optional.
- The number of submissions per day follows the general regulations (see [submission_guidance.md](submission_guidance.md)).

## 6. Docker Image

The image (see [Competition Rounds](rule.md#competition-rounds)) must contain:

- The source code (training and inference).
- The model weights.
- A `predict.sh` that reads the test set from `/data` and writes `/result/submission.jsonl`.
- A `README`.

It should also write a per-question log to `/result/logs/`, in any format, containing at least the **question ID**, the **final answer** and the **elapsed time**, plus anything else that shows how the answer was reached (e.g. generated code, execution output).

The organizers use these logs to resolve differences between your uploaded result and their re-run ([Rule 8](rule.md#8-final-score-and-reproducibility)).

## 7. Dataset Downloads

| Dataset | Download |
|---|---|
| **Data Train** | [train.zip](https://dl-challenge.zalo.ai/2026/datamate_dkewxftdzo/train.zip) |
| **Public Test** | [public_test.zip](https://dl-challenge.zalo.ai/2026/datamate_dkewxftdzo/public_test.zip) |
