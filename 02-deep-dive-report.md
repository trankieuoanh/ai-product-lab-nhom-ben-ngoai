# 🏗️ Phase 3 & 5 — AI Product Deep-Dive & Evaluation Report

## Thông tin nhóm

**Tên nhóm:** Nhom Ben Ngoai

### Thành viên

| Họ và tên      | MSSV        | Vai trò                     |
| -------------- | ----------- | --------------------------- |
| Trần Kiều Oanh | 2A202601417 | Leader / AI Prompt Engineer |

**Bài toán lựa chọn Deep-Dive:**  
**Card #1 — Xanh SM (GSM): Hệ thống Trợ lý Điều phối & Xử lý Sự cố Pin Khẩn cấp Thực địa**

---

# 🎯 1. Lý do lựa chọn bài toán

### Tính cấp thiết

Sự cố hết pin hoặc mức pin thấp (`SoC < 10%`) trực tiếp ảnh hưởng đến khả năng vận hành đội xe Xanh SM, làm giảm trải nghiệm khách hàng và có nguy cơ gây ùn tắc giao thông.

### Tác động kinh doanh

- Trung bình **80 sự cố/ngày** tại Hà Nội.
- Tiêu tốn hơn **20 giờ làm việc thủ công/ngày** của tổng đài điều vận.
- Làm thất thoát doanh thu do hủy chuyến.

### Độ khả thi kỹ thuật

- Dữ liệu GPS xe đã có sẵn.
- Trạng thái trạm sạc VinFast có API.
- Phù hợp triển khai bằng **LLM Feature** kết hợp **Rule-based Validation**.

---

# 📊 2. Problem Statement (6-field Standard)

| Field                       | Nội dung                                                                                                                                                                                                                                              |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Actor / Operator**     | Điều phối viên (Dispatcher) tại Trung tâm Điều vận Xe điện Xanh SM (GSM).                                                                                                                                                                             |
| **2. Current Workflow**     | Tài xế báo sự cố → Dispatcher mở CRM xem GPS → Tra cứu trạm sạc VinFast phù hợp dòng xe (VF5/e34/VF8) → Soạn hướng dẫn → Gọi cứu hộ nếu `SoC < 5%`.                                                                                                   |
| **3. Bottleneck**           | Tra cứu trạm sạc tương thích và soạn hướng dẫn thủ công (8–10 phút/lượt).                                                                                                                                                                             |
| **4. Business Impact**      | Lãng phí khoảng **20 giờ lao động/ngày**, tăng **15%** tỷ lệ hủy chuyến và làm tăng áp lực cho tài xế.                                                                                                                                                |
| **5. Success Metric**       | - Giảm thời gian xử lý từ **15 phút → dưới 3 phút**.<br>- Độ chính xác đề xuất trạm sạc **≥ 98%**.                                                                                                                                                    |
| **6. Operational Boundary** | **Được phép:** Truy vấn GPS, gọi API trạm sạc, tạo bản nháp hướng dẫn (`[DRAFT_ONLY]`).<br><br>**Không được phép:** AI tự gửi SMS/Push Notification khi chưa có xác nhận của Dispatcher; không đề xuất trạm sạc cách xe trên **5 km** nếu `SoC < 5%`. |

---

# 🔄 3. Future-State Workflow & AI Architecture

## AI Fit Level

**LLM Feature**

> Xử lý trích xuất ngữ cảnh, tổng hợp thông tin trạm sạc và sinh hướng dẫn chuẩn hóa.

### Quy trình vận hành tương lai

```text
┌────────────────┐     ┌────────────────────────┐     ┌────────────────────────┐
│ Bước 1         │     │ Bước 2 (🔵 AI Step)    │     │ Bước 3 (🔵 AI Step)    │
│                │ ──► │ Tự động truy vấn GPS   │ ──► │ LLM phân tích, lọc     │
│ Tài xế gửi     │     │ xe và API trạm sạc     │     │ trạm sạc phù hợp và    │
│ báo sự cố pin  │     │ VinFast lân cận        │     │ tạo Draft SMS          │
└────────────────┘     └────────────────────────┘     └────────────────────────┘
                                                                  │
                                                                  ▼
┌────────────────┐     ┌────────────────────────┐     ┌────────────────────────┐
│ Bước 5         │ ◄── │ Fallback               │ ◄── │ Bước 4 (🟢 HITL)       │
│                │     │                        │     │                        │
│ Tài xế nhận    │     │ Nếu AI lỗi hoặc        │     │ Dispatcher kiểm tra,   │
│ hướng dẫn qua  │     │ timeout thì Dispatcher │     │ chỉnh sửa và bấm       │
│ App Driver     │     │ xử lý thủ công         │     │ "Approve & Send"       │
└────────────────┘     └────────────────────────┘     └────────────────────────┘
```

---

# 🧪 4. Operational Boundaries & Adversarial Testing

Để đảm bảo an toàn vận hành, các ràng buộc được mã hóa trực tiếp trong **System Instruction** và **JSON Output**.

## Quy tắc 1 — Kiểm soát mức pin khẩn cấp

Nếu:

```text
SoC < 5%
```

AI **không được phép** hướng dẫn tài xế đến trạm sạc xa.

Thay vào đó phải trả về:

```json
{
  "action": "dispatch_mobile_charger"
}
```

để kích hoạt cứu hộ pin lưu động.

---

## Quy tắc 2 — Human-in-the-Loop (HITL)

Mọi phản hồi AI đều phải:

- Có tiền tố:

```text
[DRAFT_ONLY]
```

- Đồng thời trả về:

```json
{
  "require_human_approval": true
}
```

Dispatcher phải phê duyệt trước khi gửi cho tài xế.

---

# 🏁 5. Phase 5 — AI Readiness & Evaluation

## Checklist AI Readiness

- ✅ **Data Readiness**
  - Có API GPS xe.
  - Có API trạng thái realtime của trạm sạc VinFast.

- ✅ **Risk Control**
  - 100% thông tin gửi cho tài xế phải qua bước Human-in-the-Loop.

- ✅ **Stakeholder Readiness**
  - Điều phối viên Xanh SM sẵn sàng áp dụng công cụ AI để giảm tải giờ cao điểm.

---

## Quyết định của Ban Dự Án

- ✅ **GO**

**Phê duyệt phát triển Prototype v1.0**

---

## Justification

### 1. Chi phí - Lợi ích

- Chi phí Gemini 2.5 Flash:

```text
≈ 0.00015 USD / lượt xử lý
```

- Tiết kiệm:
  - khoảng **80%** thời gian xử lý thủ công.
  - tương đương khoảng **600 USD/tháng** chi phí nhân công trực tiếp.
  - giảm nguy cơ bồi thường do hủy chuyến.

### 2. An toàn vận hành

Nguy cơ hallucination của LLM được kiểm soát bằng hai lớp:

1. **Rule Engine**

```text
IF SoC < 5%
→ dispatch_mobile_charger
```

2. **Human-in-the-Loop**

```text
LLM Draft
      ↓
Dispatcher Review
      ↓
Approve & Send
```

Nhờ đó AI chỉ đóng vai trò **Copilot hỗ trợ điều phối**, không tự động đưa ra quyết định cuối cùng.
