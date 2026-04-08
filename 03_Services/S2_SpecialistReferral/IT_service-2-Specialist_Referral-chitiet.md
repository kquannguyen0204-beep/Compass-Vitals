# CHI TIẾT LUỒNG NGHIỆP VỤ – SERVICE 2 (Specialist_Referral): GIỚI THIỆU CHUYÊN GIA (SPECIALIST REFERRAL)

## Header

- **Service:** Specialist Referral (Giới thiệu Chuyên gia)
- **Version:** 1.0 – Chi tiết
- **Input:** US MD tạo yêu cầu giới thiệu chuyên gia cho bệnh nhân
- **Output:** Bệnh nhân được khám chuyên gia, kết quả cập nhật vào EMR, MD review
- **Điều kiện sử dụng:** Tài khoản CVH active, có gói dịch vụ (VIP hoặc Standard)

---

## Nguyên tắc cốt lõi

- Phân luồng theo gói dịch vụ: **VIP** (CS/MA hỗ trợ toàn bộ) vs **Standard** (tự phục vụ)
- VIP: MA chuẩn bị hồ sơ, liên hệ chuyên gia, đặt lịch, follow-up
- Standard: Hệ thống gửi danh sách chuyên gia, KH tự đặt lịch, tự report kết quả
- Referral Letter tự động compile 13 sections từ EMR
- Truyền hồ sơ qua 3 kênh: Secure Fax (SRFax API - HIPAA), Secure Email, EHR Portal
- 95% automation, CS chỉ can thiệp khi fatal

---

## Danh sách Flow

| Flow ID | Tên luồng | Áp dụng |
|---------|-----------|---------|
| FLOW-01 | Nhận & Phân loại yêu cầu | Chung |
| FLOW-02-VIP | Chuẩn bị hồ sơ | VIP |
| FLOW-02-STD | Gửi danh sách chuyên gia | Standard |
| FLOW-03-VIP | Truyền hồ sơ cho chuyên gia | VIP |
| FLOW-04-VIP | Đặt lịch hẹn | VIP |
| FLOW-05 | Follow-up & Nhận kết quả | Chung (VIP: MA làm, STD: KH tự làm) |
| FLOW-06 | Cập nhật EMR & MD Review | Chung |

---

## FLOW-01: NHẬN & PHÂN LOẠI YÊU CẦU

### Bước 1: US MD – Tạo yêu cầu Referral từ MD Dashboard

Bác sĩ chọn chuyên khoa, lý do giới thiệu, mức độ khẩn cấp, ghi chú lâm sàng.

- INPUT: patient_id (UUID, required), specialty_type (String, required), referral_reason (String, required – min 20 ký tự, max 500), urgency_level (String: "Routine"/"Urgent", required)
- OUTPUT: referral_id (UUID), created_at (DateTime)

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| specialty_type | Bắt buộc, phải chọn từ danh sách | "Vui lòng chọn chuyên khoa" |
| referral_reason | Bắt buộc, tối thiểu 20 ký tự, tối đa 500 ký tự | "Lý do chuyển khoa phải có ít nhất 20 ký tự" |
| urgency_level | Bắt buộc, một trong hai: "Routine" hoặc "Urgent" | "Vui lòng chọn mức độ ưu tiên" |

**UI States:**

| State | Hiển thị |
|-------|----------|
| Default | Form trống, các trường bắt buộc có dấu * |
| Validation | Highlight đỏ các trường chưa điền hoặc không hợp lệ |
| Submitting | Nút "Tạo yêu cầu" disabled, hiển thị loading spinner |
| Success | Toast "Yêu cầu chuyển khoa đã được tạo thành công" |
| Error | Toast lỗi + giữ nguyên form để user sửa |

**Wireframe ASCII – MD Dashboard Tạo Referral:**

```
┌─────────────────────────────────────────────────────────────────┐
│  [≡]        CVH PROVIDER                    [🔔 2] [👤 BS Minh]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BỆNH NHÂN: Nguyễn Văn A (35 tuổi, Nam)                         │
│  MÃ BỆNH NHÂN: CVH-2025-001234                                  │
│                                                                 │
│  ╔═════════════════════════════════════════════════════════════╗│
│  ║               + TẠO YÊU CẦU CHUYỂN KHOA                     ║│
│  ╚═════════════════════════════════════════════════════════════╝│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Chuyên khoa *                                               ││
│  │ ┌─────────────────────────────────────────────────────┐     ││
│  │ │ Chọn chuyên khoa...                              ▼  │     ││
│  │ └─────────────────────────────────────────────────────┘     ││
│  │ • Cardiology (Tim mạch)                                     ││
│  │ • Orthopedics (Cơ xương khớp)                               ││
│  │ • Dermatology (Da liễu)                                     ││
│  │ • Neurology (Thần kinh)                                     ││
│  │ • Gastroenterology (Tiêu hóa)                               ││
│  │ • Ophthalmology (Mắt)                                       ││
│  │ • ENT (Tai mũi họng)                                        ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Lý do chuyển khoa *                                         ││
│  │ ┌─────────────────────────────────────────────────────┐     ││
│  │ │ Nhập lý do chuyển khoa...                           │     ││
│  │ │                                                     │     ││
│  │ └─────────────────────────────────────────────────────┘     ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Mức độ ưu tiên *                                            ││
│  │ ○ Routine (Thông thường)                                    ││
│  │ ○ Urgent (Khẩn cấp)                                         ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  [        HỦY        ]        [   TẠO YÊU CẦU CHUYỂN KHOA   ]   │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  [🏠 Home]  [📋 Patients]  [📅 Schedule]  [📊 Reports]  [⚙️]    │
└─────────────────────────────────────────────────────────────────┘
```

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Click "Tạo yêu cầu" | Validate → Submit → Chuyển sang Bước 2 |
| Click "Hủy" | Đóng form, quay lại màn hình bệnh nhân |
| Form validation fail | Hiển thị lỗi, focus vào trường đầu tiên sai |

---

### Bước 2: Hệ thống – Xác định gói dịch vụ của bệnh nhân

Hệ thống tìm trong danh sách để xác định gói hiện tại (Connect/Plus/Premium) và trạng thái đăng ký (Đang hoạt động/Dùng thử/Quá hạn).

- INPUT: patient_id (UUID, required), referral_id (UUID, required)
- OUTPUT: package_type (String: "Care Premium"/"Care Connect"/"Care Plus"), package_status (String: "Active"/"Trial"/"Expired"), routing_decision (String: "VIP"/"Standard")

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Gói = "Care Premium" VÀ trạng thái = "Active" | Tạo công việc trong hàng đợi MA → FLOW-02-VIP |
| Gói thuộc {"Care Connect", "Care Plus"} | Chuyển sang FLOW-02-STD (tự động) |
| Gói hết hạn | Thông báo: "Gói dịch vụ đã hết hạn. Vui lòng gia hạn để tiếp tục." |

**⚠️ Lỗi:**
- Không tìm thấy thông tin gói → Auto retry 3 lần → Escalate CS nếu vẫn lỗi

---

### Bước 3: Hệ thống – Tìm kiếm chuyên gia

Lọc theo chuyên khoa, vị trí, bảo hiểm, đánh giá.

- INPUT: specialty_type (String, required), patient_insurance (Object, required), patient_address (String, required)
- OUTPUT: specialist_list (Array of Object), search_count (Number)

**Tiêu chí tìm kiếm bắt buộc:**
- MUST: Matching specialty (đúng chuyên khoa)
- MUST: In-network insurance (bảo hiểm khớp)
- MUST: Within 10 miles (khoảng cách ≤ 10 dặm)
- MUST: Accepting new patients (đang nhận bệnh nhân mới)

**⚠️ Lỗi:**
- Không tìm thấy bác sĩ phù hợp → Auto mở rộng bán kính tìm kiếm + cảnh báo
- API bác sĩ lỗi → Auto retry 3 lần

---

### Bước 4: Hệ thống – Xếp hạng & lọc chuyên gia theo Priority Score

Sắp xếp kết quả theo thứ tự ưu tiên:
1. Nói tiếng Việt (ưu tiên cao nhất)
2. Đánh giá ≥ 4.0 sao (ưu tiên cao)
3. Lịch trống sớm nhất (ưu tiên vừa)
4. Khoảng cách gần nhất

- INPUT: specialist_list (Array, required)
- OUTPUT: ranked_list (Array, 5-10 specialists), ranking_criteria_applied (Object)

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Danh sách ≥ 5 bác sĩ | Cắt lấy 5-10 bác sĩ đầu |
| Danh sách < 5 bác sĩ | Auto giảm yêu cầu đánh giá xuống 3.5 + cảnh báo |
| Danh sách < 3 bác sĩ | Escalate CS |

---

### Bước 5: Hệ thống – Cập nhật hồ sơ bệnh nhân (referral history)

- INPUT: referral_id (UUID), patient_id (UUID), ranked_list (Array)
- OUTPUT: referral_record_updated (Boolean), system_log_id (String)

Ghi nhật ký: mã yêu cầu, mã khách hàng, quyết định phân loại, thời gian, luồng được kích hoạt.

---

### Bước 6: US MD – Review danh sách chuyên gia (nếu cần)

- ⚠️ MD không review trong thời gian SLA → hệ thống auto-remind (12h → nhắc lần 1, 24h → nhắc lần 2 + cảnh báo quản lý)

**Thời gian xử lý FLOW-01:** < 1 giây (tự động)

---

## FLOW-02-VIP: CHUẨN BỊ HỒ SƠ (VIP)

### Bước 7: Hệ thống – Chuyển case vào VIP queue cho MA

- INPUT: referral_id (UUID), patient_id (UUID)
- OUTPUT: ma_task_id (String), ma_notification_sent (Boolean)

---

### Bước 8: CS/MA – Nhận case, bắt đầu xử lý

MA nhìn thấy yêu cầu chuyển khoa mới trong hàng đợi (có thông báo). Bấm để mở chi tiết. Hệ thống đánh dấu trạng thái = "Đang xử lý".

- INPUT: filter_status (String: "all"/"new"/"in_progress"/"ready"), search_query (String)
- OUTPUT: referral_list (Array), unread_count (Number)

**Wireframe ASCII – MA Dashboard Referral Queue:**

```
┌─────────────────────────────────────────────────────────────────┐
│  [≡]        CVH PROVIDER                    [🔔 3] [👤 MA Lan]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REFERRAL QUEUE                              [Filter ▼] [🔍]    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ 🔴 NEW    Nguyễn Văn A                              2m ago  ││
│  │          Cardiology • Urgent                                ││
│  │          BS Trần Minh chỉ định                              ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ 🟡 IN PROGRESS    Trần Thị B                       15m ago  ││
│  │          Dermatology • Normal                               ││
│  │          Records: 80% ready                                 ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ 🟢 READY    Lê Văn C                                1h ago  ││
│  │          Orthopedics • Normal                               ││
│  │          Awaiting appointment                               ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  Showing 4 of 12 referrals           [Load More]                │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  [🏠 Home]  [📋 Queue]  [📅 Calendar]  [📊 Reports]  [⚙️]      │
└─────────────────────────────────────────────────────────────────┘
```

**UI States:**

| State | Hiển thị |
|-------|----------|
| Default | List sorted by urgency, then created_at DESC |
| Loading | Skeleton loading |
| Empty | "No referrals in queue" + illustration |
| Filtered | Show filter chip, count of results |

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Click referral row | Navigate to Referral Detail |
| Click Filter | Show filter dropdown |

**Wireframe ASCII – MA Chi tiết Referral:**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              REFERRAL DETAIL                    [⋮ More]   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ 🔴 URGENT                                Status: NEW        ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  THÔNG TIN BỆNH NHÂN                                            │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ 👤 Nguyễn Văn A                                    35 tuổi  ││
│  │ 📱 0912 345 678                                             ││
│  │ 📧 nguyenvana@email.com                                     ││
│  │ 🏥 Insurance: Bảo Việt Premium                              ││
│  │ 📦 Package: Care Premium (VIP)                              ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  CHỈ ĐỊNH TỪ BÁC SĨ                                             │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ 👨‍⚕️ BS. Trần Minh                       26/12/2025 14:30   ││
│  │                                                             ││
│  │ Chuyên khoa: Cardiology (Tim mạch)                          ││
│  │                                                             ││
│  │ Lý do chuyển:                                               ││
│  │ "Bệnh nhân có triệu chứng đau ngực, khó thở khi gắng sức.   ││
│  │ ECG cho thấy bất thường. Cần đánh giá chuyên khoa tim mạch."││
│  │                                                             ││
│  │ Chẩn đoán sơ bộ: Nghi ngờ bệnh mạch vành                    ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              TIẾP NHẬN & XỬ LÝ                              ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

- INPUT (Detail): referral_id (String/UUID)
- OUTPUT: patient_info (Object), referral_details (Object), referring_doctor (Object)

**UI States (Detail):**

| State | Hiển thị |
|-------|----------|
| NEW | Button "Tiếp nhận & Xử lý" enabled |
| IN_PROGRESS | Show progress checklist |
| READY | Show "Proceed to Transfer" button |

**⛔ Rẽ nhánh (Detail):**

| Điều kiện | Hành động |
|-----------|-----------|
| Click "Tiếp nhận & Xử lý" | Update status → Navigate to Chuẩn bị hồ sơ |
| Click patient phone | Open dialer |
| Click patient email | Open email composer |

---

### Bước 9: Hệ thống – Auto-compile Referral Letter (13 sections từ EMR)

Hệ thống tự động thu thập và điền thông tin vào Thư Giới Thiệu Chuyển Khoa:

| Section | Tên Section | Nội dung | PHI |
|---------|-------------|----------|-----|
| 1 | Header | Logo, tên cơ sở y tế, địa chỉ, thông tin liên hệ CVH | |
| 2 | Date Line | Ngày tạo thư giới thiệu | |
| 3 | Recipient Info | Thông tin bác sĩ/phòng khám chuyên khoa nhận | |
| 4 | Patient Demographics | Họ tên, DOB, giới tính, địa chỉ, SĐT, email, ngôn ngữ ưa thích | 🔒 |
| 5 | Reason for Referral | ⭐ CRITICAL - Lý do chuyển tuyến, câu hỏi lâm sàng, urgency | 🔒 |
| 6 | Clinical Summary | HPI, PMH, PSH, Family History, Social History | 🔒 |
| 7 | Current Medications | Danh sách thuốc đang dùng (name, dose, frequency) | 🔒 |
| 8 | Insurance Information | Công ty bảo hiểm, Số thẻ thành viên, Số nhóm | 🔒 |
| 9 | Requested Services | Mức độ khẩn cấp + Lý do urgency | |
| 10 | Closing Statement | Lời kết, phương thức liên lạc ưu tiên | |
| 11 | Signature Block | Chữ ký MD, license number, NPI, contact info | |
| 12 | Included Information | Danh sách thông tin kèm theo (trích xuất từ EMR) | 🔒 |
| 13 | Footer | HIPAA notice, mã yêu cầu, thời gian tạo | |

- INPUT: referral_id (String), patient_id (String)
- OUTPUT: referral_letter_draft (Object – 13 sections), missing_fields (Array)

**⚠️ Lỗi:**
- Thiếu dữ liệu bắt buộc → Auto highlight phần thiếu + thông báo MA cần bổ sung thủ công
- HSBA không truy cập được → Auto retry 3 lần (mỗi lần cách 30 giây)
- Dữ liệu không hợp lệ (ngày âm, định dạng sai) → Auto gắn cờ cảnh báo tại ô đó

---

### Bước 10: CS/MA – Kiểm tra hồ sơ: xác nhận đầy đủ, dịch thuật nếu cần

MA kiểm tra danh sách:
- ☐ Tất cả các phần của Thư Giới Thiệu đã đầy đủ?
- ☐ Thông tin bảo hiểm đã được ghi nhận?
- ☐ Kết quả xét nghiệm có liên quan?

Nếu thiếu → MA yêu cầu thêm từ bác sĩ hoặc khách hàng.

Nếu bác sĩ chuyên khoa không nói tiếng Việt → MA tải lên bản dịch tiếng Anh.

- INPUT: referral_id (String), uploaded_files (Array), translation_needed (Boolean), translated_files (Array)
- OUTPUT: referral_letter_package (Object – 13 phần đã tổng hợp), checklist_status (Object), ready_to_transfer (Boolean)

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| ECG file | Bắt buộc nếu chuyên khoa Tim mạch | "Vui lòng tải lên kết quả ECG" |
| Translated files | Bắt buộc nếu cần dịch | "Vui lòng tải lên bản dịch" |

**UI States:**

| State | Hiển thị |
|-------|----------|
| Incomplete | Nút bị vô hiệu, hiển thị các mục còn thiếu |
| Complete | Nút được kích hoạt, màu xanh |
| Uploading | Hiển thị tiến trình tải lên |

**Wireframe ASCII – Chuẩn bị hồ sơ:**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              CHUẨN BỊ HỒ SƠ                      [💾 Lưu]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Nguyễn Văn A • Tim mạch                        Tiến độ: 80%   │
│  [████████████████░░░░]                                         │
│                                                                 │
│  📋 THƯ GIỚI THIỆU CHUYỂN KHOA (13 Phần)                        │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ✅ Phần 1-4: Tiêu đề & Thông tin KH            [Xem]        ││
│  │    Tự động tổng hợp từ hồ sơ bệnh án                        ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ ✅ Phần 5: Lý do chuyển khoa                   [Xem]        ││
│  │    "Nghi ngờ bệnh mạch vành" - Khẩn cấp                     ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ ✅ Phần 6-9: Tóm tắt lâm sàng                  [Xem]        ││
│  │    Tiền sử, Thuốc đang dùng, Bảo hiểm                       ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ ✅ Phần 10: Kết quả xét nghiệm                 [Xem]        ││
│  │    3 kết quả XN • 90 ngày gần nhất                          ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ ⚠️ Phần 10b: Báo cáo ECG                      [Tải lên]     ││
│  │    ⚠️ Cần tải lên thủ công                                  ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ ✅ Phần 11-13: Chữ ký, Đính kèm, Footer        [Xem]        ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  CẦN DỊCH SANG TIẾNG ANH?                                       │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ○ Không cần dịch (bác sĩ nói tiếng Việt)                    ││
│  │ ● Cần dịch                                                  ││
│  │   ├── ☑ Tóm tắt y khoa                                      ││
│  │   ├── ☑ Kết quả xét nghiệm                                  ││
│  │   └── ☐ Ghi chú chuyển khoa                                 ││
│  │   [📎 Tải lên bản dịch]                                     ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │           HỒ SƠ SẴN SÀNG - CHUYỂN SANG TRUYỀN              ││
│  └─────────────────────────────────────────────────────────────┘│
│  (Chưa kích hoạt cho đến khi tất cả phần bắt buộc hoàn thành)   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Bấm "HỒ SƠ SẴN SÀNG" | Chuyển đến FLOW-03-VIP (Truyền hồ sơ) |
| Bấm [Xem] từng phần | Mở popup xem chi tiết phần đó |
| Bấm [Tải lên] | Mở dialog chọn file |

⚠️ Hồ sơ thiếu thông tin → CS liên hệ KH/MD bổ sung

**Thời gian xử lý FLOW-02-VIP:** 20-40 phút

---

### Bước 11: CS/MA – Chọn chuyên gia phù hợp nhất từ danh sách đã xếp hạng

(Xem phần truyền hồ sơ – FLOW-03-VIP Bước 12)

---

## FLOW-02-STD: GỬI DANH SÁCH CHUYÊN GIA (STANDARD)

### Bước 18: Hệ thống – Auto-routing case vào Standard queue

- INPUT: referral_id (UUID), routing_decision (String: "Standard")
- OUTPUT: workflow_activated (Boolean), next_step_triggered (Boolean)

---

### Bước 19: Hệ thống – Gửi danh sách chuyên gia cho KH qua Email + SMS + App

Mỗi bác sĩ bao gồm: Tên, bằng cấp, ảnh, Địa chỉ, điện thoại, khoảng cách, Ngôn ngữ, đánh giá, Đường link đặt lịch online (nếu có).

Gửi đồng thời:
- 📧 Email: Danh sách đầy đủ với chi tiết
- 📱 SMS: Tóm tắt + link xem đầy đủ
- 📲 App: Danh sách tương tác với nút đặt lịch

- INPUT: ranked_list (Array), patient_contact (Object)
- OUTPUT: email_sent (Boolean), sms_sent (Boolean), app_notification_sent (Boolean), send_log_id (String)

**Wireframe ASCII – Customer App Danh sách Specialist (Standard):**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]         BÁC SĨ CHUYÊN KHOA ĐƯỢC ĐỀ XUẤT                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Chuyên khoa: Tim mạch (Cardiology)                             │
│  Chỉ định bởi: BS. Trần Minh                                    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ 👨‍⚕️ Dr. Nguyễn Văn Tâm                           ⭐ 4.8    ││
│  │    Cardiologist • 15 năm kinh nghiệm                        ││
│  │    📍 Heart Care Center, Q1                    2.3 km       ││
│  │    🗣️ Tiếng Việt, English                                   ││
│  │    🏥 Bảo hiểm: Bảo Việt ✓                                  ││
│  │    [       ĐẶT LỊCH NGAY       ]                            ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ 👩‍⚕️ Dr. Trần Thị Hoa                             ⭐ 4.6    ││
│  │    Cardiologist • 10 năm kinh nghiệm                        ││
│  │    📍 Cardiology Associates, Q3                 4.1 km      ││
│  │    🗣️ Tiếng Việt                                            ││
│  │    🏥 Bảo hiểm: Bảo Việt ✓                                  ││
│  │    [       ĐẶT LỊCH NGAY       ]                            ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  💡 Sau khi đặt lịch, vui lòng báo lại cho CVH                  │
│     qua app hoặc gọi 1900-xxxx                                  │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  [🏠 Home]  [📅 Lịch]  [💬 Chat]  [📋 Hồ sơ]  [⚙️]             │
└─────────────────────────────────────────────────────────────────┘
```

**⚠️ Lỗi:**
- Gửi thất bại → Auto retry 3 lần → Cảnh báo CS

---

### Bước 20: Khách hàng – Tự đặt lịch với chuyên gia

- ⚠️ KH không đặt lịch trong 7 ngày → hệ thống gửi nhắc nhở
- ⚠️ Không đặt sau 14 ngày → escalate CS

**Business Rules (Standard):**

| Rule | Mô tả |
|------|-------|
| Auto-remind Day 3 | Tự động gửi email nhắc nếu KH chưa đặt lịch sau 3 ngày |
| Auto-remind Day 7 | Tự động gửi email nhắc lần 2 sau 7 ngày |
| Escalate Day 10 | Chuyển sang trạng thái "Quá hạn", MA cần gọi điện |

**Thời gian xử lý FLOW-02-STD:** 2-5 phút (tự động)

---

## FLOW-03-VIP: TRUYỀN HỒ SƠ CHO CHUYÊN GIA (VIP)

### Bước 12: Hệ thống – Gửi hồ sơ qua 3 kênh (fallback)

- Kênh 1: Secure Fax (SRFax API – HIPAA compliant, 256-bit AES encryption)
- Kênh 2: Secure Email (mã hóa)
- Kênh 3: EHR Portal upload

MA chọn specialist clinic, chọn tài liệu gửi, cấu hình trang bìa, bấm GỬI FAX.

- INPUT: specialist_id (String, required), specialist_fax (String, required), documents (Array, required), cover_page (Object: to, from, re, urgent)
- OUTPUT: fax_id (String), status (Enum: "success"/"failed"/"pending"), pages_sent (Number), transmission_time (Number – giây), sent_timestamp (DateTime)

**Validation / Business Rules:**

| Rule | Mô tả |
|------|-------|
| Referral PDF bắt buộc | File Referral Letter PDF luôn được gửi, không thể bỏ chọn |
| Cover page mặc định | Trang bìa luôn được bật mặc định |
| Supported formats | PDF, TIFF, DOC, DOCX, JPG, PNG (tự động convert sang TIFF) |
| Retry on failure | Nếu fax fail (số bận) → tự động retry sau 5 phút, tối đa 3 lần |
| Audit logging | Mọi hành động gửi đều được ghi log với fax_id, timestamp, pages, status |

**Wireframe ASCII – Màn hình Gửi Fax:**

```
┌─────────────────────────────────────────────────────────────────┐
│  [≡]        CVH MA PORTAL                    [🔔 2] [👤 MA Lan]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📠 GỬI HỒ SƠ CHUYỂN KHOA (FAX)                                 │
│                                                                 │
│  BỆNH NHÂN: Nguyễn Văn An (CVH-20260115)                        │
│  CHUYÊN KHOA: Tim mạch | MỨC ĐỘ: 🟠 Khẩn cấp                    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ NƠI NHẬN *                                                  ││
│  │ ┌─────────────────────────────────────────────────────┐     ││
│  │ │ 🔍 Tìm bác sĩ/phòng khám chuyên khoa...            │     ││
│  │ └─────────────────────────────────────────────────────┘     ││
│  │ 👨‍⚕️ Dr. Nguyễn Văn Tâm, MD, FACC                    [✓]    ││
│  │    Heart Care Center, Q1                                    ││
│  │    📠 Fax: (028) 3855-4270                                  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ TÀI LIỆU GỬI FAX                                            ││
│  │ ☑️ Referral_Letter_CVH20260115.pdf (3 trang) [👁️ Xem]       ││
│  │    (Bắt buộc - tự động tạo từ template)                     ││
│  │ THÊM TÀI LIỆU (không bắt buộc):                             ││
│  │ [📎 Chọn từ EMR]  [📤 Tải lên file mới]                     ││
│  │ ☐ Lab_Results_Jan2026.pdf (2 trang)                         ││
│  │ ☐ ECG_Report_Jan2026.pdf (1 trang)                          ││
│  │ 📄 Tổng: 3 trang | ⏱️ ~2 phút gửi                           ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ TRANG BÌA (COVER PAGE)                          [✓] Bật     ││
│  │ To:     Dr. Nguyễn Văn Tâm                                  ││
│  │ From:   CVH Primary Care - Dr. Trần Văn Bình                ││
│  │ Re:     Patient Referral - Nguyễn Văn An                    ││
│  │ Pages:  4 (bao gồm trang bìa)                               ││
│  │ ☑️ Urgent                                                   ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  [XEM TRƯỚC FAX 👁️]                                             │
│                                                                 │
│  [❌ Hủy]              [📠 GỬI FAX]                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Wireframe ASCII – Trạng thái sau khi gửi Fax:**

```
┌─────────────────────────────────────────────────────────────────┐
│  ✅ FAX ĐÃ GỬI THÀNH CÔNG                                       │
│                                                                 │
│  Mã fax:          FAX-20260115-001                               │
│  Gửi đến:         (028) 3855-4270                                │
│  Thời gian gửi:   15/01/2026 14:32:45                            │
│  Số trang:        4 trang                                        │
│  Thời gian truyền: 48 giây                                       │
│  Trạng thái:      🟢 Delivered                                   │
│                                                                 │
│  [📞 Gọi xác nhận]  [📅 Đặt lịch hẹn]  [➡️ Tiếp tục]           │
└─────────────────────────────────────────────────────────────────┘
```

**UI States:**

| State | Hiển thị |
|-------|----------|
| Default | Chọn clinic và method |
| Sending | Progress bar, "Đang gửi..." |
| Sent | Success message, show confirmation form |
| Error | Error message, retry button |

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Kênh 1 thất bại | Tự động chuyển Kênh 2 |
| Kênh 2 thất bại | Tự động chuyển Kênh 3 |
| Cả 3 kênh thất bại | ⚠️ CS gửi thủ công |

**SRFax API Flow:**

```
CVH Server                          SRFax API
    │                                   │
    │  POST /SRF_SecWebSvc.php          │
    │  {                                │
    │    action: "Queue_Fax",           │
    │    sToFaxNumber: "...",           │
    │    sFileName_1: "referral.pdf",   │
    │    sFileContent_1: "base64..."    │
    │  }                                │
    │ ─────────────────────────────────>│
    │                                   │
    │  Response: { Status: "Success",   │
    │              FaxDetailsID: "123"} │
    │ <─────────────────────────────────│
    │                                   │
    │  Webhook: POST /api/fax-webhook   │
    │  { FaxDetailsID: "123",           │
    │    Status: "Success",             │
    │    Pages: 4 }                     │
    │ <─────────────────────────────────│
```

**Thời gian xử lý:** 2-5 phút (phần lớn tự động)

---

### Bước 13: CS/MA – Xác nhận chuyên gia đã nhận hồ sơ

CS gọi điện phòng khám chuyên gia xác nhận 3 điểm:
1. Phòng khám đã nhận hồ sơ
2. Hồ sơ đầy đủ/đọc được
3. Tên người đang xử lý

- INPUT: confirmation_number (String, required), confirmed_by (String, required), notes (String, optional)
- OUTPUT: transfer_confirmed (Boolean)

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| confirmation_number | Required | "Vui lòng nhập số xác nhận" |
| confirmed_by | Required | "Vui lòng nhập tên người xác nhận" |
| checkbox | Checked | "Vui lòng xác nhận đã liên hệ clinic" |

**Wireframe ASCII – Xác nhận Receipt:**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              XÁC NHẬN NHẬN HỒ SƠ                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ HỒ SƠ ĐÃ GỬI THÀNH CÔNG                                     │
│                                                                 │
│  Gửi đến: Heart Care Center                                     │
│  Phương thức: EMR Integration                                    │
│  Thời gian: 26/12/2025 15:42:33                                  │
│  Transmission ID: TRX-20251226-001234                            │
│                                                                 │
│  XÁC NHẬN TỪ SPECIALIST                                         │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ☐ Đã liên hệ và xác nhận specialist đã nhận hồ sơ           ││
│  │ Confirmation Number: [_______________]                       ││
│  │ Người xác nhận từ clinic: [_______________]                  ││
│  │ Ghi chú (optional): [_______________]                        ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  [XÁC NHẬN & TIẾP TỤC ĐẶT LỊCH]                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

⚠️ Hệ thống auto-remind CS nếu chưa xác nhận trong SLA (3h → nhắc lần 1, 4h → escalate supervisor)

---

## FLOW-04-VIP: ĐẶT LỊCH HẸN (VIP)

### Bước 14: CS/MA – Liên hệ phòng khám chuyên gia, đặt lịch hẹn cho KH

MA gọi điện hoặc đặt lịch online, kiểm tra lịch trống, phối hợp với KH về thời gian.

- INPUT: selected_doctor (Object, required), selected_datetime (DateTime, required)
- OUTPUT: appointment_id (String), appointment_confirmed (Boolean)

**UI States:**

| State | Hiển thị |
|-------|----------|
| Selecting | Calendar view, slot selection |
| Confirming | Loading indicator |
| Confirmed | Success message, next steps |
| Conflict | Error: "Slot không còn khả dụng" |

**Wireframe ASCII – Đặt lịch hẹn:**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              ĐẶT LỊCH HẸN                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Nguyễn Văn A • Heart Care Center                               │
│                                                                 │
│  CHỌN BÁC SĨ CHUYÊN KHOA                                        │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ● Dr. Nguyễn Văn Tâm                        ⭐ 4.8          ││
│  │   Chuyên khoa Tim mạch • 15 năm kinh nghiệm                 ││
│  │   Ngôn ngữ: Tiếng Việt, English                             ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ ○ Dr. Trần Thị Hoa                          ⭐ 4.6          ││
│  │   Chuyên khoa Tim mạch • 10 năm kinh nghiệm                 ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  CHỌN THỜI GIAN                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │       Th2        Th3        Th4        Th5        Th6       ││
│  │      27/12      28/12      29/12      30/12      31/12      ││
│  │   ┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐    ││
│  │   │ 09:00 │  │ ░░░░░ │  │ 09:00 │  │ 09:00 │  │ ░░░░░ │    ││
│  │   │ 10:00 │  │ 10:00 │  │ ░░░░░ │  │ 10:00 │  │ 10:00 │    ││
│  │   │ 14:00 │  │ 14:00 │  │ 14:00 │  │ 14:00 │  │ ░░░░░ │    ││
│  │   │ 15:00 │  │ 15:00 │  │ 15:00 │  │ ░░░░░ │  │ 15:00 │    ││
│  │   └───────┘  └───────┘  └───────┘  └───────┘  └───────┘    ││
│  │   ░░░░░ = Không khả dụng                                    ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ĐÃ CHỌN: Thứ 2, 27/12/2025 lúc 09:00                           │
│                                                                 │
│  [XÁC NHẬN ĐẶT LỊCH]                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Bước 15: CS/MA – Gửi thông tin lịch hẹn cho KH

Gửi qua SMS + Email + In-app notification: ngày giờ, địa chỉ, hướng dẫn đường đi/đậu xe, hướng dẫn chuẩn bị.

**Wireframe ASCII – Gửi thông tin cho KH:**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]         GỬI THÔNG TIN LỊCH HẸN CHO KH                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ ĐÃ ĐẶT LỊCH THÀNH CÔNG                                      │
│                                                                 │
│  PREVIEW THÔNG TIN GỬI CHO KHÁCH HÀNG                           │
│  ╔═══════════════════════════════════════════════════════╗       │
│  ║         THÔNG TIN LỊCH HẸN CHUYÊN KHOA               ║       │
│  ║                                                       ║       │
│  ║ 📅 Thứ Hai, 27/12/2025 lúc 09:00                      ║       │
│  ║ 👨‍⚕️ Bác sĩ: Dr. Nguyễn Văn Tâm                        ║       │
│  ║    Chuyên khoa: Tim mạch (Cardiology)                 ║       │
│  ║ 📍 Heart Care Center, 123 Nguyen Hue, Q1             ║       │
│  ║    [Xem bản đồ]                                       ║       │
│  ║ 🅿️ Đỗ xe: Tầng hầm B1, miễn phí 2 giờ đầu             ║       │
│  ║ 📋 Mang theo: CMND/CCCD, Thẻ bảo hiểm                ║       │
│  ║ ⚠️ Chuẩn bị: Nhịn ăn 8 tiếng, mang thuốc đang dùng   ║       │
│  ║ ☎️ Thắc mắc? Gọi: 1900-xxxx                           ║       │
│  ╚═══════════════════════════════════════════════════════╝       │
│                                                                 │
│  KÊNH GỬI                                                       │
│  ☑️ Email: nguyenvana@email.com                                  │
│  ☑️ SMS: 0912 345 678                                            │
│  ☑️ App Notification                                             │
│                                                                 │
│  [GỬI THÔNG TIN]                                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Bước 16: Hệ thống – Gửi nhắc nhở 24h trước lịch hẹn

MA gọi điện cho KH 24 giờ trước, xác nhận có thể đến, nhắc chuẩn bị, trả lời câu hỏi.

- ⚠️ KH không xác nhận → CS gọi điện nhắc
- ⚠️ KH không bắt máy → Auto gửi SMS + Email backup

**Thời gian xử lý FLOW-04-VIP:** 10-20 phút

---

## FLOW-05: FOLLOW-UP & NHẬN KẾT QUẢ

### VIP Path

#### Bước 17-VIP-A1: MA gọi KH sau 24h

MA gọi điện cho KH, hỏi thăm về buổi khám, thu thập phản hồi ban đầu.

Script gợi ý:
> "Chào anh A, em là Lan từ CVH. Em gọi để hỏi thăm về cuộc khám với Dr. Tâm hôm qua. Anh thấy cuộc khám thế nào ạ? Bác sĩ có đề xuất gì không ạ?"

#### Bước 17-VIP-A2: MA liên hệ bác sĩ chuyên khoa

MA gọi/email phòng khám chuyên khoa: yêu cầu báo cáo khám (Consult Note), xác nhận chẩn đoán/khuyến nghị, lấy đơn thuốc.

- ⚠️ Phòng khám chưa gửi kết quả trong SLA → CS gọi lại, escalate nếu cần

#### Bước 17-VIP-A3: MA ghi nhận kết quả

- INPUT: customer_called (Boolean), consult_note_received (Boolean), consult_note_file (File), diagnosis (String), treatment_plan (String), next_step (String: "continue_specialist"/"back_to_pcp"/"additional_intervention")
- OUTPUT: followup_completed (Boolean)

**Wireframe ASCII – Follow-up sau khám:**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              FOLLOW-UP SAU KHÁM                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Nguyễn Văn A • Cardiology                                      │
│  Appointment: 27/12/2025 09:00 ✅ Đã hoàn thành                  │
│                                                                 │
│  GỌI KHÁCH HÀNG                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ 📞 0912 345 678                              [📞 Gọi ngay]  ││
│  │ ☐ Đã gọi và nói chuyện với KH                               ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  LIÊN HỆ SPECIALIST                                             │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ 📞 Heart Care Center: (028) 1234-5678       [📞 Gọi ngay]  ││
│  │ ☐ Đã liên hệ và nhận consult note                           ││
│  │ Upload Consult Note: [📎 Kéo thả file hoặc click]           ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  GHI NHẬN KẾT QUẢ                                               │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Chẩn đoán: [________________________]                       ││
│  │ Đề xuất điều trị: [________________________]                ││
│  │ Kế hoạch tiếp theo:                                         ││
│  │ ○ Tiếp tục theo dõi với specialist                          ││
│  │ ● Quay lại với BS chính                                     ││
│  │ ○ Cần can thiệp bổ sung                                     ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  [LƯU & GỬI CHO BÁC SĨ REVIEW]                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Standard Path

#### Bước 21-STD: Hệ thống – Gửi email follow-up sau ngày hẹn khám (24-48h)

Hệ thống gửi email theo dõi với: 3 câu hỏi + link báo cáo kết quả + hotline.

- Không phản hồi sau 3 ngày → Auto gửi email nhắc lần 2
- Không phản hồi sau 7 ngày → Auto gửi email nhắc lần 3
- Không phản hồi sau 10 ngày → Auto chuyển case cho CS

#### Bước 22-STD: Khách hàng – Tự upload kết quả khám

KH bấm vào đường link và nhập:
- Buổi khám có diễn ra không? Có/Không
- Kết quả từ bác sĩ chuyên khoa (nhập tự do)
- Tải lên tài liệu (nếu có): PDF/JPG/PNG, tối đa 10MB mỗi file

- INPUT: visit_status (String: "completed"/"reschedule"/"cancelled", required), uploaded_files (Array, optional – max 10MB/file, tổng 30MB), diagnosis (String, optional), notes (String, optional)
- OUTPUT: report_submitted (Boolean)

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| visit_status | Bắt buộc | "Vui lòng chọn tình trạng cuộc khám" |
| uploaded_files | Tối đa 10MB mỗi file, tổng 30MB | "File quá lớn" |

**UI States:**

| State | Hiển thị |
|-------|----------|
| Default | Form trống, có thể bắt đầu nhập |
| Uploading | Progress bar cho file upload |
| Submitted | Toast "Cảm ơn bạn đã gửi báo cáo!" |
| Error | Toast lỗi, giữ nguyên form |

**Wireframe ASCII – Upload kết quả khám (Standard):**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]         BÁO CÁO KẾT QUẢ KHÁM CHUYÊN KHOA                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Chuyên khoa: Tim mạch | Bác sĩ: Dr. Nguyễn Văn Tâm            │
│  Ngày khám: 27/12/2025                                          │
│                                                                 │
│  TÌNH TRẠNG CUỘC KHÁM *                                         │
│  ● Đã khám thành công                                           │
│  ○ Chưa khám được (xin đặt lại lịch)                            │
│  ○ Hủy không khám nữa                                           │
│                                                                 │
│  TẢI LÊN GIẤY TỜ TỪ BÁC SĨ                                     │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │      📄 Kéo thả file hoặc nhấn để chọn                      ││
│  │         PDF, JPG, PNG (tối đa 10MB)                          ││
│  └─────────────────────────────────────────────────────────────┘│
│  📎 consult_note.pdf                            [✕ Xóa]         │
│  📎 prescription.jpg                            [✕ Xóa]         │
│                                                                 │
│  CHẨN ĐOÁN (nếu có): [________________________]                 │
│  GHI CHÚ THÊM: [________________________]                       │
│                                                                 │
│  [GỬI BÁO CÁO]                                                  │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  [🏠 Home]  [📅 Lịch]  [💬 Chat]  [📋 Hồ sơ]  [⚙️]             │
└─────────────────────────────────────────────────────────────────┘
```

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|-----------|
| Chọn "Đã khám thành công" | Hiển thị form upload và ghi chú |
| Chọn "Chưa khám được" | Hiển thị form đặt lại lịch |
| Chọn "Hủy không khám" | Yêu cầu lý do hủy, gửi thông báo |
| Click "Gửi báo cáo" | Validate → Submit → Cập nhật EMR |

- ⚠️ KH không upload trong 7 ngày → nhắc nhở. 14 ngày → escalate CS

**Thời gian xử lý:** VIP: 10-15 phút | Standard: 5 phút (tự động)

---

## FLOW-06: CẬP NHẬT EMR & MD REVIEW

### Hệ thống – Cập nhật kết quả chuyên gia vào EMR

Hệ thống tự động:
- 🔒 Đính kèm báo cáo khám chuyên khoa
- 🔒 Liên kết với yêu cầu chuyển khoa ban đầu
- 🔒 Cập nhật dòng thời gian bệnh nhân
- Tạo bản ghi lịch sử

---

### Hệ thống – Thông báo cho BS

Gửi thông báo: 📲 Thông báo trên ứng dụng, 📧 Email tóm tắt, 📋 Thêm vào danh sách chờ BS xem xét.

---

### MD (US) – Review kết quả chuyên gia

- INPUT: referral_id (String), md_decision (String: "agree"/"adjust"/"discuss"), care_plan_notes (String)
- OUTPUT: reviewed (Boolean), care_plan_updated (Boolean)

MD đánh giá kết quả, điều chỉnh kế hoạch điều trị nếu cần, ghi nhận vào EMR.

- ⚠️ MD không review trong SLA → hệ thống auto-remind (12h → nhắc lần 1, 24h → nhắc lần 2 + cảnh báo quản lý)
- ⚠️ Phát hiện từ khóa khẩn cấp (urgent, cancer, emergency, severe) → Auto cảnh báo MD ngay lập tức

**Wireframe ASCII – MD Review kết quả:**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              SPECIALIST RESULTS                 [🔔 Badge] │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CHI TIẾT KẾT QUẢ - NGUYỄN VĂN A                                │
│                                                                 │
│  📋 CONSULT NOTE                                      [📎 View]  │
│  Specialist: Dr. Nguyễn Văn Tâm | Date: 27/12/2025              │
│  ─────────────────────────────────────────────────────────       │
│  CHẨN ĐOÁN: Bệnh mạch vành ổn định                              │
│  KẾT QUẢ XÉT NGHIỆM BỔ SUNG:                                   │
│  • Stress test: Âm tính                                         │
│  • Echo: EF 55%, không rối loạn vận động vùng                   │
│  ĐỀ XUẤT ĐIỀU TRỊ:                                              │
│  • Aspirin 81mg daily                                            │
│  • Atorvastatin 20mg daily                                       │
│  • Lifestyle modification                                        │
│  TÁI KHÁM: Sau 3 tháng                                          │
│  ─────────────────────────────────────────────────────────       │
│                                                                 │
│  QUYẾT ĐỊNH CỦA TÔI                                             │
│  ● Đồng ý với đề xuất của specialist                             │
│  ○ Cần điều chỉnh kế hoạch điều trị                              │
│  ○ Cần thảo luận thêm với specialist                             │
│                                                                 │
│  Ghi chú cho care plan: [________________________]               │
│                                                                 │
│  [ĐÁNH DẤU ĐÃ REVIEW & CẬP NHẬT CARE PLAN]                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Hệ thống – Hoàn tất & Lưu trữ

- Hoàn tất nhật ký hệ thống
- Lưu trữ tất cả tài liệu
- Đánh dấu trạng thái = "Hoàn thành"
- Kích hoạt thanh toán/quyết toán (nếu có)

**Thời gian xử lý:** < 1 phút (hệ thống) + 5-10 phút (BS xem xét)

---

## XỬ LÝ NGOẠI LỆ

### No-show (KH không đến khám)

| Gói | Xử lý |
|-----|--------|
| **VIP** | MA gọi ngay để hỏi thăm. Đặt lại lịch ngay nếu có lý do chính đáng |
| **STD** | Gửi email nhắc nhở + hướng dẫn đặt lại lịch. Chuyển cho MA sau 3 lần nhắc không phản hồi |

### Cần xét nghiệm bổ sung

| Gói | Xử lý |
|-----|--------|
| **VIP** | MA phối hợp xét nghiệm với bác sĩ/phòng xét nghiệm. Đặt lại lịch sau khi có kết quả |
| **STD** | Hệ thống thông báo KH về yêu cầu xét nghiệm. KH tự phối hợp |

### Kết quả cần can thiệp khẩn cấp

| Gói | Xử lý |
|-----|--------|
| **Cả hai** | Phát hiện từ khóa khẩn cấp trong kết quả. Báo ngay cho BS. BS liên hệ KH trong 2 giờ |

---

## BẢNG THÔNG BÁO TỰ ĐỘNG (MỞ RỘNG CÓ TEMPLATE)

| Sự kiện | Kênh | Người nhận |
|---------|------|------------|
| Danh sách chuyên gia (Standard) | Email + SMS + In-app | Khách hàng |
| Xác nhận lịch hẹn (VIP) | SMS + Email | Khách hàng |
| Nhắc nhở 24h trước lịch hẹn | SMS + Email + Push | Khách hàng |
| Follow-up sau buổi khám | Email | Khách hàng |
| MD cần review kết quả | Alert nội bộ | US MD |
| Referral mới cần xử lý | Alert nội bộ | CS/MA |
| KH không đặt lịch (Standard, 7d) | Email + Push | Khách hàng |
| KH không upload kết quả (14d) | Email + Push | Khách hàng |

### Template chi tiết

📱 **SMS – Danh sách Specialist (Standard):**
```
[CVH] Bạn có chỉ định khám Tim mạch từ BS. Trần Minh.
Xem danh sách bác sĩ chuyên khoa và đặt lịch tại: https://cvh.app/referral/abc123
Thắc mắc? Gọi 1900-xxxx
```

📧 **Email – Danh sách Specialist (Standard):**
```
Subject: [CVH] Danh sách Bác sĩ Tim mạch cho bạn

Xin chào {patient_name},

Bác sĩ {referring_doctor} đã chỉ định bạn khám chuyên khoa Tim mạch.

Dưới đây là danh sách bác sĩ phù hợp với bạn:

1. Dr. Nguyễn Văn Tâm ⭐ 4.8
   Heart Care Center, Q1 (2.3 km)
2. Dr. Trần Thị Hoa ⭐ 4.6
   Cardiology Associates, Q3 (4.1 km)
3. Dr. Phạm Văn Đức ⭐ 4.5
   City Heart Clinic, Q7 (5.8 km)

[ĐẶT LỊCH NGAY]

Sau khi đặt lịch, vui lòng báo lại cho chúng tôi qua app hoặc gọi 1900-xxxx.

Trân trọng,
CVH Healthcare Team
```

📧 **Email – Truyền hồ sơ cho Specialist Office (VIP):**
```
Subject: Referral - {patient_name} - {specialty}

Dear Dr. {specialist_name},

We are referring {patient_name} (DOB: {date_of_birth}) for {specialty} consultation.

Reason: {reason_for_referral}

Attached: Medical Summary

Please contact us at {cvh_phone} if you need more information.

Best regards,
CVH Healthcare
```

📱 **SMS – Xác nhận lịch hẹn (VIP):**
```
[CVH] Lịch hẹn khám Tim mạch đã được đặt!
📅 Thứ Hai, 27/12/2025 lúc 09:00
👨‍⚕️ Dr. Nguyễn Văn Tâm
📍 Heart Care Center, 123 Nguyen Hue, Q1
Chi tiết: https://cvh.app/apt/xyz789
```

📧 **Email – Xác nhận lịch hẹn (VIP):**
```
Subject: [CVH] Xác nhận lịch hẹn khám Tim mạch - 27/12/2025

Xin chào {patient_name},

Lịch hẹn khám chuyên khoa của bạn đã được đặt thành công!

═══════════════════════════════════════════
        THÔNG TIN LỊCH HẸN
═══════════════════════════════════════════

📅 Thời gian: Thứ Hai, 27/12/2025 lúc 09:00
👨‍⚕️ Bác sĩ: Dr. Nguyễn Văn Tâm
🏥 Chuyên khoa: Tim mạch (Cardiology)
📍 Địa chỉ: Heart Care Center, 123 Nguyen Hue, Q1, TP.HCM [Xem bản đồ]
🅿️ Đỗ xe: Tầng hầm B1, miễn phí 2 giờ đầu

📋 Vui lòng mang theo:
   • CMND/CCCD
   • Thẻ bảo hi��m y tế
   • Danh sách thuốc đang dùng

⚠️ Chuẩn bị trước khám:
   • Nhịn ăn 8 tiếng trước khám
   • Đến trước 15 phút để làm thủ tục

═══════════════════════════════════════════

Có thắc mắc? Liên hệ MA phụ trách: {ma_name}
📞 {ma_phone} | 📧 {ma_email}

Trân trọng,
CVH Healthcare Team
```

📱 **SMS – Nhắc nhở 24h trước (VIP):**
```
[CVH] Nhắc nhở: Bạn có lịch khám Tim mạch NGÀY MAI
📅 09:00, 27/12/2025
👨‍⚕️ Dr. Nguyễn Văn Tâm
📍 Heart Care Center, Q1
Nhớ nhịn ăn 8 tiếng trước khám.
Thắc mắc? Gọi MA Lan: 0912-xxx-xxx
```

📧 **Email – Follow-up sau khám (Standard):**
```
Subject: [CVH] Cuộc khám của bạn thế nào?

Xin chào {patient_name},

Chúng tôi hy vọng cuộc khám với Dr. {specialist_name} đã diễn ra tốt đẹp!

Để chúng tôi cập nhật hồ sơ và phối hợp chăm sóc tốt hơn,
vui lòng cho biết kết quả khám:

[BÁO CÁO KẾT QUẢ KHÁM]

Bạn có thể:
• Upload giấy tờ từ bác sĩ chuyên khoa
• Ghi lại chẩn đoán và đề xuất điều trị
• Chia sẻ bất kỳ thắc mắc nào

Nếu bạn cần hỗ trợ, hãy gọi 1900-xxxx.

Trân trọng,
CVH Healthcare Team
```

📲 **Notification – Cho MD khi có kết quả:**
```
Title: Kết quả khám chuyên khoa mới
Body:
Bệnh nhân: {patient_name}
Chuyên khoa: Cardiology
BS: Dr. Nguyễn Văn Tâm
Chẩn đoán: Bệnh mạch vành ổn định
[XEM CHI TIẾT]
```

---

## BẢNG GIÁM SÁT CS-MD ĐẦY ĐỦ

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| **Tạo yêu cầu chuyển tuyến** | Bác sĩ tạo ≤ 2 phút, hệ thống ghi nhận ≤ 2 giây. 100% đủ 4 trường (chuyên khoa, lý do, triệu chứng, chẩn đoán) | Validate 4 trường real-time, auto gắn mã BN từ HSBA, auto retry 3 lần nếu DB error | DB sập >10 phút, lỗi fatal không phục hồi → CS hỗ trợ tạo yêu cầu dự phòng | ✅ Là người tạo yêu cầu, nhập đủ 4 trường |
| **Xác định gói dịch vụ** | Hệ thống xác định ≤ 2 giây. 100% đúng gói VIP/Standard | Auto truy vấn DB, kiểm tra trạng thái active, phân luồng tự động | DB sập >10 phút → CS xác nhận gói thủ công | ❌ |
| **Tìm bác sĩ chuyên khoa** | Kết quả ≤ 10 giây. 100% đáp ứng 4 tiêu chí (chuyên khoa, bảo hiểm, khoảng cách, nhận BN mới) | Truy vấn 4 tiêu chí đồng thời, auto mở rộng bán kính nếu không tìm thấy, retry 3 lần | DB bác sĩ sập >10 phút → CS điều chỉnh tiêu chí tìm kiếm | ❌ |
| **Xếp hạng và lọc** | ≤ 5 giây. 100% đúng thứ tự (tiếng Việt > đánh giá ≥4.0 > lịch trống > khoảng cách). Danh sách 5-10 BS | Auto xếp hạng 4 mức ưu tiên, lọc bỏ <4.0, fallback sang xếp đơn giản nếu lỗi | Hệ thống sập >10 phút, <3 BS → CS điều chỉnh thủ công | ❌ |
| **Cập nhật hồ sơ** | Cập nhật HSBA ≤ 5 giây. Gửi thông báo BS ≤ 3 giây. 100% đầy đủ (chẩn đoán, đơn thuốc, khuyến nghị, ngày khám) | Auto parse dữ liệu, validate 4 trường, gắn referral ID, gửi App+Email cho BS. Phát hiện từ khóa khẩn cấp → alert MD | DB sập >10 phút, không parse được → CS nhập thủ công | ✅ Khi có cảnh báo khẩn cấp |
| **Bác sĩ xem xét** | BS xem xét ≤ 24 giờ. Cập nhật kế hoạch ≤ 48 giờ. 100% xác nhận đã đọc | Auto nhắc 12h (lần 1), 24h (lần 2 + cảnh báo quản lý). Auto ghi log thời gian | Case >48h không xem → CS liên hệ BS | ✅ Xem xét, cập nhật kế hoạch điều trị |
| **Phân luồng VIP** | ≤ 3 giây. 100% Care Premium → CS. Không nhầm sang Standard | Auto phân luồng, thêm vào work queue CS, gửi App+Email cho CS | Hàng đợi sập → CS tiếp nhận thủ công | ❌ |
| **CS tiếp nhận** | Xác nhận ≤ 15 phút. Liên hệ KH ≤ 2 giờ làm việc | Auto nhắc 10 phút (lần 1), 15 phút (escalate supervisor) | ✅ Xác nhận tiếp nhận, xem thông tin, liên hệ KH | ❌ |
| **Tổng hợp hồ sơ tự động** | ≤ 5 phút. 100% đủ 13 phần. 100% dữ liệu đúng từ HSBA | Auto trích xuất HSBA, điền template 13 phần, highlight phần thiếu | HSBA sập >10 phút → CS tổng hợp thủ công | ❌ |
| **CS kiểm tra hồ sơ & dịch thuật** | CS hoàn thành ≤ 15 phút. 100% kiểm tra đủ 13 phần. 100% dịch chuẩn thuật ngữ | Checklist 13 phần tick off, spell check tự động, cảnh báo dữ liệu bất thường | ✅ Kiểm tra đầy đủ, đối chiếu HSBA, kiểm tra dịch thuật | 🔍 Hỗ trợ review dữ liệu y tế phức tạp |
| **Chọn bác sĩ chuyên khoa (VIP)** | CS hoàn thành ≤ 15 phút. 100% đáp ứng 4 tiêu chí. Ưu tiên tiếng Việt | Hiển thị danh sách đã xếp hạng, highlight tiếng Việt, cảnh báo nếu chọn <4.0 hoặc out-of-network | ✅ Xem xét danh sách, chọn BS, ghi nhận lý do | ❌ |
| **Gửi hồ sơ (3-channel fallback)** | ≤ 5 phút. Delivery confirmation ≤ 30 phút. 100% kênh bảo mật. Mã hóa AES-256 | Auto chọn kênh ưu tiên, fallback kênh 1→2→3, mã hóa HIPAA, ghi log chi tiết | Cả 3 kênh thất bại → CS gửi thủ công | ❌ |
| **Xác nhận đã nhận** | CS gọi xác nhận ≤ 4 giờ làm việc. 100% xác nhận 3 điểm | Auto tạo task xác nhận, nhắc 3h, escalate 4h. "Chưa nhận" → auto gửi lại | ✅ Gọi điện xác nhận 3 điểm | ❌ |
| **Đặt lịch & thông báo KH** | Đặt lịch ≤ 4 giờ. Xác nhận KH ≤ 1 giờ. Lịch ≤ 5 ngày. Gửi 3 kênh | Auto tính ngày, cảnh báo >5 ngày, template 5 trường, auto gửi 3 kênh | ✅ Liên hệ phòng khám, đặt lịch, xác nhận KH | ❌ |
| **Nhắc nhở 24h trước hẹn** | Đúng 24h trước (±1 giờ). 100% xác nhận 3 điểm | Auto tạo reminder, cảnh báo nếu chưa gọi sau 25h. "Không bắt máy" → auto SMS+Email | ✅ Gọi điện KH, xác nhận 3 điểm | ❌ |
| **Theo dõi sau khám (VIP)** | CS liên hệ ≤ 24 giờ. 100% hỏi 3 điểm. 100% yêu cầu kết quả | Auto nhắc CS, cảnh báo >24h, template 3 điểm | ✅ Gọi KH, hỏi thăm, yêu cầu kết quả | ❌ |
| **Thu thập kết quả** | ≤ 5 ngày làm việc. 100% đầy đủ (chẩn đoán, đơn thuốc, khuyến nghị, tái khám) | Auto nhắc CS sau 5 ngày, checklist 4 trường | ✅ Liên hệ phòng khám, kiểm tra đầy đủ, xác thực nguồn gốc | ❌ |
| **Phân luồng Standard** | ≤ 3 giây. 100% Care Connect/Plus → tự động. Không nhầm sang VIP | Auto phân luồng, kích hoạt workflow Standard, ghi log | Workflow sập >10 phút → CS hỗ trợ dự phòng | ❌ |
| **Gửi danh sách (Standard)** | ≤ 5 phút. 100% đủ thông tin 7 trường. Gửi 3 kênh. Email có hướng dẫn, SMS có hotline | Auto gửi 3 kênh đồng thời, format email+SMS, ghi log delivery | Hệ thống gửi sập >10 phút → CS thông báo qua kênh khác | ❌ |
| **KH tự đặt lịch** | Ghi nhận đúng 4 trường (ngày, giờ, tên BS, địa chỉ). Xác nhận ≤ 5 giây | Tooltip hướng dẫn, validate real-time, auto xác nhận App+Email, nhắc 3d/7d/10d | Quá 10 ngày không phản hồi → CS liên hệ | ❌ |
| **Email theo dõi sau khám (Standard)** | Gửi đúng 24-48h. Email có 3 câu hỏi + link + hotline | Auto schedule email, nhắc lần 2 (3d), lần 3 (7d), chuyển CS (10d) | Hệ thống sập >10 phút, hoặc 10 ngày không phản hồi → CS liên hệ | ❌ |
| **KH báo cáo kết quả** | 100% hiển thị đúng giao diện. Cho phép file ≤10MB. Xác nhận ≤ 5 giây | Hướng dẫn từng bước, validate file, auto lưu + gắn referral ID + thông báo BS | Lỗi upload/submit không khắc phục → CS hỗ trợ | ❌ (chỉ nhận thông báo) |

---

## DATA SPECIFICATION

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| referral_id | UUID | Auto | UUID v4 | Mã yêu cầu chuyển khoa |
| patient_id | UUID | Yes | UUID v4 | Mã bệnh nhân |
| specialty_type | String | Yes | Từ danh sách chuẩn | Chuyên khoa được chọn |
| referral_reason | String | Yes | Min 20, Max 500 ký tự | Lý do chuyển khoa |
| urgency_level | String | Yes | "Routine" \| "Urgent" | Mức độ ưu tiên |
| package_type | String | Auto | "Care Premium" \| "Care Connect" \| "Care Plus" | Gói dịch vụ KH |
| package_status | String | Auto | "Active" \| "Trial" \| "Expired" | Trạng thái gói |
| routing_decision | String | Auto | "VIP" \| "Standard" | Quyết định phân luồng |
| specialist_id | String | Yes (VIP) | UUID | ID bác sĩ chuyên khoa đã chọn |
| specialist_fax | String | Yes (VIP) | Phone format | Số fax specialist |
| specialist_name | String | Auto | Free text | Tên bác sĩ chuyên khoa |
| specialist_clinic | String | Auto | Free text | Tên phòng khám |
| specialist_address | String | Auto | Free text | Địa chỉ phòng khám |
| specialist_rating | Number | Auto | 0.0 – 5.0 | Đánh giá bác sĩ |
| specialist_language | Array | Auto | String[] | Ngôn ngữ bác sĩ nói |
| specialist_distance | Number | Auto | km/miles | Khoảng cách từ KH |
| documents | Array | Yes | File[] (PDF/TIFF/DOC/JPG/PNG) | Tài liệu gửi |
| cover_page | Object | Yes (Fax) | {to, from, re, pages, urgent} | Thông tin trang bìa fax |
| fax_id | String | Auto | From SRFax API | Mã fax |
| transmission_status | Enum | Auto | "success" \| "failed" \| "pending" | Trạng thái truyền |
| pages_sent | Number | Auto | Integer ≥ 1 | Số trang đã gửi |
| transmission_time | Number | Auto | Seconds | Thời gian truyền |
| confirmation_number | String | Yes (VIP) | Free text | Số xác nhận từ clinic |
| confirmed_by | String | Yes (VIP) | Free text | Tên người xác nhận |
| appointment_id | String | Auto | UUID | Mã lịch hẹn |
| appointment_datetime | DateTime | Yes | ISO 8601 | Ngày giờ hẹn khám |
| selected_doctor | Object | Yes | {id, name, specialty, clinic} | Bác sĩ đã chọn |
| visit_status | String | Yes (STD) | "completed" \| "reschedule" \| "cancelled" | Tình trạng cuộc khám |
| uploaded_files | Array | Optional | File[] (PDF/JPG/PNG), max 10MB/file, tổng 30MB | File KH tải lên |
| diagnosis | String | Optional | Free text | Chẩn đoán từ specialist |
| treatment_plan | String | Optional | Free text | Đề xuất điều trị |
| next_step | String | Yes (VIP) | "continue_specialist" \| "back_to_pcp" \| "additional_intervention" | Kế hoạch tiếp theo |
| md_decision | String | Yes | "agree" \| "adjust" \| "discuss" | Quyết định của MD |
| care_plan_notes | String | Optional | Free text | Ghi chú care plan |
| referral_letter_package | Object | Auto | 13 sections | Thư Giới Thiệu Chuyển Khoa |
| translation_needed | Boolean | Yes (VIP) | true/false | Có cần dịch không |
| hipaa_consent | Object | Yes | {accepted, timestamp, version} | HIPAA consent record |
| created_at | DateTime | Auto | ISO 8601 | Thời gian tạo |
| updated_at | DateTime | Auto | ISO 8601 | Thời gian cập nhật |

---

## So sánh VIP vs Standard

| Tiêu chí | VIP (Care Premium) | Standard (Care Connect/Plus) |
|----------|-----|----------|
| **Chuẩn bị hồ sơ** | MA tổng hợp Thư Giới Thiệu (13 phần) | ❌ Không có |
| **Truyền hồ sơ** | MA gửi qua kênh bảo mật (Fax/Email/EHR Portal) | ❌ Không có |
| **Đặt lịch** | MA đặt hộ, phối hợp với KH | KH tự đặt từ danh sách |
| **Nhắc nhở** | MA gọi điện 24h trước | SMS tự động |
| **Theo dõi sau khám** | MA gọi KH + liên hệ specialist | Email tự động, KH tự báo cáo |
| **Thời gian** | ≤5 ngày đến lịch hẹn đầu tiên | 7-14 ngày |
| **Mức độ hỗ trợ** | Cao (dịch vụ concierge) | Thấp (tự phục vụ) |

---

## SUCCESS METRICS

| Metric | VIP Target | Standard Target |
|--------|-----------|-----------------|
| Time to First Appointment | ≤ 5 days | ≤ 14 days |
| Referral Completion Rate | ≥ 95% | ≥ 85% |
| Customer Satisfaction | ≥ 4.7/5.0 | ≥ 4.3/5.0 |
| No-Show Rate | ≤ 5% | ≤ 10% |
| Results Documented in EMR | 100% | 100% |

---

## NGUỒN TÀI LIỆU GỐC

- Luồng nghiệp vụ: `docs/02-product/ba-specs/service-2-Specialist_Referral/base/Specialist_Referral_Touchpoint_Flow_v1.md`
- Checklist CS & MD: `docs/02-product/ba-specs/service-2-Specialist_Referral/checklist-cs-md/checklist-cs-md.md`
- Bản tóm tắt: `docs/02-product/tomtat-chitiet/tomtat/service-2-Specialist_Referral-tomtat.md`
