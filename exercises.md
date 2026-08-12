# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Một câu trả lời từ chối đúng cho yêu cầu ngoài phạm vi có rất ít từ trùng evidence, hoặc câu trả lời rất ngắn chỉ hướng dẫn liên hệ kênh hỗ trợ. | Trả lời chính sách đơn hàng, hoàn tiền, bảo hành hoặc an toàn nhưng thêm điều kiện, giá, thời hạn hay quyền lợi không có trong context. | Lấy trace để xác minh claim; block nếu có thể gây thiệt hại/vi phạm privacy. Củng cố grounding prompt và thêm kiểm tra claim-evidence. |
| Answer Relevance | Câu hỏi mơ hồ hoặc gồm nhiều ý; câu trả lời trước hết hỏi lại một thông tin quyết định (ví dụ ngày đặt hàng) thay vì đoán. | Context đúng nhưng câu trả lời sang chủ đề khác, không trả lời intent chính, hoặc từ chối một yêu cầu OrbitTech hợp lệ. | Kiểm tra intent/routing và prompt; bổ sung test cho intent đó. |
| Context Recall | Với câu hỏi đơn giản, chunk top-k bỏ một chi tiết phụ nhưng answer vẫn nêu đúng toàn bộ điều kiện thiết yếu từ context khác. | Retriever bỏ date, exception, điều kiện eligibility hoặc evidence chính nên generator không thể trả lời đủ/chính xác. | Điều tra query, chunking và top-k; chỉnh retriever hoặc thêm reranking rồi đo lại. |
| Context Precision | Retriever có một vài chunk nhiễu ở cuối nhưng evidence liên quan đã đứng đầu và câu trả lời vẫn chính xác. | Nhiễu đứng trước evidence, làm mất context window hoặc khiến answer bám vào chính sách/tình huống sai. | Rerank, cải thiện query expansion/chunk metadata và kiểm tra ranking traces. |
| Completeness | Câu trả lời cố ý ngắn cho factual lookup và chỉ thiếu chi tiết không ảnh hưởng quyết định của khách hàng. | Bỏ sót điều kiện, ngoại lệ, thời hạn, phí hoặc bước hành động cần thiết; khách hàng có thể làm sai quy trình. | So sánh với expected answer; cải thiện context coverage, prompt yêu cầu trả lời đủ phần và thêm regression case. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:* Tạo một tập 30 cặp đáp án cho cùng câu hỏi OrbitTech, trong đó đáp án A và B có chất lượng đã được human label hoặc được kiểm soát. Condition 1: judge nhận `Answer A` trước `Answer B`; Condition 2: đảo thứ tự để `Answer B` xuất hiện trước `Answer A`. Giữ nguyên prompt, rubric, model và nhiệt độ; randomize thứ tự các cặp và chạy nhiều lần. So sánh tỷ lệ A thắng và chênh lệch điểm giữa hai condition. Nếu đáp án đứng trước thắng hoặc được điểm cao hơn một cách có hệ thống dù nội dung không đổi, judge có position bias. Có thể thêm condition 3: hoán đổi nhãn A/B nhưng giữ vị trí để tách ảnh hưởng của nhãn khỏi vị trí.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:* Chấm correctness, completeness, relevance, evidence và safety/actionability riêng, không có tiêu chí “dài” hoặc “nhiều chi tiết”. Rubric phải nêu rõ câu trả lời ngắn vẫn đạt 5 nếu đúng, đủ điều kiện thiết yếu và có thể hành động; thông tin lặp, lan man hoặc claim không có evidence không được cộng điểm và có thể bị trừ. Đặt giới hạn độ dài hợp lý, yêu cầu judge nêu các claim/conditions bị thiếu thay vì đánh giá theo cảm giác, và dùng ví dụ một đáp án ngắn tốt so với một đáp án dài nhưng chứa nhiễu.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:* LLM judge có thể hiểu rubric khác người chấm, ưu ái văn phong của chính nó, hoặc bỏ qua rủi ro domain như lộ privacy, bịa điều kiện hoàn tiền và hướng dẫn an toàn sai. So sánh điểm judge với một tập human labels giúp đo agreement, phát hiện systematic bias và điều chỉnh rubric/prompt/threshold. Human labels cũng là chuẩn tham chiếu để biết một score cao của judge thực sự có nghĩa là trải nghiệm khách hàng chấp nhận được.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.80 | Đây là hàng rào an toàn chính: claim không được evidence hỗ trợ có thể dẫn đến quyết định sai về tiền, bảo hành, account hoặc safety. |
| Answer Relevance | 0.70 | Cần trả lời đúng intent phần lớn cases; một số câu mơ hồ hợp lệ cần hỏi làm rõ nên không nên đặt ngưỡng cứng như faithfulness. |
| Completeness | 0.70 | Giữ các điều kiện và bước hành động quan trọng; cho phép khác biệt nhỏ của cách diễn đạt nhưng block khi câu trả lời thường xuyên thiếu thông tin thiết yếu. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:* Dùng **offline evaluation** trên golden dataset trước mọi thay đổi về code, prompt, model, retriever, chunking hoặc trước release; đây là quality gate có thể lặp lại và so sánh với baseline. Dùng **online evaluation** sau deploy để theo dõi traffic thật: pass/failure proxy, retrieval traces, latency, cost, feedback và các drift/failure pattern mới. Dùng **human review** cho các case high-stakes hoặc không chắc chắn, như privacy/security incident, fraud, warranty/return dispute, safety issue, mẫu có score sát ngưỡng, và để định kỳ calibration LLM judge. Online signal đáng ngại phải tạo case đã ẩn danh để bổ sung vào golden dataset rồi đánh giá offline lại.

---

## Part 2 — Core Coding (14:45–15:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +10)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
