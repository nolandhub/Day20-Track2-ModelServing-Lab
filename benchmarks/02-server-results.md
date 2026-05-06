# Track 02 — Server Results

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.11 | 29000 | 52000 | 52000 | 0 |
| 50 | 0.20 | 21000 | 35000 | 35000 | 0 |

Observation: Performance is limited by the 2-core CPU environment in GitHub Codespaces.
