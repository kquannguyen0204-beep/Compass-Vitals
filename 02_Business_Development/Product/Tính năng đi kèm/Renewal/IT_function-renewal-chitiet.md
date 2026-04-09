# CHI TIẾT LUỒNG NGHIỆP VỤ – FUNCTION RENEWAL: AUTO-RENEWAL (Tự động gia hạn)

## Header

- **Function:** Auto-Renewal (Tự động gia hạn)
- **Version:** 1.3
- **Input:** Khách hàng đang sử dụng gói dịch vụ (auto-renew bật), gói sắp hết hạn
- **Output:** Gói dịch vụ được gia hạn thành công, khách hàng tiếp tục sử dụng dịch vụ không gián đoạn
- **Target User:** Khách hàng đã mua gói dịch vụ (Paid User)
- **Billing Cycles:** Monthly (hàng tháng) / Yearly (hàng năm)

---

## Điều kiện đầu vào (Pre-conditions)

- Đã thanh toán gói (Connect/Plus/Premium)
- Billing cycle: Monthly hoặc Yearly
- Auto-renew = TRUE (mặc định khi mua)
- Thẻ thanh toán hợp lệ (tokenized, chưa hết hạn, không bị block)
- Package Status = "Active"

## Nguyên tắc cốt lõi

- **Transparency:** Khách hàng luôn được thông báo trước về việc gia hạn
- **Control:** Khách hàng có quyền opt-out bất kỳ lúc nào
- **Reliability:** Retry mechanism tối đa hóa tỷ lệ thanh toán thành công
- **Customer-friendly:** Grace period và reactivation options để giữ chân khách hàng
- Nguyên tắc kiểm soát luồng: Mỗi node có INPUT/OUTPUT rõ ràng, điều kiện cụ thể, luồng đơn tuyến

> ⚠️ **QUAN TRỌNG:** Không có khái niệm "Suspend" hay "Cancel". Khi gói hết hạn, tài khoản vẫn hoạt động, user chỉ không sử dụng được services. User có thể gia hạn lại BẤT CỨ LÚC NÀO.

---

## Danh sách 5 luồng

| Flow | Tên luồng | Input | Output |
|------|-----------|-------|--------|
| FLOW-01 | Gia hạn tự động thành công | Gói sắp hết hạn (T-7), auto-renew = TRUE | Gói được gia hạn thành công, hóa đơn được gửi |
| FLOW-02 | Khách hàng Opt-out | KH muốn tắt tự động gia hạn (từ Settings hoặc Email) | Auto-renew tắt, gói hết hạn theo lịch |
| FLOW-03 | Xử lý lỗi thanh toán | Thanh toán không thành công (bị từ chối hoặc thẻ không hợp lệ) | Retry thành công hoặc gói hết hạn (Package Expired) |
| FLOW-04 | Gia hạn gói đã hết hạn | Gói đã hết hạn (Expired), KH muốn gia hạn lại | Gói được gia hạn, KH tiếp tục sử dụng services |
| FLOW-05 | Primary quản lý gia hạn cho Adult | Primary muốn bật/tắt auto-renewal cho Adult family member | Auto-renew thay đổi, Adult nhận thông báo |

## Phân nhánh trong Flow

| Flow | Phân nhánh | Mô tả |
|------|-----------|-------|
| FLOW-01 | Thanh toán thành công | Validate thẻ OK → Charge OK → Gia hạn +1 cycle → Gửi invoice |
| FLOW-01 | Thanh toán không thành công | Validate FAIL hoặc Charge FAIL → Chuyển FLOW-03 (Xử lý lỗi) |
| FLOW-02 | Tắt từ Settings | Settings → Toggle OFF → Confirm → Tắt auto-renew |
| FLOW-02 | Tắt từ Email T-7 | Click link email → Redirect Settings → Toggle OFF → Confirm |
| FLOW-02 | Đổi gói | KH click "Change Plan" → Redirect sang quy trình đổi gói |
| FLOW-03 | Thanh toán bị từ chối | Retry 1 (T+1) → Retry 2 (T+3) → Retry 3 FINAL (T+7) → Package Expired nếu fail |
| FLOW-03 | Thẻ không hợp lệ | Grace 7 ngày → KH update thẻ → Charge lại → Package Expired nếu không update |
| FLOW-04 | Nhánh A: Dùng thẻ đã lưu | Chọn thẻ cũ → Thanh toán → Gia hạn thành công |
| FLOW-04 | Nhánh B: Thêm thẻ mới | Nhập thẻ mới → Thanh toán → Gia hạn thành công |
| FLOW-04 | Nhánh C: Adult tự gia hạn | Primary từ chối → Adult dùng thẻ riêng → Thanh toán → Gia hạn (vẫn linked) |
| FLOW-05 | Primary tắt cho Adult | Quản lý gia đình → Chọn Adult → Toggle OFF → Thông báo Adult |
| FLOW-05 | Primary bật cho Adult | Quản lý gia đình → Chọn Adult → Toggle ON → Thông báo Adult |

## Sơ đồ luồng

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           FUNCTION RENEWAL: AUTO-RENEWAL                           │
│                                                                             │
│   ┌──────────────┐                                                          │
│   │   T-7 Scan   │                                                          │
│   │  (Daily Job) │                                                          │
│   └──────┬───────┘                                                          │
│          │                                                                  │
│          ▼                                                                  │
│   ┌──────────────┐     opt-out      ┌──────────────┐                       │
│   │   FLOW-01    │ ───────────────► │   FLOW-02    │                       │
│   │ Auto-Renewal │                  │   Opt-out    │                       │
│   └──────┬───────┘                  └──────┬───────┘                       │
│          │                                 │                                │
│          │ payment error                   │ gói hết hạn theo lịch          │
│          │                                 ▼                                │
│          ├─────────────────────────►┌──────────────┐                       │
│          │                          │   FLOW-03    │                       │
│          │                          │ Payment Error│                       │
│          │                          │  Handling    │                       │
│          │                          └──────┬───────┘                       │
│          │                                 │                                │
│          │                                 │ all retries/grace failed       │
│          │                                 ▼                                │
│          │                          ┌──────────────┐                       │
│          │                          │   PACKAGE    │◄────────────────────┐ │
│          │                          │   EXPIRED    │                     │ │
│          │                          └──────┬───────┘                     │ │
│          │                                 │                             │ │
│          │                                 │ user muốn gia hạn lại       │ │
│          │                                 ▼                             │ │
│          │                          ┌──────────────┐                     │ │
│          │                          │   FLOW-04    │                     │ │
│          │                          │  Gia hạn lại │                     │ │
│          │                          │ (bất cứ lúc  │                     │ │
│          │                          │    nào)      │                     │ │
│          │                          └──────┬───────┘                     │ │
│          │                                 │                             │ │
│          │              ┌──────────────────┼──────────────────┐          │ │
│          │              │                  │                  │          │ │
│          │              ▼                  ▼                  ▼          │ │
│          │       ┌────────────┐     ┌────────────┐     ┌────────────┐    │ │
│          │       │ Nhánh A:   │     │ Nhánh B:   │     │ Nhánh C:   │    │ │
│          │       │ Thẻ đã lưu │     │ Thẻ mới    │     │ Adult tự   │    │ │
│          │       │            │     │            │     │ gia hạn    │    │ │
│          │       └──────┬─────┘     └──────┬─────┘     └──────┬─────┘    │ │
│          │              │                  │                  │          │ │
│          │              └──────────────────┼──────────────────┘          │ │
│          │                                 │                             │ │
│          ▼                                 ▼                             │ │
│   ┌──────────────┐                  ┌──────────────┐                     │ │
│   │   SUCCESS    │◄─────────────────│   SUCCESS    │                     │ │
│   │  (Renewed)   │                  │  (Renewed)   │                     │ │
│   └──────────────┘                  └──────────────┘                     │ │
│                                                                          │ │
│   ⚠️ LƯU Ý: Khi Package Expired, tài khoản VẪN HOẠT ĐỘNG                │ │
│   - User có thể vào app, xem TẤT CẢ tài liệu/hồ sơ y tế                 │ │
│   - Chỉ không dùng được services (Video, Chat, AI)                       │ │
│   - Có thể gia hạn lại BẤT CỨ LÚC NÀO (không giới hạn thời gian)        │ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Timeline tổng quan

```
T-7              T=0              T+1         T+3              T+7            T+∞
 │                │                │           │                │               │
Gửi email     Validate &     Retry 1      Retry 2      Retry 3 FINAL    User có thể
thông báo     Payment        (nếu fail)   (nếu fail)   (nếu fail)       gia hạn lại
 gia hạn       attempt                                   → PACKAGE        BẤT CỨ LÚC
                                                          EXPIRED          NÀO
```

---

## FLOW-01: Gia hạn tự động thành công

### Bước 1: Daily Scan (00:00 UTC)

- **Hệ thống** — Chạy scheduled job, query: `expiry_date = TODAY + 7 days AND auto_renew = TRUE AND status = 'Active'`
- INPUT: current_date (Date, required), scan_criteria (Object: {expiry_date, auto_renew, status}, required)
- OUTPUT: eligible_subscriptions (Array of Subscription), scan_completed (Boolean), scan_duration (Number)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Scan hoàn thành | Chuyển sang gửi Email T-7 |
| Query timeout | Auto-retry 3 lần (mỗi lần cách 30 giây) |
| Job failed sau 3 lần auto-retry | CS nhận alert, báo DevOps |

⚠️ Scheduled job không khởi động sau 00:10 UTC VÀ auto-retry đã thất bại → CS nhận alert, báo cáo ngay DevOps, ghi nhận sự cố vào log khẩn cấp

### Bước 2: Gửi Email T-7

- **Hệ thống** — Gửi email thông báo gia hạn: gói, ngày gia hạn, số tiền, link opt-out, link đổi gói
- INPUT: eligible_subscriptions (Array, required), email_template_id (String, required)
- OUTPUT: emails_sent_count (Number), delivery_status (Array of {subscription_id, status}), failed_emails (Array)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| KH opt-out (click link trong email) | Chuyển FLOW-02 |
| KH đổi gói (click "Change Plan") | Quy trình đổi gói riêng |
| KH không hành động | Tiếp tục đến T=0 |
| Email bounced | Auto-retry qua SMS |

⚠️ Email service sập > 10 phút VÀ SMS cũng failed → CS nhận alert, escalate Kỹ thuật

### Bước 3: T=0 Validate thẻ

- **Hệ thống** — Kiểm tra: thẻ chưa hết hạn, status ≠ Blocked, token còn valid
- INPUT: payment_method_id (UUID, required), subscription_id (UUID, required)
- OUTPUT: card_valid (Boolean), validation_reason (String), card_expiry (Date), card_status (String)

Validation Rules:

| Field | Rule | Message lỗi |
|-------|------|-------------|
| card_expiry | expiry_date > TODAY | "Thẻ đã hết hạn" |
| card_status | status ≠ Blocked/Cancelled | "Thẻ đã bị khóa" |
| card_token | Token valid với Payment Gateway | "Token thẻ không hợp lệ" |

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Thẻ hợp lệ | Tiếp tục thanh toán (Bước 4) |
| Thẻ KHÔNG hợp lệ (expired/blocked/token invalid) | FLOW-03 Nhánh B (Grace Period) |
| API timeout | Auto-retry 3 lần (mỗi lần cách 2 giây) |
| Validate failed kỹ thuật | Phân loại vào Grace Period, KHÔNG skip |

⚠️ Tokenization service sập > 10 phút → CS nhận alert, escalate Kỹ thuật, tạm đưa tất cả thẻ vào Grace Period

### Bước 4: Thanh toán tự động

- **Hệ thống** — Tạo payment request với idempotency key → gửi Payment Gateway → nhận kết quả
- INPUT: subscription_id (UUID, required), amount (Decimal, required), currency (String, required), payment_method_id (UUID, required), idempotency_key (String, required)
- OUTPUT: transaction_id (String), payment_status (String: "succeeded" | "failed"), error_code (String, optional), error_message (String, optional)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Thành công | Tiếp tục cập nhật hệ thống (Bước 5) |
| Thất bại (card_declined, insufficient_funds) | FLOW-03 Nhánh A (Retry Mechanism) |
| Thất bại (invalid_card, expired) | FLOW-03 Nhánh B (Grace Period) |
| Mã lỗi không xác định | Mặc định vào Retry Mechanism |
| Gateway timeout > 10 giây | Auto-retry 1 lần với cùng idempotency key |

⚠️ Payment Gateway sập > 10 phút → CS nhận alert, escalate Kỹ thuật

### Bước 5: Cập nhật hệ thống

- **Hệ thống** — Cập nhật trạng thái sau thanh toán thành công
- INPUT: subscription_id (UUID, required), transaction_id (String, required), payment_amount (Decimal, required)
- OUTPUT: package_status (String: "Active"), next_billing_date (Date), invoice_pdf_url (String), updated_at (DateTime)

Xử lý:
- Package Status = "Active"
- Next Billing Date = TODAY + 1 cycle (monthly/yearly)
- Expiry Date = Next Billing Date
- Generate Invoice PDF (invoice_id, invoice_number, amount, tax, total, transaction_id)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Cập nhật thành công | Tiếp tục gửi email xác nhận (Bước 6) |
| Database update failed | Auto-retry 3 lần (mỗi lần cách 3 giây) |
| Retry vẫn failed | Auto-refund + alert DevOps |

⚠️ Database sập > 10 phút → CS nhận alert, ghi nhận danh sách giao dịch bị ảnh hưởng, báo cáo DevOps xử lý thủ công

### Bước 6: Gửi Email xác nhận + Invoice

- **Hệ thống** — Email: "Your CVH [Package] has been renewed" + đính kèm Invoice PDF
- INPUT: subscription_id (UUID, required), customer_email (String, required), invoice_pdf_url (String, required)
- OUTPUT: email_delivery_status (String), notification_sent (Boolean)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Email gửi thành công | KẾT THÚC FLOW-01 |
| Email failed | Auto-retry qua SMS, In-App notification |
| Tất cả kênh failed | Log lỗi + gửi In-App notification với link xem invoice trong app |

⚠️ Email service + SMS + App notification đều sập > 10 phút → CS nhận alert, escalate Kỹ thuật

---

## FLOW-02: Khách hàng Opt-out

**Entry Points (3 cách):**

| Entry Point | Đường dẫn | Mô tả |
|-------------|-----------|-------|
| A. Settings | Dashboard → Settings → Quản lý gói → Toggle "Tự động gia hạn" | Entry point chính, bất kỳ lúc nào |
| B. Email T-7 | Nhận email → Click "Turn off auto-renewal" | Redirect về Settings |
| C. Dashboard | Dashboard → Banner thông báo gia hạn → "Tắt gia hạn" | Hiện khi còn 7 ngày |

### Bước 1: Vào Settings

- **Khách hàng** — Dashboard → Settings → "Quản lý gói dịch vụ"
- INPUT: user_id (UUID, required), session_token (String, required)
- OUTPUT: subscription_details (Object: {package_name, status, next_billing_date, auto_renew, payment_method}), billing_history (Array)

UI States (Màn hình Settings - Subscription Management):

| State | Hiển thị |
|-------|----------|
| Default | Thông tin gói, thẻ thanh toán, lịch sử thanh toán hiển thị đầy đủ |
| Loading | Skeleton loading cho từng section |
| Error | "Không thể tải thông tin gói. Vui lòng thử lại." |

Wireframe:

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              QUẢN LÝ GÓI DỊCH VỤ                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  GÓI HIỆN TẠI                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Care Plus                                    $99/tháng │    │
│  │  ────────────────────────────────────────────────────── │    │
│  │  Trạng thái: ● Active                                   │    │
│  │  Ngày gia hạn tiếp theo: 26/01/2025                     │    │
│  │  Tự động gia hạn: ✓ Bật                                 │    │
│  │                                                         │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │    │
│  │  │ Nâng cấp gói│  │ Hạ cấp gói  │  │ Hủy gia hạn │      │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  PHƯƠNG THỨC THANH TOÁN                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  💳 Visa ****4242                    Hết hạn: 12/2026   │    │
│  │                                          [Cập nhật thẻ] │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  LỊCH SỬ THANH TOÁN                                             │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  26/12/2024   Care Plus Monthly      $99.00   ✓ Paid    │    │
│  │  26/11/2024   Care Plus Monthly      $99.00   ✓ Paid    │    │
│  │  26/10/2024   Care Plus Monthly      $99.00   ✓ Paid    │    │
│  │                                        [Xem tất cả →]   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Bước 2: Tap Toggle OFF

- **Khách hàng** — Toggle "Tự động gia hạn" từ ON → OFF
- INPUT: toggle_action (String: "off", required)
- OUTPUT: modal_displayed (Boolean)

### Bước 3: Hiển thị Confirmation Modal

- **Hệ thống** — Cảnh báo hậu quả + dropdown lý do hủy
- INPUT: subscription_id (UUID, required), expiry_date (Date, required)
- OUTPUT: modal_rendered (Boolean), features_list (Array of String)

Wireframe:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                            [X]  │
│                                                                 │
│           ⚠️ Xác nhận hủy tự động gia hạn?                      │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Nếu bạn hủy, điều gì sẽ xảy ra:                                │
│                                                                 │
│  • Dịch vụ Care Plus sẽ hết hạn vào 26/01/2025                 │
│  • Bạn sẽ mất quyền truy cập các tính năng:                    │
│    - Video Visit                                                │
│    - Chat với bác sĩ                                            │
│    - Lab Consultation                                           │
│  • Lịch sử y tế vẫn được lưu trữ                               │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Lý do hủy (không bắt buộc):                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  ▼ Chọn lý do...                                        │    │
│  │    • Giá quá cao                                        │    │
│  │    • Không sử dụng đủ                                   │    │
│  │    • Chuyển sang dịch vụ khác                           │    │
│  │    • Cần tạm nghỉ                                       │    │
│  │    • Lý do khác                                         │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌───────────────────────┐  ┌───────────────────────┐           │
│  │      GIỮ GÓI          │  │    XÁC NHẬN HỦY       │           │
│  │    (Quay lại)         │  │                       │           │
│  └───────────────────────┘  └───────────────────────┘           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Validation Rules:

| Field | Rule | Message lỗi |
|-------|------|-------------|
| confirmation | Required (phải click "Xác nhận hủy") | "Phải chọn xác nhận" |
| cancellation_reason | Optional, dropdown | — |

Danh sách lý do hủy (Cancellation Reasons):

| Mã | Lý do | Mô tả |
|----|-------|-------|
| 1 | Too expensive | Giá quá cao |
| 2 | Not using enough | Không sử dụng đủ |
| 3 | Switching provider | Chuyển sang nhà cung cấp khác |
| 4 | Need a break | Cần tạm nghỉ |
| 5 | Other | Lý do khác (free text) |

UI States (Modal):

| State | Hiển thị |
|-------|----------|
| Default | Modal hiện với 2 buttons ("Giữ gói", "Xác nhận hủy") |
| Loading | Button "Xác nhận hủy" hiện spinner |
| Success | Modal đóng, redirect về Settings với toast success |
| Error | Hiện error message, cho phép retry |

### Bước 4: Xác nhận

- **Khách hàng** — Click "Xác nhận" để xác nhận hủy auto-renewal
- INPUT: confirmation (Boolean, required), cancellation_reason (String, optional)
- OUTPUT: confirmation_received (Boolean)

### Bước 5: Xử lý & Hiển thị xác nhận

- **Hệ thống** — Set auto_renew = FALSE, log lý do, hiển thị Toast
- INPUT: subscription_id (UUID, required), confirmation (Boolean, required), cancellation_reason (String, optional)
- OUTPUT: auto_renew_updated (Boolean), cancellation_logged (Boolean), analytics_event_sent (Boolean)

Xử lý:
- Set `auto_renew = FALSE`
- Log `cancellation_reason` (nếu có)
- Log event to analytics
- Hiển thị Toast success: "Đã tắt tự động gia hạn. Gói sẽ hết hạn vào [Date]"
- ⚠️ **KHÔNG gửi email** — chỉ xác nhận In-App

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Xử lý thành công | Toast success, KẾT THÚC FLOW-02 |
| Database update failed | Auto-retry 3 lần (mỗi lần cách 2 giây) |
| Retry vẫn failed | Alert DevOps, giữ trạng thái cũ |
| Double-click | Chỉ xử lý 1 lần (idempotent) |

⚠️ Database sập > 10 phút → CS nhận alert, ghi nhận danh sách yêu cầu opt-out bị tồn đọng, báo cáo DevOps xử lý

---

## FLOW-03: Xử lý lỗi thanh toán

> Trong suốt 7 ngày xử lý, dịch vụ VẪN HOẠT ĐỘNG

**T=0: Thanh toán KHÔNG THÀNH CÔNG**

⛔ Rẽ nhánh theo loại lỗi:

| Điều kiện | Hành động |
|-----------|-----------|
| Thẻ hợp lệ bị từ chối (card_declined, insufficient_funds, do_not_honor) | **Nhánh A: Retry Mechanism** |
| Thẻ không hợp lệ (expired, blocked, token invalid) | **Nhánh B: Grace Period** |
| Mã lỗi không xác định | Mặc định vào Retry Mechanism |

### Nhánh A: Retry Mechanism

#### Bước A1: Ghi nhận Payment Failed

- **Hệ thống** — Nhận error code từ Payment Gateway, ghi nhận vào database
- INPUT: subscription_id (UUID, required), error_code (String, required), error_message (String, required), gateway_transaction_id (String, required)
- OUTPUT: payment_attempt_id (UUID), attempt_number (Integer: 1), failure_logged (Boolean)

#### Bước A2: Gửi Email thông báo

- **Hệ thống** — "Payment Failed – We'll try again" + CTA: "Update Payment Method Now"
- INPUT: customer_email (String, required), error_reason_friendly (String, required), update_card_link (String, required)
- OUTPUT: email_delivery_status (String)

#### Bước A3: Retry 1 (T+1, sau 24h)

- **Hệ thống** — Tự động charge cùng thẻ với idempotency key mới
- INPUT: subscription_id (UUID, required), payment_method_id (UUID, required), amount (Decimal, required), idempotency_key (String, required)
- OUTPUT: payment_status (String: "succeeded" | "failed"), attempt_number (Integer: 2)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Thành công | Chuyển FLOW-01 bước 5-6 (gia hạn) |
| Thất bại | Gửi email thông báo Retry 1 failed, schedule Retry 2 (T+3) |

⚠️ Scheduler sập → CS nhận alert, escalate DevOps

#### Bước A4: Retry 2 (T+3, sau 72h)

- **Hệ thống** — Tự động retry lần 2
- INPUT: subscription_id (UUID, required), payment_method_id (UUID, required), amount (Decimal, required), idempotency_key (String, required)
- OUTPUT: payment_status (String: "succeeded" | "failed"), attempt_number (Integer: 3)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Thành công | Chuyển FLOW-01 bước 5-6 (gia hạn) |
| Thất bại | Gửi email URGENT "Urgent Payment Issue", schedule Retry 3 (T+7) |

⚠️ Scheduler sập → CS nhận alert, escalate DevOps

#### Bước A5: Retry 3 FINAL (T+7)

- **Hệ thống** — Lần thử CUỐI CÙNG — không retry thêm
- INPUT: subscription_id (UUID, required), payment_method_id (UUID, required), amount (Decimal, required), idempotency_key (String, required)
- OUTPUT: payment_status (String: "succeeded" | "failed"), attempt_number (Integer: 4), is_final (Boolean: true)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Thành công | Chuyển FLOW-01 bước 5-6 (gia hạn) |
| Thất bại | **PACKAGE EXPIRED** → gửi email URGENT "Service Suspended – Action Required" → FLOW-04 khi user muốn gia hạn lại |

Bảng tổng hợp Retry:

| Attempt | Thời điểm | Mô tả | Nếu thất bại |
|---------|-----------|-------|--------------|
| Initial | T=0 | Thanh toán lần đầu | Email thông báo, lên lịch T+1 |
| Retry 1 | T+1 (24h) | Tự động retry | Email, lên lịch T+3 |
| Retry 2 | T+3 (72h) | Tự động retry | Email URGENT, lên lịch T+7 |
| Retry 3 | T+7 | FINAL retry | **PACKAGE EXPIRED** |

### Nhánh B: Grace Period

#### Bước B1: Validate thẻ failed

- **Hệ thống** — Thẻ expired, blocked, token invalid
- INPUT: payment_method_id (UUID, required), validation_result (Object: {card_status, error_type})
- OUTPUT: grace_period_triggered (Boolean)

#### Bước B2: Kích hoạt Grace Period 7 ngày

- **Hệ thống** — Kích hoạt grace period, dịch vụ VẪN HOẠT ĐỘNG
- INPUT: subscription_id (UUID, required)
- OUTPUT: in_grace_period (Boolean: true), grace_period_end_date (Date: TODAY + 7), package_status (String: "Active")

Xử lý:
- Package Status vẫn = "Active"
- `in_grace_period = TRUE`
- `grace_period_end_date = TODAY + 7`
- `account_flag = 'grace_period'` (trigger banner toàn app)

#### Bước B3: Gửi Email URGENT

- **Hệ thống** — "Action Required: Update your payment card" + CTA "Update Card Now"
- INPUT: customer_email (String, required), grace_period_end_date (Date, required), update_card_link (String, required)
- OUTPUT: email_delivery_status (String)

#### Bước B4: Hiển thị Warning Banner

- **Hệ thống** — Persistent trên mọi page: "⚠️ Update your payment card by [Date] to continue service"
- INPUT: grace_period_end_date (Date, required), days_remaining (Number, required)
- OUTPUT: banner_displayed (Boolean)

Wireframe:

```
┌─────────────────────────────────────────────────────────────────┐
│  ⚠️ Thẻ thanh toán của bạn đã hết hạn. Vui lòng cập nhật      │
│     trước 02/01/2025 để tiếp tục sử dụng dịch vụ.              │
│                                              [CẬP NHẬT NGAY →] │
└─────────────────────────────────────────────────────────────────┘
```

UI States (Banner theo ngày còn lại):

| Ngày còn lại | Style | Text |
|-------------|-------|------|
| 5-7 ngày | Background: Yellow (#FFF3CD) | "Còn X ngày để cập nhật thẻ thanh toán" |
| 2-4 ngày | Background: Orange (#FFE5D0) | "Chỉ còn X ngày! Cập nhật thẻ ngay" |
| 1 ngày | Background: Red (#F8D7DA) | "KHẨN CẤP: Cập nhật thẻ hôm nay!" |

#### Bước B5: Reminder Emails

- **Hệ thống** — Gửi email nhắc nhở theo lịch
- INPUT: subscription_id (UUID, required), days_remaining (Number, required)
- OUTPUT: reminder_sent (Boolean)

| Thời điểm | Nội dung |
|-----------|----------|
| Day 3 | "4 days left to update" |
| Day 6 | "Final reminder: 1 day left" |

#### Bước B6: KH cập nhật thẻ

- **Khách hàng** — Click "Update Card" → Nhập thẻ mới → Gateway tokenize → Save
- INPUT: card_number (String, required), card_expiry (String: MM/YY, required), card_cvv (String: 3-4 digits, required)
- OUTPUT: new_token (String), card_saved (Boolean)

Validation Rules:

| Field | Rule | Message lỗi |
|-------|------|-------------|
| card_number | Required, valid card number (Luhn) | "Số thẻ không hợp lệ" |
| card_expiry | Required, MM/YY format, > TODAY | "Ngày hết hạn không hợp lệ" |
| card_cvv | Required, 3-4 digits | "Mã CVV không hợp lệ" |

⚠️ Form không tải được > 10 phút → CS nhận alert, escalate Kỹ thuật (KHÔNG hỏi thông tin thẻ qua điện thoại)

#### Bước B7: Verify & Payment

- **Hệ thống** — Verify thẻ mới → Tự động attempt payment
- INPUT: new_payment_method_id (UUID, required), subscription_id (UUID, required), amount (Decimal, required)
- OUTPUT: payment_status (String: "succeeded" | "failed"), transaction_id (String)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Thành công | Chuyển FLOW-01 bước 5-6 (gia hạn), xóa grace_period flag, ẩn banner |
| Thất bại | Hiển thị error, cho retry ngay (KH có thể thử thẻ khác) |

#### Bước B8: Hết 7 ngày không update

- **Hệ thống** — Grace Period expired → **PACKAGE EXPIRED**
- INPUT: subscription_id (UUID, required), grace_period_end_date (Date, required)
- OUTPUT: package_status (String: "Expired"), access_level (String: "limited")

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Grace expired | PACKAGE EXPIRED → Chuyển FLOW-04 khi user muốn gia hạn lại |

### So sánh 2 nhánh

| Tiêu chí | Nhánh A: Retry Mechanism | Nhánh B: Grace Period |
|----------|--------------------------|----------------------|
| **Trigger** | Thẻ hợp lệ nhưng payment declined | Thẻ KHÔNG hợp lệ (expired/block) |
| **Thời gian** | 7 ngày (4 attempts) | 7 ngày |
| **Hành động chính** | Hệ thống tự động retry | Chờ KH cập nhật thẻ |
| **Số lần thử** | 4 lần (T=0, T+1, T+3, T+7) | Không giới hạn (KH manual) |
| **Dịch vụ khi đang xử lý** | Active (không warning) | Active (với warning banner) |
| **Kết quả nếu fail** | PACKAGE EXPIRED → FLOW-04 | PACKAGE EXPIRED → FLOW-04 |

---

## FLOW-04: Gia hạn gói đã hết hạn (Renew Expired Package)

> Khi Package Expired: Tài khoản VẪN HOẠT ĐỘNG. User xem được TẤT CẢ tài liệu/hồ sơ y tế. Chỉ không dùng được services (Video, Chat, AI). Gia hạn lại BẤT CỨ LÚC NÀO (không giới hạn thời gian).

**Quyền truy cập khi Package Expired:**

| Loại | Có thể | Không thể |
|------|--------|-----------|
| Account | Login, View profile, View settings, Update info | — |
| Medical Records | Xem TẤT CẢ hồ sơ y tế, kết quả xét nghiệm, báo cáo đã có | — |
| Documents | Xem và download tất cả tài liệu đã tạo trước đó | — |
| Services | — | Book appointments, Video call, Chat, AI features |
| Billing | Access billing, View invoices, Make payment để gia hạn | — |

**Entry Points (3 cách):**

| Entry Point | Đường dẫn | Mô tả |
|-------------|-----------|-------|
| A. Email | "Gia hạn ngay" trong email | Link trực tiếp đến Payment Form |
| B. Login | Login → Banner/Modal → "Gia hạn" | Tự động hiển thị khi Package Expired |
| C. Settings | Settings → Quản lý gói → "Gia hạn lại" | Entry point trong Settings |

### Modal Package Expired (hiển thị khi login)

Wireframe:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                            [X]  │
│                                                                 │
│                          📦                                     │
│                                                                 │
│              Gói dịch vụ của bạn đã hết hạn                    │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Gói Care Plus của bạn đã hết hạn từ ngày 02/01/2025.          │
│                                                                 │
│  ✅ Bạn VẪN CÓ THỂ:                                            │
│  ✓ Xem TẤT CẢ hồ sơ y tế, kết quả xét nghiệm                  │
│  ✓ Download tài liệu đã tạo                                    │
│  ✓ Cập nhật thông tin cá nhân                                  │
│  ✓ Xem lịch sử thanh toán                                      │
│                                                                 │
│  ❌ Bạn KHÔNG THỂ (cho đến khi gia hạn):                       │
│  ✗ Đặt lịch khám Video                                         │
│  ✗ Chat với bác sĩ                                             │
│  ✗ Sử dụng AI Health Assistant                                 │
│                                                                 │
│  💡 Bạn có thể gia hạn BẤT CỨ LÚC NÀO                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                GIA HẠN NGAY - $99.00                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│                [Tiếp tục xem tài liệu] (bỏ qua)                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Navigation:
- Click "GIA HẠN NGAY" → Màn hình Renewal Payment
- Click "Tiếp tục xem tài liệu" → Dashboard (xem tài liệu, không dùng services)
- Click [X] → Dashboard

### Nhánh A & B: User tự gia hạn (Thẻ đã lưu / Thẻ mới)

#### Bước 1: Initiate Renewal

- **Khách hàng** — Click "Gia hạn ngay" từ email, banner login, hoặc Settings
- INPUT: user_id (UUID, required), entry_point (String: "email" | "login_banner" | "settings", required)
- OUTPUT: renewal_form_displayed (Boolean)

#### Bước 2: Hiển thị Payment Form

- **Hệ thống** — Hiển thị form thanh toán gia hạn
- INPUT: subscription_id (UUID, required), package_info (Object, required), saved_payment_methods (Array, optional)
- OUTPUT: form_rendered (Boolean)

Wireframe:

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]                GIA HẠN GÓI DỊCH VỤ                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Gia hạn gói: Care Plus                                         │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│  Chi tiết thanh toán                                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Care Plus (Monthly)                            $99.00  │    │
│  │  ─────────────────────────────────────────────────────  │    │
│  │  Subtotal                                       $99.00  │    │
│  │  Tax (10%)                                       $9.90  │    │
│  │  ─────────────────────────────────────────────────────  │    │
│  │  TỔNG CỘNG                                    $108.90  │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Phương thức thanh toán                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  ● 💳 Visa ****4242 (Hết hạn: 12/2026)                  │    │
│  │  ○ Thêm thẻ mới                                         │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ☑️ Bật tự động gia hạn                                        │
│  ☑️ Lưu làm phương thức mặc định                               │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │             THANH TOÁN $108.90 & GIA HẠN                │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  🔐 Thanh toán được bảo mật bởi Stripe                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Validation Rules:

| Field | Rule | Message lỗi |
|-------|------|-------------|
| payment_method | Required | "Vui lòng chọn phương thức thanh toán" |
| new_card_details | Valid card (nếu chọn thẻ mới) | "Thông tin thẻ không hợp lệ" |

UI States:

| State | Hiển thị |
|-------|----------|
| Default | Form hiển thị, button enabled |
| Loading | Button hiện spinner "Đang xử lý..." |
| Success | Redirect về Dashboard với toast "Kích hoạt thành công!" |
| Error | Hiện error inline, button vẫn enabled để retry |

#### Bước 3: Submit Payment

- **Khách hàng** — Nhập thông tin (nếu thẻ mới) → Click "Thanh toán & Gia hạn"
- INPUT: payment_method (String: payment_method_id | "new", required), new_card_details (Object, optional), auto_renew (Boolean, required), save_as_default (Boolean, optional)
- OUTPUT: payment_submitted (Boolean)

#### Bước 4: Process Payment

- **Payment Gateway** — Xử lý thanh toán
- INPUT: payment_request (Object: {amount, currency, payment_method, idempotency_key}, required)
- OUTPUT: payment_result (Object: {status, transaction_id, error_code})

#### Bước 5: Xử lý kết quả

- **Hệ thống** — Cập nhật trạng thái hoặc hiển thị lỗi
- INPUT: payment_result (Object, required), subscription_id (UUID, required)
- OUTPUT: package_status (String), next_billing_date (Date), access_level (String)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Thành công | Package Status = "Active", Next Billing Date = TODAY + 1 cycle, email confirmation, user dùng services ngay |
| Thất bại | Hiển thị error inline, cho phép retry ngay, không giới hạn số lần |

### Nhánh C: Adult tự gia hạn khi Primary từ chối

> Điều kiện: Primary đã tắt auto-renewal cho Adult, hoặc gói của Adult đã hết hạn do Primary không thanh toán.

#### Bước C1: Adult nhận thông báo

- **Hệ thống** — Push + In-app: "Gói của bạn sẽ hết hạn. Bạn có thể tự gia hạn bằng thẻ riêng."
- INPUT: adult_user_id (UUID, required), subscription_id (UUID, required), primary_name (String, required)
- OUTPUT: notification_sent (Boolean)

#### Bước C2: Adult vào Quản lý gói

- **Adult** — Settings → Quản lý gói dịch vụ
- INPUT: adult_user_id (UUID, required)
- OUTPUT: subscription_details (Object), self_renewal_available (Boolean: true)

Wireframe (Settings của Adult khi gói sắp/đã hết hạn):

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              QUẢN LÝ GÓI DỊCH VỤ                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ⚠️ GÓI CỦA BẠN SẮP HẾT HẠN                                    │
│                                                                 │
│  GÓI HIỆN TẠI                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Care Plus                                    $99/tháng │    │
│  │  ────────────────────────────────────────────────────── │    │
│  │  Trạng thái: ⚠️ Sắp hết hạn (26/01/2025)               │    │
│  │  Tự động gia hạn: ✗ Đã tắt (bởi Primary)               │    │
│  │                                                         │    │
│  │  ⓘ Primary đã tắt tự động gia hạn cho gói của bạn      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  💡 BẠN CÓ THỂ TỰ GIA HẠN                                       │
│                                                                 │
│  Nếu muốn tiếp tục sử dụng dịch vụ, bạn có thể tự gia hạn     │
│  bằng thẻ thanh toán của riêng mình.                           │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │          TỰ GIA HẠN BẰNG THẺ CỦA TÔI                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ⓘ Bạn vẫn thuộc nhóm gia đình của [Tên Primary]              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Bước C3: Hiển thị option tự gia hạn

- **Hệ thống** — Hiển thị gói hiện tại + Button "Tự gia hạn bằng thẻ của tôi"
- INPUT: subscription_id (UUID, required), package_info (Object, required)
- OUTPUT: self_renewal_option_displayed (Boolean)

#### Bước C4: Adult chọn tự gia hạn

- **Adult** — Click "Tự gia hạn bằng thẻ của tôi"
- INPUT: adult_user_id (UUID, required)
- OUTPUT: payment_form_displayed (Boolean)

#### Bước C5: Hiển thị Payment Form

- **Hệ thống** — Form thanh toán cho Adult tự gia hạn
- INPUT: subscription_id (UUID, required), package_info (Object, required)
- OUTPUT: form_rendered (Boolean)

Wireframe:

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              TỰ GIA HẠN GÓI DỊCH VỤ                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ⓘ Bạn đang tự gia hạn gói dịch vụ bằng thẻ của mình          │
│                                                                 │
│  Gia hạn gói: Care Plus                                         │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│  Chi tiết thanh toán                                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Care Plus (Monthly)                            $99.00  │    │
│  │  ─────────────────────────────────────────────────────  │    │
│  │  Subtotal                                       $99.00  │    │
│  │  Tax (10%)                                       $9.90  │    │
│  │  ─────────────────────────────────────────────────────  │    │
│  │  TỔNG CỘNG                                    $108.90  │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Thêm thẻ thanh toán của bạn                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Số thẻ *                                               │    │
│  │  ┌───────────────────────────────────────────────────┐  │    │
│  │  │                                                   │  │    │
│  │  └───────────────────────────────────────────────────┘  │    │
│  │                                                         │    │
│  │  Ngày hết hạn *          CVV *                         │    │
│  │  ┌────────────────┐      ┌────────────────┐            │    │
│  │  │  MM / YY       │      │  •••           │            │    │
│  │  └────────────────┘      └────────────────┘            │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ☑️ Bật tự động gia hạn (bằng thẻ của tôi)                     │
│  ☑️ Lưu thẻ cho lần sau                                        │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │             THANH TOÁN $108.90 & GIA HẠN                │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  🔐 Thanh toán được bảo mật bởi Stripe                         │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│  ⓘ Sau khi gia hạn:                                            │
│  • Bạn VẪN THUỘC nhóm gia đình của [Tên Primary]               │
│  • [Tên Primary] sẽ nhận thông báo                             │
│  • Bạn tự quản lý và thanh toán gói từ nay về sau              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Validation Rules:

| Field | Rule | Message lỗi |
|-------|------|-------------|
| card_number | Required, valid card | "Số thẻ không hợp lệ" |
| card_expiry | Required, MM/YY format | "Ngày hết hạn không hợp lệ" |
| card_cvv | Required, 3-4 digits | "Mã CVV không hợp lệ" |

#### Bước C6: Adult submit payment

- **Adult** — Nhập thông tin thẻ → Click "Thanh toán & Gia hạn"
- INPUT: card_number (String, required), card_expiry (String, required), card_cvv (String, required), auto_renew (Boolean, required), save_card (Boolean, optional)
- OUTPUT: payment_submitted (Boolean)

#### Bước C7: Process Payment

- **Payment Gateway** — Xử lý thanh toán
- INPUT: payment_request (Object, required)
- OUTPUT: payment_result (Object: {status, transaction_id, error_code})

#### Bước C8: Thành công

- **Hệ thống** — Cập nhật trạng thái + thay đổi billing owner
- INPUT: subscription_id (UUID, required), adult_user_id (UUID, required), transaction_id (String, required)
- OUTPUT: package_status (String: "Active"), paid_by (String: "adult"), family_group_unchanged (Boolean: true)

Xử lý:
- Package Status = "Active"
- `paid_by = "adult"` (thay vì "primary")
- Adult VẪN THUỘC family group của Primary

#### Bước C9: Thông báo cho Primary

- **Hệ thống** — Push cho Primary: "[Tên Adult] đã tự gia hạn gói dịch vụ bằng thẻ riêng."
- INPUT: primary_user_id (UUID, required), adult_name (String, required)
- OUTPUT: notification_sent (Boolean)

**Mô hình Billing sau khi Adult tự gia hạn:**

| Tiêu chí | Trước (Primary trả) | Sau (Adult tự trả) |
|----------|---------------------|---------------------|
| Ai thanh toán | Primary | **Adult** |
| Adult thuộc family group | Có | **Vẫn có** (không thay đổi) |
| Primary thấy Adult trong danh sách | Có | **Vẫn có** (không thay đổi) |
| Primary quản lý gói Adult | Có | **Không** (Adult tự quản lý) |
| Field `paid_by` | "primary" | **"adult"** |

---

## FLOW-05: Primary quản lý gia hạn cho Adult Members

> Primary có TOÀN QUYỀN bật/tắt auto-renewal cho Adult. Adult KHÔNG CẦN xác nhận, chỉ nhận thông báo. Adult VẪN CÓ THỂ tự tắt auto-renewal từ Settings của họ.

### Bước 1: Vào Quản lý gia đình

- **Primary** — Dashboard → Quản lý gia đình
- INPUT: primary_user_id (UUID, required)
- OUTPUT: family_members (Array of {member_id, name, role, package_name, auto_renew_status, next_billing_date})

Wireframe (Danh sách Family Members):

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              QUẢN LÝ GIA ĐÌNH                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  THÀNH VIÊN GIA ĐÌNH                                            │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  👨 Nguyễn Văn A (Adult)                           [>]  │    │
│  │  Care Plus • $99/tháng • Gia hạn: 26/01/2025           │    │
│  │  Tự động gia hạn: ✓ Bật                                │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  👩 Trần Thị B (Adult)                             [>]  │    │
│  │  Care Connect • $39/tháng • Gia hạn: 15/01/2025        │    │
│  │  Tự động gia hạn: ✗ Tắt                                │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  [+] Thêm thành viên mới                               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Bước 2: Chọn Adult member

- **Primary** — Chọn Adult muốn quản lý gia hạn
- INPUT: adult_member_id (UUID, required)
- OUTPUT: adult_subscription_details (Object)

### Bước 3: Tap "Quản lý gói dịch vụ"

- **Primary** — Xem chi tiết gói của Adult
- INPUT: adult_member_id (UUID, required)
- OUTPUT: adult_package_details (Object: {package_name, status, next_billing_date, auto_renew, paid_by, payment_method})

Wireframe (Chi tiết quản lý gói của Adult):

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]        QUẢN LÝ GÓI - NGUYỄN VĂN A                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  GÓI HIỆN TẠI                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Care Plus                                    $99/tháng │    │
│  │  ────────────────────────────────────────────────────── │    │
│  │  Trạng thái: ● Active                                   │    │
│  │  Ngày gia hạn tiếp theo: 26/01/2025                     │    │
│  │                                                         │    │
│  │  Tự động gia hạn:                                       │    │
│  │  ┌────────────────────────────────────────────┐         │    │
│  │  │  🔘 ON ────────────────────────────── OFF  │         │    │
│  │  └────────────────────────────────────────────┘         │    │
│  │                                                         │    │
│  │  ⓘ Bạn thanh toán cho gói này                          │    │
│  │                                                         │    │
│  │  ┌─────────────┐  ┌─────────────┐                       │    │
│  │  │ Nâng cấp gói│  │ Hạ cấp gói  │                       │    │
│  │  └─────────────┘  └─────────────┘                       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  THÔNG TIN THANH TOÁN                                           │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Thanh toán bởi: Primary (Bạn)                          │    │
│  │  Thẻ: 💳 Visa ****4242                                  │    │
│  │  Chu kỳ tiếp theo: $99.00 vào 26/01/2025               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Bước 4: Toggle "Tự động gia hạn" ON/OFF

- **Primary** — Toggle auto-renewal cho Adult
- INPUT: adult_member_id (UUID, required), auto_renew (Boolean, required)
- OUTPUT: confirmation_modal_displayed (Boolean)

### Bước 5: Hiển thị xác nhận

- **Hệ thống** — Hiển thị modal xác nhận cho Primary

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Nếu TẮT | Cảnh báo: "Gói của [Adult] sẽ hết hạn vào [Date]" |
| Nếu BẬT | Thông báo: "Gói sẽ tự động gia hạn vào [Date] với $[Amount]" |

Wireframe (Modal xác nhận tắt gia hạn cho Adult):

```
┌─────────────────────────────────────────────────────────────────┐
│                                                            [X]  │
│                                                                 │
│           ⚠️ Tắt tự động gia hạn cho Nguyễn Văn A?             │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Nếu bạn tắt, điều gì sẽ xảy ra:                                │
│                                                                 │
│  • Gói Care Plus của Nguyễn Văn A sẽ hết hạn vào 26/01/2025    │
│  • Sau ngày này, thành viên sẽ mất quyền truy cập:             │
│    - Video Visit                                                │
│    - Chat với bác sĩ                                            │
│    - Lab Consultation                                           │
│  • Nguyễn Văn A sẽ nhận thông báo về thay đổi này              │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  ┌───────────────────────┐  ┌───────────────────────┐           │
│  │      GIỮ GÓI          │  │    XÁC NHẬN TẮT       │           │
│  │    (Quay lại)         │  │                       │           │
│  └───────────────────────┘  └───────────────────────┘           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Bước 6: Primary Click "Xác nhận"

- **Primary** — Xác nhận thay đổi
- INPUT: confirmation (Boolean, required)
- OUTPUT: confirmation_received (Boolean)

### Bước 7: Xử lý hệ thống

- **Hệ thống** — Cập nhật auto_renew + thông báo Adult
- INPUT: adult_member_id (UUID, required), auto_renew (Boolean, required), primary_user_id (UUID, required)
- OUTPUT: auto_renew_updated (Boolean), notification_sent_to_adult (Boolean), updated_at (DateTime)

Xử lý:
- Set `auto_renew = TRUE/FALSE`
- Gửi Push + In-app notification cho Adult: "[Primary] đã [bật/tắt] tự động gia hạn cho gói của bạn"

**Mô hình quyền quản lý gia hạn:**

| Hành động | Primary | Adult |
|-----------|---------|-------|
| Bật/Tắt auto-renewal cho chính mình | ✅ Có quyền | ✅ Có quyền |
| Bật/Tắt auto-renewal cho Adult | ✅ TOÀN QUYỀN | ✅ Có quyền (tự quản lý) |
| Nhận thông báo khi Primary thay đổi | — | ✅ Push + In-app |
| Thanh toán gia hạn cho Adult | ✅ Primary thanh toán | ✅ Adult tự trả (MỚI) |
| Tự gia hạn khi Primary từ chối | — | ✅ CÓ THỂ (MỚI) → FLOW-04 |

> Lưu ý: Khi cả Primary và Adult cùng quản lý: **Hành động sau cùng sẽ có hiệu lực**

---

## Bảng thông báo tự động (mở rộng có template)

| Sự kiện | Kênh | Người nhận | Nội dung |
|---------|------|------------|----------|
| Daily Scan hoàn thành | Email | KH | Thông báo gia hạn sắp diễn ra (T-7) |
| Thanh toán thành công | Email | KH | "Your CVH Package has been renewed" + Invoice PDF |
| Payment Failed (Retry) | Email | KH | "Payment Failed – We'll try again" + CTA update thẻ |
| Retry 1 thất bại | Email | KH | Thông báo retry 1 failed |
| Retry 2 thất bại | Email URGENT | KH | Cảnh báo nghiêm trọng |
| Retry 3 FINAL thất bại | Email URGENT | KH | "Service Suspended – Action Required" |
| Thẻ không hợp lệ | Email URGENT | KH | "Action Required: Update your payment card" |
| Grace Day 3 | Email | KH | "4 days left to update" |
| Grace Day 6 | Email | KH | "Final reminder: 1 day left" |
| Opt-out xác nhận | Toast In-App | KH | "Đã tắt tự động gia hạn" (KHÔNG gửi email) |
| Reactivate thành công | Email | KH | "Welcome back!" + Invoice PDF |
| Primary bật/tắt cho Adult | Push + In-App | Adult | Thông báo thay đổi |
| Adult tự gia hạn | Push | Primary | "[Adult] đã tự gia hạn bằng thẻ riêng" |
| Package Expired → KH chưa renew | Email | KH | Gợi ý gia hạn lại (FLOW-04 entry) |
| Hệ thống gặp sự cố fatal | Alert | CS/DevOps | Escalation tự động |

### Template chi tiết

📧 **Email T-7: Thông báo gia hạn sắp diễn ra**
- Subject: "Your CVH [Package Name] will renew on [Date] for $[Amount]"
- Body: Tên KH, Gói, Chu kỳ, Số tiền, Ngày thanh toán, Thẻ (masked), Link quản lý cài đặt, Link đổi gói, Link hủy tự động gia hạn

📧 **Email Xác nhận Opt-out**
- Subject: "Auto-renewal turned off for your CVH [Package Name]"
- Body: Tên KH, Gói, Ngày hết hạn, Danh sách tính năng sẽ mất, Lịch sử y tế vẫn lưu, Link bật lại auto-renewal

📧 **Email Payment Failed**
- Subject: "Payment Failed for CVH Renewal - We'll try again"
- Body: Tên KH, Số tiền, Thẻ (masked), Lý do decline, Cam kết dịch vụ vẫn hoạt động, Tự động retry 24h, Link cập nhật phương thức thanh toán

📧 **Email URGENT - Final Retry Warning**
- Subject: "⚠️ URGENT: Update Payment Card - Final Retry in 72 Hours"
- Body: Tên KH, Đã thử 3 lần thất bại, Lần cuối trong 72h, Hậu quả nếu thất bại, Link cập nhật ngay

📧 **Email Package Expired**
- Subject: "Your CVH [Package Name] has expired - Renew anytime"
- Body: Tên KH, Danh sách CÓ THỂ (login, xem hồ sơ, download), Danh sách KHÔNG THỂ (Video, Chat, AI), Link gia hạn ngay, Không giới hạn thời gian

📧 **Email Renewal Successful**
- Subject: "Your CVH [Package Name] has been renewed - Receipt inside"
- Body: Tên KH, Invoice #, Ngày thanh toán, Gói, Chu kỳ, Số tiền, Mã giao dịch, Ngày thanh toán tiếp theo, Invoice PDF đính kèm

📧 **Email Reactivation Successful**
- Subject: "Welcome back! Your CVH [Package Name] is reactivated"
- Body: Tên KH, Danh sách tính năng đã hoạt động lại, Ngày hết hạn mới, Số tiền, Mã giao dịch, Invoice PDF đính kèm

📱 **Push - Primary TẮT auto-renewal cho Adult**
- "Tự động gia hạn đã bị tắt. [Tên Primary] đã tắt tự động gia hạn cho gói Care Plus của bạn. Gói sẽ hết hạn vào [Date]."

📱 **Push - Primary BẬT auto-renewal cho Adult**
- "Tự động gia hạn đã được bật. [Tên Primary] đã bật tự động gia hạn cho gói Care Plus. Gói sẽ gia hạn vào [Date] với $[Amount]."

📱 **Push - Adult tự tắt gia hạn (gửi cho Primary)**
- "Thành viên đã tắt tự động gia hạn. [Tên Adult] đã tắt tự động gia hạn cho gói Care Plus. Gói sẽ hết hạn vào [Date]."

📱 **Push - Adult có thể tự gia hạn (khi Primary từ chối)**
- "Gói dịch vụ sắp hết hạn. [Tên Primary] đã tắt tự động gia hạn. Bạn có thể tự gia hạn bằng thẻ riêng của mình."

📱 **Push - Adult tự gia hạn thành công (gửi cho Primary)**
- "Thành viên đã tự gia hạn. [Tên Adult] đã tự gia hạn gói Care Plus bằng thẻ riêng của họ."

---

## Bảng giám sát CS-MD đầy đủ

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Daily Scan | Hoàn thành < 3 phút; 100% gói đủ ĐK được query; 0% bỏ sót | Chạy 00:00 UTC, query DB, retry 3 lần (cách 30s), auto-alert DevOps nếu fail | Job không khởi động sau 00:10 UTC VÀ auto-retry thất bại → báo DevOps | ❌ |
| Gửi Email T-7 | Gửi < 5 phút sau scan; 100% delivery; 0% gửi nhầm | Trigger email queue, auto-fill template, retry qua SMS nếu bounced, alert nếu lỗi > 5% | Email service sập > 10 phút VÀ SMS failed → escalate Kỹ thuật | ❌ |
| Validate Thẻ | Validate < 3 giây/thẻ; 100% chính xác; 0% skip | Gọi API tokenization, retry 3 lần (cách 2s), classify vào Grace nếu failed kỹ thuật | Tokenization service sập > 10 phút → escalate Kỹ thuật, tạm Grace Period | ❌ |
| Thanh Toán | Kết quả < 10 giây; 0% charge sai; 0% charge trùng | Idempotency key, retry 1 lần nếu timeout, ghi nhận kết quả ngay | Payment Gateway sập > 10 phút → escalate Kỹ thuật | ❌ |
| Cập Nhật Hệ Thống | < 1 phút; 100% Active; 100% billing date đúng | Cập nhật DB, generate Invoice PDF, retry 3 lần (cách 3s), auto-refund nếu vẫn fail | Database sập > 10 phút → ghi danh sách giao dịch bị ảnh hưởng, báo DevOps | ❌ |
| Gửi Email Xác Nhận | ≤ 1 phút; 100% delivery; Invoice đính kèm đúng | Trigger email, đính kèm Invoice, retry qua SMS/In-App nếu fail | Email + SMS + App notification sập > 10 phút → escalate Kỹ thuật | ❌ |
| Nhận Email T-7 (KH) | Email delivered; 100% link opt-out hoạt động | Theo dõi delivery, test link opt-out, retry qua SMS nếu bounced | Email service sập > 10 phút VÀ SMS failed → escalate Kỹ thuật | ❌ |
| Click Opt-out | 100% redirect chính xác; tải trang < 2 giây | Token xác thực, redirect đúng trang Settings, optimize cache | Trang cài đặt sập > 10 phút → escalate Kỹ thuật | ❌ |
| Hiển Thị Modal | < 1 giây; 100% nội dung đầy đủ; dropdown hoạt động | Render modal với dữ liệu thực, fallback static nếu JS error | Frontend sập > 10 phút → escalate Kỹ thuật | ❌ |
| Xác Nhận (KH click) | 100% ghi nhận tín hiệu; 0% opt-out khi chưa confirm | Lắng nghe click, validate session, idempotent (1 lần) | ❌ | ❌ |
| Xử Lý Hệ Thống (opt-out) | < 3 giây; 100% auto_renew=FALSE; 100% log reason | Cập nhật DB, log analytics, retry 3 lần (cách 2s), giữ trạng thái cũ nếu fail | Database sập > 10 phút → ghi danh sách opt-out tồn đọng, báo DevOps | ❌ |
| Payment Failed | Nhận + ghi mã lỗi < 20 giây; 0% bỏ sót; 0% ghi nhầm | Nhận callback, parse mã lỗi, timeout 20s = failed, chuyển Phân Loại Lỗi | Payment Gateway sập > 10 phút → ghi danh sách giao dịch treo | ❌ |
| Phân Loại Lỗi | < 3 giây; 100% phân loại đúng nhánh | Tra bảng phân loại mã lỗi, mã không xác định → Retry (mặc định an toàn) | ❌ | ❌ |
| Gửi Email Thông Báo (failed) | ≤ 1 phút; 100% delivery; link update thẻ hoạt động | Auto-fill template, test link, retry qua SMS | Email + SMS sập > 10 phút → escalate Kỹ thuật | ❌ |
| Retry 1 (T+1) | Kích hoạt đúng T+1 (±30 phút); dùng đúng thẻ | Scheduler, idempotency key mới, thành công → gia hạn, thất bại → email + schedule Retry 2 | Scheduler sập, không kích hoạt sau T+1 + 1h → escalate DevOps | ❌ |
| Retry 2 (T+3) | Kích hoạt đúng T+3 (±30 phút); email URGENT khi fail | Scheduler, thành công → gia hạn, thất bại → email URGENT + schedule Retry 3 | Scheduler sập, không kích hoạt sau T+3 + 1h → escalate DevOps | ❌ |
| Retry 3 FINAL (T+7) | Kích hoạt đúng T+7 (±30 phút); là lần cuối; EXPIRED < 1 phút nếu fail | Scheduler, lần cuối, thành công → gia hạn, thất bại → PACKAGE EXPIRED + email URGENT | Scheduler sập, không kích hoạt sau T+7 + 1h → escalate DevOps | ❌ |
| Kích Hoạt Grace Period | Banner < 1 phút; 100% đúng 7 ngày; service ACTIVE | Set grace_period_end, account_flag, banner toàn app, hết 7 ngày → EXPIRED | ❌ | ❌ |
| KH Update Thẻ | Form < 2 giây; 100% tokenize (PCI-DSS); 0% plain text | Form + validate real-time, tokenize qua Gateway (không qua server CVH), auto charge | Form không tải > 10 phút → escalate Kỹ thuật (KHÔNG hỏi thẻ qua ĐT) | ❌ |
| Xử Lý Kết Quả | Chuyển luồng < 1 phút; email ≤ 2 phút | Thành công → gia hạn, thất bại → EXPIRED, gửi email, log audit | Database sập khi cập nhật → ghi danh sách treo, báo DevOps | ❌ |
| Quyết Định Reactivate | 100% nút hoạt động từ 3 entry point; tải < 2 giây | Test định kỳ, auto-generate link mới nếu hỏng, redirect nếu đã active | Tất cả 3 entry point sập > 10 phút → escalate Kỹ thuật | ❌ |
| Hiển Thị Payment Form (reactivate) | Form < 2 giây; 100% số tiền đúng; auto-renew mặc định TRUE | Load từ DB, masked card, validate số tiền vs bảng giá | Form không tải > 10 phút → escalate Kỹ thuật | ❌ |
| Submit Payment (reactivate) | Validate đầy đủ; 0% submit thiếu info; 100% mã hóa | Validate real-time, nút chỉ active khi hợp lệ, iframe Gateway, idempotency key | ❌ | ❌ |
| Xử Lý Payment (reactivate) | Kết quả < 30 giây; 0% charge sai; 0% charge trùng | Idempotency key, retry 1 lần nếu timeout, ghi nhận kết quả | Payment Gateway sập > 10 phút → escalate Kỹ thuật | ❌ |
| Cập Nhật Hệ Thống (reactivate) | < 1 phút; Active + Full access; billing date đúng | Cập nhật status/access, xóa grace/suspended flag, ẩn banner, auto-refund nếu DB fail | Database sập > 10 phút sau charge → ghi danh sách treo, báo DevOps refund | ❌ |
| Gửi Email Xác Nhận (reactivate) | ≤ 2 phút; 100% delivery; tone tích cực | Template "Welcome back!", Invoice PDF, retry In-App nếu fail | Email + App notification sập > 10 phút → escalate Kỹ thuật | ❌ |

---

## Data Specification

### Subscription Object

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| subscription_id | UUID | Auto | "sub_abc123" | ID duy nhất của subscription |
| customer_id | UUID | Yes | "cus_xyz789" | ID khách hàng |
| package_id | String | Yes | "pkg_care_plus" | ID gói dịch vụ |
| package_name | String | Yes | "Care Plus" | Tên gói |
| status | Enum | Yes | "active" \| "expired" | Trạng thái gói |
| billing_cycle | Enum | Yes | "monthly" \| "yearly" | Chu kỳ thanh toán |
| price | Decimal | Yes | 99.00 | Giá gói (USD) |
| auto_renew | Boolean | Yes | true/false | Tự động gia hạn |
| current_period_start | DateTime | Yes | "2024-12-26T00:00:00Z" | Ngày bắt đầu kỳ hiện tại |
| current_period_end | DateTime | Yes | "2025-01-26T00:00:00Z" | Ngày kết thúc kỳ hiện tại |
| next_billing_date | DateTime | Yes | "2025-01-26T00:00:00Z" | Ngày thanh toán tiếp theo |
| payment_method_id | UUID | Yes | "pm_def456" | ID phương thức thanh toán |
| in_grace_period | Boolean | Yes | false | Đang trong grace period |
| grace_period_end_date | DateTime | No | null | Ngày hết grace period |
| scheduled_downgrade | Object | No | null | Thông tin hạ cấp đã lên lịch |
| cancellation_reason | String | No | null | Lý do hủy (nếu opt-out) |
| paid_by | Enum | Yes | "primary" \| "adult" | Ai thanh toán (cho Family member) |
| created_at | DateTime | Auto | "2024-10-26T10:00:00Z" | Ngày tạo subscription |
| updated_at | DateTime | Auto | "2024-12-26T10:00:00Z" | Ngày cập nhật gần nhất |

### Payment Attempt Object

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| attempt_id | UUID | Auto | "att_abc123" | ID lần thử thanh toán |
| subscription_id | UUID | Yes | "sub_abc123" | ID subscription liên quan |
| amount | Decimal | Yes | 99.00 | Số tiền thanh toán |
| currency | String | Yes | "USD" | Loại tiền |
| status | Enum | Yes | "succeeded" \| "failed" \| "pending" | Trạng thái |
| attempt_number | Integer | Yes | 1-4 | Số lần thử |
| payment_method_id | UUID | Yes | "pm_def456" | ID phương thức thanh toán |
| error_code | String | No | "card_declined" | Mã lỗi (nếu failed) |
| error_message | String | No | "Your card was declined" | Mô tả lỗi |
| gateway_transaction_id | String | Yes | "ch_xyz789" | ID giao dịch từ Payment Gateway |
| created_at | DateTime | Auto | "2024-12-26T00:00:00Z" | Thời gian thử |

### Invoice Object

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| invoice_id | UUID | Auto | "inv_abc123" | ID hóa đơn |
| invoice_number | String | Auto | "INV-2024-001234" | Số hóa đơn |
| customer_id | UUID | Yes | "cus_xyz789" | ID khách hàng |
| subscription_id | UUID | Yes | "sub_abc123" | ID subscription |
| amount_due | Decimal | Yes | 99.00 | Số tiền phải trả |
| amount_paid | Decimal | Yes | 99.00 | Số tiền đã trả |
| tax | Decimal | Yes | 9.90 | Thuế |
| total | Decimal | Yes | 108.90 | Tổng cộng |
| status | Enum | Yes | "paid" \| "open" \| "void" | Trạng thái |
| payment_intent_id | UUID | Yes | "pi_abc123" | ID payment intent |
| pdf_url | String | Yes | "https://..." | URL file PDF |
| created_at | DateTime | Auto | "2024-12-26T00:00:00Z" | Ngày tạo |
| paid_at | DateTime | No | "2024-12-26T00:01:00Z" | Ngày thanh toán |

### Renewal Event Types

| Event Type | Description | Trigger |
|------------|-------------|---------|
| `renewal.notification_sent` | Đã gửi email thông báo T-7 | Daily scan |
| `renewal.payment_succeeded` | Thanh toán gia hạn thành công | Payment Gateway response |
| `renewal.payment_failed` | Thanh toán gia hạn thất bại | Payment Gateway response |
| `renewal.retry_scheduled` | Đã lên lịch retry | After payment failure |
| `renewal.retry_attempted` | Đang thử retry | Scheduled job |
| `renewal.opted_out` | KH đã opt-out | Customer action |
| `renewal.grace_period_started` | Bắt đầu grace period | Card validation failed |
| `renewal.card_updated` | KH đã cập nhật thẻ | Customer action |
| `subscription.expired` | Gói dịch vụ hết hạn | All retries failed / Grace expired / Opt-out |
| `subscription.renewed` | Gói được gia hạn lại | Customer manual payment |
| `subscription.adult_self_renewed` | Adult tự gia hạn bằng thẻ riêng | Adult manual payment |

---

## Compliance & Security

### HIPAA Compliance — PHI Risk: LOW

| Loại thông tin | Xử lý | Ghi chú |
|----------------|--------|---------|
| Subscription info | ✅ Có | Package, billing cycle, dates |
| Payment info | ✅ Có | Card last 4 digits, amount, transaction |
| Account info | ✅ Có | Name, email, customer ID |
| Medical records | ❌ Không | Không truy cập, không gửi qua email |
| Diagnosis/Treatment | ❌ Không | Không liên quan đến quy trình này |

### PCI DSS Compliance

| Requirement | Implementation |
|-------------|---------------|
| Tokenization | CVH không lưu raw card numbers |
| Secure transmission | TLS 1.3 cho mọi API calls |
| 3D Secure | Áp dụng cho high-risk transactions |
| Access control | Chỉ System/Gateway có quyền charge |
| Audit logging | Mọi transaction được log đầy đủ |

### GDPR/CCPA Compliance

| Customer Right | Implementation |
|----------------|---------------|
| Right to opt-out | Có thể hủy auto-renewal bất cứ lúc nào |
| Right to access | Có thể xem billing history, invoices |
| Right to deletion | Có thể yêu cầu xóa tài khoản |
| Right to portability | Có thể export data |
| Consent management | 7-day advance notification trước mỗi charge |

---

## Nguồn tài liệu gốc

- `docs/02-product/ba-specs/function-renewal/base/Renewal_Touchpoint_Flow_v1.md`
- `docs/02-product/ba-specs/function-renewal/checklist-cs-md/checklist-cs-md.md`
- `docs/02-product/tomtat-chitiet/tomtat/function-renewal-tomtat.md`
