# Q&A: Context Growth Pattern in Traces

## Q1: Incomplete context carry-over across turns

**Question:** In multi-turn sessions, the next turn (e.g., 2) input does not include all tokens from the previous turn (e.g., 1) output. For example, if turn 1 output is tokens `3 4 5 6 7`, turn 2 input may only show `4 5 6`, with tokens `3` and `7` missing.

**Answer:** The missing tokens are special tokens (`<think>` and `</think>`) injected by the chat template. The application strips these tokens when assembling the context for the next turn, so they do not appear in subsequent requests sent to the cluster.

---

## Q2: Last block hash mismatch

**Question:** The hash of the last block in a trace may differ from what is expected. For example, given input `1 2 3` and output `4`, the trace may show block hash for `1 2 3 5` instead of `1 2 3 4`.

**Answer:** The last input block may contain padding tokens. When the first output token is generated, it replaces part of the padding, changing the block's content and therefore its hash. The discrepancy reflects this block hash update, not a missing or corrupted token.
