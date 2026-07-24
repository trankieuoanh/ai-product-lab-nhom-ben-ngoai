# 📝 Phase 6 — AI Log & Reflection (Nhật ký đồng hành cùng AI)

**Người thực hiện:** Trần Kiều Oanh

---

# 🤖 1. AI đã hỗ trợ tôi như thế nào? (AI Assistance)

Trong suốt quá trình thực hiện **Lab 02**, tôi sử dụng **Gemini** và **ChatGPT** như một **Thought Partner (Đối tác tư duy)** để hỗ trợ các công việc sau:

### Brainstorm bài toán vận hành

- Nhập vai AI Engineer tại Vin Smart Future để tìm kiếm các **bottleneck** trong quy trình vận hành.
- Xác định các bài toán thực tế như:
  - Điều phối xe điện Xanh SM.
  - Phân loại phản ánh cư dân Vinhomes.
  - Đánh giá mức độ phù hợp để ứng dụng AI.

### Thiết kế Prompt & JSON Schema

AI hỗ trợ:

- Thiết kế **System Prompt**.
- Xây dựng **Pydantic Schema** cho JSON Output.
- Chuẩn hóa cấu trúc phản hồi.
- Tối ưu các ràng buộc (Instruction & Guardrails).

### Tạo Adversarial Test Cases

Sử dụng AI để đóng vai một tài xế cố tình:

- jailbreak prompt,
- bypass Operational Boundary,
- đưa ra các tình huống khẩn cấp giả lập,

nhằm kiểm thử khả năng tuân thủ các quy tắc an toàn của hệ thống.

---

# ⚠️ 2. AI đã mắc sai lầm / ảo giác gì? (AI Hallucinations & Failures)

Trong quá trình làm việc, AI đã đưa ra một số đề xuất chưa phù hợp.

## 1. Đề xuất giải pháp quá tự trị (Over-agentic)

Ban đầu, AI đề xuất:

- tự động gửi SMS,
- tự động gọi xe cứu hộ,

mà không yêu cầu Dispatcher phê duyệt.

Điều này vi phạm nguyên tắc **Human-in-the-Loop (HITL)** của Xanh SM. Nếu AI xác định sai vị trí hoặc trạm sạc, xe có thể hết pin giữa đường và gây ảnh hưởng đến an toàn vận hành.

---

## 2. Không nhận diện đúng giới hạn kỹ thuật

Trong tình huống:

```text
SoC = 2%
Khoảng cách trạm sạc = 8 km
```

AI vẫn cố gắng đề xuất tài xế tiếp tục di chuyển đến trạm sạc.

Theo quy trình vận hành, đây là trường hợp phải **điều xe cứu hộ pin lưu động**, không được hướng dẫn tài xế tiếp tục lái xe.

---

# 🛠️ 3. Tôi đã điều chỉnh và khắc phục ra sao? (Iterations & Fixes)

## Bổ sung Operational Boundary

Tôi viết lại **System Instruction** bằng quy tắc logic rõ ràng:

```text
Nếu SoC < 5%
⇒ action = "dispatch_mobile_charger"
```

hoặc dưới dạng biểu thức:

```text
Nếu SoC < 5%
⇒ action = "dispatch_mobile_charger"
```

Điều này buộc AI phải ưu tiên phương án cứu hộ thay vì đề xuất trạm sạc.

---

## Bổ sung Human-in-the-Loop

Tất cả phản hồi của AI đều phải:

- có tiền tố:

```text
[DRAFT_ONLY]
```

- và luôn trả về:

```json
{
  "require_human_approval": true
}
```

Dispatcher là người đưa ra quyết định cuối cùng trước khi gửi hướng dẫn cho tài xế.

---

## Cải thiện Prompt thử nghiệm

Tôi bổ sung các ví dụ **Few-shot Prompting** cho các trường hợp:

- xe còn dưới 5% pin,
- trạm sạc quá xa,
- tài xế yêu cầu bỏ qua quy định,
- các tình huống jailbreak hoặc bypass.

Nhờ đó, mô hình hiểu rõ:

- khi nào cần từ chối yêu cầu,
- khi nào phải kích hoạt quy trình cứu hộ,
- và luôn tuân thủ các ràng buộc an toàn đã định nghĩa.
