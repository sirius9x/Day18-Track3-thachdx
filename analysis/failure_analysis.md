# Failure Analysis — Lab 18: Production RAG

**Nhóm:** [Tên nhóm]  
**Thành viên:** [Tên 1 → M1] · [Tên 2 → M2] · [Tên 3 → M3] · [Tên 4 → M4]

---

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | 0.8258 | 0.6750 | -0.1508 |
| Answer Relevancy | 0.7630 | 0.6759 | -0.0871 |
| Context Precision | 0.9250 | 0.9292 | +0.0042 |
| Context Recall | 0.9250 | 0.8083 | -0.1167 |

## Bottom-5 Failures

### #1
- **Question:** Thông tin lương thuộc cấp độ phân loại dữ liệu nào?
- **Expected:** Theo quy chế chi trả lương, thông tin lương được phân loại là dữ liệu Bí mật, cấm chia sẻ với đồng nghiệp. Theo chính sách phân loại dữ liệu, dữ liệu Bí mật (cấp 3) phải mã hóa khi truyền và hạn chế truy cập theo need-to-know.
- **Got:** (Xem chi tiết log sinh câu trả lời)
- **Worst metric:** Faithfulness (0.0)
- **Error Tree:** Output sai → Context đúng? → Query OK? →
- **Root cause:** LLM hallucinating
- **Suggested fix:** Tighten prompt, lower temperature

### #2
- **Question:** Bao lâu phải đổi mật khẩu một lần?
- **Expected:** Theo chính sách hiện hành (v2.0), mật khẩu phải được thay đổi mỗi 120 ngày. Chính sách cũ yêu cầu 90 ngày nhưng đã bị thay thế.
- **Got:** (Xem chi tiết log)
- **Worst metric:** Faithfulness (0.0)
- **Error Tree:** Output sai → Context đúng? → Query OK? →
- **Root cause:** LLM hallucinating
- **Suggested fix:** Tighten prompt, lower temperature

### #3
- **Question:** Mật khẩu phải có tối thiểu bao nhiêu ký tự?
- **Expected:** Theo chính sách hiện hành (v2.0), mật khẩu phải có tối thiểu 12 ký tự. Chính sách cũ (v1.0) yêu cầu 8 ký tự nhưng đã bị thay thế.
- **Got:** (Xem chi tiết log)
- **Worst metric:** Faithfulness (0.0)
- **Error Tree:** Output sai → Context đúng? → Query OK? →
- **Root cause:** LLM hallucinating
- **Suggested fix:** Tighten prompt, lower temperature

### #4
- **Question:** Lương thử việc của nhân viên Junior mức cao nhất là bao nhiêu?
- **Expected:** Junior cao nhất là 20.000.000 VNĐ/tháng. Lương thử việc = 85% x 20.000.000 = 17.000.000 VNĐ/tháng.
- **Got:** (Xem chi tiết log)
- **Worst metric:** Faithfulness (0.0)
- **Error Tree:** Output sai → Context đúng? → Query OK? →
- **Root cause:** LLM hallucinating
- **Suggested fix:** Tighten prompt, lower temperature

### #5
- **Question:** Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt?
- **Expected:** Đơn hàng trên 50.000.000 VNĐ cần Tổng Giám đốc (CEO) phê duyệt.
- **Got:** (Xem chi tiết log)
- **Worst metric:** Context Recall (0.0)
- **Error Tree:** Output đúng? → Context sai? → Query rewrite OK? →
- **Root cause:** Missing relevant chunks
- **Suggested fix:** Improve chunking or add BM25

## Case Study (cho presentation)

**Question chọn phân tích:**

**Error Tree walkthrough:**
1. Output đúng? →
2. Context đúng? →
3. Query rewrite OK? →
4. Fix ở bước:

**Nếu có thêm 1 giờ, sẽ optimize:**
-
