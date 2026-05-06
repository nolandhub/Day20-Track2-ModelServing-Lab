# Reflection — Lab 20 (Personal Report)

> **Đây là báo cáo cá nhân.** Mỗi học viên chạy lab trên laptop của mình, với spec của mình. Số liệu của bạn không so sánh được với bạn cùng lớp — chỉ so sánh **before vs after trên chính máy bạn**. Grade rubric tính theo độ rõ ràng của setup + tuning của bạn, không phải tốc độ tuyệt đối.

---

**Họ Tên:** Đỗ Lê Thành Nhân
**Cohort:** A20-K1
**Ngày submit:** 2026-05-06

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

- **OS:** Linux 6.8.0-1044-azure (x86_64) - GitHub Codespaces
- **CPU:** AMD EPYC 7763 64-Core Processor
- **Cores:** 2 physical / 2 logical cores
- **CPU extensions:** AVX2 available
- **RAM:** 7.8 GB
- **Accelerator:** CPU only (no discrete accelerator)
- **llama.cpp backend đã chọn:** CPU (AVX/NEON tuning)
- **Recommended model tier:** TinyLlama-1.1B

**Setup story** (≤ 80 chữ): Bài lab chạy trên môi trường GitHub Codespaces (2-core). Tôi đã phải giới hạn số luồng biên dịch (`-j 1`) khi build llama.cpp từ nguồn để tránh lỗi tràn RAM (OOM) khiến trình duyệt bị treo. Model TinyLlama được chọn vì phù hợp với giới hạn 8GB RAM của Codespaces.

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

| Model                                | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
| ------------------------------------ | --------: | ----------------: | ----------------: | -------------------: | ------------------: |
| tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf |      2326 |         439 / 503 |       51.8 / 53.1 |   3468 / 3804 / 3834 |                19.3 |
| tinyllama-1.1b-chat-v1.0.Q2_K.gguf   |      1062 |         695 / 865 |       58.2 / 61.0 |   4250 / 4487 / 4497 |                17.2 |

**Một quan sát** (≤ 50 chữ): Kết quả khá bất ngờ khi bản Q2_K lại chậm hơn Q4_K_M về cả TTFT và TPOT. Có thể do trên model siêu nhỏ (1.1B) và CPU 2 nhân, chi phí giải nén của Q2 cao hơn hoặc bị nhiễu do môi trường cloud dùng chung.

---

## 3. Track 02 — llama-server load test

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
| ----------: | --------: | ------------: | -----------: | -----------: | -------: |
|          10 |      0.11 |         29000 |        52000 |        52000 |        0 |
|          50 |      0.20 |         21000 |        35000 |        35000 |        0 |

**KV-cache observation** (từ `record-metrics.py`): peak `llamacpp:kv_cache_usage_ratio` ở concurrency 50 ≈ 0.15, do TinyLlama dùng rất ít context và số lượng request thực tế thành công còn thấp nên chưa chiếm dụng nhiều cache.

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** stub: localhost only (Codespaces)
- **N17 (Data pipeline):** stub: in-memory list
- **N18 (Lakehouse):** stub: SQLite
- **N19 (Vector + Feature Store):** stub: TOY_DOCS (local list)

**Nơi tốn nhiều ms nhất** trong pipeline (đo bằng `time.perf_counter` trong `pipeline.py`):

- embed: 0.0 ms (stub)
- retrieve: 0.0 ms (stub)
- llama-server: ~13325.7 ms

**Reflection** (≤ 60 chữ): Bottleneck rõ ràng nằm ở khâu inference của llama-server (chiếm 99.9% thời gian). Việc chạy trên CPU 2 nhân khiến tốc độ sinh từ bị giới hạn, trong khi các bước còn lại dùng dữ liệu giả lập nên gần như không tốn thời gian.

---

## 5. Bonus — The single change that mattered most

**Change:** Giới hạn tiến trình biên dịch `-j 1` khi build llama.cpp.

**Before vs after** (paste 2-3 dòng từ sweep output):

```
before: make build-llama (crashes at 39% due to OOM)
after:  cmake --build build -j 1 (Success)
speedup: ~N/A (từ không chạy được thành chạy được)
```

**Tại sao nó work** (1–2 đoạn ngắn — đây là phần grader đọc kỹ nhất):
Trên môi trường Codespaces với RAM hạn chế (8GB), việc sử dụng `-j` mặc định sẽ kích hoạt nhiều luồng biên dịch song song, mỗi luồng ngốn hàng GB RAM dẫn đến hệ thống kích hoạt OOM Killer để bảo vệ OS, gây sập kết nối. Việc ép về 1 luồng giúp kiểm soát bộ nhớ ổn định hơn.

---

## 6. (Optional) Điều ngạc nhiên nhất

Tôi ngạc nhiên khi thấy bản Q2 lại chậm hơn Q4 trên CPU máy ảo này, chứng tỏ không phải lúc nào nén sâu cũng mang lại tốc độ nếu năng lực xử lý (compute) của CPU quá yếu so với băng thông bộ nhớ.

---

## 7. Self-graded checklist

- [x] `hardware.json` đã commit
- [x] `models/active.json` đã commit
- [x] `benchmarks/01-quickstart-results.md` đã commit
- [x] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit
- [x] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep)
- [x] Ít nhất 6 screenshots trong `submission/screenshots/` (xem `submission/screenshots/README.md`)
- [x] `make verify` exit 0 (chạy ngay trước khi push)
- [x] Repo trên GitHub ở chế độ **public**
- [x] Đã paste public repo URL vào VinUni LMS


---

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Nếu private, grader không xem được → 0 điểm.
