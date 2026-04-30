## Assumptions
- All of the assumptions above are equal to the assumptions of the [PII Redaction server](https://docs.google.com/document/d/1iS9IoJvbHzywiIZalkvSsEyRBcB4cDyc1iJQ8zHxoSk/edit?tab=t.271u61su25kw).



## Cost Projection by Scenario

This service integrates ML inference with LLM-based category generation. Since LLM latency depends on request payload and GCP Vertex AI conditions, exact P99 cannot be guaranteed. The estimates below are based on load tests with a simulated 2.5-second LLM response, yielding a measured P99 of <4 seconds. The load test assumes all users are on iOS devices. For Android devices, the model is lighter, which may result in increased throughput.

Based on the steady-state load test results, a single GPU pod for **camerarollphotosuggestion** handles **521.28 TPM**. The minimum cluster (4 GPU pods) provides a total baseline capacity of **~2,085 TPM**.

| Target Region | Exp. User Base (daily) | Exp. Throughput (TPM) | Est. Daily Cost (before discount) | Est. Daily Cost (after 21% discount) | Configuration Detail |
| --- | --- | --- | --- | --- | --- |
| **Baseline (Min)** | 0 - 2M | **2,085 TPM** | **$116** | **$92** | 4 GPU(g5.2xlarge) Pods |
| **AUS 100%**, adoption 50% | 380K | 264 TPM | $116 | $92 | Same as baseline. |
| **USA 100%**, adoption 50% | 1.3M | 900 TPM | $116 | $92 | Same as baseline. |
| **2026 Expansion Plan** (AUS+CAN+USA 30%) | 795K | 550 TPM | $116 | $92 | Same as baseline. |
| **Full North America** (AUS+CAN+USA 100%) | 1.7M | 1,200 TPM | $116 | $92 | Same as baseline. |
| **Global 100%**, adoption 50% | 20M | 14,000 TPM | **$781** | **$617** | Scale out Baseline x ~6.7 |

> TPM (Transactions Per Minute) requirements are derived to process total daily traffic within 24 hours.

---

## Details: Unit Metrics & Baseline Capacity (Resource Basis)

The system is consolidated into a single service responsible for both inference and LLM handling. To ensure High Availability (HA) across 4 Availability Zones (AZs), the baseline is set to 4 Pods.

| Service Name | Role | Resource Type | Min. Configuration | Unit Cost per Pod |
| --- | --- | --- | --- | --- |
| **camerarollphotosuggestion** | **ML Model and LLM Handling** | **GPU (g5.2xlarge)** | **4 Pods (1 Pod/Node)** | **$1.212/hr** |

### Load Test Benchmark Summary (Per GPU Pod)

The following data represents the performance of a single **g5.2xlarge** pod under a concurrency of 30, as captured in the steady-state results.

* **Throughput:** 521.28 TPM (8.69 RPS)
* **Latency (Successful Requests):**
* **Average:** 3482.20 ms
* **P50 (Median):** 3507.22 ms
* **P99:** **3827.55 ms**


* **Resource Usage:**
* **CPU Utilization:** 40.9% (avg), 76.0% (max)
* **GPU Utilization:** 66.0% (avg), **100.0% (max)**


* **Success Rate:** 100.0% (0.00% Error Rate)
