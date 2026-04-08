# CHI TIẾT LUỒNG NGHIỆP VỤ – SERVICE 7 (Routine_Checkup): KHÁM SỨC KHỎE ĐỊNH KỲ (ROUTINE HEALTH CHECKUP)

## Header

- **Service:** Routine Health Checkup (Quản lý Sức khỏe Định kỳ)
- **Version:** 1.0
- **Input:** Hệ thống phát hiện checkup mới hoàn tất (trigger-based)
- **Output:** Health Plan cá nhân hóa (AI hoặc MD-approved) + Cập nhật EHR + Lịch chu kỳ tiếp theo
- **Compliance:** HIPAA, ICD-10, LOINC
- **Điều kiện sử dụng:**
  - KH PHẢI đã mua gói dịch vụ có bao gồm Service Quản lý Sức khỏe Định kỳ
  - KH PHẢI có tài khoản CVH đang hoạt động
  - KH PHẢI có dữ liệu EHR từ các dịch vụ trước (Service 4 (Chronic_Management)/CDM, v.v.)
  - Trigger: KH vừa hoàn tất cuộc khám mới với CVH (dữ liệu mới được cập nhật vào EHR)
  - Toàn bộ dữ liệu xử lý là PHI — tuân thủ HIPAA

---

## Nguyên tắc cốt lõi

- **AI-First:** AI phân tích, tạo Plan, quyết định routing
- **Dual-Branch Automation:** Branch A (AI trực tiếp) vs Branch B (MD review)
- **Trigger-Based Activation:** Hệ thống tự phát hiện (polling mỗi 60s), không cần KH kích hoạt
- **Periodic Cycle:** Tự động lên lịch chu kỳ tiếp theo dựa trên mức độ rủi ro
- **PHI Protection:** HIPAA encrypted PDF

---

## Danh sách Flow

| Flow | Tên | Mô tả |
|------|-----|-------|
| FLOW-01 Branch A | AI Plan trực tiếp | Kết quả bình thường → AI gửi Plan cho KH ngay |
| FLOW-01 Branch B | MD Review | Kết quả bất thường → MD US review + 3 options |

---

## PHẦN CHUNG (Bước 0-6)

### Bước 0. Hệ thống (Trigger) – Phát hiện checkup mới hoàn tất

- Polling mỗi 60s, SLA: phát hiện trong 3 phút, accuracy 100%

- **INPUT:** checkup_completion_event (Object, required), patient_id (String, required), subscription_package (Object, required)
- **OUTPUT:** trigger_activated (Boolean), trigger_timestamp (Date)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| subscription_package | KH PHẢI có gói dịch vụ bao gồm Service 7 (Routine_Checkup) | "KH chưa mua gói dịch vụ có Service này" |
| checkup_completion_event | PHẢI có dữ liệu khám mới cập nhật vào EHR | "Không phát hiện dữ liệu khám mới" |

**UI States:**
- Màn hình nội bộ hệ thống — KH KHÔNG thấy
- Running: Polling liên tục mỗi 60s
- Triggered: Phát hiện checkup mới → kích hoạt flow
- Blocked: KH không có gói dịch vụ → KHÔNG kích hoạt

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| KH có gói dịch vụ + checkup mới | Kích hoạt → Bước 1 |
| KH chưa mua gói có Service này | KHÔNG kích hoạt Service |

---

### Bước 1. Hệ thống – Thu thập dữ liệu (SLA: ≤ 30s)

- Kéo toàn bộ kết quả checkup mới: lịch sử khám, chẩn đoán, đơn thuốc, chỉ số sinh hiệu, kết quả CLS
- Auto-retry 3 lần nếu DB lỗi (mỗi lần cách 5 giây)

- **INPUT:** trigger_event (Object, required), patient_id (String, required)
- **OUTPUT:** new_checkup_data (Object: {cls_results, vital_signs, clinical_notes, diagnoses, prescriptions}, required)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| new_checkup_data | Dữ liệu PHẢI đầy đủ, không null, không duplicate | "Dữ liệu thu thập không đầy đủ" |
| patient_id | PHẢI tồn tại trong hệ thống | "Không tìm thấy bệnh nhân" |

**UI States:**
- Màn hình nội bộ — KH KHÔNG thấy
- Loading: "Đang thu thập dữ liệu từ cuộc khám mới..."
- Success: ✅ hiển thị từng mục đã thu thập (CLS, sinh hiệu, ghi chú)
- Error: DB lỗi → auto retry 3 lần

⚠️ **Lỗi:** DB failure sau 3 retry → alert Dev Team → CS nhận leo thang nếu Dev không xử lý > 10 phút

---

### Bước 2. Hệ thống – Truy xuất EHR (SLA: ≤ 20s)

- Truy xuất và tổng hợp toàn bộ lịch sử sức khỏe của KH từ hệ thống EHR
- Kiểm tra tính toàn vẹn dữ liệu (nhân khẩu học, tiền sử bệnh, thuốc, dị ứng, kết quả xét nghiệm, chỉ số sinh tồn)
- Auto-retry 3 lần

- **INPUT:** patient_id (String, required)
- **OUTPUT:** ehr_full_record (Object: {demographics, medical_history, medications, allergies, lab_results, vital_signs}, required)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| ehr_full_record | PHẢI có ít nhất 1 lần khám trước đó trong EHR | "Không đủ dữ liệu EHR để lập Plan định kỳ" |
| ehr_sections | PHẢI có đầy đủ các mục bắt buộc (nhân khẩu học, tiền sử bệnh, thuốc, dị ứng, kết quả xét nghiệm, chỉ số sinh tồn) | "EHR thiếu dữ liệu bắt buộc" |

⚠️ **Lỗi:** EHR thiếu section quan trọng → alert Dev Team → CS leo thang nếu Dev không phản hồi

---

### Bước 3. Hệ thống (AI) – Xử lý / Tổng hợp (SLA: ≤ 20s)

- Chuẩn hóa theo ICD-10, LOINC
- Xử lý, chuẩn hóa, tổng hợp dữ liệu mới với toàn bộ EHR — tạo health profile tổng quan
- Kiểm tra logic dữ liệu: giá trị ngoài phạm vi hợp lý → đánh dấu outlier
- Đối chiếu với phạm vi tham chiếu theo tuổi/giới tính

- **INPUT:** new_checkup_data (Object, required), ehr_full_record (Object, required)
- **OUTPUT:** processed_data (Object: {health_profile, historical_comparison, trend_indicators}, required)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| processed_data | Dữ liệu PHẢI đầy đủ, không có trường rỗng bắt buộc | "Dữ liệu xử lý không đầy đủ" |
| unit_mapping | Chuẩn hóa đơn vị đo PHẢI đúng chuẩn y khoa (ICD-10, LOINC) | "Lỗi mapping đơn vị đo" |

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Dữ liệu xử lý thành công | Chuyển bước 4 |
| AI flag dữ liệu y tế có vấn đề (outlier, mâu thuẫn) | Flag cho **MD** review mapping, không dừng quy trình |

⚠️ **Lỗi:** AI sập > 10 phút → CS nhận alert → cảnh báo Dev Team & Manager

---

### Bước 4. Hệ thống (AI) – Phân tích / Đánh giá (SLA: ≤ 30s, accuracy ≥ 95% vs MD audit)

- Đánh giá toàn diện sức khỏe: so sánh với lịch sử, xác định xu hướng, phát hiện bất thường, risk factors
- Phân loại mức độ nguy cơ: thấp/trung bình/cao/nguy hiểm
- Critical alerts → gửi trực tiếp cho **MD**

- **INPUT:** processed_data (Object, required)
- **OUTPUT:** analysis_report (Object: {abnormalities, risk_factors, trends, risk_level, critical_alerts}, required)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| analysis_report | Báo cáo PHẢI xác định rõ: bất thường / không bất thường | "Không xác định được tình trạng sức khỏe" |
| risk_level | PHẢI là thấp/trung bình/cao/nguy hiểm | "Mức độ nguy cơ không hợp lệ" |
| confidence | Kết quả confidence < 85% → tự động flag để MD review | "Độ tin cậy phân tích thấp" |

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Phân tích hoàn tất bình thường | Chuyển bước 5 |
| Phát hiện chỉ số CRITICAL | Alert ngay cho MD (không qua CS) |
| Phát hiện tình trạng cấp cứu | Chuyển ngay sang Emergency Protocol |

⚠️ **Lỗi:** AI crash → CS báo kỹ thuật

---

### Bước 5. Hệ thống (AI) – Tạo Health Plan (SLA: ≤ 30s)

- Theo CVH medical template, song ngữ (VN-EN)
- Lập Plan cho CẢ 2 luồng trước khi xác định luồng
- Nội dung Plan: khuyến nghị thuốc, lối sống, xét nghiệm bổ sung, biện pháp phòng ngừa
- Đề xuất chu kỳ (6 hoặc 12 tháng) dựa trên tình trạng sức khỏe

- **INPUT:** analysis_report (Object, required)
- **OUTPUT:** health_plan (Object: {overall_status, key_findings, medication_recommendations, lifestyle_recommendations, additional_tests, preventive_measures, next_cycle, warning_signs}, required), proposed_cycle (Number: 6|12, required)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| health_plan | Plan PHẢI có ít nhất: khuyến nghị + chu kỳ | "Plan không đầy đủ thông tin bắt buộc" |
| proposed_cycle | CHỈ được chọn 6 tháng hoặc 12 tháng | "Chu kỳ không hợp lệ" |
| plan_sections | Kiểm tra tất cả sections bắt buộc có đầy đủ | "Tóm tắt thiếu section bắt buộc" |
| terminology | Thuật ngữ y khoa PHẢI đúng chuẩn | "Lỗi thuật ngữ y tế" |
| bilingual | Song ngữ Việt-Anh PHẢI đúng và đủ | "Thiếu bản dịch song ngữ" |

---

### Bước 6. Hệ thống (AI) – Xác định luồng routing (SLA: ≤ 15s, accuracy ≥ 98%)

- Áp dụng cây quyết định dựa trên mức độ rủi ro, phát hiện bất thường, xu hướng
- Mặc định khi không chắc chắn → Branch B (safe approach)

- **INPUT:** analysis_report (Object, required)
- **OUTPUT:** routing_decision (String: "BRANCH_A"|"BRANCH_B", required), routing_reason (String, required)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| routing_decision | PHẢI là A hoặc B — không có giá trị khác | "Không xác định được luồng xử lý" |

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Tất cả chỉ số bình thường, xu hướng ổn định, không triệu chứng mới | → Branch A (gửi trực tiếp) |
| Có chỉ số bất thường, xu hướng xấu đi, triệu chứng mới, cần MD đánh giá | → Branch B (MD review) |
| AI không chắc chắn (borderline) | → Branch B (mặc định an toàn) |

**Wireframe ASCII (Bước 1-6):**

```
┌─────────────────────────────────────────────────────────────────┐
│                  ⚙️ XỬ LÝ DỮ LIỆU SỨC KHỎE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🔄 Đang thu thập dữ liệu từ cuộc khám mới...                  │
│     ✅ Kết quả xét nghiệm (CLS)                                │
│     ✅ Chỉ số sinh hiệu                                        │
│     ✅ Ghi chú lâm sàng                                        │
│                                                                 │
│  🔄 Đang truy xuất EHR đầy đủ...                               │
│     ✅ Lịch sử khám bệnh                                       │
│     ✅ Chẩn đoán y tế                                          │
│     ✅ Đơn thuốc hiện tại                                       │
│     ✅ Lịch sử xét nghiệm                                      │
│                                                                 │
│  🤖 AI đang xử lý và tổng hợp dữ liệu...                      │
│     ████████████████░░░░  75%                                   │
│                                                                 │
│  🔒 Dữ liệu được xử lý trong môi trường bảo mật HIPAA         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────┐
│                  🤖 AI PHÂN TÍCH SỨC KHỎE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📊 Phân tích tình trạng sức khỏe:                              │
│     • So sánh chỉ số với lịch sử khám                          │
│     • Xác định xu hướng (cải thiện / ổn định / xấu đi)        │
│     • Phát hiện bất thường                                     │
│     • Đánh giá risk factors                                    │
│                                                                 │
│  📋 Lập Plan định kỳ:                                           │
│     • Khuyến nghị thuốc (tiếp tục / điều chỉnh)               │
│     • Lối sống (chế độ ăn, vận động, giấc ngủ)                │
│     • Xét nghiệm bổ sung (nếu cần)                            │
│     • Biện pháp phòng ngừa                                     │
│                                                                 │
│  ⏱️ Đề xuất chu kỳ: [6 tháng] / [12 tháng]                    │
│                                                                 │
│  🔀 Xác định luồng:                                            │
│     ┌──────────────────────────────────────────────────┐       │
│     │ Kết quả: [Nhánh A — Gửi trực tiếp]              │       │
│     │     hoặc [Nhánh B — MD Review cần thiết]          │       │
│     └──────────────────────────────────────────────────┘       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## BRANCH A: AI PLAN TRỰC TIẾP

- Không có bước bổ sung
- AI gửi Plan cho KH ngay → chuyển Phần chung cuối (bước 7-8)

---

## BRANCH B: MD REVIEW (3 OPTIONS)

### CS thông báo + đặt lịch

- **Hệ thống** – Thông báo KH cần MD review + CS chủ động gọi KH trong 2h
  - Đặt lịch hẹn trong 48h

- **INPUT:** patient_info (Object, required), analysis_report (Object, required), health_plan (Object, required)
- **OUTPUT:** notification_sent (Boolean), booking_status (String), appointment_datetime (Date)

**UI States:**
- Auto: App push + Email gửi ngay cho KH (nội dung: kết quả cần tư vấn, mời đặt lịch)
- Booking: Hiển thị danh sách slot trống MD US trong 48h tới trên app
- Waiting: KH dừng > 60s → tự gửi link hướng dẫn đặt lịch qua chatbox
- Reminder: Sau 24h KH chưa đặt lịch → reminder lần 2

⚠️ **Lỗi:**
- CS xử lý no-show
- CS gọi lại sau 4h, 24h nếu KH không trả lời
- KH từ chối/không liên lạc được sau 48h → CS báo MD

---

### Option 1: Review + Confirm (nhanh) — Bước B1a

- **MD US** – Review Plan AI đã tạo

- **INPUT:** health_plan (Object, required), analysis_report (Object, required), patient_info (Object, required)
- **OUTPUT:** plan_approved (Boolean), md_notes (String, optional)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| option_selection | MD PHẢI chọn 1 trong 3 options | "Vui lòng chọn cách xử lý" |
| response_time | PHẢI phản hồi trong 24h | "Nhắc nhở: Plan đang chờ review" |

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| MD đồng ý Plan AI | Approve (e-sign) → gửi Plan → Bước 7 |
| MD cần chỉnh sửa | Chuyển → Option 3 |

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────────┐
│                  🔔 THÔNG BÁO MỚI — MD Dashboard                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ⚠️ PLAN ĐỊNH KỲ CẦN REVIEW                                    │
│                                                                 │
│  Bệnh nhân: [Họ tên KH]                                        │
│  Mã BN:     [CVH-XXXX-XXXXXX]                                  │
│  Ngày khám:  [DD/MM/YYYY]                                      │
│                                                                 │
│  📊 Tóm tắt phân tích AI:                                      │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ • Tình trạng tổng thể: [Cần theo dõi]               │       │
│  │ • Chỉ số bất thường: [Danh sách chỉ số]             │       │
│  │ • Xu hướng: [Mô tả xu hướng]                        │       │
│  │ • AI đề xuất chu kỳ: [6 tháng / 12 tháng]           │       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                 │
│  Chọn cách xử lý:                                              │
│  ┌──────────────────┐ ┌──────────────────┐ ┌────────────────┐  │
│  │ ✅ Review +       │ │ 📞 Tạo Video    │ │ ✏️ Cập nhật    │  │
│  │    Xác nhận      │ │    Call          │ │    Plan        │  │
│  │  (Option 1)      │ │  (Option 2)     │ │  (Option 3)    │  │
│  └──────────────────┘ └──────────────────┘ └────────────────┘  │
│                                                                 │
│  [XEM PLAN ĐẦY ĐỦ]    [XEM EHR]    [XEM BÁO CÁO PHÂN TÍCH]  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────┐
│                  ✅ REVIEW PLAN ĐỊNH KỲ                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Bệnh nhân: [Họ tên KH]        Mã BN: [CVH-XXXX-XXXXXX]       │
│                                                                 │
│  ──────────────────────────────────────────────────────────     │
│  📋 NỘI DUNG PLAN ĐỊNH KỲ (AI-generated)                       │
│  ──────────────────────────────────────────────────────────     │
│                                                                 │
│  1. Tình trạng sức khỏe tổng quan: [...]                        │
│  2. Phát hiện chính: [...]                                      │
│  3. Khuyến nghị thuốc: [...]                                    │
│  4. Khuyến nghị lối sống: [...]                                 │
│  5. Xét nghiệm bổ sung: [...]                                  │
│  6. Biện pháp phòng ngừa: [...]                                 │
│  7. Chu kỳ tiếp theo: [6/12 tháng]                             │
│                                                                 │
│  ──────────────────────────────────────────────────────────     │
│                                                                 │
│  Ghi chú MD (tùy chọn): [_______________________________]      │
│                                                                 │
│  [   ✅ XÁC NHẬN VÀ GỬI CHO KH   ]   [  ← QUAY LẠI  ]       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Option 2: Video Call (chi tiết) — Bước B1b → B2 → B3

#### Bước B1b: Tạo Video Call

- **MD US** – Quyết định cần giải thích trực tiếp, tạo lịch Video Call

- **INPUT:** health_plan (Object, required), patient_id (String, required)
- **OUTPUT:** call_schedule (Object: {date, time, duration}, required), call_notes (String, optional)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| call_date | Ngày call PHẢI trong tương lai | "Ngày call không hợp lệ" |
| call_duration | Thời lượng 20-30 phút | "Thời lượng call phải từ 20-30 phút" |

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────────┐
│                  📞 TẠO LỊCH VIDEO CALL                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Bệnh nhân: [Họ tên KH]        Mã BN: [CVH-XXXX-XXXXXX]       │
│                                                                 │
│  📅 Chọn thời gian call:                                        │
│     Ngày:  [DD/MM/YYYY  ▼]                                     │
│     Giờ:   [HH:MM       ▼]                                     │
│                                                                 │
│  ⏱️ Thời lượng dự kiến: 20-30 phút                              │
│                                                                 │
│  📝 Ghi chú cho cuộc call (nội dung cần giải thích):            │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ [                                                    ]│       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                 │
│  [   📞 TẠO LỊCH CALL   ]        [  HỦY  ]                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Bước B2: Video Call với KH (20-30 phút)

- **MD US** – Video Call:
  - Review kết quả chi tiết
  - Phỏng vấn lâm sàng
  - Giải thích + tư vấn
  - Thống nhất kế hoạch theo dõi

- **INPUT:** health_plan (Object, required), analysis_report (Object, required), patient_info (Object, required)
- **OUTPUT:** call_completed (Boolean), call_duration (Number), call_notes (String)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| call_recording_consent | Call PHẢI được ghi nhận (consent + HIPAA) | "Chưa có consent ghi nhận cuộc gọi" |

**UI States:**
- Preparing: Tự load Plan AI + EHR lên giao diện MD trước giờ call
- Waiting: KH không join sau 10 phút → push notification + alert CS
- In-call: Video đang diễn ra, giám sát chất lượng kết nối
- Ended: Tự gửi khảo sát satisfaction

⚠️ **Lỗi:**
- MD trễ > 5 phút → CS alert
- Lỗi kết nối → CS hỗ trợ (tạo link dự phòng, gửi SMS)
- KH no-show → CS liên hệ reschedule trong 2h

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────────┐
│                  📞 VIDEO CALL — GIẢI THÍCH PLAN                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌────────────────────────────────────────────────┐             │
│  │                                                │             │
│  │              📹 VIDEO CALL                      │             │
│  │           MD ↔ KH (20-30 phút)                 │             │
│  │                                                │             │
│  └────────────────────────────────────────────────┘             │
│                                                                 │
│  📋 Plan AI hiển thị bên cạnh (để MD tham chiếu)               │
│  📊 Báo cáo phân tích hiển thị (để MD giải thích)              │
│                                                                 │
│  Nội dung call:                                                 │
│  • MD giải thích Plan và các chỉ số bất thường                 │
│  • KH đặt câu hỏi, thắc mắc                                   │
│  • MD giải đáp, tư vấn thêm                                    │
│  • Thống nhất kế hoạch theo dõi                                │
│                                                                 │
│  [   KẾT THÚC CALL   ]                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Bước B3: Xác nhận sau call

- **MD US** – Approve Plan sau call

- **INPUT:** health_plan (Object, required), call_notes (String, optional)
- **OUTPUT:** plan_approved (Boolean), md_signature (Object: {e_sign, timestamp}, required)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────────┐
│                  ✅ XÁC NHẬN SAU VIDEO CALL                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📞 Call hoàn tất: [DD/MM/YYYY HH:MM] — [XX phút]              │
│                                                                 │
│  Ghi chú sau call:                                              │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ [                                                    ]│       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                 │
│  [   ✅ XÁC NHẬN PLAN VÀ GỬI CHO KH   ]                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Option 3: Update Plan (chỉnh sửa) — Bước B1c → B4 → B5

#### Bước B1c + B4: MD chỉnh sửa Plan

- **MD US** – Chỉnh sửa Plan:
  - Inline editing
  - Thêm/bớt khuyến nghị
  - Điều chỉnh follow-up schedule

- **INPUT:** health_plan (Object, required)
- **OUTPUT:** updated_plan (Object, required), edit_reason (String, required), edit_changelog (Array, required)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| edit_reason | BẮT BUỘC nhập khi chỉnh sửa Plan | "Vui lòng nhập lý do chỉnh sửa" |
| updated_plan | PHẢI có ít nhất: khuyến nghị + chu kỳ | "Plan không đầy đủ thông tin bắt buộc" |
| proposed_cycle | CHỈ chọn 6 tháng hoặc 12 tháng | "Chu kỳ không hợp lệ" |

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────────┐
│                  ✏️ CHỈNH SỬA PLAN ĐỊNH KỲ                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Bệnh nhân: [Họ tên KH]        Mã BN: [CVH-XXXX-XXXXXX]       │
│                                                                 │
│  ──────────────────────────────────────────────────────────     │
│  📋 PLAN ĐỊNH KỲ (ĐANG CHỈNH SỬA)                              │
│  ──────────────────────────────────────────────────────────     │
│                                                                 │
│  1. Tình trạng tổng quan: [_____ Editable ______]               │
│  2. Phát hiện chính:      [_____ Editable ______]               │
│  3. Khuyến nghị thuốc:    [_____ Editable ______]               │
│  4. Khuyến nghị lối sống: [_____ Editable ______]               │
│  5. Xét nghiệm bổ sung:  [_____ Editable ______]               │
│  6. Biện pháp phòng ngừa: [_____ Editable ______]               │
│  7. Chu kỳ tiếp theo:     [6 tháng ▼ / 12 tháng]               │
│                                                                 │
│  📝 Lý do chỉnh sửa (BẮT BUỘC):                                │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ [                                                    ]│       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                 │
│  [   💾 LƯU THAY ĐỔI   ]   [   ↩️ HOÀN TÁC   ]   [  HỦY  ] │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Bước B5: Xác nhận Plan đã sửa

- **MD US** – Approve Plan đã chỉnh (SLA: ≤ 10 phút sau call)
  - E-signature, locked sau khi ký

- **INPUT:** updated_plan (Object, required), edit_reason (String, required)
- **OUTPUT:** plan_approved (Boolean), md_signature (Object: {e_sign, timestamp}, required), plan_locked (Boolean)

⚠️ **Lỗi:**
- MD > 30 phút chưa approve → hệ thống nhắc (30' và 60')
- MD > 60 phút chưa approve → alert CS
- Lỗi kỹ thuật ký số → CS leo thang Tech Support

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────────┐
│                  ✅ XÁC NHẬN PLAN ĐÃ CHỈNH SỬA                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📋 Tóm tắt thay đổi:                                          │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ • [Mục thay đổi 1]: [Cũ] → [Mới]                    │       │
│  │ • [Mục thay đổi 2]: [Cũ] → [Mới]                    │       │
│  │ • Lý do: [Lý do MD nhập]                             │       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                 │
│  [   ✅ XÁC NHẬN VÀ GỬI CHO KH   ]   [  ← QUAY LẠI CHỈNH SỬA  ] │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## PHẦN CHUNG CUỐI (Bước 7-8)

### Bước 7. Hệ thống – Gửi Plan cho KH (SLA: ≤ 1 phút)

- Multi-channel: App + Email (HIPAA encrypted PDF) + Portal
- Nhánh A: ghi "Phê duyệt bởi: AI System"
- Nhánh B: ghi "Phê duyệt bởi: [Tên MD]"

- **INPUT:** health_plan (Object, required — bản AI gốc hoặc bản MD đã xác nhận/chỉnh sửa), patient_id (String, required)
- **OUTPUT:** delivery_status (Object: {email_sent, push_sent, portal_updated}, required), pdf_encrypted (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| patient_id_match | Gửi đúng Plan cho đúng KH (100% trùng khớp KH ID) | "Lỗi: Plan không khớp KH" |
| pdf_encryption | PDF PHẢI mã hóa theo chuẩn HIPAA | "Lỗi mã hóa PDF" |
| pdf_watermark | PDF PHẢI có watermark PHI | "Thiếu watermark PHI" |

⚠️ **Lỗi:**
- Delivery thất bại → retry tối đa 3 lần
- Sau 3 retry thất bại → CS liên hệ KH xác minh/cập nhật thông tin liên lạc, gửi lại qua kênh khác
- KH không mở sau 24h → reminder lần 2
- KH không mở sau 7 ngày → CS gọi xác nhận

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────────┐
│                  📋 PLAN SỨC KHỎE ĐỊNH KỲ                       │
│                  (Hiển thị trên App / Email / Portal)            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Xin chào [Họ tên KH],                                         │
│                                                                 │
│  Plan sức khỏe định kỳ của bạn đã sẵn sàng.                    │
│                                                                 │
│  ──────────────────────────────────────────────────────────     │
│  📊 TÌNH TRẠNG SỨC KHỎE HIỆN TẠI                               │
│  ──────────────────────────────────────────────────────────     │
│  Đánh giá tổng thể: [🟢 Ổn định / 🟡 Cần theo dõi / 🔴 Cần chú ý] │
│                                                                 │
│  📈 Cải thiện: [Danh sách chỉ số cải thiện]                    │
│  ➡️ Ổn định:   [Danh sách chỉ số ổn định]                      │
│  📉 Cần chú ý: [Danh sách chỉ số cần chú ý]                   │
│                                                                 │
│  ──────────────────────────────────────────────────────────     │
│  💡 KHUYẾN NGHỊ                                                 │
│  ──────────────────────────────────────────────────────────     │
│  💊 Thuốc: [Khuyến nghị thuốc]                                  │
│  🏃 Lối sống: [Khuyến nghị lối sống]                            │
│  🧪 Xét nghiệm: [Xét nghiệm bổ sung nếu cần]                  │
│  🛡️ Phòng ngừa: [Biện pháp phòng ngừa]                         │
│                                                                 │
│  ──────────────────────────────────────────────────────────     │
│  📅 LỊCH THEO DÕI                                               │
│  ──────────────────────────────────────────────────────────     │
│  Chu kỳ tiếp theo: [6 / 12] tháng                              │
│  Ngày hẹn dự kiến: [DD/MM/YYYY]                                │
│                                                                 │
│  ──────────────────────────────────────────────────────────     │
│  🚨 DẤU HIỆU CẢNH BÁO (Liên hệ ngay nếu gặp)                 │
│  ──────────────────────────────────────────────────────────     │
│  • [Dấu hiệu 1]                                                │
│  • [Dấu hiệu 2]                                                │
│  • [Dấu hiệu 3]                                                │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐      │
│  │ 📄 TẢI PDF   │  │ 📅 ĐẶT LỊCH  │  │ 📞 LIÊN HỆ HỖ TRỢ│      │
│  └──────────────┘  └──────────────┘  └──────────────────┘      │
│                                                                 │
│  🔒 Tài liệu PHI — Tuân thủ HIPAA                              │
│  Phê duyệt bởi: [Tên MD / AI System]                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Bước 8. Hệ thống – Cập nhật EHR + Lên lịch chu kỳ tiếp theo (SLA: ≤ 1 phút)

- Tự động tính ngày checkup tiếp theo dựa trên risk level
- Lên lịch reminder trước 30 ngày
- Đóng case

- **INPUT:** health_plan (Object, required), proposed_cycle (Number: 6|12, required), plan_date (Date, required)
- **OUTPUT:** ehr_updated (Boolean), next_checkup_date (Date), reminder_scheduled (Boolean), case_closed (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| next_checkup_date | Lịch hẹn PHẢI khớp chu kỳ AI đề xuất (6/12 tháng) | "Lịch hẹn không khớp chu kỳ" |
| ehr_update | Plan PHẢI được lưu đúng vào EHR section: Routine Checkup History | "Lỗi cập nhật EHR" |

⚠️ **Lỗi:** System failure → CS báo kỹ thuật

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────────┐
│                  ⚙️ CẬP NHẬT HỆ THỐNG (TỰ ĐỘNG)                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ Cập nhật EHR:                                               │
│     • Plan định kỳ đã lưu vào hồ sơ                            │
│     • Kết quả phân tích đã cập nhật                             │
│     • Khuyến nghị đã ghi nhận                                   │
│                                                                 │
│  ✅ Thiết lập lịch kỳ tiếp theo:                                │
│     • Chu kỳ: [6 / 12] tháng                                   │
│     • Ngày hẹn dự kiến: [DD/MM/YYYY]                           │
│     • Reminder sẽ được gửi trước 30 ngày                       │
│                                                                 │
│  ✅ Case Closed                                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## XỬ LÝ TRƯỜNG HỢP ĐẶC BIỆT

| Trường hợp | Xử lý |
|------------|--------|
| Không có subscription | Chặn, gợi ý đăng ký |
| EHR không đầy đủ | AI ghi nhận hạn chế, recommend Branch B |
| Emergency detected | Alert MD ngay, escalate Service 1 (Access_to_Care_247) |
| MD timeout | Hệ thống nhắc ở 30' và 60', CS escalate. Nếu 48h không phản hồi → Escalate cho MD khác |
| KH không phản hồi | Nhắc nhở tự động, CS follow-up (gọi lại 4h, 24h) |
| Cần specialist referral | Chuyển Service 2 (Specialist_Referral) |
| KH không mở Plan trong 7 ngày | Hệ thống gửi nhắc nhở qua email/push notification |
| KH hủy gói dịch vụ | Xóa lịch hẹn kỳ tiếp + Không gửi reminder |

---

## BẢNG THÔNG BÁO TỰ ĐỘNG

| Sự kiện | Kênh | Người nhận |
|---------|------|------------|
| Plan kết quả sẵn sàng | Email (PDF) + Push | KH |
| MD cần review (Branch B) | Alert nội bộ (Dashboard + Push + Email) | MD US |
| Nhắc nhở KH (Branch B scheduling) | Email + Push | KH |
| Video Call confirmation (Branch B) | Email | KH |
| Follow-up reminder | Push | KH |
| Lịch checkup tiếp theo | Push + Email | KH |

### Template chi tiết

📧 **Email Plan kết quả sẵn sàng:**
- Subject: "CVH: Plan Sức khỏe Định kỳ của bạn đã sẵn sàng"
- Body:
```
═══════════════════════════════════════════════════════════════
                         CVH HEALTH
           PLAN SỨC KHỎE ĐỊNH KỲ CỦA BẠN ĐÃ SẴN SÀNG
═══════════════════════════════════════════════════════════════

Xin chào {ten_khach_hang},

Plan sức khỏe định kỳ của bạn đã được hoàn tất và sẵn sàng
xem trên app CVH Health.

───────────────────────────────────────────────────────────────
📋 THÔNG TIN
───────────────────────────────────────────────────────────────

• Ngày lập Plan: {ngay_lap_plan}
• Dựa trên cuộc khám: {ngay_kham}
• Phê duyệt bởi: {ten_md_hoac_ai}

───────────────────────────────────────────────────────────────
📊 TÌNH TRẠNG: {trang_thai_tong_the}
───────────────────────────────────────────────────────────────

{mo_ta_trang_thai}

───────────────────────────────────────────────────────────────
💡 KHUYẾN NGHỊ CHÍNH
───────────────────────────────────────────────────────────────

1. {khuyen_nghi_1}
2. {khuyen_nghi_2}
3. {khuyen_nghi_3}

───────────────────────────────────────────────────────────────
📅 LẦN KHÁM TIẾP THEO
───────────────────────────────────────────────────────────────

• Chu kỳ: {chu_ky} tháng
• Dự kiến: {ngay_hen_tiep}

───────────────────────────────────────────────────────────────

┌────────────────────────────────────────────────────────────┐
│ [XEM PLAN ĐẦY ĐỦ]    [TẢI PDF]    [ĐẶT LỊCH KHÁM TIẾP]  │
└────────────────────────────────────────────────────────────┘

📞 Cần hỗ trợ?
• Hotline: 1800-XXX-XXX (miễn phí)
• Email: support@cvhealth.com
• Chat trong app CVH Health

⚠️ Email này chứa thông tin y tế bảo mật (PHI).
   Tuân thủ HIPAA. Nếu không phải người nhận,
   vui lòng xóa và thông báo cho chúng tôi.

═══════════════════════════════════════════════════════════════
                        CVH Health
               Chăm sóc sức khỏe từ xa
           © 2026 CVH Health. All rights reserved.
═══════════════════════════════════════════════════════════════
```

📱 **Push Notification (App):**
- Title: "Plan Sức khỏe Định kỳ đã sẵn sàng"
- Body: "Plan của bạn đã được lập. Nhấn để xem chi tiết và khuyến nghị."
- Action: Mở màn hình Plan trong App

🔔 **Thông báo cho MD (Nhánh B):**
- Title: "⚠️ Plan định kỳ cần review"
- Body: "Plan của BN {ten_KH} ({ma_BN}) cần review. AI phát hiện chỉ số bất thường."
- Kênh: MD Dashboard + Push Notification + Email

📱 **Nhắc nhở KH chưa mở Plan:**
- Title: "Nhắc nhở: Plan Sức khỏe Định kỳ chưa được xem"
- Body: "Plan của bạn đã sẵn sàng từ {ngay}. Nhấn để xem khuyến nghị sức khỏe."
- Thời điểm: Sau 7 ngày nếu KH chưa mở Plan

📧 **Email Video Call (Option 2 — Nhánh B):**
- Subject: "CVH: Lịch hẹn Video Call giải thích Plan Sức khỏe Định kỳ"
- Nội dung: Thông tin lịch call (ngày, giờ, thời lượng), hướng dẫn tham gia, link call

---

## BẢNG GIÁM SÁT CS-MD ĐẦY ĐỦ

- **Hệ thống auto ~95%:** Trigger detection (60s polling), data collection, EHR retrieval, AI processing, AI analysis, AI Plan, AI routing, gửi Plan, cập nhật EHR, lên lịch chu kỳ
- **CS can thiệp:** Branch B — gọi KH trong 2h, đặt lịch 48h, no-show, lỗi kỹ thuật video, MD trễ > 5', delivery fail 3 lần, system crash
- **MD can thiệp:** AI flag dữ liệu có vấn đề, critical alerts, Branch B review/approve (3 options), periodic audit

### 12 trạm giám sát

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| EHR Change Detection | Phát hiện ≤ 3 phút, accuracy 100%. Polling mỗi 60s | 🤖 100%: Truy vấn mỗi 60s, phân loại mức độ thay đổi (Thông thường/Đáng chú ý/Nghiêm trọng). Delay > 5 phút → tự restart → alert Dev Team. Phát hiện thay đổi nghiêm trọng → push alert MD | ⚠️ Background job sập > 10 phút và Dev Team không phản hồi → CS ghi nhận sự cố, thông báo nội bộ | ❌ Không can thiệp. Audit định kỳ hàng tuần đánh giá logic kích hoạt |
| Thu thập dữ liệu | ≤ 30s, thu thập 100% dữ liệu thay đổi, không duplicate | 🤖 100%: Truy vấn DB lấy records thay đổi, kiểm tra tính toàn vẹn (không null, không duplicate). DB lỗi → retry 3 lần (cách 5s). Retry thất bại → log + alert Dev Team | ⚠️ DB sập > 10 phút, Dev không xử lý → CS cảnh báo khẩn cấp Tech Support & Manager | ❌ |
| Truy xuất EHR | ≤ 20s, truy xuất đúng & đầy đủ 100% EHR | 🤖 100%: Truy vấn toàn bộ EHR theo Patient ID, kiểm tra đầy đủ các mục bắt buộc, đối chiếu EHR version. Không đầy đủ → retry 3 lần → alert Dev Team | ⚠️ Không truy xuất được EHR sau 3 retry, Dev không phản hồi → CS xác minh Patient ID, leo thang Tech Support | ❌ (chỉ tham gia nếu EHR có dữ liệu y tế mâu thuẫn cần chẩn đoán) |
| AI xử lý tổng hợp | ≤ 20s, chuẩn hóa đúng ICD-10/LOINC, không lỗi mapping | 🤖 100%: Chuẩn hóa đơn vị, đồng bộ thuật ngữ, kiểm tra outlier, đối chiếu phạm vi tham chiếu theo tuổi/giới tính. Lỗi mapping → thử alternative mapping. Outlier → flag MD, không dừng | ⚠️ AI sập > 10 phút → CS ghi nhận, cảnh báo Dev Team & Manager | ✅ Nhận flag dữ liệu bất thường/lỗi mapping → review và xác nhận/điều chỉnh. Audit định kỳ |
| AI phân tích đánh giá | ≤ 30s, accuracy ≥ 95% vs MD audit | 🤖 100%: So sánh lịch sử, phân loại nguy cơ (thấp/trung bình/cao/nguy hiểm), đánh dấu vượt ngưỡng. Không hoàn thành 3 phút → retry + log. Critical → alert MD. Confidence < 85% → flag MD | ⚠️ AI sập > 10 phút → CS leo thang Dev Team | ✅ Can thiệp NGAY khi nhận alert CRITICAL. Review báo cáo phân tích nếu rủi ro cao. Audit định kỳ ≥ 95% agreement |
| AI lập Plan | ≤ 30s, tuân thủ 100% template CVH, song ngữ VN-EN | 🤖 100%: Generate theo template, kiểm tra hoàn chỉnh sections, kiểm tra ngữ pháp/thuật ngữ. Thiếu section → retry. Không thể tạo → alert Dev + MD | ⚠️ AI không thể tạo Plan → CS leo thang Dev Team | ❌ (Luồng A). ✅ Can thiệp ở trạm MD review (Luồng B). Audit định kỳ |
| AI xác định luồng | ≤ 15s, accuracy ≥ 98% | 🤖 100%: Cây quyết định (rủi ro, bất thường, xu hướng). A = bình thường. B = bất thường. Không rõ → mặc định B. Log lý do phân luồng. Case B → notify CS | ⚠️ AI không phân loại được → CS mặc định chuyển B. Cảnh báo Dev Team | ❌ Audit định kỳ, review borderline cases |
| Thông báo KH + đặt lịch (B) | CS liên hệ ≤ 2h, lịch đặt ≤ 48h | 🤖 95%: App push + Email tự động. Hiển thị slot trống 48h. KH dừng > 60s → gửi link hướng dẫn. 24h chưa đặt → reminder. 24h vẫn chưa → alert CS | ⚠️ CS gọi KH trong 2h nếu chưa tự đặt. Hỗ trợ đặt lịch qua điện thoại. Follow-up 4h, 24h. Báo MD nếu KH từ chối/không liên lạc được sau 48h | ❌ Nhận thông báo từ CS về KH từ chối/không liên lạc được |
| MD US Video Call (B) | Call đúng giờ ±5 phút, 20-30 phút | 🤖 95%: Bật video platform đúng giờ, load Plan + EHR lên giao diện MD. KH không join 10 phút → push + alert CS. Gửi khảo sát satisfaction sau call | ⚠️ KH không join → gọi nhắc. No-show → reschedule 2h. Lỗi kết nối → hướng dẫn qua điện thoại. Link sập → tạo link dự phòng + SMS | ✅ MD US tham gia call đúng giờ, tư vấn theo Plan AI + Báo cáo, ghi chép EMR, chỉnh sửa Plan nếu cần |
| MD US chỉnh sửa + phê duyệt (B) | ≤ 10 phút sau call, ký số hợp lệ | 🤖 95%: Mở Plan cho MD review sau call, enable chế độ chỉnh sửa + track changes. 30 phút chưa duyệt → reminder MD. 60 phút → reminder + alert CS. Log tất cả thay đổi + ký số | ⚠️ MD > 60 phút do lỗi ký số → CS ping MD nội bộ, leo thang Tech Support | ✅ MD US review + chỉnh sửa + phê duyệt (ký số) trong 10 phút |
| Gửi Plan | ≤ 1 phút, delivery rate 100%, gửi đúng KH | 🤖 100%: Mã hóa PDF HIPAA, gửi đa kênh (App push + Email). Kiểm tra KH ID, PDF hợp lệ, email valid. Thất bại → retry 3 lần. KH không mở 24h → reminder | ⚠️ Gửi thất bại 3 retry (bounce, số sai) → CS liên hệ xác minh, cập nhật info, gửi lại kênh khác. KH không mở 24h → CS gọi xác nhận | ❌ |
| Cập nhật EHR + Lịch theo dõi | ≤ 1 phút, lịch đúng logic risk level | 🤖 100%: Lưu Plan vào EHR (section: Routine Checkup History), tính ngày checkup tiếp (risk level + guidelines), kiểm tra EHR cập nhật + lịch tạo đúng, đóng case. Thất bại → alert Tech Support | ⚠️ Không cập nhật EHR > 10 phút → CS cảnh báo Tech Support, ghi nhận sự cố | ❌ |

---

## DATA SPECIFICATION

### Bảng tổng hợp Input/Output cho các node chính

| Node ID | Tên Node | Input Fields | Output Fields | Validation Rules |
|---------|----------|-------------|---------------|-----------------|
| T-0 | Trigger | Kết quả khám mới, KH ID, Gói dịch vụ | Trigger activated | KH PHẢI có gói dịch vụ bao gồm Service 7 (Routine_Checkup) |
| T-1 | Thu thập dữ liệu mới | Trigger, KH ID | Dữ liệu khám mới (CLS, sinh hiệu, ghi chú) | Dữ liệu PHẢI được cập nhật vào EHR trước khi thu thập |
| T-2 | Truy xuất EHR | KH ID | EHR đầy đủ (lịch sử khám, chẩn đoán, đơn thuốc, sinh hiệu) | PHẢI có ít nhất 1 lần khám trước đó trong EHR |
| T-3 | AI xử lý tổng hợp | Dữ liệu khám mới + EHR | Dữ liệu đã xử lý (health profile, so sánh, xu hướng) | Dữ liệu PHẢI đầy đủ, không có trường rỗng bắt buộc |
| T-4 | AI phân tích đánh giá | Dữ liệu đã xử lý | Báo cáo phân tích (bất thường, risk factors, xu hướng) | Báo cáo PHẢI xác định rõ: bất thường / không bất thường |
| T-5 | AI lập Plan | Báo cáo phân tích | Plan định kỳ (khuyến nghị, chu kỳ, lịch theo dõi) | Chu kỳ CHỈ 6 hoặc 12 tháng. Plan PHẢI có khuyến nghị + chu kỳ |
| T-6 | AI xác định luồng | Báo cáo phân tích | Quyết định: Nhánh A hoặc Nhánh B | PHẢI là A hoặc B, không có giá trị khác |
| B1a | MD Review + Xác nhận | Plan AI | Plan đã xác nhận | MD PHẢI xác nhận hoặc chuyển sang Option khác |
| B1b | Tạo Video Call | Plan AI | Lịch call (ngày, giờ, thời lượng) | Ngày call PHẢI trong tương lai, thời lượng 20-30 phút |
| B2 | Video Call | Plan AI + Báo cáo phân tích | Hoàn tất giải thích | Call PHẢI được ghi nhận (consent + HIPAA) |
| B3 | Xác nhận sau call | Plan AI | Plan đã xác nhận | MD PHẢI xác nhận sau call |
| B1c | Cập nhật Plan | Plan AI | Chế độ chỉnh sửa | — |
| B4 | MD chỉnh sửa Plan | Plan AI | Plan đã chỉnh sửa | Lý do chỉnh sửa BẮT BUỘC. Plan PHẢI có khuyến nghị + chu kỳ |
| B5 | Xác nhận Plan đã sửa | Plan đã chỉnh sửa | Plan đã xác nhận | MD PHẢI review toàn bộ thay đổi trước khi xác nhận |
| T-7 | Gửi Plan cho KH | Plan (A: AI gốc / B: MD xác nhận) | KH nhận Plan (Email PDF + App + Portal) | Email PHẢI mã hóa. PDF PHẢI có watermark PHI |
| T-8 | Cập nhật EHR + Lịch kỳ tiếp | Plan định kỳ | EHR cập nhật + Lịch hẹn kỳ tiếp | Lịch hẹn PHẢI khớp chu kỳ AI đề xuất (6/12 tháng) |

### Dữ liệu Plan định kỳ (Output chính)

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| patient_id | String | Yes | CVH-XXXX-XXXXXX | Mã bệnh nhân |
| patient_name | String | Yes | — | Họ tên KH |
| patient_dob | Date | Yes | DD/MM/YYYY | Ngày sinh |
| patient_age | Number | Yes | — | Tuổi |
| overall_status | String | Yes | "Ổn định" / "Cần theo dõi" / "Cần chú ý" | Tình trạng sức khỏe tổng quan |
| historical_comparison | Object | Yes | {improved: [], stable: [], worsened: []} | So sánh với lần khám trước |
| key_findings | Object | Yes | {positive: [], monitoring: [], changes: []} | Phát hiện chính |
| risk_factors | Object | Yes | {lifestyle: [], medical: [], lab: []} | Yếu tố nguy cơ |
| medication_recommendations | Array | Yes | [{drug, action: "continue"/"adjust"/"new", details}] | Khuyến nghị thuốc |
| lifestyle_recommendations | Object | Yes | {diet: "", exercise: "", sleep: "", stress: ""} | Khuyến nghị lối sống |
| additional_tests | Array | Optional | [{test_name, timeline, reason}] | Xét nghiệm bổ sung |
| preventive_measures | Array | Optional | [{measure, type: "vaccination"/"screening"/"risk_reduction"}] | Biện pháp phòng ngừa |
| next_cycle | Number | Yes | 6 hoặc 12 (tháng) | Chu kỳ tiếp theo |
| next_appointment_date | Date | Yes | DD/MM/YYYY | Ngày hẹn kỳ tiếp |
| warning_signs | Array | Yes | [String] | Dấu hiệu cảnh báo cần liên hệ ngay |
| approved_by | String | Yes | "AI System" (Nhánh A) / Tên MD (Nhánh B) | Người/hệ thống phê duyệt |
| plan_date | Date | Yes | DD/MM/YYYY | Ngày lập Plan |
| checkup_date | Date | Yes | DD/MM/YYYY | Ngày cuộc khám |
| branch | String | Yes | "A" / "B" | Nhánh xử lý |
| md_signature | Object | Nhánh B | {e_sign, timestamp, md_name} | Ký số MD |
| edit_reason | String | Option 3 | — | Lý do chỉnh sửa (nếu MD chỉnh) |
| edit_changelog | Array | Option 3 | [{field, old_value, new_value}] | Log thay đổi |
| pdf_encrypted | Boolean | Yes | true | PDF mã hóa HIPAA |
| language | String | Yes | "vi-en" | Song ngữ Việt-Anh |

---

## NGUỒN TÀI LIỆU GỐC

- Luồng nghiệp vụ: `docs/02-product/ba-specs/service-7-Routine_Checkup/base/Routine_Checkup_Touchpoint_Flow_v1.md`
- Checklist CS & MD: `docs/02-product/ba-specs/service-7-Routine_Checkup/checklist-cs-md/checklist-cs-md.md`
- Bản tóm tắt: `docs/02-product/tomtat-chitiet/tomtat/service-7-Routine_Checkup-tomtat.md`
