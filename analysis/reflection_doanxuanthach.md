# Individual Reflection — Lab 18

**Tên:** Đoàn Xuân Thạch

**MSSV:** 2A202600950

**Module phụ trách:** Toàn bộ (M1, M2, M3, M4, M5)

---

## 1. Đóng góp kỹ thuật

- Module đã implement: M1 (Chunking), M2 (Search), M3 (Rerank), M4 (Eval), M5 (Enrichment).
- Các hàm/class chính đã viết: 
  - `chunk_semantic()`, `chunk_hierarchical()`, `chunk_structure_aware()`
  - `HybridSearch` (kết hợp BM25 và Dense Search qua `reciprocal_rank_fusion()`)
  - `CrossEncoderReranker.rerank()`
  - `evaluate_ragas()`
  - Các hàm gọi OpenAI API trong M5: `summarize_chunk()`, `generate_hypothesis_questions()`, `contextual_prepend()`, `extract_metadata()`, `_enrich_single_call()`.
- Số tests pass: 37/37

## 2. Kiến thức học được

- Khái niệm mới nhất: Kỹ thuật Enrichment như Hypothesis Question-Answer (HyQA) và Contextual Prepend giúp bridge được khoảng cách từ vựng (vocabulary gap) giữa câu hỏi của user và raw text.
- Điều bất ngờ nhất: Điểm số của Production RAG (đặc biệt là Faithfulness và Context Recall) lại bị giảm so với Naive Baseline. Lý do chính là khi gọi Enrichment API (M5) bị lỗi parse JSON hàng loạt do giới hạn timeout/rate-limit của OpenAI, khiến nhiều chunks bị mất hoặc metadata không được format đúng, từ đó làm giảm chất lượng retrieval.
- Kết nối với bài giảng (slide nào): Semantic chunking, Hybrid Search kết hợp BM25 và Vector, Cross-encoder Reranking để đẩy relevant documents lên top.

## 3. Khó khăn & Cách giải quyết

- Khó khăn lớn nhất: Việc thiết kế kiến trúc truy xuất (Retrieval) giữa M1 (Hierarchical Chunking) và M2 (Hybrid Search). Khi chia văn bản thành các Child chunks nhỏ (256 ký tự) để search chính xác hơn, hệ thống lại cung cấp trực tiếp mảnh Child ngắn ngủi này cho LLM thay vì trích xuất ngược lại Parent chunk lớn. Điều này vô tình bẻ vụn văn bản, khiến Context truyền vào bị thiếu hụt bối cảnh trầm trọng. Ngoài ra, sự chênh lệch từ vựng (vocabulary gap) giữa ngôn ngữ câu hỏi của người dùng và từ ngữ trong tài liệu hành chính cũng cản trở thuật toán Dense Search.
- Cách giải quyết: 
  - Về mặt thuật toán, áp dụng Reciprocal Rank Fusion (RRF) ở M2 để bù đắp điểm yếu của Dense search bằng cách kết hợp chính xác từ khoá của BM25.
  - Về mặt xử lý dữ liệu, triển khai các kỹ thuật Enrichment (M5) như Contextual Prepend và HyQA để tự động làm giàu ngữ nghĩa cho từng chunk trước khi Index vào cơ sở dữ liệu.
- Thời gian nghiên cứu và debug giải thuật: ~2 tiếng.

## 4. Nếu làm lại

- Sẽ làm khác điều gì: Ưu tiên sửa lại luồng truy vấn (Retrieval Flow) kết nối giữa M1 và M2: Khi Search trả về top các Child chunks, em sẽ lấy `parent_id` để query và nạp toàn bộ Parent chunks (2048 ký tự) vào Context cho LLM. Việc này sẽ giải quyết tận gốc vấn đề "Hallucination" và kéo điểm Faithfulness lên cao. Ngoài ra ở M5, em sẽ thiết lập `response_format={"type": "json_object"}` để API OpenAI trả về JSON chuẩn, tránh lỗi vỡ cấu trúc metadata do Markdown.
- Module nào muốn thử tiếp: Áp dụng mô hình tối ưu như FlashRank thay cho Cross-Encoder ở Module 3 (Rerank) nhằm tăng tốc độ suy luận. Đồng thời tối ưu thử nghiệm Tuning tham số k trong thuật toán RRF ở Module 2 để cân bằng tốt hơn giữa BM25 và Vector Search.

## 5. Tự đánh giá

| Tiêu chí | Tự chấm (1-5) |
|----------|---------------|
| Hiểu bài giảng | 5 |
| Code quality | 4 |
| Teamwork | 5 |
| Problem solving | 5 |
