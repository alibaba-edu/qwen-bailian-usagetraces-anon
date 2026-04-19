# Qwen-Bailian Anonymous Dataset

This repository provides **anonymized, production-derived LLM usage traces**
collected from a **Qwen model serving cluster on Aliyun Bailian**.
The dataset is designed for **trace-driven evaluation of LLM serving systems**,
including caching, batching, scheduling, and end-to-end inference optimization.

The traces in this repository represent different usage scenarios:

| **Scenario**       | **Description**                 | **Trace File**                                                   |
| ------------------ | ------------------------------- | ---------------------------------------------------------------- |
| **To-C Trace**     | Chat-style interactive services | [`qwen_traceA_blksz_16.jsonl`](./qwen_traceA_blksz_16.jsonl)     |
| **To-B Trace**     | API-driven task automation      | [`qwen_traceB_blksz_16.jsonl`](./qwen_traceB_blksz_16.jsonl)     |
| **Thinking Trace** | Reasoning-intensive chat        | [`qwen_thinking_blksz_16.jsonl`](./qwen_thinking_blksz_16.jsonl) |
| **Coder Trace**    | Code generation                 | [`qwen_coder_blksz_16.jsonl`](./qwen_coder_blksz_16.jsonl)       |

---

## 🔥 What’s New

- **New Thinking Trace**
  Captures long-form reasoning workloads with **long output lengths**.
- **New Coder Trace**
  Represents code-generation and interactive programming workloads.
- **Official Trace Replayer**
  We open-sourced a **high-fidelity, timestamp-faithful trace replayer** for end-to-end benchmarking:  
  👉 [https://github.com/blitz-serving/trace-replayer](https://github.com/blitz-serving/trace-replayer)

## Overview

This dataset contains a two-hour sampled anonymized KVCache trace of requests sent to a single
Qwen model serving cluster on **Aliyun Bailian**.
It is used for validating design techniques for LLM serving systems
as well as inspiring future usage with the following
key workload characteristics collected:

- Temporal distribution of requests;
- Input/output token length;
- Session structure and chat turn patterns;
- Request type composition (text, search, image, file)

## Key Features

- **Production-Representative**: Subset retains real-world traffic patterns
- **Privacy-Compliant**: Salted hashing + domain remapping anonymization
- **Structured Format**: JSON Lines with schema documentation
- **Apache 2.0 Licensed**: Permissive open-source license for commercial use

For insights drawn from this dataset and techniques validated with it,
please refer to our works:
- [Optimizing KVCache cache design. (KVCache@ATC'25)](https://arxiv.org/abs/2506.02634)
- [Simple yet effective LLM scheduling. (LMetric@OSDI'26)](https://arxiv.org/abs/2603.15202)

## Data Specification of the Traces 

Each file contains a representative workload, 
e.g., `qwen_traceB_blksz_16.jsonl` refer to a to-B trace collected at 2024.12.
Each record contains the following information: 

``` jsonc
{
  "chat_id": 159,                                   // Randomized chat identifier
  "parent_chat_id": 55,                             // -1 for root requests
  "timestamp": 61.114,                              // Seconds since request arrive
  "input_length": 521,                              // Input token count
  "output_length": 132,                             // Output token count
  "type": "text",                                   // Request type: text/search/image/file
  "turn": 2,                                        // Conversation turn number
  "hash_ids": [1089, 1090, 1091, 6326, ..., 13148]  // Salted SipHash blocks (16 tokens per block)
}
```

## Anonymization Process

1. **Token Block Hashing**:
    - Group tokens into 16-token blocks
    - Apply salted SipHash-2-4 to each block

2. **Domain Remapping**:
    - Map hash values to sequential integers
    - Breaks correlation between hash IDs and original content

3. **ID Randomization**:
    - Replace chat IDs with sequential integers
    - No linkage to user accounts or device identifiers

4. **Time-based Anonymization**:
    - All timestamps are normalized to trace-relative values, starting from 0 at the beginning of each trace file. Original absolute timestamps (e.g., Unix time) are removed to prevent temporal correlation with external events or user behavior patterns.

## Trace Replayer (Recommended)

To enable **end-to-end, trace-driven benchmarking**, we provide an official
open-source **Trace Replayer**:

👉 [https://github.com/blitz-serving/trace-replayer](https://github.com/blitz-serving/trace-replayer)

**Trace Replayer** is a **Rust-based, high-throughput replay engine** that:

* Reconstructs **synthetic prompts** from input length + block hashes
* Preserves **KVCache hit/miss patterns**
* Replays requests against real backends (e.g., vLLM) via standard APIs
* Records per-request latency, TTFT/TPOT (backend-dependent), and timing drift

It can achieve **100+ QPS and 500K+ tokens/s** using **~30 CPU threads**,
sufficient to stress-test **16–32 instance Qwen3-30B-A3B deployments**.

Supported backends include **OpenAI-compatible APIs**, **TGI**, and **AIBrix**.

## FAQ

For common questions about trace patterns (e.g., missing tokens across turns, block hash mismatches), see [docs/qa-context-growth-pattern.md](./docs/qa-context-growth-pattern.md).

## Privacy & Compliance

- **No PII**: All content hashed with irreversible cryptographic functions
- **Unlinkable**: No cross-session or user-device associations preserved
- **GDPR/CCPA Compliant**: Meets anonymous data standards under major regulations

## License

[Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)
> A permissive license allowing commercial use and modifications, requiring only preservation of the license notice in derivative works

## Citation

If you find this dataset useful or use it in your research, 
please kindly cite our paper using the following bib, thanks! 

```
@inproceedings {kvcache,
  title={KVCache Cache in the Wild: Characterizing and Optimizing KVCache Cache at a Large Cloud Provider},
  author={Wang, Jiahao and Han, Jinbo and Wei, Xingda and Shen, Sijie and Zhang, Dingyan and Fang, Chenguang and Chen, Rong and Yu, Wenyuan and Chen, Haibo}, 
  booktitle = {2025 USENIX Annual Technical Conference (USENIX ATC 25)},
  year = {2025},
  url = {https://www.usenix.org/conference/atc25/presentation/wang-jiahao},
  publisher = {USENIX Association},
  month = jul,
}
```
