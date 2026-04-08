# CHI TIẾT LUỒNG NGHIỆP VỤ – SERVICE 4 (Chronic_Management): QUẢN LÝ BỆNH MÃN TÍNH (CHRONIC DISEASE MANAGEMENT)

## Header

- **Service:** Chronic Disease Management (CDM)
- **Version:** 1.1.0
- **Input:** Bệnh nhân (BN) đăng ký CDM — tài khoản CVH active + đang điều trị bệnh mãn tính + consent
- **Output:** Care Plan cá nhân hóa + 4 Chương trình quản lý active + Theo dõi liên tục
- **Thời lượng:** Continuous (vòng lặp liên tục)
- **Compliance:** CMS CCM, NCQA PCMH, HL7 FHIR R4, ADA 2025

---

## Nguyên tắc cốt lõi

- **AI-first Model:** AI phân tích, MD phê duyệt
- **Continuous Loop:** 4 chương trình (P1-P4) chạy SONG SONG liên tục
- **Care Plan review:** Mỗi 3 THÁNG
- BN chỉ nhập dữ liệu thô (bữa ăn, vận động, giấc ngủ). AI phân tích tính toán metrics (calo, dinh dưỡng, chất lượng giấc ngủ, v.v.)

---

## Danh sách Happy Path

| HP | Tên | Mô tả |
|----|-----|-------|
| HP-01 | Đăng ký & Đánh giá Ban đầu | BN đăng ký → Care Plan + Kích hoạt 4 Chương trình |
| HP-02 | Chu trình Cập nhật Điều trị | Nhắc thuốc → Đánh giá tuân thủ → Điều chỉnh Care Plan |
| HP-03 | Chu trình Cập nhật Lối sống | Dữ liệu lối sống → AI phân tích → Coaching cá nhân hóa |
| HP-04 | Chu trình Xử lý Kết quả XN | Lịch XN → Nhận kết quả → Đánh giá → Điều chỉnh Care Plan |
| HP-05 | Chuyên khoa Chuyên gia | Trigger → Chuyển Service 2 (Specialist_Referral) → Cập nhật Care Plan |
| HP-06 | Sự kiện Cấp tính | Trigger → Chuyển Service 1 (Access_to_Care_247) → Điều chỉnh Care Plan |

---

## Sơ đồ luồng tổng quan

```
                              ┌─────────────────┐
                              │  BN Đăng ký CDM │
                              │     (HP-01)     │
                              └────────┬────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AI ĐÁNH GIÁ TÌNH TRẠNG Y KHOA                        │
│         CDM_03 + CDM_03B (CMS) + CDM_03C (NCQA)                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TẠO KẾ HOẠCH CHĂM SÓC (CARE PLAN v2.0)              │
│         CDM_04 + CDM_04B (CMS) + CDM_04C (NCQA) → MD Approval               │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         KÍCH HOẠT 4 CHƯƠNG TRÌNH (SONG SONG)                │
├─────────────────┬─────────────────┬─────────────────┬───────────────────────┤
│ P1: ĐIỀU TRỊ    │ P2: SINH HOẠT   │ P3: REFERRAL    │ P4: CẤP TÍNH         │
│ (HP-02 + HP-04) │ (HP-03)         │ (HP-05)         │ (HP-06)              │
│ [ACTIVE]        │ [ACTIVE]        │ [CONDITIONAL]   │ [ON-DEMAND]          │
└────────┬────────┴────────┬────────┴────────┬────────┴───────────┬───────────┘
         │                 │                 │                    │
         └─────────────────┴─────────────────┴────────────────────┘
                                       │
                                       ▼
                         ┌─────────────────────────┐
                         │    CẬP NHẬT KẾT QUẢ     │
                         │  (Feedback → AI Loop)   │
                         └─────────────────────────┘
```

---

## HP-01: ĐĂNG KÝ & ĐÁNH GIÁ BAN ĐẦU

> **Tổng quan:** BN đăng ký → AI khởi tạo hồ sơ → Thu thập dữ liệu → Đánh giá → Tạo Care Plan → Kích hoạt 4 chương trình

```
CDM_01 → CDM_02 → CDM_03/03B/03C → CDM_04/04B/04C → CDM_05
```

### Bước 1: BN xác nhận đăng ký CDM

- **Đối tượng:** BN
- **Mô tả:** Đồng ý điều khoản CDM + consent AI. Chọn bệnh chính, bệnh phụ
- **INPUT:** primary_condition (Enum, required), secondary_conditions (Array, optional), consent_cdm (Boolean, required), consent_ai (Boolean, required)
- **OUTPUT:** profile_id (UUID)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| primary_condition | Required | Vui lòng chọn bệnh mãn tính chính |
| consent_cdm | Must be true | Vui lòng đồng ý điều khoản |
| consent_ai | Must be true | Vui lòng cho phép AI phân tích |

**UI States:**
- Default: Dropdown chưa chọn, Checkbox chưa tick, Button disabled (xám)
- Valid: Dropdown đã chọn, 2 Checkbox đã tick, Button enabled (xanh)
- Error: Border đỏ + Message lỗi cho field chưa điền

**WIREFRAME:**

```
┌─────────────────────────────────────────────────────────────┐
│ ← Quay lại                           CVH Logo               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│        🏥 ĐĂNG KÝ QUẢN LÝ BỆNH MÃN TÍNH                    │
│                                                             │
│  Bệnh mãn tính chính:                                       │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ ▼  Tiểu đường Type 2 (E11.9)                         │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  Bệnh đồng mắc:                                             │
│  ☑ Tăng huyết áp (I10)                                     │
│  ☑ Rối loạn lipid máu (E78.0)                              │
│  ☐ Bệnh tim mạch                                           │
│                                                             │
│  ☑ Tôi đồng ý sử dụng dịch vụ CDM                          │
│  ☑ Tôi cho phép AI phân tích dữ liệu y tế của tôi          │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              🚀 ĐĂNG KÝ NGAY                          │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Bước 2: Khởi tạo CDM Profile (AI)

- **Đối tượng:** AI
- **Mô tả:** Tạo profile với status = INITIATING (SLA: 20s)
- **INPUT:** profile_id (UUID, required), patient_id (UUID, required)
- **OUTPUT:** ehr_data (Object — tiền sử bệnh, thuốc đang dùng, XN gần nhất, sinh hiệu)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| EHR đầy đủ | Chuyển bước đánh giá |
| Thiếu dữ liệu bắt buộc | Gắn cờ "Incomplete", tiếp tục với dữ liệu có sẵn |
| Lỗi kỹ thuật | Retry 3 lần (5s/lần) → alert DevOps nếu fail |

---

### Bước 3: Thu thập dữ liệu EHR (AI)

- **Đối tượng:** AI
- **Mô tả:** Truy xuất diagnoses, medications, labs từ EHR
- **INPUT:** profile_id (UUID, required)
- **OUTPUT:** ehr_data (Object — diagnoses, medications, labs, vitals)

---

### Bước 4: Đánh giá tình trạng y khoa (AI)

- **Đối tượng:** AI
- **Mô tả:** Đánh giá 5 yếu tố lâm sàng: mức độ bệnh, nguy cơ biến chứng, khoảng trống điều trị, ưu tiên can thiệp, đề xuất ban đầu (SLA: 60s)
- **INPUT:** profile_id (UUID, required), ehr_data (Object, required)
- **OUTPUT:** assessment_report (Object — severity, risks, gaps, priorities, recommendations)

⚠️ Phát hiện red flag lâm sàng → tự động gửi cảnh báo cho MD VN

---

### Bước 5: Thu thập Problem List [CMS] (AI)

- **Đối tượng:** AI
- **Mô tả:** Thu thập ≥2 chronic conditions theo CMS CCM
- **INPUT:** profile_id (UUID, required), diagnoses (Array, required)
- **OUTPUT:** problem_list (Array), ccm_eligible (Boolean)

---

### Bước 6: Thu thập Patient Preferences [NCQA] (AI/BN)

- **Đối tượng:** AI/BN
- **Mô tả:** Sở thích, SDOH, cultural preferences
- **INPUT:** profile_id (UUID, required)
- **OUTPUT:** preferences (Object)

---

### Bước 7: Tạo Care Plan v2.0 (AI)

- **Đối tượng:** AI
- **Mô tả:** 12 sections theo US CCM standards (SLA: 45s)
- **INPUT:** assessment (Object, required), problem_list (Array, required), preferences (Object, required)
- **OUTPUT:** care_plan (Object — medications, goals, lifestyle_plan, follow_up_schedule, lab_schedule, alert_thresholds)

  Bao gồm:
  - Problem List [CMS]
  - Patient Preferences [NCQA]
  - Medications, Goals, Lifestyle plan, Follow-up schedule

⚠️ Thiếu thành phần → highlight, không cho xuất kế hoạch

---

### Bước 8: MD VN approve

- **Đối tượng:** MD VN
- **Mô tả:** Review và phê duyệt Care Plan (SLA: 1h)
- **INPUT:** care_plan_id (UUID, required)
- **OUTPUT:** approval_status (Enum: Approve/Adjust/Reject), md_vn_signature (String)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Approve | Chuyển bước MD US |
| Adjust | AI tạo lại Care Plan theo chỉnh sửa |
| Reject | Quay lại đánh giá |

**WIREFRAME:**

```
┌─────────────────────────────────────────────────────────────┐
│ ← Trang chủ                 KẾ HOẠCH CHĂM SÓC               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📋 KẾ HOẠCH CHĂM SÓC CÁ NHÂN                              │
│     v1.0 | ✅ Đã phê duyệt bởi Dr. Nguyen Van A            │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  💊 THUỐC ĐIỀU TRỊ                              [Chi tiết >]│
│  ├─ Metformin 1000mg - 2 lần/ngày                          │
│  ├─ Lisinopril 20mg - 1 lần/ngày                           │
│  └─ Atorvastatin 20mg - 1 lần/ngày                         │
│                                                             │
│  🎯 MỤC TIÊU                                    [Chi tiết >]│
│  ├─ HbA1c: <7.0% (hiện tại: 7.8%)                          │
│  └─ Huyết áp: <130/80                                      │
│                                                             │
│  🥗 LỐI SỐNG                                    [Chi tiết >]│
│  ├─ Chế độ ăn: DASH diet, 1800 kcal/ngày                   │
│  └─ Vận động: 150 phút/tuần                                │
│                                                             │
│  📅 LỊCH THEO DÕI                               [Chi tiết >]│
│  ├─ XN HbA1c: Mỗi 3 tháng                                  │
│  └─ Khám mắt: Hàng năm                                     │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │           ✅ TÔI ĐÃ ĐỌC VÀ HIỂU KẾ HOẠCH             │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Bước 9: MD US Final approval

- **Đối tượng:** MD US
- **Mô tả:** Phê duyệt cuối với digital signature (SLA: 1h giờ làm việc US)
- **INPUT:** care_plan_id (UUID, required), md_vn_approval (Object, required)
- **OUTPUT:** md_us_signature (String), care_plan_status (Enum: Approved/Modified/Rejected)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Approve | Chuyển Patient Acknowledgment |
| Modify | Alert CS và MD VN điều chỉnh |
| Reject | Alert CS và MD VN tạo đề xuất mới |

---

### Bước 10: Patient Acknowledgment (BN)

- **Đối tượng:** BN
- **Mô tả:** Xác nhận đã đọc Care Plan
- **INPUT:** care_plan_id (UUID, required), patient_id (UUID, required)
- **OUTPUT:** acknowledgment_status (Boolean), acknowledged_at (DateTime)

---

### Bước 11: Kích hoạt 4 Chương trình (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Kích hoạt 4 Programs (SLA: 15s): P1 Quản lý Thuốc (HP-02), P2 Quản lý Lối sống (HP-03), P3 Theo dõi Xét nghiệm (HP-04), P4 Theo dõi Biến chứng (HP-05/HP-06)
- **INPUT:** care_plan_id (UUID, required)
- **OUTPUT:** p1_status (ACTIVE), p2_status (ACTIVE), p3_status (STANDBY), p4_status (STANDBY)

⚠️ Chương trình nào fail → retry riêng lẻ 3 lần. Không kích hoạt được → gắn cờ lỗi

---

### Bước 12: Gửi email Welcome CDM + Care Plan approved (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Gửi email Welcome CDM + Care Plan approved cho BN
- **INPUT:** patient_id (UUID, required), care_plan_id (UUID, required)
- **OUTPUT:** email_delivery_status (Enum: Sent/Failed)

---

## HP-02: CHU TRÌNH CẬP NHẬT ĐIỀU TRỊ (MEDICATION)

> **Tổng quan:** Nhắc thuốc → BN xác nhận → AI phân tích → Đánh giá hiệu quả → Điều chỉnh nếu cần

```
CDM_06 → CDM_07 → CDM_08 → CDM_09 → CDM_10 → CDM_11 → CDM_12 → [Loop]
```

### Bước 1: Gửi nhắc thuốc (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Push + SMS theo lịch Care Plan, đúng giờ (±2 phút)
- **INPUT:** medication_id (UUID, required), scheduled_time (DateTime, required)
- **OUTPUT:** reminder_id (UUID), delivery_status (Enum: Delivered/Failed)

**UI States:**
- Push Notification: Hiển thị tên thuốc, liều lượng, hướng dẫn
- Actions: [Đã uống] [Nhắc lại 15p] [Bỏ qua]

**WIREFRAME (Push Notification):**

```
┌─────────────────────────────────────────────────────────────┐
│ CVH                                           now           │
│ 💊 Đến giờ uống thuốc                                       │
│ Metformin 1000mg - Uống với bữa sáng                        │
│    [Đã uống]    [Nhắc lại 15p]    [Bỏ qua]                 │
└─────────────────────────────────────────────────────────────┘
```

**WIREFRAME (In-App):**

```
┌─────────────────────────────────────────────────────────────┐
│ ← Quay lại                    NHẮC THUỐC                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    💊                                       │
│              ĐẾN GIỜ UỐNG THUỐC                             │
│                  08:00 SA                                   │
│                                                             │
│  THUỐC CẦN UỐNG:                                           │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 💊 Metformin 1000mg                                   │  │
│  │    Uống với bữa sáng                                  │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 💊 Lisinopril 20mg                                    │  │
│  │    Uống buổi sáng                                     │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              ✅ ĐÃ UỐNG TẤT CẢ                        │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  [⏰ Nhắc lại 15p]              [❌ Bỏ qua hôm nay]        │
│                                                             │
│  ⚠️ Có tác dụng phụ? [Báo cáo]                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

⚠️ Gửi fail → retry 3 lần qua kênh dự phòng (app fail → SMS, SMS fail → In-App)

---

### Bước 2: BN xác nhận (BN)

- **Đối tượng:** BN
- **Mô tả:** Đã uống / Bỏ qua
- **INPUT:** reminder_id (UUID, required), action (Enum: TAKEN/SNOOZED/SKIPPED, required)
- **OUTPUT:** adherence_record (Object — timestamp, status, notes)

⚠️ Bỏ liều → tự động gắn cờ và đưa vào luồng Phân tích Tuân thủ

---

### Bước 3: Phân tích tuân thủ (AI)

- **Đối tượng:** AI
- **Mô tả:** Tính adherence rate, phát hiện patterns bỏ liều (SLA: 15s)
- **INPUT:** patient_id (UUID, required), date_range (Object, required)
- **OUTPUT:** adherence_analysis (Object — adherence_rate, missed_doses, patterns, trend)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Adherence ≥ 90% | Tiếp tục theo dõi |
| > 2 liều liên tiếp bỏ | Escalate CS |

⚠️ > 2 liều liên tiếp bỏ → CS gọi BN, tìm lý do, phân loại (logistics vs y khoa)

---

### Bước 4: Đánh giá hiệu quả điều trị (AI)

- **Đối tượng:** AI
- **Mô tả:** Đánh giá dựa trên vitals/labs (SLA: 20s). Phân loại: Đạt mục tiêu / Tiến triển tốt / Không cải thiện / Xấu đi
- **INPUT:** patient_id (UUID, required), vitals (Object, required), labs (Object, required)
- **OUTPUT:** effectiveness_report (Object — classification, comparison_with_target, details)

⚠️ "Không cải thiện" hoặc "Xấu đi" → **MD can thiệp** review

---

### Bước 5: Phát hiện tác dụng phụ / tương tác thuốc (AI)

- **Đối tượng:** AI
- **Mô tả:** Phân tích triệu chứng tự báo cáo, chỉ số bất thường, drug-drug interaction (SLA: 15s). Phân loại: Nhẹ / Trung bình / Nặng / Nguy hiểm
- **INPUT:** patient_id (UUID, required), symptoms (Array, required)
- **OUTPUT:** side_effects (Object — severity, type, drug_interactions)

⚠️ Nghiêm trọng/Nguy hiểm → **MD can thiệp** ngay. Escalate khẩn cấp sang Service 1 (Access_to_Care_247) nếu tình trạng nguy hiểm

---

### Bước 6: Tạo đề xuất điều chỉnh (AI)

- **Đối tượng:** AI
- **Mô tả:** Tạo recommendations bao gồm: lý do điều chỉnh, phác đồ mới, dự kiến hiệu quả, rủi ro cần theo dõi (SLA: 20s)
- **INPUT:** assessment (Object, required), side_effects (Object, optional)
- **OUTPUT:** recommendations (Object — reason, new_regimen, expected_outcome, risks)

---

### Bước 7: MD VN Review đề xuất (MD VN)

- **Đối tượng:** MD VN
- **Mô tả:** Review và phê duyệt/điều chỉnh đề xuất (SLA: 1h)
- **INPUT:** recommendations (Object, required)
- **OUTPUT:** md_vn_decision (Enum: Approve/Adjust/Reject), md_vn_signature (String)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Approve | Chuyển MD US phê duyệt |
| Adjust | Ghi rõ lý do, chuyển MD US |
| Reject | Ghi rõ lý do, cảnh báo CS follow-up |

---

### Bước 8: MD US Phê duyệt cuối (MD US)

- **Đối tượng:** MD US
- **Mô tả:** Phê duyệt cuối với digital signature (SLA: 1h giờ US). Đơn thuốc chỉ có hiệu lực khi có chữ ký MD US
- **INPUT:** recommendations (Object, required), md_vn_approval (Object, required)
- **OUTPUT:** md_us_signature (String), prescription_status (Enum: Active/Modified/Rejected)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Approve | Kích hoạt đơn thuốc + thông báo BN |
| Modify | Alert CS + MD VN điều chỉnh |
| Reject | Alert CS + MD VN tạo đề xuất mới |

---

### Bước 9: Cập nhật Care Plan + thông báo BN (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Cập nhật phác đồ mới, lịch theo dõi, ngưỡng cảnh báo vào Care Plan và EHR. Thông báo BN (SLA: 20s)
- **INPUT:** recommendations (Object, required), approval (Object, required)
- **OUTPUT:** updated_care_plan (Object), notification_status (Enum: Sent/Failed)

⚠️ Lỗi đồng bộ EHR → retry 3 lần. Gửi thông báo fail → alert CS

---

## HP-03: CHU TRÌNH CẬP NHẬT LỐI SỐNG (LIFESTYLE)

> **Tổng quan:** BN nhập dữ liệu thô → AI phân tích → Tính điểm → AI coaching → CS can thiệp nếu cần

```
CDM_13A → CDM_13B → CDM_13C → CDM_13D → CDM_13E → CDM_13F → CDM_13G → CDM_14 → [Score ≥60%? → CDM_15] / [<60% → CDM_16] → CDM_17 (monthly)
```

> ⚠️ **LƯU Ý QUAN TRỌNG:**
> - BN chỉ nhập **dữ liệu thô** (bữa ăn, hoạt động, giờ ngủ)
> - **AI phân tích** để tính toán metrics (calories, nutrients, sleep quality...)
> - BN không thể tự biết lượng calo, vitamin trong bữa ăn

### Thu thập dữ liệu hàng ngày

#### Bước 1: Gửi nhắc ghi nhật ký lối sống hàng ngày (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Push notification đúng giờ (±2 phút), nhắc ghi 7 mục: ăn uống, vận động, cân nặng, huyết áp, đường huyết, giấc ngủ, tâm trạng
- **INPUT:** patient_id (UUID, required), schedule (Object, required)
- **OUTPUT:** reminder_status (Enum: Sent/Failed)

---

#### Bước 2: Ghi bữa ăn (BN)

- **Đối tượng:** BN
- **Mô tả:** Nhập món ăn, khẩu phần (text/ảnh). **AI Vision** nhận diện thực phẩm từ ảnh
- **INPUT:** meals (Array, required — meal_type, description), meal_photos (Array, optional), water_liters (Number, optional)
- **OUTPUT:** raw_entry_id (UUID)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| meals | ≥1 bữa required | Vui lòng nhập ít nhất 1 bữa ăn |
| meal_type | Enum: breakfast/lunch/dinner/snack | Vui lòng chọn loại bữa ăn |

**UI States:**
- Default: Form trống, gợi ý "Ghi bữa ăn của bạn hôm nay"
- Typing: Hiển thị nội dung đang nhập
- Photo: Camera mở → AI Vision nhận diện → điền sẵn tên món
- Idle > 60s: Popup "Bạn đã ăn chưa? Ghi lại ngay nhé!"
- Success: Push "✅ Bữa [sáng/trưa/tối] đã được ghi lại!"

---

#### Bước 3: AI Phân tích dinh dưỡng (AI)

- **Đối tượng:** AI
- **Mô tả:** Tính calo, carbs, protein, chất béo, vitamins từ bữa ăn. So sánh với Daily Target trong Care Plan (SLA: 30s, sai số ≤ 10%)
- **INPUT:** raw_entry_id (UUID, required)
- **OUTPUT:** calories (Number), carbs_g (Number), protein_g (Number), fat_g (Number), fiber_g (Number), vitamins (Object), nutrition_score (Number 0-100)

⚠️ API lỗi → Retry 2 lần → Ghi nhận "Chưa phân tích" để xử lý batch sau

---

#### Bước 4: Ghi vận động (BN/Device)

- **Đối tượng:** BN/Device
- **Mô tả:** Nhập loại hoạt động, thời gian (thủ công hoặc sync từ Apple Health/Google Fit/Fitbit)
- **INPUT:** activity_type (String, required), activity_duration (Number, required — phút), steps (Number, optional — từ wearable)
- **OUTPUT:** raw_entry_id (UUID)

**UI States:**
- Wearable connected: Auto sync mỗi 5 phút khi App mở
- Wearable disconnected: Form nhập tay tự động hiện
- Wearable not synced > 24h: Push "Kiểm tra kết nối thiết bị đo của bạn"
- Duplicate data (wearable + nhập tay): Auto chọn nguồn wearable

---

#### Bước 5: AI Phân tích vận động (AI)

- **Đối tượng:** AI
- **Mô tả:** Tính calo đốt (MET values), quãng đường, nhịp tim TB, kiểm tra HR an toàn (50-85% max HR) (SLA: 30s)
- **INPUT:** raw_entry_id (UUID, required)
- **OUTPUT:** calories_burned (Number), distance_km (Number), avg_heart_rate (Number), active_minutes (Number), exercise_score (Number 0-100)

⚠️ Nhịp tim > 85% max HR → Alert ngay: "⚠️ Nhịp tim cao bất thường. Nghỉ ngơi ngay và theo dõi."
⚠️ Nhịp tim bất thường liên tục > 90% max HR → MD VN review

---

#### Bước 6: Ghi giấc ngủ (BN/Device)

- **Đối tượng:** BN/Device
- **Mô tả:** Nhập giờ đi ngủ, thức dậy (thủ công hoặc wearable sync)
- **INPUT:** bed_time (Time, required), wake_time (Time, required)
- **OUTPUT:** raw_entry_id (UUID)

**UI States:**
- Wearable: Auto pull dữ liệu giấc ngủ mỗi sáng 7:00
- Không wearable: Push nhắc lúc 8:00 "🌅 Chào buổi sáng! Bạn ngủ lúc mấy giờ tối qua?"
- Đến 10:00 chưa ghi: Push lần 2
- Dữ liệu bất thường (ngủ >16h hoặc <2h): Gắn flag để AI xác minh

---

#### Bước 7: AI Phân tích giấc ngủ (AI)

- **Đối tượng:** AI
- **Mô tả:** Tính thời lượng, % giấc ngủ sâu, Sleep Score (SLA: 30s)
- **INPUT:** raw_entry_id (UUID, required)
- **OUTPUT:** sleep_duration (Number — giờ), deep_sleep_pct (Number), rem_sleep_pct (Number), sleep_score (Number 0-100)

⚠️ Sleep Score liên tục < 50 trong 7 ngày → MD VN review điều chỉnh khuyến nghị giấc ngủ

---

#### Bước 8: Check-in tâm trạng hàng ngày (BN)

- **Đối tượng:** BN
- **Mô tả:** Nhập tâm trạng (1-5), mức năng lượng (1-10) (trigger 20:00)
- **INPUT:** mood_level (Enum 1-5, required), energy_level (Number 1-10, required)
- **OUTPUT:** raw_entry_id (UUID)

**UI States:**
- Nhắc: Push lúc 20:00 → "🧘 Tâm trạng của bạn hôm nay thế nào?"
- Chưa check-in 22:00: Push lần 2
- Tâm trạng tốt: "Tuyệt vời! Giữ tinh thần lạc quan nhé! 🌟"
- Tâm trạng kém: "Hôm nay có vẻ khó khăn. Hãy thử bài thở 3 phút 🧘"
- Chuỗi tâm trạng tiêu cực ≥ 3 ngày: Gắn flag cảnh báo

⚠️ Tâm trạng tiêu cực cực độ + dấu hiệu khủng hoảng → CS alert ngay, liên hệ BN
⚠️ Xu hướng tiêu cực > 7 ngày → MD VN review (có thể liên quan tác dụng phụ thuốc)

**WIREFRAME (Nhập lối sống — Input thô):**

```
┌─────────────────────────────────────────────────────────────┐
│ ← Quay lại              GHI NHẬN LỐI SỐNG                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📅 Ngày: 21/01/2026                                       │
│                                                             │
│  🍎 DINH DƯỠNG                                   [Nhập >]  │
│     ┌─────────────────────────────────────────────────┐    │
│     │ Bữa sáng: Phở bò (1 tô), cà phê sữa            │    │
│     │ Bữa trưa: Cơm gà (1 đĩa), canh rau             │    │
│     │ Bữa tối: Chưa nhập                              │    │
│     │ 📷 Ảnh bữa ăn: 2 ảnh                           │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  🏃 VẬN ĐỘNG                                     [Nhập >]  │
│     ┌─────────────────────────────────────────────────┐    │
│     │ ⌚ Sync từ Apple Watch (06:35)                  │    │
│     │ Bước chân: 8,500 bước                          │    │
│     │ Thời gian vận động: 50 phút                    │    │
│     │ Loại: Đi bộ nhanh                              │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  😴 GIẤC NGỦ                                     [Nhập >]  │
│     ┌─────────────────────────────────────────────────┐    │
│     │ Đi ngủ: 22:30 | Thức dậy: 06:30                │    │
│     │ (Sync từ Apple Watch)                          │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  😊 TÂM TRẠNG                                    [Nhập >]  │
│     ┌─────────────────────────────────────────────────┐    │
│     │ Tâm trạng: 😊 Tích cực                         │    │
│     │ Năng lượng: 7/10                               │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  ⌚ Sync: Apple Watch (06:35)               [Sync lại]     │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │         🤖 GỬI CHO AI PHÂN TÍCH                      │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Tổng hợp & Coaching

#### Bước 9: AI Tổng hợp 4 lĩnh vực lối sống (AI)

- **Đối tượng:** AI
- **Mô tả:** Trigger 22:00 mỗi ngày (hoặc khi đủ 4 loại dữ liệu). Kết hợp điểm 4 lĩnh vực + phân tích tương quan (SLA: 2 phút)
- **INPUT:** raw_entry_id (UUID, required)
- **OUTPUT:** aggregated_data (Object — nutrition, exercise, sleep, mood summaries)

⚠️ Thiếu 1-2 loại dữ liệu → Tổng hợp với dữ liệu hiện có, ghi chú "Chưa đủ dữ liệu [loại]"
⚠️ Thiếu cả 4 → Bỏ qua ngày, không tạo báo cáo

---

#### Bước 10: Tính điểm Lối sống (AI)

- **Đối tượng:** AI
- **Mô tả:** Tính điểm weighted 0-100 (SLA: 30s):
  - Dinh dưỡng 25% + Vận động 30% + Giấc ngủ 25% + Tâm trạng 20%
- **INPUT:** entry_id (UUID, required)
- **OUTPUT:** nutrition_score (Number), exercise_score (Number), sleep_score (Number), mood_score (Number), overall_score (Number)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| ≥ 80: Excellent | Coaching khích lệ |
| 60-79: On target | Coaching duy trì |
| 40-59: Needs improvement | Trigger CS can thiệp (SLA: 4h) |
| < 40: High risk | Trigger CS + MD review |

**WIREFRAME (Kết quả AI phân tích):**

```
┌─────────────────────────────────────────────────────────────┐
│ ← Quay lại           KẾT QUẢ PHÂN TÍCH AI                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│        🤖 AI ĐÃ PHÂN TÍCH DỮ LIỆU CỦA BẠN                  │
│           📅 Ngày: 21/01/2026                               │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  🍎 DINH DƯỠNG                              Score: 72/100  │
│     ┌─────────────────────────────────────────────────┐    │
│     │ Calories:  1,750 kcal (Mục tiêu: 1,800)   ✅   │    │
│     │ Carbs:     210g (Mục tiêu: 180g)          ⚠️   │    │
│     │ Protein:   65g (Mục tiêu: 70g)            ✅   │    │
│     │ Fat:       58g (Mục tiêu: 60g)            ✅   │    │
│     │ Nước:      2.0L (Mục tiêu: 2.5L)          ⚠️   │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  🏃 VẬN ĐỘNG                                Score: 85/100  │
│     ┌─────────────────────────────────────────────────┐    │
│     │ Calories đốt:    230 kcal                       │    │
│     │ Quãng đường:     6.2 km                         │    │
│     │ Nhịp tim TB:     98 bpm                         │    │
│     │ Thời gian active: 50 phút (Mục tiêu: 30)  ✅   │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  😴 GIẤC NGỦ                                Score: 88/100  │
│     ┌─────────────────────────────────────────────────┐    │
│     │ Thời lượng:   8 giờ (Mục tiêu: 7-8h)      ✅   │    │
│     │ Deep sleep:   22% (Mục tiêu: ≥20%)        ✅   │    │
│     │ Sleep score:  85/100                            │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  😊 TÂM TRẠNG                               Score: 70/100  │
│     ┌─────────────────────────────────────────────────┐    │
│     │ Tâm trạng: Tích cực 😊                          │    │
│     │ Năng lượng: 7/10                                │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│        ⭐ ĐIỂM LỐI SỐNG TỔNG THỂ: 78/100                   │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │            💬 XEM AI COACHING                        │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

#### Bước 11: Kiểm tra ngưỡng (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Phân loại kết quả và trigger can thiệp
- **INPUT:** overall_score (Number, required)
- **OUTPUT:** classification (Enum), trigger_action (Enum)

⚠️ Điểm 40-59 → CS gọi BN, dùng Motivational Interviewing, đặt SMART goals (SLA: 4h)
⚠️ Điểm < 40 → CS + **MD review**
⚠️ Điểm < 60 liên tục ≥ 3 ngày → Flag để MD VN review monthly

---

#### Bước 12: AI Coaching cá nhân hóa (AI)

- **Đối tượng:** AI
- **Mô tả:** Tạo coaching từ 500+ templates, dựa trên: Điểm từng lĩnh vực + Xu hướng 7 ngày + Bệnh lý + SMART Goals (SLA: 2 phút)
- **INPUT:** scores (Object, required), history (Array, required)
- **OUTPUT:** coaching_message (String — điểm tốt, lĩnh vực cần cải thiện, mục tiêu ngày mai)

Gửi:
- Push tổng hợp cuối ngày (22:00-22:30)
- Email coaching hàng tuần (Chủ nhật 8:00)

**WIREFRAME (AI Coaching):**

```
┌─────────────────────────────────────────────────────────────┐
│ ← Quay lại                    AI COACHING                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│        🤖 COMPASS AI HEALTH COACH                          │
│        Điểm Lối sống Hôm nay: 78/100 ⭐                    │
│                                                             │
│  💬 NHẬN XÉT:                                              │
│  "Chào bạn! Hôm nay bạn đã làm rất tốt với 8,500 bước     │
│   và 50 phút vận động. Giấc ngủ 8 tiếng cũng tuyệt!       │
│                                                             │
│   Tuy nhiên, lượng carbs (210g) cao hơn mục tiêu (180g).  │
│   Ngày mai hãy thử giảm phần cơm ở bữa trưa nhé!"         │
│                                                             │
│  ✅ ĐIỂM TỐT:                                              │
│  • Vận động vượt mục tiêu                                  │
│  • Giấc ngủ đủ và chất lượng tốt                           │
│                                                             │
│  ⚠️ CẦN CẢI THIỆN:                                         │
│  • Giảm lượng carbs (-30g/ngày)                            │
│                                                             │
│  🎯 MỤC TIÊU NGÀY MAI:                                     │
│  • Calories: ≤1,800 kcal                                   │
│  • Carbs: ≤180g                                            │
│  • Bước chân: ≥8,000                                       │
│                                                             │
│  [👍 Hữu ích]              [👎 Không hữu ích]              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

#### Bước 13: Cập nhật EHR (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Lưu dữ liệu lối sống hàng ngày + báo cáo hàng tháng vào EHR (SLA: 2 phút)
- **INPUT:** 30_day_data (Object, required)
- **OUTPUT:** goal_adjustments (Object), monthly_report (Object)

⚠️ EHR write failed → Retry 3 lần (5s/lần). Lỗi → lưu queue xử lý sau. 0% mất dữ liệu
⚠️ Xu hướng điểm giảm liên tục > 2 tuần → cảnh báo MD

---

## HP-04: CHU TRÌNH XỬ LÝ KẾT QUẢ XÉT NGHIỆM (LAB)

> **Tổng quan:** AI kiểm tra lịch → Tạo phiếu → Nhắc BN → Phân tích kết quả → Điều chỉnh

```
CDM_18 → CDM_19 → CDM_20 → CDM_21 → [Loop]
```

### Bước 1: Kiểm tra lịch XN hàng ngày (AI)

- **Đối tượng:** AI
- **Mô tả:** Check Care Plan schedule hàng ngày lúc 00:00 (±5 phút). Lọc XN đến hạn (7 ngày tới), quá hạn. Đánh dấu quá hạn > 14 ngày
- **INPUT:** care_plan_id (UUID, required), schedule (Object, required)
- **OUTPUT:** lab_orders_due (Array), overdue_labs (Array)

⚠️ XN quá hạn > 14 ngày → cảnh báo CS follow-up BN

---

### Bước 2: Tạo Phiếu XN (AI)

- **Đối tượng:** AI
- **Mô tả:** Tạo phiếu chỉ định XN đầy đủ: thông tin BN, chẩn đoán, XN theo mã LOINC, chỉ định lâm sàng, tên MD VN, QR/barcode (SLA: 10s). Status = Draft, chờ MD US phê duyệt
- **INPUT:** lab_order_id (UUID, required)
- **OUTPUT:** lab_order (Object — patient_info, diagnosis, tests_loinc, clinical_indication, md_name, qr_code), status (Draft)

---

### Bước 3: MD US Phê duyệt phiếu XN (MD US)

- **Đối tượng:** MD US
- **Mô tả:** Phê duyệt chỉ định XN với digital signature (SLA: 1h giờ US). Phiếu XN chỉ có hiệu lực khi có chữ ký MD US
- **INPUT:** lab_order_id (UUID, required)
- **OUTPUT:** md_us_signature (String), lab_order_status (Enum: Approved/Modified/Rejected)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Approve | Kích hoạt phiếu + gửi nhắc BN |
| Modify | Alert CS + MD VN điều chỉnh |
| Reject | Alert CS + MD VN tạo lại |

---

### Bước 4: Gửi nhắc BN đi XN (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Gửi nhắc qua app + email (3-5 ngày trước hẹn, trong 15 phút sau phê duyệt): PDF phiếu XN, hướng dẫn chuẩn bị (nhịn ăn...), gợi ý lab gần nhất, link đặt lịch
- **INPUT:** lab_order_id (UUID, required), patient_id (UUID, required)
- **OUTPUT:** reminder_status (Enum: Sent/Failed)

⚠️ Gửi fail cả 2 kênh sau 3 retry → alert CS gọi điện thông báo BN

---

### Bước 5: BN đi xét nghiệm

- **Đối tượng:** BN
- **Mô tả:** BN đến lab thực hiện XN. Hệ thống ghi nhận check-in (thời gian, địa điểm, XN đã làm), cập nhật status = Completed
- **INPUT:** lab_order_id (UUID, required)
- **OUTPUT:** completion_status (Enum: Completed/Pending)

⚠️ Không đi sau 3 ngày → auto nhắc (app + SMS)
⚠️ Tổng 7 ngày không đi → escalate CS gọi BN, đặt lại lịch (+7-14 ngày)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| BN đi XN | Chờ kết quả |
| 3 ngày chưa đi | Auto reminder |
| 7 ngày chưa đi | CS gọi + reschedule |

---

### Bước 6: Nhận kết quả qua HL7/FHIR (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Nhận kết quả từ lab, validate format HL7/FHIR, verify Patient ID, kiểm tra mã hóa
- **INPUT:** lab_order_id (UUID, required), lab_results (Object — HL7/FHIR format)
- **OUTPUT:** validated_results (Object), validation_status (Enum: Valid/Invalid)

⚠️ Format sai → gắn cờ lỗi, alert CS follow-up với lab
⚠️ Patient ID không khớp → từ chối nhận, alert DevOps
⚠️ Thiếu mã hóa → từ chối, cảnh báo bảo mật

---

### Bước 7: Nhập kết quả vào EHR (AI)

- **Đối tượng:** AI
- **Mô tả:** Parse dữ liệu (tên XN, giá trị, đơn vị, reference range), map vào EHR fields (SLA: 30s)
- **INPUT:** lab_order_id (UUID, required), results (Object, required)
- **OUTPUT:** ehr_update_status (Enum: Success/Failed)

---

### Bước 8: So sánh với khoảng tham chiếu (AI)

- **Đối tượng:** AI
- **Mô tả:** So sánh từng chỉ số với reference range (theo tuổi/giới/bệnh). Phân loại: Normal/Borderline/Abnormal/Critical (SLA: 15s)
- **INPUT:** results (Object, required), patient_demographics (Object, required)
- **OUTPUT:** comparison_report (Object — classification per test, flags)

⛔ Critical → alert MD ngay lập tức → MD gọi BN, xử trí khẩn → có thể chuyển Service 1 (Access_to_Care_247)

---

### Bước 9: Đánh giá mức kiểm soát bệnh (AI)

- **Đối tượng:** AI
- **Mô tả:** Đánh giá tổng hợp dựa trên: kết quả XN hiện tại, lịch sử, mục tiêu điều trị. Phân loại: Kiểm soát tốt / Trung bình / Kiểm soát kém / Nguy hiểm (SLA: 45s)
- **INPUT:** results (Object, required), history (Array, required), targets (Object, required)
- **OUTPUT:** control_assessment (Object — classification, clinical_significance)

⚠️ "Kiểm soát kém" hoặc "Nguy hiểm" → alert **MD** review + đề xuất thay đổi phác đồ

---

### Bước 10: Phân tích xu hướng 3-6-12 tháng (AI)

- **Đối tượng:** AI
- **Mô tả:** Trend lines cho HbA1c, LDL, BP. Phát hiện pattern bất thường (tăng/giảm đột ngột). Dự báo risk (SLA: 30s)
- **INPUT:** patient_id (UUID, required), date_range (Object, required)
- **OUTPUT:** trend_analysis (Object — trend_lines, patterns, risk_forecast)

⚠️ Xu hướng xấu đi → alert MD review

---

### Bước 11: Phân loại kết quả (AI)

- **Đối tượng:** AI
- **Mô tả:** Phân loại và quyết định hành động tiếp theo (SLA: 20s)
- **INPUT:** comparison_report (Object, required), control_assessment (Object, required)
- **OUTPUT:** final_classification (Enum), triggered_action (Enum)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Normal | Thông báo BN, tiếp tục theo dõi |
| Borderline | Coaching tăng cường lối sống + CS gọi BN |
| Abnormal | MD review, có thể điều chỉnh điều trị |
| Dangerous | MD can thiệp khẩn cấp, có thể chuyển Service 1 (Access_to_Care_247) |

**WIREFRAME (Kết quả XN):**

```
┌─────────────────────────────────────────────────────────────┐
│ ← Quay lại                  KẾT QUẢ XÉT NGHIỆM              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📅 Ngày XN: 15/01/2026                                    │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ HbA1c: 7.2%                          ⚠️ Cận biên      │  │
│  │ Mục tiêu: <7.0% | Trước: 7.8% (↓ Cải thiện)          │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Cholesterol: 185 mg/dL               ✅ Bình thường   │  │
│  │ Mục tiêu: <200 | Trước: 210 (↓ Cải thiện)            │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  💬 NHẬN XÉT CỦA BÁC SĨ:                                   │
│  "Kết quả HbA1c cải thiện từ 7.8% xuống 7.2%. Tiếp tục    │
│   duy trì chế độ hiện tại."                                │
│  - Dr. Nguyen Van A                                        │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              📥 TẢI PDF KẾT QUẢ                      │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## HP-05: CHUYÊN KHOA CHUYÊN GIA (SPECIALIST REFERRAL)

> **Tổng quan:** P3 = TRIGGER POINT → Service 2 (Specialist_Referral) → Nhận kết quả → Cập nhật Care Plan

```
[Trigger] → CDM_22 → SERVICE 2 (Specialist_Referral) → CDM_23
```

### Triggers kích hoạt

| Điều kiện kích hoạt | Ví dụ |
|----------------------|-------|
| AI phát hiện biến chứng | Bệnh võng mạc → Bác sĩ mắt |
| Kết quả XN bất thường | Protein niệu → Bác sĩ thận |
| Không đáp ứng điều trị | Cần ý kiến chuyên gia |
| Khuyến nghị theo protocol | DM >5 năm → Tầm soát tim mạch |

### Bước 1: Phát hiện điều kiện cần giới thiệu chuyên gia (AI)

- **Đối tượng:** AI
- **Mô tả:** Phát hiện từ dữ liệu lâm sàng (auto 95%). So sánh với guideline chuyển khoa chuẩn (SLA: 60s)
- **INPUT:** trigger (Object, required), referral_type (String, required)
- **OUTPUT:** referral_recommendation (Object — specialty, reason, urgency)

---

### Bước 2: MD VN xác nhận và chỉ định chuyên khoa

- **Đối tượng:** MD VN
- **Mô tả:** Review dữ liệu lâm sàng, xác nhận cần chuyển khoa, xác định chuyên khoa phù hợp
- **INPUT:** referral_recommendation (Object, required)
- **OUTPUT:** md_vn_confirmation (Boolean), specialty (String), referral_reason (String)

---

### Bước 3: Tạo referral case → Service 2 (Specialist_Referral) (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Tạo referral case, chuyển sang Service 2 (Specialist_Referral) với đầy đủ dữ liệu BN (Patient info, diagnosis, treatment history, lab results, reason, specialty, urgency), sync hồ sơ y tế (SLA: 30s)
- **INPUT:** patient_id (UUID, required), referral_data (Object, required)
- **OUTPUT:** referral_id (UUID), handoff_status (Enum: Success/Failed)

---

### Bước 4: Nhận kết quả chuyên gia (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Nhận kết quả từ Service 2 (Specialist_Referral) (SLA: 1-4 tuần): Specialist assessment, New diagnosis, Treatment recommendations, Follow-up plan, Medication changes. Parse và lưu vào EHR
- **INPUT:** referral_id (UUID, required), results (Object, required)
- **OUTPUT:** updated_ehr (Boolean), notification_status (Enum: Sent/Failed)

⚠️ Quá 4 tuần chưa có kết quả → alert CS follow-up

---

### Bước 5: MD tích hợp kết quả vào Care Plan

- **Đối tượng:** MD
- **Mô tả:** Review kết quả từ specialist, tích hợp recommendations vào treatment plan, điều chỉnh Care Plan
- **INPUT:** referral_id (UUID, required), specialist_results (Object, required)
- **OUTPUT:** updated_care_plan (Object)

---

## HP-06: SỰ KIỆN CẤP TÍNH (ACUTE EVENT)

> **Tổng quan:** P4 = SAFETY NET → Service 1 (Access_to_Care_247) → Nhận kết quả → Cập nhật Care Plan

```
[Trigger] → CDM_24 → SERVICE 1 (Access_to_Care_247) → CDM_25
```

### Triggers kích hoạt

| Điều kiện kích hoạt | Chi tiết |
|----------------------|----------|
| Hạ đường huyết nặng | Glucose < 54 mg/dL |
| Tăng HA cấp | BP > 180/120 |
| Đau ngực / Khó thở | Cần đánh giá khẩn cấp |
| Tác dụng phụ nghiêm trọng | Phản ứng cần can thiệp |
| Biến chứng cấp tính | DKA, HHS, stroke symptoms |

### Bước 1: Phát hiện sự kiện cấp tính (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Phát hiện trong 30s, phân loại Emergency/Urgent. CS + MD on-call được alert đồng thời
- **INPUT:** event_type (String, required), severity (Enum: Emergency/Urgent, required)
- **OUTPUT:** acute_event_id (UUID), urgency_level (Enum)

---

### Bước 2: Tạo emergency case → Service 1 (Access_to_Care_247) (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Tạo emergency case trong Service 1 (Access_to_Care_247) (24/7 Access to Care) với đầy đủ thông tin (Patient info, acute event, vitals, medications, history, urgency). Alert MD on-call ngay lập tức (SLA: 15s)
- **INPUT:** acute_event_id (UUID, required), patient_data (Object, required)
- **OUTPUT:** emergency_case_id (UUID), handoff_status (Enum: Success/Failed)

⚠️ MD on-call không phản hồi trong 2 phút → re-alert lần 2 + cảnh báo CS
⚠️ Mức Emergency → gửi hướng dẫn xử trí ban đầu cho BN qua app

---

### Bước 3: Nhận kết quả xử lý từ Service 1 (Access_to_Care_247) (Hệ thống)

- **Đối tượng:** Hệ thống
- **Mô tả:** Nhận kết quả: MD assessment, Immediate treatment, Disposition (ER/Hospitalized/Home), Medication changes, Follow-up instructions (SLA: 24h)
- **INPUT:** emergency_case_id (UUID, required)
- **OUTPUT:** resolution (Object), ehr_updated (Boolean)

⚠️ Quá 24 giờ chưa có kết quả → CS follow-up với team Service 1 (Access_to_Care_247)

---

### Bước 4: MD Review kết quả (MD)

- **Đối tượng:** MD
- **Mô tả:** Review kết quả xử lý, đánh giá cần điều chỉnh treatment plan không
- **INPUT:** resolution (Object, required)
- **OUTPUT:** adjustment_needed (Boolean)

---

### Bước 5: Đề xuất điều chỉnh Care Plan (AI)

- **Đối tượng:** AI
- **Mô tả:** Đề xuất điều chỉnh 5 yếu tố: (1) Risk assessment, (2) Monitoring frequency, (3) Treatment plan, (4) Ngưỡng cảnh báo, (5) Patient education. MD approve (SLA: 2h)
- **INPUT:** acute_event_id (UUID, required), resolution (Object, required)
- **OUTPUT:** updated_care_plan (Object), md_approval (Boolean)

---

## BẢNG THÔNG BÁO TỰ ĐỘNG (MỞ RỘNG VỚI TEMPLATE)

| Sự kiện | Kênh | Người nhận |
|---------|------|------------|
| Welcome CDM | Email | BN |
| Care Plan approved | Email + In-app | BN |
| Nhắc thuốc | Push + SMS | BN |
| Nhắc ghi nhật ký lối sống | Push | BN |
| AI Coaching score | Push + In-app | BN |
| Kết quả XN | Push + In-app | BN |
| Nhắc đi xét nghiệm (3-5 ngày trước) | App + Email | BN |
| Alert cấp tính | Push + SMS | BN + CS + MD on-call |
| > 2 liều thuốc bỏ liên tiếp | Alert nội bộ | CS |
| Điểm lối sống < 40 | Alert nội bộ | CS + MD |
| Kết quả XN Dangerous/Critical | Alert nội bộ | MD |
| AI phát hiện tác dụng phụ nghiêm trọng | Alert nội bộ | MD |

### Template chi tiết

📧 **Email chào mừng CDM:**
```
Subject: [CVH] Chào mừng đến Chương trình Quản lý Bệnh Mãn tính

Kính gửi {{patient_name}},

Chúc mừng bạn đã đăng ký thành công Chương trình CDM!

🏥 THÔNG TIN:
- Ngày đăng ký: {{enrollment_date}}
- Care Specialist: {{cs_name}}

📱 BƯỚC TIẾP THEO:
1. Mở app CVH xem Care Plan
2. Bật thông báo nhắc thuốc
3. Bắt đầu ghi lối sống hàng ngày

Trân trọng,
CVH Care Team
```

📧 **Email Care Plan đã duyệt:**
```
Subject: [CVH] Kế hoạch Chăm sóc đã sẵn sàng!

Kính gửi {{patient_name}},

Care Plan của bạn đã được bác sĩ phê duyệt.

📋 TÓM TẮT:
- Thuốc: {{medication_count}} loại
- Mục tiêu HbA1c: {{hba1c_target}}
- XN tiếp theo: {{next_lab_date}}

📱 Mở app CVH → Kế hoạch Chăm sóc

Trân trọng,
Dr. {{md_name}} & CVH Care Team
```

📱 **SMS Nhắc thuốc:**
```
[CVH] Nhắc thuốc: {{medication_name}} {{dosage}}. {{instruction}}. Xác nhận tại app.
```

📱 **Push Nhắc thuốc:**
```
Title: Đến giờ uống thuốc 💊
Body: {{medication_name}} {{dosage}} - {{instruction}}
Actions: [Đã uống] [Nhắc lại 15p] [Bỏ qua]
```

📱 **Push Nhắc ghi lối sống:**
```
Title: Đừng quên ghi lối sống hôm nay! 📝
Body: Chỉ mất 2 phút để AI coaching bạn hiệu quả hơn.
```

📱 **Push AI Coaching:**
```
Title: 🎯 Điểm Lối sống: {{score}}/100
Body: {{summary_message}}
```

📱 **Push Kết quả XN:**
```
Title: 📊 Kết quả Xét nghiệm đã có
Body: HbA1c: {{value}} - {{status}}. Xem chi tiết →
```

📱 **Push Cảnh báo Cấp tính:**
```
Title: 🚨 CẢNH BÁO Y TẾ
Body: {{alert_message}}. Liên hệ ngay nếu cần hỗ trợ.
```

---

## BẢNG GIÁM SÁT CS-MD ĐẦY ĐỦ VỚI KPI

### HP-01: Đăng ký & Đánh giá Ban đầu (5 trạm)

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Khởi tạo Hồ sơ | AI xử lý ≤ 20s, EHR đầy đủ | 🤖 100%: Truy vấn EHR, validate, retry 3 lần (5s/lần). Thiếu dữ liệu → gắn cờ "Incomplete". Vượt 20s → alert DevOps | ❌ Không can thiệp (mọi lỗi hệ thống tự xử lý) | ❌ Không can thiệp |
| Đánh giá Y khoa | AI ≤ 60s, đủ 5 yếu tố, theo guideline | 🤖 100%: Chạy AI model, so sánh guideline, validate logic. Vượt 60s → alert DevOps. Red flag → alert MD VN | ❌ Không can thiệp | ❌ Không can thiệp |
| Tạo Kế hoạch | AI ≤ 45s, đủ 4 thành phần, theo guideline | 🤖 100%: Tạo Care Plan, validate 4 thành phần. Thiếu → không cho xuất. Vượt 45s → alert DevOps | ❌ Không can thiệp | ❌ Không can thiệp |
| Lưu EHR | ≤ 5s, không mất dữ liệu, đúng format | 🤖 100%: Lưu EHR, verify toàn vẹn, backup audit. Lỗi → retry 3 lần (3s/lần), rollback nếu fail | ❌ Không can thiệp | ❌ Không can thiệp |
| Kích hoạt Chương trình | ≤ 15s, đủ 4 programs Active | 🤖 100%: Kích hoạt 4 programs, verify Active. Fail → retry riêng lẻ 3 lần. Vượt 15s → alert DevOps | ❌ Không can thiệp | ❌ Không can thiệp |

### HP-02: Chu trình Cập nhật Điều trị (10 trạm)

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Nhắc thuốc | Đúng giờ ±2 phút, đúng kênh/nội dung/BN | 🤖 100%: Gửi đúng giờ, validate nội dung, log. Fail → retry 3 lần qua kênh dự phòng | ⚠️ Fail hoàn toàn sau 3 retry → CS kiểm tra log, cập nhật kênh gửi | ❌ |
| Xác nhận BN | Ghi nhận đầy đủ (thời gian, trạng thái), lưu EHR real-time | 🤖 100%: Ghi nhận, validate, lưu EHR. Bỏ liều → gắn cờ. Lỗi → retry 3 lần | ⚠️ App crash hoàn toàn → CS báo DevOps | ❌ |
| Phân tích Tuân thủ | AI ≤ 15s, % tuân thủ chính xác, phát hiện >2 liều bỏ | 🤖 100%: Tính %, phát hiện pattern. >2 liều bỏ → alert CS | ❌ | ❌ |
| Escalate CS | Alert ≤ 15s khi >2 liều bỏ, đủ thông tin BN | 🤖 100%: Alert CS Dashboard (patient ID, liều bỏ, lịch sử). CS chưa xem 30p → re-alert | ✅ CS gọi BN, hỏi lý do (quên/tác dụng phụ/hết thuốc). Logistics → CS xử lý. Y tế → escalate MD | ❌ (trừ CS escalate vì lý do y tế) |
| Đánh giá Hiệu quả | AI ≤ 20s, đánh giá BP/glucose/HR/triệu chứng, so sánh với target | 🤖 100%: Thu thập chỉ số, so sánh target, phân loại. "Không cải thiện"/"Xấu đi" → alert MD | ❌ | ⚠️ MD review khi nhận alert "Không cải thiện"/"Xấu đi" → đề xuất điều chỉnh phác đồ |
| Phát hiện Tác dụng phụ | AI ≤ 15s, phân loại Nhẹ/TB/Nặng/Nguy hiểm, kiểm tra drug-drug interaction | 🤖 100%: Phân tích triệu chứng + chỉ số + drug interaction. Nặng/Nguy hiểm → alert MD khẩn | ❌ | ⚠️ MD review Nặng/Nguy hiểm → giảm liều/thay thuốc/stop. Nguy hiểm → escalate Service 1 (Access_to_Care_247) |
| Tạo Đề xuất | AI ≤ 20s, đủ: lý do, phác đồ mới, hiệu quả dự kiến, rủi ro | 🤖 100%: Tạo đề xuất theo guideline, kiểm tra contraindication. Xong → alert MD VN review 1h | ❌ | ✅ MD VN review đề xuất AI, kiểm tra phác đồ, chuyển MD US |
| MD VN Review | ≤ 1h, quyết định rõ (Approve/Adjust/Reject), chữ ký điện tử | 🤖 95%: Alert MD VN, theo dõi thời gian, block chuyển nếu chưa ký. Reject → alert CS | ⚠️ MD VN quá 1h → CS liên hệ nhắc | ✅ MD VN review clinical data, quyết định, ký điện tử |
| MD US Phê duyệt | ≤ 1h (giờ US), chữ ký điện tử, đơn thuốc chỉ active sau ký | 🤖 95%: Alert MD US, theo dõi thời gian, block đơn thuốc chưa ký. Reject → alert CS + MD VN | ⚠️ MD US quá 1h → CS follow-up khẩn | ✅ MD US final review, kiểm tra guideline US, ký điện tử (bắt buộc) |
| Cập nhật Kế hoạch | ≤ 20s, đồng bộ EHR, thông báo BN, audit trail | 🤖 100%: Cập nhật Care Plan, đồng bộ EHR, ghi audit. Fail → retry 3 lần. Thông báo BN qua app + email | ⚠️ Gửi thông báo fail → CS gọi BN thông báo thay đổi điều trị | ❌ |

### HP-03: Chu trình Cập nhật Lối sống (14 trạm)

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Nhắc ghi nhật ký | Đúng giờ ±2 phút, đủ 7 mục | 🤖 100%: Gửi đúng giờ, validate nội dung, log | ❌ | ❌ |
| Ghi Bữa ăn | 100% lưu thành công, ảnh ≤ 10s | 🤖 100%: AI Vision nhận diện, validate form, retry 3 lần | ⚠️ App crash > 2h → CS hỗ trợ ghi bù | ❌ |
| AI Phân tích Dinh dưỡng | ≤ 30s, sai số ≤ 10%, có nhận xét | 🤖 100%: Gọi API, tính calo/carbs/protein/fat, so sánh target | ⚠️ AI sai nghiêm trọng → CS báo kỹ thuật fix | ❌ |
| Ghi Vận động | 100% wearable sync, ≤ 5 phút | 🤖 100%: Sync Apple Health/Google Fit/Fitbit mỗi 5 phút. Duplicate → chọn wearable | ⚠️ Wearable không sync ≥ 24h → CS hỗ trợ kết nối | ❌ |
| AI Phân tích Vận động | ≤ 30s, so sánh target, cảnh báo HR | 🤖 100%: Tính calo đốt (MET), kiểm tra HR zone. HR > 85% max → alert BN | ❌ | ⚠️ HR bất thường liên tục > 90% max → MD VN review |
| Ghi Giấc ngủ | 100% ghi nhận, wearable sync ≤ 30 phút | 🤖 100%: Auto pull wearable 7:00. Không wearable → push 8:00. Bất thường → flag | ⚠️ Wearable không sync > 3 ngày → CS hỗ trợ reset | ❌ |
| AI Phân tích Giấc ngủ | ≤ 30s, Sleep Score chuẩn lâm sàng | 🤖 100%: Tính Sleep Score, so sánh target. <5h liên tiếp 3 ngày → flag | ❌ | ⚠️ Sleep Score < 50 liên tục 7 ngày → MD VN review |
| Check-in Tâm trạng | Nhắc 20:00, ≥ 70% tỷ lệ hoàn thành | 🤖 100%: Push 20:00 + 22:00. Gán tag. Tiêu cực ≥ 3 ngày → flag | ⚠️ Khủng hoảng tâm lý → CS liên hệ BN ngay | ⚠️ Tiêu cực > 7 ngày → MD VN review (tác dụng phụ thuốc?) |
| AI Tổng hợp | ≤ 2 phút, phân tích tương quan 4 lĩnh vực | 🤖 100%: Trigger 22:00. Thiếu 1-2 loại → tổng hợp có ghi chú. Thiếu cả 4 → bỏ qua | ❌ | ❌ |
| Tính Điểm Lối sống | ≤ 30s, 0-100 weighted, nhất quán | 🤖 100%: Dinh dưỡng 25% + Vận động 30% + Giấc ngủ 25% + Tâm trạng 20%. Biểu đồ 7/30 ngày | ❌ | ⚠️ Điểm < 40 → MD VN xem xét |
| Kiểm tra Ngưỡng | ≤ 5s, phân loại đúng, trigger đúng | 🤖 100%: ≥80 Excellent / 60-79 On target / 40-59 Trigger CS / <40 Trigger CS + MD | ❌ (can thiệp ở trạm tiếp) | ⚠️ Điểm < 40 → alert MD VN |
| Can thiệp CS | Alert ≤ 1 phút, CS liên hệ ≤ 4h | 🤖 95%: Alert CS Dashboard, pull lịch sử 7-30 ngày, gợi ý 3 coaching. Chưa contact 2h → nhắc. 4h → escalate Lead | ✅ CS gọi BN, Motivational Interviewing, đặt SMART Goal, ghi nhận, theo dõi sau 3 ngày | ⚠️ Rào cản do bệnh lý → MD VN điều chỉnh mục tiêu |
| Tạo Coaching | ≤ 2 phút, 500+ templates, không mâu thuẫn y khoa | 🤖 100%: Push 22:00-22:30, Email Chủ nhật 8:00. Theo dõi Open Rate | ⚠️ Coaching sai → CS rút và gửi đính chính 1h | ⚠️ MD VN kiểm tra định kỳ hàng tháng |
| Cập nhật EHR | ≤ 2 phút, 0% mất dữ liệu | 🤖 100%: Lưu raw + điểm + coaching. Retry 3 lần. Fail → queue. Báo cáo tháng ngày 1 | ❌ | ⚠️ MD review EHR trong Monthly Review |

### HP-04: Chu trình Xử lý Kết quả XN (13 trạm)

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Kiểm tra Lịch | Chạy đúng 00:00 (±5 phút), quét tất cả BN | 🤖 100%: Job hàng ngày, lọc XN 7 ngày tới + quá hạn. Quá hạn > 14 ngày → alert CS | ⚠️ XN quá hạn > 14 ngày → CS gọi nhắc BN | ❌ |
| Tạo Phiếu XN | ≤ 10s, đầy đủ thông tin + LOINC + QR | 🤖 100%: Tạo phiếu từ EHR, LOINC code, QR/barcode. Status = Draft → alert MD US | ❌ | ❌ (chờ MD US phê duyệt) |
| MD US Phê duyệt XN | ≤ 1h (giờ US), chữ ký điện tử | 🤖 95%: Alert MD US, theo dõi thời gian, block phiếu chưa ký. Reject → alert CS + MD VN | ⚠️ MD US quá 1h → CS follow-up khẩn | ✅ MD US review, kiểm tra guideline, ký điện tử (bắt buộc) |
| Gửi Nhắc BN | ≤ 15 phút sau duyệt, app + email, đính kèm PDF | 🤖 100%: Gửi 2 kênh, đính kèm phiếu + hướng dẫn + lab gần nhất + link đặt lịch | ⚠️ Fail cả 2 kênh → CS gọi BN thông báo | ❌ |
| BN đi XN | Ghi nhận check-in đầy đủ, status = Completed | 🤖 100%: Ghi nhận, cập nhật status. Chưa đi 3 ngày → auto reminder | ⚠️ App crash → CS hướng dẫn BN | ❌ |
| Reminder | Auto sau 3 ngày, app + SMS | 🤖 100%: Phát hiện chưa đi, gửi nhẹ nhàng. 7 ngày tổng → alert CS | ⚠️ 7 ngày chưa đi → CS gọi + đặt lại lịch | ❌ |
| Reschedule | ≤ 30s, +7-14 ngày, reset reminder | 🤖 100%: Tạo lịch mới, gửi thông báo, cập nhật Care Plan, reset bộ đếm, audit trail | ⚠️ BN không đồng ý lịch tự động → CS phối hợp tìm lịch phù h���p | ❌ |
| Lab gửi KQ | Lab gửi đúng SLA (24-72h), HL7/FHIR, mã hóa | 🤖 100%: Nhận, validate format/ID/mã hóa. Sai → gắn cờ. ID không khớp → từ chối | ⚠️ Lab trễ SLA → CS follow-up lab | ❌ |
| Nhập EHR | AI ≤ 30s, parse chính xác, map đúng field | 🤖 100%: Parse, map, validate, gắn Patient ID/timestamp. Error → gắn cờ, alert DevOps | ❌ | ❌ |
| So sánh | AI ≤ 15s, reference range đúng (tuổi/giới/bệnh), phân loại đúng | 🤖 100%: So sánh, phân loại Normal/Borderline/Abnormal/Critical. Critical → alert MD khẩn | ❌ | ⚠️ Critical → MD review ngay, gọi BN, xử trí khẩn, chuyển Service 1 (Access_to_Care_247) nếu nguy cấp |
| Đánh giá | AI ≤ 45s, tổng hợp 3 yếu tố (hiện tại/lịch sử/mục tiêu) | 🤖 100%: Phân loại Kiểm soát tốt/TB/Kém/Nguy hiểm. "Kém"/"Nguy hiểm" → alert MD | ❌ | ⚠️ MD review + đề xuất thay đổi phác đồ |
| Phân tích Xu hướng | AI ≤ 30s, trend 3-6-12 tháng, biểu đồ | 🤖 100%: Trend line HbA1c/LDL/BP, phát hiện pattern, dự báo risk. Xấu → alert MD | ❌ | ⚠️ Xu hướng xấu → MD phân tích nguyên nhân |
| Phân loại | AI ≤ 20s, đúng 4 mức, trigger đúng action | 🤖 100%: Normal → thông báo BN. Borderline → coaching + alert CS. Abnormal → alert MD. Dangerous → MD khẩn | ⚠️ Borderline → CS gọi BN tư vấn lối sống | ⚠️ Abnormal → MD review. Dangerous → MD can thiệp khẩn |

### HP-05: Chuyên khoa Chuyên gia (3 trạm)

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Điều kiện kích hoạt | AI ≤ 60s, đúng guideline, ghi rõ lý do + chuyên khoa | 🤖 95%: Phát hiện từ data lâm sàng, so sánh guideline, xác định specialty → alert MD VN | ❌ | ✅ MD VN review, xác nhận chuyển khoa, ghi lý do, trigger Service 2 (Specialist_Referral) |
| Chuyển Service 2 (Specialist_Referral) | ≤ 30s, đầy đủ data, sync medical records | 🤖 100%: Tạo referral case, sync records, lưu referral ID. Fail → retry + alert DevOps | ⚠️ BN có câu hỏi → CS giải thích + hỗ trợ đặt lịch | ❌ (MD VN đã quyết định) |
| Kết quả phản hồi | Nhận trong 1-4 tuần, đầy đủ, parse đúng EHR | 🤖 100%: Nhận, parse, lưu EHR. Quá 4 tuần → alert CS | ⚠️ Quá 4 tuần → CS follow-up specialist | ✅ MD review kết quả, tích hợp vào Care Plan |

### HP-06: Sự kiện Cấp tính (4 trạm)

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Sự kiện cấp tính | Phát hiện ≤ 30s, phân loại Emergency/Urgent | 🤖 100%: Monitoring liên tục, phân loại, ghi nhận đầy đủ. Emergency → hướng dẫn BN qua app. Alert MD on-call + CS đồng thời | ⚠️ CS gọi BN kiểm tra, hướng dẫn xử trí ban đầu, hỗ trợ gọi cấp cứu nếu MD chỉ định | ✅ MD on-call đánh giá, quyết định Home/ER/Hospital, hướng dẫn xử trí khẩn |
| Chuyển Service 1 (Access_to_Care_247) | ≤ 15s, đầy đủ info, alert MD on-call | 🤖 100%: Tạo emergency case, alert MD on-call ngay. Chưa respond 2 phút → re-alert + CS | ⚠️ CS theo dõi MD respond, hỗ trợ BN, phối hợp team Service 1 (Access_to_Care_247) | ❌ (MD on-call Service 1 (Access_to_Care_247) tiếp nhận) |
| Kết quả xử lý | Nhận trong 24h, đầy đủ, lưu EHR | 🤖 100%: Nhận, parse, lưu EHR, tạo follow-up tasks. Quá 24h → alert CS | ⚠️ CS follow-up Service 1 (Access_to_Care_247), đảm bảo BN hiểu hướng dẫn | ✅ MD review kết quả, đánh giá điều chỉnh treatment plan |
| Điều chỉnh Kế hoạch | ≤ 2h, đủ 5 yếu tố, audit trail | 🤖 95%: AI đề xuất 5 yếu tố, validate, lưu audit. Vượt 2h → alert CS nhắc MD | ⚠️ MD quá 2h → CS follow-up. BN hỏi → CS giải thích | ✅ MD approve điều chỉnh, ghi rationale, follow-up sát BN |

---

## DATA SPECIFICATION

### Input/Output theo Function

| Function | Input | Output | Validation |
|----------|-------|--------|------------|
| CDM_01 | patient_id, condition, consent | profile_id, status | Account active, consent=true |
| CDM_02 | profile_id | ehr_data | Valid profile |
| CDM_03 | profile_id, ehr_data | assessment_report | EHR complete |
| CDM_03B | profile_id, diagnoses | problem_list, ccm_eligible | ≥2 conditions |
| CDM_03C | profile_id | preferences | Patient consent |
| CDM_04 | assessment, problem_list, prefs | care_plan | All inputs complete |
| CDM_04B | care_plan_id | care_coordination | Care Plan approved |
| CDM_04C | care_plan_id | barriers, solutions | Care Plan approved |
| CDM_05 | care_plan_id | p1-p4 status | MD signed |
| CDM_06 | medication_id, scheduled_time | reminder_id, delivery_status | Profile active |
| CDM_07 | reminder_id, action | adherence_record | Valid reminder |
| CDM_08 | patient_id, date_range | adherence_analysis | ≥7 days data |
| CDM_09 | patient_id, vitals, labs | effectiveness_report | Recent data |
| CDM_10 | patient_id, symptoms | side_effects | Valid medications |
| CDM_11 | assessment, side_effects | recommendations | MD review |
| CDM_12 | recommendations, approval | updated_care_plan | MD approval |
| CDM_13 | lifestyle_data | entry_id, aggregated_data | ≥1 category |
| CDM_14 | entry_id | scores (0-100) | Data complete |
| CDM_15 | scores, history | coaching_message | Score calculated |
| CDM_16 | low_score_alert | cs_intervention | Score <60% |
| CDM_17 | 30_day_data | goal_adjustments | Sufficient data |
| CDM_18 | care_plan_id, schedule | lab_order_id | Schedule defined |
| CDM_19 | lab_order_id | completion_status | Order exists |
| CDM_20 | lab_order_id, results | analysis, classification | Results received |
| CDM_21 | analysis, approval | updated_care_plan | MD review |
| CDM_22 | trigger, referral_type | referral_id, handoff | MD approval |
| CDM_23 | referral_id, results | updated_care_plan | Results received |
| CDM_24 | event_type, severity | acute_event_id, handoff | Severity assessed |
| CDM_25 | acute_event_id, resolution | updated_care_plan | Resolution received |

### Core Data Models

**CDM Profile:**
```json
{
  "profile_id": "UUID",
  "patient_id": "UUID",
  "status": "INITIATING|ASSESSED|CARE_PLAN_READY|ACTIVE",
  "primary_condition": "ICD-10",
  "care_plan_id": "UUID",
  "programs": {
    "p1_treatment": "ACTIVE",
    "p2_lifestyle": "ACTIVE",
    "p3_referral": "STANDBY|ACTIVE|COMPLETED",
    "p4_acute": "STANDBY|ACTIVE|RESOLVED"
  }
}
```

**Medication Reminder:**
```json
{
  "reminder_id": "UUID",
  "medication_name": "String",
  "dosage": "String",
  "scheduled_time": "DateTime",
  "response": "TAKEN|SNOOZED|SKIPPED|NO_RESPONSE"
}
```

**Lifestyle Entry:**

> ⚠️ **Phân biệt rõ:** `raw_input` = BN nhập/wearable sync | `ai_analysis` = AI phân tích

```json
{
  "entry_id": "UUID",
  "entry_date": "Date",
  "nutrition": {
    "raw_input": {
      "meals": [
        { "meal_type": "breakfast", "description": "Phở bò 1 tô, cà phê sữa" },
        { "meal_type": "lunch", "description": "Cơm gà 1 đĩa, canh rau" }
      ],
      "photos": ["photo_url_1", "photo_url_2"],
      "water_liters": 2.0
    },
    "ai_analysis": {
      "calories": 1750,
      "carbs_g": 210,
      "protein_g": 65,
      "fat_g": 58,
      "fiber_g": 15,
      "vitamins": { "A": "80%", "C": "120%", "D": "40%" }
    }
  },
  "exercise": {
    "raw_input": {
      "activity_type": "walking",
      "duration_minutes": 50,
      "steps": 8500,
      "source": "apple_watch"
    },
    "ai_analysis": {
      "calories_burned": 230,
      "distance_km": 6.2,
      "avg_heart_rate": 98,
      "active_minutes": 50
    }
  },
  "sleep": {
    "raw_input": {
      "bed_time": "22:30",
      "wake_time": "06:30",
      "source": "apple_watch"
    },
    "ai_analysis": {
      "duration_hours": 8,
      "deep_sleep_pct": 22,
      "rem_sleep_pct": 25,
      "sleep_score": 85
    }
  },
  "mood": {
    "raw_input": {
      "mood_level": "positive",
      "energy_level": 7
    }
  },
  "scores": {
    "nutrition_score": 72,
    "exercise_score": 85,
    "sleep_score": 88,
    "mood_score": 70,
    "overall_score": 78
  },
  "analyzed_at": "DateTime"
}
```

### Tổng hợp Data Fields

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| profile_id | UUID | Auto | UUID v4 | ID hồ sơ CDM |
| patient_id | UUID | Yes | UUID v4 | ID bệnh nhân |
| primary_condition | String | Yes | ICD-10 code | Bệnh mãn tính chính |
| secondary_conditions | Array | No | ICD-10 codes | Bệnh đồng mắc |
| consent_cdm | Boolean | Yes | true | Đồng ý điều khoản CDM |
| consent_ai | Boolean | Yes | true | Cho phép AI phân tích |
| care_plan_id | UUID | Auto | UUID v4 | ID kế hoạch chăm sóc |
| care_plan_status | Enum | Auto | DRAFT/PENDING_APPROVAL/APPROVED/ACTIVE | Trạng thái Care Plan |
| medication_name | String | Yes | — | Tên thuốc |
| dosage | String | Yes | — | Liều lượng |
| scheduled_time | DateTime | Yes | ISO 8601 | Giờ uống thuốc |
| adherence_rate | Number | Auto | 0-100% | Tỷ lệ tuân thủ thuốc |
| meals | Array | Yes (≥1) | {meal_type, description} | Bữa ăn thô |
| meal_photos | Array | No | URL | Ảnh bữa ăn |
| activity_type | String | Yes | Enum | Loại vận động |
| activity_duration | Number | Yes | Phút | Thời gian vận động |
| steps | Number | No | Từ wearable | Số bước chân |
| bed_time | Time | Yes | HH:mm | Giờ đi ngủ |
| wake_time | Time | Yes | HH:mm | Giờ thức dậy |
| mood_level | Enum | Yes | 1-5 | Thang tâm trạng |
| energy_level | Number | Yes | 1-10 | Mức năng lượng |
| overall_score | Number | Auto | 0-100, weighted | Điểm lối sống tổng |
| nutrition_score | Number | Auto | 0-100 | Điểm dinh dưỡng |
| exercise_score | Number | Auto | 0-100 | Điểm vận động |
| sleep_score | Number | Auto | 0-100 | Điểm giấc ngủ |
| mood_score | Number | Auto | 0-100 | Điểm tâm trạng |
| lab_order_id | UUID | Auto | UUID v4 | ID phiếu XN |
| lab_tests | Array | Yes | LOINC codes | Danh sách XN |
| lab_results | Object | Auto | HL7/FHIR format | Kết quả XN |
| referral_id | UUID | Auto | UUID v4 | ID chuyển khoa |
| acute_event_id | UUID | Auto | UUID v4 | ID sự kiện cấp tính |
| urgency_level | Enum | Yes | Emergency/Urgent | Mức độ khẩn cấp |
| md_vn_signature | String | Yes (khi approve) | Digital signature | Chữ ký MD VN |
| md_us_signature | String | Yes (khi approve) | Digital signature | Chữ ký MD US (bắt buộc cho đơn thuốc/phiếu XN) |

---

## NGUỒN TÀI LIỆU GỐC

- Luồng nghiệp vụ: `docs/02-product/ba-specs/service-4-Chronic_Management/base/Service_6_Touchpoint_Flow_v1.0.md`
- Checklist CS & MD: `docs/02-product/ba-specs/service-4-Chronic_Management/checklist-cs-md/checklist-cs-md.md`
- Bản tóm tắt: `docs/02-product/tomtat-chitiet/tomtat/service-4-Chronic_Management-tomtat.md`
