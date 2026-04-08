# CHI TIẾT LUỒNG NGHIỆP VỤ – SERVICE 5 (Asynchronous_Messaging): NHẮN TIN BẤT ĐỒNG BỘ (ASYNCHRONOUS MESSAGING)

## Header

- **Service:** Asynchronous Messaging (Nhắn tin bất đồng bộ)
- **Version:** 1.0.0
- **Input:** Khách hàng có tài khoản CVH active muốn đặt câu hỏi
- **Output:** Câu trả lời AI (y tế có disclaimer / phi y tế không disclaimer)
- **Thời gian phản hồi AI:** < 10 giây
- **Compliance:** HIPAA (Medical flow) + Standard (Non-Medical flow)

---

## Nguyên tắc cốt lõi

- **AI-first Model:** AI trả lời trước, chỉ escalate khi cần
- **HIPAA Conditional:** Chỉ áp dụng HIPAA cho câu hỏi y tế (AES-256 at rest, TLS 1.3 in transit, audit trail 7 năm)
- **Loop Support:** KH có thể hỏi nhiều câu liên tiếp trong 1 session
- AI KHÔNG kê đơn, KHÔNG chẩn đoán, KHÔNG thay thế bác sĩ
- Phát hiện Red Flags → alert cấp cứu ngay

---

## Danh sách Flow

| Flow | Tên | Mô tả |
|------|-----|-------|
| FLOW-A | Câu hỏi Y tế (Medical) | HIPAA + Disclaimer + Red Flag detection |
| FLOW-B | Câu hỏi Phi Y tế (Non-Medical) | Không HIPAA, không disclaimer |

### So sánh FLOW-A vs FLOW-B

| Tiêu chí | FLOW-A (Medical) | FLOW-B (Non-Medical) |
|----------|-------------------|----------------------|
| HIPAA | ✅ Bắt buộc | ❌ Không |
| Disclaimer | ✅ "Thông tin tham khảo, không thay thế bác sĩ" | ❌ Không |
| Red Flags | ✅ Phát hiện cấp cứu | ❌ Không |
| Escalation | Service 1 (Access_to_Care_247) (cấp cứu) / Service 3 (Schedule_Video_Visit) (video visit) | CS support |
| Lưu trữ | AES-256, audit 7 năm | TLS, 90 ngày |
| AI Filters | Medical safety filters | General content filters |
| Knowledge Base | CVH Medical Library | CVH Help Center / General knowledge |
| Audit Trail | ✅ Bắt buộc | ❌ Tùy chọn |

---

## PHẦN CHUNG (Bước 1-2)

### Bước 1. Khách hàng – Truy cập dịch vụ Messaging từ Dashboard

- **INPUT:** account_id (String, required), session_token (String, required)
- **OUTPUT:** service_interface (Object), session_id (String)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| account_id | Tài khoản PHẢI active | "Tài khoản không hợp lệ hoặc đã bị khóa" |
| session_token | Token PHẢI valid | "Phiên đăng nhập hết hạn, vui lòng đăng nhập lại" |

#### UI States

- **Default:** Dashboard hiển thị danh sách dịch vụ, card Async Messaging có nút [BẮT ĐẦU]
- **Loading:** Spinner khi tải giao diện dịch vụ
- **Error:** "Hệ thống đang gặp sự cố. Vui lòng thử lại sau 5 phút."

#### Wireframe ASCII

```
┌─────────────────────────────────────┐
│         CVH HEALTHCARE APP          │
├─────────────────────────────────────┤
│                                     │
│  📱 Dashboard                       │
│                                     │
│  ┌───────────────────────────────┐  │
│  │ 💬 Async Messaging            │  │
│  │    Hỏi đáp AI — Y tế &       │  │
│  │    Thông tin chung            │  │
│  │    [BẮT ĐẦU]                 │  │
│  └───────────────────────────────┘  │
│                                     │
│  ┌───────────────────────────────┐  │
│  │ 📞 24/7 Access to Care        │  │
│  └───────────────────────────────┘  │
│                                     │
│  ┌───────────────────────────────┐  │
│  │ 📹 Video Visit                │  │
│  └───────────────────────────────┘  │
│                                     │
└─────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Tài khoản active, token valid | Hiển thị giao diện dịch vụ |
| Tài khoản inactive/không tồn tại | Từ chối truy cập, yêu cầu đăng nhập |
| Token hết hạn | Redirect về màn hình đăng nhập |

#### ⚠️ Lỗi

- Hệ thống sập > 10 phút → CS nhận cảnh báo, liên hệ IT team xử lý khẩn
- Tải > 3 giây → Auto retry 3 lần (mỗi lần cách 2s)
- Retry failed 3 lần → Auto gửi cảnh báo kỹ thuật + hiển thị: "Hệ thống đang gặp sự cố. Vui lòng thử lại sau 5 phút."

---

### Bước 2. Khách hàng – Chọn loại câu hỏi: "Y tế" hoặc "Phi Y tế"

- **INPUT:** session_id (String, required)
- **OUTPUT:** question_type (String: MEDICAL | NON-MEDICAL, required)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| question_type | PHẢI chọn 1 trong 2 loại | "Vui lòng chọn loại câu hỏi" |

#### UI States

- **Default:** 2 card hiển thị rõ ràng: Medical (có badge 🔒 HIPAA) và Non-Medical
- **Hover:** Card được highlight
- **Selected:** Card chọn có border xanh, auto chuyển màn hình tiếp theo

#### Wireframe ASCII

```
┌─────────────────────────────────────┐
│      💬 ASYNC MESSAGING             │
├─────────────────────────────────────┤
│                                     │
│  Bạn muốn hỏi về vấn đề gì?       │
│                                     │
│  ┌───────────────────────────────┐  │
│  │ 🏥 CÂU HỎI Y TẾ (Medical)   │  │
│  │                               │  │
│  │ Triệu chứng, thuốc, điều trị,│  │
│  │ sức khỏe cá nhân              │  │
│  │                               │  │
│  │ 🔒 Bảo mật HIPAA              │  │
│  └───────────────────────────────┘  │
│                                     │
│  ┌───────────────────────────────┐  │
│  │ 📋 CÂU HỎI CHUNG             │  │
│  │    (Non-Medical)              │  │
│  │                               │  │
│  │ Hướng dẫn app, dịch vụ CVH,  │  │
│  │ dinh dưỡng, kỹ thuật         │  │
│  └───────────────────────────────┘  │
│                                     │
└─────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Chọn Y tế (Medical) | → FLOW-A (bật HIPAA mode) |
| Chọn Phi Y tế (Non-Medical) | → FLOW-B (General mode) |

#### ⚠️ Lỗi

- Button lỗi → Auto retry lại giao diện
- Định tuyến sai luồng → Auto rollback về màn hình chọn + log lỗi
- Lỗi định tuyến hàng loạt → CS nhận cảnh báo, báo IT sửa gấp

---

## FLOW-A: CÂU HỎI Y TẾ (MEDICAL)

### Sơ đồ luồng

```
┌─────────────────────────────────────────────────────────────────┐
│              FLOW-A: MEDICAL QUESTION (8 TRẠM)                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TRẠM 1: KH truy cập Async Messaging (App/Web)                 │
│           ↓                                                     │
│  TRẠM 2: KH chọn loại câu hỏi = MEDICAL                       │
│           ↓                                                     │
│  TRẠM 3: KH nhập câu hỏi về sức khỏe  ◄──────────────┐        │
│           ↓                                            │        │
│  TRẠM 4: AI phân tích + HIPAA filters + safety check   │        │
│           ↓                                            │        │
│  TRẠM 5: AI trả lời + disclaimer + references          │        │
│           ↓                                            │        │
│  DECISION: Hỏi thêm?                                  │        │
│           ├── CÓ → ─────────────────────────────────────┘        │
│           └── KHÔNG ↓                                           │
│  TRẠM 6: Hoàn tất session                                      │
│           ↓                                                     │
│  TRẠM 7: Đánh giá (optional)                                   │
│           ↓                                                     │
│  TRẠM 8: Kết thúc — đóng session, lưu lịch sử (encrypted)     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Bước 3. Khách hàng – Nhập câu hỏi y tế (10-2000 ký tự)

- Auto-draft mỗi 10 giây
- **INPUT:** session_id (String, required), question_type (String: MEDICAL, required)
- **OUTPUT:** question_text (String, required — PHI data), timestamp (DateTime)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| question_text | BẮT BUỘC — không được để trống | "Vui lòng nhập câu hỏi của bạn" |
| question_text | Tối thiểu 10 ký tự | "Câu hỏi quá ngắn, vui lòng mô tả chi tiết hơn" |
| question_text | Tối đa 2.000 ký tự | "Câu hỏi quá dài, vui lòng rút gọn" |
| question_text | Không chứa nội dung vi phạm | "Nội dung không phù hợp, vui lòng hỏi câu khác" |

#### UI States

- **Default:** Ô nhập trống, placeholder "Mô tả triệu chứng, thời gian xuất hiện, mức độ...", nút Gửi disabled
- **Typing:** Đếm ký tự realtime, nút Gửi enabled khi ≥ 10 ký tự
- **Error:** Border đỏ + message lỗi bên dưới
- **Idle > 60s:** Auto popup: "Cần mẫu câu hỏi gợi ý?" + hiển thị 3 mẫu phổ biến
- **Mất kết nối:** Auto lưu vào bộ nhớ cục bộ + thông báo: "Nội dung đã được lưu tạm. Kết nối lại sẽ tiếp tục."

#### Wireframe ASCII

```
┌─────────────────────────────────────┐
│  💬 Hỏi về sức khỏe của bạn        │
├─────────────────────────────────────┤
│                                     │
│  [Nhập câu hỏi của bạn ở đây...]   │
│                                     │
│                                     │
│  📝 Ví dụ:                         │
│  • "Thuốc X có tác dụng phụ gì?"   │
│  • "Tôi bị đau đầu 3 ngày, có     │
│     nghiêm trọng không?"           │
│  • "Trẻ em có thể dùng thuốc Y    │
│     không?"                        │
│                                     │
│  [    GỬI CÂU HỎI    ]            │
│                                     │
│  🔒 Câu hỏi của bạn được mã hóa   │
│     và bảo mật theo tiêu chuẩn     │
│     HIPAA                          │
└─────────────────────────────────────┘
```

#### ⚠️ Lỗi

- Mất data → Auto-draft lưu tạm mỗi 10 giây
- Nút gửi lỗi → Auto retry 3 lần
- Mất dữ liệu hàng loạt do lỗi database → CS nhận cảnh báo, thông báo IT xử lý khẩn

---

### Bước 4. Hệ thống (AI) – Phân tích câu hỏi với HIPAA

- Mã hóa PHI (AES-256)
- Ghi audit trail
- Safety filters + Red Flag detection
- **INPUT:** question_text (String, required — PHI), question_type (String: MEDICAL), patient_context (Object, optional)
- **OUTPUT:** analysis_result (Object), safety_check_result (Object), is_emergency (Boolean)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| question_text | PHẢI qua HIPAA filter | N/A (internal) |
| question_text | PHẢI qua medical safety check | N/A (internal) |

#### UI States

- **Loading:** Spinner + "Đang phân tích..." + thông báo HIPAA bảo mật
- **Timeout > 15s:** "Đang xử lý... Vui lòng chờ thêm vài giây." + auto retry
- **Emergency detected:** Chuyển ngay sang màn hình cảnh báo khẩn cấp

#### Wireframe ASCII

```
┌─────────────────────────────────────┐
│  🤖 CVH AI Assistant                │
├─────────────────────────────────────┤
│                                     │
│        ⏳ Đang phân tích...          │
│                                     │
│  🔒 Câu hỏi của bạn đang được xử   │
│     lý với tiêu chuẩn bảo mật      │
│     HIPAA                          │
│                                     │
└─────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Phân tích bình thường | → Bước 5 (AI trả lời) |
| Red Flags detected | → Alert cấp cứu ngay (màn hình cảnh báo khẩn cấp) |
| Câu hỏi quá phức tạp | → Gắn cờ vàng ⚠️ + đưa vào hàng đợi MD review |
| HIPAA compliance lỗi | → Auto dừng xử lý + cảnh báo hệ thống bảo mật |

#### ⚠️ Lỗi

- AI sập > 10 phút → CS báo kỹ thuật
- Red flags (đau ngực, khó thở nặng, đột quỵ, ý định tự tử, chảy máu nặng, mất ý thức) → **MD can thiệp**
- HIPAA violation → CS nhận cảnh báo nghiêm trọng, báo IT/Security team xử lý khẩn

### Bảng Red Flags

| Triệu chứng | Mức độ | Hành động |
|-------------|--------|-----------|
| Đau ngực (Chest pain) | 🔴 Khẩn cấp | Alert + Hướng dẫn gọi 911 / Service 1 (Access_to_Care_247) |
| Khó thở nặng (Severe SOB) | 🔴 Khẩn cấp | Alert + Hướng dẫn gọi 911 / Service 1 (Access_to_Care_247) |
| Triệu chứng đột quỵ (FAST) | 🔴 Khẩn cấp | Alert + Hướng dẫn gọi 911 |
| Ý định tự tử (Suicidal ideation) | 🔴 Khẩn cấp | Alert + Hotline tâm lý + Service 1 (Access_to_Care_247) |
| Chảy máu nặng (Severe bleeding) | 🔴 Khẩn cấp | Alert + Hướng dẫn gọi 911 |
| Mất ý thức (Loss of consciousness) | 🔴 Khẩn cấp | Alert + Hướng dẫn gọi 911 |

### Wireframe cảnh báo khẩn cấp (Trường hợp đặc biệt)

```
┌─────────────────────────────────────┐
│  🚨 CẢNH BÁO KHẨN CẤP             │
├─────────────────────────────────────┤
│                                     │
│  Triệu chứng của bạn có thể cần   │
│  chăm sóc y tế ngay lập tức.      │
│                                     │
│  Vui lòng:                         │
│  🚑 Gọi 911 NGAY nếu có dấu hiệu:│
│     đau ngực, khó thở nặng, mất   │
│     ý thức                         │
│                                     │
│  📞 Sử dụng Service 1 (Access_to_Care_247) (24/7       │
│     Access to Care) để tư vấn ngay │
│                                     │
│  🏥 Đến phòng cấp cứu gần nhất    │
│                                     │
│  ❌ Chúng tôi KHÔNG THỂ xử lý các  │
│  tình huống khẩn cấp qua Async    │
│  Messaging.                        │
│                                     │
│  [Gọi 911]                        │
│  [Service 1 (Access_to_Care_247) — 24/7]               │
│  [Tìm ER gần nhất]                │
└─────────────────────────────────────┘
```

---

### Bước 5. Hệ thống (AI) – Hiển thị câu trả lời + Disclaimer bắt buộc

- Kèm references y khoa
- Nút escalation: "Đặt lịch Video Visit" (→ Service 3 (Schedule_Video_Visit)) / "Liên hệ cấp cứu" (→ Service 1 (Access_to_Care_247))
- **INPUT:** analysis_result (Object, required), safety_check_result (Object, required)
- **OUTPUT:** answer_text (String, required), disclaimer (String, required), references (Array[Object], required), warning_signs (Array[String], required)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| answer_text | PHẢI có nội dung trả lời | N/A (internal) |
| disclaimer | PHẢI có disclaimer y tế trong MỌI câu trả lời | N/A (internal) |
| references | PHẢI có references từ nguồn y tế uy tín | N/A (internal) |

#### UI States

- **Loading > 30s:** "Đang tạo câu trả lời... Vui lòng chờ."
- **Default:** Hiển thị câu trả lời + disclaimer + references + warning signs + nút hỏi thêm/kết thúc
- **Escalation suggested:** Hiển thị thêm nút Service 1 (Access_to_Care_247) / Service 3 (Schedule_Video_Visit)

#### Wireframe ASCII

```
┌─────────────────────────────────────┐
│  🤖 Trả lời từ CVH AI Assistant    │
├─────────────────────────────────────┤
│                                     │
│  [Câu trả lời chi tiết...]         │
│                                     │
│  📚 Tài liệu tham khảo:           │
│  • [Reference 1 — CVH Medical      │
│     Library]                       │
│  • [Reference 2 — Trusted source]  │
│                                     │
│  ⚠️ Lưu ý quan trọng:              │
│  Thông tin này chỉ mang tính tham  │
│  khảo và không thay thế tư vấn y   │
│  tế chuyên nghiệp. Vui lòng tham  │
│  khảo ý kiến bác sĩ để được tư    │
│  vấn cụ thể cho tình trạng sức    │
│  khỏe của bạn.                     │
│                                     │
│  💡 Khi nào cần gặp bác sĩ:        │
│  [Danh sách warning signs]         │
│                                     │
├─────────────────────────────────────┤
│  Bạn còn câu hỏi nào khác về sức  │
│  khỏe không?                       │
│                                     │
│  [Có, tôi muốn hỏi thêm]          │
│  [Không, cảm ơn]                   │
└─────────────────────────────────────┘
```

### Wireframe đề nghị escalation (Trường hợp đặc biệt)

```
┌─────────────────────────────────────┐
│  💬 Để được hỗ trợ tốt hơn         │
├─────────────────────────────────────┤
│                                     │
│  ✅ Service 1 (Access_to_Care_247) — 24/7 Access to Care │
│     • Tư vấn với bác sĩ ngay       │
│     • Có thể kê đơn thuốc          │
│     [Bắt đầu]                      │
│                                     │
│  ✅ Service 3 (Schedule_Video_Visit) — Video Visit          │
│     • Đặt lịch khám qua video      │
│     • Tư vấn chi tiết với bác sĩ   │
│     [Đặt lịch]                     │
│                                     │
│  [Tiếp tục hỏi AI]                 │
│                                     │
└─────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Câu hỏi phức tạp | Escalate → MD review + hiển thị nút Service 1 (Access_to_Care_247) / Service 3 (Schedule_Video_Visit) |
| Yêu cầu kê đơn thuốc | AI từ chối + gợi ý Service 1 (Access_to_Care_247) / Service 3 (Schedule_Video_Visit) |
| Yêu cầu xét nghiệm | AI từ chối + gợi ý Service 3 (Schedule_Video_Visit) |
| Cần khám trực tiếp | Thông báo + đặt lịch Video Visit (Service 3 (Schedule_Video_Visit)) |

#### ⚠️ Lỗi

- Câu trả lời bị phản hồi tiêu cực → **MD review chất lượng**
- AI liên tục vượt ngưỡng 30s → CS nhận cảnh báo nghiêm trọng, chuyển cho MD review

---

### Bước 6. Khách hàng – Quyết định hỏi thêm hoặc kết thúc

- **INPUT:** answer_displayed (Boolean, required)
- **OUTPUT:** user_choice (String: YES | NO, required)

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Hỏi thêm câu (YES) | → Quay lại bước 3 (loop, giữ nguyên MEDICAL) |
| Kết thúc (NO) | → Bước 7 (Hoàn tất) |

---

### Bước 7. Khách hàng – Feedback (tùy chọn, có thể skip)

- **INPUT:** session_summary (Object, required)
- **OUTPUT:** is_helpful (Boolean, optional), star_rating (Number: 1-5, optional), comment (String, optional — max 500 ký tự)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| star_rating | 1-5 sao | "Vui lòng chọn từ 1 đến 5 sao" |
| comment | Tối đa 500 ký tự | "Góp ý quá dài, vui lòng rút gọn" |

#### UI States

- **Default:** Form feedback hiển thị: Helpful/Not Helpful, stars, ô nhận xét, nút Gửi + nút Bỏ qua
- **Submitted:** "Cảm ơn bạn đã đánh giá!"
- **Skipped:** Tự động chuyển bước 8

#### Wireframe ASCII

```
┌─────────────────────────────────────┐
│  📊 Đánh giá trải nghiệm           │
├─────────────────────────────────────┤
│                                     │
│  Câu trả lời có hữu ích không?    │
│  [👍 Có]   [👎 Không]              │
│                                     │
│  Đánh giá chung:                   │
│  ⭐⭐⭐⭐⭐                            │
│                                     │
│  Góp ý (optional):                 │
│  [................................] │
│                                     │
│  [Gửi đánh giá]  [Bỏ qua]        │
│                                     │
└─────────────────────────────────────┘
```

#### ⚠️ Lỗi

- Rating < 3 sao → CS review + MD review (nếu câu hỏi y tế)
- Lỗi gửi feedback → Auto retry 3 lần, retry failed → log lỗi và đóng form

---

### Bước 8. Hệ thống – Kết thúc session

- Lưu lịch sử (HIPAA encrypted), cleanup temp data
- **INPUT:** feedback_data (Object, optional), conversation_history (Array[Object], required)
- **OUTPUT:** session_closed (Boolean), history_saved (Boolean)

#### UI States

- **Default:** Hiển thị thông báo cảm ơn + link xem lịch sử + nút về trang chủ
- **Error:** "Đang lưu dữ liệu..." + auto retry

#### Wireframe ASCII

```
┌─────────────────────────────────────┐
│  ✅ Cảm ơn bạn!                     │
├─────────────────────────────────────┤
│                                     │
│  Session của bạn đã được lưu.      │
│  Bạn có thể xem lại lịch sử hội   │
│  thoại bất kỳ lúc nào.            │
│                                     │
│  [Xem lịch sử]                    │
│  [Về trang chủ]                    │
│                                     │
└─────────────────────────────────────┘
```

#### ⚠️ Lỗi

- Lưu lịch sử lỗi → Auto retry 3 lần, retry failed → hàng đợi lưu lại sau
- Mã hóa HIPAA lỗi → Auto dừng + cảnh báo đội ngũ kỹ thuật ngay lập tức
- Nhiều session không lưu được → CS nhận alert, báo IT/Security team xử lý khẩn

---

## FLOW-B: CÂU HỎI PHI Y TẾ (NON-MEDICAL)

### Danh mục câu hỏi phi y tế

| Danh mục | Ví dụ câu hỏi |
|----------|----------------|
| Hướng dẫn sử dụng App | "Làm sao đổi mật khẩu?", "Cách đặt lịch video visit?", "Xem lịch sử khám ở đâu?" |
| Thông tin dịch vụ | "Care Plus package có gì?", "Service 1 (Access_to_Care_247) hoạt động như thế nào?", "Quota là gì?" |
| Quản lý tài khoản | "Cách cập nhật thông tin cá nhân?", "Thêm người phụ thuộc?", "Hủy subscription?" |
| Thanh toán / Billing | "Cách xem hóa đơn?", "Phương thức thanh toán?", "Hoàn tiền như thế nào?" |
| Sức khỏe tổng quát | "Dinh dưỡng cho người tiểu đường?", "Cách giảm stress?", "Vận động bao nhiêu là đủ?" |
| Hỗ trợ kỹ thuật | "App bị lỗi?", "Không vào được video call?", "Không nhận được notification?" |

### Sơ đồ luồng

```
┌─────────────────────────────────────────────────────────────────┐
│            FLOW-B: NON-MEDICAL QUESTION (8 TRẠM)                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TRẠM 1: KH truy cập Async Messaging (App/Web)                 │
│           ↓                                                     │
│  TRẠM 2: KH chọn loại câu hỏi = NON-MEDICAL                   │
│           ↓                                                     │
│  TRẠM 3: KH nhập câu hỏi về dịch vụ/thông tin  ◄─────┐        │
│           ↓                                            │        │
│  TRẠM 4: AI phân tích (general content filters)        │        │
│           ↓                                            │        │
│  TRẠM 5: AI trả lời + references                       │        │
│           ↓                                            │        │
│  DECISION: Hỏi thêm?                                  │        │
│           ├── CÓ → ──────────────────────────────────────┘        │
│           └── KHÔNG ↓                                           │
│  TRẠM 6: Hoàn tất session                                      │
│           ↓                                                     │
│  TRẠM 7: Đánh giá (optional)                                   │
│           ↓                                                     │
│  TRẠM 8: Kết thúc — đóng session, lưu lịch sử                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Bước 3. Khách hàng – Nhập câu hỏi phi y tế (5-2000 ký tự)

- Auto-draft mỗi 10 giây
- **INPUT:** session_id (String, required), question_type (String: NON-MEDICAL, required)
- **OUTPUT:** question_text (String, required), timestamp (DateTime)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| question_text | BẮT BUỘC — không được để trống | "Vui lòng nhập câu hỏi của bạn" |
| question_text | Tối thiểu 5 ký tự | "Câu hỏi quá ngắn, vui lòng mô tả chi tiết hơn" |
| question_text | Tối đa 2.000 ký tự | "Câu hỏi quá dài, vui lòng rút gọn" |
| question_text | Không chứa spam / nội dung không phù hợp | "Nội dung không phù hợp, vui lòng hỏi câu khác" |

#### UI States

- **Default:** Ô nhập trống, placeholder "Nhập câu hỏi về dịch vụ, thanh toán, tài khoản...", nút Gửi disabled
- **Typing:** Đếm ký tự realtime, nút Gửi enabled khi ≥ 5 ký tự
- **Error:** Border đỏ + message lỗi bên dưới
- **Idle > 60s:** Auto popup gợi ý: "Bạn muốn hỏi về: Hóa đơn / Đặt lịch / Thông tin gói dịch vụ?"
- **Mất kết nối:** Auto lưu bộ nhớ cục bộ + thông báo: "Nội dung đã được lưu tạm."

#### Wireframe ASCII

```
┌─────────────────────────────────────┐
│  💬 Hỏi về dịch vụ CVH             │
├─────────────────────────────────────┤
│                                     │
│  [Nhập câu hỏi của bạn ở đây...]   │
│                                     │
│                                     │
│  📝 Ví dụ:                         │
│  • "Làm sao đổi mật khẩu?"        │
│  • "Care Plus có những gì?"        │
│  • "Cách đặt lịch video visit?"    │
│                                     │
│  [    GỬI CÂU HỎI    ]            │
│                                     │
└─────────────────────────────────────┘
```

#### ⚠️ Lỗi

- Mất data → Auto-draft lưu tạm mỗi 10 giây
- Nút gửi lỗi → Auto retry 3 lần
- Mất dữ liệu hàng loạt → CS nhận cảnh báo, báo IT xử lý

---

### Bước 4. Hệ thống (AI) – Phân tích (không HIPAA)

- Content filters, spam detection, violation logging
- Fallback sang rule-based filter nếu AI lỗi
- **INPUT:** question_text (String, required), question_type (String: NON-MEDICAL)
- **OUTPUT:** analysis_result (Object), content_filter_result (Object)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| question_text | PHẢI qua general content filter | N/A (internal) |

#### UI States

- **Loading:** Spinner + "Đang xử lý..."
- **Timeout > 10s:** Auto retry + "Đang xử lý..."

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Phân tích bình thường | → Bước 5 (AI trả lời) |
| Phát hiện câu hỏi y tế | Thông báo đây là câu hỏi y tế → đề nghị chuyển FLOW-A |
| Vi phạm nội dung | Từ chối + hiển thị lý do + log vào database |
| Spam/lạm dụng | Auto gắn cờ tài khoản + alert CS |

#### ⚠️ Lỗi

- Phát hiện lạm dụng → CS review
- AI sập > 10 phút → CS nhận alert, báo IT xử lý
- Content filter lỗi → Tự động fallback về bộ lọc rule-based

---

### Bước 5. Hệ thống (AI) – Hiển thị câu trả lời (không có disclaimer y tế)

- Kèm references, xử lý out-of-scope
- **INPUT:** analysis_result (Object, required)
- **OUTPUT:** answer_text (String, required), references (Array[Object], optional), tips (Array[String], optional)

#### Validation Rules

| Field | Rule | Message lỗi |
|-------|------|-------------|
| answer_text | PHẢI có nội dung trả lời | N/A (internal) |
| answer_text | KHÔNG có disclaimer y tế | N/A (internal) |

#### UI States

- **Loading > 20s:** "Đang tạo câu trả lời..."
- **Default:** Hiển thị câu trả lời + references + tips + nút hỏi thêm/kết thúc
- **Out-of-scope:** "Vui lòng liên hệ hotline 1900xxxx để được hỗ trợ thêm."

#### Wireframe ASCII

```
┌─────────────────────────────────────┐
│  🤖 Trả lời từ CVH Assistant       │
├─────────────────────────────────────┤
│                                     │
│  [Câu trả lời chi tiết với         │
│   step-by-step instructions        │
│   nếu cần...]                      │
│                                     │
│  📚 Tài liệu tham khảo:           │
│  • [Link to CVH Help Center]       │
│  • [Link to relevant guide]        │
│                                     │
│  💡 Tips:                           │
│  [Additional helpful tips]          │
│                                     │
├─────────────────────────────────────┤
│  Bạn còn câu hỏi nào khác không?  │
│                                     │
│  [Có, tôi muốn hỏi thêm]          │
│  [Không, cảm ơn]                   │
└─────────────────────────────────────┘
```

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Vấn đề kỹ thuật phức tạp | Tạo ticket cho Support Team |
| Vấn đề tài khoản cụ thể | Kết nối Customer Service |
| Tranh chấp thanh toán | Chuyển đến Billing Team |
| Báo lỗi ứng dụng | Ghi nhận + tạo bug ticket |
| Câu hỏi ngoài phạm vi | Thông báo + liên hệ support |

#### ⚠️ Lỗi

- AI liên tục vượt ngưỡng 20s → CS nhận alert
- Câu trả lời sai lệch nghiêm trọng → CS ghi nhận

---

### Bước 6. Khách hàng – Hỏi thêm (loop) hoặc kết thúc

- **INPUT:** answer_displayed (Boolean, required)
- **OUTPUT:** user_choice (String: YES | NO, required)

#### ⛔ Rẽ nhánh

| Điều kiện | Hành động |
|-----------|-----------|
| Hỏi thêm câu (YES) | → Quay lại bước 3 (loop, giữ nguyên NON-MEDICAL) |
| Kết thúc (NO) | → Bước 7 (Hoàn tất) |

---

### Bước 7. Khách hàng – Feedback (tùy chọn)

- Giống FLOW-A bước 7 (xem ở trên)
- ⚠️ Rating < 3 sao → CS review (không cần MD review vì phi y tế)

---

### Bước 8. Hệ thống – Kết thúc session: lưu lịch sử (standard, 90 ngày)

- **INPUT:** feedback_data (Object, optional), conversation_history (Array[Object], required)
- **OUTPUT:** session_closed (Boolean), history_saved (Boolean)
- Lưu trữ bảo mật tiêu chuẩn (TLS), không cần HIPAA
- Thời gian lưu: 90 ngày

#### ⚠️ Lỗi

- Lưu lịch sử lỗi → Auto retry 3 lần
- Nhiều session không lưu được → CS nhận alert, báo IT xử lý

---

## XỬ LÝ TRƯỜNG HỢP ĐẶC BIỆT

### FLOW-A

| Trường hợp | Xử lý AI | Hướng dẫn KH |
|------------|-----------|--------------|
| Emergency / Red Flags | Hiển thị cảnh báo khẩn cấp, từ chối xử lý qua Messaging | Gọi 911 hoặc dùng Service 1 (Access_to_Care_247) (24/7 Access) ngay |
| Câu hỏi phức tạp | Thông báo cần đánh giá chi tiết hơn | Đề nghị dùng Service 1 (Access_to_Care_247) hoặc Service 3 (Schedule_Video_Visit) |
| Cần khám trực tiếp | Thông báo triệu chứng cần khám trực tiếp | Đặt lịch Video Visit (Service 3 (Schedule_Video_Visit)) |
| Yêu cầu kê đơn thuốc | Thông báo AI không thể kê đơn | Service 1 (Access_to_Care_247) hoặc Service 3 (Schedule_Video_Visit) để được kê đơn |
| Cần xét nghiệm | Thông báo có thể cần xét nghiệm | Service 3 (Schedule_Video_Visit) để được chỉ định xét nghiệm |
| Nội dung vi phạm điều khoản | Từ chối lịch sự | Giải thích lý do, đề nghị hỏi câu khác |
| Ngôn ngữ không hỗ trợ | Thông báo chỉ hỗ trợ tiếng Việt và tiếng Anh | Chuyển ngôn ngữ |
| Câu hỏi không rõ ràng | Yêu cầu diễn đạt lại | KH nhập lại câu hỏi |

### FLOW-B

| Trường hợp | Xử lý AI | Hướng dẫn KH |
|------------|-----------|--------------|
| Vấn đề kỹ thuật phức tạp | Thông báo cần hỗ trợ kỹ thuật chuyên sâu | Tạo ticket cho Support Team |
| Vấn đề liên quan tài khoản cụ thể | Thông báo cần kiểm tra tài khoản | Kết nối Customer Service |
| Phát hiện câu hỏi y tế | Thông báo đây là câu hỏi y tế | Chuyển sang Medical flow (FLOW-A) |
| Tranh chấp thanh toán | Thông báo cần xem xét cụ thể | Chuyển đến Billing Team |
| Báo lỗi ứng dụng | Ghi nhận và cảm ơn | Tạo bug ticket |
| Spam / Nội dung không phù hợp | Từ chối xử lý | Thông báo + đề nghị hỏi câu hợp lệ |
| Câu hỏi ngoài phạm vi | Thông báo nằm ngoài phạm vi hỗ trợ | Liên hệ support |

---

## BẢNG THÔNG BÁO TỰ ĐỘNG (MỞ RỘNG CÓ TEMPLATE)

| Sự kiện | Kênh | Người nhận |
|---------|------|------------|
| Emergency alert (Red Flags) | Push + In-app | KH + CS + MD |
| Escalation (câu hỏi phức tạp) | Alert nội bộ | MD |
| Technical escalation | Alert nội bộ | Kỹ thuật |
| Session summary | Email | KH |
| Re-engagement (KH lâu không dùng) | Push | KH |

### Template chi tiết

🚨 **In-App: Cảnh báo khẩn cấp (FLOW-A)**
- Title: "🚨 CẢNH BÁO KHẨN CẤP"
- Nội dung: "Triệu chứng của bạn có thể cần chăm sóc y tế ngay lập tức. Vui lòng gọi 911 hoặc sử dụng Service 1 (Access_to_Care_247) (24/7 Access to Care)."
- Thời điểm: Ngay khi AI phát hiện Red Flag tại Trạm 4
- Người nhận: Khách hàng

💬 **In-App: Đề nghị Escalation (FLOW-A)**
- Title: "💬 Để được hỗ trợ tốt hơn"
- Nội dung: "Câu hỏi này cần đánh giá chi tiết hơn. Bạn có thể sử dụng Service 1 (Access_to_Care_247) (24/7 Access to Care) hoặc Service 3 (Schedule_Video_Visit) (Video Visit) để được tư vấn trực tiếp với bác sĩ."
- Thời điểm: Khi AI xác định câu hỏi quá phức tạp tại Trạm 4/5
- Người nhận: Khách hàng

🔧 **In-App: Escalation kỹ thuật (FLOW-B)**
- Title: "🔧 Yêu cầu hỗ trợ kỹ thuật"
- Nội dung: "Vấn đề này cần hỗ trợ kỹ thuật chuyên sâu. Chúng tôi đã tạo yêu cầu hỗ trợ cho bạn. Đội ngũ support sẽ liên hệ trong vòng 24h."
- Thời điểm: Khi AI xác định cần human support tại Trạm 4/5
- Người nhận: Khách hàng

📧 **Email: Tổng kết session**
- Subject: "[CVH] Tổng kết phiên hỏi đáp của bạn"
- Body: "Xin chào [Tên KH], cảm ơn bạn đã sử dụng dịch vụ Async Messaging. Bạn có thể xem lại lịch sử hội thoại trong ứng dụng CVH. Nếu cần hỗ trợ thêm, hãy liên hệ chúng tôi."
- Thời điểm: Sau khi session kết thúc (Trạm 8)
- Người nhận: KH (email đã đăng ký)

📱 **Push: Nhắc nhở sử dụng dịch vụ**
- Title: "💬 Bạn có câu hỏi cần giải đáp?"
- Nội dung: "Hỏi ngay CVH AI Assistant — hỗ trợ 24/7 về sức khỏe và dịch vụ CVH."
- Thời điểm: Periodic (KH active chưa dùng dịch vụ trong 30 ngày)
- Người nhận: KH có tài khoản active

---

## BẢNG GIÁM SÁT CS-MD ĐẦY ĐỦ

- **Hệ thống auto ~95%:** AI trả lời, HIPAA encryption, audit trail, safety filters, Red Flag detection
- **CS can thiệp:** System sập > 10 phút, HIPAA violation, phản hồi tiêu cực, phát hiện lạm dụng, lỗi delivery hàng loạt
- **MD can thiệp:** Red flags (cấp cứu), câu hỏi phức tạp escalated, review phản hồi y tế tiêu cực, cập nhật Medical Library định kỳ

### 11 trạm giám sát

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Access Service | Tải < 3s (app < 2s, giao diện < 1s) | 🤖 100%: Auto retry 3 lần (mỗi 2s), auto restart phiên nếu treo, ghi log mốc thời gian + session ID | Hệ thống sập > 10 phút (retry thất bại) → liên hệ IT khẩn | ❌ |
| Select Question Type | Ghi nhận đúng loại, chuyển đúng luồng | 🤖 100%: Auto hiển thị mô tả mỗi lựa chọn, auto retry nếu button lỗi, auto rollback nếu định tuyến sai | Lỗi định tuyến hàng loạt (retry thất bại) → báo IT sửa gấp | ❌ |
| Enter Question (Medical) | Ghi nhận đầy đủ nội dung, không mất data | 🤖 100%: Auto-draft mỗi 10s, auto lưu local nếu mất kết nối, auto retry gửi 3 lần, gợi ý mẫu câu hỏi nếu idle > 60s | Mất dữ liệu hàng loạt (database lỗi, retry thất bại) → báo IT khẩn | ❌ |
| AI Analyze (HIPAA) | Phân tích < 15s (AI < 10s, HIPAA < 5s, safety < 5s), phát hiện Red Flags | 🤖 95%: Auto mã hóa PHI + audit trail, auto safety filters, auto cờ đỏ 🚨 (khẩn cấp) + cờ vàng ⚠️ (phức tạp), auto retry nếu > 15s | Lỗi HIPAA nghiêm trọng / AI sập > 10 phút → báo IT/Security khẩn | ✅ Cờ đỏ → MD đánh giá ngay, quyết định 911. Cờ vàng → MD review thủ công |
| AI Answer (+ Disclaimer) | Tạo câu trả lời < 10s, có disclaimer + references bắt buộc | 🤖 95%: Auto tạo câu trả lời + references + disclaimer chuẩn, auto nút escalation Service 1 (Access_to_Care_247) / Service 3 (Schedule_Video_Visit) nếu phức tạp, auto từ chối + hướng dẫn 911 nếu khẩn cấp | AI vượt 30s liên tục / khiếu nại câu trả lời sai gây hại → chuyển MD review | ✅ Review escalated + quality review. Cập nhật CVH Medical Library |
| Enter Question (Non-Medical) | Ghi nhận đầy đủ nội dung, không mất data | 🤖 100%: Auto-draft mỗi 10s, auto lưu local nếu mất kết nối, auto retry gửi 3 lần, gợi ý chủ đề nếu idle > 60s | Mất dữ liệu hàng loạt (retry thất bại) → báo IT | ❌ |
| AI Analyze (General) | Phân tích < 10s, phát hiện spam/vi phạm | 🤖 100%: Auto content filters + spam detection, auto fallback rule-based nếu AI lỗi, auto từ chối vi phạm + log, auto gắn cờ tài khoản nếu nhiều vi phạm | Lạm dụng hàng loạt / AI sập > 10 phút → review log, quyết định khóa tài khoản | ❌ |
| AI Answer (No Disclaimer) | Tạo câu trả lời < 15s, KHÔNG có disclaimer y tế | 🤖 100%: Auto references từ Help Center, auto kiểm tra không thêm disclaimer y tế, auto từ chối lịch sự nếu vi phạm, auto liên hệ hotline nếu out-of-scope | AI vượt 20s liên tục / câu trả lời sai nghiêm trọng → ghi nhận | ❌ |
| Complete | Hiển thị < 2s, tạo summary < 3s | 🤖 100%: Auto chuyển màn hình hoàn tất, auto tạo session summary, auto hiển thị form feedback + nút Skip | Không hiển thị hàng loạt (retry failed) → báo IT | ❌ |
| Feedback | Ghi nhận đầy đủ feedback, cho phép bỏ qua | 🤖 100%: Auto ghi nhận + link session ID, auto retry 3 lần nếu lỗi gửi, auto alert CS nếu < 3 sao | Feedback âm (< 3 sao) về câu trả lời sai → review log, báo MD nếu medical | ✅ Review negative medical feedback |
| End | Đóng session < 2s, lưu lịch sử < 3s, mã hóa (Medical) < 5s | 🤖 100%: Medical → auto mã hóa end-to-end + audit trail HIPAA (7 năm). Non-Medical → standard (90 ngày). Auto cleanup temp data, auto retry 3 lần nếu lỗi | Database lỗi sau retry / HIPAA violation → báo IT/Security khẩn | ✅ Có quyền xem medical history (phân quyền HIPAA) |

---

## DATA SPECIFICATION

### Bảng tổng hợp Data Fields

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| account_id | String | Yes | — | Mã tài khoản KH |
| session_token | String | Yes | JWT / OAuth token | Token xác thực phiên đăng nhập |
| session_id | String | Auto | UUID | Mã phiên hội thoại |
| question_type | String | Yes | MEDICAL \| NON-MEDICAL | Loại câu hỏi KH chọn |
| question_text | String | Yes | Medical: 10-2000 ký tự, Non-Medical: 5-2000 ký tự | Nội dung câu hỏi |
| timestamp | DateTime | Auto | ISO 8601 | Thời gian gửi câu hỏi |
| patient_context | Object | No | — | Ngữ cảnh bệnh nhân (Medical only) |
| analysis_result | Object | Auto | — | Kết quả phân tích AI |
| safety_check_result | Object | Auto | — | Kết quả kiểm tra an toàn y tế (Medical only) |
| is_emergency | Boolean | Auto | true/false | Phát hiện Red Flag hay không |
| content_filter_result | Object | Auto | — | Kết quả lọc nội dung (Non-Medical only) |
| answer_text | String | Auto | — | Nội dung câu trả lời AI |
| disclaimer | String | Auto | Nội dung chuẩn bắt buộc | Disclaimer y tế (Medical only) |
| references | Array[Object] | Auto | {title, url, source} | Tài liệu tham khảo |
| warning_signs | Array[String] | Auto | — | Danh sách dấu hiệu cần gặp bác sĩ (Medical only) |
| tips | Array[String] | Auto | — | Gợi ý bổ sung (Non-Medical only) |
| user_choice | String | Yes | YES \| NO | KH chọn hỏi thêm hay kết thúc |
| session_summary | Object | Auto | {question_count, types, duration} | Tổng kết session |
| is_helpful | Boolean | No | true/false | Đánh giá hữu ích |
| star_rating | Number | No | 1-5 | Đánh giá sao |
| comment | String | No | Max 500 ký tự | Góp ý |
| conversation_history | Array[Object] | Auto | {question, answer, timestamp} | Lịch sử hội thoại |
| session_closed | Boolean | Auto | true/false | Trạng thái đóng session |
| history_saved | Boolean | Auto | true/false | Trạng thái lưu lịch sử |

### HIPAA Data Handling (FLOW-A)

| Yếu tố | Yêu cầu |
|---------|---------|
| Mã hóa lưu trữ | AES-256 at rest |
| Mã hóa truyền tải | TLS 1.3 in transit |
| Kiểm soát truy cập | Role-based access control, MFA cho admin |
| Audit Trail | Ghi nhật ký mọi truy cập, lưu trữ 7 năm |
| Thông báo vi phạm | Thông báo trong vòng 72 giờ khi phát hiện vi phạm |
| Quyền bệnh nhân | Truy cập, xóa, xuất dữ liệu cá nhân |
| BAA | Business Associate Agreement với nhà cung cấp AI |
| Thời gian lưu trữ | 7 năm (Medical conversation history) |

### Standard Data Handling (FLOW-B)

| Yếu tố | Yêu cầu |
|---------|---------|
| Mã hóa truyền tải | TLS (Standard encryption) |
| Lưu trữ | Standard security, không cần HIPAA |
| Thời gian lưu trữ | 90 ngày (Non-Medical conversation history) |
| Quyền KH | Truy cập, xem, xóa lịch sử hội thoại |

---

## NGUỒN TÀI LIỆU GỐC

- Luồng nghiệp vụ: `docs/02-product/ba-specs/service-5-Asynchronous_Messaging/base/Asynchronous_Messaging_Touchpoint_Flow_v1.md`
- Checklist CS & MD: `docs/02-product/ba-specs/service-5-Asynchronous_Messaging/checklist-cs-md/checklist-cs-md.md`
- Bản tóm tắt: `docs/02-product/tomtat-chitiet/tomtat/service-5-Asynchronous_Messaging-tomtat.md`
