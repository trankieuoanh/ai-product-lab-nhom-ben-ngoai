# Phase 1 — SCAN (5 Bài toán Vận hành Vingroup)

| #   | Subsidiary | Lens               | Mô tả ngắn bài toán / Bottleneck                                                                                                            |
| --- | ---------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Xanh SM    | Tốn thời gian      | Điều phối viên xử lý thủ công sự cố sạc pin / xe sắp hết pin thực địa của tài xế (mất 12–15 phút/lượt).                                     |
| 2   | Vinhomes   | Lặp lại            | Phân loại và điều hướng tự động phản ánh/khiếu nại của cư dân trên App Vinhomes Resident tới đúng BQL tòa nhà.                              |
| 3   | VinFast    | Lặp lại            | Đối chiếu tự động hóa đơn sạc điện hằng tuần từ các trạm sạc đối tác liên kết với dữ liệu xe thực tế.                                       |
| 4   | Vinmec     | Pain từ người khác | Bác sĩ tốn 20–30 phút/bệnh nhân để tóm tắt hồ sơ xuất viện (Discharge Summary) từ ghi chú lâm sàng và kết quả xét nghiệm.                   |
| 5   | Vinpearl   | AI-upgrade         | Tổng hợp và phân loại các phản hồi khẩn cấp (1–2 sao) từ khách hàng trên các nền tảng OTA (Booking, Agoda) để cảnh báo cho General Manager. |

---

# 🃏 Phase 2 — QUICK-ASSESS (3 Quick Problem Cards)

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán: Điều phối & gợi ý trạm sạc khẩn cấp khi xe hết pin│
│ giữa đường đón khách.                                       │
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác _________________  │
│                                                             │
│ Ai đang đau?                                                 │
│ Tài xế (lo cạn pin), Điều phối viên (quá tải).              │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Tài xế gọi Hotline báo SoC < 10%                       │
│   → 2. Điều phối viên tra GPS xe trên CRM                   │
│   → 3. Tìm trạm sạc VinFast còn trụ tương thích             │
│   → 4. Soạn hướng dẫn đường đi gửi tài xế                   │
│   → 5. Gọi xe cứu hộ nếu SoC < 5%                           │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất?                            │
│ Bước 3 & 4 (⏱ ~10 phút/lượt)                                │
│                                                             │
│ AI có thể nhảy vào hỗ trợ ở bước nào?                       │
│ Bước 3 & 4 (Auto pull data + Draft SMS)                     │
│                                                             │
│ Đo thành công bằng gì?                                      │
│ - Giảm thời gian xử lý từ 15 phút → dưới 3 phút.            │
│ - Độ chính xác gợi ý trạm sạc ≥ 98%.                        │
│                                                             │
│ Quick Architecture: [ ] No AI [ ] Rule [x] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Phân loại & Routing phản ánh cư dân Vinhomes.     │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [x] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác _________________  │
│                                                             │
│ Ai đang đau?                                                 │
│ Cư dân (chờ lâu), BQL tòa nhà (ngập ticket).                │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Cư dân gửi phản ánh trên App Resident                  │
│   → 2. CSKH đọc và phân loại thủ công                       │
│   → 3. Chuyển ticket tới đúng phòng ban                     │
│   → 4. Gửi xác nhận tiếp nhận cho cư dân                    │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất?                            │
│ Bước 2 & 3 (⏱ 45–120 phút/ticket)                           │
│                                                             │
│ AI có thể nhảy vào hỗ trợ ở bước nào?                       │
│ Bước 2 & 3 (Auto Classification + Auto Routing)             │
│                                                             │
│ Đo thành công bằng gì?                                      │
│ - Giảm thời gian route từ 60 phút → dưới 2 phút.            │
│ - Độ chính xác phân loại ≥ 92%.                             │
│                                                             │
│ Quick Architecture: [ ] No AI [ ] Rule [x] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán: Soạn thảo tóm tắt hồ sơ xuất viện.                │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [ ] Vinhomes  │
│                     [x] Vinmec   [ ] Khác _________________  │
│                                                             │
│ Ai đang đau?                                                 │
│ Bác sĩ lâm sàng (quá tải công việc hành chính).             │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Mở hồ sơ bệnh án điện tử (EMR)                         │
│   → 2. Đọc ghi chú điều trị, xét nghiệm, đơn thuốc          │
│   → 3. Soạn tóm tắt xuất viện                               │
│   → 4. Rà soát, ký duyệt và in                              │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất?                            │
│ Bước 2 & 3 (⏱ ~25 phút/bệnh nhân)                           │
│                                                             │
│ AI có thể nhảy vào hỗ trợ ở bước nào?                       │
│ Bước 2 & 3 (Extract Information + Draft Summary)            │
│                                                             │
│ Đo thành công bằng gì?                                      │
│ - Giảm thời gian từ 25 phút → dưới 5 phút.                  │
│ - Bác sĩ chỉ cần chỉnh sửa <10% nội dung.                   │
│                                                             │
│ Quick Architecture: [ ] No AI [ ] Rule [x] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```
