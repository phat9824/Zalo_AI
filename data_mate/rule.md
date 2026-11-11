# Rules

## Competition Rounds

The competition runs on a leaderboard with two rounds.

### Round 1 — Public Test

Teams develop their solutions, run them on the public test set and upload `submission.jsonl`.

Scores appear on the public leaderboard and are **for reference only** during development.

### Final Solution Submission

Before the solution deadline, the final solution is submitted as a **Docker image**, following the organizers' separate [Docker guideline](zac2026_DataMate_docker_guideline_final.pdf).

For scoring, the organizers mount the test set into the container at `/data` and collect `/result/submission.jsonl`.

### Round 2 — Private Test

After teams submit and freeze their solutions, the organizers release the private test set.

Teams run their frozen solution on it and upload `submission.jsonl`.

**The final ranking is based on the organizers' re-run of the submitted Docker images** ([Rule 8](#8-final-score-and-reproducibility)).

Dates for each round are in [submission_guidance.md](submission_guidance.md).

---

## Technical Requirements

Every solution must meet **all** of the following requirements.

### 1. Teams and Eligibility

Teams and eligibility follow the general regulations of Zalo AI Challenge 2026.

### 2. Models

You may use any model whose weights were publicly released **before 26 October 2026** under a licence that permits such use, and you may fine-tune it freely.

Across all models used at inference by your solution (language models, embedding models, rerankers, classifiers), whether loaded at the same time or not, the **total number of parameters must not exceed 15 billion**, counting all parameters of mixture-of-experts models.

- Any quantization is allowed.
- The models used must be **declared in the solution README**.

### 3. No External Services

Inference must run **fully offline** inside your Docker image:

- No calls to commercial LLM APIs.
- No downloads.
- No network access of any kind.

The image is run with networking disabled.

### 4. No Human in the Loop

Answers must be produced by your system **without manual intervention**.

Hard-coding answers, or logic keyed on question or business identifiers, is **prohibited**.

### 5. Code Execution

Your agent **may generate and execute code** inside the container; this is expected but not required.

The data directory is mounted **read-only**; write temporary files elsewhere.

### 6. Data

You **may** use:

- The provided training data.
- Data you synthesize yourself.
- Publicly available datasets and models.

You may **not**:

- Use the public or private test data for training.
- Collect answers to test questions by any means.

### 7. Compute Reference

Solutions are verified on a machine with the following specifications:

| Component | Specification |
|---|---|
| GPU | 2 × NVIDIA RTX 5090 |
| VRAM | 32 GB per GPU (64 GB total) |
| RAM | 512 GB |
| Time limit | 6 hours wall-clock, **including model loading** |

Your solution must process the **entire private test set** within 6 hours of wall-clock time on this machine.

Blackwell GPUs require **CUDA ≥ 12.8** and a matching PyTorch/vLLM build. The Docker guideline specifies the base image to use.

### 8. Final Score and Reproducibility

The score of the organizers' re-run of your submitted Docker image on the private test set is your **final score**.

The `submission.jsonl` you upload during the private test phase determines only the provisional leaderboard and which teams are re-run.

Your solution must be deterministic enough that the two runs agree. A solution is considered reproducible if:

$$\left| PA_{\text{uploaded}} - PA_{\text{re-run}} \right| \le 0.03$$

If the difference exceeds `0.03`, the organizers will:

1. Examine the logs written by your image (see [Docker Image](data.md#6-docker-image)).
2. Possibly run the image once more.
3. Possibly request additional information from the team.
4. **Disqualify the entry** if the difference cannot be explained.

If the re-run exceeds the time limit of [Rule 7](#7-compute-reference) or terminates early:

- Only the answers written to `/result/submission.jsonl` before termination are scored.
- The remaining questions count as `abstain`.
