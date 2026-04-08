# CHI TIẾT LUỒNG NGHIỆP VỤ – SERVICE 6 (Lab_Results_Consultation): TƯ VẤN KẾT QUẢ XÉT NGHIỆM (LAB RESULTS CONSULTATION)

## Header

- **Service:** Lab Results Consultation (Tư vấn kết quả xét nghiệm)
- **Version:** 1.0.0
- **Input:** Khách hàng upload kết quả xét nghiệm (PDF/JPG/PNG, max 10MB, max 10 files)
- **Output:** Care Plan cá nhân hóa (AI hoặc MD-approved) + Cập nhật EHR + Lịch follow-up
- **Pricing:** FLOW-A: phí cơ bản | FLOW-B: phí cơ bản + $50-100 (Video Call MD US)
- **Compliance:** HIPAA, AES-256 encryption

---

## Nguyên tắc cốt lõi

- **AI-first Model:** AI phân tích, tạo Care Plan, quyết định routing
- **AI Auto-routing:** Khách hàng KHÔNG chọn flow — AI tự quyết định FLOW-A hay FLOW-B
- **Pay-per-use:** Tính phí theo lượt sử dụng
- AI OCR confidence threshold: 85% (dưới 85% → manual review)
- 3-tier severity: Normal / Attention / Critical

---

## Danh sách Flow

| Flow | Tên | Mô tả | Thời gian |
|------|-----|-------|-----------|
| FLOW-A | AI Care Plan trực tiếp | AI gửi Care Plan cho KH ngay | 5-10 phút |
| FLOW-B | MD US Review + Video Call | MD US xem xét + video call tư vấn | 30-50 phút + đặt lịch |

### So sánh FLOW-A vs FLOW-B

| Tiêu chí | Luồng A: AI gửi trực tiếp | Luồng B: MD US Review + Phê duyệt |
|----------|---------------------------|-------------------------------------|
| **Khi nào AI chọn** | Bất thường mức Bình thường/Cần chú ý, không cần thuốc | Chỉ số nguy hiểm, cần kê đơn, mẫu phức tạp |
| **AI lập Kế hoạch** | **CÓ** (bước 6 — phần chung) | **CÓ** (bước 6 — phần chung) |
| **MD review Kế hoạch** | **KHÔNG** | CÓ — MD tư vấn, chỉnh sửa + phê duyệt |
| **Kế hoạch gửi cho KH** | Bản AI gốc (tự động) | Bản MD đã chỉnh sửa/phê duyệt |
| **Chi phí** | Phí cơ bản (rẻ hơn) | Phí cơ bản + $50-100 |
| **Thời gian** | 5-10 phút (tự động) | 30-50 phút + chờ lịch |
| **Số bước riêng** | 0 bước | 3 bước (B1-B3) |
| **Tổng số trạm** | 9 trạm (7 chung đầu + 0 riêng + 2 chung sau) | 12 trạm (7 chung đầu + 3 riêng + 2 chung sau) |
| **KH chọn luồng** | **KHÔNG** — AI tự động xác định | **KHÔNG** — AI tự động xác định |

---

## PHẦN CHUNG (Bước 1-7)

### Bước 1. Khách hàng – Upload kết quả xét nghiệm

- **INPUT:** lab_result_file (File[], required), file_count (Number: 1-10, required)
- **OUTPUT:** upload_id (String), file_ids (String[]), storage_location (String)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Định dạng file | Chỉ chấp nhận PDF, JPG, PNG | "Định dạng file không hỗ trợ. Vui lòng upload PDF, JPG hoặc PNG" |
| Kích thước file | Mỗi file tối đa 10MB | "File vượt quá giới hạn 10MB. Vui lòng nén file hoặc chia nhỏ" |
| Số lượng file | Tối đa 10 files mỗi lần upload | "Tối đa 10 files mỗi lần upload" |
| Quét mã độc | File phải qua quét an toàn | "File bị từ chối vì lý do bảo mật. Vui lòng kiểm tra và thử lại" |

#### UI States

- **Default:** Màn hình upload với khu vực kéo thả, nút [Chọn file] và [📷 Chụp ảnh]
- **Uploading:** Thanh tiến trình hiển thị phần trăm upload
- **Success:** File hiển thị với ✅ và tên file, kích thước, nút 🗑️ xóa
- **Error (format):** "Định dạng file không hỗ trợ. Vui lòng upload PDF, JPG hoặc PNG"
- **Error (size):** "File vượt quá giới hạn 10MB"
- **Error (malware):** "File bị từ chối vì lý do bảo mật"

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│     TƯ VẤN KẾT QUẢ XÉT NGHIỆM             │
├─────────────────────────────────────────────┤
│                                             │
│  Vui lòng upload file kết quả xét nghiệm    │
│  từ phòng lab bên ngoài                      │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │                                     │    │
│  │   Kéo thả file hoặc nhấn để chọn    │    │
│  │                                     │    │
│  │   Định dạng: PDF, JPG, PNG          │    │
│  │   Tối đa: 10MB / file              │    │
│  │   Tối đa: 10 files                 │    │
│  │                                     │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  [Chọn file]        [📷 Chụp ảnh]          │
│                                             │
│  📄 lab_results_2026.pdf    2.4 MB  ✅  🗑️  │
│  📄 blood_test.jpg          1.1 MB  ✅  🗑️  │
│                                             │
│  ████████████████████████████░░  85%        │
│                                             │
│  [         Upload         ]                 │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| File hợp lệ (đúng format, ≤ 10MB, sạch malware) | Lưu trữ mã hóa AES-256, chuyển bước 2 |
| File sai định dạng | Thông báo lỗi, yêu cầu upload đúng PDF/JPG/PNG |
| File > 10MB | Thông báo lỗi, hướng dẫn nén file |
| File chứa mã độc | Từ chối file, cách ly, yêu cầu KH kiểm tra lại |

#### ⚠️ Lỗi

- File được mã hóa AES-256 ngay khi upload, lưu trữ tuân thủ HIPAA
- KH có thể chụp ảnh kết quả XN trực tiếp bằng camera
- DB sập > 10 phút → CS nhận cảnh báo, gọi điện KH xin lỗi, hướng dẫn gửi file qua kênh dự phòng, báo IT xử lý khẩn cấp
- Lỗi upload sau auto-retry 3 lần → Thông báo: "Hệ thống đang bận. Vui lòng thử lại sau 5 phút." + ghi log IT kiểm tra

---

### Bước 2. Hệ thống – Truy xuất EHR

- **INPUT:** user_id (UUID, required — tự động từ phiên đăng nhập)
- **OUTPUT:** ehr_summary (Object: medical_history, medications, allergies, previous_labs)

#### UI States

- **Loading:** 🔄 "Đang tải hồ sơ y khoa..."
- **Success:** Hiển thị 4 mục: Lịch sử bệnh án, Thuốc đang dùng, Dị ứng, Kết quả XN trước đó
- **Empty (KH mới):** "Chưa có lịch sử y khoa" — tiếp tục bình thường
- **Error:** "Không thể tải hồ sơ y khoa. Vui lòng thử lại."

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│        HỒ SƠ Y KHOA CỦA BẠN                │
├─────────────────────────────────────────────┤
│                                             │
│  🔄 Đang tải hồ sơ y khoa...                │
│                                             │
│  ▼ Lịch sử bệnh án                          │
│  ├── Tiểu đường Type 2 (đang điều trị)      │
│  └── Tăng huyết áp (đang điều trị)          │
│                                             │
│  ▼ Thuốc đang dùng                           │
│  └── Metformin 500mg — 2 lần/ngày           │
│                                             │
│  ⚠️ Dị ứng                                   │
│  └── Penicillin — Phát ban (mức trung bình) │
│                                             │
│  ▼ Kết quả XN trước đó                       │
│  ├── 15/10/2025: Glucose 142 mg/dL (CAO)    │
│  └── 15/10/2025: HbA1c 6.8% (CAO)          │
│                                             │
│  [         Tiếp tục         ]               │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| EHR load thành công (≤ 3s) | Hiển thị tóm tắt EHR, chuyển bước 3 |
| KH mới (chưa có EHR) | Hiển thị "Chưa có lịch sử y khoa", tiếp tục bước 3 |
| EHR load thất bại | Auto-retry 2 lần. Vẫn lỗi → CS escalate IT |

#### ⚠️ Lỗi

- Hệ thống tự động thực hiện, KH không cần thao tác
- SLA: ≤ 3 giây để load EHR
- Dữ liệu EHR giới hạn 2 năm gần nhất cho kết quả XN cũ
- Mọi truy cập EHR được ghi HIPAA audit log
- EHR load thất bại sau 2 lần retry → CS escalate IT khẩn cấp, tạm dừng case
- Hiển thị nhầm dữ liệu KH khác → vi phạm HIPAA nghiêm trọng → CS báo cáo sự cố theo quy trình HIPAA Breach

---

### Bước 3. Khách hàng – Nhập thông tin lab

- **INPUT:** lab_name (String, required), test_date (Date, required), reason_for_test (String, required), additional_notes (String, optional), referring_doctor (String, optional)
- **OUTPUT:** lab_info_id (String), input_status (String)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Tên phòng lab | Bắt buộc, 2-100 ký tự | "Vui lòng nhập tên phòng lab" |
| Ngày xét nghiệm | Bắt buộc, không được tương lai, trong 1 năm qua | "Ngày xét nghiệm không hợp lệ. Vui lòng chọn ngày trong vòng 1 năm qua" |
| Lý do khám | Bắt buộc, tối thiểu 10 ký tự | "Vui lòng mô tả chi tiết hơn lý do khám (tối thiểu 10 ký tự)" |
| Ghi chú bổ sung | Không bắt buộc, tối đa 1000 ký tự | "Ghi chú vượt quá 1000 ký tự" |
| Bác sĩ giới thiệu | Không bắt buộc, tối đa 100 ký tự | — |

#### UI States

- **Default:** Form trống với placeholder rõ ràng (VD: "VD: LabCorp, Quest Diagnostics...")
- **Typing:** Validate real-time, highlight trường lỗi ngay khi nhập sai
- **Valid:** Border xanh khi field hợp lệ
- **Error:** Border đỏ + message lỗi cụ thể bên dưới field
- **Idle > 5 phút:** Push notification nhắc nhở: "Vui lòng hoàn tất thông tin để tiếp tục phân tích kết quả XN"
- **Session timeout:** 15 phút không thao tác → timeout

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│        THÔNG TIN XÉT NGHIỆM                │
├─────────────────────────────────────────────┤
│                                             │
│  🏥 Tên phòng lab *                          │
│  ┌──────────────────────────────────────┐   │
│  │ Quest Diagnostics              ▼     │   │
│  └──────────────────────────────────────┘   │
│  (LabCorp | Quest | Bệnh viện | Khác)      │
│                                             │
│  📅 Ngày xét nghiệm *                       │
│  ┌──────────────────────────────────────┐   │
│  │ 10/01/2026                    📅     │   │
│  └──────────────────────────────────────┘   │
│                                             │
│  📝 Lý do khám / Triệu chứng *              │
│  ┌──────────────────────────────────────┐   │
│  │ Khám định kỳ hàng năm. Gần đây hay  │   │
│  │ mệt mỏi và khát nước nhiều hơn      │   │
│  │ bình thường.                         │   │
│  └──────────────────────────────────────┘   │
│  56/500 ký tự                               │
│                                             │
│  📝 Ghi chú bổ sung (không bắt buộc)        │
│  ┌──────────────────────────────────────┐   │
│  │ Đã nhịn ăn 12 tiếng trước XN        │   │
│  └──────────────────────────────────────┘   │
│                                             │
│  👨‍⚕️ Bác sĩ giới thiệu (không bắt buộc)     │
│  ┌──────────────────────────────────────┐   │
│  │ Dr. Nguyen Van A                     │   │
│  └──────────────────────────────────────┘   │
│                                             │
│  [Quay lại]          [Tiếp tục]             │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Tất cả field bắt buộc hợp lệ | Lưu thông tin, chuyển bước 4 |
| Thiếu field bắt buộc hoặc sai format | Highlight trường lỗi, hiển thị message cụ thể |
| Session timeout 15 phút | Redirect về màn hình trước, yêu cầu nhập lại |

#### ⚠️ Lỗi

- Hệ thống gợi ý các lab phổ biến (LabCorp, Quest Diagnostics) trong dropdown
- Nếu lab không có trong danh sách → chọn "Khác" và nhập tên tự do
- Hệ thống sập > 10 phút → CS nhận cảnh báo, hỗ trợ KH nhập lại dữ liệu qua kênh dự phòng

---

### Bước 4. Hệ thống (AI OCR) – Trích xuất dữ liệu từ file upload

- **INPUT:** file_ids (String[], required), storage_paths (String[], required)
- **OUTPUT:** extraction_id (String), extracted_data (Object[]: parameter_name, value, unit, reference_range, status), confidence_score (Number)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Độ tin cậy OCR | Tự động chấp nhận nếu ≥ 85% | — |
| Trường tin cậy thấp | Đánh dấu vàng nếu < 85% | "Vui lòng kiểm tra các trường đánh dấu" |
| Tài liệu không đọc được | Chất lượng quá thấp | "Tài liệu không đủ rõ để đọc. Vui lòng upload bản scan chất lượng cao hơn" |
| Định dạng lab không nhận diện | Không nhận ra format lab | "Định dạng tài liệu không quen thuộc. Vui lòng nhập dữ liệu thủ công" |

#### UI States

- **Processing:** 🔄 "Đang trích xuất dữ liệu... Vui lòng chờ" + thanh tiến trình
- **Success (high confidence):** Bảng hiển thị chỉ số, giá trị, khoảng tham chiếu, trạng thái (🟢/🟡/🔴). "Cần xác nhận: 0 trường"
- **Partial (low confidence):** Bảng với trường confidence thấp highlight vàng để KH kiểm tra
- **Error (unreadable):** "Tài liệu không đủ rõ để đọc. Vui lòng upload bản scan chất lượng cao hơn"
- **OCR > 30 giây:** Thông báo: "Đang xử lý file, vui lòng chờ thêm vài giây..."

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│        TRÍCH XUẤT DỮ LIỆU XÉT NGHIỆM      │
├─────────────────────────────────────────────┤
│                                             │
│  🔄 Đang trích xuất dữ liệu... Vui lòng chờ│
│                                             │
│  Độ tin cậy: 96.7%  ✅                       │
│                                             │
│  ┌──────────────────────────────────────┐   │
│  │ Chỉ số          │ Giá trị │ Khoảng  │   │
│  ├──────────────────┼─────────┼─────────┤   │
│  │ Glucose, Fasting │ 142     │ 70-99   │   │
│  │                  │ mg/dL   │ mg/dL   │   │
│  │ Status: 🟡 CAO   │         │         │   │
│  ├──────────────────┼─────────┼─────────┤   │
│  │ HbA1c            │ 7.2 %   │ < 5.7 % │   │
│  │ Status: 🟡 CAO   │         │         │   │
│  ├──────────────────┼─────────┼─────────┤   │
│  │ Total Cholesterol│ 215     │ < 200   │   │
│  │                  │ mg/dL   │ mg/dL   │   │
│  │ Status: 🟡 CAO   │         │         │   │
│  ├──────────────────┼─────────┼─────────┤   │
│  │ ALT (SGPT)       │ 45 U/L  │ 7-56 U/L│   │
│  │ Status: 🟢 BT    │         │         │   │
│  └──────────────────┴─────────┴─────────┘   │
│                                             │
│  ⚠️ Cần xác nhận: 0 trường                  │
│                                             │
│  [Upload lại]        [Xác nhận]             │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Confidence ≥ 85% | Tự động chấp nhận, KH xác nhận → chuyển bước 5 |
| Confidence < 85% | Highlight trường cần xác nhận, KH kiểm tra và sửa dữ liệu |
| Confidence < 90% | MD xác nhận thuật ngữ y khoa |
| Tài liệu không đọc được | Cho phép upload lại hoặc nhập thủ công |

#### ⚠️ Lỗi

- KH PHẢI xác nhận dữ liệu trích xuất trước khi tiếp tục
- KH có thể chỉnh sửa bất kỳ giá trị nào
- SLA: ≤ 30 giây, accuracy ≥ 95%
- OCR > 30 giây → auto alert kỹ thuật
- Hệ thống OCR sập > 10 phút → CS nhận alert, tiến hành review thủ công

---

### Bước 5. Hệ thống (AI) – Phân tích kết quả (SLA: ≤ 30s)

- **INPUT:** extracted_data (Object[], required — từ bước 4), ehr_summary (Object, required — từ bước 2), lab_info (Object, required — từ bước 3)
- **OUTPUT:** analysis_id (String), analysis_report (Object), severity_summary (Object: normal_count, attention_count, critical_count), patterns_detected (String[]), risk_assessment (String)

#### UI States

- **Processing:** 🔄 "Đang phân tích... Vui lòng chờ" + thanh tiến trình
- **Success:** Hiển thị tổng quan (🟢 Bình thường / 🟡 Cần chú ý / 🔴 Nguy hiểm) + các phát hiện chính + pattern nhận diện
- **Critical detected:** Cảnh báo khẩn cấp, khuyến nghị gặp bác sĩ ngay

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│        PHÂN TÍCH KẾT QUẢ XÉT NGHIỆM       │
├─────────────────────────────────────────────┤
│                                             │
│  🔄 Đang phân tích... Vui lòng chờ          │
│                                             │
│  ✅ Phân tích hoàn tất                       │
│                                             │
│  📊 TỔNG QUAN:                               │
│  ┌────────────────────────────────────┐     │
│  │  🟢 Bình thường: 12 chỉ số         │     │
│  │  🟡 Cần chú ý:   3 chỉ số         │     │
│  │  🔴 Nguy hiểm:   0 chỉ số         │     │
│  └────────────────────────────────────┘     │
│                                             │
│  📋 CÁC PHÁT HIỆN CHÍNH:                    │
│  ┌────────────────────────────────────┐     │
│  │ 🟡 Glucose (142 mg/dL)             │     │
│  │    Đường huyết lúc đói tăng cao     │     │
│  │                                    │     │
│  │ 🟡 HbA1c (7.2%)                    │     │
│  │    Trên mục tiêu kiểm soát tiểu    │     │
│  │    đường (mục tiêu < 7%)           │     │
│  │                                    │     │
│  │ 🟡 Cholesterol (215 mg/dL)          │     │
│  │    Cholesterol toàn phần cao giới   │     │
│  │    hạn                              │     │
│  └────────────────────────────────────┘     │
│                                             │
│  🔍 PATTERN NHẬN DIỆN:                       │
│  ┌────────────────────────────────────┐     │
│  │ ⚠️ Nguy cơ Hội chứng chuyển hóa    │     │
│  │    (Glucose cao + Cholesterol cao)  │     │
│  └────────────────────────────────────┘     │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Phân tích hoàn tất (không Critical) | Chuyển bước 6 |
| Phát hiện giá trị Nguy hiểm (🔴 Critical) | Alert MD ngay lập tức + vẫn chuyển bước 6 |

#### ⚠️ Lỗi

- Bước này hệ thống tự động, KH chỉ xem kết quả
- 3-tier severity: Normal / Attention / Critical
- Pattern recognition + cross-reference với EHR (lịch sử bệnh, thuốc đang dùng)
- Critical values → alert MD ngay lập tức
- Phân tích > 30 giây → auto cảnh báo kỹ thuật
- Engine AI sập > 10 phút → CS nhận alert, cảnh báo MD nếu có chỉ số nguy kịch, thông báo KH về sự chậm trễ

---

### Bước 6. Hệ thống (AI) – Tạo Care Plan (SLA: ≤ 60s)

- **INPUT:** analysis_report (Object, required — từ bước 5), ehr_summary (Object, required — từ bước 2), lab_info (Object, required — từ bước 3)
- **OUTPUT:** plan_id (String), consultation_plan (Object: summary, questions, recommendations, follow_up_schedule), path_suggestion (String: A/B)

#### UI States

- **Processing:** 🔄 "AI đang tạo Kế hoạch Chăm sóc cá nhân hóa cho bạn..." + thanh tiến trình (✅ Phân tích mức độ nghiêm trọng → ✅ Tạo khuyến nghị sơ bộ → 🔄 Lập kế hoạch theo dõi → ⬜ Xác định luồng xử lý)
- **Success:** Kế hoạch sẵn sàng, chuyển bước 7

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│     AI ĐANG LẬP KẾ HOẠCH CHĂM SÓC          │
├─────────────────────────────────────────────┤
│                                             │
│  🔄 AI đang tạo Kế hoạch Chăm sóc           │
│     cá nhân hóa cho bạn...                   │
│                                             │
│  ████████████████████░░░░░░░░  60%          │
│                                             │
│  ✅ Phân tích mức độ nghiêm trọng           │
│  ✅ Tạo khuyến nghị sơ bộ                    │
│  🔄 Lập kế hoạch theo dõi                    │
│  ⬜ Xác định luồng xử lý                    │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⚠️ Lỗi

- AI lập Kế hoạch cho CẢ 2 LUỒNG trước khi xác định luồng
- Kế hoạch bao gồm: tóm tắt tình trạng, khuyến nghị lối sống, chế độ ăn, lịch theo dõi, câu hỏi lâm sàng
- Bước tự động, KH không thao tác
- Tạo Kế hoạch > 60 giây → auto cảnh báo kỹ thuật
- Lỗi không tạo được Kế hoạch → auto retry, thông báo kỹ thuật
- MD periodic guideline review — đảm bảo phù hợp guideline y khoa mới nhất

---

### Bước 7. Hệ thống (AI) – Xác định luồng routing (SLA: ≤ 3s)

- **INPUT:** analysis_report (Object, required — từ bước 5), severity_summary (Object, required — từ bước 5)
- **OUTPUT:** selected_path (String: A/B), next_function (String)

#### UI States

- **Processing:** AI đang xác định hình thức tư vấn phù hợp...
- **Result - Luồng A:** Card "🤖 AI TƯ VẤN TỰ ĐỘNG" — ⏱️ 5-10 phút, 💰 Phí cơ bản
- **Result - Luồng B:** Card "👨‍⚕️ VIDEO CALL VỚI BÁC SĨ MỸ" — ⏱️ 30-50 phút + lịch, 💰 Phí cơ bản + $50-100
- **Auto-redirect:** Hệ thống tự động chuyển KH vào luồng phù hợp

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│       XÁC ĐỊNH HÌNH THỨC TƯ VẤN            │
├─────────────────────────────────────────────┤
│                                             │
│  ✅ AI đã phân tích và xác định hình thức    │
│     tư vấn phù hợp cho bạn:                 │
│                                             │
│  ┌────────────────────────────────────┐     │
│  │  🤖 AI TƯ VẤN TỰ ĐỘNG             │     │
│  │                                    │     │
│  │  Các chỉ số của bạn ở mức "Cần    │     │
│  │  chú ý" (không nguy hiểm), không  │     │
│  │  cần kê đơn thuốc mới.            │     │
│  │                                    │     │
│  │  ⏱️ Thời gian: 5-10 phút          │     │
│  │  💰 Chi phí: Phí cơ bản            │     │
│  └────────────────────────────────────┘     │
│                                             │
│  — HOẶC —                                   │
│                                             │
│  ┌────────────────────────────────────┐     │
│  │  👨‍⚕️ VIDEO CALL VỚI BÁC SĨ MỸ     │     │
│  │                                    │     │
│  │  Cần kê đơn thuốc, chỉ số nguy    │     │
│  │  hiểm, hoặc tình trạng phức tạp.  │     │
│  │                                    │     │
│  │  ⏱️ Thời gian: 30-50 phút + lịch  │     │
│  │  💰 Chi phí: Phí cơ bản + $50-100 │     │
│  └────────────────────────────────────┘     │
│                                             │
│  Hệ thống tự động chuyển bạn vào luồng      │
│  phù hợp...                                  │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Bình thường / Cần chú ý + không cần thuốc | → FLOW-A: Chuyển thẳng bước 8 (Gửi Kế hoạch) |
| Nguy hiểm / Critical / Cần kê đơn / Phức tạp | → FLOW-B: Chuyển bước B1 (Thanh toán) |
| Confidence < 80% | MD review trước khi thực thi |
| Không chắc chắn (edge case) | Mặc định → FLOW-B (safe approach) |

#### ⚠️ Lỗi

- AI TỰ ĐỘNG xác định luồng — KH KHÔNG chọn
- Ghi rõ lý do phân luồng vào log (để audit)
- Phân luồng > 3 giây → auto alert kỹ thuật
- Confidence < 80% → auto flag để MD review trước khi thực thi
- KH khiếu nại phân luồng sai → CS giải thích lý do, hỗ trợ đặt lịch Luồng B nếu KH đồng ý

---

## FLOW-A: AI CARE PLAN TRỰC TIẾP

- Không có bước bổ sung — AI gửi Care Plan cho KH ngay
- Kế hoạch AI đã lập ở bước 6 được chuyển **thẳng** sang bước chung "Gửi Kế hoạch" (bước 8)
- **KHÔNG** có bước tư vấn, **KHÔNG** có MD review, **KHÔNG** có phê duyệt

→ Chuyển sang Phần chung cuối (bước 8-9)

---

## FLOW-B: MD US REVIEW + VIDEO CALL

### Bước B1. Khách hàng – Thanh toán ($50-100, PCI-DSS) + Đặt lịch Video Call

- **INPUT:** payment_method (Object, required), preferred_date (Date, required), preferred_time (Time, required), timezone (String, required)
- **OUTPUT:** booking_id (String), payment_id (String), appointment (Object: date, time, md_name, meeting_link), meeting_link (String)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Ngày hẹn | Trong 7 ngày tới, tối thiểu 2 giờ từ hiện tại | "Vui lòng chọn ngày trong 7 ngày tới" |
| Khung giờ | Phải có slot trống | "Khung giờ này không còn trống" |
| Thông tin thẻ | Thẻ hợp lệ | "Thông tin thẻ không hợp lệ" |
| Thanh toán | Thanh toán thành công | "Thanh toán không thành công. Vui lòng thử lại" |

#### UI States

- **Default:** Hiển thị danh sách dịch vụ ($50-100), lịch khả dụng real-time, form thanh toán
- **Selecting date:** Calendar hiển thị các ngày có slot trống (7 ngày tới)
- **Selecting time:** Danh sách khung giờ khả dụng theo múi giờ KH
- **Processing payment:** Spinner "Đang xử lý thanh toán..."
- **Payment success:** "Thanh toán thành công. Lịch hẹn: [ngày/giờ]"
- **Payment fail:** "Thanh toán chưa thành công. Vui lòng thử lại hoặc liên hệ hỗ trợ."
- **Idle > 2 phút:** Popup nhắc nhở: "Hoàn tất đặt lịch để tiếp tục"

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│     THANH TOÁN & ĐẶT LỊCH VIDEO CALL       │
├─────────────────────────────────────────────┤
│                                             │
│  📋 DỊCH VỤ: Video Call với Bác sĩ Mỹ       │
│  ┌────────────────────────────────────┐     │
│  │ ✅ Tư vấn trực tiếp 30 phút        │     │
│  │ ✅ Phân tích kết quả XN chi tiết    │     │
│  │ ✅ Kê đơn thuốc (nếu phù hợp)      │     │
│  │ ✅ Kế hoạch Chăm sóc cá nhân hóa   │     │
│  │ ✅ Hỗ trợ 7 ngày sau tư vấn         │     │
│  └────────────────────────────────────┘     │
│                                             │
│  📅 CHỌN NGÀY:                               │
│  ┌──────────────────────────────────────┐   │
│  │  16/01 (T5)  17/01 (T6)  19/01 (CN) │   │
│  │     ✅                               │   │
│  └──────────────────────────────────────┘   │
│                                             │
│  ⏰ CHỌN GIỜ:                                │
│  ┌──────────────────────────────────────┐   │
│  │  [09:00]  [10:00]  [14:00]  [15:00] │   │
│  │     ✅                               │   │
│  └──────────────────────────────────────┘   │
│  Múi giờ: Asia/Ho_Chi_Minh (UTC+7)         │
│                                             │
│  💳 THANH TOÁN:                              │
│  ┌──────────────────────────────────────┐   │
│  │ Số thẻ: [____ ____ ____ ____]        │   │
│  │ MM/YY:  [__/__]   CVC: [___]        │   │
│  └──────────────────────────────────────┘   │
│                                             │
│  🏷️ Mã giảm giá (nếu có): [___________]    │
│                                             │
│  ┌────────────────────────────────────┐     │
│  │ Tổng cộng:           $99.00 USD    │     │
│  └────────────────────────────────────┘     │
│                                             │
│  [     Thanh toán $99.00     ]              │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Thanh toán thành công + có slot | Xác nhận lịch, gửi thông báo 3 kênh (App + Email + SMS), chuyển bước B2 |
| Thanh toán thất bại lần 1 | Cho phép thử lại hoặc đổi phương thức |
| Thanh toán thất bại sau 2 lần | CS hỗ trợ thanh toán |
| Không có slot trống | CS hỗ trợ đặt lịch hoặc đưa vào danh sách chờ |

#### ⚠️ Lỗi

- Thanh toán qua cổng bảo mật PCI-DSS, không lưu thông tin thẻ
- Tất cả thời gian hiển thị theo múi giờ KH (timezone-aware)
- Hủy lịch miễn phí trước 24 giờ
- Sau thanh toán: gửi xác nhận qua Email + SMS + App đồng thời 3 kênh
- Thanh toán fail sau 2 lần retry → CS nhận alert, gọi điện hỗ trợ KH
- KH muốn thanh toán chuyển khoản trực tiếp → CS hỗ trợ thủ công

---

### Bước B2. MD US + Khách hàng – Video Call tư vấn (20-30 phút)

- **INPUT:** booking_id (String, required), analysis_report (Object, required — từ bước 5), ehr_summary (Object, required — từ bước 2), consultation_plan (Object, required — từ bước 6)
- **OUTPUT:** visit_id (String), visit_notes (Object), prescription (Object, optional), recording_url (String)

#### UI States

- **Pre-call (15 phút trước):** Nhắc nhở KH + MD US qua SMS/App/Email
- **Waiting:** "Đang chờ bác sĩ tham gia..."
- **Connected:** Video call 2 chiều, REC indicator, timer, controls (tắt tiếng, tắt camera, chia sẻ MH, chat, kết thúc)
- **Connection issue:** Tự động gợi ý "Chuyển sang audio only?"
- **Reconnecting:** "Đang kết nối lại..." (tự động retry)
- **Completed:** "Cuộc tư vấn đã kết thúc. Bác sĩ đang chuẩn bị Kế hoạch Chăm sóc..."

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│         PHÒNG TƯ VẤN VIDEO                  │
├─────────────────────────────────────────────┤
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │                                     │    │
│  │         [VIDEO — MD US]             │    │
│  │     Dr. John Smith, MD              │    │
│  │     Internal Medicine               │    │
│  │                                     │    │
│  │            ┌──────────┐             │    │
│  │            │ [VIDEO   │             │    │
│  │            │  — KH]   │             │    │
│  │            └──────────┘             │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  🔴 REC    ⏱️ 28:45                         │
│                                             │
│  [🔇 Tắt tiếng] [📷 Tắt camera]            │
│  [🖥️ Chia sẻ MH] [💬 Chat]                 │
│  [🔴 Kết thúc]                              │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Video call hoàn tất (20-30 phút) | Chuyển bước B3 |
| KH no-show (không tham gia) | Chờ 10 phút, gửi nhắc nhở. Vẫn no-show → đánh dấu, cho phép đặt lại |
| MD US trễ > 5 phút | CS alert, gọi điện KH xin lỗi |
| Lỗi kết nối > 2 phút | CS hỗ trợ kỹ thuật, đặt lại lịch nếu cần |

#### ⚠️ Lỗi

- Cấu trúc cuộc gọi 30 phút: Giới thiệu (2-3') → Review kết quả XN (5-7') → Phỏng vấn lâm sàng (8-10') → Đánh giá & Kế hoạch (8-10') → Kết luận (2-3')
- MD US xác minh danh tính KH ở đầu cuộc gọi
- PHẢI có sự đồng ý ghi hình trước khi bắt đầu
- E-prescribing nếu cần (KHÔNG kê thuốc kiểm soát đặc biệt qua telemedicine)
- Recording encrypted, lưu trữ 90 ngày
- Nhắc nhở MD: 30 phút và 15 phút trước cuộc gọi
- Test link video tự động 10 phút trước + link dự phòng
- Kết nối yếu → tự động gợi ý chuyển audio only
- Gián đoạn kết nối → auto reconnect (tối đa 3 lần)
- MD trễ > 5 phút → CS alert, gọi điện KH
- Gián đoạn kéo dài > 2 phút → CS hỗ trợ, reschedule nếu cần

---

### Bước B3. MD US – Chỉnh sửa + Phê duyệt Care Plan (SLA: ≤ 10 phút sau call)

- **INPUT:** care_plan (Object, required — bản nháp từ bước 6)
- **OUTPUT:** approval_id (String), signature (Object: md_name, timestamp, e_signature), care_plan_status (String: approved)

#### UI States

- **Review:** Hiển thị Kế hoạch AI với các section có nút [✏️ Sửa]: Tóm tắt tình trạng, Kết quả XN chi tiết, Kế hoạch điều trị, Hướng dẫn dùng thuốc, Khuyến nghị lối sống, Lịch theo dõi
- **Editing:** Inline editing, tự động highlight phần MD sửa (version control)
- **Checklist:** 6 mục checklist phê duyệt (tóm tắt đúng, chẩn đoán chính xác, kế hoạch điều trị đúng, đơn thuốc chính xác, không chống chỉ định bỏ sót, tuân thủ HIPAA)
- **Signing:** Tích hợp e-signature (DocuSign/Adobe Sign)
- **Signed:** Kế hoạch được KHÓA — không chỉnh sửa được
- **Overdue (> 10 phút):** Hệ thống gửi reminder cho MD
- **Overdue (> 30 phút):** CS nhắc MD. Hệ thống nhắc ở 30 phút và 60 phút

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│     PHÊ DUYỆT KẾ HOẠCH CHĂM SÓC           │
│     (Chỉ MD US — Giao diện bác sĩ)          │
├─────────────────────────────────────────────┤
│                                             │
│  📋 KẾ HOẠCH CHĂM SÓC — Bản nháp AI        │
│                                             │
│  ▼ Tóm tắt tình trạng          [✏️ Sửa]    │
│  ▼ Kết quả XN chi tiết         [✏️ Sửa]    │
│  ▼ Kế hoạch điều trị           [✏️ Sửa]    │
│  ▼ Hướng dẫn dùng thuốc        [✏️ Sửa]    │
│  ▼ Khuyến nghị lối sống        [✏️ Sửa]    │
│  ▼ Lịch theo dõi               [✏️ Sửa]    │
│                                             │
│  ✅ CHECKLIST PHÊ DUYỆT:                    │
│  ☑️ Tóm tắt phản ánh đúng tư vấn            │
│  ☑️ Chẩn đoán chính xác                     │
│  ☑️ Kế hoạch điều trị đúng                   │
│  ☑️ Đơn thuốc chính xác (nếu có)            │
│  ☑️ Không có chống chỉ định bỏ sót          │
│  ☑️ Tuân thủ HIPAA                           │
│                                             │
│  🔐 KÝ SỐ:                                  │
│  Tôi xác nhận Kế hoạch Chăm sóc này phản   │
│  ánh chính xác tư vấn của tôi với bệnh nhân │
│                                             │
│  [     ✅ Phê duyệt & Ký số     ]           │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| MD phê duyệt + ký số thành công | Kế hoạch KHÓA, chuyển bước 8 |
| MD chỉnh sửa Kế hoạch | Lưu 2 version (AI gốc + MD đã duyệt), ghi nhật ký thay đổi |
| MD phát hiện lỗi nghiêm trọng | Yêu cầu AI tạo lại Kế hoạch |
| MD > 30 phút chưa approve | CS nhắc, hệ thống nhắc ở 30 phút và 60 phút |

#### ⚠️ Lỗi

- CHỈ bác sĩ đã thực hiện video call mới được phê duyệt
- Sau khi ký số → Kế hoạch được KHÓA (không ai chỉnh sửa được)
- PHẢI hoàn tất phê duyệt trong 2 giờ
- Lưu song song 2 version: bản AI gốc + bản MD đã duyệt
- Lỗi e-signature → auto alert IT + thông báo MD thử lại
- MD > 30 phút chưa approve → CS escalate cho MD Manager

→ Chuyển sang Phần chung cuối (bước 8-9)

---

## PHẦN CHUNG CUỐI (Bước 8-9)

### Bước 8. Hệ thống – Gửi Care Plan cho KH qua 3 kênh

- **INPUT:** care_plan (Object, required — Luồng A: bản AI gốc / Luồng B: bản MD đã duyệt)
- **OUTPUT:** delivery_id (String), channels (Object: email_status, app_status, sms_status), pdf_url (String)

#### UI States

- **Sending:** Đang gửi Kế hoạch Chăm sóc...
- **Success:** ✅ "Kế hoạch Chăm sóc của bạn đã sẵn sàng!" + 📧 Đã gửi qua Email + 📱 Đã gửi thông báo trên app + 📲 Đã gửi SMS
- **Partial:** Gửi thành công ít nhất 1 kênh, retry các kênh còn lại
- **Error:** Delivery thất bại sau 3 lần retry → CS hỗ trợ gửi thủ công

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│     KẾ HOẠCH CHĂM SÓC ĐÃ SẴN SÀNG         │
├─────────────────────────────────────────────┤
│                                             │
│  ✅ Kế hoạch Chăm sóc của bạn đã sẵn sàng!  │
│                                             │
│  ┌────────────────────────────────────┐     │
│  │ 📋 Kế hoạch Chăm sóc               │     │
│  │                                    │     │
│  │ 📅 Ngày: 14/01/2026                │     │
│  │ 👨‍⚕️ Bác sĩ: Dr. John Smith         │     │
│  │ 📊 Loại: [AI / Video Call]         │     │
│  │                                    │     │
│  │ [Xem Care Plan]  [Tải PDF]         │     │
│  │                  [Chia sẻ]         │     │
│  └────────────────────────────────────┘     │
│                                             │
│  📧 Đã gửi qua Email                        │
│  📱 Đã gửi thông báo trên app               │
│  📲 Đã gửi SMS                              │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Gửi thành công ≥ 1 kênh | Chuyển bước 9 |
| Gửi thất bại 1-2 kênh | Auto retry 3 lần cho kênh lỗi |
| Gửi thất bại tất cả kênh sau 3 lần retry | CS hỗ trợ gửi thủ công |

#### ⚠️ Lỗi

- Gửi qua tất cả kênh khả dụng: Email (PDF mã hóa) + App push + SMS
- PDF mã hóa HIPAA, link tải có thời hạn 30 ngày
- SMS chỉ chứa link, không chứa thông tin y tế
- KH có thể yêu cầu gửi lại bất kỳ lúc nào
- Phải có ít nhất 1 kênh gửi thành công
- Tracking delivery status: App (delivered/read), Email (sent/delivered/opened)
- Delivery fail sau 3 lần → CS gửi thủ công, hướng dẫn KH truy cập trong mục "Hồ sơ sức khỏe" trên App

---

### Bước 9. Hệ thống – Cập nhật EHR + Lên lịch follow-up

- **INPUT:** care_plan (Object, required), session_data (Object, required)
- **OUTPUT:** update_id (String), encounter_id (String), reminders_created (Object[]), session_status (String: closed)

#### UI States

- **Updating:** ✅ Đang cập nhật hồ sơ y khoa...
- **Success:** Hiển thị danh sách đã cập nhật (Kết quả XN → Hồ sơ y khoa, Kế hoạch Chăm sóc → Đính kèm, Đơn thuốc nếu có → Danh sách thuốc, Chẩn đoán → Danh sách bệnh lý) + Lịch theo dõi (XN lại, tái khám, tái kê đơn) với ngày hạn và ngày nhắc nhở
- **Completed:** ✅ HOÀN TẤT + nút [Về Trang chủ]

#### Wireframe ASCII

```
┌─────────────────────────────────────────────┐
│     CẬP NHẬT HỒ SƠ & LỊCH THEO DÕI        │
├─────────────────────────────────────────────┤
│                                             │
│  ✅ Hồ sơ y khoa đã được cập nhật            │
│                                             │
│  📋 ĐÃ CẬP NHẬT:                            │
│  ✅ Kết quả XN mới → Hồ sơ y khoa           │
│  ✅ Kế hoạch Chăm sóc → Đính kèm            │
│  ✅ Đơn thuốc (nếu có) → Danh sách thuốc    │
│  ✅ Chẩn đoán → Danh sách bệnh lý           │
│                                             │
│  📅 LỊCH THEO DÕI:                          │
│  ┌────────────────────────────────────┐     │
│  │ 🔬 XN lại HbA1c + Glucose          │     │
│  │    Hạn: 14/04/2026                 │     │
│  │    Nhắc nhở: 07/04/2026            │     │
│  │                                    │     │
│  │ 👨‍⚕️ Tái khám với bác sĩ             │     │
│  │    Hạn: 14/04/2026                 │     │
│  │    Nhắc nhở: 31/03/2026            │     │
│  │                                    │     │
│  │ 💊 Tái kê đơn Metformin            │     │
│  │    Hạn: 14/03/2026                 │     │
│  │    Nhắc nhở: 07/03/2026            │     │
│  └────────────────────────────────────┘     │
│                                             │
│  [    Về Trang chủ    ]                     │
│                                             │
│  ✅ HOÀN TẤT                                │
│                                             │
└─────────────────────────────────────────────┘
```

#### ⚠️ Lỗi

- Hệ thống tự động thực hiện, KH không cần thao tác
- Lịch nhắc nhở gửi trước ngày hẹn: XN lại (7 ngày), tái khám (14 ngày), thuốc (7 ngày)
- KH có thể quản lý (bật/tắt) nhắc nhở trong cài đặt app
- Nếu Luồng B có đơn thuốc → cập nhật vào danh sách thuốc đang dùng
- Cập nhật EHR phải là atomic (toàn bộ hoặc không có gì)
- Cập nhật EHR fail → auto alert kỹ thuật + retry
- Lịch theo dõi không được tạo → auto alert kỹ thuật
- MD periodic audit — review định kỳ các case đã Closed

---

## XỬ LÝ TRƯỜNG HỢP ĐẶC BIỆT

| Trường hợp | Xử lý |
|------------|--------|
| File không đọc được (chất lượng thấp) | Thông báo lỗi, yêu cầu KH upload bản scan chất lượng cao hơn |
| File không đúng định dạng (khác PDF/JPG/PNG) | Thông báo lỗi, hướng dẫn KH upload đúng định dạng |
| File chứa mã độc | Từ chối file, cách ly, thông báo KH kiểm tra lại |
| OCR confidence thấp (< 85%) | Đánh dấu trường cần xác nhận, yêu cầu KH kiểm tra và sửa dữ liệu |
| Critical values | Alert MD ngay lập tức, khuyến nghị KH gặp bác sĩ ngay, tự động xếp vào Luồng B |
| Bệnh nhân mới (ít EHR) | Tạo hồ sơ EHR mới, AI phân tích không có dữ liệu so sánh lịch sử, recommend FLOW-B |
| Payment thất bại | Auto-retry, sau 2 lần → CS hỗ trợ |
| No-show (Video Call) | Chờ 10 phút, gửi nhắc nhở. Nếu vẫn no-show → đánh dấu, cho phép đặt lại, áp dụng chính sách hoàn tiền |
| Lỗi kỹ thuật Video Call | Tự động kết nối lại (tối đa 3 lần), chuyển sang âm thanh, hoặc đặt lại lịch |
| Phát hiện tình huống cấp cứu | MD US hướng dẫn KH gọi cấp cứu ngay, ghi nhật ký khẩn cấp |
| Hệ thống kê đơn không khả dụng | MD US ghi chú đơn thuốc, theo dõi kê đơn thủ công sau |

---

## BẢNG THÔNG BÁO TỰ ĐỘNG

| Sự kiện | Kênh | Người nhận |
|---------|------|------------|
| Care Plan sẵn sàng | Email (PDF) + Push + SMS | KH |
| Xác nhận lịch Video Call (Flow B) | Email + SMS + App | KH + MD US |
| Nhắc nhở Video Call 15 phút trước | SMS + App | KH |
| Follow-up reminder | Push | KH |
| MD cần review (Flow B) | Alert nội bộ | MD US |
| Critical values detected | Alert nội bộ | MD |

### Template thông báo chi tiết

#### 📧 Email — Kế hoạch Chăm sóc đã sẵn sàng

- **Subject:** Care Plan của bạn đã sẵn sàng — CVH Healthcare
- **Người nhận:** Khách hàng (email đã đăng ký)
- **Thời điểm:** Ngay sau khi Kế hoạch Chăm sóc được gửi (bước 8)
- **Body:**

```
Xin chào [Tên KH],

Care Plan của bạn đã được [bác sĩ [Tên MD US] phê duyệt / AI tạo tự động]
và sẵn sàng để xem.

📋 Tóm tắt:
- Ngày tư vấn: [DD/MM/YYYY]
- Bác sĩ: [Tên MD US] (nếu Luồng B)
- Loại tư vấn: [AI tự động / Video Call]

📎 Đính kèm: Care_Plan_[DD_MM_YYYY].pdf

🔐 Bạn cũng có thể xem Care Plan trong ứng dụng CVH Healthcare.

Nếu có câu hỏi, vui lòng liên hệ: support@cvh.healthcare

Trân trọng,
CVH Healthcare Team
```

#### 📱 SMS — Thông báo Kế hoạch

- **Nội dung:** `[CVH] Care Plan của bạn đã sẵn sàng. Xem tại: cvh.healthcare/cp/[short_code] hoặc trong ứng dụng CVH.`
- **Người nhận:** Khách hàng (SĐT đã đăng ký)
- **Thời điểm:** Ngay sau khi Kế hoạch Chăm sóc được gửi

#### 📱 Push Notification — Kế hoạch sẵn sàng

- **Title:** Care Plan đã sẵn sàng!
- **Body:** Care Plan của bạn đã sẵn sàng! Tap để xem.
- **Người nhận:** Khách hàng (app)
- **Thời điểm:** Ngay sau khi Kế hoạch Chăm sóc được gửi

#### 📧 Email — Xác nhận lịch hẹn Video Call (Chỉ Luồng B)

- **Subject:** Xác nhận lịch hẹn Video Call — CVH Healthcare
- **Người nhận:** Khách hàng + MD US
- **Thời điểm:** Ngay sau khi thanh toán + đặt lịch thành công (bước B1)
- **Body:**

```
Xin chào [Tên KH],

Lịch hẹn Video Call của bạn đã được xác nhận:

📅 Ngày: [DD/MM/YYYY]
⏰ Giờ: [HH:MM] ([Múi giờ])
👨‍⚕️ Bác sĩ: [Tên MD US], [Chuyên khoa]
⏱️ Thời lượng: 30 phút
💰 Đã thanh toán: $[XX] USD

🔗 Link tham gia: [Video Call Link]

📋 Lưu ý:
- Vui lòng tham gia đúng giờ
- Kiểm tra camera/microphone trước cuộc gọi
- Chuẩn bị câu hỏi muốn hỏi bác sĩ

Hủy/Đổi lịch miễn phí trước 24 giờ.

Trân trọng,
CVH Healthcare Team
```

#### 📱 SMS — Nhắc nhở Video Call (Chỉ Luồng B)

- **Nội dung:** `[CVH] Nhắc nhở: Video Call với Dr. [Tên] lúc [HH:MM] hôm nay. Link: [URL]`
- **Người nhận:** Khách hàng
- **Thời điểm:** 15 phút trước giờ hẹn

#### 📱 Push Notification — Nhắc nhở theo dõi

- **Title:** Nhắc nhở theo dõi sức khỏe
- **Body:** Đã đến lúc [xét nghiệm lại / tái khám / tái kê đơn]. Xem chi tiết.
- **Người nhận:** Khách hàng (app)
- **Thời điểm:** Theo lịch nhắc nhở đã thiết lập (bước 9)

---

## BẢNG GIÁM SÁT CS-MD ĐẦY ĐỦ

### Tổng quan

- **Hệ thống auto ~95%:** Upload, EHR retrieval, AI OCR, AI analysis, AI Care Plan, AI routing, gửi Care Plan, cập nhật EHR
- **CS can thiệp (chỉ Fatal < 5%):** DB sập > 10 phút, payment thất bại 2 lần, không có slot lịch, MD trễ > 5 phút, lỗi kết nối video > 2 phút, delivery thất bại 3 lần, MD > 30 phút chưa approve
- **MD can thiệp:** OCR confidence < 90% (xác nhận thuật ngữ), critical value alerts, routing confidence < 80%, video call tư vấn (Flow B), approve Care Plan (Flow B), periodic guideline review + audit

### 12 trạm giám sát chi tiết

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| 1. Upload kết quả XN | Validate real-time (PDF/JPG/PNG, ≤10MB), mã hóa AES-256 ngay | 🤖 100%: Validate format/size, quét malware, mã hóa HIPAA, retry 3 lần (cách 2s) | DB sập > 10 phút (tự phục hồi thất bại) → gọi KH, kênh dự phòng, báo IT | ❌ |
| 2. Truy xuất EHR | Tải ≤ 3s, match chính xác KH ID, log HIPAA | 🤖 100%: Double-check KH ID, load 4 phần (bệnh án, thuốc, dị ứng, XN cũ), retry 2 lần | EHR load fail sau 2 retry / hiển thị nhầm dữ liệu KH → escalate IT, HIPAA Breach report | ❌ |
| 3. Nhập thông tin lab | Validate real-time (ngày XN, tên lab, lý do), auto-save | 🤖 100%: Placeholder rõ ràng, push notification nhắc nhở > 5 phút, highlight trường lỗi | Hệ thống sập > 10 phút / dữ liệu bị mất → hỗ trợ nhập lại qua kênh dự phòng | ❌ |
| 4. AI OCR trích xuất | OCR ≤ 30s, accuracy ≥ 95%, nhận diện format lab | 🤖 100%: Trích xuất tự động, cải thiện chất lượng ảnh, flag confidence < 90% | OCR accuracy < 95% do lỗi engine / OCR sập > 10 phút → review thủ công, báo IT | Confidence < 90%: xác nhận thuật ngữ y khoa |
| 5. AI phân tích | Phân tích ≤ 30s, 3-tier severity, cross-reference EHR | 🤖 100%: So sánh reference range theo tuổi/giới, phân tích xu hướng, flag Critical | Engine AI sập > 10 phút → cảnh báo MD, thông báo KH chậm trễ | Critical value alerts → review + double-check phân loại |
| 6. AI tạo Care Plan | Tạo ≤ 60s, đầy đủ 3 phần (giải thích, khuyến nghị, lịch theo dõi) | 🤖 100%: Tạo Kế hoạch 3 phần, format PDF, lưu version audit, retry khi lỗi | Lỗi không tạo được sau nhiều retry → thông báo KH chậm trễ, báo IT | Periodic guideline review — cập nhật template khi có guideline mới |
| 7. AI xác định luồng | Phân luồng ≤ 3s, ghi rõ lý do (audit) | 🤖 100%: Áp dụng bộ quy tắc (A: BT/Chú ý, B: Nguy hiểm/Kê đơn), flag confidence < 80% | KH khiếu nại phân luồng sai → giải thích lý do, hỗ trợ đặt lịch B | Confidence < 80%: review + xác nhận luồng. Audit định kỳ |
| 8. Thanh toán + Đặt lịch (B) | PCI-DSS, hiển thị real-time availability, xác nhận 3 kênh | 🤖 100%: Cổng thanh toán bảo mật, retry 2 lần (cách 5s), gửi xác nhận App+Email+SMS | Payment fail 2x / no slots / thanh toán ngoài hệ thống → gọi KH hỗ trợ | ❌ |
| 9. MD US Video Call (B) | MD đúng giờ (≤ 5' trễ), kết nối ≤ 10s, tư vấn 20-30 phút, ghi hình | 🤖 95%: Nhắc nhở MD 30'+15', chuẩn bị tài liệu sẵn, test link 10' trước, giám sát chất lượng real-time, auto reconnect | MD trễ > 5' / lỗi kết nối > 2' / MD không tham gia → gọi KH xin lỗi, reschedule | MD US tư vấn: review tài liệu, giải thích XN, trả lời câu hỏi, kê đơn nếu cần |
| 10. MD US chỉnh sửa + Approve (B) | Approve ≤ 10 phút sau call, e-signature hợp lệ | 🤖 95%: Hiển thị Kế hoạch AI ngay sau call, inline editing, e-signature tích hợp, reminder 10'+30'+60' | MD > 30' chưa approve → escalate MD Manager | MD US review toàn bộ, chỉnh sửa nếu cần, ký số phê duyệt |
| 11. Gửi Care Plan | Gửi ≤ 5s, đúng phiên bản (AI/MD), mã hóa HIPAA | 🤖 100%: Xác định đúng version, mã hóa PDF, gửi song song 2 kênh (app+email), retry 3 lần | Gửi fail 3 lần (cả App và Email) / KH khiếu nại → gửi thủ công | ❌ |
| 12. Cập nhật EHR + Follow-up | Cập nhật ≤ 5s, tạo lịch theo dõi đúng Kế hoạch | 🤖 100%: Cập nhật EHR atomic, tạo lịch + nhắc nhở tự động, đánh dấu case Closed | Lỗi cập nhật EHR hoàn toàn → cập nhật thủ công, hỗ trợ KH | MD periodic audit — review chất lượng Kế hoạch định kỳ |

---

## DATA SPECIFICATION

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| upload_id | String | Auto | UUID | Mã upload session |
| file_ids | String[] | Auto | UUID[] | Danh sách mã file đã upload |
| lab_result_file | File[] | Yes | PDF/JPG/PNG, max 10MB/file, max 10 files | File kết quả xét nghiệm |
| user_id | UUID | Auto | UUID | Mã khách hàng (từ phiên đăng nhập) |
| ehr_summary | Object | Auto | {medical_history, medications, allergies, previous_labs} | Tóm tắt EHR |
| lab_name | String | Yes | 2-100 ký tự | Tên phòng lab |
| test_date | Date | Yes | Không tương lai, trong 1 năm qua | Ngày xét nghiệm |
| reason_for_test | String | Yes | ≥ 10 ký tự | Lý do khám / triệu chứng |
| additional_notes | String | No | ≤ 1000 ký tự | Ghi chú bổ sung |
| referring_doctor | String | No | ≤ 100 ký tự | Bác sĩ giới thiệu |
| lab_info_id | String | Auto | UUID | Mã thông tin lab đã lưu |
| extraction_id | String | Auto | UUID | Mã trích xuất OCR |
| extracted_data | Object[] | Auto | [{parameter_name, value, unit, reference_range, status}] | Dữ liệu XN đã trích xuất |
| confidence_score | Number | Auto | 0-100% | Độ tin cậy OCR |
| analysis_id | String | Auto | UUID | Mã phân tích |
| analysis_report | Object | Auto | {findings, severity_classification, patterns, risk_assessment} | Báo cáo phân tích AI |
| severity_summary | Object | Auto | {normal_count, attention_count, critical_count} | Tóm tắt mức độ nghiêm trọng |
| patterns_detected | String[] | Auto | — | Patterns nhận diện được |
| risk_assessment | String | Auto | — | Đánh giá risk tổng thể |
| plan_id | String | Auto | UUID | Mã Kế hoạch Chăm sóc |
| consultation_plan | Object | Auto | {summary, questions, recommendations, follow_up_schedule} | Kế hoạch Chăm sóc |
| selected_path | String | Auto | A / B | Luồng được AI chọn |
| payment_method | Object | Yes (B) | PCI-DSS compliant | Phương thức thanh toán |
| preferred_date | Date | Yes (B) | Trong 7 ngày tới, ≥ 2h từ hiện tại | Ngày hẹn video call |
| preferred_time | Time | Yes (B) | Slot trống | Giờ hẹn video call |
| timezone | String | Yes (B) | IANA timezone | Múi giờ KH |
| booking_id | String | Auto (B) | UUID | Mã đặt lịch |
| payment_id | String | Auto (B) | UUID | Mã thanh toán |
| meeting_link | String | Auto (B) | HTTPS URL | Link video call |
| visit_id | String | Auto (B) | UUID | Mã lượt tư vấn |
| visit_notes | Object | Auto (B) | {summary, clinical_findings, recommendations} | Ghi chú tư vấn MD |
| prescription | Object | No (B) | {medications[], pharmacy, e_prescribe_id} | Đơn thuốc điện tử |
| recording_url | String | Auto (B) | HTTPS URL, encrypted, 90 ngày | Link recording video call |
| approval_id | String | Auto (B) | UUID | Mã phê duyệt |
| signature | Object | Auto (B) | {md_name, timestamp, e_signature} | Chữ ký số MD |
| care_plan_status | String | Auto | draft / approved / delivered / closed | Trạng thái Kế hoạch |
| delivery_id | String | Auto | UUID | Mã gửi Kế hoạch |
| pdf_url | String | Auto | HTTPS URL, mã hóa HIPAA, hạn 30 ngày | Link tải PDF Care Plan |
| update_id | String | Auto | UUID | Mã cập nhật EHR |
| encounter_id | String | Auto | UUID | Mã encounter |
| reminders_created | Object[] | Auto | [{type, due_date, reminder_date, channel}] | Danh sách nhắc nhở |
| session_status | String | Auto | active / closed | Trạng thái session |

---

## NGUỒN TÀI LIỆU GỐC

- Luồng nghiệp vụ: `docs/02-product/ba-specs/service-6-Lab_Results_Consultation/base/Lab_Results_Consultation_Touchpoint_Flow_v1.md`
- Checklist CS & MD: `docs/02-product/ba-specs/service-6-Lab_Results_Consultation/checklist-cs-md/checklist-cs-md.md`
- Bản tóm tắt: `docs/02-product/tomtat-chitiet/tomtat/service-6-Lab_Results_Consultation-tomtat.md`
