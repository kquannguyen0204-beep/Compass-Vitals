# CHI TIẾT LUỒNG NGHIỆP VỤ – SERVICE 3 (Schedule_Video_Visit): ĐẶT LỊCH KHÁM VIDEO (SCHEDULE VIDEO VISIT)

## Header

- **Service:** Schedule Video Visit (Đặt lịch khám Video)
- **Version:** 1.0.0
- **Input:** Khách hàng có gói Care Plus/Premium cần tư vấn video
- **Output:** Chẩn đoán + Pathway điều trị (Kê đơn / Xét nghiệm / Theo dõi / Cấp cứu)
- **Thời lượng:** 20-30 phút/buổi video + 24-72h theo dõi
- **Điều kiện sử dụng:** Care Plus (5 lượt/năm), Care Premium (unlimited), Care Connect (không có quyền)
- **Compliance:** HIPAA + State Licensing + EPCS (Controlled Substances) + CLIA

---

## Nguyên tắc cốt lõi

- **Scheduled Care Model:** Đặt lịch trước 24h
- **Direct Order Creation:** US MD tạo + approve đồng thời (không qua VN MD)
- **Multi-pathway Treatment:** Emergency / Prescription / Lab / Monitoring
- **Quota-based Access:** Care Plus 5/năm, Premium unlimited, Connect không có
- **Emergency Priority:** < 15 phút
- **Cancellation Deadline:** 2h trước lịch hẹn (miễn phí), < 2h qua hotline (trừ quota)
- **Cross-service:** Lab/Imaging → Service 10 (Lab_Imaging_Management), Specialist Referral → Service 2 (Specialist_Referral)

---

## Danh sách Flow

| Flow ID | Tên luồng | Input | Output | Số trạm |
|---------|-----------|-------|--------|---------|
| FLOW-01 | Hủy lịch hẹn | KH muốn hủy lịch video visit | Lịch đã hủy, quota/refund xử lý xong | 4 |
| FLOW-02 | Cấp cứu trong Video Visit | BS phát hiện tình huống khẩn cấp khi đang khám | Handoff thành công đến ER/911 | 11 |
| FLOW-03 | Kê đơn sau Video Visit | BS kê đơn thuốc sau khi khám video | Đơn thuốc đã được tạo trong hệ thống CVH | 13 |
| FLOW-04 | Xét nghiệm sau Video Visit | BS chỉ định xét nghiệm/chụp hình sau khám video | Chỉ định XN đã được tạo trong hệ thống CVH | 13 |
| FLOW-05 | Theo dõi sau Video Visit | BS chỉ định theo dõi (từ V4 / sau FLOW-03 / sau FLOW-04) | Tình trạng KH ổn định | 17 |

### Chi tiết phân nhánh

| Flow | Phân nhánh | Mô tả |
|------|------------|-------|
| FLOW-01 | Nhánh A: Hủy trước 2h | Hủy miễn phí qua App, KHÔNG tính quota |
| FLOW-01 | Nhánh B: Hủy trong 2h | PHẢI gọi hotline, CÓ THỂ tính quota (Care Plus) |
| FLOW-02 | Không phân nhánh | Phát hiện → Emergency Order → ER Protocol → Handoff |
| FLOW-03 | Nhánh A: Đóng case | Hoàn thành kê đơn, không cần theo dõi |
| FLOW-03 | Nhánh B: Chuyển Monitoring | Cần theo dõi → Chuyển FLOW-05 (Trạm 12-17) |
| FLOW-04 | Nhánh A: Đóng case | Hoàn thành chỉ định XN, không cần theo dõi |
| FLOW-04 | Nhánh B: Chuyển Monitoring | Cần theo dõi → Chuyển FLOW-05 (Trạm 12-17) |
| FLOW-05 | Điểm nhập #1: Từ V4 | Thực hiện ĐẦY ĐỦ 17 trạm |
| FLOW-05 | Điểm nhập #2: Từ FLOW-03 | BỎ QUA Trạm 1-11, chỉ Trạm 12-17 |
| FLOW-05 | Điểm nhập #3: Từ FLOW-04 | BỎ QUA Trạm 1-11, chỉ Trạm 12-17 |
| FLOW-05 | Final Review: Ổn định | Đóng case |
| FLOW-05 | Final Review: Xấu đi | Escalate (Emergency / Thuốc mới / XN mới / Video Visit mới) |

---

## FLOW-03: KÊ ĐƠN SAU VIDEO VISIT (LUỒNG CHÍNH)

### Sơ đồ

```
┌─ BOOKING ──────────────────────────────────────────────────────────────────┐
│ Booking → Confirm → Reminders                                              │
└───────────────────────┬────────────────────────────────────────────────────┘
                        ↓
┌─ PREP ────────────────────────────────────────────────────────────────────┐
│ Collect Docs → AI Analysis → VN MD Review                                  │
└───────────────────────┬────────────────────────────────────────────────────┘
                        ↓
┌─ VIDEO ───────────────────────────────────────────────────────────────────┐
│ Join Call → Clinical Interview → Diagnosis → Treatment Decision (Rx)       │
└───────────────────────┬────────────────────────────────────────────────────┘
                        ↓
┌─ PRESCRIPTION ────────────────────────────────────────────────────────────┐
│ Rx Order Creation ⭐ → EMR Update → Completion ─┬─ [Đóng case]            │
│                                                   └─ [Theo dõi] → FLOW-05  │
└────────────────────────────────────────────────────────────────────────────┘
```

### Bước 1: Booking — Khách hàng đặt lịch video visit

- **Ai làm:** Khách hàng
- **Mô tả:** KH đặt lịch video visit qua App/Web: chọn ngày giờ (trước 24h), mô tả triệu chứng, chọn bác sĩ (nếu có)
  - Hệ thống kiểm tra quota: Care Plus ≤ 5/năm, Premium unlimited
  - ⛔ Hết quota → gợi ý mua thêm (Function Registration - FLOW-07) | Còn quota → tiếp tục

**INPUT/OUTPUT:**
- INPUT: appointment_date (Date, required), appointment_time (String, required), chief_complaint (String, required, max 500 ký tự), symptoms (String, required, max 1000 ký tự), symptom_onset (String, optional), pain_level (Number: 1-10, optional), preferred_md_id (String, optional), service_plan (String: CarePlus/CarePremium, required)
- OUTPUT: appointment_id (String, auto-generated), status (String: "Pending"), quota_remaining (Number, Care Plus only)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Ngày giờ | PHẢI ≥ 24h từ thời điểm hiện tại | "Vui lòng chọn thời gian ít nhất 24h sau" |
| Lý do khám chính | BẮT BUỘC, tối đa 500 ký tự | "Vui lòng nhập lý do khám" |
| Triệu chứng | BẮT BUỘC, tối đa 1000 ký tự | "Vui lòng mô tả triệu chứng" |
| Gói dịch vụ | PHẢI là Care Plus hoặc Care Premium | "Dịch vụ này chỉ dành cho gói Care Plus/Premium" |
| Quota (Care Plus) | PHẢI còn quota (< 5 lần/năm) | "Bạn đã hết lượt khám trong năm. Vui lòng nâng cấp gói" |
| Slot | Không trùng slot đã đặt | "Khung giờ này đã được đặt. Vui lòng chọn khung giờ khác" |

**UI States:**
- Default: Form trống, nút "Xác nhận đặt lịch" disabled
- Typing: Validate real-time, border xanh khi hợp lệ
- Error: Border đỏ + message lỗi dưới field
- Loading: Spinner khi kiểm tra quota/slot
- Success: Popup "Đặt lịch thành công! Mã: [ID]"
- Blocked (hết quota): Popup "Quota đã hết. Nâng cấp gói?"

**Wireframe ASCII:**

```
┌─────────────────────────────────────┐
│      ĐẶT LỊCH KHÁM VIDEO           │
├─────────────────────────────────────┤
│                                     │
│  📅 Chọn ngày:                      │
│  [  Lịch hiển thị slot trống  ]    │
│  (Slot xanh = Available)            │
│  (Slot xám = Đã đặt)               │
│                                     │
│  🕐 Chọn giờ: [▼ Chọn slot   ]    │
│                                     │
│  ⚠️ Phải đặt trước ít nhất 24h     │
│                                     │
│  📝 Lý do khám chính *:            │
│  [__________________________ ]      │
│  (Tối đa 500 ký tự)                │
│                                     │
│  📝 Triệu chứng *:                 │
│  [__________________________ ]      │
│  (Tối đa 1000 ký tự)               │
│                                     │
│  📅 Khi nào bắt đầu:               │
│  [__________________________ ]      │
│                                     │
│  😣 Mức độ đau (1-10):             │
│  [▼ 1-10                     ]     │
│                                     │
│  📊 Quota còn lại: 3/5 lượt/năm   │
│                                     │
│  [  XÁC NHẬN ĐẶT LỊCH  ]         │
│                                     │
└─────────────────────────────────────┘
```

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Còn quota + slot trống + dữ liệu hợp lệ | Tạo Appointment ID → Bước 2 |
| Hết quota (Care Plus) | Gợi ý nâng cấp gói (Function Registration - FLOW-07) |
| Slot trùng | Auto suggest slot khác gần nhất |
| Gói Care Connect | Thông báo không có quyền |

---

### Bước 2: Confirm — Hệ thống gửi xác nhận

- **Ai làm:** Hệ thống
- **Mô tả:** Gửi email/SMS/Push xác nhận lịch hẹn cho KH

**INPUT/OUTPUT:**
- INPUT: appointment_id (String, required)
- OUTPUT: confirmation_sent (Boolean), notification_channels (Array: [Email, SMS, Push])

⚠️ Gửi trong 1 phút sau khi đặt lịch thành công

---

### Bước 3: Reminders — Hệ thống nhắc nhở

- **Ai làm:** Hệ thống
- **Mô tả:** Chuỗi reminder: 3h trước (có Confirm/Cancel) + 30 phút trước (chỉ nhắc)

**INPUT/OUTPUT:**
- INPUT: appointment_id (String, required), appointment_time (DateTime, required), confirm_status (String, required)
- OUTPUT: reminder_sent (Boolean), kh_response (String: Confirmed/Canceled/NoResponse)

**UI States — Reminder 3h:**
- Default: Hiển thị thông tin lịch hẹn + countdown + nút Xác nhận/Hủy
- Confirmed: "✅ Đã xác nhận!" + SMS xác nhận trong 30 giây
- Canceled: Chuyển FLOW-01

**Wireframe ASCII — Reminder 3h:**

```
┌─────────────────────────────────────┐
│   ⏰ NHẮC NHỞ CUỘC HẸN             │
├─────────────────────────────────────┤
│                                     │
│  Cuộc khám sắp bắt đầu trong 3 giờ!│
│                                     │
│  📅 Ngày: 10/02/2026               │
│  🕐 Giờ: 14:00 (EST)              │
│  👨‍⚕️ BS: Dr. Smith                  │
│                                     │
│  ⏳ Còn lại: 02:59:45              │
│                                     │
│  [ ✅ XÁC NHẬN THAM GIA ]          │
│  [ ❌ HỦY LỊCH HẸN      ]          │
│                                     │
│  ℹ️ Bạn có thể hủy miễn phí        │
│  đến 2h trước cuộc hẹn             │
│                                     │
└─────────────────────────────────────┘
```

**Wireframe ASCII — Reminder 30 phút:**

```
┌─────────────────────────────────────┐
│   🎥 CHUẨN BỊ CUỘC HẸN            │
├─────────────────────────────────────┤
│                                     │
│  Cuộc khám sắp bắt đầu trong       │
│  30 phút!                           │
│                                     │
│  ⏳ Còn lại: 29:45                  │
│                                     │
│  📋 Checklist chuẩn bị:            │
│  ☐ Kết nối internet ổn định        │
│  ☐ Camera hoạt động                │
│  ☐ Microphone hoạt động            │
│  ☐ Nơi riêng tư, yên tĩnh         │
│  ☐ Ánh sáng đủ                     │
│  ☐ Danh sách thuốc đang dùng      │
│  ☐ Câu hỏi muốn hỏi bác sĩ       │
│                                     │
│  [ ✅ TÔI ĐÃ SẴN SÀNG ]           │
│                                     │
│  ⚠️ KHÔNG có nút "Hủy"            │
│  Cần hủy? Gọi: 1900-xxxx          │
│  (Hủy <2h sẽ mất 1 lượt khám)     │
│                                     │
└─────────────────────────────────────┘
```

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| KH click "Xác nhận" | Status = Confirmed → gửi SMS xác nhận trong 30 giây |
| KH click "Hủy" (> 2h) | Chuyển FLOW-01 Nhánh A |
| KH không phản hồi T-2h30 | Gửi follow-up SMS |
| T-30 phút | Gửi reminder chỉ nhắc, KHÔNG có nút Hủy |

---

### Bước 4: Collect Docs — Thu thập tài liệu

- **Ai làm:** Hệ thống
- **Mô tả:** Thu thập hồ sơ KH tự động từ EMR: tiền sử bệnh, thuốc đang dùng, dị ứng, kết quả XN

**INPUT/OUTPUT:**
- INPUT: patient_id (String, required), emr_data (Object, required)
- OUTPUT: documents_package (Object: {medical_history, current_medications, allergies, lab_results, chief_complaint})

⚠️ Thu thập đầy đủ trong 2-3 phút

---

### Bước 5: AI Analysis — Phân tích AI

- **Ai làm:** AI Agent
- **Mô tả:** Phân tích hồ sơ, tạo summary 2-3 trang, phát hiện tương tác thuốc, gợi ý câu hỏi cho BS

**INPUT/OUTPUT:**
- INPUT: documents_package (Object, required), chief_complaint (String, required)
- OUTPUT: ai_summary (Object: {summary_text, drug_interactions, suggested_questions, differential_diagnosis}), analysis_confidence (Number: 0-100)

⚠️ Hoàn thành trong 3-5 phút

---

### Bước 6: VN MD Review — BS Việt Nam review

- **Ai làm:** BS Việt Nam
- **Mô tả:** Review AI summary, validate differential diagnosis, flag concerns cho US MD

**INPUT/OUTPUT:**
- INPUT: ai_summary (Object, required), full_records (Object, required)
- OUTPUT: prep_complete (Object: {clinical_notes, flagged_concerns, validated_diagnosis})

⚠️ Review 2-4h trước cuộc hẹn
⚠️ VN MD không review < 5 phút trước giờ hẹn → CS alert

---

### Bước 7: Join Call — Vào video call

- **Ai làm:** Khách hàng / BS Mỹ
- **Mô tả:** KH và US MD vào video call đúng giờ (waiting room 5 phút trước)

**INPUT/OUTPUT:**
- INPUT: video_link (String, required), patient_id (String, required), md_id (String, required)
- OUTPUT: call_started (Boolean), technical_check (Object: {camera, mic, internet})

**UI States:**
- Waiting: Hiển thị trạng thái thiết bị, "Bác sĩ sẽ tham gia trong giây lát..."
- Connected: Video call bắt đầu
- Technical Error: Thông báo lỗi kết nối + auto reconnect

**Wireframe ASCII:**

```
┌─────────────────────────────────────┐
│   🎥 PHÒNG CHỜ VIDEO               │
├─────────────────────────────────────┤
│                                     │
│  ✅ Camera: Đang hoạt động          │
│  ✅ Microphone: Đang hoạt động      │
│  ✅ Internet: Ổn định               │
│                                     │
│  Bác sĩ sẽ tham gia trong          │
│  giây lát...                        │
│                                     │
│  ⏳ Đang chờ...                     │
│                                     │
│  [📷 Tắt camera]  [🎤 Tắt mic]    │
│                                     │
└─────────────────────────────────────┘
```

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| KH + BS vào đúng giờ | Bắt đầu video call |
| KH no-show sau 5 phút | Auto-call KH |
| KH no-show sau 10 phút | Đánh dấu no-show → No-show Policy |
| Lỗi kết nối | Auto reconnect → fallback gọi điện thoại |

---

### Bước 8: Clinical Interview — Phỏng vấn lâm sàng

- **Ai làm:** BS Mỹ
- **Mô tả:** Phỏng vấn lâm sàng: HPI, review of systems, khám hạn chế qua video

**INPUT/OUTPUT:**
- INPUT: call_started (Boolean, required), prep_data (Object, required)
- OUTPUT: clinical_data (Object: {hpi, review_of_systems, exam_findings, vital_signs_reported})

⚠️ Duration 15-25 phút

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Phát hiện triệu chứng cấp cứu | Chuyển FLOW-02 (Emergency) |
| Bình thường | Tiếp tục Bước 9 |

---

### Bước 9: Diagnosis — Chẩn đoán

- **Ai làm:** BS Mỹ
- **Mô tả:** Chẩn đoán, xác định ICD-10 codes

**INPUT/OUTPUT:**
- INPUT: clinical_data (Object, required)
- OUTPUT: diagnosis (Object: {icd10_codes (Array, required), assessment (String, required), differential_diagnosis (Array, optional)})

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| ICD-10 codes | BẮT BUỘC, ít nhất 1 code | "Vui lòng nhập mã ICD-10" |

---

### Bước 10: Treatment Decision — Quyết định điều trị

- **Ai làm:** BS Mỹ
- **Mô tả:** Quyết định kê đơn thuốc, giải thích treatment plan cho KH

**INPUT/OUTPUT:**
- INPUT: diagnosis (Object, required)
- OUTPUT: treatment_pathway (String: Rx/Lab/Monitor/Emergency/Combined, required)

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Kê đơn thuốc | FLOW-03 Bước 11 (tiếp) |
| Xét nghiệm | Chuyển FLOW-04 |
| Theo dõi | Chuyển FLOW-05 |
| Kết hợp nhiều pathway | Thực hiện tuần tự |

---

### Bước 11: Rx Order Creation ⭐ — Tạo đơn thuốc

- **Ai làm:** BS Mỹ / Hệ thống
- **Mô tả:** US MD tạo đơn thuốc với status='approved' (tạo + phê duyệt đồng thời)

**INPUT/OUTPUT:**
- INPUT: rx_decision (Object, required), drug_name (String, required), dosage (String, required), frequency (String, required), duration_days (Number, required, > 0), quantity (Number, required, > 0), refills (Number, optional, default 0), sig (String, optional), clinical_notes (String, optional)
- OUTPUT: prescription_order (Object: {order_id, status: 'approved', approved_by, created_at}), drug_interaction_check (Boolean), allergy_check (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Tên thuốc | BẮT BUỘC, từ drug database | "Vui lòng chọn thuốc từ danh mục" |
| Liều dùng | BẮT BUỘC, numeric | "Vui lòng nhập liều dùng" |
| Tần suất | BẮT BUỘC, từ danh sách chuẩn | "Vui lòng chọn tần suất" |
| Thời gian | BẮT BUỘC, > 0 ngày | "Vui lòng nhập thời gian sử dụng" |
| Số lượng | BẮT BUỘC, > 0 | "Vui lòng nhập số lượng" |
| Drug interaction | Tự động kiểm tra, alert nếu có interaction | "⚠️ Phát hiện tương tác thuốc: [chi tiết]" |
| Allergy check | Tự động kiểm tra, alert nếu trùng dị ứng | "⚠️ KH dị ứng với [thuốc]: [chi tiết]" |

**Wireframe ASCII:**

```
┌─────────────────────────────────────────┐
│   💊 TẠO ĐƠN THUỐC                    │
├─────────────────────────────────────────┤
│                                         │
│  KH: Nguyễn Văn A | 35M                │
│  Chẩn đoán: [ICD-10: J06.9]           │
│                                         │
│  ─── Thông tin đơn thuốc ───           │
│                                         │
│  💊 Tên thuốc *:    [_____________]    │
│  📊 Liều dùng *:    [_____________]    │
│  ⏰ Tần suất *:     [▼ Chọn      ]    │
│  📅 Thời gian *:    [___ ngày    ]    │
│  🔢 Số lượng *:     [_____________]    │
│  🔄 Refills:        [_____________]    │
│  📝 Hướng dẫn SD:   [_____________]    │
│  📝 Ghi chú LS:     [_____________]    │
│                                         │
│  ⚠️ Cảnh báo tương tác thuốc: Không   │
│  ⚠️ Dị ứng: Không phát hiện            │
│                                         │
│  [ ✅ TẠO ĐƠN THUỐC ]                 │
│  (Tạo + phê duyệt đồng thời)          │
│                                         │
│  ☐ Cần theo dõi → Chuyển Monitoring    │
│                                         │
└─────────────────────────────────────────┘
```

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Drug interaction detected | Alert BS, chọn thuốc thay thế |
| KH dị ứng với thuốc | Alert BS, chọn thuốc khác |
| Controlled substance | Yêu cầu DEA EPCS compliance |
| Tạo thành công | → Bước 12 |

---

### Bước 12: EMR Update — Cập nhật EMR

- **Ai làm:** Hệ thống
- **Mô tả:** Cập nhật đơn thuốc vào EMR, KH có thể xem/xuất từ App

**INPUT/OUTPUT:**
- INPUT: prescription_order (Object, required)
- OUTPUT: emr_updated (Boolean), export_available (Boolean)

⚠️ 100% orders PHẢI được cập nhật vào EMR

---

### Bước 13: Completion — Hoàn tất

- **Ai làm:** Hệ thống
- **Mô tả:** Đóng case HOẶC chuyển FLOW-05 nếu cần theo dõi

**INPUT/OUTPUT:**
- INPUT: emr_updated (Boolean, required), needs_monitoring (Boolean, required)
- OUTPUT: case_status (String: "Closed" | "Monitoring"), monitor_order (Object, optional)

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Chỉ kê đơn, không cần theo dõi | Đóng case |
| Thuốc có tác dụng phụ cần theo dõi | Chuyển FLOW-05, Trạm 12 |
| Cần đánh giá hiệu quả điều trị | Chuyển FLOW-05, Trạm 12 |
| Triệu chứng có thể tái phát | Chuyển FLOW-05, Trạm 12 |

> ⚠️ **CÁC BƯỚC HẬU XỬ LÝ (sau Completion):**
> - **Archive:** Lưu trữ hồ sơ buổi khám (video recording nếu có consent, visit note, đơn thuốc) — mã hóa AES-256 theo chuẩn HIPAA, lưu trữ tối thiểu 7 năm
> - **Billing:** Xử lý thanh toán — trừ quota (Care Plus: X/5), trigger billing workflow
> - **Survey:** Gửi khảo sát đánh giá dịch vụ cho KH sau 24h

---

## FLOW-04: XÉT NGHIỆM SAU VIDEO VISIT

### Sơ đồ

```
┌─ BOOKING ──────────────────────────────────────────────────────────────────┐
│ Booking → Confirm → Reminders                                              │
└───────────────────────┬────────────────────────────────────────────────────┘
                        ↓
┌─ PREP ────────────────────────────────────────────────────────────────────┐
│ Collect Docs → AI Analysis → VN MD Review                                  │
└───────────────────────┬────────────────────────────────────────────────────┘
                        ↓
┌─ VIDEO ───────────────────────────────────────────────────────────────────┐
│ Join Call → Clinical Interview → Diagnosis → Treatment Decision (Lab)      │
└───────────────────────┬────────────────────────────────────────────────────┘
                        ↓
┌─ LAB ORDER ───────────────────────────────────────────────────────────────┐
│ Lab Order Creation ⭐ → EMR Update → Completion ─┬─ [Đóng case]            │
│                                                    └─ [Theo dõi] → FLOW-05  │
└────────────────────────────────────────────────────────────────────────────┘
```

- Bước 1-9: Giống FLOW-03 (Đặt lịch → Video → Chẩn đoán)
- **Bước 10:** US MD quyết định cần xét nghiệm/hình ảnh

### Bước 11: Lab Order Creation ⭐ — Tạo chỉ định XN

- **Ai làm:** BS Mỹ / Hệ thống
- **Mô tả:** US MD tạo chỉ định với status='approved' (tạo + phê duyệt đồng thời)

**INPUT/OUTPUT:**
- INPUT: lab_decision (Object, required), test_type (String: BloodTest/Urinalysis/STI/Imaging, required), panel_name (String, required), clinical_indications (String, required), icd10_codes (Array, required), urgency (String: Routine/Urgent/STAT, required), patient_instructions (String, optional)
- OUTPUT: lab_order (Object: {order_id, status: 'approved', approved_by, created_at}), order_type (String)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Loại XN | BẮT BUỘC, từ danh sách | "Vui lòng chọn loại xét nghiệm" |
| Tên XN/Panel | BẮT BUỘC | "Vui lòng nhập tên xét nghiệm" |
| Chỉ định lâm sàng | BẮT BUỘC | "Vui lòng nhập chỉ định lâm sàng" |
| ICD-10 codes | BẮT BUỘC | "Vui lòng nhập mã ICD-10" |
| Mức độ khẩn | BẮT BUỘC | "Vui lòng chọn mức độ khẩn cấp" |

**Wireframe ASCII:**

```
┌─────────────────────────────────────────┐
│   🔬 TẠO CHỈ ĐỊNH XÉT NGHIỆM          │
├─────────────────────────────────────────┤
│                                         │
│  KH: Nguyễn Văn A | 35M                │
│  Chẩn đoán: [ICD-10: E11.9]           │
│                                         │
│  ─── Thông tin chỉ định ───            │
│                                         │
│  🔬 Loại XN *:      [▼ Chọn      ]    │
│  • Blood test   • Urinalysis            │
│  • Imaging      • STI testing           │
│                                         │
│  📋 Tên XN/Panel *: [_____________]    │
│  📝 Chỉ định LS *:  [_____________]    │
│  📝 Hướng dẫn *:    [_____________]    │
│  (VD: Nhịn ăn 12h, uống nhiều nước)   │
│                                         │
│  🏥 ICD-10 codes:   [_____________]    │
│  ⚡ Mức độ khẩn:    [▼ Routine   ]    │
│  • Routine  • Urgent  • STAT           │
│                                         │
│  [ ✅ TẠO CHỈ ĐỊNH ]                   │
│  (Tạo + phê duyệt đồng thời)          │
│                                         │
│  ☐ Cần theo dõi → Chuyển Monitoring    │
│                                         │
└─────────────────────────────────────────┘
```

**Loại chỉ định phổ biến:**

| Loại | Ví dụ |
|------|-------|
| Blood tests | CBC, CMP, Lipid panel, HbA1c, Thyroid panel |
| Urinalysis | Tổng phân tích nước tiểu |
| STI testing | Panel STI |
| Imaging | X-ray (ngực, chi, cột sống), CT scan, MRI, Siêu âm, Nhũ ảnh |

> ⚠️ **CROSS-SERVICE:** Sau khi tạo order, việc quản lý tiếp theo thuộc **→ Service 10 (Lab_Imaging_Management) - Lab/Imaging Management**

- **Bước 12-13:** Giống FLOW-03 (EMR Update → Completion)

---

## FLOW-05: THEO DÕI SAU VIDEO VISIT (MONITORING)

### 3 Entry Points

| Điểm nhập | Từ | Trạm bắt đầu | Trạm thực hiện | Quota |
|------------|-----|---------------|-----------------|-------|
| **#1** | V4 (Treatment Decision) | Trạm 1 | 1-17 (ĐẦY ĐỦ) | Tính 1 quota |
| **#2** | FLOW-03 (Completion) | Trạm 12 | 12-17 (RÚT GỌN) | KHÔNG tính thêm |
| **#3** | FLOW-04 (Completion) | Trạm 12 | 12-17 (RÚT GỌN) | KHÔNG tính thêm |

### Sơ đồ (Đầy đủ 17 trạm)

```
BOOKING(1-3) → PREP(4-6) → VIDEO(7-10) → Monitor Order(11)
    → Setup Protocol(12) → Check-in 1(13) → Check-in 2/3(14)
    → Final Review(15) → Survey(16) → Completion(17)
```

### Sơ đồ (Rút gọn Trạm 12-17)

```
[Từ FLOW-03/04] → Setup Protocol(12) → Check-in 1(13) → Check-in 2/3(14)
    → Final Review(15) → Survey(16) → Completion(17)
```

### Bước 12: Setup Protocol — Thiết lập protocol

- **Ai làm:** Hệ thống
- **Mô tả:** Thiết lập protocol theo dõi (24h/48h/72h)

**INPUT/OUTPUT:**
- INPUT: approved_monitor_order (Object, required) HOẶC escalation_from_flow03_04 (Object)
- OUTPUT: protocol_active (Object: {protocol_type, check_in_schedule, monitoring_params})

**Monitoring Protocols:**

| Protocol | Lịch check-in | Trường hợp sử dụng | Thời gian |
|----------|---------------|---------------------|-----------|
| **24h** | 1 check-in sau 24h | Triệu chứng nhẹ, low risk | 24h |
| **48h** | 2 check-ins (24h, 48h) | Triệu chứng moderate | 48h |
| **72h** | 3 check-ins (24h, 48h, 72h) | Persistent/complex | 72h |

---

### Bước 13: Check-in 1 — Check-in lần đầu

- **Ai làm:** AI / Khách hàng
- **Mô tả:** Check-in lần đầu qua App (AI hỏi, KH trả lời triệu chứng)

**INPUT/OUTPUT:**
- INPUT: protocol (Object, required), symptom_comparison (String: Better/Same/Worse, required), symptom_severity (Number: 1-10, required), new_symptoms (Boolean, required), new_symptoms_detail (String, optional), medication_adherence (Boolean, required nếu từ FLOW-03), side_effects (Boolean, required nếu từ FLOW-03), side_effects_detail (String, optional), vitals (Object: {blood_pressure, temperature, spo2}, optional)
- OUTPUT: status_update_1 (Object: {assessment, trend, ai_classification})

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Tình trạng so sánh | BẮT BUỘC chọn 1 | "Vui lòng chọn tình trạng" |
| Mức độ triệu chứng | BẮT BUỘC, 1-10 | "Vui lòng chọn mức độ" |
| Triệu chứng mới | BẮT BUỘC chọn Có/Không | "Vui lòng trả lời" |
| Uống thuốc đúng liều | BẮT BUỘC (nếu từ FLOW-03) | "Vui lòng trả lời" |
| Tác dụng phụ | BẮT BUỘC (nếu từ FLOW-03) | "Vui lòng trả lời" |

**Wireframe ASCII:**

```
┌─────────────────────────────────────┐
│   📋 CHECK-IN THEO DÕI SỨC KHỎE    │
├─────────────────────────────────────┤
│                                     │
│  🕐 Check-in lần 1/3 (Protocol 72h)│
│                                     │
│  1. Tình trạng hôm nay so với      │
│     hôm qua?                       │
│  ( ) Tốt hơn   ( ) Như cũ          │
│  ( ) Xấu hơn                       │
│                                     │
│  2. Mức độ triệu chứng (1-10):     │
│  [▼ 1-10                     ]     │
│                                     │
│  3. Có triệu chứng mới không?      │
│  ( ) Không   ( ) Có                 │
│  Nếu có: [___________________]     │
│                                     │
│  4. Đang uống thuốc đúng liều?     │
│  ( ) Có      ( ) Không              │
│                                     │
│  5. Tác dụng phụ?                   │
│  ( ) Không   ( ) Có                 │
│  Nếu có: [___________________]     │
│                                     │
│  📊 Dấu hiệu sinh tồn (nếu có):   │
│  Huyết áp:  [___/___]              │
│  Nhiệt độ:  [___°C  ]              │
│  SpO2:      [___%   ]              │
│                                     │
│  [  GỬI CHECK-IN  ]                │
│                                     │
└─────────────────────────────────────┘
```

⚠️ KH không check-in sau 4h → CS gọi điện
⚠️ Có red flags → MD review ngay

---

### Bước 14: Check-in 2/3 — Check-in tiếp theo

- **Ai làm:** AI / Khách hàng
- **Mô tả:** Check-in tiếp theo (1-2 lần tùy protocol)

**INPUT/OUTPUT:**
- INPUT: status_update_1 (Object, required), (giống Check-in 1)
- OUTPUT: status_updates (Array of status_update objects)

⚠️ KH không check-in → CS gọi
⚠️ Xu hướng xấu đi (worsening trend) → MD review

**Check-in reminders:**

| Timeline | Kênh | Hành động nếu bỏ lỡ |
|----------|------|---------------------|
| 2h trước | Push + SMS | Reminder |
| Quá hạn 2h | Push + SMS | "Chưa hoàn thành check-in" |
| Quá hạn 6h | Phone call | CC gọi kiểm tra |
| Không liên lạc | Escalate | MD quyết định close hoặc emergency check |

---

### Bước 15: Final Review — Đánh giá cuối cùng

- **Ai làm:** BS VN / BS Mỹ
- **Mô tả:** Review toàn bộ status updates, đánh giá tình trạng

**INPUT/OUTPUT:**
- INPUT: all_status_updates (Array, required), ai_assessment (String: Stable/Improving/Worsening, required)
- OUTPUT: final_assessment (Object: {decision, md_notes}), decision (String: Close/Extend/Escalate)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────┐
│   📊 FINAL REVIEW — MONITORING          │
├─────────────────────────────────────────┤
│                                         │
│  KH: Nguyễn Văn A | 35M                │
│  Protocol: 72h | Check-ins: 3/3 ✅      │
│                                         │
│  ─── Tóm tắt Check-ins ───             │
│                                         │
│  📅 24h: Tốt hơn | Đau 6/10 | Uống thuốc ✅ │
│  📅 48h: Tốt hơn | Đau 4/10 | Uống thuốc ✅ │
│  📅 72h: Tốt hơn | Đau 2/10 | Uống thuốc ✅ │
│                                         │
│  🤖 AI Assessment: IMPROVING            │
│                                         │
│  ─── Quyết định ───                    │
│                                         │
│  ( ) ✅ Đóng case (ổn định)             │
│  ( ) ⏰ Gia hạn thêm 24-48h            │
│  ( ) ⚠️ Escalate                        │
│      ( ) Emergency Protocol              │
│      ( ) Thuốc mới                      │
│      ( ) Xét nghiệm mới                 │
│      ( ) Video Visit mới                │
│                                         │
│  📝 Ghi chú bác sĩ:                    │
│  [_______________________________]      │
│                                         │
│  [ XÁC NHẬN QUYẾT ĐỊNH ]               │
│                                         │
└─────────────────────────────────────────┘
```

⛔ **Rẽ nhánh — Final Review Decision:**

| Tình trạng | Hành động |
|-------------|-----------|
| Ổn định / Improving | Đóng case |
| Same (không thay đổi) | Gia hạn thêm 24-48h HOẶC đóng case |
| Worsening (xấu đi) | Escalate: Emergency / Thuốc mới / XN mới / Video Visit mới |

⚠️ MD không review sau 20h → CS nhắc MD

---

### Bước 16: Survey — Khảo sát

- **Ai làm:** Hệ thống
- **Mô tả:** Gửi khảo sát đánh giá dịch vụ cho KH

**INPUT/OUTPUT:**
- INPUT: final_assessment (Object, required)
- OUTPUT: survey_sent (Boolean)

---

### Bước 17: Completion — Hoàn tất Monitoring

- **Ai làm:** Hệ thống
- **Mô tả:** Cập nhật EMR, đóng case

**INPUT/OUTPUT:**
- INPUT: survey (Object, optional), final_assessment (Object, required)
- OUTPUT: case_closed (Boolean), emr_updated (Boolean)

---

## FLOW-01: HỦY LỊCH HẸN

### Sơ đồ

```
KH đặt lịch → KH nhận xác nhận → KH yêu cầu hủy ─┬─ [>2h] → Hủy miễn phí → END
                                                     │
                                                     └─ [<2h] → Gọi hotline → CC xử lý → END
```

### Nhánh A: Hủy miễn phí (> 2h trước lịch hẹn)

- **Bước 3A:** Khách hàng — Click "Cancel Appointment" trong App

**INPUT/OUTPUT:**
- INPUT: appointment_id (String, required)
- OUTPUT: cancel_request (Object: {request_type: "free", timestamp})

- **Bước 4A-1:** Hệ thống — Xác nhận thời điểm hủy > 2h trước cuộc hẹn

**INPUT/OUTPUT:**
- INPUT: cancel_request (Object, required), appointment_time (DateTime, required)
- OUTPUT: eligible_for_free (Boolean)

- **Bước 4A-2:** Hệ thống — Release slot, hoàn quota (nếu đã trừ), cập nhật trạng thái

**INPUT/OUTPUT:**
- INPUT: eligible_for_free (Boolean, required)
- OUTPUT: cancellation_complete (Object: {status: "Canceled by Customer", quota_impact: "No deduction", slot_released: true})

- **Bước 4A-3:** Hệ thống — Gửi Email + SMS xác nhận hủy, đề xuất đặt lịch mới

**INPUT/OUTPUT:**
- INPUT: cancellation_complete (Object, required)
- OUTPUT: notification_sent (Boolean), channels (Array: [Email, SMS])

**Wireframe ASCII:**

```
┌─────────────────────────────────────┐
│         HỦY LỊCH HẸN               │
├─────────────────────────────────────┤
│                                     │
│  ⚠️ Bạn có chắc muốn hủy          │
│  lịch hẹn này?                     │
│                                     │
│  📅 Ngày: 10/02/2026               │
│  🕐 Giờ: 14:00 (EST)              │
│  👨‍⚕️ BS: Dr. Smith                  │
│                                     │
│  ✅ Hủy miễn phí (còn > 2h)        │
│  Quota không bị trừ                 │
│                                     │
│  📝 Lý do hủy (tùy chọn):         │
│  [▼ Chọn lý do              ]      │
│  • Không còn cần khám               │
│  • Có việc bận đột xuất             │
│  • Muốn đổi thời gian khác         │
│  • Lý do khác                       │
│                                     │
│  [ ❌ XÁC NHẬN HỦY ]              │
│  [    Quay lại      ]              │
│                                     │
└─────────────────────────────────────┘
```

### Nhánh B: Hủy muộn (< 2h trước lịch hẹn)

- **Bước 3B:** Khách hàng — Hệ thống thông báo PHẢI gọi hotline

**INPUT/OUTPUT:**
- INPUT: appointment_id (String, required)
- OUTPUT: hotline_redirect (Object: {message, hotline_number: "1900-xxxx"})

- **Bước 4B-1:** Khách hàng — Gọi hotline 1900-xxxx
- **Bước 4B-2:** Care Coordinator — Xác nhận danh tính, hỏi lý do hủy

**INPUT/OUTPUT:**
- INPUT: call_connected (Boolean, required)
- OUTPUT: reason_captured (Object: {reason, is_emergency})

- **Bước 4B-3:** Care Coordinator — Quyết định quota

**INPUT/OUTPUT:**
- INPUT: reason_captured (Object, required)
- OUTPUT: quota_decision (String: "waived" | "deducted")

⛔ **Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Emergency reason (nhập viện) | Hủy miễn phí, KHÔNG trừ quota |
| Non-emergency | Tính quota (Care Plus) |

- **Bước 4B-4:** Hệ thống — Hủy lịch, trừ/không trừ quota, ghi log, gửi xác nhận

**Wireframe ASCII:**

```
┌─────────────────────────────────────┐
│     ⚠️ KHÔNG THỂ HỦY QUA HỆ THỐNG │
├─────────────────────────────────────┤
│                                     │
│  Cuộc hẹn của bạn còn < 2 giờ.     │
│  Để hủy lịch lúc này, vui lòng gọi:│
│                                     │
│  ☎️ Hotline: 1900-xxxx (24/7)       │
│                                     │
│  ⚠️ Lưu ý: Hủy ít hơn 2h trước    │
│  sẽ mất 1 lượt khám (Care Plus)    │
│                                     │
│  [    Quay lại      ]              │
│                                     │
└─────────────────────────────────────┘
```

### Xử lý trường hợp đặc biệt

| Trường hợp | Xử lý |
|-------------|-------|
| Emergency cancellation (nhập viện) | Hủy miễn phí bất kể thời điểm, yêu cầu documentation (hospital admission) |
| System outage (lỗi hệ thống CVH) | CVH chủ động hủy, hoàn quota đầy đủ, đề xuất reschedule + compensation |
| MD cancellation (BS không available) | CVH hủy, hoàn quota, ưu tiên reschedule với MD khác trong 24h |
| Reschedule thay vì hủy | > 2h: Chọn slot mới qua App; < 2h: Gọi hotline, tính quota cho lịch cũ |

### Quota Policy theo gói

| Gói | Quota/năm | Hủy trước 2h | Hủy trong 2h |
|-----|-----------|--------------|---------------|
| Care Plus | 5 visits | KHÔNG tính quota | Tính quota (trừ emergency) |
| Care Premium | Unlimited | KHÔNG tính quota | KHÔNG tính quota |

### No-show Policy

| Trường hợp | Quota Impact | Hành động |
|-------------|-------------|-----------|
| KH đã confirm + no-show | KHÔNG mất quota | Follow-up email, đề xuất reschedule miễn phí |
| KH chưa confirm + no-show | Mất quota (Care Plus) | Follow-up email, ghi nhận no-show |
| No-show 2 lần trong 3 tháng | Warning | Cảnh báo, có thể hạn chế booking |
| Care Premium | KHÔNG mất quota (mọi TH) | Follow-up email |

---

## FLOW-02: CẤP CỨU TRONG VIDEO VISIT

### Sơ đồ

```
Booking → Confirm → Reminders → Collect Docs → AI Analysis → VN MD Review
    → Join Call → Clinical Interview → ⚠️ Emergency Detected → Emergency Order → ER Protocol → END
```

### Bước 9: Emergency Detected — BS phát hiện cấp cứu

- **Ai làm:** BS Mỹ
- **Mô tả:** BS phát hiện triệu chứng KHẨN CẤP, click "Emergency Alert"

**INPUT/OUTPUT:**
- INPUT: clinical_data (Object, required), emergency_type (String, required)
- OUTPUT: emergency_alert (Object: {alert_id, timestamp, type, patient_location})

**Wireframe ASCII — Emergency Alert:**

```
┌─────────────────────────────────────────┐
│  🎥 Video Call — KH: Nguyễn Văn A      │
│  Tuổi: 55M | CC: Đau ngực             │
├─────────────────────────────────────────┤
│                                         │
│  [🚨 EMERGENCY ALERT — NÚT ĐỎ]       │
│                                         │
│  📝 Ghi chú lâm sàng:                  │
│  [Đau ngực bắt đầu 30 phút trước...]  │
│                                         │
│  📍 Vị trí KH:                          │
│  123 Main St, Boston, MA 02101         │
│                                         │
│  📞 Liên hệ: (617) 555-1234           │
│                                         │
└─────────────────────────────────────────┘
```

**Khi click Emergency Alert:**

```
┌─────────────────────────────────────────┐
│  ⚠️ XÁC NHẬN CẤP CỨU                  │
├─────────────────────────────────────────┤
│                                         │
│  Đây có phải tình huống đe dọa         │
│  tính mạng không?                       │
│                                         │
│  [ ✅ CÓ — KÍCH HOẠT CẤP CỨU ]       │
│  [ ❌ KHÔNG — QUAY LẠI        ]       │
│                                         │
└─────────────────────────────────────────┘
```

### Bước 10: Emergency Order — Tạo lệnh cấp cứu

- **Ai làm:** Hệ thống
- **Mô tả:** Tạo lệnh chuyển cấp cứu ngay lập tức

**INPUT/OUTPUT:**
- INPUT: emergency_alert (Object, required)
- OUTPUT: emergency_order (Object: {order_id, status: "active", priority: "critical"})

### Bước 11: ER Protocol — Quy trình cấp cứu

- **Ai làm:** Care Coordinator
- **Mô tả:** Kích hoạt quy trình ER: Join 3-way call, gọi 911, hướng dẫn KH

**INPUT/OUTPUT:**
- INPUT: emergency_order (Object, required), patient_location (String, required), emergency_contact (String, required)
- OUTPUT: er_contacted (Boolean), ambulance_dispatched (Boolean), handoff_complete (Boolean)

⚠️ CC pickup < 30 giây
⚠️ 911 call < 5 phút
⚠️ Handoff Success Rate = 100%

**Wireframe ASCII — CC Emergency Dashboard:**

```
┌──────────────────────────────────────────┐
│  🚨 EMERGENCY ALERT                     │
├──────────────────────────────────────────┤
│                                          │
│  KH: Nguyễn Văn A, 55M                  │
│  BS: Dr. Smith                           │
│  Thời gian alert: 10:45:23 AM           │
│                                          │
│  📍 Vị trí: 123 Main St, Boston, MA     │
│  📞 SĐT: (617) 555-1234                │
│                                          │
│  CC: Đau ngực dữ dội                    │
│                                          │
│  [THAM GIA VIDEO]  [GỌI 911]  [GỌI KH] │
│                                          │
└──────────────────────────────────────────┘
```

> ⚠️ **EMERGENCY RED FLAGS** (BS PHẢI alert ngay):
> - Đau ngực (chest pain) — nghi ngờ heart attack
> - Khó thở nặng (severe shortness of breath)
> - Triệu chứng đột quỵ (FAST: Face drooping, Arm weakness, Speech difficulty, Time)
> - Xuất huyết nặng không kiểm soát
> - Mất ý thức hoặc altered mental status
> - Phản ứng dị ứng nặng (anaphylaxis)
> - Đau bụng dữ dội (nghi appendicitis, ectopic pregnancy)
> - Co giật (seizure/convulsion)
> - Ý định tự tử có kế hoạch (suicidal ideation with plan)

**Xử lý trường hợp đặc biệt:**

| Trường hợp | Xử lý |
|-------------|-------|
| KH mất ý thức trong call | BS trigger alert ngay, CC gọi 911 + emergency contact, BS giữ video để theo dõi |
| KH từ chối gọi 911 | BS giải thích rõ rủi ro, đề xuất "Let me call for you", document refusal, follow-up 1h |
| Emergency contact không available | Gọi 911 trực tiếp, request welfare check, thử liên hệ neighbor/building manager |

> ⚠️ **QUOTA:** Emergency KHÔNG tính quota. Nếu emergency phát hiện trong call → refund quota đã trừ.

---

## BẢNG THÔNG BÁO TỰ ĐỘNG (MỞ RỘNG VỚI TEMPLATE)

| Sự kiện | Kênh | Người nhận |
|---------|------|------------|
| Xác nhận đặt lịch | Email + SMS + Push | Khách hàng |
| Nhắc nhở 3h trước (confirm/cancel) | Email + SMS + Push | Khách hàng |
| SMS xác nhận sau khi KH click Confirm | SMS | Khách hàng |
| Nhắc nhở 30 phút trước (ready) | SMS + Push | Khách hàng |
| Hủy miễn phí (> 2h) | Email + SMS + Push | Khách hàng |
| Hủy muộn (< 2h) | Email + SMS + Push | Khách hàng |
| Visit Summary (sau buổi khám) | Email | Khách hàng |
| Prescription notification | SMS + Push | Khách hàng |
| No-show notification | Email + SMS | Khách hàng |
| Monitoring check-in reminder | Push + SMS | Khách hàng |
| MD notification khi KH hủy | Email + SMS | US MD |

### Template chi tiết

📧 **Email xác nhận đặt lịch:**
- Subject: "Xác nhận đặt lịch khám video — [Date/Time]"
- Body:
```
Xin chào [Tên KH],

Bạn đã đặt lịch khám video thành công!

📅 Ngày: [Date]
🕐 Giờ: [Time] ([Timezone])
👨‍⚕️ Bác sĩ: Dr. [Name], [Specialty]

🔗 Link tham gia: [HIPAA-compliant video link]
📅 Thêm vào lịch: [Google] [Apple] [Outlook]

⚠️ Lưu ý:
- Bạn có thể hủy miễn phí đến 2h trước cuộc hẹn
- Hãy chuẩn bị internet ổn định, camera và mic trước khi tham gia

Cần hỗ trợ? Liên hệ: 1900-xxxx (24/7)

CVH Healthcare Team
```

📱 **SMS xác nhận đặt lịch:**
```
Đặt lịch thành công! Khám video [Date] lúc [Time].
Link: [shortlink]
Hủy miễn phí đến 2h trước.
```

📱 **SMS Reminder 3h:**
```
Cuộc khám sắp bắt đầu trong 3 giờ!
Thời gian: [Date] [Time]
Bác sĩ: [Doctor Name]
Xác nhận: [link1]
Hủy lịch: [link2]
```

📱 **SMS xác nhận sau khi KH click Confirm:**
```
✅ Đã xác nhận!
Cuộc khám: [Date] [Time]
Link tham gia: [link]
Chuẩn bị:
- Kiểm tra internet
- Kiểm tra camera/mic
- Tìm nơi riêng tư
```

📱 **SMS Reminder 30 phút:**
```
🎥 Cuộc khám sắp bắt đầu trong 30 phút!
Thời gian: [Time]
Link tham gia: [link]
Vui lòng chuẩn bị:
✓ Internet ổn định
✓ Camera/mic hoạt động
✓ Nơi riêng tư
```

📧 **Email xác nhận hủy miễn phí:**
```
Lịch hẹn video visit của bạn đã được hủy thành công.

Thông tin lịch hẹn:
- Ngày giờ: [DateTime]
- Bác sĩ: Dr. [Name]

Quota Care Plus: Đã hoàn lại (nếu có)
Bạn vẫn còn [X] lượt khám trong năm nay.

Bạn có thể đặt lịch mới bất kỳ lúc nào.
[ĐẶT LỊCH MỚI]

CVH Team
```

📧 **Email xác nhận hủy muộn:**
```
Lịch hẹn video visit của bạn đã được hủy.

Thông tin lịch hẹn:
- Ngày giờ: [DateTime]
- Bác sĩ: Dr. [Name]

LƯU Ý: Vì hủy trong vòng 2h trước cuộc hẹn,
visit này sẽ tính vào quota Care Plus của bạn.

Quota còn lại tháng này: [X]/5

CVH Team
```

📧 **Email Visit Summary:**
```
Cảm ơn bạn đã sử dụng dịch vụ Video Visit của CVH!

Thông tin cuộc hẹn:
- Ngày giờ: [DateTime]
- Bác sĩ: Dr. [Name], [Specialty]

Chẩn đoán: [Diagnosis]
Điều trị: [Treatment plan summary]

Đơn thuốc: [Medication names, dosage] (nếu có)
Chỉ định XN: [Lab/Imaging orders] (nếu có)

Bạn có thể xem chi tiết trong ứng dụng CVH:
Menu > Hồ sơ sức khỏe > [Đơn thuốc / Chỉ định XN]

Hướng dẫn:
- [Uống thuốc đúng liều/đến lab đúng hẹn]
- [Tái khám nếu triệu chứng không cải thiện sau X ngày]
- [Dấu hiệu cần đi cấp cứu ngay]

CVH Team
```

📱 **SMS/Push Prescription notification:**
```
Đơn thuốc của bạn đã được tạo trong hệ thống CVH.
Bạn có thể xem và xuất đơn thuốc từ ứng dụng CVH.
Vào: Menu > Hồ sơ sức khỏe > Đơn thuốc
```

📧 **Email No-show notification:**
```
Bạn đã bỏ lỡ cuộc hẹn khám video hôm nay.

Thông tin lịch hẹn:
- Ngày giờ: [DateTime]
- Bác sĩ: Dr. [Name]

[Nếu đã confirm: Quota KHÔNG bị trừ]
[Nếu chưa confirm: Quota đã bị trừ. Còn lại: X/5 lượt]

Bạn có muốn đặt lịch lại?
[ĐẶT LỊCH MỚI]

Mọi thắc mắc: 1900-xxxx (24/7)

CVH Team
```

📱 **Push/SMS Monitoring check-in reminder:**
```
📋 Đã đến giờ check-in theo dõi sức khỏe!
Nhấn vào đây để hoàn thành check-in.
[link]
```

📧 **Email/SMS MD notification khi KH hủy:**
```
Lịch hẹn đã bị hủy:
- KH: [Patient Name]
- Thời gian: [DateTime]
- Lý do: [Reason]
Slot đã được giải phóng.
```

---

## BẢNG GIÁM SÁT CS-MD ĐẦY ĐỦ VỚI KPI

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Booking | 100% form đầy đủ, 100% trạng thái = "Chờ xác nhận", 0% duplicate | Auto tooltip hướng dẫn, validate real-time, check duplicate slot → suggest slot khác, check quota → popup nâng cấp, DB error → retry 3 lần (2s/lần) | DB sập > 10 phút (auto-healing failed), KH khiếu nại nghiêm trọng → CS gọi lại đặt lịch thủ công | ❌ |
| Confirm | 100% gửi trong 1 phút, 100% đúng kênh (Email + SMS + Push) | Auto gửi qua 3 kênh, retry nếu fail 1 kênh, log delivery status | Gateway sập > 30 phút → CS gọi KH xác nhận bằng điện thoại | ❌ |
| Cancel Request | 100% phân loại đúng (> 2h vs < 2h) | Auto check thời gian, > 2h → form hủy, < 2h → redirect hotline | App sập hoàn toàn → CS nhận call trực tiếp | ❌ |
| Process Cancellation | 100% quota xử lý đúng | Auto release slot, hoàn/trừ quota, gửi xác nhận | Lỗi quota → CS xử lý thủ công | ❌ |
| Reminders | 100% gửi đúng T-3h và T-30min | Auto schedule, gửi 3 kênh (3h) hoặc 2 kênh (30min), follow-up nếu không phản hồi | Gateway sập > 30 phút → CS gọi KH nhắc nhở | ❌ |
| Collect Docs | 100% thu thập trong 2-3 phút | Auto kéo dữ liệu EMR, flag missing data | ❌ | VN MD: xử lý data conflicts |
| AI Analysis | 100% hoàn thành trong 3-5 phút | Auto phân tích, tạo summary, phát hiện drug interaction | ❌ | VN MD: review critical flags |
| VN MD Review | Review 2-4h trước cuộc hẹn | Auto assign VN MD, gửi reminder, track thời gian review | Alert nếu VN MD no-show < 5 phút trước giờ hẹn | VN MD review AI summary, validate, flag concerns |
| Join Call | Waiting room 5 phút trước, auto-call nếu no-show 5 phút | Auto technical check, auto-call KH nếu no-show | Lỗi kết nối nghiêm trọng → CS hỗ trợ troubleshoot | ❌ |
| Clinical Interview | Duration 15-25 phút | Auto recording (nếu consent), auto note-taking | ❌ | US MD thực hiện khám |
| Emergency Detected | CC pickup < 30 giây | Auto alert CC Dashboard | ❌ | US MD phát hiện, click Emergency Alert |
| Emergency Order | Tạo order < 1 phút | Auto tạo emergency order | ❌ | US MD xác nhận |
| ER Protocol | 911 call < 5 phút, Handoff 100% | Auto log timeline | CS gọi 911, join 3-way call, liên hệ ER | ❌ |
| Diagnosis | PHẢI có ICD-10 code | Auto suggest ICD-10 codes | ❌ | US MD chẩn đoán |
| Treatment Decision (Rx) | PHẢI chọn ít nhất 1 pathway | Auto hiển thị options | ❌ | US MD quyết định |
| Rx Order Creation | Drug interaction check, Allergy check | Auto check tương tác thuốc + dị ứng, alert nếu có | ❌ | US MD tạo đơn thuốc (status='approved') |
| EMR Update | 100% orders cập nhật EMR | Auto sync EMR, retry nếu fail | ❌ | ❌ |
| Completion (Rx) | Auto đóng case hoặc trigger FLOW-05 | Auto xử lý, gửi survey 24h sau | ❌ | ❌ |
| Treatment Decision (Lab) | PHẢI có clinical indication + ICD-10 | Auto validate required fields | ❌ | US MD quyết định |
| Lab Order Creation | PHẢI có ICD-10 code | Auto validate | ❌ | US MD tạo order (status='approved') |
| Treatment Decision (Monitor) | PHẢI chọn protocol duration | Auto hiển thị protocol options | ❌ | US MD quyết định |
| Monitor Order | Protocol setup trong 5 phút | Auto thiết lập protocol | ❌ | US MD tạo order (status='approved') |
| Setup Protocol | 100% protocol active | Auto lên lịch check-in | ❌ | ❌ |
| Check-in 1 | KH check-in đúng hạn | Auto gửi reminder 2h trước, push + SMS | CS gọi nếu KH không check-in > 4h | MD review nếu red flags |
| Check-in 2/3 | KH check-in đúng hạn | Auto gửi reminder, track trend | CS gọi nếu KH không check-in > 4h | MD review nếu worsening trend |
| Final Review | MD review trong 20h | Auto gửi AI assessment, reminder MD | CS nhắc MD nếu > 20h | VN/US MD final review, quyết định |
| Survey | Gửi survey 24h sau | Auto gửi survey | ❌ | ❌ |
| Completion (Monitor) | 100% EMR cập nhật | Auto cập nhật EMR, đóng case | ❌ | ❌ |

---

## DATA SPECIFICATION

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| appointment_id | String | Auto | SVV-YYYYMMDD-XXXXX | Mã lịch hẹn |
| patient_id | String | Yes | CVH-XXXXXXX | Mã khách hàng |
| md_id | String | Yes | MD-XXXXXXX | Mã bác sĩ |
| appointment_date | Date | Yes | YYYY-MM-DD, ≥ now + 24h | Ngày hẹn |
| appointment_time | String | Yes | HH:MM (timezone) | Giờ hẹn |
| chief_complaint | String | Yes | Max 500 ký tự | Lý do khám chính |
| symptoms | String | Yes | Max 1000 ký tự | Mô tả triệu chứng |
| symptom_onset | String | No | Free text | Khi nào bắt đầu |
| pain_level | Number | No | 1-10 | Mức độ đau |
| service_plan | String | Yes | CarePlus / CarePremium | Gói dịch vụ |
| quota_remaining | Number | Auto | 0-5 (Care Plus only) | Số lượt còn lại |
| status | String | Auto | Pending / Confirmed / Canceled / Completed / No-show | Trạng thái lịch hẹn |
| cancellation_reason | String | No | Dropdown selection | Lý do hủy |
| video_link | String | Auto | HIPAA-compliant URL | Link video call |
| documents_package | Object | Auto | {medical_history, medications, allergies, lab_results} | Hồ sơ y tế |
| ai_summary | Object | Auto | {summary_text, drug_interactions, suggested_questions} | Tóm tắt AI |
| clinical_data | Object | Auto | {hpi, ros, exam_findings} | Dữ liệu lâm sàng |
| diagnosis | Object | Yes (MD) | {icd10_codes[], assessment} | Chẩn đoán |
| prescription_order | Object | Conditional | {drug_name, dosage, frequency, duration, quantity, refills, sig, status:'approved'} | Đơn thuốc |
| lab_order | Object | Conditional | {test_type, panel_name, clinical_indications, icd10_codes, urgency, status:'approved'} | Chỉ định XN |
| monitor_order | Object | Conditional | {protocol_type: 24h/48h/72h, monitoring_params} | Chỉ định theo dõi |
| emergency_order | Object | Conditional | {emergency_type, patient_location, emergency_contact, priority:'critical'} | Lệnh cấp cứu |
| check_in_data | Object | Conditional | {symptom_comparison, severity, new_symptoms, medication_adherence, side_effects, vitals} | Dữ liệu check-in |
| final_assessment | Object | Conditional | {decision: Close/Extend/Escalate, md_notes} | Đánh giá cuối |
| hipaa_audit_log | Object | Auto | {action, actor, timestamp, ip_address} | Log kiểm toán HIPAA |

---

## NGUỒN TÀI LIỆU GỐC

- Luồng nghiệp vụ: `docs/02-product/ba-specs/service-3-Schedule_Video_Visit/base/Schedule_Video_Visit_Touchpoint_Flow_v1.md`
- Checklist CS & MD: `docs/02-product/ba-specs/service-3-Schedule_Video_Visit/checklist-cs-md/checklist-cs-md.md`
- Bản tóm tắt: `docs/02-product/tomtat-chitiet/tomtat/service-3-Schedule_Video_Visit-tomtat.md`
