# CHI TIẾT LUỒNG NGHIỆP VỤ – SERVICE 1 (Access_to_Care_247): 24/7 ACCESS TO CARE

## Header

- **Service:** 24/7 Access to Care
- **Service ID:** Service-1 (Access_to_Care_247)
- **Version:** 1.2.0
- **Input:** KH có triệu chứng → chat/voice với AI
- **Output:** Thuốc / Xét nghiệm / Theo dõi / Cấp cứu
- **Điều kiện sử dụng:** KH ≥18 tuổi, Active, đã Onboarding (Health Overview Complete), có gói 24/7 Access to Care
- **Duration:** < 15 phút (Emergency) / Vài giờ - 7 ngày (khác)
- **Compliance:** HIPAA + DEA (Controlled Substances) + CLIA (Lab)

---

## Nguyên tắc cốt lõi

| STT | Nguyên tắc | Mô tả |
|-----|-----------|-------|
| 1 | **Ưu tiên cấp cứu** | Trường hợp khẩn cấp PHẢI được xử lý ưu tiên cao nhất, KHÔNG cần US MD approval |
| 2 | **Hoạt động 24/7** | Dịch vụ PHẢI hoạt động liên tục 24/7, không gián đoạn |
| 3 | **2 lớp duyệt bác sĩ** | Mọi chỉ định điều trị (trừ Emergency) PHẢI qua 2 lớp duyệt: VN MD → US MD |
| 4 | **3 giai đoạn chỉ định** | Quy trình: Đề xuất chỉ định → Draft Orders → Actual Orders |
| 5 | **AI sàng lọc trước** | AI PHẢI thực hiện sàng lọc lâm sàng đầu tiên trước khi chuyển cho bác sĩ |

---

## Danh sách luồng con

| Flow ID | Tên luồng | Input | Output |
|---------|----------|-------|--------|
| FLOW-A | Emergency | KH có triệu chứng KHẨN CẤP (đau ngực, khó thở nặng, đột quỵ...) | Kích hoạt quy trình ER (gọi 911, liên hệ ER gần nhất) |
| FLOW-B | Prescription | KH cần kê đơn thuốc (nhiễm trùng, đau, dị ứng...) | Chỉ định thuốc lưu vào EMR (có thể → FLOW-D) |
| FLOW-C | Lab/Imaging | KH cần xét nghiệm/chụp hình (máu, X-ray, CT...) | Chỉ định Lab/Imaging (có thể → FLOW-D) |
| FLOW-D | Monitoring | KH cần theo dõi tại nhà (từ phần chung / sau FLOW-B / sau FLOW-C) | Tình trạng KH ổn định, không cần can thiệp thêm |

---

## Phân nhánh trong Flow

| Flow | Phân nhánh | Mô tả |
|------|-----------|-------|
| FLOW-A | Không phân nhánh | Luồng đơn tuyến: Intake → AI Screening → AI Proposal → Priority → ER Protocol → END |
| FLOW-B | Nhánh kết thúc / Nhánh theo dõi | Sau khi tạo đơn thuốc → Đóng case HOẶC → Chuyển FLOW-D (nếu cần theo dõi) |
| FLOW-C | Nhánh kết thúc / Nhánh theo dõi | Sau khi tạo chỉ định XN → Đóng case HOẶC → Chuyển FLOW-D (nếu cần theo dõi) |
| FLOW-D | 3 điểm nhập | #1: Từ phần chung (đầy đủ 13 trạm) / #2: Từ FLOW-B (rút gọn trạm 9-13) / #3: Từ FLOW-C (rút gọn trạm 9-13) |

⚠️ **LƯU Ý:** FLOW-B, FLOW-C, FLOW-D (điểm nhập #1) chia sẻ **phần chung 7 trạm đầu**. FLOW-A sử dụng phần chung rút gọn (4 trạm đầu), KHÔNG cần US MD approval. Bác sĩ có thể kết hợp nhiều hướng (Thuốc + XN + Theo dõi) trong cùng 1 case.

---

## Sơ đồ luồng

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│                        PHẦN CHUNG — 3-STAGE ORDER LIFECYCLE                         │
│                                                                                      │
│  [1.Intake] → [2.AI Screening] → [3.AI Proposal + Đề xuất chỉ định]            │
│      → [4.Priority/Routing] → [5.VN MD Review + Create Draft Orders]                │
│      → [6.Draft Order Manager] → [7.US MD Unified Review]                           │
│                                                                                      │
│      → APPROVED → [Chuyển nháp → Chỉ định chính thức]                               │
│          ├── KHẨN CẤP → FLOW-A                                                      │
│          ├── Thuốc    → FLOW-B                                                       │
│          ├── Xét nghiệm → FLOW-C                                                    │
│          └── Theo dõi → FLOW-D                                                       │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

---

# PHẦN CHUNG (7 bước — dùng cho FLOW-B, C, D)

---

## Bước 1: KH chọn dịch vụ "24/7 Access to Care" từ Dashboard (Intake)

**Mô tả:** KH chọn Text Chat hoặc Voice Chat (có thể chuyển đổi giữa 2 phương thức trong phiên). Hệ thống tạo Case ID.

**INPUT/OUTPUT:**

- INPUT: user_action (String: "select_service", required), chat_mode (String: "text" | "voice", required), patient_id (String, auto from session)
- OUTPUT: case_id (String: CVH-YYYYMMDD-XXXXX, auto-generated), intake_data (Object), chat_session_id (String)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Nội dung chat | PHẢI có ít nhất 1 từ mô tả triệu chứng | "Vui lòng mô tả triệu chứng của bạn" |
| Voice Chat | PHẢI nhận diện được giọng nói | "Không nhận diện được giọng nói, vui lòng thử lại hoặc chuyển sang Text Chat" |
| Tài khoản | PHẢI Active + đã Onboarding + có gói 24/7 | "Bạn chưa đủ điều kiện sử dụng dịch vụ này" |

**UI STATES:**

```
States:
- Default: Dashboard hiển thị card "24/7 Access to Care" + button "BẮT ĐẦU TƯ VẤN"
- Chọn phương thức: Hiển thị 2 option "💬 Text Chat" và "🎙️ Voice Chat"
- Chat Active: Giao diện chatbot mở, Case ID hiển thị header, input box + nút GỬI + nút 🎙
- Voice Error: Auto chuyển Text mode + thông báo "Đang chuyển sang nhập liệu văn bản"
```

**WIREFRAME — Màn hình Chọn dịch vụ:**

```
┌─────────────────────────────────────┐
│          CVH HEALTHCARE             │
│           DASHBOARD                 │
├─────────────────────────────────────┤
│                                     │
│  Xin chào, [Tên KH]!               │
│                                     │
│  ┌─────────────────────────────┐    │
│  │ 🏥 24/7 Access to Care     │    │
│  │   Tư vấn triệu chứng      │    │
│  │   cấp tính bất kỳ lúc nào  │    │
│  │   [  BẮT ĐẦU TƯ VẤN  ]    │    │
│  └─────────────────────────────┘    │
│                                     │
│  ┌─────────────────────────────┐    │
│  │ 📅 Schedule Video Visit    │    │
│  └─────────────────────────────┘    │
│                                     │
│  ┌─────────────────────────────┐    │
│  │ 💬 Asynchronous Messaging  │    │
│  └─────────────────────────────┘    │
│                                     │
└─────────────────────────────────────┘
```

**WIREFRAME — Chọn phương thức:**

```
┌─────────────────────────────────────┐
│      24/7 ACCESS TO CARE            │
│      Chọn phương thức tư vấn        │
├─────────────────────────────────────┤
│                                     │
│  ┌─────────────────────────────┐    │
│  │ 💬 Text Chat                │    │
│  │   Nhắn tin với AI Chatbot   │    │
│  └─────────────────────────────┘    │
│                                     │
│  ┌─────────────────────────────┐    │
│  │ 🎙️ Voice Chat              │    │
│  │   Trò chuyện bằng giọng nói│    │
│  └─────────────────────────────┘    │
│                                     │
│  ⚠️ Bạn có thể chuyển đổi giữa    │
│  2 phương thức trong phiên tư vấn   │
│                                     │
└─────────────────────────────────────┘
```

**WIREFRAME — AI Chatbot mô tả triệu chứng:**

```
┌─────────────────────────────────────┐
│      AI CHATBOT — TƯ VẤN           │
│      Case ID: CVH-20260205-XXXXX    │
├─────────────────────────────────────┤
│                                     │
│  🤖 AI: Xin chào! Tôi có thể giúp │
│  gì cho bạn hôm nay?               │
│                                     │
│  👤 KH: Tôi bị đau bụng từ sáng   │
│  nay.                               │
│                                     │
│  🤖 AI: Tôi hiểu. Để giúp bạn     │
│  tốt hơn, cho tôi hỏi thêm:       │
│  Đau ở vị trí nào cụ thể?          │
│  Bên trái, bên phải, hay giữa bụng?│
│                                     │
│  ┌─────────────────────────────┐    │
│  │ Nhập tin nhắn...           │    │
│  │                    [GỬI] 🎙│    │
│  └─────────────────────────────┘    │
│                                     │
└─────────────────────────────────────┘
```

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| KH chọn Text Chat | Mở giao diện text chatbot |
| KH chọn Voice Chat | Mở giao diện voice chatbot (speech-to-text) |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Voice lỗi | Auto chuyển Text mode + thông báo |
| App sập >10 phút | CS gọi KH ghi nhận thủ công |
| Mất kết nối | Auto retry 3 lần (mỗi lần cách 2s) → thông báo "Đang kết nối lại..." |
| Ứng dụng lỗi | Auto lưu dữ liệu tạm → Khôi phục khi KH quay lại |

---

## Bước 2: AI phỏng vấn lâm sàng qua chatbot (AI Screening)

**Mô tả:** AI hỏi: triệu chứng chính, vị trí, mức độ đau (1-10), thời gian, yếu tố tăng/giảm, triệu chứng kèm theo. AI đối chiếu với tiền sử bệnh, thuốc đang dùng, dị ứng từ EMR.

**INPUT/OUTPUT:**

- INPUT: intake_data (Object, required), health_overview (Object, from EMR), ehr_data (Object: tiền sử bệnh, thuốc đang dùng, dị ứng)
- OUTPUT: clinical_data (Object: triệu chứng chi tiết, vị trí, mức độ, thời gian, tiền sử), screening_complete (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| AI Screening | AI PHẢI đối chiếu với EHR | N/A (internal) |
| Thời gian phỏng vấn | PHẢI hoàn tất ≤ 5 phút | Cảnh báo nội bộ nếu vượt 5 phút |
| Dữ liệu thu thập | PHẢI đủ: triệu chứng chính, vị trí, thời gian, mức độ đau, tính chất, yếu tố tăng/giảm, đã tự điều trị chưa, triệu chứng kèm | AI tự hỏi lại trường còn thiếu |

**UI STATES:**

```
States:
- Active: AI đang hỏi câu hỏi lâm sàng, KH trả lời
- Timeout 2 phút: AI gửi tin nhắc "Bạn vẫn đang ở đây không?"
- Timeout 5 phút: Lưu trạng thái "Tạm dừng" + gửi notification để KH quay lại
- Screening Complete: AI xác nhận đã thu thập đủ thông tin
```

**WIREFRAME:**

```
┌─────────────────────────────────────┐
│      AI CHATBOT — PHỎNG VẤN        │
│      LÂM SÀNG                      │
├─────────────────────────────────────┤
│                                     │
│  🤖 AI: Từ 1 đến 10, với 10 là     │
│  đau nhất, bạn đánh giá mức độ     │
│  đau là bao nhiêu?                  │
│                                     │
│  👤 KH: Khoảng 7.                  │
│                                     │
│  🤖 AI: Đau có lan ra chỗ khác     │
│  không? Bạn có sốt, buồn nôn,      │
│  hoặc tiêu chảy không?              │
│                                     │
│  [AI tiếp tục hỏi cho đến khi      │
│   thu thập đủ thông tin...]         │
│                                     │
│  ┌─────────────────────────────┐    │
│  │ Nhập tin nhắn...           │    │
│  │                    [GỬI] 🎙│    │
│  └─────────────────────────────┘    │
│                                     │
└─────────────────────────────────────┘
```

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| Phát hiện dấu hiệu nguy hiểm (đau ngực, khó thở nặng, đột quỵ, xuất huyết nặng, mất ý thức, chấn thương nặng, dị ứng nặng) | CHUYỂN NGAY FLOW-A |
| Thu thập đủ thông tin | Chuyển bước 3 (AI Proposal) |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| KH không phản hồi >2 phút | AI nhắc "Bạn vẫn đang ở đây không?" |
| KH không phản hồi >5 phút | Lưu "Tạm dừng" + notification |
| AI sập | CS phỏng vấn qua điện thoại, nhập thủ công |
| Thiếu trường bắt buộc | Auto hỏi lại đúng trường còn thiếu |

---

## Bước 3: AI phân tích + tạo Đề xuất chỉ định (AI Proposal)

**Mô tả:** AI tạo Differential Diagnosis + đánh giá Severity (Emergency/Urgent/Routine). AI tạo ORDER RECOMMENDATIONS: thuốc / XN / theo dõi. AI kiểm tra tương tác thuốc, dị ứng, chống chỉ định → gắn WARNING nếu có.

**INPUT/OUTPUT:**

- INPUT: clinical_data (Object, required), ehr_data (Object: tiền sử, thuốc, dị ứng)
- OUTPUT: differential_diagnosis (Array[Object: {diagnosis, probability}]), severity (String: "Emergency" | "Urgent" | "Routine"), order_recommendations (Array[Object]), ai_confidence (Number: 0-100), proposal_id (String)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Severity | PHẢI thuộc: Emergency/Urgent/Routine | N/A (internal) |
| Thời gian tạo đề xuất | PHẢI ≤ 2 phút (Emergency) / ≤ 3 phút (Lab/Imaging) | Cảnh báo nội bộ |
| Order Recommendations (thuốc) | PHẢI có: tên thuốc, liều lượng, tần suất, thời gian, lý do | N/A (internal) |
| Order Recommendations (XN) | PHẢI có: tên xét nghiệm/hình ảnh, lý do chỉ định, mức độ cần thiết | N/A (internal) |

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| Severity = EMERGENCY | CHUYỂN FLOW-A |
| AI confidence <85% | Gắn cờ "Cần xem xét lại" → MD kiểm tra ≤2 phút |
| Severity = Urgent/Routine | Chuyển bước 4 (Priority/Routing) |
| Có tương tác thuốc | Gắn WARNING trên order recommendation |

---

## Bước 4: Hệ thống tính điểm ưu tiên + phân bổ VN MD (Priority/Routing)

**Mô tả:** Điểm ưu tiên = Severity × Trọng số gói (Premium: SLA ≤1h / Plus: ≤2h / Connect: ≤4h). Auto gán VN MD theo lịch trực + gửi notification cho VN MD.

**INPUT/OUTPUT:**

- INPUT: severity (String, required), service_package (String: "Premium" | "Plus" | "Connect", required), proposal_id (String)
- OUTPUT: priority_score (Number), assigned_vn_md (String: doctor_id), sla_deadline (DateTime), queue_position (Number)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Priority Score | = Severity × Trọng số cam kết dịch vụ | N/A (internal) |
| VN MD assignment | PHẢI có VN MD available theo lịch trực | Alert CS nếu không có |

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| Severity = EMERGENCY | Priority Score = 10, đẩy đầu hàng đợi, alert CS ngay |
| Severity khác | Tính Priority Score bình thường, xếp hàng ưu tiên |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Hàng đợi sập | CS xử lý thủ công qua kênh dự phòng (SMS/Email) |
| Vượt 30 giây không vào hàng đợi | Auto cảnh báo CS mức độ nghiêm trọng |

---

## Bước 5: VN MD review Order Recommendations + tạo Draft Orders

**Mô tả:** VN MD xem AI Proposal + Order Recommendations + lịch sử dị ứng/tương tác thuốc/tiền sử. VN MD quyết định từng recommendation: **Chấp thuận / Điều chỉnh / Từ chối / Thêm mới** (ghi lý do nếu Điều chỉnh/Từ chối). VN MD tạo Draft Orders (status: chờ_duyệt_bác_sĩ_mỹ).

**INPUT/OUTPUT:**

- INPUT: proposal (Object, required), order_recommendations (Array[Object], required), patient_allergy_history (Array), drug_interaction_data (Array), medical_history (Object)
- OUTPUT: draft_orders (Array[Object: {order_id, type, details, status: "pending_usmd_review", vn_md_decision, reason}]), vn_md_notes (String)

**SLA:** Premium ≤5 phút / Plus ≤10 phút / Connect ≤15 phút

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| VN MD Review | PHẢI review từng đề xuất chỉ định | "Vui lòng review tất cả recommendations trước khi gửi" |
| Lý do Điều chỉnh/Từ chối | PHẢI ghi rõ lý do | "Vui lòng ghi rõ lý do điều chỉnh/từ chối" |
| Draft Orders | PHẢI có status hợp lệ | N/A (internal) |

**UI STATES:**

```
States:
- Queue View: Danh sách case sắp theo Priority Score, hiển thị Case ID, Triệu chứng, Score, SLA
- Case Review: Hiển thị Patient Info + AI Screening Results + AI Differential Diagnosis + ORDER RECOMMENDATIONS
- Per Recommendation: 3 button [✅ Approve] [✏️ Modify] [❌ Reject] + [+ THÊM CHỈ ĐỊNH MỚI]
- SLA Warning: Còn 3 phút → auto reminder
- SLA Exceeded: Alert CS
```

**WIREFRAME:**

```
┌─────────────────────────────────────────────────────┐
│          PROVIDER DASHBOARD — VN MD                  │
├─────────────────────────────────────────────────────┤
│                                                      │
│  📋 QUEUE (Sắp xếp theo Điểm ưu tiên)             │
│  ┌─────────────────────────────────────────────┐     │
│  │ Case ID          │ Triệu chứng │ Score │ SLA│    │
│  │ CVH-...-00123    │ Đau bụng P 7/10│ 7   │45m│    │
│  │ CVH-...-00121    │ Sốt cao 39.5  │ 6   │1h20│    │
│  │ CVH-...-00119    │ Đau đầu 3 ngày│ 5   │2h45│    │
│  └─────────────────────────────────────────────┘     │
│                                                      │
│  ═══ CASE FILE — CVH-...-00123 ═══                   │
│                                                      │
│  👤 Patient: [Tên], [Tuổi], [Giới]                  │
│  📦 Package: Care Premium | SLA: 1 giờ              │
│                                                      │
│  📝 AI Screening Results:                            │
│  - Đau bụng phải dưới, 7/10, từ sáng                │
│  - Kèm buồn nôn, không sốt                          │
│                                                      │
│  🤖 AI Differential Diagnosis:                       │
│  1. Viêm ruột thừa (65%)                            │
│  2. Viêm dạ dày ruột (20%)                          │
│  3. Sỏi thận (15%)                                  │
│                                                      │
│  📋 ORDER RECOMMENDATIONS:                           │
│  ┌──────────────────────────────────────────┐        │
│  │ #1 Drug: Amoxicillin 500mg TID x 7d     │        │
│  │    [✅ Approve] [✏️ Modify] [❌ Reject]  │        │
│  │ #2 Lab: CBC + CMP                        │        │
│  │    [✅ Approve] [✏️ Modify] [❌ Reject]  │        │
│  │ #3 Imaging: CT Abdomen STAT              │        │
│  │    [✅ Approve] [✏️ Modify] [❌ Reject]  │        │
│  └──────────────────────────────────────────┘        │
│                                                      │
│  [+ THÊM CHỈ ĐỊNH MỚI]                              │
│                                                      │
│  [    GỬI DRAFT ORDERS CHO US MD    ]                │
│                                                      │
└─────────────────────────────────────────────────────┘
```

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Còn 3 phút SLA | Auto reminder VN MD |
| Không phản hồi 10 phút | Alert CS → CS gọi VN MD nhắc |

---

## Bước 6: Hệ thống tổng hợp Draft Orders (Draft Order Manager)

**Mô tả:** Auto tổng hợp tất cả Draft Orders → tạo giao diện tổng quan cho US MD (≤1 phút). Auto gửi notification US MD.

**INPUT/OUTPUT:**

- INPUT: draft_orders (Array[Object], required)
- OUTPUT: draft_orders_summary (Object), us_md_notification_sent (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Tổng hợp | Tất cả draft PHẢI có status hợp lệ | N/A (internal) |
| Thời gian | PHẢI ≤ 1 phút | Alert CS nếu vượt |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Tổng hợp lỗi | CS liên hệ US MD qua kênh dự phòng |
| Notification thất bại | Auto retry 3 lần → alert CS |

---

## Bước 7: US MD review toàn bộ + ký duyệt (US MD Unified Review)

**Mô tả:** US MD xem AI Proposal + Clinical Data + TẤT CẢ Draft Orders (trên 1 màn hình Unified View). US MD PHẢI có license tại tiểu bang KH cư trú. US MD ký duyệt điện tử (audit log: User ID, Timestamp, IP, Device).

**INPUT/OUTPUT:**

- INPUT: proposal (Object, required), clinical_data (Object, required), all_draft_orders (Array[Object], required), patient_state (String: tiểu bang cư trú)
- OUTPUT: overall_decision (String: "Approve" | "Request_Modification"), e_signature (Object: {user_id, timestamp, ip, device}), us_md_notes (String, optional)

**SLA:** Premium ≤5 phút / Plus, Connect ≤15 phút

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| US MD License | PHẢI có license tại state của KH | "Bạn không có license tại tiểu bang của bệnh nhân" |
| E-Signature | PHẢI xác thực PIN/Password | "Vui lòng xác thực chữ ký điện tử" |
| Review | PHẢI review toàn bộ draft orders | "Vui lòng review tất cả draft orders" |

**UI STATES:**

```
States:
- Unified View: Hiển thị toàn bộ AI Proposal + Clinical Data + Draft Orders trên 1 màn hình
- Checklist: ✓ Clinical Appropriateness, ✓ Medication Safety, ✓ Lab/Imaging Justification, ✓ Regulatory Compliance, ✓ Documentation Quality
- Approve: Button [APPROVE ALL] → Form E-Signature
- Modify: Button [REQUEST MODIFICATION] → Ghi lý do + gợi ý → Quay lại VN MD
- SLA Warning: Còn 5 phút → auto reminder
```

**WIREFRAME:**

```
┌─────────────────────────────────────────────────────┐
│          US MD UNIFIED REVIEW DASHBOARD              │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ═══ CASE — CVH-...-00123 ═══                        │
│                                                      │
│  📋 TỔNG QUAN:                                       │
│  - Patient: [Tên], [Tuổi], [Giới], [State]          │
│  - AI Severity: Urgent                               │
│  - VN MD: Dr. Nguyen Van A                           │
│                                                      │
│  📝 CLINICAL DATA:                                   │
│  [Full transcript + AI analysis + VN MD notes]       │
│                                                      │
│  📦 TẤT CẢ DRAFT ORDERS:                            │
│  ┌──────────────────────────────────────────┐        │
│  │ ☐ Rx: Amoxicillin 500mg TID x 7d       │        │
│  │ ☐ Lab: CBC + CMP                        │        │
│  │ ☐ Imaging: CT Abdomen STAT              │        │
│  │ ☐ Monitor: 48h protocol                 │        │
│  └──────────────────────────────────────────┘        │
│                                                      │
│  ✓ Clinical Appropriateness                          │
│  ✓ Medication Safety                                 │
│  ✓ Lab/Imaging Justification                         │
│  ✓ Regulatory Compliance                             │
│  ✓ Documentation Quality                             │
│                                                      │
│  [  APPROVE ALL  ]  [  REQUEST MODIFICATION  ]       │
│                                                      │
│  🔐 E-Signature: [PIN/Password] [XÁC NHẬN]          │
│                                                      │
└─────────────────────────────────────────────────────┘
```

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| APPROVE → Thuốc | Chuyển FLOW-B bước 8 |
| APPROVE → Xét nghiệm | Chuyển FLOW-C bước 8 |
| APPROVE → Theo dõi | Chuyển FLOW-D bước 8 |
| APPROVE → Kết hợp nhiều hướng | Thực hiện song song nhiều flow |
| YÊU CẦU CHỈNH SỬA | Ghi lý do + gợi ý → quay lại bước 5 (VN MD chỉnh sửa) → gửi lại US MD |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Còn 5 phút SLA | Auto reminder US MD |
| Không phản hồi 10 phút | Alert CS → CS gọi US MD |
| US MD không available | Chuyển bác sĩ trực on-call |

---

# FLOW-A: EMERGENCY (< 15 phút, KHÔNG cần US MD)

> Dùng bước 1-2 phần chung → tiếp:

---

## Bước 3A: AI xác định Severity = EMERGENCY

**Mô tả:** AI gắn nhãn EMERGENCY + tạo đề xuất Emergency Protocol (≤2 phút).

**INPUT/OUTPUT:**

- INPUT: clinical_data (Object, required)
- OUTPUT: severity (String: "Emergency"), emergency_proposal (Object), ai_confidence (Number)

**Dấu hiệu nguy hiểm — Triệu chứng PHẢI kích hoạt FLOW-A:**

| STT | Triệu chứng |
|-----|-------------|
| 1 | Đau ngực (chest pain) |
| 2 | Khó thở nặng (severe shortness of breath) |
| 3 | Triệu chứng đột quỵ (FAST: Face, Arms, Speech, Time) |
| 4 | Xuất huyết nặng không kiểm soát được |
| 5 | Mất ý thức hoặc thay đổi ý thức đột ngột |
| 6 | Chấn thương nặng |
| 7 | Phản ứng dị ứng nghiêm trọng (anaphylaxis) |

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| AI confidence ≥85% | Chuyển bước 4A |
| AI confidence <85% | MD xác nhận/điều chỉnh ≤2 phút |

---

## Bước 4A: Hệ thống set Priority Score = 10 → đẩy đầu hàng đợi → alert CS

**INPUT/OUTPUT:**

- INPUT: severity (String: "Emergency", required)
- OUTPUT: priority_score (Number: 10), queue_position (Number: 1), cs_alert_sent (Boolean)

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Hàng đợi sập | CS nhận SMS/Email dự phòng |
| Vượt 30 giây | Auto cảnh báo CS mức nghiêm trọng |

---

## Bước 5A: CS/Điều phối viên kích hoạt ER Protocol (≤10 phút từ khi nhận alert)

**Mô tả:** CS gọi KH xác nhận tình trạng (≤2 phút) → Gọi 911 → Liên hệ ER gần nhất → Gửi thông báo người thân → Log đầy đủ. MD standby tư vấn nếu CS yêu cầu.

**INPUT/OUTPUT:**

- INPUT: patient_info (Object: tên, SĐT, địa chỉ, tình trạng), emergency_proposal (Object)
- OUTPUT: er_contacted (Boolean), call_911_log (Object), referral_letter (Object), family_notified (Boolean), er_protocol_log (Object)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Thời gian phản hồi | CS PHẢI liên hệ KH ≤2 phút từ alert | Cảnh báo khi còn 2 phút đến ngưỡng 10 phút |
| Checklist ER | PHẢI hoàn thành tất cả bước | Auto cảnh báo CS nếu bỏ sót |

**WIREFRAME:**

```
┌─────────────────────────────────────┐
│      ⚠️ KHẨN CẤP — ER PROTOCOL     │
├─────────────────────────────────────┤
│                                     │
│  🚨 TÌNH TRẠNG KHẨN CẤP           │
│                                     │
│  Triệu chứng của bạn cần được      │
│  cấp cứu ngay lập tức!             │
│                                     │
│  📞 GỌI 911 NGAY                   │
│  [   GỌI 911   ]                   │
│                                     │
│  🏥 BỆNH VIỆN GẦN NHẤT:           │
│  1. [Tên BV] - 2.3 km              │
│     📍 [Địa chỉ]                   │
│     📞 [Số ĐT]                     │
│     [CHỈ ĐƯỜNG]                     │
│                                     │
│  2. [Tên BV] - 4.1 km              │
│     📍 [Địa chỉ]                   │
│     📞 [Số ĐT]                     │
│     [CHỈ ĐƯỜNG]                     │
│                                     │
│  📄 Thư giới thiệu đã được tạo     │
│  và gửi qua email/app.             │
│                                     │
│  👩‍⚕️ Điều phối viên chăm sóc sẽ gọi cho   │
│  bạn trong vòng 5 phút.            │
│                                     │
└─────────────────────────────────────┘
```

**⛔ Rẽ nhánh (trường hợp đặc biệt):**

| Điều kiện | Hành động |
|-----------|----------|
| Không liên lạc được KH | Gọi 911 với địa chỉ trong hồ sơ |
| KH từ chối đi ER nhưng nghiêm trọng | Ghi nhận từ chối điều trị (refusal), theo dõi 1 giờ |
| AI không xác định được severity | Chuyển Điều phối viên đánh giá qua điện thoại |

---

## Bước 6A: Hệ thống đóng case (≤2 phút)

**INPUT/OUTPUT:**

- INPUT: er_protocol_log (Object, required)
- OUTPUT: case_status (String: "Emergency_Closed"), emr_timeline (Object), notification_sent (Boolean)

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Đóng lỗi | CS đóng thủ công + xác nhận KH qua điện thoại |
| Notification thất bại | Auto retry 3 lần |

---

# FLOW-B: PRESCRIPTION (Phần chung 7 bước + 2 bước)

> Sau bước 7 (US MD Approve):

---

## Bước 8B: Hệ thống chuyển Draft Orders → Đơn thuốc chính thức (≤2 phút)

**Mô tả:** Chuyển draft → actual Rx orders → Lưu bảng ac247_actual_orders → Tạo prescription document → Cập nhật EMR. KH tự đến pharmacy (KHÔNG auto chọn nhà thuốc/chuyển e-prescription).

**INPUT/OUTPUT:**

- INPUT: draft_orders (Array[Object], status: "usmd_approved", required)
- OUTPUT: actual_rx_orders (Array[Object], saved to ac247_actual_orders), prescription_document (Object), emr_updated (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Draft Orders | CHỈ chuyển khi status = usmd_approved | N/A (internal) |
| Tương tác thuốc | Kiểm tra drug-drug interaction | Cảnh báo VN MD đổi thuốc |
| Dị ứng thuốc | Kiểm tra patient allergy | CHẶN đơn, VN MD PHẢI chọn thuốc khác |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Chuyển đổi lỗi | Retry 3 lần (mỗi lần cách 2s) → CS báo kỹ thuật, giữ case treo |
| Phát hiện tương tác thuốc | Cảnh báo VN MD đổi thuốc |
| Dị ứng thuốc | CHẶN đơn, VN MD PHẢI chọn thuốc khác |

---

## Bước 9B: Hệ thống hoàn tất case (≤3 phút)

**Mô tả:** Status = "Hoàn_tất" → Gửi đơn thuốc cho KH qua App + Email → Lưu timeline EMR → Đóng case.

**INPUT/OUTPUT:**

- INPUT: actual_rx_orders (Array[Object], required)
- OUTPUT: case_status (String: "Hoàn_tất"), notification_sent (Boolean), emr_timeline (Object)

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| KHÔNG cần theo dõi | Case Closed |
| CẦN theo dõi tác dụng phụ/hiệu quả | Chuyển FLOW-D bước 9D (bỏ qua bước 1-8) |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Gửi lỗi | Retry 3 lần → CS gửi thủ công qua email |

---

# FLOW-C: LAB/IMAGING (Phần chung 7 bước + 2 bước)

> Sau bước 7 (US MD Approve):

---

## Bước 8C: Hệ thống chuyển Draft → Chỉ định XN chính thức (≤2 phút)

**Mô tả:** Chuyển draft → actual lab/imaging orders → Tạo giấy chỉ định → Cập nhật EMR. Auto gửi giấy chỉ định + hướng dẫn cho KH qua App. KH tự đến cơ sở XN (KHÔNG auto đặt lịch).

**INPUT/OUTPUT:**

- INPUT: draft_orders (Array[Object], status: "usmd_approved", required)
- OUTPUT: actual_lab_orders (Array[Object]), lab_requisition_document (Object), emr_updated (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Draft Orders | CHỈ chuyển khi status = usmd_approved | N/A (internal) |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Chuyển đổi lỗi | Retry 3 lần → CS báo kỹ thuật, hỗ trợ KH thủ công |

---

## Bước 9C: Hệ thống hoàn tất

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| KHÔNG cần theo dõi | Case Closed |
| CẦN theo dõi sau XN | Chuyển FLOW-D bước 9D (bỏ qua bước 1-8) |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Kết quả XN có giá trị nguy hiểm | Cảnh báo khẩn cấp VN MD + US MD + gọi KH ngay |

---

# FLOW-D: MONITORING

## 3 điểm nhập:

| Từ đâu | Bắt đầu | US MD Approval |
|--------|---------|---------------|
| Phần chung (bước 7) | Bước 8D (đủ 13 bước) | CẦN |
| FLOW-B (bước 9B) | Bước 9D (5 bước rút gọn) | ĐÃ CÓ từ FLOW-B |
| FLOW-C (bước 9C) | Bước 9D (5 bước rút gọn) | ĐÃ CÓ từ FLOW-C |

### So sánh các điểm nhập FLOW-D

| Tiêu chí | Điểm nhập #1 (Từ phần chung) | Điểm nhập #2 (Từ FLOW-B) | Điểm nhập #3 (Từ FLOW-C) |
|----------|------------------------------|--------------------------|--------------------------|
| **Số trạm** | 13 (đầy đủ) | 5 (rút gọn: trạm 9-13) | 5 (rút gọn: trạm 9-13) |
| **US MD Approval** | CẦN | ĐÃ CÓ từ FLOW-B | ĐÃ CÓ từ FLOW-C |
| **3-Stage Lifecycle** | CÓ | KHÔNG | KHÔNG |
| **Check-in nội dung** | Theo dõi triệu chứng chung | Thêm: tuân thủ thuốc, tác dụng phụ | Thêm: thay đổi sau XN, tuân thủ kế hoạch |

---

### Chỉ điểm nhập #1 — Bước 8D (4 bước phụ):

## Bước 8D-a: AI đề xuất protocol theo dõi (≤1 phút)

**Mô tả:** AI đề xuất: protocol 24h/48h/72h + thông số cần theo dõi.

**INPUT/OUTPUT:**

- INPUT: clinical_data (Object, required), severity (String)
- OUTPUT: monitoring_proposal (Object: {protocol: "24h" | "48h" | "72h", parameters: Array, interval: String})

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| Protocol không phù hợp | MD can thiệp điều chỉnh |
| Đề xuất OK | Chuyển VN MD review |

---

## Bước 8D-b: VN MD duyệt đề xuất theo dõi

**Mô tả:** VN MD xác nhận/điều chỉnh protocol + thông số + tạo chỉ định theo dõi + hướng dẫn KH.

**INPUT/OUTPUT:**

- INPUT: monitoring_proposal (Object, required)
- OUTPUT: monitoring_order (Object: {protocol, parameters, instructions}), vn_md_decision (String)

**SLA:** Premium ≤5 phút / Plus ≤10 phút / Connect ≤15 phút

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Còn 3 phút SLA | Auto reminder VN MD |
| Không phản hồi 10 phút | Alert CS → CS gọi VN MD |

---

## Bước 8D-c: Hệ thống tạo lệnh điều trị (≤1 phút) → US MD review + ký duyệt

**Mô tả:** US MD xác nhận hợp lệ y khoa/pháp lý Mỹ → ký điện tử → status = "đã_phê_duyệt".

**INPUT/OUTPUT:**

- INPUT: monitoring_order (Object, required)
- OUTPUT: treatment_order (Object), us_md_approval (Object: {e_signature, timestamp}), status (String: "đã_phê_duyệt")

**SLA US MD:** Premium ≤5 phút / Plus, Connect ≤10 phút

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| Phê duyệt | Chuyển bước 8D-d |
| Yêu cầu điều chỉnh | Quay lại VN MD |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Không phản hồi | Alert CS → CS gọi US MD |

---

## Bước 8D-d: Hệ thống tạo Monitor Order chính thức (≤2 phút)

**Mô tả:** Gán Tracking ID → Lưu bảng lệnh theo dõi → Cập nhật EMR → Notification KH.

**INPUT/OUTPUT:**

- INPUT: treatment_order (Object, status: "đã_phê_duyệt", required)
- OUTPUT: monitor_order (Object), tracking_id (String, unique), emr_updated (Boolean), notification_sent (Boolean)

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Lỗi tạo lệnh | Retry 3 lần → alert CS |
| Notification thất bại | Retry 3 lần → alert CS |

---

### Tất cả điểm nhập — Bước 9D-13D:

## Bước 9D: Hệ thống thiết lập protocol (≤3 phút)

**Mô tả:** Thiết lập lịch check-in: 24h (nhẹ) / 48h (trung bình, 2 lần) / 72h (phức tạp, 3 lần). Tạo mốc nhắc nhở tự động + cấu hình chatbot AI → Kích hoạt protocol.

**INPUT/OUTPUT:**

- INPUT: monitor_order (Object, required) HOẶC monitoring_context_from_flow_b_c (Object)
- OUTPUT: protocol_active (Boolean), check_in_schedule (Array[DateTime]), chatbot_configured (Boolean)

**Các lịch theo dõi (Monitoring Protocols):**

| Protocol | Lịch check-in | Trường hợp sử dụng | Thời gian |
|----------|--------------|---------------------|----------|
| **24h Protocol** | Check-in sau 24h | Triệu chứng nhẹ, rủi ro thấp | 24h |
| **48h Protocol** | Check-in sau 24h, 48h | Triệu chứng trung bình, cần 2 lần check-in | 48h |
| **72h Protocol** | Check-in sau 24h, 48h, 72h | Triệu chứng dai dẳng, phức tạp | 72h |

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Protocol | PHẢI thuộc: 24h/48h/72h | N/A (internal) |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Protocol không kích hoạt | Retry 3 lần → CS báo kỹ thuật |

---

## Bước 10D: AI check-in lần 1 với KH qua chatbot (≤10 phút)

**Mô tả:** AI gửi nhắc check-in (2h trước lịch) → KH trả lời: tình trạng (Cải thiện/Giữ nguyên/Xấu đi), mức đau 1-10, triệu chứng mới, tuân thủ thuốc, tác dụng phụ. AI lưu + phân tích → gắn cờ đỏ nếu bất thường.

**INPUT/OUTPUT:**

- INPUT: protocol (Object, required), check_in_schedule (DateTime)
- OUTPUT: status_update_1 (Object: {symptom_status, pain_level, new_symptoms, medication_compliance, side_effects}), red_flag (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| Triệu chứng hiện tại | PHẢI chọn 1 trong 3 option | "Vui lòng chọn tình trạng triệu chứng" |
| Mức độ đau | PHẢI là số từ 1-10 | "Vui lòng nhập số từ 1 đến 10" |
| Triệu chứng mới | Nếu chọn "Có" → PHẢI mô tả | "Vui lòng mô tả triệu chứng mới" |
| Tác dụng phụ | Nếu chọn "Có" → PHẢI mô tả | "Vui lòng mô tả tác dụng phụ" |

**WIREFRAME:**

```
┌─────────────────────────────────────┐
│      📊 CHECK-IN MONITORING         │
│      Lần 1 / 24h Protocol           │
├─────────────────────────────────────┤
│                                     │
│  🤖 AI: Chào bạn! Đã đến lúc       │
│  cập nhật tình trạng sức khỏe.      │
│                                     │
│  1. Triệu chứng hiện tại:          │
│  ○ Cải thiện                        │
│  ○ Giữ nguyên                       │
│  ○ Xấu đi                          │
│                                     │
│  2. Mức độ đau hiện tại (1-10):    │
│  [____]                             │
│                                     │
│  3. Có triệu chứng mới không?      │
│  ○ Không                            │
│  ○ Có → [Mô tả: ____________]      │
│                                     │
│  4. Đã dùng thuốc đều đặn chưa?    │
│     (nếu có đơn thuốc)             │
│  ○ Có                               │
│  ○ Không → [Lý do: __________]      │
│                                     │
│  5. Có tác dụng phụ không?          │
│  ○ Không                            │
│  ○ Có → [Mô tả: ____________]      │
│                                     │
│  📷 [Tải ảnh lên] (nếu cần)        │
│  🎙️ [Ghi âm] (nếu cần)            │
│                                     │
│  [    GỬI CHECK-IN    ]             │
│                                     │
└─────────────────────────────────────┘
```

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| KH không phản hồi 30 phút | Auto nhắc lần 2 |
| KH không phản hồi 1 giờ | Nhắc lần 3 + alert nội bộ |
| KH không phản hồi 4 giờ | Alert CS → CS gọi KH |
| Cờ đỏ (tình trạng xấu nghiêm trọng) | Alert MD → BS liên hệ KH trực tiếp |

---

## Bước 11D: AI check-in lần 2/3 (≤10 phút/lần)

**Mô tả:** Giống bước 10D + AI so sánh xu hướng với lần check-in trước.

**INPUT/OUTPUT:**

- INPUT: protocol (Object, required), previous_status_updates (Array[Object])
- OUTPUT: status_update_n (Object), trend_analysis (Object: {direction: "improving" | "stable" | "worsening"}), red_flag (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| AI | PHẢI so sánh với baseline (lần check-in trước) | N/A (internal) |

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| Triệu chứng XẤU ĐI | Chuyển VN MD → có thể tạo case mới |
| Triệu chứng KHẨN CẤP | Chuyển FLOW-A |
| Cần thêm XN | Chuyển FLOW-C |
| Cần thay đổi thuốc | Chuyển FLOW-B |
| ỔN ĐỊNH | Tiếp bước 12D |

---

## Bước 12D: BS VN/Mỹ đánh giá tổng kết (≤30 phút sau check-in cuối)

**Mô tả:** BS review toàn bộ dữ liệu check-in → dashboard xu hướng → kết luận cuối (hồi phục / cần can thiệp / chuyển dịch vụ) → ghi khuyến nghị → lưu EMR.

**INPUT/OUTPUT:**

- INPUT: all_status_updates (Array[Object], required), trend_analysis (Object)
- OUTPUT: final_conclusion (String: "recovered" | "need_intervention" | "transfer_service"), recommendations (String), emr_updated (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| BS Review | PHẢI review trong vòng 6h sau check-in cuối | Alert CS nếu vượt |
| Thời gian đánh giá | PHẢI ≤ 30 phút | Reminder khi còn 5 phút |

**⛔ Rẽ nhánh:**

| Điều kiện | Hành động |
|-----------|----------|
| Đã hồi phục | Chuyển bước 13D |
| Cần can thiệp thêm | Chuyển dịch vụ khác hoặc tạo case mới |

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Còn 5 phút SLA | Auto reminder BS |
| Không phản hồi 20 phút | Alert CS → CS gọi BS |

---

## Bước 13D: Hệ thống hoàn tất case (≤1 phút)

**Mô tả:** Status = "Hoàn_tất" → Tạo báo cáo tổng kết → Gửi KH qua App + Email → Lưu EMR → Đóng case.

**INPUT/OUTPUT:**

- INPUT: final_conclusion (Object, required), all_case_data (Object)
- OUTPUT: case_status (String: "Hoàn_tất"), summary_report (Object), notification_sent (Boolean), emr_timeline (Object)

**WIREFRAME — Tóm tắt tư vấn:**

```
┌─────────────────────────────────────┐
│      ✅ TƯ VẤN HOÀN TẤT            │
│      Case ID: CVH-20260205-XXXXX    │
├─────────────────────────────────────┤
│                                     │
│  📋 TÓM TẮT TƯ VẤN                │
│                                     │
│  🏥 Triệu chứng: Đau bụng phải    │
│  dưới, 7/10, kèm buồn nôn         │
│                                     │
│  🩺 Chẩn đoán sơ bộ:               │
│  Nghi viêm ruột thừa               │
│                                     │
│  💊 Đơn thuốc:                      │
│  Amoxicillin 500mg - 3 lần/ngày    │
│  x 7 ngày                          │
│  → Đã lưu vào EMR                  │
│                                     │
│  🔬 Xét nghiệm:                    │
│  CT Scan Bụng (STAT)               │
│  → Chỉ định đã tạo                 │
│                                     │
│  📊 Theo dõi:                       │
│  Check-in sau 24h, 48h             │
│                                     │
│  ⚠️ DẤU HIỆU CẦN BÁO NGAY:       │
│  - Đau tăng nhanh hoặc lan rộng    │
│  - Sốt cao > 38.5°C                │
│  - Nôn ra máu hoặc phân đen        │
│  → Gọi CVH 24/7 hoặc đến ER ngay  │
│                                     │
│  [XEM CHI TIẾT]  [TẢI PDF]         │
│                                     │
│  ⭐ Đánh giá trải nghiệm           │
│  ☆ ☆ ☆ ☆ ☆                        │
│                                     │
└─────────────────────────────────────┘
```

**⚠️ Lỗi:**

| Lỗi | Xử lý |
|-----|-------|
| Gửi lỗi | Retry 3 lần → CS gửi thủ công |

---

# THÔNG BÁO TỰ ĐỘNG

## Bảng tổng hợp

| Sự kiện | Kênh | Người nhận |
|---------|------|-----------|
| Tạo Case ID | SMS | KH |
| Severity = EMERGENCY | SMS + Push + Gọi điện | KH |
| Đơn thuốc tạo xong | SMS + App | KH |
| 2h trước check-in | Push + SMS | KH |
| Quá hạn check-in 2h | Push + SMS | KH |
| Quá hạn check-in 6h | Gọi điện (CS gọi) | KH |
| Case mới cần review | Push + Email | VN MD |
| Case hoàn tất | Email (tóm tắt + đơn thuốc/giấy XN/hướng dẫn) | KH |

## Template chi tiết

📱 **SMS xác nhận tiếp nhận:**
- Thời điểm: Ngay sau khi Case ID được tạo (Trạm 1)
- Nội dung: "[CVH] Yêu cầu tư vấn của bạn đã được ghi nhận (Case: [Case ID]). Bác sĩ sẽ xem xét trong thời gian sớm nhất theo gói dịch vụ của bạn."

🚨 **SMS + Push Emergency:**
- Thời điểm: Khi AI xác định Severity = EMERGENCY (Trạm 3 FLOW-A)
- SMS: "⚠️ [CVH] KHẨN CẤP: Triệu chứng của bạn cần cấp cứu ngay! Gọi 911 hoặc đến ER gần nhất. Điều phối viên chăm sóc sẽ gọi cho bạn ngay."
- Push: "🚨 KHẨN CẤP — Nhấn để xem hướng dẫn cấp cứu và bệnh viện gần nhất"

📧 **Email tóm tắt tư vấn:**
- Thời điểm: Sau khi case hoàn tất
- Subject: "[CVH] Tóm tắt Tư vấn Y tế - [DD/MM/YYYY]"
- Body: Thông tin case, triệu chứng, chẩn đoán sơ bộ, đơn thuốc (nếu có), xét nghiệm (nếu có), hướng dẫn theo dõi, dấu hiệu cần báo ngay, lịch tái khám
- Đính kèm: Lab Requisition, Patient Education Sheet, Monitoring Instructions

💊 **SMS + Push đơn thuốc:**
- Thời điểm: Sau khi Actual Rx Order được tạo (FLOW-B, Trạm 8)
- Nội dung: "[CVH] Đơn thuốc của bạn đã được tạo và lưu vào hồ sơ y tế. Xem chi tiết trong app."

📊 **Push + SMS nhắc check-in:**
- Thời điểm: 2 giờ trước lịch check-in (FLOW-D)
- Nội dung: "[CVH] Đã đến lúc check-in tình trạng sức khỏe! Nhấn để cập nhật."

⏰ **Push + SMS quá hạn check-in:**
- Thời điểm: Quá hạn 2 giờ so với lịch check-in
- Nội dung: "[CVH] Bạn chưa hoàn thành check-in. Vui lòng cập nhật tình trạng sức khỏe ngay."

📞 **Gọi điện khi không phản hồi:**
- Thời điểm: Quá hạn 6 giờ so với lịch check-in
- Người thực hiện: Điều phối viên chăm sóc
- Nội dung: Kiểm tra tình trạng KH, xác nhận KH an toàn, cập nhật monitoring

🩺 **Push + Email cho VN MD:**
- Thời điểm: Khi case mới được route đến VN MD (Trạm 4)
- Nội dung: "[CVH] Case mới: [Case ID] - [Tóm tắt triệu chứng] - Priority: [Score] - SLA: [Countdown]"

---

# BẢNG GIÁM SÁT CS-MD ĐẦY ĐỦ

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Intake | Ghi nhận đúng/đủ dữ liệu; chatbot ổn định; không lỗi kỹ thuật | 🤖 AUTO 100%: Hướng dẫn tự động; Voice lỗi → Text; Mất kết nối → retry 3 lần (2s); Lưu tạm + khôi phục | ⚠️ App/Chatbot sập >10 phút sau 3 retry → CS gọi KH ghi nhận thủ công | ❌ Không can thiệp |
| AI Screening | Hoàn tất ≤ 5 phút; câu hỏi phù hợp; dữ liệu đủ; ghi nhận chính xác | 🤖 AUTO 100%: Tự động phỏng vấn; Timeout 2 phút → nhắc; 5 phút → "Tạm dừng" + notification; Thiếu trường → hỏi lại | ⚠️ AI sập hoàn toàn sau 3 retry / vòng lặp không thoát → CS phỏng vấn qua điện thoại, nhập thủ công | ❌ Không can thiệp |
| AI Proposal (Emergency) | Tạo đề xuất ≤ 2 phút; phân tích đúng tiêu chí Emergency; Severity đúng | 🤖 AUTO 100%: Phân tích SOAP data; đánh giá tiêu chí Emergency; gắn nhãn Severity; tạo Emergency Protocol; verify | ❌ Không can thiệp | ⚠️ AI confidence <85% → MD xác nhận/điều chỉnh ≤2 phút |
| AI Proposal (+ Order Recommendations) | Tạo đề xuất ≤ 2 phút; đủ thông tin thuốc; kiểm tra tương tác/dị ứng/chống chỉ định | 🤖 AUTO 100%: Phân tích + lịch sử y tế; kiểm tra drug interaction; tạo Order Recommendations đầy đủ; gắn WARNING | ❌ Không can thiệp | ⚠️ WARNING tương tác thuốc nghiêm trọng / đề xuất không phù hợp → MD điều chỉnh |
| AI Proposal (+ XN/CĐHA) | Tạo đề xuất ≤ 3 phút; đủ thông tin XN; kiểm tra chống chỉ định | 🤖 AUTO 100%: Tạo đề xuất XN/CĐHA; kiểm tra chống chỉ định; gắn WARNING | ❌ Không can thiệp | ⚠️ WARNING chống chỉ định / đề xuất không phù hợp → MD điều chỉnh |
| AI Proposal (Theo dõi) | Tạo đề xuất ≤ 1 phút; protocol 24h/48h/72h; thông số theo dõi đủ | 🤖 AUTO 100%: Tạo đề xuất protocol; xác định interval; liệt kê thông số | ❌ Không can thiệp | ⚠️ Protocol không phù hợp → MD điều chỉnh |
| Priority/Routing | Vào hàng đợi ngay lập tức; Priority Score đúng; cảnh báo CS | 🤖 AUTO 100%: Set Priority Score; đẩy đầu hàng đợi (Emergency); gửi alert CS; verify vị trí | ⚠️ Hàng đợi sập sau 3 retry → CS xử lý thủ công qua SMS/Email | ❌ Không can thiệp |
| VN MD Review | Premium ≤5 phút / Plus ≤10 phút / Connect ≤15 phút; review tất cả recommendations | 🤖 95%: Auto assign VN MD; hiển thị thông tin; đo thời gian; nhắc 3 phút trước SLA | ⚠️ VN MD không phản hồi <3 phút SLA → CS gọi VN MD khẩn cấp | ✅ VN MD: Review + Chấp thuận/Điều chỉnh/Từ chối/Thêm mới + ghi lý do |
| Draft Order Manager | Tổng hợp ≤ 1 phút; đầy đủ; đúng format; notification US MD | 🤖 AUTO 100%: Tổng hợp Draft Orders; tạo giao diện tổng quan; gửi notification US MD | ⚠️ Tổng hợp lỗi sau 3 retry / notification thất bại → CS liên hệ US MD kênh dự phòng | ❌ Không can thiệp |
| US MD Unified Review | Premium ≤5 phút / Plus,Connect ≤15 phút; review toàn bộ; xác nhận hợp lệ y khoa/pháp lý | 🤖 95%: Auto assign US MD; Unified View; highlight WARNING; đo thời gian; nhắc 5 phút trước SLA | ⚠️ US MD không phản hồi <5 phút SLA → CS gọi US MD khẩn cấp | ✅ US MD: Review toàn bộ + xác nhận y khoa/pháp lý Mỹ + Phê duyệt/Điều chỉnh/Từ chối |
| ER Protocol | CS liên hệ KH + gọi 911/ER ≤ 10 phút | 🤖 95%: Tổng hợp thông tin KH; hiển thị checklist ER; ghi nhận mốc thời gian; đo thời gian; cảnh báo 2 phút trước ngưỡng | ✅ CS thực hiện: Gọi KH ≤2 phút; Gọi 911; Liên hệ ER; Gửi thông báo người thân; Log đầy đủ | ⚠️ MD standby tư vấn nếu CS yêu cầu |
| Convert Draft to Actual Rx | Chuyển đổi ≤ 2 phút; status đúng; lưu EMR | 🤖 AUTO 100%: Chuyển draft → actual; cập nhật status; lưu bảng actual_orders; tạo prescription; cập nhật EMR; retry 3 lần nếu lỗi | ⚠️ Chuyển đổi thất bại sau 3 retry → CS báo kỹ thuật, giữ case treo | ❌ Không can thiệp |
| Convert Draft to Actual Lab | Chuyển đổi ≤ 2 phút; status đúng; lưu EMR | 🤖 AUTO 100%: Chuyển draft → actual; tạo giấy chỉ định; cập nhật EMR; gửi hướng dẫn KH; retry 3 lần | ⚠️ Chuyển đổi thất bại sau 3 retry → CS báo kỹ thuật, hỗ trợ KH thủ công | ❌ Không can thiệp |
| VN MD Review (Theo dõi) | SLA theo gói; duyệt protocol; tạo chỉ định; hướng dẫn KH | 🤖 95%: Auto assign; hiển thị đề xuất AI; đo thời gian; nhắc 3 phút trước SLA | ⚠️ VN MD không phản hồi <3 phút SLA → CS gọi VN MD | ✅ VN MD: Duyệt/điều chỉnh protocol + thông số + hướng dẫn KH |
| US MD Review (Theo dõi) | Premium ≤5 phút / Plus,Connect ≤10 phút | 🤖 95%: Auto assign; hiển thị lệnh điều trị; highlight; đo thời gian; nhắc 3 phút trước SLA | ⚠️ US MD không phản hồi <3 phút SLA → CS gọi US MD | ✅ US MD: Review + xác nhận y khoa/pháp lý Mỹ + phê duyệt/điều chỉnh |
| US MD Approval (Theo dõi) | Ký duyệt ≤ 1 phút; chữ ký điện tử; xác nhận trách nhiệm | 🤖 95%: Hiển thị form ký duyệt; ghi nhận mốc thời gian; cập nhật status "đã_phê_duyệt"; trigger Monitor Order; verify chữ ký; log audit | ⚠️ Trigger không kích hoạt / chữ ký lỗi → CS báo kỹ thuật | ✅ US MD: Ký điện tử xác nhận |
| Monitor Order | Tạo lệnh ≤ 2 phút; Tracking ID duy nhất; lưu EMR; notification KH | 🤖 AUTO 100%: Tạo lệnh theo dõi; gán Tracking ID; lưu EMR; notification KH; retry 3 lần | ⚠️ Tạo lệnh thất bại / notification thất bại → CS xác nhận thủ công | ❌ Không can thiệp |
| Setup Protocol | Thiết lập ≤ 3 phút; lịch check-in đúng; chatbot cấu hình; kích hoạt | 🤖 AUTO 100%: Đọc chỉ định; thiết lập lịch 24h/48h/72h; tạo mốc nhắc; cấu hình chatbot; kích hoạt; verify; retry 3 lần | ⚠️ Protocol không kích hoạt sau 3 retry → CS báo kỹ thuật | ❌ Không can thiệp |
| Check-in 1 | Nhắc đúng thời điểm; hoàn tất ≤ 10 phút; câu hỏi phù hợp; dữ liệu đủ; chatbot ổn định | 🤖 95%: Tự động kích hoạt; gửi nhắc 2h trước; AI hỏi câu lâm sàng; lưu dữ liệu; phân tích; gắn cờ đỏ nếu bất thường; alert MD | ⚠️ KH không phản hồi 4 giờ / chatbot lỗi → CS gọi KH check-in qua điện thoại | ⚠️ Cờ đỏ: tình trạng xấu nghiêm trọng → BS liên hệ KH trực tiếp |
| Check-in 2/3 | Như Check-in 1 + so sánh xu hướng; cảnh báo vượt ngưỡng | 🤖 95%: Như Check-in 1 + so sánh lần trước; phát hiện xu hướng; cờ đỏ/vượt ngưỡng → alert MD | ⚠️ KH không phản hồi 4 giờ / chatbot lỗi → CS gọi KH | ⚠️ Cờ đỏ / vượt ngưỡng / cần đánh giá lại → BS liên hệ KH + điều chỉnh kế hoạch |
| Final Review | BS hoàn tất ≤ 30 phút; duyệt toàn bộ check-in; đánh giá tổng thể; kết luận + khuyến nghị; lưu EMR | 🤖 95%: Tổng hợp dữ liệu check-in; tạo dashboard xu hướng; notification BS; đo thời gian; nhắc 5 phút trước SLA | ⚠️ BS không hoàn tất sau 30 phút → CS gọi BS khẩn cấp | ✅ BS VN/Mỹ: Review toàn bộ; đánh giá tiến triển; kết luận; khuyến nghị; quyết định chuyển dịch vụ |
| Completion (Emergency) | Đóng case ≤ 2 phút; status = "Emergency_Closed"; timeline EMR; notification KH | 🤖 AUTO 100%: Cập nhật status; lưu timeline EMR; notification KH; verify; retry 3 lần | ⚠️ Đóng lỗi / notification thất bại → CS đóng thủ công + xác nhận KH | ❌ Không can thiệp |
| Completion (Prescription) | Hoàn tất ≤ 3 phút; status = "Hoàn_tất"; gửi đơn thuốc App+Email; timeline EMR | 🤖 AUTO 100%: Cập nhật status; gửi đơn thuốc App+Email; lưu timeline EMR; đóng case; retry 3 lần | ⚠️ Notification/đơn thuốc thất bại sau 3 retry → CS gửi thủ công | ❌ Không can thiệp |
| Completion (Monitoring) | Hoàn tất ≤ 1 phút; status = "Hoàn_tất"; báo cáo tổng kết App+Email; timeline EMR | 🤖 AUTO 100%: Cập nhật status; tạo báo cáo tổng kết; gửi App+Email; lưu EMR; đóng case; retry 3 lần | ⚠️ Notification/báo cáo thất bại sau 3 retry → CS gửi thủ công | ❌ Không can thiệp |

---

# DATA SPECIFICATION

## Quy tắc kiểm tra (Validation Rules)

| Node ID | Tên trạm | Quy tắc kiểm tra |
|---------|---------|------------------|
| COMMON-01 | Intake | Triệu chứng PHẢI có nội dung |
| COMMON-02 | AI Screening | AI PHẢI đối chiếu với EHR |
| COMMON-03 | AI Proposal | Mức độ nghiêm trọng PHẢI thuộc: Emergency/Urgent/Routine |
| COMMON-04 | Priority/Routing | Điểm ưu tiên = Mức độ nghiêm trọng × Trọng số cam kết dịch vụ |
| COMMON-05 | VN MD Review | VN MD PHẢI review từng đề xuất chỉ định |
| COMMON-06 | Draft Order Manager | Tất cả draft PHẢI có status hợp lệ |
| COMMON-07 | US MD Unified Review | US MD PHẢI có license tại state của KH |
| FLOW-A-05 | ER Protocol | Thời gian phản hồi < 5 phút |
| FLOW-B-08 | Chuyển nháp → Đơn thuốc | CHỈ chuyển khi status = usmd_approved |
| FLOW-C-08 | Chuyển nháp → Xét nghiệm | CHỈ chuyển khi status = usmd_approved |
| FLOW-D-09 | Thiết lập lịch theo dõi | Lịch PHẢI thuộc: 24h/48h/72h |
| FLOW-D-10 | Check-in 1 | KH PHẢI trả lời ít nhất câu hỏi bắt buộc |
| FLOW-D-11 | Check-in 2/3 | AI PHẢI so sánh với baseline |
| FLOW-D-12 | Đánh giá tổng kết | BS PHẢI review trong vòng 6h sau check-in cuối |
| FLOW-D-13 | Hoàn tất | Audit log PHẢI được tạo |

## Thời gian phản hồi theo gói dịch vụ

| Gói dịch vụ | Giá | Thời gian phản hồi | Trọng số cam kết DV |
|-------------|-----|--------------------|--------------------|
| Care Premium | $249/tháng | Trong vòng 1 giờ | Cao nhất |
| Care Plus | $99/tháng | Trong vòng 2 giờ | Trung bình |
| Care Connect | $39/tháng | Trong vòng 4 giờ | Tiêu chuẩn |

## Bảng tổng hợp data fields

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| case_id | String | Auto | CVH-YYYYMMDD-XXXXX | Mã case duy nhất |
| patient_id | String | Auto | From session | Mã bệnh nhân |
| chat_mode | String | Yes | "text" \| "voice" | Phương thức chat |
| chat_session_id | String | Auto | UUID | ID phiên chat |
| intake_data | Object | Yes | {symptoms, timestamp} | Dữ liệu triệu chứng ban đầu |
| clinical_data | Object | Yes | {chief_complaint, location, pain_level, duration, aggravating_factors, alleviating_factors, associated_symptoms, self_treatment} | Dữ liệu lâm sàng đầy đủ |
| pain_level | Number | Yes | 1-10 | Mức độ đau |
| differential_diagnosis | Array[Object] | Auto | [{diagnosis, probability}] | Chẩn đoán phân biệt |
| severity | String | Auto | "Emergency" \| "Urgent" \| "Routine" | Mức độ nghiêm trọng |
| ai_confidence | Number | Auto | 0-100 (%) | Độ tin cậy AI |
| priority_score | Number | Auto | 1-10 | Điểm ưu tiên |
| service_package | String | Yes | "Premium" \| "Plus" \| "Connect" | Gói dịch vụ |
| sla_deadline | DateTime | Auto | ISO 8601 | Hạn SLA |
| assigned_vn_md | String | Auto | doctor_id | VN MD được gán |
| assigned_us_md | String | Auto | doctor_id | US MD được gán |
| order_recommendations | Array[Object] | Auto | [{type, name, dosage, frequency, duration, reason}] | Đề xuất chỉ định từ AI |
| draft_orders | Array[Object] | Yes | [{order_id, type, details, status, vn_md_decision, reason}] | Chỉ định nháp |
| draft_order_status | String | Yes | "pending_usmd_review" \| "usmd_approved" \| "modification_requested" | Trạng thái draft |
| vn_md_decision | String | Yes | "approve" \| "modify" \| "reject" \| "add_new" | Quyết định VN MD |
| us_md_decision | String | Yes | "approve_all" \| "request_modification" | Quyết định US MD |
| e_signature | Object | Yes | {user_id, timestamp, ip, device} | Chữ ký điện tử US MD |
| actual_rx_orders | Array[Object] | Auto | Saved to ac247_actual_orders | Đơn thuốc chính thức |
| prescription_document | Object | Auto | PDF | Tài liệu đơn thuốc |
| actual_lab_orders | Array[Object] | Auto | Saved to actual_lab_orders | Chỉ định XN chính thức |
| lab_requisition_document | Object | Auto | PDF | Giấy chỉ định XN |
| monitor_order | Object | Auto | {protocol, parameters, tracking_id} | Lệnh theo dõi |
| tracking_id | String | Auto | Unique | Mã theo dõi |
| monitoring_protocol | String | Yes | "24h" \| "48h" \| "72h" | Protocol theo dõi |
| check_in_schedule | Array[DateTime] | Auto | ISO 8601 | Lịch check-in |
| status_update | Object | Yes | {symptom_status, pain_level, new_symptoms, medication_compliance, side_effects} | Dữ liệu check-in |
| symptom_status | String | Yes | "improving" \| "stable" \| "worsening" | Tình trạng triệu chứng |
| red_flag | Boolean | Auto | true/false | Cờ đỏ bất thường |
| trend_analysis | Object | Auto | {direction, data_points} | Phân tích xu hướng |
| final_conclusion | String | Yes | "recovered" \| "need_intervention" \| "transfer_service" | Kết luận cuối |
| case_status | String | Auto | "Active" \| "Paused" \| "Emergency_Closed" \| "Hoàn_tất" | Trạng thái case |
| emr_timeline | Object | Auto | Full timeline | Timeline lưu EMR |
| audit_log | Object | Auto | {user_id, timestamp, action, ip} | Log kiểm toán |

---

# NGUỒN TÀI LIỆU GỐC

| Tài liệu | Đường dẫn |
|-----------|----------|
| Luồng nghiệp vụ (v1.2.0) | docs/02-product/ba-specs/service-1-Access_to_Care_247/base/Access_To_Care_247_Touchpoint_Flow_v1.md |
| Checklist CS & MD | docs/02-product/ba-specs/service-1-Access_to_Care_247/checklist-cs-md/checklist-cs-md.md |
| Bản tóm tắt | docs/02-product/tomtat-chitiet/tomtat/service-1-Access_to_Care_247-tomtat.md |
