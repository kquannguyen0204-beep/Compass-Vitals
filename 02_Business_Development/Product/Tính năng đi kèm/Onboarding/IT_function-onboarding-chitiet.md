# CHI TIẾT LUỒNG NGHIỆP VỤ – FUNCTION ONBOARDING: ONBOARDING (MEDICAL PROFILE & HEALTH OVERVIEW)

## Header

- **Function:** Onboarding (Medical Profile & Health Overview)
- **Version:** 5.1
- **Input:** Bệnh nhân mới, lần đăng nhập đầu tiên, chưa có hồ sơ y tế
- **Output:** Hồ sơ y tế hoàn chỉnh (4 giai đoạn, ~160-200 items + files) + AI-generated Health Overview
- **Điều kiện sử dụng:** Tài khoản CVH active, đã hoàn tất đăng ký (Function Registration), HIPAA đã consent ở Function Registration
- **Duration:** 30-45 phút (người khỏe mạnh) / 35-60 phút (bệnh nhân phức tạp)

---

## Nguyên tắc cốt lõi

- 2 chế độ thu thập dữ liệu: **AI Chatbox Mode** (lần đầu onboarding, Giai đoạn 1) và **Form Mode** (Giai đoạn 2-4 + chỉnh sửa sau onboarding)
- Auto-save mỗi 30 giây, Draft lưu 7 ngày
- Health Overview CHỈ tạo khi hoàn tất cả 4 Stage
- Nếu chưa hoàn tất → Progress Tracker (Partial Activated) + nhắc nhở (3, 7, 14 ngày)
- Nguyên tắc kiểm soát luồng: Mỗi node có INPUT/OUTPUT rõ ràng, điều kiện cụ thể, luồng đơn tuyến

---

## Danh sách luồng con

| Luồng | Mô tả |
|-------|-------|
| Onboarding lần đầu | Tutorial → 4 Stages → AI Analysis → Health Overview |
| Resume Draft | Tiếp tục từ bản nháp đã lưu (trong 7 ngày) |
| Edit Profile | Chỉnh sửa hồ sơ sau onboarding (Form Mode) |

---

## Phân nhánh trong Flow

| Phân nhánh | Mô tả |
|------------|-------|
| Xem Tutorial | First Login → Tutorial 2-3 phút → AI Chatbox (GĐ1) → GĐ 2-4 → AI Analysis → Health Overview |
| Skip Tutorial | First Login → Skip → AI Chatbox (GĐ1) → GĐ 2-4 → AI Analysis → Health Overview |
| Nam (Male) | 4 Giai đoạn (không có câu hỏi OBGYN) → AI Analysis → Health Overview |
| Nữ (Female) | 4 Giai đoạn (bao gồm OBGYN trong Family History) → AI Analysis → Health Overview |
| Có Upload tài liệu | GĐ4: Upload (PDF/JPG/DICOM) → AI OCR → Auto-fill → Validate → Health Overview |
| Không Upload | GĐ 1-3 → Validate → Progress Tracker (có thể upload sau) |
| Resume (nếu có draft) | Login lại → Resume draft (lưu trong 7 ngày) → Tiếp tục từ giai đoạn đã lưu |
| Chỉnh sửa Profile | Dashboard → Edit Profile → Form truyền thống → Save → Health Overview cập nhật |

---

## Sơ đồ luồng

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        FUNCTION ONBOARDING: ONBOARDING                                        │
│                   (Medical Profile & Health Overview)                                │
│                        4 GIAI ĐOẠN THU THẬP THÔNG TIN                                │
│                                                                                      │
│   ┌──────────────┐                                                                   │
│   │ First Login  │                                                                   │
│   │ (New Patient)│                                                                   │
│   └──────┬───────┘                                                                   │
│          │  (HIPAA đã consent ở Function Registration - Registration)                            │
│          ▼                                                                           │
│   ┌──────────────┐     Skip      ┌──────────────┐                                   │
│   │  Tutorial?   │ ────────────► │   Continue   │                                   │
│   └──────┬───────┘               └──────┬───────┘                                   │
│          │ Watch                        │                                            │
│          ▼                              │                                            │
│   ┌──────────────┐                      │                                            │
│   │ Tutorial     │                      │                                            │
│   │ (2-3 min)    │                      │                                            │
│   └──────┬───────┘                      │                                            │
│          └──────────────┬───────────────┘                                            │
│                         ▼                                                            │
│          ┌──────────────────────┐                                                    │
│          │   Check Draft?      │                                                    │
│          └──────────┬──────────┘                                                    │
│             Có draft │ Không draft                                                   │
│          ┌───────────┴───────────┐                                                   │
│          ▼                       ▼                                                   │
│   ┌──────────────┐       ┌──────────────┐                                           │
│   │Resume Session│       │  Giai đoạn 1 │                                           │
│   └──────┬───────┘       └──────┬───────┘                                           │
│          │                      │                                                    │
│          └──────────┬───────────┘                                                    │
│                     ▼                                                                │
│          ┌──────────────────┐                                                        │
│          │   Giai đoạn 2    │                                                        │
│          └────────┬─────────┘                                                        │
│                   ▼                                                                  │
│          ┌──────────────────┐                                                        │
│          │   Giai đoạn 3    │                                                        │
│          └────────┬─────────┘                                                        │
│            Có upload │ Không upload                                                   │
│          ┌───────────┴───────────┐                                                   │
│          ▼                       ▼                                                   │
│   ┌──────────────┐       ┌──────────────────┐                                       │
│   │  Giai đoạn 4 │       │ Progress Tracker │                                       │
│   └──────┬───────┘       │ (Partial)        │                                       │
│          │               └──────────────────┘                                       │
│          ▼                                                                           │
│   ┌──────────────────┐                                                               │
│   │   AI Analysis    │                                                               │
│   │   (20-30 sec)    │                                                               │
│   └────────┬─────────┘                                                               │
│            ▼                                                                         │
│   ┌──────────────────┐                                                               │
│   │ Health Overview  │                                                               │
│   └────────┬─────────┘                                                               │
│            ▼                                                                         │
│   ┌──────────────────┐                                                               │
│   │    COMPLETE      │                                                               │
│   │  (Fully Active)  │                                                               │
│   └──────────────────┘                                                               │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## LUỒNG CHÍNH: ONBOARDING LẦN ĐẦU

### Bước 1-2: Đăng nhập & Tutorial

#### Bước 1: First Login

- **Hệ thống** – Phát hiện first login, tạo session onboarding
- INPUT: user_id (String, required), session_token (String, required), is_first_login (Boolean, required)
- OUTPUT: onboarding_session_id (String), onboarding_ui_displayed (Boolean)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| First login thành công | Chuyển sang Tutorial |
| Lỗi xác thực | Auto retry 3 lần (cách 2 giây) |
| Retry failed | Hiển thị: "Đăng nhập thất bại. Vui lòng thử lại sau 5 phút hoặc kiểm tra kết nối mạng." |

⚠️ Auth DB sập > 10 phút → CS nhận alert, liên hệ khách thông báo sự cố

#### Bước 2: Tutorial

- **Khách hàng** – Xem Tutorial video giới thiệu (2-3 phút)
- INPUT: onboarding_session_id (String, required)
- OUTPUT: tutorial_status (String: "completed" | "skipped")

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Xem xong Tutorial | Ghi trạng thái "completed", chuyển Check Draft |
| Skip Tutorial | Ghi trạng thái "skipped", chuyển Check Draft |
| Video lỗi tải | Auto retry 3 lần |
| Retry video failed | Hiển thị: "Video tạm thời không khả dụng. Nhấn Bỏ qua để tiếp tục" |

⚠️ Màn hình Tutorial chặn toàn bộ flow, không thể thoát → CS nhận escalation alert, bypass qua kênh hỗ trợ

UI States:
- Default: Video player loading
- Playing: Video đang phát, nút Skip hiển thị
- Completed: Auto chuyển bước tiếp
- Error: Thông báo lỗi + nút "Bỏ qua"

---

### Bước 3: Kiểm tra Draft

- **Hệ thống** – Query draft theo user ID, lọc draft hết hạn (> 7 ngày)
- INPUT: user_id (String, required)
- OUTPUT: has_valid_draft (Boolean), draft_data (Object, optional), draft_stage (Number, optional)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Có draft hợp lệ (< 7 ngày) | Chuyển sang Resume Session |
| Không có draft | Bắt đầu Giai đoạn 1 |
| Draft hết hạn (> 7 ngày) | Xóa draft, bắt đầu Giai đoạn 1 |
| Truy vấn lỗi | Auto retry 3 lần → vẫn lỗi: mặc định Giai đoạn 1, ghi log |

⚠️ Draft DB sập > 10 phút → CS nhận alert, hỗ trợ khách bắt đầu lại từ Giai đoạn 1

---

### Stage 1: Basic Profile & Medical Essentials (5-7 phút, 15-20 items)

- **Chế độ:** AI Chatbox Mode (lần đầu) / Form Mode (chỉnh sửa)
- INPUT: user_id (String, required), pre_filled_data (Object: {full_name, gender, dob} từ Registration)
- OUTPUT: stage_1_data (Object), stage_1_completed (Boolean), draft_id (String)

#### Hai chế độ thu thập

| Chế độ | Bối cảnh | INPUT | OUTPUT | Giao diện |
|--------|----------|-------|--------|-----------|
| 🤖 AI Chatbox Mode | First Login (Onboarding) | User text messages | Structured medical data | Chatbox conversational |
| 📝 Form Mode | Edit Profile (sau onboarding) | Form fields | Updated medical data | Form truyền thống |

> Giai đoạn 2, 3, 4 vẫn sử dụng Form Mode trong cả hai bối cảnh.

#### AI Agent Behavior

| Khía cạnh | Quy định |
|-----------|----------|
| Ngôn ngữ | Tiếng Việt, lịch sự, chuyên nghiệp như bác sĩ |
| Hỏi từng phần | Tối đa 1-2 câu hỏi/lượt |
| Validation | Validate ngay khi user trả lời, hỏi lại nếu không rõ |
| Confirm | Confirm thông tin quan trọng (bệnh lý, thuốc, dị ứng) |
| Skip logic | Nếu user nói "không có" → skip các câu hỏi chi tiết |
| Context aware | Ghi nhớ giới tính để hỏi đúng (anh/chị) |

#### Wireframe AI Chatbox

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              Thu thập thông tin y tế                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ 🤖 AI: Xin chào anh Nguyễn Văn A! Tôi sẽ giúp anh │   │
│   │    hoàn thành hồ sơ y tế. Thông tin cơ bản của anh:│   │
│   │    • Họ tên: Nguyễn Văn A                          │   │
│   │    • Giới tính: Nam                                │   │
│   │    • Ngày sinh: 15/03/1989                         │   │
│   │    Thông tin này đúng không ạ?                      │   │
│   │                                                     │   │
│   │    [Đúng rồi ✓]  [Chỉnh sửa ✏️]                    │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ 📊 Tiến độ: ████░░░░░░ 20% (Đang: Thông tin cơ bản)│   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ 💬 Nhập tin nhắn...                           [Gửi]│   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### A. THÔNG TIN CƠ BẢN (3 items - Pre-fill từ Registration)

> ⚠️ 3 trường đã được thu thập từ Function Registration - Registration. AI Agent hiển thị thông tin đã có và yêu cầu KH xác nhận hoặc chỉnh sửa.

| Field | Type | Required | Format/Rule | Nguồn |
|-------|------|----------|-------------|-------|
| full_name | String | Yes | First Name + Last Name | Pre-fill từ Registration |
| gender | String | Yes | Nam / Nữ / Khác | Pre-fill từ Registration |
| dob | Date | Yes | MM/DD/YYYY | Pre-fill từ Registration |

Validation Rules:

| Field | Rule | Message lỗi |
|-------|------|-------------|
| full_name | Không rỗng, chỉ chữ và dấu cách | "Vui lòng nhập họ tên hợp lệ" |
| gender | Phải là Nam/Nữ/Khác | "Vui lòng chọn giới tính" |
| dob | Date hợp lệ, tuổi ≥ 18 | "Ứng dụng chỉ dành cho người từ 18 tuổi trở lên" |

#### B. PAST MEDICAL HISTORY (10-15 items)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| hypertension | Boolean | Yes | Có/Không |
| hypertension_year | Number | Conditional (nếu có) | Năm chẩn đoán |
| hypertension_treatment | Boolean | Conditional | Đang điều trị? |
| diabetes | Boolean | Yes | Có/Không |
| diabetes_type | String | Conditional | Type 1 / Type 2 |
| diabetes_year | Number | Conditional | Năm chẩn đoán |
| diabetes_treatment | Boolean | Conditional | Đang điều trị? |
| cardiovascular | Boolean | Yes | Có/Không |
| cardiovascular_type | String | Conditional | Loại bệnh |
| cardiovascular_year | Number | Conditional | Năm chẩn đoán |
| high_cholesterol | Boolean | Yes | Có/Không |
| cholesterol_medication | Boolean | Conditional | Đang dùng thuốc? |
| asthma_copd | Boolean | Yes | Có/Không |
| asthma_severity | String | Conditional | Nhẹ / Trung bình / Nặng |
| liver_disease | Boolean | Yes | Có/Không |
| liver_disease_type | String | Conditional | Loại bệnh gan |
| kidney_disease | Boolean | Yes | Có/Không |
| kidney_stage | String | Conditional | Giai đoạn |
| cancer | Boolean | Yes | Có/Không |
| cancer_type | String | Conditional | Loại ung thư |
| cancer_year | Number | Conditional | Năm chẩn đoán |
| cancer_status | String | Conditional | Đang điều trị / Thuyên giảm / Khỏi |
| autoimmune | Boolean | Yes | Có/Không |
| autoimmune_type | String | Conditional | Loại bệnh (Lupus, RA, Hashimoto, MS...) |
| mental_disorder | Boolean | Yes | Có/Không |
| mental_disorder_type | String | Conditional | Loại (Trầm cảm, Lo âu, ADHD...) |
| mental_disorder_treatment | Boolean | Conditional | Đang điều trị? |
| other_conditions | String | Optional | Bệnh khác |

#### C. MEDICATIONS (Thuốc đang sử dụng)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| medications | Array[Object] | Optional | Mỗi item: {name, dosage, frequency, reason} |
| medication_name | String | Yes (per item) | Tên thuốc |
| medication_dosage | String | Yes (per item) | Liều lượng |
| medication_frequency | String | Yes (per item) | Tần suất |
| medication_reason | String | Yes (per item) | Lý do sử dụng |
| using_sleep_medicine | Boolean | Yes | Đang dùng thuốc ngủ? |
| using_pain_medicine_regularly | Boolean | Yes | Đang dùng thuốc giảm đau thường xuyên? |

> Bao gồm thuốc kê đơn, không kê đơn (OTC), vitamin, thảo dược

#### D. ALLERGIES (Dị ứng)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| drug_allergies | Array[Object] | Optional | Mỗi item: {drug_name, reaction} |
| food_allergies | Array[Object] | Optional | Mỗi item: {food_name, reaction} |
| latex_allergy | Boolean | Yes | Có/Không |
| pollen_allergy | Boolean | Yes | Có/Không |
| dust_allergy | Boolean | Yes | Có/Không |
| other_allergies | String | Optional | Mô tả |

**Hệ thống** – Auto-save mỗi 30s, real-time validation, skip logic

⚠️ AI Chatbox sập > 10 phút → CS nhận alert, báo kỹ thuật

UI States (AI Chatbox):
- Default: Chatbox mở, AI gửi câu chào đầu tiên
- Typing: Hiển thị "AI đang soạn...", user input enabled
- Validating: AI xác nhận thông tin trước khi chuyển câu tiếp
- Error: Thông báo lỗi kết nối, nút retry
- Auto-saving: Indicator "Đã lưu" hiển thị mỗi 30s
- Completed: Thông báo hoàn tất Stage 1, chuyển Stage 2

Hoàn tất → chuyển Stage 2

---

### Stage 2: Extended Medical & Social History (18-21 phút, 72-85 items)

- INPUT: user_id (String, required), stage_1_data (Object, required)
- OUTPUT: stage_2_data (Object), stage_2_completed (Boolean)

#### A. SURGICAL HISTORY (Tiền sử phẫu thuật)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| has_surgery | Boolean | Yes | Chưa bao giờ / Có |
| surgeries | Array[Object] | Conditional | Mỗi item: {type, year, hospital, complications, notes} |
| surgery_type | String | Yes (per item) | Loại phẫu thuật |
| surgery_year | Number | Yes (per item) | Năm |
| surgery_hospital | String | Optional | Bệnh viện |
| surgery_complications | Boolean | Yes (per item) | Có biến chứng? |
| surgery_complication_desc | String | Conditional | Mô tả biến chứng |
| serious_post_surgery_complications | Boolean | Yes | Biến chứng nghiêm trọng sau phẫu thuật? |
| serious_complication_desc | String | Conditional | Mô tả |

#### B. FAMILY HISTORY (Tiền sử gia đình, 10 bệnh lý)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| family_cardiovascular | Boolean | Yes | Có/Không |
| family_cardiovascular_who | String | Conditional | Người nào |
| family_cardiovascular_age | Number | Conditional | Tuổi phát hiện. ⚠️ Lưu ý nếu < 55 (nam) / < 65 (nữ) |
| family_diabetes | Boolean | Yes | Có/Không |
| family_diabetes_who | String | Conditional | Người nào |
| family_diabetes_type | String | Conditional | Type 1 / Type 2 |
| family_hypertension | Boolean | Yes | Có/Không |
| family_hypertension_who | String | Conditional | Người nào |
| family_stroke | Boolean | Yes | Có/Không |
| family_stroke_who | String | Conditional | Người nào |
| family_stroke_age | Number | Conditional | Tuổi khi xảy ra |
| family_cancer | Boolean | Yes | Có/Không |
| family_cancer_who | String | Conditional | Người nào |
| family_cancer_type | String | Conditional | Loại ung thư |
| family_cancer_age | Number | Conditional | Tuổi phát hiện |
| family_alzheimer | Boolean | Yes | Có/Không |
| family_alzheimer_who | String | Conditional | Người nào |
| family_osteoporosis | Boolean | Yes | Có/Không |
| family_osteoporosis_who | String | Conditional | Người nào |
| family_autoimmune | Boolean | Yes | Có/Không |
| family_autoimmune_who | String | Conditional | Người nào |
| family_autoimmune_type | String | Conditional | Loại bệnh |
| family_mental | Boolean | Yes | Có/Không |
| family_mental_who | String | Conditional | Người nào |
| family_mental_type | String | Conditional | Loại |
| family_early_death | Boolean | Yes | Tử vong sớm (< 60 tuổi)? |
| family_early_death_who | String | Conditional | Người nào |
| family_early_death_cause | String | Conditional | Nguyên nhân |
| family_early_death_age | Number | Conditional | Tuổi |

#### C. PREVIOUS HOSPITALIZATIONS (Tiền sử nhập viện, 5 năm gần nhất)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| has_hospitalization | Boolean | Yes | Có/Không (không tính phẫu thuật nhỏ/ngoại trú) |
| hospitalizations | Array[Object] | Conditional | Mỗi item: {year, reason, duration, outcome} |

#### D. SOCIAL HISTORY (Lịch sử xã hội, 4 sections)

**D.1 Nghề Nghiệp & Môi Trường Làm Việc:**

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| occupation | String | Yes | Nghề nghiệp hiện tại |
| work_environment | String | Yes | Văn phòng / Ngoài trời / Công nghiệp / Y tế / Giáo dục / Xây dựng / Tại nhà / Khác |
| work_hours_per_week | String | Yes | < 40 / 40-50 / 50-60 / > 60 giờ |
| hazard_exposure | Boolean | Yes | Tiếp xúc hóa chất/độc hại/bức xạ/tiếng ồn? |
| hazard_type | String | Conditional | Loại gì |

**D.2 Thói Quen Hút Thuốc Lá:**

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| smoking_status | String | Yes | Không bao giờ / Đã bỏ / Đang hút |
| smoking_quit_year | Number | Conditional (đã bỏ) | Năm bỏ |
| smoking_years_before_quit | Number | Conditional (đã bỏ) | Số năm hút trước khi bỏ |
| cigarettes_per_day | Number | Conditional (đang/đã hút) | Số điếu/ngày |
| smoking_start_age | Number | Conditional | Tuổi bắt đầu |
| smoking_total_years | Number | Conditional | Tổng số năm |
| pack_years | Number | Auto-calculated | (Số điếu/ngày ÷ 20) × Số năm hút |
| quit_desire | String | Conditional (đang hút) | Sẵn sàng bỏ / Chưa sẵn sàng / Đã thử thất bại / Không ý định |

**D.3 Thói Quen Uống Rượu/Bia:**

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| alcohol_status | String | Yes | Không uống / Thỉnh thoảng / Thường xuyên |
| drinks_per_week | Number | Conditional | Số đồ uống chuẩn/tuần (1 lon bia 330ml = 1 ly rượu vang 150ml = 1 shot 45ml) |
| drink_type | Array[String] | Conditional | Bia / Rượu vang / Rượu mạnh / Rượu trắng / Khác |
| binge_drinking | String | Conditional | Không bao giờ / < 1 lần/tháng / 1-2 lần/tháng / > 2 lần/tháng |

**D.4 Sử Dụng Chất Kích Thích:**

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| substance_use | Array[String] | Yes | Không bao giờ / Marijuana / Opioid / Cocaine / Meth / Ecstasy / Heroin / Thuốc an thần / Khác |
| substance_frequency | String | Conditional | Thỉnh thoảng / Thường xuyên / Hàng ngày |
| quit_substance_desire | String | Conditional | Có / Không chắc / Không |

#### E. VITAL SIGNS & ANTHROPOMETRICS (7 items)

| Field | Type | Required | Format/Rule | Validation |
|-------|------|----------|-------------|------------|
| height_cm | Number | Yes | cm | > 0, hợp lý (50-250cm) |
| weight_kg | Number | Yes | kg | > 0, hợp lý (2-300kg) |
| bmi | Number | Auto-calculated | kg/m² | Cân nặng / [Chiều cao(m)]². Phân loại: < 18.5 Thiếu cân, 18.5-24.9 Bình thường, 25-29.9 Thừa cân, 30-34.9 Béo phì I, 35-39.9 Béo phì II, ≥ 40 Béo phì III |
| waist_cm | Number | Optional | cm | ⚠️ Nguy cơ cao nếu: Nam > 90cm, Nữ > 80cm |
| systolic_bp | Number | Optional | mmHg | Phân loại: < 120 Bình thường, 120-129 Tăng nhẹ, 130-139 Giai đoạn 1, ≥ 140 Giai đoạn 2 |
| diastolic_bp | Number | Optional | mmHg | < 80 Bình thường, 80-89 Giai đoạn 1, ≥ 90 Giai đoạn 2 |
| heart_rate | Number | Optional | bpm | < 60 Chậm, 60-100 Bình thường, > 100 Nhanh |
| spo2 | Number | Optional | % | 95-100% Bình thường, 90-94% Thấp nhẹ, < 90% ⚠️ Thấp |

#### F. REVIEW OF SYSTEMS (ROS) – 25-30 items trên 8 hệ cơ quan

_"Trong 2 tuần qua, bạn có gặp các triệu chứng sau không?"_

| Hệ cơ quan | Triệu chứng (Boolean mỗi item) |
|------------|-------------------------------|
| F.1 CONSTITUTIONAL (Toàn thân) | Sốt/ớn lạnh, Giảm cân không chủ đích (> 5% trong 6 tháng), Mệt mỏi kéo dài, Đổ mồ hôi đêm |
| F.2 NEUROLOGICAL (Thần kinh) | Đau đầu thường xuyên, Chóng mặt/choáng váng, Ngất xỉu, Tê bì/yếu chi, Thay đổi thị lực, Thay đổi trí nhớ/khả năng tập trung, Run/co giật |
| F.3 CARDIOVASCULAR (Tim mạch) | Đau ngực, Hồi hộp/nhịp tim không đều, Khó thở khi gắng sức, Phù chân/cổ chân, Khó thở khi nằm (orthopnea) |
| F.4 RESPIRATORY (Hô hấp) | Ho kéo dài (> 2 tuần), Khó thở khi nghỉ ngơi, Thở khò khè, Ho ra máu |
| F.5 GASTROINTESTINAL (Tiêu hóa) | Buồn nôn/nôn, Đau bụng, Tiêu chảy/táo bón, Máu trong phân, Khó nuốt, Ợ nóng/trào ngược |
| F.6 GENITOURINARY (Tiết niệu) | Đi tiểu buốt/đau/có máu, Đi tiểu đêm nhiều lần, Tiểu không tự chủ, Rối loạn chức năng sinh dục |
| F.7 MUSCULOSKELETAL (Cơ xương khớp) | Đau khớp, Sưng khớp, Cứng khớp buổi sáng, Đau lưng, Hạn chế vận động, Yếu cơ |
| F.8 SKIN/LYMPHATIC (Da liễu) | Phát ban/ngứa, Thay đổi nốt ruồi (kích thước, màu, hình dạng), Vết thương lành chậm, Hạch to, Thay đổi màu da |

#### G. REPRODUCTIVE/ENDOCRINE HEALTH (10-15 items, conditional theo giới tính)

**G.1 FOR WOMEN (Nữ giới):**

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| menarche_age | Number | Yes | Tuổi có kinh lần đầu |
| menstrual_cycle | String | Yes | Đều đặn (26-32 ngày) / Không đều / Đã mãn kinh |
| last_menstrual_period | Date | Conditional | DD/MM/YYYY |
| menopause_status | String | Yes | Chưa mãn kinh / Tiền mãn kinh / Đã mãn kinh |
| menopause_year | Number | Conditional | Năm mãn kinh |
| menopause_age | Number | Conditional | Tuổi mãn kinh |
| gravida | Number | Yes | Số lần mang thai |
| para | Number | Yes | Số con sinh sống |
| miscarriages | Number | Yes | Số lần sảy thai |
| abortions | Number | Yes | Số lần nạo phá thai |
| currently_pregnant | String | Yes | Không / Có / Không chắc chắn |
| contraception | String | Optional | Loại thuốc tránh thai |
| hormone_therapy | String | Optional | HRT loại |
| last_pap_smear_date | Date | Optional | DD/MM/YYYY |
| last_pap_smear_result | String | Optional | Bình thường / Bất thường / Chưa làm / Không nhớ |
| last_mammography_date | Date | Conditional (≥ 40 tuổi) | DD/MM/YYYY |
| last_mammography_result | String | Conditional | Bình thường / Bất thường / Chưa làm / Không nhớ |

**G.2 FOR MEN (Nam giới):**

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| prostate_symptoms | Array[String] | Yes | Không / Tiểu khó / Tiểu nhiều lần / Tiểu không hết / Khác |
| erectile_dysfunction | String | Yes | Không / Thỉnh thoảng / Thường xuyên |
| last_psa_date | Date | Conditional (≥ 50 tuổi) | DD/MM/YYYY |
| last_psa_result | Number | Conditional | ng/mL |
| last_dre_date | Date | Optional | DD/MM/YYYY |
| last_dre_result | String | Optional | Bình thường / Bất thường / Chưa làm |
| low_testosterone_symptoms | Array[String] | Yes | Mệt mỏi / Giảm ham muốn / Giảm cơ / Tăng mỡ bụng / Khó tập trung / Không có |
| testosterone_treatment | Boolean | Yes | Có/Không |
| testosterone_treatment_type | String | Conditional | Loại |

**G.3 FOR ALL (Chung – Tuyến giáp):**

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| hyperthyroid_symptoms | Array[String] | Yes | Run tay / Vã mồ hôi / Tim nhanh / Giảm cân / Lo lắng / Không có |
| hypothyroid_symptoms | Array[String] | Yes | Mệt mỏi / Tăng cân / Rụng tóc / Sợ lạnh / Da khô / Táo bón / Không có |

**Hệ thống** – Auto-save, draft lưu 7 ngày

⚠️ Data storage sập → CS báo kỹ thuật

UI States:
- Default: Form hiển thị, fields trống
- Typing: Validate real-time từng field, highlight lỗi ngay tại ô nhập
- Saving: "Đang lưu..." hiển thị
- Saved: "Đã lưu" indicator
- Error: Border đỏ + message lỗi tại field
- Completed: Chuyển Stage 3

Hoàn tất → chuyển Stage 3

---

### Stage 3: Lifestyle Assessment & Vaccination (14-17 phút, 59-75 items)

- INPUT: user_id (String, required), stage_1_data (Object), stage_2_data (Object)
- OUTPUT: stage_3_data (Object), stage_3_completed (Boolean)

#### A. NUTRITION PILLAR (12 câu hỏi)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| meals_per_day | String | Yes | 1 / 2 / 3 / 4+ bữa |
| skip_meals | String | Yes | Không bao giờ / Thỉnh thoảng / Thường xuyên bỏ sáng / Thường xuyên bỏ bữa khác |
| veggie_fruit_servings | String | Yes | < 1 / 1-2 / 3-4 / 5+ khẩu phần/ngày |
| water_intake | String | Yes | < 1L / 1-1.5L / 1.5-2L / > 2L |
| fast_food_per_week | String | Yes | Không / 1 lần / 2-3 lần / 4+ lần |
| sweets_per_week | String | Yes | Không / 1-2 lần / 3-5 lần / Hàng ngày |
| processed_food | String | Yes | Không / 1-2 bữa/tuần / 3-5 bữa/tuần / Hầu hết |
| protein_per_meal | String | Yes | Mỗi bữa / Hầu hết / Một vài / Hiếm khi |
| caffeine_per_day | String | Yes | Không / 1 / 2-3 / 4+ cốc |
| sugary_drinks | String | Yes | Không / 1-2 lần/tuần / 3-5 lần/tuần / Hàng ngày |
| special_diet | Array[String] | Yes | Không / Vegetarian / Vegan / Keto / Paleo / Mediterranean / IF / Khác |
| supplements | Boolean | Yes | Có/Không. Nếu có: loại gì |

#### B. MOVEMENT PILLAR (12 câu hỏi)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| vigorous_exercise_min_week | String | Yes | 0 / 1-30 / 30-75 / 75-150 / 150+ phút |
| walking_min_day | String | Yes | < 10 / 10-20 / 20-30 / 30-60 / > 60 phút |
| gym_per_week | String | Yes | Không / 1 / 2-3 / 4-5 / 6+ lần |
| strength_training | String | Yes | Không bao giờ / Thỉnh thoảng / 1/tuần / 2-3/tuần / 4+/tuần |
| strength_exercises | Array[String] | Yes | Không / Bodyweight / Free weights / Machine / Resistance bands / CrossFit |
| sitting_hours_day | String | Yes | < 4 / 4-6 / 6-8 / 8-10 / > 10 giờ |
| breaks_while_sitting | String | Yes | Mỗi 30-60 phút / Thỉnh thoảng / Hiếm khi / Không |
| yoga_stretching | String | Yes | Không / Thỉnh thoảng / 1-2/tuần / 3+/tuần |
| balance_training | String | Yes | Không / Thỉnh thoảng / Thường xuyên |
| musculoskeletal_pain | String | Yes | Không / Thỉnh thoảng / Thường xuyên / Mạn tính (+ vị trí) |
| movement_difficulty | String | Yes | Không / Hơi khó / Khá khó / Rất khó |
| fitness_self_score | Number | Yes | Scale 1-10 |

#### C. RECOVERY PILLAR (12 câu hỏi)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| sleep_hours | String | Yes | < 5 / 5-6 / 6-7 / 7-8 / 8-9 / > 9 giờ |
| sleep_quality_score | Number | Yes | Scale 1-10 |
| bedtime | String | Yes | Trước 22:00 / 22-23 / 23-00 / 00-01 / Sau 01:00 |
| wake_time | String | Yes | Trước 05:00 / 05-06 / 06-07 / 07-08 / Sau 08:00 |
| insomnia | String | Yes | Không / 1-2 đêm/tuần / 3-5 đêm/tuần / Hầu như mỗi đêm |
| night_waking | String | Yes | Không / Thỉnh thoảng / 2-3 lần/đêm / Rất nhiều |
| morning_feeling | String | Yes | Sảng khoái / Khá ổn / Mệt mỏi / Kiệt sức |
| napping | String | Yes | Không / Thỉnh thoảng / < 30 phút / 30-60 phút / > 60 phút |
| sleep_aids | String | Yes | Không / Thỉnh thoảng / 3-5/tuần / Hàng ngày (+ loại gì) |
| screen_before_bed | String | Yes | Tắt > 1h / 30-60 phút / 15-30 phút / Dùng đến khi ngủ |
| sleep_environment | Array[String] | Yes | Tối yên tĩnh / Mát mẻ / Tiếng ồn / Ánh sáng / Quá nóng-lạnh |
| recovery_feeling | String | Yes | Hoàn toàn / Phần nào / Thường mệt / Luôn kiệt sức |

#### D. MIND-BODY PILLAR (12 câu hỏi)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| stress_level | Number | Yes | Scale 1-10 |
| stress_sources | Array[String] | Yes | Công việc / Tài chính / Quan hệ / Sức khỏe / Không có / Khác |
| overwhelmed | String | Yes | Không / Thỉnh thoảng / Thường xuyên / Hầu như luôn |
| mood_2weeks | String | Yes | Tốt / Bình thường / Hơi thấp / Rất thấp |
| anxiety | String | Yes | Không / Thỉnh thoảng / Thường xuyên / Hàng ngày |
| phq2_interest | String | Yes | Không / Vài ngày / > nửa số ngày / Gần như mỗi ngày |
| phq2_depression | String | Yes | Không / Vài ngày / > nửa số ngày / Gần như mỗi ngày |
| mindfulness | String | Yes | Không / Đã thử / 1-2/tuần / 3-5/tuần / Hàng ngày |
| stress_activities | Array[String] | Yes | Không có / Thể dục / Thiền / Đọc sách / Nhạc / Bạn bè / Hobby / Khác |
| social_connection | String | Yes | Rất phong phú / Đủ dùng / Hơi thiếu / Cô đơn |
| social_activities | String | Yes | Mỗi tuần / 1-2/tháng / Hiếm khi / Không |
| hobbies | String | Yes | Có thường xuyên / Có ít thời gian / Muốn nhưng không biết / Không có |
| life_purpose | String | Yes | Rất rõ ràng / Phần nào / Không chắc / Mơ hồ |

#### E. VACCINATION STATUS (5 câu hỏi)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| covid_vaccine | String | Yes | Chưa / 1 mũi / 2 mũi / 3+ mũi. Nếu có: tháng/năm cuối |
| flu_vaccine | String | Yes | Có (12 tháng qua) / Không / Không nhớ |
| other_vaccines | Array[Object] | Optional | Mỗi item: {type, year}. Types: Viêm gan B, Uốn ván, HPV, Zona (≥50), Phế cầu (≥65), MMR, Thủy đậu, Khác |
| vaccination_records | String | Yes | Có (upload GĐ4) / Có nhưng không biết ở đâu / Không có / Không chắc |
| vaccine_consultation | String | Yes | Có / Không / Sẽ quyết định sau |

#### F. FUNCTIONAL STATUS ASSESSMENT (8-12 items, conditional: > 70 tuổi HOẶC bệnh mạn tính)

**F.1 ADL (Hoạt động sinh hoạt hàng ngày):**

| Hoạt động | Type | Options |
|-----------|------|---------|
| Tắm rửa | String | Tự làm / Cần hỗ trợ / Phụ thuộc hoàn toàn |
| Ăn uống | String | Tự làm / Cần hỗ trợ / Phụ thuộc hoàn toàn |
| Mặc quần áo | String | Tự làm / Cần hỗ trợ / Phụ thuộc hoàn toàn |
| Đi lại trong nhà | String | Tự làm / Cần hỗ trợ / Phụ thuộc hoàn toàn |
| Đại/tiểu tiện tự chủ | String | Tự làm / Cần hỗ trợ / Phụ thuộc hoàn toàn |
| Di chuyển từ giường/ghế | String | Tự làm / Cần hỗ trợ / Phụ thuộc hoàn toàn |

**F.2 IADL (Hoạt động sinh hoạt mở rộng):**

| Hoạt động | Type | Options |
|-----------|------|---------|
| Đi chợ/mua sắm | String | Tự làm / Cần hỗ trợ / Không làm được |
| Nấu ăn | String | Tự làm / Cần hỗ trợ / Không làm được |
| Quản lý tài chính | String | Tự làm / Cần hỗ trợ / Không làm được |
| Sử dụng điện thoại | String | Tự làm / Cần hỗ trợ / Không làm được |
| Phương tiện công cộng | String | Tự làm / Cần hỗ trợ / Không làm được |
| Tự quản lý thuốc | String | Tự làm / Cần hỗ trợ / Không làm được |

**F.3 Functional Capacity Scales:**

| Field | Type | Required | Options |
|-------|------|----------|---------|
| nyha_class | String | Conditional (bệnh tim) | Class I-IV / Không áp dụng |
| mmrc_dyspnea | String | Conditional (bệnh phổi) | Grade 0-4 / Không áp dụng |
| ecog_performance | String | Optional | 0-4 / Không áp dụng |

#### G. SDOH – Yếu tố xã hội ảnh hưởng sức khỏe (6-8 items)

| Field | Type | Required | Format/Rule |
|-------|------|----------|-------------|
| housing_status | String | Yes | Ổn định / Không ổn định / Vô gia cư |
| food_security | String | Yes | Luôn đủ / Thỉnh thoảng thiếu / Thường xuyên không đủ |
| transportation | String | Yes | Phương tiện riêng / Công cộng / Khó khăn |
| financial_strain | String | Yes | Không / Đôi khi / Thường xuyên / Rất khó khăn |
| social_support | String | Yes | Có người hỗ trợ / Ít / Cô đơn |
| education | String | Yes | Dưới cấp 3 / Cấp 3 / Cao đẳng-ĐH / Sau ĐH |
| health_literacy | String | Yes | Hiểu rõ / Đôi khi cần giải thích / Thường cần hỗ trợ |
| employment_status | String | Yes | Toàn thời gian / Bán thời gian / Tự kinh doanh / Nghỉ hưu / Thất nghiệp / Không thể làm việc |

**Hệ thống** – Form với skip logic, auto-save

⚠️ Form system sập → CS báo kỹ thuật

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Hoàn tất GĐ3, có tài liệu upload | Chuyển Stage 4 |
| Hoàn tất GĐ3, không upload | Chuyển Progress Tracker (Luồng B) |

---

### Stage 4: Objective Data Upload (9-21 phút, uploads + 13-21 items)

- INPUT: user_id (String, required), stage_1-3_data (Object)
- OUTPUT: stage_4_data (Object), uploaded_files (Array), stage_4_completed (Boolean)

#### A. XÉT NGHIỆM MÁU (Upload PDF/Image)

**Essential Lab Panels:**

| Panel | Chỉ số |
|-------|--------|
| 1. CBC (Công thức máu) | WBC, RBC, Hemoglobin, Hematocrit, Platelets, MCV, MCH, MCHC |
| 2. Lipid Panel (Mỡ máu) | Total Cholesterol, LDL-C, HDL-C, Triglycerides, LDL/HDL ratio, Non-HDL |
| 3. Glucose Metabolism | Fasting Glucose, HbA1c |
| 4. Liver Function | AST, ALT, GGT, Alk Phos, Total Bilirubin, Albumin, Total Protein |
| 5. Kidney Function | Creatinine, BUN, eGFR, Uric Acid |
| 6. Thyroid Panel | TSH, Free T4, Free T3 (optional) |
| 7. Electrolytes | Na, K, Cl, HCO3, Ca |

**Advanced Lab Panels (nếu có):**

| Panel | Chỉ số |
|-------|--------|
| 8. Inflammatory Markers | hs-CRP, ESR, Homocysteine |
| 9. Advanced Lipid | Lipoprotein(a), ApoB, ApoA1, sdLDL |
| 10. Insulin Resistance | Fasting Insulin, HOMA-IR |
| 11. Hormone (Nam) | Total/Free Testosterone, SHBG, Estradiol, DHEA-S, Cortisol, IGF-1 |
| 11. Hormone (Nữ) | Estradiol, Progesterone, FSH, LH, DHEA-S, Testosterone, Cortisol, Prolactin |
| 12. Vitamin & Mineral | Vitamin D, B12, Folate, Iron panel (Serum Iron, TIBC, Ferritin), Magnesium, Zinc |

#### B. HÌNH ẢNH Y KHOA (Upload Image/PDF)

| Category | Items |
|----------|-------|
| Cardiac Imaging | ECG/EKG, Echocardiogram, Cardiac CT (CAC, CTA), Cardiac MRI, Stress Test |
| Body Composition | DEXA Scan, CT/MRI Body Composition, BIA |
| Chest Imaging | Chest X-ray, Low-Dose CT Chest, PFT |
| Abdominal Imaging | Abdominal Ultrasound, Liver Elastography, CT/MRI Abdomen |
| Vascular Imaging | Carotid Ultrasound (IMT), ABI, Venous Doppler |
| Other | Colonoscopy, Endoscopy, Mammogram (Nữ), Pap Smear (Nữ), Eye Exam |

#### C. THIẾT BỊ ĐEO & HOME MONITORING

**Wearables Integration:**
- Apple Health, Google Fit, Fitbit, Garmin, Oura Ring, WHOOP, Samsung Health, Khác
- Dữ liệu đồng bộ: Sleep data, Activity levels, Heart rate (HR nghỉ, HR max, HRV), SpO2, Calories, Activity trends

**Home Monitoring Devices:**
- Blood Pressure Monitor, Weight Scale, Glucose Monitor (CGM), Pulse Oximeter, Smart Thermometer

#### D. HỒ SƠ Y TẾ KHÁC (Upload Documents)

- Medical Records: Discharge Summaries, Consultation Notes, Procedure Reports
- Medication: Prescription Records, Treatment Plans
- Vaccination Records: Sổ Tiêm Chủng
- Specialty: Genetic Testing, Allergy Test, Sleep Study, Microbiome Testing
- Previous Assessments: Executive Health Checkup, Annual Physical, Wellness Screening

#### E. PREVENTIVE CARE & CANCER SCREENING (8-10 items)

| Field | Type | Required | Condition | Options |
|-------|------|----------|-----------|---------|
| colonoscopy | Object | Yes | | Chưa làm / Đã làm: {year, result: Bình thường/Polyp/Bất thường} |
| mammography | Object | Conditional | Nữ ≥ 40 tuổi | Chưa làm / Đã làm: {year, result} |
| pap_smear | Object | Conditional | Nữ | Chưa làm / Đã làm: {year, result} |
| psa | Object | Conditional | Nam ≥ 50 tuổi | Chưa làm / Đã làm: {year, result ng/mL} |
| low_dose_ct_lung | Object | Conditional | Hút thuốc ≥ 20 pack-years | Chưa làm / Đã làm: {year, result} |
| dermatology_screening | Object | Yes | | Chưa làm / Đã làm: {year, result} |
| dexa_scan | Object | Conditional | Nữ ≥ 65, Nam ≥ 70 | Chưa làm / Đã làm: {year, t_score, result} |
| aaa_ultrasound | Object | Conditional | Nam 65-75, từng hút thuốc | Chưa làm / Đã làm: {year, result} |
| eye_exam | String | Yes | | Chưa làm / < 12 tháng / > 12 tháng |
| dental_checkup | String | Yes | | < 6 tháng / 6-12 tháng / > 12 tháng / Chưa bao giờ |

#### F. COGNITIVE FUNCTION ASSESSMENT (5-11 items, CHỈ > 65 tuổi)

**F.1 Sàng lọc nhận thức:**

| Field | Type | Required | Options |
|-------|------|----------|---------|
| memory_change | String | Yes | Không / Quên vặt / Thường xuyên quên / Quên nghiêm trọng |
| financial_management_difficulty | String | Yes | Không / Đôi khi nhầm / Cần hỗ trợ |
| medication_management | String | Yes | Không / Thỉnh thoảng quên / Thường quên / Cần hỗ trợ |
| getting_lost | String | Yes | Không / Hiếm / Thỉnh thoảng / Thường xuyên |
| face_recognition | String | Yes | Không / Đôi khi quên tên / Thường không nhận ra |
| personality_change | String | Yes | Không / Nhẹ / Đáng kể |
| cognitive_test_history | Object | Optional | Chưa làm / Đã làm: {year, result} (Mini-Cog, MMSE, MoCA) |

**F.2 Đánh giá nguy cơ ngã (> 65 tuổi):**

| Field | Type | Required | Options |
|-------|------|----------|---------|
| falls_past_year | String | Yes | Không / 1 lần / 2+ lần |
| fear_of_falling | String | Yes | Không / Hơi lo / Rất sợ |
| balance_difficulty | String | Yes | Không / Đôi khi / Thường xuyên |
| mobility_aids | String | Yes | Không / Gậy / Walker / Xe lăn |

**File Upload Guidelines:**

| Định dạng | Kích thước | Ghi chú |
|-----------|-----------|---------|
| PDF | ≤ 10MB | Ưu tiên cho xét nghiệm, báo cáo |
| JPG/PNG | ≤ 10MB | Cho ảnh chụp kết quả |
| HEIC | ≤ 10MB | Từ iPhone |
| DICOM | ≤ 50MB | Cho hình ảnh y khoa (CT, MRI, X-ray) |

Validation Rules (Upload):

| Field | Rule | Message lỗi |
|-------|------|-------------|
| file_format | PDF/JPG/PNG/HEIC/DICOM | "Định dạng file không được hỗ trợ. Vui lòng upload PDF, JPG, PNG, HEIC hoặc DICOM" |
| file_size (non-DICOM) | ≤ 10MB | "File quá lớn. Vui lòng upload file dưới 10MB" |
| file_size (DICOM) | ≤ 50MB | "File quá lớn. Vui lòng upload file dưới 50MB" |

**Hệ thống** – Upload file, kiểm tra format/size, AI OCR trích xuất, mã hóa PHI

⚠️ File storage sập → CS báo kỹ thuật

⛔ Rẽ nhánh (Upload):

| Điều kiện | Hành động |
|-----------|-----------|
| File sai định dạng | Hiển thị thông báo lỗi + hướng dẫn |
| Upload thành công | AI OCR trích xuất dữ liệu |
| Upload lỗi | Auto retry 3 lần → thông báo: "Tải file thất bại. Vui lòng thử lại sau 5 phút." |

UI States (Upload):
- Default: Khu vực drag-and-drop hoặc nút chọn file
- Uploading: Progress bar cho từng file
- Processing: "AI đang xử lý..."
- Success: ✓ File uploaded + tên file
- Error: ✗ Thông báo lỗi + nút retry
- OCR Complete: Hiển thị dữ liệu trích xuất

---

### Bước AI Analysis & Health Overview

- **Hệ thống** – AI xử lý trong 20-30 giây
- INPUT: stage_1_data + stage_2_data + stage_3_data + stage_4_data (Object, required), uploaded_files (Array)
- OUTPUT: health_overview (Object: {body_systems_risk[8], cvd_risk_10yr, biological_age, care_plan_12mo, care_gaps}), pdf_file (File)

**AI Processing Pipeline:**
1. OCR Text Extraction – Trích xuất text từ ảnh/PDF
2. Data Normalization – Chuẩn hóa đơn vị, format
3. Value Interpretation – Phân tích giá trị (optimal/normal/at-risk/critical)
4. Trend Analysis – So sánh với data trước đó
5. Abnormality Detection – Gắn cờ giá trị bất thường
6. Risk Scoring – Tính điểm nguy cơ cho từng hệ thống
7. Cross-Reference – Liên kết với triệu chứng & lịch sử bệnh
8. Evidence Synthesis – Tổng hợp bằng chứng
9. Personalized Recommendations – Đề xuất cá nhân hóa

**Health Overview Components:**

| Component | Mô tả |
|-----------|-------|
| 8 Body Systems Risk | Risk scores cho 8 hệ cơ quan (0-100) |
| 10-year Disease Predictions | CVD, ASCVD, CVA, CHF, Diabetes predictions |
| Biological Age | So sánh với tuổi thực |
| 12-month Care Plan | Thuốc, xét nghiệm, tiêm chủng, SMART goals |
| Lab Integration | Giá trị xét nghiệm + interpretation + trends |
| Imaging Summary | Findings từ hình ảnh y khoa |
| Wearables Insights | Trends từ thiết bị đeo (sleep, activity, HR) |

**Care Gap Detection:**

| Condition | Alert |
|-----------|-------|
| Pap smear > 3 năm | ⚠️ Overdue cervical cancer screening |
| Mammogram > 2 năm (≥ 40 tuổi) | ⚠️ Overdue breast cancer screening |
| Đái tháo đường + chưa khám mắt | ⚠️ Annual eye exam recommended |

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Hoàn tất < 60s | Hiển thị Health Overview |
| Timeout > 60s | Thông báo: "Hệ thống đang xử lý. Kết quả sẽ gửi qua email trong 30 phút." |
| AI Analysis sập sau 3 retry | CS nhận alert, thông báo khách xử lý thủ công trong 24 giờ |

⚠️ AI phát hiện dữ liệu y tế bất thường/mâu thuẫn → **MD can thiệp** xem xét trước khi gửi khách

UI States:
- Processing: Thanh tiến trình, "Đang phân tích dữ liệu..."
- Complete: Health Overview hiển thị đầy đủ
- Timeout: Thông báo gửi email
- Error: Thông báo lỗi, CS sẽ liên hệ

---

### Bước Hoàn tất

- **Hệ thống** – Cập nhật trạng thái tài khoản: **Fully Activated**
- INPUT: user_id (String, required), health_overview (Object, required)
- OUTPUT: account_status (String: "fully_activated"), access_rights (Object), audit_log (Object)

Hệ thống tự động:
1. Cập nhật trạng thái → Fully Activated
2. Mở quyền truy cập đầy đủ theo gói dịch vụ
3. Gửi email + in-app notification: "Chúc mừng! Tài khoản của bạn đã được kích hoạt đầy đủ."
4. Ghi audit log hoàn tất onboarding kèm timestamp

⚠️ Status update sập sau 3 retry → CS nhận alert, kích hoạt thủ công

---

## LUỒNG PHỤ: PROGRESS TRACKER (PARTIAL ACTIVATED)

- Khi khách hàng hoàn tất < 4 Stage
- INPUT: user_id (String, required), stages_completed (Number), stages_data (Object)
- OUTPUT: progress_display (Object), account_status (String: "partial_activated")

**Hệ thống:**
1. Hiển thị tiến trình (VD: 3/4 stages done), CTA cho stage còn thiếu
2. Trạng thái: Partial Activated (truy cập dịch vụ cơ bản)
3. Lưu raw data vào Internal Dashboard cho BD/MD/CS
4. Auto-reminders: 24h, 3 ngày, 7 ngày, 14 ngày

Wireframe Progress Tracker:

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              HEALTH OVERVIEW                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    🔒 Health Overview đang được xây dựng...                │
│    Hoàn thành tất cả các bước để xem                       │
│    báo cáo sức khỏe toàn diện của bạn                      │
│                                                             │
│    📊 TIẾN ĐỘ HOÀN THÀNH                                   │
│    ████████████████████████░░░░░░░░░░  75%                 │
│                                                             │
│    ✅ Bước 1: Thông tin cơ bản           Hoàn thành        │
│    ✅ Bước 2: Lịch sử y khoa             Hoàn thành        │
│    ✅ Bước 3: Đánh giá lối sống          Hoàn thành        │
│    ⏳ Bước 4: Upload dữ liệu             Chưa hoàn thành   │
│                                                             │
│    💡 Tại sao cần hoàn thành đủ 4 bước?                    │
│    Để đảm bảo Health Overview chính xác và toàn diện       │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              TIẾP TỤC BƯỚC 4                        │  │
│    └─────────────────────────────────────────────────────┘  │
│    [Làm sau]                                               │
└─────────────────────────────────────────────────────────────┘
```

Progress Tracker States:

| Trạng thái | Progress Bar | CTA Button |
|------------|--------------|------------|
| 1/4 hoàn thành | ████░░░░░░░░ 25% | "Tiếp tục Bước 2" |
| 2/4 hoàn thành | ████████░░░░ 50% | "Tiếp tục Bước 3" |
| 3/4 hoàn thành | ████████████░░ 75% | "Tiếp tục Bước 4" |
| 4/4 hoàn thành | ████████████████ 100% | → Chuyển AI Analysis |

⚠️ Render sập sau 3 retry → CS nhận alert, thông báo email

---

## LUỒNG PHỤ: RESUME DRAFT

- INPUT: user_id (String, required), draft_id (String, required)
- OUTPUT: draft_data (Object), resume_stage (Number), progress_bar (Object)

**Bước 1: Resume Session**
- Hệ thống hiển thị thông báo resume + progress bar
- Khách chọn "Tiếp tục" hoặc "Bắt đầu lại"

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Chọn "Tiếp tục" | Chuyển đến giai đoạn còn dang dở |
| Chọn "Bắt đầu lại" | Xóa draft, khởi động Giai đoạn 1 |
| Load draft lỗi | Auto retry 3 lần → mặc định Giai đoạn 1, ghi log |

**Bước 2: Continue Stage**
- Hệ thống pre-fill dữ liệu đã lưu
- Đặt con trỏ tại field đầu tiên chưa điền
- Auto-save tiếp tục mỗi 30s

⚠️ Draft data corrupted → CS nhận alert, hỗ trợ bắt đầu lại

---

## LUỒNG PHỤ: EDIT PROFILE (SAU ONBOARDING)

- **1. Khách hàng** – Dashboard → Hồ sơ y tế → Chỉnh sửa ("Edit Profile" hoặc "Update Medical Info")
- INPUT: user_id (String, required), current_profile_data (Object)

- **2. Hệ thống** – Chuyển sang Form Mode, pre-fill dữ liệu hiện tại
- OUTPUT: form_mode_displayed (Boolean), all_fields_prefilled (Boolean)

- **3. Khách hàng** – Chỉnh sửa thông tin trên form 4 stages

- **4. Hệ thống** – Real-time validation, change tracking (chỉ ghi nhận fields thay đổi), auto-save

- **5. Khách hàng** – Lưu thay đổi (nhấn Save)

- **6. Hệ thống** – Validate toàn bộ → lưu changed fields vào EMR → ghi audit log
- INPUT: changed_fields (Object), user_id (String)
- OUTPUT: save_success (Boolean), audit_log (Object)

⛔ Rẽ nhánh (Save):

| Điều kiện | Hành động |
|-----------|-----------|
| Validation pass | Lưu changed fields, hiển thị "Thông tin đã được cập nhật thành công" |
| Validation fail | Highlight lỗi, yêu cầu sửa |
| Lưu lỗi | Auto retry 3 lần → thông báo lỗi, giữ dữ liệu cũ |

⚠️ DB sập > 10 phút → CS nhận alert, thông báo dữ liệu chưa lưu

- **7. Hệ thống** – Re-generate Health Overview: AI tính toán lại (≤ 45 giây)
  - Hiển thị: "Đang cập nhật Health Overview của bạn..."
  - ⚠️ AI sập sau 3 retry → CS báo kỹ thuật, thông báo kết quả trong 24h
  - ⚠️ AI phát hiện dữ liệu bất thường sau cập nhật → **MD can thiệp** review trước khi gửi khách

- **8. Hệ thống** – Hiển thị Health Overview cập nhật, tạo PDF mới, notification trong app

⚠️ Render sập → gửi PDF qua email

---

## Smart Features

| Feature | Trigger | Action | Lợi ích |
|---------|---------|--------|---------|
| Smart Expansion | Check vào checkbox bệnh | Hiển thị 2-4 câu hỏi bổ sung bên dưới (15-30s) | Chi tiết mà không overwhelm |
| Skip Logic | Check "Không có bệnh" / "No allergies" | Disable checkbox khác, auto scroll NEXT | Tiết kiệm thời gian người khỏe |
| Photo Upload | Chọn upload ảnh thuốc | AI OCR nhận diện thuốc (10s vs 5 phút manual) | Giảm lỗi chính tả, tăng completion |
| Progress Indicator | Mỗi bước | Thanh tiến trình (VD: 35%, Section 3/8) | Giảm drop-off |
| Save & Resume | Auto mỗi 30s | Lưu draft 7 ngày, email nhắc ngày 1, 3, 7 | Không mất dữ liệu |

---

## OUTPUT SYSTEM (v5.0)

### Health Overview (chỉ khi hoàn tất 4 Stage)

- 8 Body Systems Risk (điểm số + phân loại rủi ro)
- 10-year Disease Predictions
- Biological Age
- 12-month Care Plan
- Lab Integration (kết quả xét nghiệm)
- Imaging Summary
- Wearables Insights

### Internal Dashboard (cho BD/MD/CS)

- Raw data từ tất cả stages
- Alerts cho dữ liệu bất thường
- Không hiển thị cho khách hàng

| Role | Quyền xem | Quyền hành động |
|------|-----------|-----------------|
| BD | Raw data, Progress | Gọi/Email nhắc nhở KH |
| MD | Raw data, Alerts, Health Overview | Ghi chú y khoa, Flag issues |
| CS | Progress, Basic info | Hỗ trợ KH hoàn thành onboarding |

### Re-generate Health Overview Triggers

**Auto (thay đổi lớn):** Thêm thuốc mới, thêm bệnh mãn tính, upload xét nghiệm mới, thay đổi đáng kể trong hồ sơ

**Không trigger (thay đổi nhỏ):** Sửa typo, cập nhật liên hệ, thêm dị ứng thực phẩm nhẹ

**Manual:** Settings → "Refresh Health Overview" → AI re-analyze

---

## BẢNG THÔNG BÁO TỰ ĐỘNG

| Sự kiện | Kênh | Người nhận |
|---------|------|------------|
| Onboarding hoàn tất (Fully Activated) | Email + In-app | Khách hàng |
| Health Overview sẵn sàng | Email + In-app | Khách hàng |
| Health Overview timeout (> 60s) | Email (khi sẵn sàng) | Khách hàng |
| Nhắc hoàn tất stages còn thiếu | Email (24h, 3d, 7d, 14d) | Khách hàng |
| Draft sắp hết hạn | Email | Khách hàng |
| AI phát hiện dữ liệu bất thường | Alert nội bộ | MD |

### Template thông báo

📧 **Email hoàn tất Onboarding:**
- Subject: "Chúc mừng! Tài khoản CVH Healthcare của bạn đã được kích hoạt đầy đủ"
- Body: Tên KH, Trạng thái tài khoản (Fully Activated), Link xem Health Overview, Tóm tắt gói dịch vụ

📧 **Email Health Overview sẵn sàng:**
- Subject: "Health Overview của bạn đã sẵn sàng!"
- Body: Tên KH, Tóm tắt 8-system score, Link xem chi tiết, Link tải PDF

📧 **Email Health Overview timeout:**
- Subject: "Health Overview của bạn đang được xử lý"
- Body: "Kết quả sẽ gửi qua email trong 30 phút"

📧 **Email nhắc hoàn tất:**
- Subject: "Hoàn thành hồ sơ y tế để nhận Health Overview"
- Body: Tên KH, Tiến độ (X/4 stages), Stage còn thiếu, CTA link tiếp tục, Thời gian ước tính

📧 **Email draft sắp hết hạn:**
- Subject: "Bản nháp hồ sơ y tế của bạn sắp hết hạn"
- Body: Tên KH, Thời gian còn lại, Link tiếp tục

📱 **In-app notification hoàn tất:**
- Nội dung: "Chúc mừng! Tài khoản của bạn đã được kích hoạt đầy đủ."

📱 **In-app notification Health Overview cập nhật:**
- Nội dung: "Health Overview của bạn đã được cập nhật"

🔔 **Alert nội bộ MD:**
- Nội dung: "[Patient ID] – AI phát hiện dữ liệu y tế bất thường/mâu thuẫn. Cần review trước khi gửi khách."

---

## BẢNG GIÁM SÁT CS-MD ĐẦY ĐỦ

- **Hệ thống auto ≥ 95%:** Mọi bước đều auto-save, auto-retry, validation real-time
- **CS chỉ can thiệp khi FATAL (< 5%):** Module/DB sập > 10 phút, tất cả retry thất bại
- **MD can thiệp:** CHỈ khi AI phát hiện dữ liệu y tế bất thường hoặc mâu thuẫn

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| First Login | 100% session ghi nhận đúng, 0% lỗi kỹ thuật | Auto retry 3 lần (2s), ghi log toàn bộ | Auth DB sập > 10 phút (auto-healing failed) | ❌ |
| Tutorial | Video tải ≤ 2s, 100% trạng thái completed/skipped ghi nhận đúng | Auto retry 3 lần, hiển thị "Bỏ qua" nếu lỗi | Màn tutorial chặn toàn bộ flow | ❌ |
| Check Draft | ≤ 2s trả kết quả, 100% phát hiện đúng draft, 0% false positive | Retry 3 lần → mặc định GĐ1, ghi log | Draft DB sập > 10 phút | ❌ |
| Giai đoạn 1 (AI Chatbox) | 100% câu trả lời ghi nhận đúng, skip logic chính xác, auto-save 30s | AI hỏi lại nếu sai, auto-save, resume khi gián đoạn | AI Chatbox sập > 10 phút | ❌ |
| Giai đoạn 2 | 100% fields bắt buộc validate, auto-save 30s, 0% mất dữ liệu | Validate real-time, auto-save, draft 7 ngày | Data storage sập > 10 phút | ❌ |
| Giai đoạn 3 | 100% fields validate, skip logic chính xác, 0% lỗi hiển thị | Skip logic, validate real-time, auto-save 30s | Form system sập > 10 phút | ❌ |
| Giai đoạn 4 (Upload) | 100% file xác nhận thành công, AI OCR đúng, 0% mất file | Kiểm tra format/size, retry 3 lần, mã hóa PHI | File storage sập > 10 phút | ❌ |
| AI Analysis | ≤ 30s phân tích, 8 hệ cơ quan đủ, 0% lỗi tính toán | Thanh tiến trình, timeout → gửi email 30 phút | AI sập sau 3 retry → xử lý thủ công 24h | ✅ Dữ liệu bất thường/mâu thuẫn |
| Health Overview | ≤ 3s hiển thị, 100% khớp AI Analysis, PDF đúng | Retry render 3 lần → gửi PDF email | Render sập sau 3 retry | ❌ |
| Complete | ≤ 2 phút cập nhật, 100% status đúng, 0% cấp sai quyền | Cập nhật status, mở quyền, gửi email/SMS | Status update sập sau 3 retry | ❌ |
| Progress Tracker | ≤ 3s hiển thị, CTA đúng, nhắc nhở 24h/3d/7d/14d | Render + CTA, auto-reminders | Render sập sau 3 retry | ❌ |
| Complete Partial | ≤ 2s cập nhật, quyền cơ bản đúng, 0% mất dữ liệu | Cập nhật Partial Activated, nhắc 3/7/14 ngày | Status update sập sau 3 retry | ❌ |
| Resume Session | ≤ 3s load, 100% draft đúng vị trí, progress bar chính xác | Load draft, render progress, 2 lựa chọn | Draft data corrupted sau 3 retry | ❌ |
| Continue Stage | 100% pre-fill đúng, auto-save 30s, skip logic đúng | Pre-fill, con trỏ field đầu chưa điền | Data storage sập > 10 phút | ❌ |
| Edit Profile | Chuyển Form Mode thành công, 100% pre-fill đúng | Retry load 3 lần | Form Mode sập sau 3 retry | ❌ |
| Form Mode | ≤ 3s tải form, validate real-time, chỉ ghi fields thay đổi | Render form, tracking changes | Form render sập sau 3 retry | ❌ |
| Save | ≤ 5s lưu, 100% thay đổi lưu đúng, 0% mất dữ liệu | Validate → lưu changed fields → audit log | DB sập > 10 phút sau 3 retry | ❌ |
| Re-generate Health Overview | ≤ 45s, 100% phản ánh dữ liệu mới, 8-system chính xác | Thông báo đang cập nhật, timeout → email 30 phút | AI sập sau 3 retry → xử lý 24h | ✅ Dữ liệu bất thường sau cập nhật |
| Complete (Edit) | ≤ 3s, 100% khớp re-generate, PDF đúng | Hiển thị + PDF + notification | Render sập sau 3 retry | ❌ |

---

## DATA SPECIFICATION

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| user_id | String | Yes | Auto from Function Registration | Mã người dùng |
| onboarding_session_id | String | Auto | UUID | Session onboarding |
| is_first_login | Boolean | Auto | true/false | Lần đăng nhập đầu tiên |
| tutorial_status | String | Auto | "completed" / "skipped" | Trạng thái tutorial |
| draft_id | String | Auto | UUID | Mã bản nháp |
| draft_expiry | Date | Auto | created_at + 7 ngày | Hạn draft |
| stage_1_data | Object | Yes | {basic_profile, medical_history, medications, allergies} | Dữ liệu GĐ1 |
| stage_2_data | Object | Yes | {surgical, family, hospitalization, social, vital_signs, ros, reproductive} | Dữ liệu GĐ2 |
| stage_3_data | Object | Yes | {nutrition, movement, recovery, mind_body, vaccination, functional_status, sdoh} | Dữ liệu GĐ3 |
| stage_4_data | Object | Yes | {lab_tests, imaging, wearables, medical_records, preventive_care, cognitive} | Dữ liệu GĐ4 |
| uploaded_files | Array[Object] | Optional | Mỗi item: {file_id, file_name, file_type, file_size, upload_time, ocr_status} | Files uploaded |
| stages_completed | Number | Auto | 0-4 | Số stages hoàn tất |
| account_status | String | Auto | "onboarding" / "partial_activated" / "fully_activated" | Trạng thái tài khoản |
| health_overview | Object | Conditional (4/4) | {body_systems_risk[8], cvd_risk_10yr, biological_age, care_plan_12mo, lab_integration, imaging_summary, wearables_insights} | Kết quả AI Analysis |
| health_overview_pdf | File | Conditional | PDF | File PDF Health Overview |
| auto_save_timestamp | DateTime | Auto | ISO 8601 | Thời điểm auto-save cuối |
| audit_log | Array[Object] | Auto | {action, timestamp, user_id, changes} | Nhật ký thay đổi |
| pack_years | Number | Auto-calculated | (cigarettes/day ÷ 20) × years | Pack-years hút thuốc |
| bmi | Number | Auto-calculated | weight_kg / (height_m)² | BMI |
| phq2_score | Number | Auto-calculated | Sum of PHQ-2 items (0-6) | Điểm sàng lọc trầm cảm |
| hipaa_consent | Object | Yes (from Function Registration) | {accepted, timestamp, version} | HIPAA consent record |

---

## KPI & METRICS

### KPI theo từng Giai đoạn

| Metric | Giai đoạn 1 | Giai đoạn 2 | Giai đoạn 3 | Giai đoạn 4 |
|--------|-------------|-------------|-------------|-------------|
| Completion Rate | ≥ 95% | ≥ 85% | ≥ 80% | ≥ 60% |
| Time to Complete | 5-7 min | 8-10 min | 10-12 min | 5-15 min |
| Profile Quality | ≥ 85% | ≥ 90% | ≥ 95% | N/A |
| Skip Rate | < 5% | < 10% | < 15% | < 30% |

### KPI tổng thể

| Metric | Target |
|--------|--------|
| Full Completion (4 stages) | ≥ 60% trong 30 ngày |
| Stage 1-3 Completion | ≥ 80% |
| Data Upload Rate | ≥ 50% |
| AI Processing Success | ≥ 95% |
| User Satisfaction | ≥ 4.5/5 |
| Drop-off Rate per Stage | < 10% |

### Thời gian hoàn thành

| Đối tượng | GĐ 1 | GĐ 1+2 | GĐ 1+2+3 | Full (4 GĐ) |
|-----------|-------|---------|-----------|-------------|
| Người khỏe mạnh | 5-7 min | 13-17 min | 23-29 min | 30-45 min |
| Có bệnh nền | 6-8 min | 14-18 min | 25-32 min | 35-50 min |
| Bệnh nhân phức tạp | 7-9 min | 16-20 min | 28-35 min | 40-60 min |

---

## Auto-Save & Draft Rules

| Rule | Value |
|------|-------|
| Auto-save interval | Mỗi 30 giây |
| Draft retention | 7 ngày |
| Email reminder | Ngày 1, 3, 7 |
| After 7 days | Soft delete draft |
| Resume link | Valid 30 ngày |

---

## NGUỒN TÀI LIỆU GỐC

- Luồng nghiệp vụ: `docs/02-product/ba-specs/function-onboarding/base/Onboarding_Touchpoint_Flow_v1.md`
- Checklist CS & MD: `docs/02-product/ba-specs/function-onboarding/checklist-cs-md/checklist-cs-md.md`
- Bản tóm tắt: `docs/02-product/tomtat-chitiet/tomtat/function-onboarding-tomtat.md`
