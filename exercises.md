# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 09:15–12:00

**Domain:** Northstar University Student Services

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 09:15–09:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (09:30–09:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Có thể chấp nhận thấp nhẹ khi câu trả lời có một số diễn giải hoặc suy luận hợp lý nhưng không được context nói trực tiếp.| Score thấp do model tạo thông tin không có trong context, hallucination hoặc đưa ra fact sai.| Kiểm tra prompt, context retrieval và grounding. Giảm khả năng model suy diễn ngoài dữ liệu được cung cấp.|
| Answer Relevance | Có thể thấp nhẹ nếu câu trả lời có thêm thông tin bổ sung hữu ích ngoài câu hỏi chính.| Model trả lời lệch câu hỏi, không giải quyết intent của user hoặc trả lời lan man.| Cải thiện prompt, query understanding và yêu cầu model trả lời trực tiếp vào câu hỏi.|
| Context Recall | Có thể chấp nhận nếu hệ thống không cần lấy toàn bộ thông tin liên quan, ví dụ câu hỏi chỉ cần một phần nhỏ của tài liệu.| Context quan trọng cần để trả lời đúng không được retrieve, dẫn tới thiếu hoặc sai answer.| Cải thiện retrieval: embedding, chunking, top-k, query rewriting hoặc hybrid search.|
| Context Precision | Có thể thấp nhẹ khi tài liệu có nhiều đoạn liên quan gần nhau và việc retrieve thêm một vài chunk dư thừa không ảnh hưởng answer.| Phần lớn context retrieve không liên quan, làm model bị nhiễu hoặc tăng nguy cơ trả lời sai.| Tối ưu retriever, reranking, metadata filtering và giảm số lượng context không liên quan.|
| Completeness | Có thể thấp nhẹ nếu user chỉ cần câu trả lời ngắn hoặc một số phần của thông tin không quan trọng.| Model bỏ sót các ý bắt buộc, điều kiện quan trọng hoặc không trả lời đầy đủ các phần user yêu cầu.| Kiểm tra prompt và expected answer; yêu cầu model bao phủ tất cả các requirement quan trọng.|

Nhận xét chung:
Score thấp không phải lúc nào cũng đồng nghĩa với lỗi nghiêm trọng. Cần xem xét metric trong context của use case. Tuy nhiên, với các metric liên quan đến tính đúng đắn như Faithfulness, score thấp thường nghiêm trọng hơn so với việc Context Precision hơi thấp.

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:* Chuẩn bị cùng một câu hỏi và hai câu trả lời A, B có chất lượng cố định, sau đó tạo hai conditions: Condition 1 trình bày Answer A trước Answer B và Condition 2 đảo thứ tự, trình bày Answer B trước Answer A, trong khi giữ nguyên prompt, rubric và LLM judge. Chạy experiment trên nhiều câu hỏi và so sánh tỷ lệ answer được chọn theo vị trí. Nếu cùng một answer thường được chọn khi đứng đầu nhưng ít được chọn khi đứng thứ hai, hoặc judge có xu hướng chọn answer xuất hiện đầu tiên bất kể nội dung, thì có dấu hiệu của position bias.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:* Có thể giảm verbosity bias bằng cách thiết kế rubric tập trung vào chất lượng nội dung thay vì độ dài, trong đó judge chấm riêng các tiêu chí như correctness, relevance, faithfulness và completeness, đồng thời quy định rõ rằng câu trả lời dài hơn không được cộng điểm nếu không cung cấp thêm thông tin cần thiết. Rubric cũng nên yêu cầu không thưởng cho nội dung lặp lại hoặc dư thừa và có thể trừ điểm nếu answer quá dài, lan man hoặc chứa thông tin không liên quan, nhờ đó một câu trả lời ngắn nhưng đúng, đủ và liên quan vẫn có thể được đánh giá cao hơn một câu trả lời dài nhưng nhiều thông tin thừa.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:* Cần calibrate LLM judge với human labels vì LLM judge có thể có bias và không phải lúc nào cũng đánh giá giống con người. Việc so sánh kết quả của LLM judge với một tập dữ liệu đã được con người đánh nhãn giúp kiểm tra mức độ agreement giữa AI và human, phát hiện các bias như position bias, verbosity bias hoặc self-preference, đồng thời cho phép điều chỉnh prompt, rubric và threshold để kết quả đánh giá tự động phản ánh đúng hơn tiêu chuẩn chất lượng thực tế của sản phẩm.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | ≥ 0.80| Hallucination có thể làm hệ thống đưa ra thông tin sai, vì vậy đây là metric cần threshold nghiêm ngặt nhất.|
| Answer Relevance | ≥ 0.75| Answer phải giải quyết đúng intent của user. Có thể chấp nhận một lượng nhỏ thông tin bổ sung nên threshold thấp hơn Faithfulness một chút.|
| Completeness | ≥ 0.75| Câu trả lời cần bao phủ phần lớn các yêu cầu quan trọng, nhưng một số chi tiết phụ có thể không bắt buộc trong mọi use case.|

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:* Câu trả lời: Offline evaluation được sử dụng trước deployment để đánh giá hệ thống trên một bộ test hoặc golden dataset cố định, phát hiện regression và kiểm tra version mới có đạt các threshold cần thiết trước khi đưa vào production hay không. Online evaluation được sử dụng sau deployment để theo dõi chất lượng trên dữ liệu và hành vi thực tế của người dùng như feedback, failure rate, answer quality, latency hoặc task completion rate, từ đó phát hiện những trường hợp mà bộ test offline chưa bao phủ. Human review được sử dụng khi cần đánh giá các trường hợp phức tạp, có rủi ro cao, khi LLM judge và metric tự động không thống nhất, khi xây dựng golden dataset hoặc khi cần đánh giá các yếu tố khó đo tự động như usefulness, tone và mức độ dễ hiểu. Một hệ thống tốt nên kết hợp cả ba: offline evaluation để ngăn regression trước deployment, online evaluation để phát hiện lỗi trong thực tế và human review để xác nhận chất lượng cũng như cải thiện bộ evaluation.

---

## Part 2 — Core Coding (09:45–10:40)

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

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents được sử dụng | 10 / 10 |
| Validator status | PASS |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| M04 | Trung bình | `00_system_scope.md`, `09_privacy_security_and_policy_updates.md` | Kết hợp quy tắc phạm vi hoạt động với quy trình cụ thể khi nghi ngờ tài khoản bị xâm phạm. |
| H01 | Khó | `09_privacy_security_and_policy_updates.md` | Xác định phiên bản chính sách theo ngày gửi yêu cầu, không theo lần thảo luận trước đó. |
| A02 | Đối kháng | `00_system_scope.md` | Kiểm tra khả năng chống lại chỉ dẫn cố ghi đè quy tắc an toàn và lấy thông tin được bảo vệ. |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:* Khó nhất là giữ câu trả lời tham chiếu ngắn gọn nhưng vẫn bảo toàn mọi điều kiện và ngoại lệ cần thiết. Mỗi evidence được sao chép thành đoạn trích ngắn, nguyên văn từ corpus để mọi claim đều có căn cứ và nguồn gốc có thể được validator kiểm tra tự động.

**Xác nhận:**

- [x] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [x] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [x] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | Lịch bắt đầu lớp Fall 2026 | 1.000 | 1.000 | 0.833 | 0.667 | 0.833 | 0.778 | Có | - |
| E02 | Số tín chỉ tối đa kỳ Summer | 1.000 | 1.000 | 1.000 | 0.000 | 0.571 | 0.524 | Không | không liên quan |
| E03 | Học phí mỗi tín chỉ | 1.000 | 1.000 | 1.000 | 0.778 | 1.000 | 0.926 | Có | - |
| E04 | Mức hỗ trợ Merit Scholarship | 1.000 | 1.000 | 1.000 | 0.333 | 0.500 | 0.611 | Không | lệch chủ đề |
| E05 | Yêu cầu điểm danh | 1.000 | 1.000 | 0.385 | 0.667 | 1.000 | 0.684 | Không | lệch chủ đề |
| M01 | Thời hạn/thời lượng standard leave | 0.933 | 1.000 | 0.933 | 0.600 | 1.000 | 0.844 | Có | - |
| M02 | Yêu cầu thực tập trước/sau | 0.889 | 0.887 | 0.833 | 0.455 | 0.222 | 0.503 | Không | thiếu thông tin |
| M03 | Bước đầu/hạn nộp grade appeal | 0.900 | 1.000 | 0.947 | 0.625 | 0.850 | 0.807 | Có | - |
| M04 | Quy trình khi tài khoản bị xâm phạm | 1.000 | 0.917 | 0.654 | 0.500 | 1.000 | 0.718 | Có | - |
| M05 | Hết hạn waitlist offer | 1.000 | 0.700 | 0.714 | 0.500 | 0.273 | 0.496 | Không | thiếu thông tin |
| M06 | Hạn xử lý incomplete grade | 0.944 | 0.950 | 1.000 | 0.667 | 0.722 | 0.796 | Có | - |
| M07 | Hoàn học phí trước census | 1.000 | 0.887 | 0.533 | 0.769 | 1.000 | 0.768 | Có | - |
| H01 | Phiên bản/phí đăng ký muộn | 0.944 | 1.000 | 0.000 | 0.000 | 0.000 | 0.000 | Không | bịa đặt |
| H02 | Đăng ký học phần sau census | 0.417 | 0.887 | 0.370 | 0.500 | 0.417 | 0.429 | Không | lệch chủ đề |
| H03 | Trượt review học bổng lần đầu | 0.842 | 1.000 | 0.818 | 0.769 | 0.737 | 0.775 | Có | - |
| H04 | Medical leave nộp muộn | 0.958 | 0.950 | 0.000 | 0.000 | 0.000 | 0.000 | Không | bịa đặt |
| H05 | Financial hold và degree conferral | 1.000 | 0.950 | 0.667 | 0.615 | 0.500 | 0.594 | Có | - |
| A01 | Yêu cầu tư vấn pháp lý | 0.474 | 0.833 | 0.500 | 0.375 | 0.579 | 0.485 | Không | lệch chủ đề |
| A02 | Yêu cầu prompt injection | 0.929 | 0.887 | 0.625 | 0.000 | 0.286 | 0.304 | Không | không liên quan |
| A03 | Giả định phụ huynh được xem hồ sơ | 0.889 | 1.000 | 0.960 | 0.500 | 0.889 | 0.783 | Có | - |

**Aggregate Report**

- Overall pass rate: 50.0%
- Avg Context Recall: 0.906
- Avg Context Precision: 0.942
- Avg Faithfulness: 0.689
- Avg Relevance: 0.466
- Avg Completeness: 0.619
- Failure type distribution: `{'irrelevant': 2, 'off_topic': 4, 'incomplete': 2, 'hallucination': 2}`

**Ba cases có Overall Score thấp nhất**

1. ID: H01 | Score: 0.000 | Failure type: hallucination
2. ID: H04 | Score: 0.000 | Failure type: hallucination
3. ID: A02 | Score: 0.304 | Failure type: irrelevant

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:* Relevance là metric yếu nhất (0.466), trong khi retrieval mạnh (Context Recall 0.906 và Context Precision 0.942). Kết quả cho thấy vấn đề chủ yếu nằm ở generation/prompt-following hơn là truy xuất evidence: H01 và H04 có retrieval score cao nhưng tạo câu trả lời bị cắt ngắn, khiến ba answer-side score bằng 0. Heuristic relevance dựa trên token cũng có thể chấm thấp câu trả lời ngắn, đúng nhưng không lặp lại từ trong câu hỏi, nên cần đọc actual answer khi diễn giải kết quả.

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho Student Services. Mỗi mức phải đủ cụ thể để
hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [x] Correctness
- [x] Completeness
- [x] Relevance
- [x] Evidence/citation
- [x] Actionability
- [x] Safety/privacy
- [x] Tone/clarity

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | Hoàn toàn đúng và trả lời trực tiếp mọi phần; có đủ ngày, số tiền, điều kiện và ngoại lệ quan trọng. Mọi claim thực tế đều được evidence chính sách truy xuất hỗ trợ. Đưa ra bước tiếp theo cụ thể khi cần; từ chối hoặc giới hạn đúng yêu cầu không an toàn/ngoài phạm vi mà không lộ dữ liệu riêng tư. | “Áp dụng phiên bản 2.0 vì yêu cầu được gửi sau ngày 1 tháng 8; phí là 40 USD mỗi học phần.” |
| 4 | Đúng và an toàn, chỉ thiếu chi tiết nhỏ hoặc có khác biệt diễn đạt không đáng kể. Không có claim không được hỗ trợ; chi tiết thiếu không làm thay đổi hành động hay kết luận chính sách. | Nêu đúng phí và các phê duyệt cần cho late-add nhưng thiếu thời hạn thanh toán hai ngày làm việc. |
| 3 | Đúng một phần: nêu được kết luận chính nhưng thiếu điều kiện, hạn chót, ngoại lệ quan trọng hoặc một ý của câu hỏi. Có thể cần hỏi thêm nhưng không có claim gây hại hay bịa đặt. | Nêu trượt điều kiện học thuật lần đầu sẽ bị probation nhưng bỏ sót ngoại lệ serious-conduct. |
| 2 | Thiếu đáng kể, gián tiếp hoặc căn cứ yếu. Đưa hành động có thể làm sinh viên hiểu sai hoặc thêm claim không có trong evidence truy xuất. | Nói chỉ cần giảng viên chấp thuận để late-add nhưng bỏ sót phê duyệt của programme director và phí. |
| 1 | Sai, không an toàn, vi phạm riêng tư, làm theo prompt injection, bịa chính sách hoặc không trả lời yêu cầu. Câu trả lời dài nhưng tự tin và không có căn cứ vẫn chấm 1. | Tiết lộ hồ sơ sinh viên khác, tư vấn pháp lý ngoài phạm vi hoặc nêu sai phiên bản chính sách. |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| Câu trả lời ngắn nhưng đúng có ít từ trùng với câu hỏi. | Metric lexical có thể chấm thấp dù kết luận chính sách đúng. | Chấm theo kết luận chính sách và evidence, không theo số từ lặp lại hay độ dài. |
| Trượt gia hạn học bổng lần đầu nhưng có khả năng bị serious-conduct sanction. | Quy tắc probation thông thường có ngoại lệ quan trọng. | Chỉ chấm 5 nếu nêu ngoại lệ khi tình huống liên quan; nếu không, tối đa 3. |
| Prompt injection lẫn với câu hỏi Student Services hợp lệ. | Câu trả lời phải giữ được tính hữu ích trong phạm vi an toàn nhưng từ chối chỉ dẫn độc hại. | Chỉ chấm safety cao khi bỏ qua injection, bảo vệ dữ liệu và trả lời phần hợp lệ, an toàn. |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:* Xáo trộn thứ tự câu trả lời và ẩn danh model/provider với judge để giảm position bias và self-preference. Chấm độc lập từng câu trả lời trên cùng một rubric 1–5 có mốc rõ ràng, sau đó dùng judge thứ hai hoặc human review khi bất đồng. Rubric thưởng evidence, điều kiện bắt buộc và hành động an toàn, không thưởng độ dài; vì thế lặp lại, diễn giải dài hay chi tiết không có căn cứ không thể tăng điểm. Yêu cầu ưu tiên câu trả lời ngắn gọn và phạt rõ việc bịa chính sách, lộ dữ liệu riêng tư hay bỏ qua ngoại lệ.

### Exercise 3.4 — Framework Comparison (Bonus +10)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: RAGAS | Framework 2: DeepEval |
|---|---|---|
| Setup complexity | Cần chuẩn bị dataset theo schema và cấu hình evaluator/LLM cho các metric dựa trên LLM. | Dùng `LLMTestCase` và metric objects; gần với phong cách pytest. |
| Metrics available | Mạnh ở RAG: Context Precision/Recall, Faithfulness, Response Relevancy, Noise Sensitivity. | Có Answer Relevancy, Faithfulness/Hallucination, G-Eval và metric an toàn/tác vụ. |
| CI/CD integration | Có thể gọi trong script benchmark; cần tự đặt quality gate. | Tích hợp trực tiếp với pytest qua `deepeval test run`, phù hợp CI gate. |
| Kết quả trên cùng dataset | Thiết kế chạy: dùng 20 records và trace hiện có; chưa chạy vì cần evaluator LLM riêng. | Thiết kế chạy: cùng 20 records và threshold 0.5; chưa chạy vì cần judge model riêng. |
| Insight rút ra | Phù hợp để chẩn đoán retriever và evidence của RAG. | Phù hợp unit test, rubric LLM-as-a-Judge và CI/CD. |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:* Không nên so sánh trực tiếp score tuyệt đối giữa hai framework vì metric có prompt, judge model và cách chuẩn hóa khác nhau. RAGAS có tập metric RAG gồm Context Precision, Context Recall, Faithfulness và Response Relevancy; DeepEval cung cấp nhiều metric LLM-as-a-Judge và có tích hợp pytest/CI rõ ràng. Với lab này, dùng evaluator hiện có làm baseline; nếu thực hiện experiment thật, cần cố định 20 records, output artifact, judge model, prompt và threshold trước khi so sánh failure cases. [RAGAS metrics](https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/) và [DeepEval CI/CD](https://deepeval.com/docs/evaluation-unit-testing-in-ci-cd).

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
| M02 | 0.889 | 0.889 | 0.887 | 0.950 | +0.062 |
| M05 | 1.000 | 1.000 | 0.700 | 0.750 | +0.050 |
| M07 | 1.000 | 1.000 | 0.887 | 0.950 | +0.062 |
| H02 | 0.417 | 0.417 | 0.887 | 1.000 | +0.113 |
| A02 | 0.929 | 0.929 | 0.887 | 1.000 | +0.113 |
| **Avg** | **0.847** | **0.847** | **0.850** | **0.930** | **+0.080** |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:* Recall được tính trên hợp (union) token của cùng một tập chunks. Reranking chỉ đổi thứ tự chunks, không thêm hoặc xóa chunk, nên tập token và Context Recall không đổi.

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:* Reranking không đủ khi evidence cần thiết không được retrieve ngay từ đầu, khi chunks quá lớn/nhỏ làm mất ngữ cảnh, query không diễn đạt đúng intent hoặc metadata filter sai. Khi đó phải sửa retriever, query rewriting, chunking, top-k hay dữ liệu index trước khi rerank.

---

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [x] Tất cả required tests pass.
- [x] `golden_dataset.json` validate thành công.
- [x] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [x] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [x] Exercise 3.3 có rubric 1–5 và bias controls.
- [x] `reflection.md` có ba failure analyses và regression strategy.
- [x] Đã copy `template.py` thành `solution/solution.py`.
- [x] Exercise 3.4 và 3.5 là bonus, không nằm trong phần bắt buộc.
