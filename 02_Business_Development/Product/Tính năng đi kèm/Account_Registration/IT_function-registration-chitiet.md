# CHI TIẾT LUỒNG NGHIỆP VỤ – FUNCTION REGISTRATION: ĐĂNG KÝ TÀI KHOẢN (ACCOUNT REGISTRATION)

## Header

- **Function:** Đăng ký tài khoản (Account Registration)
- **Version:** 3.1
- **Input:** Người dùng ≥ 18 tuổi có nhu cầu sử dụng dịch vụ y tế từ xa (chưa có tài khoản hoặc đã có tài khoản)
- **Output:** Người dùng có tài khoản active, có gói dịch vụ phù hợp, có thể truy cập và sử dụng hệ thống
- **Điều kiện sử dụng:** App chỉ dành cho người ≥ 18 tuổi. Phone là primary identifier (bắt buộc). OTP chỉ gửi qua SMS. Không hỗ trợ Minor (< 18 tuổi). HIPAA Consent bắt buộc trước khi gửi OTP.

---

## Nguyên tắc cốt lõi

- Mỗi node phải có INPUT rõ ràng, OUTPUT rõ ràng, ĐIỀU KIỆN cụ thể
- Mỗi luồng là ĐƠN TUYẾN (từ A → B → C, không có nhánh ngầm)
- Phone là bắt buộc cho mọi phương thức đăng ký
- HIPAA Consent bắt buộc trước khi gửi OTP (sau khi DOB ≥ 18 được xác nhận)

---

## Danh sách Flow

| Flow ID | Tên luồng | Mô tả ngắn |
|---------|-----------|-------------|
| FLOW-01 | Đăng ký tài khoản | Tạo tài khoản mới + chọn gói dịch vụ |
| FLOW-02 | Đăng nhập | Đăng nhập bằng Phone/Email/Google/Apple |
| FLOW-03 | Quên mật khẩu | Đặt lại mật khẩu qua OTP SMS |
| FLOW-04 | Thêm Family Member (Adult) | Primary mời Adult ≥ 18 tuổi vào gia đình |
| FLOW-05 | Chọn Member khi Login | Chọn context Dashboard khi có Family Members |
| FLOW-06 | Nâng cấp/Hạ cấp gói | Thay đổi gói dịch vụ hiện tại |
| FLOW-07 | Mua Add-on Service | Mua dịch vụ lẻ hoặc add-on subscription |

---

## FLOW-01: ĐĂNG KÝ TÀI KHOẢN

### Phần chung (bước 1-2)

#### Bước 1: Truy cập hệ thống

- **Mô tả:** Khách hàng truy cập website CVH hoặc mobile app CVH Healthcare
- **INPUT:** user_action (String: "register" | "login")
- **OUTPUT:** redirect_to (String: Màn hình tiếp theo)

**Wireframe ASCII – Homepage:**

```
┌─────────────────────────────────────────────────────────────┐
│  [Logo CVH]                    [ĐĂNG NHẬP]  [ĐĂNG KÝ]       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│           ╔═══════════════════════════════════╗             │
│           ║    CHĂM SÓC SỨC KHỎE TOÀN DIỆN    ║             │
│           ║     CHO BẠN VÀ GIA ĐÌNH           ║             │
│           ╚═══════════════════════════════════╝             │
│                                                             │
│                  ┌─────────────────────┐                    │
│                  │    BẮT ĐẦU NGAY     │  ← Primary CTA     │
│                  └─────────────────────┘                    │
│                                                             │
│    ┌───────────┐    ┌───────────┐    ┌───────────┐          │
│    │ Connect   │    │   Plus    │    │  Premium  │          │
│    │  $39/th   │    │  $99/th   │    │  $249/th  │          │
│    │           │    │           │    │           │          │
│    │ ✓ Video   │    │ ✓ Connect │    │ ✓ Plus    │          │
│    │ ✓ Chat    │    │ ✓ Lab     │    │ ✓ Family  │          │
│    │ ✓ AI      │    │ ✓ Mental  │    │ ✓ Premium │          │
│    └───────────┘    └───────────┘    └───────────┘          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Click [ĐĂNG KÝ] hoặc [BẮT ĐẦU NGAY] | Màn hình Nhập Email (Bước 2) |
| Click [ĐĂNG NHẬP] | Chuyển FLOW-02 |

---

#### Bước 2: Nhập Phone hoặc Email

- **Mô tả:** Khách hàng nhập Phone hoặc Email vào ô input duy nhất
- **INPUT:** email (String: Email user nhập)
- **OUTPUT:** email_valid (Boolean), email_exists (Boolean)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]                  ĐĂNG KÝ                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    ┌─────────────┐                          │
│                    │   [1] ─ 2 ─ 3   │  ← Progress          │
│                    └─────────────┘                          │
│                                                             │
│    Nhập email để bắt đầu                                    │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📧  Email của bạn                                   │  │
│    │     _________________________________________       │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ⚠️ Email không hợp lệ (hiện khi lỗi)                     │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                    TIẾP TỤC                         │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ─────────────────── hoặc ───────────────────             │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  [G]  Tiếp tục với Google                           │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  []  Tiếp tục với Apple                            │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    Đã có tài khoản? [Đăng nhập ngay]                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| email | Regex: `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$` | "Email không hợp lệ" |
| email | Required | "Vui lòng nhập email" |

**UI States:**

| State | Hiển thị |
|-------|----------|
| Default | Input trống, Button disabled (xám) |
| Typing | Input active (border xanh) |
| Valid | Button enabled (xanh dương) |
| Error | Border đỏ + Icon ⚠️ + Message lỗi |
| Loading | Button hiện spinner "Đang kiểm tra..." |

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Chứa "@" → Email | Nhánh B (Email) |
| 10 số → Phone | Nhánh A (Phone) |
| Nhấn "Tiếp tục với Google/Apple" | Nhánh C |
| Email/Phone đã tồn tại | Chuyển FLOW-02 (Đăng nhập) |
| Email/Phone chưa tồn tại | Chuyển màn hình Nhập thông tin |

---

### Nhánh A: Đăng ký bằng Phone

#### Bước 3A: Kiểm tra Phone

- **Mô tả:** Hệ thống kiểm tra Phone (format US: 10 số, area code 3 số + 7 số) + kiểm tra trùng
- **INPUT:** phone_number (String, required)
- **OUTPUT:** phone_valid (Boolean), phone_exists (Boolean)

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| phone_number | Format US (10 số: 3-3-4, VD: (555) 123-4567) | "Số điện thoại không hợp lệ" |
| phone_number | Chưa tồn tại trong hệ thống | "Số này đã được đăng ký. Đăng nhập?" |

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Phone đã tồn tại | Chuyển FLOW-02 (Đăng nhập) |
| Phone hợp lệ | Tiếp tục bước 4A |

---

#### Bước 4A: Nhập thông tin cơ bản

- **Mô tả:** Khách hàng nhập Mật khẩu + Xác nhận, Họ tên, Email (optional), Giới tính, Ngày sinh
- **INPUT:** email (String, readonly từ bước trước), password (String, required), confirm_pwd (String, required), full_name (String, required), gender (String: Nam/Nữ/Khác, required), phone_number (String, required), date_of_birth (Date, required)
- **OUTPUT:** all_fields_valid (Boolean), dob_valid (Boolean), otp_generated (String), otp_sent_to (Object: {sms: phone, email: email})

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]                  ĐĂNG KÝ                               │
├─────────────────────────────────────────────────────────────┤
│                    ┌─────────────┐                          │
│                    │   ✓ ─ [2] ─ 3   │  ← Progress          │
│                    └─────────────┘                          │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📧  example@email.com                    [🔒 Locked]│  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 🔐  Mật khẩu *                              [👁️]    │  │
│    │     ••••••••••                                      │  │
│    └─────────────────────────────────────────────────────┘  │
│    ✓ Tối thiểu 8 ký tự  ✓ 1 chữ hoa  ✓ 1 số               │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 🔐  Xác nhận mật khẩu *                     [👁️]    │  │
│    │     ••••••••••                                      │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 👤  Họ và tên đầy đủ *                              │  │
│    │     Nguyễn Văn A                                    │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ ⚧  Giới tính *                                      │  │
│    │     [Nam]  [Nữ]  [Khác]                             │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📱  Số điện thoại *                                 │  │
│    │     +1 │ (555) 123-4567                             │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📅  Ngày sinh *                                      │  │
│    │     15 / 03 / 1989                                  │  │
│    └─────────────────────────────────────────────────────┘  │
│    ⚠️ Bạn phải từ 18 tuổi trở lên (hiện khi < 18)          │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                    TIẾP TỤC                         │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Password Strength Indicator:**

```
[████░░░░░░] Yếu (đỏ)
[██████░░░░] Trung bình (vàng)
[██████████] Mạnh (xanh)
```

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| password | Min 8 ký tự | "Mật khẩu phải có ít nhất 8 ký tự" |
| password | Ít nhất 1 chữ hoa | "Mật khẩu phải có ít nhất 1 chữ hoa" |
| password | Ít nhất 1 số | "Mật khẩu phải có ít nhất 1 số" |
| confirm_pwd | Phải khớp password | "Mật khẩu xác nhận không khớp" |
| full_name | Min 2 ký tự | "Vui lòng nhập họ tên" |
| full_name | Không chứa số | "Họ tên không được chứa số" |
| gender | Required | "Vui lòng chọn giới tính" |
| phone_number | 10 số, format US (3-3-4) | "Số điện thoại không hợp lệ" |
| date_of_birth | Required | "Vui lòng nhập ngày sinh" |
| date_of_birth | Valid date, < today | "Ngày sinh không hợp lệ" |
| date_of_birth | Age ≥ 18 | "Ứng dụng chỉ dành cho người từ 18 tuổi trở lên" |

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| DOB < 18 | Chặn đăng ký |
| Hợp lệ | Chuyển màn hình HIPAA Consent (4.5A) |

---

#### Bước 4.5A: HIPAA Consent

- **Mô tả:** Khách hàng đồng ý điều khoản quyền riêng tư y tế HIPAA, ký tên điện tử
- **INPUT:** full_name (String, từ bước trước để verify), hipaa_consent (Boolean, checkbox bắt buộc), e_signature (String, gõ họ tên)
- **OUTPUT:** consent_accepted (Boolean), consent_timestamp (DateTime, UTC), consent_version (String: "HIPAA-v1.0")

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]           ĐỒNG Ý ĐIỀU KHOẢN Y TẾ                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                        🔒                                   │
│                                                             │
│    THÔNG BÁO VỀ QUYỀN RIÊNG TƯ Y TẾ                         │
│    (HIPAA Notice of Privacy Practices)                      │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ CVH Healthcare cam kết bảo vệ thông tin sức khỏe    │  │
│    │ của bạn theo tiêu chuẩn HIPAA.                      │  │
│    │                                                     │  │
│    │ Chúng tôi sẽ:                                       │  │
│    │ • Thu thập thông tin y tế để cung cấp dịch vụ      │  │
│    │ • Chia sẻ với bác sĩ/nhân viên y tế khi cần        │  │
│    │ • Bảo mật dữ liệu theo chuẩn mã hóa                │  │
│    │                                                     │  │
│    │ Quyền của bạn:                                      │  │
│    │ • Truy cập và xem hồ sơ y tế của bạn               │  │
│    │ • Yêu cầu chỉnh sửa thông tin không chính xác      │  │
│    │ • Thu hồi đồng ý bất cứ lúc nào                    │  │
│    │                                                     │  │
│    │ [Xem đầy đủ chính sách →]                          │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ ☐ Tôi đã đọc và đồng ý với Chính sách Quyền        │  │
│    │   riêng tư và Điều khoản HIPAA của CVH Healthcare * │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ✍️ Chữ ký điện tử *                                      │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ Nguyễn Văn A                                        │  │
│    └─────────────────────────────────────────────────────┘  │
│    (Gõ họ tên đầy đủ để xác nhận đồng ý)                    │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                 ĐỒNG Ý & TIẾP TỤC                   │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ⚠️ Bạn cần đồng ý với điều khoản HIPAA để tiếp tục       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| hipaa_consent | Bắt buộc checked | "Vui lòng đọc và đồng ý với điều khoản HIPAA" |
| e_signature | Bắt buộc | "Vui lòng ký tên để xác nhận" |
| e_signature | Khớp với full_name (case-insensitive) | "Chữ ký phải khớp với họ tên đã nhập trước đó" |

**UI States:**

| State | Hiển thị |
|-------|----------|
| Default | Checkbox unchecked, input chữ ký trống, button disabled |
| Checkbox checked | Checkbox có tick, button vẫn disabled (chờ e-signature) |
| Signature valid | Chữ ký khớp với họ tên, button enabled |
| Error | Border đỏ + message lỗi dưới field |

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| hipaa_consent = true AND e_signature khớp | Lưu consent record → Gửi OTP SMS → Màn hình OTP |
| hipaa_consent = false HOẶC e_signature sai | Hiển thị lỗi, không cho tiếp tục |
| Nhấn [←] Back | Quay lại màn hình nhập thông tin |
| Nhấn "Xem đầy đủ chính sách" | Mở popup/trang chi tiết chính sách HIPAA |

---

#### Bước 5A: Xác thực OTP

- **Mô tả:** Hệ thống gửi OTP qua SMS (chỉ SMS). Khách nhập OTP 6 số
- **INPUT:** otp_input (String, 6 số), otp_expected (String, từ bước trước), attempt_count (Number, max 3)
- **OUTPUT:** otp_verified (Boolean), attempts_remaining (Number)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              XÁC THỰC SỐ ĐIỆN THOẠI                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                        📱                                   │
│                                                             │
│    Chúng tôi đã gửi mã xác thực đến                         │
│    (555) ***-4567                                           │
│                                                             │
│    ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐    │
│    │  1  │  │  2  │  │  3  │  │  4  │  │  _  │  │  _  │    │
│    └─────┘  └─────┘  └─────┘  └─────┘  └─────┘  └─────┘    │
│                                                             │
│              Mã có hiệu lực trong: 04:32                    │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                    XÁC NHẬN                         │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    Không nhận được mã? [GỬI LẠI] (sau 30s)                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| otp_input | Đúng 6 số | "Vui lòng nhập đủ 6 số" |
| otp_input | Khớp otp_expected | "Mã xác thực không đúng" |
| otp_input | Chưa hết hạn (5 phút) | "Mã đã hết hạn, vui lòng gửi lại" |

**UI States:**

| State | Hiển thị |
|-------|----------|
| Default | 6 ô trống, countdown chạy |
| Typing | Ô đang focus có border xanh |
| OTP Đúng | Border xanh, tự động submit |
| OTP Sai | Border đỏ + shake animation + "Mã không đúng. Còn X lần thử" |
| Hết hạn | Countdown = 00:00 + "Mã đã hết hạn" + Button GỬI LẠI active |
| Blocked | "Bạn đã nhập sai 3 lần. Vui lòng thử lại sau 15 phút" |

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| OTP đúng | Chuyển bước 6 (Tạo tài khoản) |
| OTP sai, còn lượt | Cho nhập lại, hiện số lần còn lại |
| OTP sai 3 lần | Block 15 phút |

⚠️ OTP hết hạn 5 phút → hiển thị nút "Gửi lại OTP"

---

### Nhánh B: Đăng ký bằng Email

#### Bước 3B: Kiểm tra Email

- **Mô tả:** Hệ thống kiểm tra Email format + kiểm tra trùng
- **INPUT:** email (String, required)
- **OUTPUT:** email_valid (Boolean), email_exists (Boolean)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Email đã tồn tại | Chuyển FLOW-02 |
| Email hợp lệ | Tiếp tục bước 4B |

#### Bước 4B: Nhập thông tin

- **Mô tả:** Nhập Mật khẩu + Xác nhận, Họ tên, Số điện thoại (bắt buộc), Giới tính, Ngày sinh (bắt buộc, validate ≥ 18)
- **INPUT:** Giống bước 4A
- **OUTPUT:** Giống bước 4A

**Validation Rules:** Giống bước 4A

#### Bước 4.5B: HIPAA Consent

- Giống bước 4.5A

#### Bước 5B: Xác thực OTP

- **Mô tả:** Gửi OTP qua SMS (chỉ SMS, không gửi Email). Logic giống 5A

---

### Nhánh C: Đăng ký bằng Google/Apple

#### Bước 3C: Xác thực Google/Apple

- **Mô tả:** Popup xác thực → Hệ thống nhận Email + Họ tên
- **INPUT:** oauth_token (String)
- **OUTPUT:** email (String), full_name (String)

#### Bước 4C: Kiểm tra email

- ⛔ Email đã tồn tại → chuyển FLOW-02 | Chưa tồn tại → tiếp tục

#### Bước 5C: Bổ sung thông tin

- **Mô tả:** Form: Email + Họ tên (readonly). Nhập: Số điện thoại (bắt buộc), Giới tính, Ngày sinh (bắt buộc, validate ≥ 18)
- **INPUT:** email (String, readonly), full_name (String, readonly), phone_number (String, required), gender (String: Nam/Nữ/Khác, required), date_of_birth (Date, required)
- **OUTPUT:** otp_sent (Boolean), dob_valid (Boolean)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              HOÀN TẤT ĐĂNG KÝ                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    ┌─────────┐                              │
│                    │  [G]    │  Đăng ký với Google          │
│                    └─────────┘                              │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📧  Email                                [🔒 Locked]│  │
│    │     nguyen.van.a@gmail.com                          │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 👤  Họ và tên                            [🔒 Locked]│  │
│    │     Nguyễn Văn A                                    │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📱  Số điện thoại *                                 │  │
│    │     +1 │ (___) ___-____                             │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ ⚧  Giới tính *                                      │  │
│    │     [Nam]  [Nữ]  [Khác]                             │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📅  Ngày sinh *                                      │  │
│    │     DD / MM / YYYY                                  │  │
│    └─────────────────────────────────────────────────────┘  │
│    ⚠️ Bạn phải từ 18 tuổi trở lên (hiện khi < 18)          │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                    TIẾP TỤC                         │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| phone_number | Required | "Vui lòng nhập số điện thoại" |
| phone_number | Format US (10 số: 3-3-4) | "Số điện thoại không hợp lệ" |
| phone_number | Chưa tồn tại trong hệ thống | "Số điện thoại đã được sử dụng" |
| gender | Required | "Vui lòng chọn giới tính" |
| date_of_birth | Required | "Vui lòng nhập ngày sinh" |
| date_of_birth | Valid date, < today | "Ngày sinh không hợp lệ" |
| date_of_birth | Age ≥ 18 | "Ứng dụng chỉ dành cho người từ 18 tuổi trở lên" |

#### Bước 5.5C: HIPAA Consent – Giống 4.5A

#### Bước 6C: Gửi OTP SMS – Logic giống 5A

---

### Phần chung tiếp theo (sau xác thực OTP)

#### Bước 6: Tạo tài khoản

- **Mô tả:** Hệ thống tạo mã bệnh nhân (CVH-YYYYMMDD-XXXXX), lưu thông tin, tạo hồ sơ y tế (trống)
- **INPUT:** all validated fields từ các bước trước
- **OUTPUT:** patient_id (String: CVH-YYYYMMDD-XXXXX), account_status (String: "active")
- Nhánh C: tài khoản liên kết Google/Apple, không có mật khẩu

---

#### Bước 7: Chọn hình thức sử dụng

- **Mô tả:** Khách chọn "DÙNG THỬ MIỄN PHÍ" hoặc "MUA NGAY"
- **INPUT:** user_choice (String: "trial" | "purchase")
- **OUTPUT:** redirect_to (String)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              CHỌN HÌNH THỨC SỬ DỤNG                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│           ✅ Tài khoản đã được tạo thành công!              │
│                                                             │
│    Chào mừng, Nguyễn Văn A                                  │
│    Hãy chọn cách bạn muốn bắt đầu:                          │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  🎁  DÙNG THỬ MIỄN PHÍ                              │  │
│    │                                                     │  │
│    │  • 14 ngày trải nghiệm đầy đủ                       │  │
│    │  • Gói Care Connect ($39/tháng)                     │  │
│    │  • Không cần thẻ tín dụng                           │  │
│    │  • Tự động hết hạn, không tự gia hạn                │  │
│    │                                                     │  │
│    │              [BẮT ĐẦU DÙNG THỬ]                     │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  💳  MUA NGAY                                       │  │
│    │                                                     │  │
│    │  • Chọn gói phù hợp với nhu cầu                     │  │
│    │  • Care Connect / Plus / Premium                    │  │
│    │  • Thêm dịch vụ lẻ khi cần                          │  │
│    │                                                     │  │
│    │              [XEM CÁC GÓI DỊCH VỤ]                  │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| user_choice = "trial" | Kích hoạt Trial → Dashboard |
| user_choice = "purchase" | Chuyển màn hình Chọn sản phẩm |

---

### Nhánh Trial: Dùng thử miễn phí

#### Bước 8: Kích hoạt Trial

- **Mô tả:** Hệ thống gán gói Care Connect, thời hạn 14 ngày, lên lịch email nhắc nhở (ngày 7, 11, 13, 14)
- **INPUT:** patient_id (String)
- **OUTPUT:** package (String: "Care Connect"), trial_dates (Object: {start, end}), status (String: "trial")
- → Chuyển bước 12

---

### Nhánh Mua ngay: Thanh toán

#### Bước 9: Chọn sản phẩm

- **Mô tả:** Chọn gói Connect $39 / Plus $99 / Premium $149 hoặc dịch vụ lẻ
- **INPUT:** selected_package (String: "connect" | "plus" | "premium"), selected_addons (Array<String>)
- **OUTPUT:** cart_items (Array), total_monthly (Number), total_onetime (Number)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]                CHỌN GÓI DỊCH VỤ                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  GÓI SUBSCRIPTION                                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ○  Care Connect                            $39/tháng│    │
│  │    Video visit, Chat, AI Health Assistant           │    │
│  ├─────────────────────────────────────────────────────┤    │
│  │ ●  Care Plus                    ⭐ PHỔ BIẾN  $99/tháng│   │
│  │    Connect + Lab + Mental Health                    │    │
│  ├─────────────────────────────────────────────────────┤    │
│  │ ○  Care Premium                           $249/tháng│    │
│  │    Plus + Family + Premium Support                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  DỊCH VỤ LẺ (tùy chọn)                                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ☐  Video Visit (1 lần)                         $45  │    │
│  │ ☐  Lab Consultation                            $35  │    │
│  │ ☐  Mental Health Session                       $75  │    │
│  │ ☐  Second Opinion                             $150  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  Tổng:                                           $99/tháng  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    TIẾP TỤC                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| selected_package | Bắt buộc chọn 1 | "Vui lòng chọn gói dịch vụ" |

---

#### Bước 10: Xem tóm tắt đơn hàng

- **Mô tả:** Chi tiết sản phẩm, Subtotal, Tax, Total, ngày thanh toán tiếp theo
- **INPUT:** cart_items (Array), promo_code (String, optional)
- **OUTPUT:** subtotal (Number), tax (Number), total (Number), next_billing_date (Date)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              TÓM TẮT ĐƠN HÀNG                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Chi tiết đơn hàng                                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Care Plus (Monthly)                          $99.00 │    │
│  │   └─ Bao gồm: Video, Chat, Lab, Mental              │    │
│  ├─────────────────────────────────────────────────────┤    │
│  │ Video Visit (1 lần)                          $45.00 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  Subtotal                                        $144.00    │
│  Tax (10%)                                        $14.40    │
│  ─────────────────────────────────────────────────────────  │
│  TỔNG CỘNG                                       $158.40    │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 🎟️  Mã giảm giá                        [ÁP DỤNG]    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  📅 Ngày thanh toán tiếp theo: 26/01/2025                   │
│  🔄 Tự động gia hạn hàng tháng                              │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              TIẾN HÀNH THANH TOÁN                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

#### Bước 11: Thanh toán

- **Mô tả:** Thanh toán qua Payment Gateway (giao diện bên thứ 3)
- **INPUT:** card_info (Object, từ Payment Gateway)
- **OUTPUT:** payment_status (String), transaction_id (String)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Thành công | Kích hoạt gói → bước 12 |
| Thất bại | Cho phép thử lại |

⚠️ Payment Gateway sập > 10 phút → CS báo kỹ thuật

---

### Hoàn tất đăng ký (chung)

#### Bước 12: Thiết lập đăng nhập nhanh

- **Mô tả:** Nếu thiết bị hỗ trợ sinh trắc học → popup hỏi bật Face ID/Touch ID
- **INPUT:** enable_biometric (Boolean)
- **OUTPUT:** biometric_enabled (Boolean)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                         ┌─────┐                             │
│                         │ 😊  │                             │
│                         └─────┘                             │
│                                                             │
│              Bật đăng nhập bằng Face ID?                    │
│                                                             │
│    Sử dụng Face ID để đăng nhập nhanh chóng và             │
│    an toàn hơn mà không cần nhập mật khẩu.                  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                  BẬT FACE ID                        │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│                    [Để sau]                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Điều kiện hiển thị:**

| Thiết bị | Hiển thị |
|----------|----------|
| iOS (có Face ID) | Hiện "Bật Face ID?" |
| iOS (có Touch ID) | Hiện "Bật Touch ID?" |
| Android (có Fingerprint) | Hiện "Bật vân tay?" |
| Desktop | KHÔNG hiển thị màn này |

---

#### Bước 13: Gửi email chào mừng

- **Mô tả:** Hệ thống gửi email chào mừng: thông tin tài khoản, thông tin gói Trial/Paid, link bắt đầu sử dụng

#### Bước 14: Vào Dashboard

- **Mô tả:** Redirect vào Dashboard. **Hoàn tất FLOW-01.**

**Wireframe ASCII – Dashboard:**

```
┌─────────────────────────────────────────────────────────────┐
│  [≡]        CVH Healthcare              [🔔] [👤 Nguyễn A] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Xin chào, Nguyễn Văn A! 👋                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 🎁 Trial còn 14 ngày              [NÂNG CẤP NGAY]   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌─────────┐  │
│  │ 📹        │  │ 💬        │  │ 🧪        │  │ 🧠      │  │
│  │ Video     │  │ Chat      │  │ Lab       │  │ Mental  │  │
│  │ Visit     │  │ Bác sĩ    │  │ Results   │  │ Health  │  │
│  └───────────┘  └───────────┘  └───────────┘  └─────────┘  │
│                                                             │
│  Hoạt động gần đây                                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 📋 Chưa có hoạt động nào                            │    │
│  │    Hãy bắt đầu với một cuộc tư vấn video!           │    │
│  │           [ĐẶT LỊCH TƯ VẤN ĐẦU TIÊN]                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  Hoàn thiện hồ sơ (30%)                                     │
│  [████████░░░░░░░░░░░░░░░░░░░░]                              │
│  [HOÀN THIỆN NGAY]                                          │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  [🏠 Home]   [📅 Lịch]   [💬 Chat]   [📋 Hồ sơ]   [⚙️]     │
└─────────────────────────────────────────────────────────────┘
```

**Hiển thị theo trạng thái:**

| Trạng thái | Banner hiển thị |
|------------|------------------|
| Trial | "🎁 Trial còn X ngày [NÂNG CẤP NGAY]" |
| Paid - Active | Không hiện banner |
| Trial sắp hết (≤3 ngày) | "⚠️ Trial sắp hết! [NÂNG CẤP NGAY]" (màu cam) |

---

## FLOW-02: ĐĂNG NHẬP

### Nhánh A: Đăng nhập bằng Phone/Email + Password

#### Bước 1: Nhập Phone hoặc Email

- **Mô tả:** Khách hàng nhập Phone hoặc Email vào ô input duy nhất
- **INPUT:** email (String, required)
- **OUTPUT:** login_success (Boolean), session_token (String), has_family_members (Boolean)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]                  ĐĂNG NHẬP                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    ┌─────────┐                              │
│                    │  [CVH]  │                              │
│                    └─────────┘                              │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📧  Email                                           │  │
│    │     example@email.com                               │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 🔐  Mật khẩu                                 [👁️]   │  │
│    │     ••••••••                                        │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ☐ Ghi nhớ đăng nhập                                      │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                    ĐĂNG NHẬP                        │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│                  [Quên mật khẩu?]                           │
│                                                             │
│    ─────────────────── hoặc ───────────────────             │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  [G]  Tiếp tục với Google                           │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  []  Tiếp tục với Apple                            │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 😊  Đăng nhập bằng Face ID                          │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    Chưa có tài khoản? [Đăng ký ngay]                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| email | Required | "Vui lòng nhập email" |
| password | Required | "Vui lòng nhập mật khẩu" |
| password | Max 5 lần sai | "Tài khoản bị khóa 30 phút" |

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Không tìm thấy tài khoản | Chuyển FLOW-01 (Đăng ký) |
| Tìm thấy | Tiếp tục bước 2 |

---

#### Bước 2: Nhập mật khẩu

- **Mô tả:** Nhập mật khẩu. Options: "Ghi nhớ đăng nhập", "Quên mật khẩu?" → FLOW-03
- **INPUT:** password (String, required), remember_me (Boolean, optional)
- **OUTPUT:** login_success (Boolean)

#### Bước 3: Xác thực

- **Mô tả:** Hệ thống kiểm tra mật khẩu + trạng thái tài khoản
- **INPUT:** email (String), password (String)
- **OUTPUT:** login_success (Boolean), session_token (String)

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Đúng | Bước 4 |
| Sai tối đa 5 lần | Khóa tài khoản 30 phút |
| Tài khoản không có mật khẩu | Hiển thị: "Vui lòng đăng nhập bằng Google/Apple hoặc [Đặt mật khẩu]" |

---

### Nhánh B: Đăng nhập bằng Google/Apple

#### Bước 1B: Chọn Google/Apple

- **Mô tả:** Nhấn "Tiếp tục với Google" hoặc "Tiếp tục với Apple"

#### Bước 2B: Xác thực OAuth

- **Mô tả:** Popup xác thực → Hệ thống nhận Email

#### Bước 3B: Kiểm tra email

- ⛔ Email chưa tồn tại → chuyển FLOW-01 | Email đã tồn tại → tự động liên kết (nếu chưa) → bước 4

---

### Phần chung (sau xác thực)

#### Bước 4: Xác thực sinh trắc học (nếu đã bật)

- **Mô tả:** Nếu đã bật Face ID/Touch ID → xác thực sinh trắc học
- ⛔ Thành công → Dashboard | Thất bại → yêu cầu nhập mật khẩu

#### Bước 5: Đăng nhập thành công

- **Mô tả:** Đăng nhập thành công → Dashboard
- ⛔ Nếu có Family Members → hiển thị màn chọn member (FLOW-05)

### Xử lý đặc biệt

- Tài khoản không có mật khẩu + đăng nhập bằng Email → Hiển thị: "Vui lòng đăng nhập bằng Google/Apple hoặc [Đặt mật khẩu]"
- Google/Apple + email đã tồn tại + chưa liên kết → Tự động liên kết → Dashboard

---

## FLOW-03: QUÊN MẬT KHẨU

#### Bước 1: Nhấn "Quên mật khẩu?"

- **Mô tả:** Khách nhấn "Quên mật khẩu?" tại màn đăng nhập

#### Bước 2: Nhập Phone hoặc Email

- **Mô tả:** Nhập Phone hoặc Email đã đăng ký
- **INPUT:** email (String, required)
- **OUTPUT:** otp_sent (Boolean – luôn true vì bảo mật)
- Lưu ý bảo mật: luôn hiển thị "Đã gửi mã" dù tài khoản có tồn tại hay không

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              QUÊN MẬT KHẨU                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                        🔐                                   │
│                                                             │
│    Nhập email đã đăng ký để nhận mã xác thực                │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📧  Email                                           │  │
│    │     _________________________________________       │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                    GỬI MÃ XÁC THỰC                  │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    Nhớ mật khẩu? [Đăng nhập]                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

#### Bước 3: Gửi OTP qua SMS

- **Mô tả:** Hệ thống gửi OTP chỉ qua SMS đến SĐT đã đăng ký
- Nếu input là Phone → gửi SMS đến Phone đó
- Nếu input là Email → tìm Phone của account → gửi SMS đến Phone đó
- Hiển thị: "Mã xác thực đã được gửi đến số (555) ***-4567"

#### Bước 4: Nhập OTP

- **Mô tả:** Nhập OTP 6 số. Hiệu lực 5 phút. Tối đa 3 lần sai → khóa 30 phút
- **INPUT:** otp_input (String, 6 số)
- **OUTPUT:** otp_verified (Boolean)

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| otp_input | Đúng 6 số | "Vui lòng nhập đủ 6 số" |
| otp_input | Khớp OTP đã gửi | "Mã xác thực không đúng" |
| otp_input | Chưa hết hạn 5 phút | "Mã đã hết hạn" |

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| OTP đúng | Chuyển bước 5 |
| OTP sai 3 lần | Khóa 30 phút, hiển thị đếm ngược |

---

#### Bước 5: Tạo mật khẩu mới

- **Mô tả:** Nhập mật khẩu mới (min 8, 1 chữ hoa, 1 số) + xác nhận
- **INPUT:** new_password (String, required), confirm_password (String, required)
- **OUTPUT:** password_updated (Boolean)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              ĐẶT MẬT KHẨU MỚI                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                        🔑                                   │
│                                                             │
│    Tạo mật khẩu mới cho tài khoản của bạn                   │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 🔐  Mật khẩu mới *                           [👁️]   │  │
│    │     ••••••••                                        │  │
│    └─────────────────────────────────────────────────────┘  │
│    ✓ Tối thiểu 8 ký tự  ✓ 1 chữ hoa  ✗ 1 số                │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 🔐  Xác nhận mật khẩu mới *                  [👁️]   │  │
│    │     ••••••••                                        │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                  ĐẶT MẬT KHẨU MỚI                   │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Validation Rules:** Giống validation mật khẩu khi đăng ký (min 8, 1 upper, 1 num)

---

#### Bước 6: Cập nhật mật khẩu

- **Mô tả:** Lưu mật khẩu mới (hashed), đăng xuất tất cả thiết bị, gửi SMS thông báo

#### Bước 7: Tự động đăng nhập

- **Mô tả:** Tự động đăng nhập → Dashboard

### Xử lý đặc biệt

- Tài khoản không tồn tại → vẫn hiển thị "Đã gửi mã" (bảo mật) nhưng không gửi SMS
- Tài khoản Google/Apple → hiển thị: "Tài khoản sử dụng Google/Apple. Vui lòng đăng nhập bằng Google/Apple"
- OTP sai 3 lần → khóa 30 phút, hiển thị thời gian đếm ngược

---

## FLOW-04: THÊM FAMILY MEMBER – ADULT (≥ 18 TUỔI) – INVITATION MODEL

### Nguyên tắc

- Invitation via Link: gửi LINK WEB trong SMS/Email (không gửi OTP trực tiếp)
- Pre-paid by Primary: Primary chọn gói + thanh toán TRƯỚC khi gửi lời mời
- Adult chủ động: Adult click link → Xác thực OTP → Tạo tài khoản → Dashboard
- Data Separation (HIPAA): Primary KHÔNG THỂ xem dữ liệu y tế của Adult trừ khi Adult share quyền

### Luồng Primary gửi lời mời

#### Bước 1: Vào Quản lý gia đình

- **Mô tả:** Primary vào Dashboard → Cài đặt → Quản lý gia đình

**Wireframe ASCII – Quản lý gia đình:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              QUẢN LÝ GIA ĐÌNH                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  THÀNH VIÊN (3)                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 👤 Nguyễn Văn Hùng (Tôi)           [Chủ tài khoản]  │    │
│  │    Premium • $149/tháng                         ⚙️  │    │
│  ├─────────────────────────────────────────────────────┤    │
│  │ 👩 Vợ Mai (34 tuổi)                    [Vợ/Chồng]   │    │
│  │    Plus • $99/tháng                     🔒      ⚙️  │    │
│  │    Tài khoản riêng biệt                             │    │
│  ├─────────────────────────────────────────────────────┤    │
│  │ 👨 Con Cường (24 tuổi)         [Con trưởng thành]   │    │
│  │    Connect • $39/tháng                  🔒      ⚙️  │    │
│  │    Tài khoản riêng biệt                             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  LỜI MỜI CHỜ (1)                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 📱 (555) ***-4567                    ⏳ Chờ xác nhận │    │
│  │    Cha/Mẹ • Plus $99/th • Còn 6 ngày [GỬI LẠI] [HỦY]│    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │           + MỜI THÀNH VIÊN MỚI                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ═══════════════════════════════════════════════════════    │
│  💳 THANH TOÁN (Linked Billing)                             │
│  • Hùng (Premium)                              $149         │
│  • Vợ Mai (Plus)                                $99         │
│  • Con Cường (Connect)                          $39         │
│  ─────────────────────────────────────────────────────────  │
│  TỔNG HÀNG THÁNG                               $287         │
│  Phương thức: Visa ****1234                    [Thay đổi]   │
│  Ngày gia hạn: 01/01/2026                                   │
│                                                             │
│  ⚠️ Primary KHÔNG THỂ xem hồ sơ y tế của Adult             │
│     (Data Separation theo chuẩn HIPAA)                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Icon giải thích:**

| Icon | Ý nghĩa |
|------|---------|
| ⚙️ | Menu actions (Xem billing, Hủy liên kết) |
| 🔒 | Adult có tài khoản riêng biệt (Data Separation) |
| ⏳ | Đang chờ Adult xác nhận lời mời |

---

#### Bước 2: Nhấn "+ MỜI THÀNH VIÊN MỚI"

#### Bước 3: Nhập thông tin Adult

- **Mô tả:** Nhập Họ tên, Phone (tùy chọn), Email (tùy chọn) — phải nhập ít nhất 1, Ngày sinh (≥ 18), Quan hệ
- **INPUT:** full_name (String, required), phone_number (String, optional - ít nhất 1 trong 2), email (String, optional - ít nhất 1 trong 2), date_of_birth (Date, required, ≥ 18), relationship (String: Vợ/Chồng/Con trưởng thành/Cha/Mẹ/Khác, required)
- **OUTPUT:** validation_passed (Boolean)

**Wireframe ASCII – Form nhập thông tin:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              THÊM NGƯỜI LỚN                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 👤  Họ và tên *                                     │  │
│    │     Trần Thị Mai                                    │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📱  Số điện thoại (tùy chọn)                        │  │
│    │     +1 (555) 123-4567                               │  │
│    └─────────────────────────────────────────────────────┘  │
│    💡 Nếu có: gửi OTP trực tiếp                             │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📧  Email (tùy chọn)                                │  │
│    │     mai@email.com                                   │  │
│    └─────────────────────────────────────────────────────┘  │
│    💡 Dùng để nhận thông báo                                │
│                                                             │
│    ⚠️ Phải nhập ít nhất 1 trong 2 (Phone hoặc Email)        │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 📅  Ngày sinh *                                     │  │
│    │     15 / 03 / 1990                   [📅 Chọn ngày] │  │
│    └─────────────────────────────────────────────────────┘  │
│    ✓ 34 tuổi - Đủ điều kiện                                 │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ 👨‍👩‍👧  Quan hệ *                           [▼]        │  │
│    │     Vợ/Chồng                                        │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │             TIẾP TỤC - CHỌN GÓI                     │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Validation Rules:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| full_name | Required, min 2 ký tự | "Vui lòng nhập họ tên" |
| phone_number | Ít nhất 1 trong 2 (phone/email) | "Vui lòng nhập số điện thoại hoặc email" |
| phone_number | Nếu có: format US (10 số: 3-3-4) | "Số điện thoại không hợp lệ" |
| phone_number | Nếu có: chưa tồn tại trong hệ thống | "Số điện thoại đã được sử dụng" |
| email | Ít nhất 1 trong 2 (phone/email) | "Vui lòng nhập số điện thoại hoặc email" |
| email | Nếu có: format hợp lệ | "Email không hợp lệ" |
| email | Nếu có: chưa tồn tại | "Email đã được sử dụng" |
| date_of_birth | Required | "Vui lòng chọn ngày sinh" |
| date_of_birth | ≥ 18 tuổi | "Ứng dụng chỉ dành cho người từ 18 tuổi trở lên" |
| relationship | Required | "Vui lòng chọn quan hệ" |

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Phone/Email đã tồn tại | Báo lỗi |
| DOB < 18 | "Ứng dụng chỉ dành cho người từ 18 tuổi trở lên" |
| Hợp lệ | Chuyển chọn gói |

---

#### Bước 4: Chọn gói cho Adult

- **Mô tả:** Chọn gói Connect $39 / Plus $99 / Premium $149. Hiển thị tác động hóa đơn
- **INPUT:** selected_package (String: "connect" | "plus" | "premium", required)
- **OUTPUT:** billing_impact (Object: {current_total, new_total, difference})

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]          CHỌN GÓI CHO TRẦN THỊ MAI                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ ○  Connect                               $39/tháng  │  │
│    │    • 2 lượt khám video                              │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ ●  Plus                    ⭐ Phổ biến   $99/tháng  │  │
│    │    • 5 lượt khám video                              │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ ○  Premium                              $149/tháng  │  │
│    │    • Không giới hạn                                 │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    📋 Tác động hóa đơn:                                     │
│    Hiện tại: $149/tháng                                     │
│    Sau thêm: $248/tháng (+$99)                              │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │           THANH TOÁN VÀ GỬI LỜI MỜI                 │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 5: Thanh toán gói cho Adult

- **Mô tả:** Primary thanh toán TRƯỚC khi gửi lời mời → Payment Gateway

#### Bước 6: Tạo Invitation record

- **Mô tả:** Hệ thống tạo Invitation
- **OUTPUT:** invitation_id (UUID), invitation_token (String, unique), invitation_link (String: https://cvh.app/invite/{token}), status: "pending", hết hạn 7 ngày

#### Bước 7: Gửi LINK qua SMS/Email

- **Mô tả:** Gửi LINK qua SMS (nếu có Phone) và/hoặc Email (nếu có Email)

#### Bước 8: Hiển thị xác nhận

- **Mô tả:** Hiển thị trạng thái "Chờ chấp nhận", hết hạn 7 ngày. Primary có thể [Gửi lại] hoặc [Hủy lời mời]

---

### Luồng Adult chấp nhận – Nhánh A: Invitation CÓ Phone

#### Bước 1: Adult click link từ SMS/Email

#### Bước 2: Validate token

- **Mô tả:** Hệ thống validate token (hợp lệ + chưa hết hạn)
- ⛔ Token không hợp lệ/hết hạn → "Link đã hết hạn hoặc không hợp lệ"

#### Bước 3: Gửi OTP SMS tự động

- **Mô tả:** Tự động gửi OTP SMS đến số trong invitation

#### Bước 4: Nhập OTP 6 số

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]            XÁC THỰC TÀI KHOẢN                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    Xin chào Trần Thị Mai!                                   │
│                                                             │
│    Bạn được Nguyễn Văn Hùng mời                             │
│    tham gia CVH với gói Plus.                               │
│                                                             │
│    Chúng tôi đã gửi mã xác thực                             │
│    đến số (555) ***-4567                                     │
│                                                             │
│    🔢 Nhập mã OTP (6 số)                                    │
│    ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐                     │
│    │   │ │   │ │   │ │   │ │   │ │   │                     │
│    └───┘ └───┘ └───┘ └───┘ └───┘ └───┘                     │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │                    XÁC NHẬN                         │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    Chưa nhận được mã? [Gửi lại]                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 5: Tạo Password

- **Mô tả:** Họ tên + Email (readonly từ invitation), Mật khẩu (min 8, 1 chữ hoa, 1 số)

#### Bước 6: Consent

- **Mô tả:** Đồng ý liên kết gia đình, xác nhận gói (đã được thanh toán), hồ sơ y tế RIÊNG TƯ và TÁCH BIỆT, e-signature

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]            ĐỒNG Ý ĐIỀU KHOẢN                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    📋 THỎA THUẬN THAM GIA                                   │
│    Bạn sẽ được liên kết với tài khoản                       │
│    gia đình của: 👤 Nguyễn Văn Hùng                         │
│                                                             │
│    📦 Gói dịch vụ: Plus ($99/tháng)                         │
│    💳 Thanh toán bởi: Nguyễn Văn Hùng                       │
│                                                             │
│    ⚠️ QUAN TRỌNG:                                           │
│    • Hồ sơ y tế của bạn là RIÊNG TƯ                         │
│      và HOÀN TOÀN TÁCH BIỆT                                 │
│    • Nguyễn Văn Hùng KHÔNG THỂ xem                          │
│      dữ liệu sức khỏe của bạn                               │
│                                                             │
│    ☐ Tôi đã đọc và đồng ý với Điều khoản sử dụng           │
│                                                             │
│    ✍️ Ký tên điện tử:                                       │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  [Vẽ chữ ký hoặc gõ tên]                            │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              XÁC NHẬN VÀ HOÀN TẤT                   │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 7: Tạo tài khoản Adult

- **Mô tả:** Tạo tài khoản Adult (active), gán gói, liên kết billing với Primary, cập nhật invitation = "accepted"

#### Bước 8: Redirect vào Dashboard

#### Bước 9: Face ID/Touch ID (tùy chọn)

#### Bước 10: Chuyển đến FUNCTION ONBOARDING (Onboarding)

- HIPAA Consent → Tutorial → Medical Profile → AI Analysis → READY TO USE

---

### Luồng Adult chấp nhận – Nhánh B: Invitation CHỈ có Email

- Bước 1-2: giống Nhánh A
- **Bước 3:** Adult nhập số điện thoại (form bổ sung)
- **Bước 4:** Hệ thống gửi OTP SMS đến số vừa nhập
- **Bước 5:** Nhập OTP
- Bước 6-10: giống Nhánh A

### Trạng thái Invitation

| Trạng thái | Mô tả | Hiển thị |
|------------|-------|----------|
| pending | Đã gửi link, chờ Adult click | "Chờ chấp nhận" + [Gửi lại] |
| accepted | Adult đã tạo tài khoản | "Đã tham gia" |
| expired | Quá 7 ngày, chưa accept | "Hết hạn" + [Gửi lại lời mời] |
| cancelled | Primary hủy (hoàn tiền theo policy) | Không hiển thị |

---

## FLOW-05: CHỌN MEMBER KHI LOGIN (CÓ HIPAA SHARE)

#### Bước 1: Kiểm tra family members

- **Mô tả:** Sau xác thực, hệ thống kiểm tra KH có family members (Adult) không

#### Bước 2: Hiển thị danh sách

- **Mô tả:** Nếu có ≥ 1 Adult member (đã accept) → hiển thị danh sách
- **INPUT:** selected_member_id (String)
- **OUTPUT:** dashboard_context (Object), view_mode (String: "full" | "view_only")

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│                   CHỌN HỒ SƠ                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    Xin chào, Nguyễn Văn A!                                  │
│    Tài khoản gia đình của bạn                               │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  ┌─────┐                                            │  │
│    │  │ 👤  │  Chính tôi (Nguyễn Văn A)                  │  │
│    │  └─────┘  Premium                         [CHỌN →]  │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    THÀNH VIÊN GIA ĐÌNH (2)                                  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  ┌─────┐                                    🔗      │  │  ← ĐÃ SHARE
│    │  │ 👩  │  Vợ Mai (34 tuổi)                          │  │
│    │  └─────┘  Plus • Đã chia sẻ quyền xem     [CHỌN →]  │  │  ← CÓ THỂ CHỌN
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  ┌─────┐                                    🔒      │  │  ← CHƯA SHARE
│    │  │ 👨  │  Con Cường (24 tuổi)                       │  │
│    │  └─────┘  Connect • Chưa chia sẻ quyền             │  │  ← KHÔNG THỂ CHỌN
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ⓘ Để xem hồ sơ thành viên, họ cần bật chia sẻ          │
│       từ Settings của họ (theo chuẩn HIPAA)                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Hiển thị member:**

| Loại | Hiển thị | Icon | Có thể chọn |
|------|----------|------|-------------|
| Primary | Luôn hiện | 👤 | ✅ Có |
| Adult (đã share) | Hiện + "Đã chia sẻ quyền xem" | 🔗 | ✅ Có |
| Adult (chưa share) | Hiện + "Chưa chia sẻ quyền" | 🔒 | ❌ Không |

#### Bước 3: Chọn member

- ⛔ "Chính tôi" → Dashboard Primary | Adult (đã share) → Dashboard Adult (View-only mode)

#### Bước 4: Dashboard load đúng context

- Nếu xem Adult → gửi Push notification cho Adult: "[Tên Primary] đang xem hồ sơ của bạn"

### View-only Mode (Primary xem Adult)

- ✅ Cho phép: Xem lịch sử khám, kết quả xét nghiệm, đơn thuốc, đổi gói/mua add-on
- ❌ Không cho phép: Đặt lịch khám, chat với bác sĩ, sửa hồ sơ y tế, tải/export dữ liệu

---

### Adult cấp quyền xem (HIPAA Authorization)

#### Bước 1-3: Adult vào Settings → Quyền riêng tư → Bật toggle

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              QUYỀN RIÊNG TƯ                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  CHIA SẺ HỒ SƠ SỨC KHỎE                                     │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  👤 Nguyễn Văn A (Primary)                          │    │
│  │                                                     │    │
│  │  Cho phép xem hồ sơ sức khỏe:                       │    │
│  │  ┌────────────────────────────────────────┐         │    │
│  │  │  🔘 ON ────────────────────────── OFF  │         │    │
│  │  └────────────────────────────────────────┘         │    │
│  │                                                     │    │
│  │  ⓘ Khi bật, Nguyễn Văn A có thể xem:               │    │
│  │    • Lịch sử khám bệnh                             │    │
│  │    • Kết quả xét nghiệm                            │    │
│  │    • Đơn thuốc                                     │    │
│  │                                                     │    │
│  │  ⚠️ Bạn có thể tắt bất cứ lúc nào                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  LỊCH SỬ TRUY CẬP                                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  👁️ Nguyễn Văn A đã xem hồ sơ - Hôm nay 14:30      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 4: HIPAA Authorization modal

- **INPUT:** consent_checkbox (Boolean), e_signature (String)
- **OUTPUT:** authorization_id (UUID)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│                                                        [X]  │
│           📋 ỦY QUYỀN XEM HỒ SƠ SỨC KHỎE                   │
│              (HIPAA Authorization)                          │
│                                                             │
│  Bạn đang cho phép Nguyễn Văn A xem:                        │
│  ✓ Thông tin sức khỏe cá nhân                              │
│  ✓ Lịch sử khám bệnh                                       │
│  ✓ Kết quả xét nghiệm                                      │
│  ✓ Đơn thuốc đã kê                                         │
│                                                             │
│  QUYỀN CỦA BẠN:                                             │
│  • Thu hồi quyền xem bất cứ lúc nào                        │
│  • Được thông báo khi Primary xem hồ sơ                    │
│                                                             │
│  ☐ Tôi đã đọc và đồng ý với các điều khoản trên            │
│                                                             │
│  Ký tên điện tử:                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  ✍️ _________________________________                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              XÁC NHẬN ỦY QUYỀN                       │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 5-6: Ký tên + Lưu share_access = TRUE

- Lưu share_access = TRUE + timestamp + e-signature
- Push notification cho Primary

---

### Adult thu hồi quyền xem (Revoke Access)

- **Bước 1-2:** Settings → "Quyền riêng tư" → Tắt toggle Share
- **Bước 3:** Modal xác nhận thu hồi
- **Bước 4:** Cập nhật share_access = FALSE. Push notification cho Primary
- Thu hồi có hiệu lực ngay lập tức. Adult có thể bật lại (cần ký HIPAA Authorization mới)

---

## FLOW-06: NÂNG CẤP / HẠ CẤP GÓI DỊCH VỤ

**Điều kiện:** Subscription phải ở trạng thái ACTIVE. Nếu Past Due/Restricted → phải thanh toán trước.

### Nhánh A: Upgrade (Nâng cấp)

#### Bước 1: Vào Quản lý gói

- **Mô tả:** Dashboard → Settings → "Quản lý gói dịch vụ" → "Nâng cấp gói"
- **INPUT:** action (String: "upgrade")
- **OUTPUT:** redirect_to (String)

**Wireframe ASCII – Quản lý gói:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              QUẢN LÝ GÓI DỊCH VỤ                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  GÓI HIỆN TẠI                                       │  │
│    │  ┌─────┐                                            │  │
│    │  │ ⭐  │  Care Plus                    $99/tháng    │  │
│    │  └─────┘                                            │  │
│    │  • 5 lượt khám video/tháng (đã dùng 2)             │  │
│    │  • Xét nghiệm cơ bản                               │  │
│    │  • Chat AI không giới hạn                          │  │
│    │  Chu kỳ: 01/01 - 31/01/2026                        │  │
│    │  Ngày gia hạn: 01/02/2026                          │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              NÂNG CẤP GÓI                    [>]    │  │
│    └─────────────────────────────────────────────────────┘  │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              HẠ CẤP GÓI                      [>]    │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 2: Kiểm tra trạng thái

- ⛔ ACTIVE → tiếp tục | Past Due/Restricted → "Vui lòng thanh toán trước khi nâng cấp"

#### Bước 3: Hiển thị gói cao hơn

- Connect → Plus/Premium | Plus → Premium

#### Bước 4: Chọn gói mới

#### Bước 5: Hiển thị breakdown chi phí

- **INPUT:** selected_package (String)
- **OUTPUT:** proration_amount (Number), new_monthly_price (Number)
- Phí nâng cấp = (Giá mới - Giá cũ) × ngày còn lại / 30

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              NÂNG CẤP GÓI                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    Gói hiện tại: Care Plus ($99/tháng)                     │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ ●  Care Premium                    ⭐  $149/tháng   │  │
│    │    • Không giới hạn khám video                      │  │
│    │    • Xét nghiệm đầy đủ + Ưu tiên                   │  │
│    │    • Quản lý sức khỏe gia đình                      │  │
│    │    • Tư vấn chuyên gia 24/7                        │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    📋 Chi tiết thanh toán:                                  │
│    Gói hiện tại:           $99/tháng                       │
│    Gói mới:                $149/tháng                      │
│    Chênh lệch:             $50/tháng                       │
│    Số ngày còn lại:        20 ngày                         │
│    ─────────────────────────────────────────────           │
│    Phí nâng cấp ngay:      $33.33                          │
│    (= $50 × 20/30)                                         │
│    Từ kỳ sau (01/02):      $149/tháng                      │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │           XÁC NHẬN NÂNG CẤP - $33.33               │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 6-7: Xác nhận → Payment Gateway

- ⛔ Thành công → bước 8 | Thất bại → cho phép thử lại

#### Bước 8: Kích hoạt gói mới

- Cập nhật gói (hiệu lực ngay), giữ nguyên Next Billing Date, cập nhật quota (không reset lượt đã dùng)

#### Bước 9: Xác nhận thành công

- "Nâng cấp thành công!" + thông tin gói mới + quota mới

**Lưu ý:**
- Tối đa 2 lần upgrade/chu kỳ
- Quota mới = Quota gói mới - Số lượt đã dùng (KHÔNG reset)
- Ví dụ: Gói Connect 2 visits/tháng, đã dùng 1, upgrade lên Plus 5 visits → Còn 4 visits

---

### Nhánh B: Downgrade (Hạ cấp)

#### Bước 1: Vào Quản lý gói → "Hạ cấp gói"

#### Bước 2: Kiểm tra điều kiện

⛔ Rẽ nhánh:

| Điều kiện | Hành động |
|-----------|-----------|
| Đã upgrade trong 24h | Chặn: "Vui lòng đợi 24h sau khi nâng cấp" |
| Đã downgrade trong chu kỳ | Chặn: "Chỉ được hạ cấp 1 lần/chu kỳ" |
| OK | Tiếp tục |

**Validation Rules:**

| Rule | Message lỗi |
|------|-------------|
| Đã upgrade trong 24h | "Vui lòng đợi 24h sau khi nâng cấp để có thể hạ cấp" |
| Đã downgrade trong chu kỳ | "Bạn chỉ được hạ cấp 1 lần trong mỗi chu kỳ thanh toán" |

#### Bước 3: Hiển thị gói thấp hơn

- Premium → Plus/Connect | Plus → Connect

#### Bước 4: Chọn gói mới

#### Bước 5: Thông báo

- Gói mới có hiệu lực từ Next Billing Date, không hoàn tiền chu kỳ hiện tại

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              HẠ CẤP GÓI                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    Gói hiện tại: Care Premium ($149/tháng)                 │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ ○  Care Plus                           $99/tháng    │  │
│    │    • 5 lượt khám video/tháng                        │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │ ●  Care Connect                        $39/tháng    │  │
│    │    • 2 lượt khám video/tháng                        │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    ⚠️ LƯU Ý QUAN TRỌNG:                                     │
│    • Bạn vẫn dùng Care Premium đến 31/01/2026              │
│    • Từ 01/02/2026, chuyển sang Care Connect               │
│    • Không hoàn tiền cho chu kỳ hiện tại                   │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              XÁC NHẬN HẠ CẤP                        │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 6: Xác nhận (không cần thanh toán)

#### Bước 7: Ghi nhận scheduled_downgrade

#### Bước 8: Xác nhận: "Đã lên lịch hạ cấp"

**Lưu ý:**
- Tối đa 1 lần downgrade/chu kỳ
- Cooldown 24h sau upgrade
- Downgrade có hiệu lực từ kỳ sau
- Không hoàn tiền

---

### Nhánh C: Đổi gói cho Family Members (Adult)

- **Bước 1:** Primary vào Dashboard → "Quản lý gia đình"
- **Bước 2:** Chọn Adult muốn đổi gói
- **Bước 3:** Hệ thống hiển thị gói hiện tại: tên gói, giá, quota còn lại, ngày gia hạn
- **Bước 4:** Primary chọn "Đổi gói dịch vụ"
- **Bước 5:** Chọn gói mới (cao hơn hoặc thấp hơn)
- **Bước 6:** Upgrade: hiển thị proration, Primary thanh toán | Downgrade: thông báo hiệu lực từ kỳ sau
- **Bước 7:** Cập nhật gói Adult. Push + In-app notification cho Adult

### Mô hình Billing cho Family

| Hành động | Ai thực hiện | Ai thanh toán |
|-----------|-------------|--------------|
| Thêm Adult (ban đầu) | Primary | Primary |
| Primary đổi gói Adult | Primary | Primary |
| Primary mua add-on cho Adult | Primary | Primary |
| Adult tự đổi gói | Adult | Adult (thẻ riêng) |
| Adult tự mua add-on | Adult | Adult (thẻ riêng) |

---

## FLOW-07: MUA ADD-ON SERVICE

**Điều kiện:** Subscription phải ACTIVE (với subscription add-on). Pay-per-use cho phép cả khi Past Due (Hard Restriction).

### Danh sách Add-on

| Loại | Dịch vụ | Giá | Mô tả |
|------|---------|-----|-------|
| Pay-per-use | Schedule Video Visit | $50/lần | Đặt lịch khám video |
| Pay-per-use | Annual Health Check | $100/lần | Khám sức khỏe tổng quát |
| Pay-per-use | Medication Review | $25/lần | Tư vấn thuốc |
| Pay-per-use | Lab Results Consultation | $20/lần | Tư vấn kết quả xét nghiệm |
| Subscription | Chronic Management | $30/tháng | Quản lý bệnh mãn tính |

### Nhánh A: Mua Pay-per-use (Theo lần)

#### Bước 1: Vào Marketplace

- Dashboard → Settings → "Dịch vụ bổ sung"

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]              DỊCH VỤ BỔ SUNG                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    DỊCH VỤ THEO THÁNG                                       │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  🏥  Chronic Management              $30/tháng  [>] │  │
│    │      Quản lý bệnh mãn tính, nhắc thuốc             │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    MUA DỊCH VỤ LẺ (Pay-per-use)                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  📹  Schedule Video Visit            $50/lần    [>] │  │
│    └─────────────────────────────────────────────────────┘  │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  🩺  Annual Health Check            $100/lần    [>] │  │
│    └─────────────────────────────────────────────────────┘  │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  💊  Medication Review               $25/lần    [>] │  │
│    └─────────────────────────────────────────────────────┘  │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  🔬  Lab Results Consultation        $20/lần    [>] │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 2: Kiểm tra trạng thái

- ⛔ ACTIVE → hiển thị | Past Due (Soft) → chặn | Past Due (Hard) → cho phép mua

#### Bước 3-4: Hiển thị danh sách → Chọn dịch vụ

#### Bước 5: Xem chi tiết & Số lượng

- **INPUT:** quantity (Number: 1-10)
- **OUTPUT:** total_amount (Number), credits_after (Number)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]           SCHEDULE VIDEO VISIT                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    📹 Đặt lịch khám video với bác sĩ                        │
│    • Khám bệnh qua video call                               │
│    • Thời gian: 15-30 phút/lần                              │
│    • Nhận đơn thuốc điện tử                                 │
│                                                             │
│    Giá: $50/lần                                             │
│    Số lượng:    [-]    1    [+]                              │
│    ─────────────────────────────────────────────────────    │
│    Tổng thanh toán:           $50                           │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │              MUA NGAY - $50                         │  │
│    └─────────────────────────────────────────────────────┘  │
│    Số lượt hiện có: 0 lượt                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 6-7: Xác nhận → Payment Gateway (1 lần)

#### Bước 8: Ghi nhận lượt đã mua, cho phép sử dụng ngay

#### Bước 9: "Đã mua [X] lượt [Tên dịch vụ]!"

**Lưu ý:** Lượt mua lẻ không hết hạn. Thứ tự trừ: lượt mua lẻ TRƯỚC, quota gói SAU

| Ví dụ | Mô tả |
|-------|-------|
| KH có | Care Plus (5 video/tháng) + 2 lượt video mua lẻ |
| Lần khám 1-2 | Trừ lượt mua lẻ |
| Lần khám 3-7 | Trừ quota gói |
| Lần khám 8+ | Báo hết lượt, gợi ý mua thêm |

---

### Nhánh B: Đăng ký Subscription Add-on (Theo tháng)

#### Bước 1-2: Vào Marketplace → Kiểm tra trạng thái

- ⛔ ACTIVE → hiển thị | Past Due → chặn

#### Bước 3: Hiển thị Chronic Management $30/tháng

- Nếu đã đăng ký → "Đang sử dụng"

#### Bước 4-5: Chọn → Xem chi tiết

- **INPUT:** confirm (Boolean)
- **OUTPUT:** proration_amount (Number), new_monthly_total (Number)

**Wireframe ASCII:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]           CHRONIC MANAGEMENT                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    🏥 Quản lý bệnh mãn tính toàn diện                       │
│    ✓ Theo dõi chỉ số sức khỏe hàng ngày                    │
│    ✓ Nhắc uống thuốc đúng giờ                              │
│    ✓ Tư vấn định kỳ với bác sĩ chuyên khoa                 │
│    ✓ Báo cáo sức khỏe hàng tháng                           │
│    ✓ Cảnh báo khi chỉ số bất thường                        │
│                                                             │
│    📋 Chi tiết thanh toán:                                  │
│    Phí hàng tháng:         $30/tháng                       │
│    Số ngày còn lại:        20 ngày                         │
│    Phí kỳ này:             $20.00 (= $30 × 20/30)         │
│    ─────────────────────────────────────────────           │
│    Hóa đơn mới: Care Plus $99 + Chronic $30 = $129/tháng  │
│                                                             │
│    ┌─────────────────────────────────────────────────────┐  │
│    │         ĐĂNG KÝ NGAY - $20.00                       │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Bước 6-7: Xác nhận → Payment Gateway (phí proration)

#### Bước 8: Tạo subscription mới (ID riêng), sync billing date với gói chính

#### Bước 9: "Đã kích hoạt Chronic Management!" — có hiệu lực ngay

---

### Hủy Add-on Subscription

- **Bước 1:** Settings → "Dịch vụ bổ sung của tôi"
- **Bước 2:** Tap "Hủy" trên Chronic Management
- **Bước 3:** Thông báo: vẫn dùng đến hết kỳ, không hoàn tiền, có thể đăng ký lại
- **Bước 4:** Tắt auto-renewal, add-on vẫn chạy đến hết kỳ
- **Bước 5:** "Đã hủy. Vẫn dùng đến [Ngày]"

**Wireframe ASCII – Quản lý Add-on:**

```
┌─────────────────────────────────────────────────────────────┐
│  [←]           DỊCH VỤ CỦA TÔI                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    DỊCH VỤ ĐANG SỬ DỤNG                                     │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  🏥  Chronic Management              $30/tháng      │  │
│    │      Đang hoạt động • Gia hạn 01/02/2026           │  │
│    │                                              [HỦY]  │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    LƯỢT DỊCH VỤ CÒN LẠI                                     │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  📹  Schedule Video Visit                  3 lượt   │  │
│    │                                       [MUA THÊM]    │  │
│    └─────────────────────────────────────────────────────┘  │
│    ┌─────────────────────────────────────────────────────┐  │
│    │  🔬  Lab Results Consultation              1 lượt   │  │
│    │                                       [MUA THÊM]    │  │
│    └─────────────────────────────────────────────────────┘  │
│                                                             │
│    💡 Lượt dịch vụ không hết hạn, dùng khi nào cũng được   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Nhánh C: Mua Add-on cho Family Members (Adult)

- **Bước 1:** Primary vào Dashboard → "Quản lý gia đình"
- **Bước 2:** Chọn Adult muốn mua add-on
- **Bước 3:** "Mua dịch vụ bổ sung" → hiển thị marketplace
- **Bước 4:** Chọn dịch vụ (pay-per-use hoặc subscription)
- **Bước 5:** Hiển thị chi tiết, giá, số lượng
- **Bước 6:** Xác nhận → Primary thanh toán
- **Bước 7:** Add-on gán cho Adult. Push + In-app notification cho Adult

**Lưu ý:** Mỗi Adult có lượt add-on riêng (KHÔNG share giữa các member)

---

## BẢNG THÔNG BÁO TỰ ĐỘNG (MỞ RỘNG VỚI TEMPLATE)

| Sự kiện | Kênh | Người nhận |
|---------|------|------------|
| Kích hoạt Trial thành công | Email | Khách hàng |
| Thanh toán thành công (Purchase) | Email | Khách hàng |
| Trial sắp hết (ngày 7, 11, 13, 14) | SMS | Khách hàng |
| Primary gửi lời mời Family | SMS/Email | Adult được mời |
| Adult cấp quyền xem hồ sơ (HIPAA) | Push notification | Primary |
| Adult cấp quyền xem hồ sơ (HIPAA) | Email | Adult (xác nhận) |
| Adult thu hồi quyền xem | Push notification | Primary |
| Primary xem hồ sơ Adult | Push notification + In-app | Adult |
| Primary đổi gói cho Adult | Push + In-app notification | Adult |
| Primary mua add-on cho Adult | Push + In-app notification | Adult |
| Mật khẩu đã thay đổi | SMS | Khách hàng |

### Template chi tiết

#### 📧 Email chào mừng (Trial)

```
Subject: Chào mừng {full_name} đến với CVH Healthcare!

Xin chào {full_name},

Tài khoản Trial của bạn đã được kích hoạt!

Thông tin Trial:
• Gói: Care Connect
• Thời hạn: 14 ngày
• Ngày bắt đầu: {trial_start_date}
• Ngày hết hạn: {trial_end_date}

Bạn có thể sử dụng đầy đủ tính năng của gói Care Connect
trong thời gian Trial.

[BẮT ĐẦU SỬ DỤNG]

Mọi thắc mắc xin liên hệ hotline: +84 866 886 016

Trân trọng,
CVH Healthcare Team
```

#### 📧 Email xác nhận thanh toán (Purchase)

```
Subject: Xác nhận đăng ký gói {package_name} - CVH Healthcare

Xin chào {full_name},

Cảm ơn bạn đã đăng ký {package_name}!

Chi tiết đơn hàng:
• Mã giao dịch: {transaction_id}
• Gói: {package_name}
• Giá: ${price}/tháng
• Ngày thanh toán tiếp theo: {next_billing_date}

[XEM HÓA ĐƠN CHI TIẾT]

Trân trọng,
CVH Healthcare Team
```

#### 📱 SMS nhắc nhở Trial sắp hết

```
CVH Healthcare: Trial cua ban con {days_remaining} ngay.
Nang cap ngay de khong bi gian doan dich vu.
Hotline: +84866886016
```

#### 📱 SMS OTP

```
Mã xác thực CVH: {OTP}. Hiệu lực 5 phút.
```

#### 📧 Email mời tham gia Family

```
Subject: Lời mời tham gia gia đình CVH từ {primary_name}

Chào {invited_name},

{primary_name} đã mời bạn tham gia nhóm gia đình CVH
với tư cách là {relationship}.

Khi chấp nhận:
✓ {primary_name} sẽ thanh toán gói dịch vụ của bạn
✓ Bạn giữ nguyên gói {current_package} hiện tại
✓ Bạn giữ quyền quản lý hồ sơ sức khỏe riêng
✓ Bạn có thể chọn share hoặc không share với {primary_name}

[CHẤP NHẬN LỜI MỜI]

Hoặc copy link: {invitation_link}

Lời mời hết hạn sau 7 ngày.

Trân trọng,
CVH Healthcare Team
```

#### 🔔 Push Notification – Adult cấp quyền xem (HIPAA)

```
Title: Quyền xem hồ sơ đã được cấp
Body: {adult_name} đã cho phép bạn xem hồ sơ sức khỏe của họ.
```

#### 📧 Email xác nhận cấp quyền cho Adult

```
Subject: Xác nhận cấp quyền xem hồ sơ sức khỏe

Chào {adult_name},

Bạn đã cấp quyền cho {primary_name} xem hồ sơ sức khỏe của bạn.

Thời gian: {datetime}
HIPAA Authorization ID: {authorization_id}

Quyền của bạn:
• Thu hồi quyền xem bất cứ lúc nào từ Settings → Quyền riêng tư
• Nhận thông báo mỗi khi hồ sơ được xem

Trân trọng,
CVH Healthcare Team
```

#### 🔔 Push Notification – Adult thu hồi quyền xem

```
Title: Quyền xem hồ sơ đã bị thu hồi
Body: {adult_name} đã thu hồi quyền xem hồ sơ sức khỏe.
```

#### 🔔 Push Notification – Primary xem hồ sơ Adult

```
Title: Hồ sơ của bạn đang được xem
Body: {primary_name} vừa xem hồ sơ sức khỏe của bạn.
```

#### 📱 SMS thông báo mật khẩu đã thay đổi

```
CVH Healthcare: Mat khau tai khoan cua ban da duoc thay doi.
Neu khong phai ban, lien he ngay: +84866886016
```

---

## BẢNG GIÁM SÁT CS-MD ĐẦY ĐỦ

> Hệ thống auto ≥ 95%. CS chỉ can thiệp khi FATAL (< 5%). MD: KHÔNG can thiệp trong toàn bộ Function Registration.

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| Truy cập hệ thống | Tải ≤ 2s; 100% hiển thị đúng; 0% lỗi | Retry 3 lần, CDN fallback; Hiển thị "Hệ thống đang bận"; Alert kỹ thuật nếu lỗi >5 lần/phút | Server sập >10 phút, tự phục hồi thất bại → CS báo kỹ thuật | ❌ |
| Chọn phương thức đăng ký | 100% ghi nhận đúng; 3 lựa chọn hiển thị đủ; 0% lỗi | Log phương thức realtime; OAuth lỗi → gợi ý Phone/Email; Button không phản hồi → auto reload | Cả 3 phương thức sập >10 phút → CS báo kỹ thuật | ❌ |
| Xác nhận Phone/Email | Validation ≤ 1s; 100% phát hiện đúng; 0% false positive/negative | Log thời gian validate; DB timeout → retry 3 lần; Regex fallback; Hiển thị lỗi tại ô nhập | Database validation sập >10 phút → CS báo kỹ thuật | ❌ |
| Nhập thông tin cơ bản | 100% validate đủ điều kiện (≥18, mật khẩu mạnh); 0% mất dữ liệu | Auto-save session; Thanh độ mạnh realtime; DOB <18 → hiển thị ngay; Dừng >60s → popup hỗ trợ | Form treo, không submit sau 3 retry → CS báo kỹ thuật | ❌ |
| Gửi OTP SMS | OTP gửi ≤ 15s; 100% gửi đúng số; 100% OTP hợp lệ 5 phút; 0% duplicate | Log delivery status; Retry 3 lần; Chuyển nhà mạng dự phòng; Thất bại >5%/giờ → alert; Nút "Gửi lại" sau 30s | SMS gateway sập toàn diện >10 phút → CS báo kỹ thuật | ❌ |
| Xác thực OTP | 100% OTP đúng chấp nhận; 100% sai từ chối; Auto đếm ngược | Log số lần sai/session; OTP hết hạn → auto xóa; Sai 3 lần → khóa 5 phút; Countdown realtime | Module xác thực OTP sập → CS báo kỹ thuật | ❌ |
| Tạo tài khoản | Hoàn tất ≤ 3s; 100% mã không trùng; 100% dữ liệu đủ; 0% lỗi | Log thời gian tạo; DB lỗi → retry 3 lần; Trùng mã → auto generate lại; Fail → hoàn tác + thông báo | Database sập >10 phút → CS ghi nhận, báo kỹ thuật | ❌ |
| Chọn hình thức sử dụng | 100% hiển thị đúng 2 lựa chọn; 100% ghi nhận đúng; 0% định tuyến sai | Log lựa chọn realtime; Button lỗi → auto reload; Redirect sai → auto correct | Cả 2 lựa chọn sập >10 phút → CS báo kỹ thuật | ❌ |
| Chọn gói dịch vụ | 100% giá đúng; 100% Order tạo đúng; 0% sai lệch giá | So sánh giá vs DB realtime; Giá sai → auto refresh + alert; Order fail → retry 3 lần | Catalog giá sập >10 phút → CS báo kỹ thuật | ❌ |
| Thanh toán | Gateway ≤ 2s; Kết quả ≤ 10s; 0% duplicate charge; 0% mất dữ liệu | Log mọi transaction; Phát hiện duplicate → auto rollback; Timeout → hiển thị "Đang xử lý"; Thất bại → hiển thị lý do | Payment Gateway sập >10 phút → CS báo kỹ thuật + Gateway provider | ❌ |
| Kích hoạt gói | ≤ 1 phút; 100% đúng loại; 100% ngày hết hạn đúng; 0% kích hoạt nhầm | Log thời gian kích hoạt; Đối soát 100%/giờ; Fail → retry 3 lần; 3 lần fail → hoàn tiền + thông báo | Hệ thống kích hoạt sập, đã thanh toán nhưng không kích hoạt → CS báo kỹ thuật | ❌ |
| Thiết lập Face ID | 100% hiển thị đúng; 100% lưu đúng trạng thái; 0% lỗi kỹ thuật | Log trạng thái; Lỗi → "Không thể thiết lập, bật lại trong Cài đặt"; Cho phép bỏ qua | Module sinh trắc sập, không cho bỏ qua → CS báo kỹ thuật | ❌ |
| Gửi email chào mừng | Email ≤ 2 phút; 100% đủ thông tin; 100% delivery rate | Log delivery + timestamp; Bounce → retry 3 lần; Bounce liên tiếp → In-App thay thế; Template lỗi → dự phòng | Email service sập >10 phút → CS báo kỹ thuật | ❌ |
| Vào Dashboard | Tải ≤ 3s; 100% redirect đúng; 100% hiển thị đúng; 0% trắng màn hình | Log thời gian tải; Tải >5s → auto refresh; Redirect sai → auto correct; Dữ liệu thiếu → auto fetch lại | Dashboard sập hoàn toàn → CS báo kỹ thuật | ❌ |
| Nhập Phone/Email (Login) | 100% tra cứu đúng; 0% nhầm tài khoản | Log thời gian tra cứu; DB timeout → retry 3 lần; Không tìm thấy → "Đăng ký ngay?" | Database sập >10 phút → CS báo kỹ thuật | ❌ |
| Nhập mật khẩu (Login) | 100% đúng chấp nhận; 100% sai từ chối; Auto khóa sau 5 lần; 0% khóa nhầm | Log số lần sai; Sai lần 3 → cảnh báo "Còn 2 lần"; Sai lần 5 → khóa 30 phút + email; Auto mở khóa | Module xác thực sập → CS báo kỹ thuật | ❌ |
| Xác thực đăng nhập | Kết quả ≤ 2s; 100% kiểm tra đúng trạng thái; 0% cho đăng nhập khi không active | Log mọi lần xác thực; Auth timeout → retry 3 lần; Expired → redirect gia hạn | Auth service sập >10 phút → CS báo kỹ thuật | ❌ |
| Xác thực sinh trắc học | Màn hình ≤ 1s; 100% nhận diện đúng; 0% nhận nhầm; Auto fallback mật khẩu | Log kết quả; Thất bại 3 lần → fallback mật khẩu; Sensor lỗi → chuyển mật khẩu | Module sinh trắc sập, không cho chuyển mật khẩu → CS báo kỹ thuật | ❌ |
| Quên mật khẩu | 100% nút đúng vị trí; 100% điều hướng đúng | Log tỷ lệ sử dụng; Điều hướng fail → auto retry | Tính năng reset sập >10 phút → CS báo kỹ thuật | ❌ |
| Tạo mật khẩu mới | 100% validate đúng; 0% chấp nhận mật khẩu yếu | Log số lần nhập lại; Thanh độ mạnh realtime; Gợi ý cụ thể | Module validate sập → CS báo kỹ thuật | ❌ |
| Cập nhật mật khẩu | ≤ 3s; 100% mật khẩu cũ vô hiệu ngay; 100% đăng xuất thiết bị khác | Log thời gian cập nhật; DB write fail → retry 3 lần; Đăng xuất fail → retry; Audit log HIPAA | Database sập >10 phút → CS báo kỹ thuật | ❌ |
| Tự động đăng nhập | ≤ 3s redirect Dashboard; 100% đúng tài khoản; 0% nhập lại thủ công | Log kết quả auto-login; Thất bại → redirect đăng nhập + "Mật khẩu đã đặt lại" | Auto-login sập → CS báo kỹ thuật | ❌ |
| Quản lý gia đình | Tải ≤ 2s; 100% danh sách đúng; 0% nhầm thành viên | Log thời gian tải; Fail → retry 3 lần; Kiểm tra nhất quán | Module sập >10 phút → CS báo kỹ thuật | ❌ |
| Tạo Invitation | ≤ 3s; 100% token duy nhất; 100% thời hạn 7 ngày đúng; 0% tạo khi chưa thanh toán | Log mọi invitation; Token trùng → auto generate lại; Fail → retry 3 lần; Audit log HIPAA | Database sập >10 phút → CS báo kỹ thuật | ❌ |
| Gửi Link SMS/Email | ≤ 45s; 100% gửi đúng số/email; 100% link hoạt động | Log delivery status; SMS fail → retry 3 lần → thử Email; Cả 2 fail → In-App alert cho Primary | SMS + Email đồng thời sập >10 phút → CS báo kỹ thuật | ❌ |
| Adult click link | Tải ≤ 2s; 100% token hợp lệ chấp nhận; 100% hết hạn từ chối | Log mọi lần validate; Hết hạn → "Liên hệ Primary gửi lại"; Đã dùng → "Đăng nhập nếu đã có TK" | Module validate token sập >10 phút → CS báo kỹ thuật | ❌ |
| Tạo tài khoản Adult | ≤ 3s; 100% liên kết đúng Primary; 100% gói đúng; 0% lỗi billing | Đối soát 100% Adult vs billing Primary; Fail → retry 3 lần; Billing fail → alert ngay | Lỗi billing nghiêm trọng → CS báo kỹ thuật | ❌ |
| Hiển thị danh sách Member | ≤ 2s; 100% đúng đủ member active; 0% hiển thị đã xóa/chờ | Log thời gian tải; Fail → retry 3 lần; Kiểm tra realtime | Module Family Member sập >10 phút → CS báo kỹ thuật | ❌ |
| Chọn Member | 100% Dashboard đúng context; 0% nhầm dữ liệu y tế | Log member + context; Load nhầm → auto reset; Alert bảo mật nếu rò rỉ HIPAA | Rò rỉ dữ liệu y tế → CS + Compliance Officer ngay | ❌ |
| Quản lý gói | Tải ≤ 2s; 100% đúng gói + ngày hết hạn; 0% sai trạng thái | Log thời gian tải; Fail → retry 3 lần; Kiểm tra realtime | Module sập >10 phút → CS báo kỹ thuật | ❌ |
| Kiểm tra subscription | ≤ 1s; 100% đúng trạng thái billing; 0% ACTIVE khi quá hạn | Đồng bộ billing realtime; Past Due → cảnh báo + nút thanh toán; Timeout → cache gần nhất | Billing đồng bộ sập >10 phút → CS báo kỹ thuật | ❌ |
| Hiển thị gói upgrade/downgrade | Tải ≤ 2s; 100% giá đúng; gói hiện tại đánh dấu rõ | So sánh giá vs DB realtime; Giá sai → auto refresh + alert | Catalog sập >10 phút → CS báo kỹ thuật | ❌ |
| Breakdown chi phí | ≤ 2s; 100% proration đúng; 0% sai số tiền | Kiểm tra công thức mỗi deploy; Timeout → retry + loading; Sai tiền → alert, không cho confirm | Module tính phí sập >10 phút → CS báo kỹ thuật | ❌ |
| Marketplace | Tải ≤ 2s; 100% giá đúng; 100% phân biệt rõ loại; 0% dịch vụ không khả dụng | So sánh giá vs DB realtime; Giá sai → auto refresh + alert; Không khả dụng → auto ẩn | Catalog sập >10 phút → CS báo kỹ thuật | ❌ |
| Cấp quyền sử dụng add-on | ≤ 2 phút; 100% đúng số lượt; 100% billing sync đúng; 0% cấp nhầm | Đối soát 100% giao dịch vs quyền; Fail → retry 3 lần; Cấp nhầm → auto rollback | Hệ thống cấp quyền sập → CS báo kỹ thuật | ❌ |

---

## DATA SPECIFICATION

### Bảng tổng hợp tất cả data fields

| Node ID | Input Fields | Output Fields | Validation Rules |
|---------|-------------|--------------|-----------------|
| REG-01 | email | email_valid, email_exists | Regex email format |
| REG-02 | email, password, confirm_pwd, full_name, gender, phone, dob | all_fields_valid, dob_valid | Password: min 8, 1 upper, 1 num; Gender required; DOB ≥ 18 |
| REG-HIPAA | hipaa_consent, e_signature, full_name | consent_accepted, consent_timestamp, consent_version | Checkbox required, e_signature match full_name |
| REG-03 | otp_input, attempt_count | otp_verified, attempts_remaining | Max 3 attempts, 5 min expiry |
| REG-04 | (all from REG-02) | patient_id, account_status | Auto-generated ID: CVH-YYYYMMDD-XXXXX |
| REG-05 | user_choice | redirect_to | Enum: trial \| purchase |
| TRIAL-01 | patient_id | package, trial_dates, status | Duration 14 days |
| PURCHASE-01 | patient_id | cart_items, total | At least 1 item |
| PURCHASE-03 | card_info, address | payment_status, transaction_id | Luhn, PCI DSS |
| LOGIN-02 | email, password | login_success, session_token | Max 5 attempts |
| RESET-03 | new_password, confirm | password_valid | Same as REG-02 |
| FAMILY-02-ADULT | full_name, gender, phone/email (at least 1), dob, relationship, selected_package | invitation_id, invitation_token, invitation_link, package_paid | Gender required; Age ≥ 18, package_paid = true |
| FAMILY-ACCEPT | invitation_token, phone (if not in invitation), otp, password | user_id, account_status, linked_family | OTP verified, password valid |

### Bảng chi tiết Data Fields

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| patient_id | String | Auto | CVH-YYYYMMDD-XXXXX | Mã bệnh nhân, tự động tạo |
| email | String | Yes* | Regex email | Email đăng ký (*hoặc phone) |
| phone_number | String | Yes* | 10 số, format US (3-3-4) | SĐT đăng ký (*hoặc email) |
| password | String | Yes | Min 8, 1 upper, 1 num | Mật khẩu (hashed) |
| full_name | String | Yes | Min 2 ký tự, không chứa số | Họ tên đầy đủ |
| gender | String | Yes | Enum: Nam/Nữ/Khác | Giới tính |
| date_of_birth | Date | Yes | DD/MM/YYYY, Age ≥ 18 | Ngày sinh |
| hipaa_consent | Object | Yes | {accepted, timestamp, version, e_signature} | HIPAA consent record |
| otp_code | String | Auto | 6 số, hết hạn 5 phút | Mã OTP |
| biometric_enabled | Boolean | No | true/false | Đã bật Face ID/Touch ID |
| account_status | String | Auto | Enum: active/locked/expired | Trạng thái tài khoản |
| subscription_status | String | Auto | Enum: ACTIVE/Past Due/Trial | Trạng thái gói |
| selected_package | String | Yes | Enum: connect/plus/premium | Gói dịch vụ đã chọn |
| trial_start_date | Date | Auto | ISO 8601 | Ngày bắt đầu trial |
| trial_end_date | Date | Auto | trial_start + 14 days | Ngày hết hạn trial |
| next_billing_date | Date | Auto | ISO 8601 | Ngày thanh toán kế tiếp |
| transaction_id | String | Auto | UUID | Mã giao dịch thanh toán |
| invitation_id | String | Auto | UUID | ID lời mời |
| invitation_token | String | Auto | Unique token | Token cho link invitation |
| invitation_status | String | Auto | Enum: pending/accepted/expired/cancelled | Trạng thái lời mời |
| invitation_expiry | Date | Auto | created_at + 7 days | Ngày hết hạn lời mời |
| relationship | String | Yes | Enum: Vợ/Chồng/Con trưởng thành/Cha/Mẹ/Khác | Quan hệ với Primary |
| share_access | Boolean | No | true/false | Đã share quyền xem HIPAA |
| authorization_id | String | Auto | UUID | ID HIPAA Authorization |
| e_signature | String | Yes* | Khớp full_name | Chữ ký điện tử (*khi cần consent) |
| session_token | String | Auto | JWT | Token phiên đăng nhập |
| remember_me | Boolean | No | true/false | Ghi nhớ đăng nhập |
| scheduled_downgrade | Object | Auto | {new_package, effective_date} | Lịch hạ cấp gói |
| promo_code | String | No | Alphanumeric | Mã giảm giá |
| add_on_credits | Object | Auto | {service_name: remaining_credits} | Lượt dịch vụ lẻ còn lại |

---

## NGUỒN TÀI LIỆU GỐC

- Luồng nghiệp vụ: `docs/02-product/ba-specs/function-registration/base/Account_Registration_Touchpoint_Flow_v2.md`
- Checklist CS & MD: `docs/02-product/ba-specs/function-registration/checklist-cs-md/checklist-cs-md.md`
- Bản tóm tắt: `docs/02-product/tomtat/function-registration.md`
