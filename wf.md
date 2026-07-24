# 🖼️ Phase 3.1 — Current-State & Future-State Workflow Diagrams

**Dự án:** Trợ lý AI Điều vận & Xử lý Sự cố Pin Thực địa (GSM / Xanh SM)

**Đơn vị:** Vin Smart Future — Tập đoàn Vingroup

**Tác giả:** Nguyễn Văn An (Leader) & VinSmart AI Innovators

---

# 📊 1. Current-State Workflow Diagram (Sơ đồ Quy trình Hiện tại)

Sơ đồ thể hiện quy trình thủ công gồm **5 bước** mà điều phối viên (Dispatcher) thực hiện khi xử lý sự cố hết pin của tài xế Xanh SM.

```mermaid
graph TD
    classDef bottleneck fill:#7f1d1d,stroke:#ef4444,stroke-width:2px,color:#fff;
    classDef normal fill:#1e293b,stroke:#38bdf8,stroke-width:2px,color:#fff;
    classDef startend fill:#0f172a,stroke:#f59e0b,stroke-width:2px,color:#fff;

    START(["🚕 Tài xế gặp sự cố pin (SoC < 10%)"]):::startend

    B1["<b>Bước 1: Báo sự cố pin</b><br/>⏱️ 2 phút<br/><i>Tài xế gọi Hotline / Dispatcher mở Ticket</i>"]:::normal
    B2["<b>Bước 2: Tra định vị GPS</b><br/>⏱️ 2 phút<br/><i>Mở CRM Fleet Map kiểm tra tọa độ xe</i>"]:::normal
    B3["<b>Bước 3: Tra trạm sạc trống</b><br/>⏱️ 5 phút 🔴<br/><i>Đối chiếu loại trụ sạc VinFast tương thích</i>"]:::bottleneck
    B4["<b>Bước 4: Soạn SMS chỉ dẫn</b><br/>⏱️ 5 phút 🔴<br/><i>Viết hướng dẫn chỉ đường thủ công</i>"]:::bottleneck
    B5["<b>Bước 5: Chốt lệnh / Điều cứu hộ</b><br/>⏱️ 1 phút<br/><i>Gửi SMS hoặc điều xe cứu hộ</i>"]:::normal

    END(["🏁 Tài xế di chuyển tới trạm sạc hoặc xe cứu hộ xuất phát"]):::startend

    START --> B1
    B1 -->|"BKS xe & Hotline"| B2
    B2 -->|"Tọa độ GPS & SoC%"| B3
    B3 -->|"Danh sách trạm phù hợp"| B4
    B4 -->|"Draft SMS"| B5
    B5 --> END
```

---

# ⏱️ 2. Bảng Phân Tích Chi Tiết Quy Trình Thủ Công

| Bước | Tên tác vụ                 | Actor         | Thời gian | Hệ thống tích hợp        | Bottleneck?     | Mô tả khó khăn & rủi ro                                                                                          |
| ---: | -------------------------- | ------------- | --------- | ------------------------ | --------------- | ---------------------------------------------------------------------------------------------------------------- |
|    1 | Tiếp nhận cuộc gọi sự cố   | Tài xế / CSKH | 2 phút    | Hotline → CRM Ticket     | ❌              | Tài xế hoảng hốt khi `SoC < 10%`, vị trí báo qua điện thoại có thể không chính xác.                              |
|    2 | Tra cứu định vị GPS xe     | Dispatcher    | 2 phút    | Biển số → CRM Map        | ❌              | Dispatcher nhập biển số để tìm vị trí và mức pin thực tế.                                                        |
|    3 | Tra cứu trạm sạc VinFast   | Dispatcher    | 5 phút    | GPS → VinFast Dashboard  | 🔴 Bottleneck 1 | Lọc thủ công loại trụ sạc (60kW/150kW/250kW), kiểm tra tương thích với VF5/VF8/e34 và trạng thái trống realtime. |
|    4 | Soạn tin nhắn hướng dẫn    | Dispatcher    | 5 phút    | Raw Data → Driver App    | 🔴 Bottleneck 2 | Soạn hướng dẫn bằng tay, dễ sai tên đường hoặc khoảng cách.                                                      |
|    5 | Điều xe cứu hộ / Chốt lệnh | Dispatcher    | 1 phút    | Dispatcher → Rescue Team | ❌              | Nếu `SoC < 5%`, kích hoạt xe sạc pin lưu động.                                                                   |

### Tổng kết

- **Tổng thời gian xử lý:** **15 phút / lượt sự cố**
- **Thời gian Bottleneck:** **10 / 15 phút (66,7%)**

---

# 🔮 3. Future-State Workflow Diagram (Quy trình Có AI)

Quy trình sau khi bổ sung **LLM Feature** kết hợp **Human-in-the-Loop (HITL)**.

```mermaid
graph TD
    classDef aistep fill:#0369a1,stroke:#38bdf8,stroke-width:2px,color:#fff;
    classDef hitlstep fill:#15803d,stroke:#4ade80,stroke-width:2px,color:#fff;
    classDef fallbackstep fill:#b45309,stroke:#fbbf24,stroke-width:2px,color:#fff;
    classDef normal fill:#1e293b,stroke:#94a3b8,stroke-width:1px,color:#fff;

    F1["<b>Bước 1: Báo sự cố pin</b><br/>⏱️ 30 giây<br/><i>Tài xế nhấn nút khẩn trên App</i>"]:::normal

    F2["<b>Bước 2: Auto-pull Data</b> 🔵<br/>⏱️ 5 giây<br/><i>Tự lấy GPS & trạng thái trạm sạc</i>"]:::aistep

    F3["<b>Bước 3: AI Draft & Rule Check</b> 🔵<br/>⏱️ 10 giây<br/><i>Gemini tạo hướng dẫn + Rule SoC &lt; 5%</i>"]:::aistep

    F4{"<b>Bước 4: Dispatcher Review</b> 🟢<br/>⏱️ 15 giây<br/><i>Approve & Send</i>"}:::hitlstep

    FB["<b>Fallback Plan</b><br/><i>AI lỗi hoặc Timeout → Dispatcher xử lý thủ công</i>"]:::fallbackstep

    F5["<b>Bước 5: Gửi hướng dẫn</b><br/>⏱️ Gần như tức thì<br/><i>Driver App nhận lộ trình hoặc xe cứu hộ xuất phát</i>"]:::normal

    F1 --> F2
    F2 --> F3
    F3 -->|Success| F4
    F3 -.->|Timeout / API Error| FB
    FB --> F4
    F4 -->|Approve| F5
```

---

# 📈 4. Bảng So Sánh Hiệu Quả (KPI Impact)

| Chỉ số                             | Current State      | Future State         | Cải thiện             |
| ---------------------------------- | ------------------ | -------------------- | --------------------- |
| Thời gian xử lý 1 sự cố            | 15 phút            | < 1 phút             | 🟢 Giảm 93,3%         |
| Thời gian tra cứu & soạn hướng dẫn | 10 phút            | 15 giây              | 🟢 Giảm 97,5%         |
| Độ chính xác trạm sạc              | ~85%               | ≥98%                 | 🟢 Tăng 13%           |
| Năng suất Dispatcher               | 4 sự cố/giờ        | >40 sự cố/giờ        | 🟢 Tăng khoảng 10 lần |
| Xử lý `SoC < 5%`                   | Có nguy cơ sai sót | Rule + HITL + Rescue | 🛡️ An toàn hơn        |

---

# 🛠️ 5. Hướng dẫn xuất file PNG / PDF

## Cách 1 — Từ file SVG

1. Mở `04-workflow-diagram.svg` bằng Chrome, Edge hoặc VS Code.
2. Nhấn **Ctrl + P**.
3. Chọn **Save as PDF** hoặc chụp màn hình lưu thành:

```text
04-workflow-diagram.png
```

---

## Cách 2 — Dùng Python

Thực thi:

```bash
python generate_diagram_png.py
```

Kết quả sẽ sinh file:

```text
04-workflow-diagram.png
```

tại thư mục gốc của dự án.
