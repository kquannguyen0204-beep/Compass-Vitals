# TỔNG QUAN HỆ THỐNG TÀI LIỆU — COMPASS VITALS

> **Đọc file này trước khi truy cập bất kỳ tài liệu nào trong hệ thống.**
> File này đóng vai trò bản đồ định hướng, giúp AI và người dùng hiểu đúng cấu trúc, phân loại và ranh giới giữa các vùng thông tin.

---

## 1. TỔNG QUAN HỆ THỐNG

Hệ thống tài liệu Compass Vitals được tổ chức thành **4 module chức năng** và **1 thư mục nền tảng** đặt độc lập tại cấp cao nhất. Cấu trúc này được thiết kế để AI phân tích đúng vai trò từng loại tài liệu, tránh nhầm lẫn giữa **quy trình vận hành nội bộ** và **dịch vụ mà khách hàng thực sự được hưởng**.

---

## 1.1. MÔ TẢ CÁC VÙNG THÔNG TIN

Hệ thống được chia thành **6 vùng thông tin** chính, mỗi vùng có vai trò và ranh giới riêng biệt:

| Vùng thông tin | Mô tả | Mục đích |
|---|---|---|
| **Tài liệu Nền tảng** (`00_Tong_Quan_He_Thong_Tai_Lieu/`) | Hai tài liệu cấp cao nhất, áp dụng xuyên suốt toàn bộ hệ thống | Cung cấp bản đồ định hướng và quy chế quản lý dữ liệu cho toàn tổ chức — đọc trước tiên, áp dụng cho mọi module |
| **MODULE 01 — Governance** (`01_Governance/`) | Tài liệu về tầm nhìn, pháp lý, quản trị dự án và đánh giá vendor | Lưu giữ nền tảng pháp lý và chiến lược của dự án; dùng để AI và Ban Quản trị hiểu đúng định hướng, phạm vi và cơ sở pháp lý |
| **MODULE 02 — Business Development** (`02_Business_Development/`) | Tài liệu nghiên cứu thị trường, định vị thương hiệu, kế hoạch kinh doanh và mô tả sản phẩm | Hỗ trợ đội ngũ sales, marketing và CS triển khai hoạt động kinh doanh; các tài liệu trong đây phản ánh góc nhìn BD/Marketing, **không phải tài liệu vận hành dịch vụ** |
| **MODULE 03 — Services** (`03_Services/`) | Danh mục các dịch vụ thực tế mà khách hàng đăng ký và trả tiền để sử dụng | Là nguồn thông tin chính xác và duy nhất để mô tả dịch vụ; AI phải dùng vùng này khi trả lời câu hỏi về "Compass Vitals cung cấp dịch vụ gì" |
| **MODULE 04 — Service Management** (`04_Service_Management/`) | Tài liệu SOP vận hành, KPI, dashboard và quy trình quản trị hệ thống | Hỗ trợ giám sát, đo lường và cải tiến hiệu quả vận hành nội bộ — không dùng để mô tả dịch vụ khách hàng *(thư mục đang trong quá trình bổ sung tài liệu)* |
| **Vùng Cách ly** (`_Loai_Khoi_He_Thong/`) | Tài liệu tạm thời bị loại khỏi hệ thống vì chưa hoàn thiện hoặc có thể gây nhiễu *(thư mục chưa được tạo — đang chờ phê duyệt nội dung trước khi kích hoạt)* | Ngăn AI sử dụng dữ liệu chưa chính xác hoặc chưa được phê duyệt; **AI tuyệt đối không được truy cập vùng này khi thư mục được tạo** |

---

### Sơ đồ phân cấp vùng thông tin

```
COMPASS VITALS — HỆ THỐNG TÀI LIỆU
│
├── 00_Tong_Quan_He_Thong_Tai_Lieu/   [NỀN TẢNG — ĐỌC TRƯỚC TIÊN]
│   ├── 00_Tong_Quan_He_Thong_Tai_Lieu.md    ← Bản đồ hệ thống (file này)
│   └── Quy che - Quy dinh/
│       └── BO_QuyCheQuanLyDuLieu.md         ← Quy chế áp dụng toàn tổ chức
│
├── 01_Governance/                    [QUẢN TRỊ & PHÁP LÝ]
│   ├── Ho so phap ly/
│   ├── Ho so quan tri doanh nghiep/
│   └── Vendor/
│
├── 02_Business_Development/          [KINH DOANH & MARKETING]
│   ├── Maketing/
│   │   ├── Design/
│   │   └── Market Research/
│   │       ├── Personas/
│   │       └── Survey/
│   ├── Product/
│   └── Report&Plan/
│
├── 03_Services/                      [DỊCH VỤ THỰC TẾ — NGUỒN CHÍNH XÁC DUY NHẤT]
│   ├── S1_AccesstoCare247/
│   ├── S2_SpecialistReferral/
│   ├── S3_VideoVisit/
│   ├── S4_ChronicManagement/
│   ├── S5_AsynchronousMessaging/
│   ├── S6_LabResultsConsultation/
│   ├── S7_RoutineCheckup/
│   ├── S8_CMR/
│   ├── S9_TMR/
│   └── S10_LabImagingManagement/
│
├── 04_Service_Management/            [VẬN HÀNH & KPI — đang bổ sung tài liệu]
│
└── _Loai_Khoi_He_Thong/  ⛔ [VÙNG CÁCH LY — chưa tạo, sẽ kích hoạt khi cần]
```

---

## 2. TÀI LIỆU NỀN TẢNG

Hai tài liệu trong `00_Tong_Quan_He_Thong_Tai_Lieu/` có giá trị áp dụng toàn hệ thống:

| File | Mô tả | Mục đích |
|---|---|---|
| `00_Tong_Quan_He_Thong_Tai_Lieu.md` | Bản đồ tổng thể của hệ thống tài liệu, mô tả cấu trúc, phân loại và ranh giới các vùng thông tin | Đọc trước tiên — giúp AI và người dùng định hướng đúng trước khi truy cập bất kỳ tài liệu nào khác |
| `Quy che - Quy dinh/BO_QuyCheQuanLyDuLieu.md` | Bộ quy tắc và chính sách quản lý, sử dụng, bảo mật dữ liệu trong tổ chức | Đảm bảo mọi hoạt động liên quan đến dữ liệu đều tuân thủ quy định nội bộ — áp dụng cho toàn bộ 4 module |

---

## 3. CẤU TRÚC 4 MODULE CHÍNH

### MODULE 01 — Tổng quan dự án & Quản trị (`01_Governance/`)

**Mô tả:** Vùng lưu trữ các tài liệu nền tảng pháp lý và chiến lược của dự án Compass Vitals — bao gồm hồ sơ doanh nghiệp, định hướng phát triển và đánh giá nhà cung cấp.

**Mục đích:** Cung cấp cho AI và Ban Quản trị cơ sở pháp lý chính xác, tầm nhìn chiến lược và bối cảnh để đưa ra các quyết định đúng đắn; không dùng để mô tả dịch vụ hoặc quy trình vận hành.

| Thư mục con | Mô tả | Mục đích |
|---|---|---|
| `Ho so phap ly/` | Hồ sơ pháp lý công ty (Articles of Incorporation, Business Entity Filing, EIN Letter) | Lưu trữ giấy tờ pháp lý để tra cứu khi cần xác minh tư cách pháp nhân |
| `Ho so quan tri doanh nghiep/` | Executive Summary, Project Description (EN & VN), CV Governance | Mô tả tổng thể dự án cho các bên liên quan và nhà đầu tư |
| `Vendor/` | Báo cáo đánh giá vendor theo từng dịch vụ (Video, SMS, Email, Fax, Payment, Lab, Diagnostic Imaging, E-Prescribing) và tổng hợp chi phí năm đầu | Hỗ trợ quyết định mua sắm và quản lý ngân sách vận hành |

---

### MODULE 02 — Phát triển kinh doanh, Sales & Marketing (`02_Business_Development/`)

**Mô tả:** Vùng lưu trữ tài liệu nghiên cứu thị trường, định vị thương hiệu, kế hoạch kinh doanh và mô tả sản phẩm từ góc độ BD/Marketing.

**Mục đích:** Trang bị cho đội sales, marketing và CS dữ liệu thị trường, chiến lược định vị và thông tin sản phẩm để tiếp cận và chuyển đổi khách hàng.

| Thư mục con | Mô tả | Mục đích |
|---|---|---|
| `Maketing/` | Bối cảnh dự án, định vị giá trị & USPs, định hướng thông báo hành trình khách hàng | Xây dựng nền tảng thương hiệu và thông điệp truyền thông |
| `Maketing/Design/` | Brand Guidelines (EN & VN), Docs Template, Leaflet (Du học sinh & Phụ huynh), Logo | Tài sản thương hiệu và thiết kế truyền thông |
| `Maketing/Market Research/` | Phân tích 7Ps đối thủ cạnh tranh, Painpoints, Phân tích đối tác y tế | Dữ liệu thị trường và cạnh tranh |
| `Maketing/Market Research/Personas/` | Persona: Du học sinh, Phụ nữ 26–45, Small Business Owner | Chân dung khách hàng mục tiêu cho chiến lược marketing |
| `Maketing/Market Research/Survey/` | Khảo sát cộng đồng người Việt tại Mỹ và du học sinh | Dữ liệu sơ cấp từ nhóm khách hàng mục tiêu |
| `Product/` | Mô tả chi tiết các dịch vụ từ góc độ sản phẩm | Cung cấp góc nhìn sản phẩm cho đội ngũ BD/Sales |
| `Report&Plan/` | KPI hành trình khách hàng, kế hoạch kinh doanh 12 tháng | Theo dõi tiến độ và định hướng chiến lược kinh doanh |

---

### MODULE 03 — Dịch vụ cung cấp (`03_Services/`)

**Mô tả:** Vùng thông tin chứa toàn bộ tài liệu mô tả các dịch vụ thực tế mà khách hàng đăng ký sử dụng và trả phí. Đây là **nguồn thông tin duy nhất và chính xác** để xác định "Compass Vitals cung cấp dịch vụ gì".

**Mục đích:** Đảm bảo AI luôn trả lời đúng khi được hỏi về dịch vụ; cung cấp tài liệu vận hành chuẩn cho đội CS và bác sĩ khi thực hiện từng dịch vụ. Mỗi thư mục con là một dịch vụ độc lập, hoàn chỉnh.

| Thư mục | Service ID | Tên dịch vụ | Mô tả ngắn |
|---|---|---|---|
| `S1_AccesstoCare247/` | Service-1 | 24/7 Access to Care | Kênh tiếp cận y tế 24/7 — AI sàng lọc, VN MD & US MD duyệt; xử lý cấp cứu, kê đơn, xét nghiệm và theo dõi |
| `S2_SpecialistReferral/` | Service-2 | Specialist Referral | Giới thiệu và điều phối khám chuyên khoa; phân luồng VIP (CS/MA hỗ trợ toàn bộ) vs Standard (tự phục vụ) |
| `S3_VideoVisit/` | Service-3 | Schedule Video Visit | Tư vấn y tế từ xa qua video; áp dụng theo gói Care Plus/Premium; đặt lịch trước 24h |
| `S4_ChronicManagement/` | Service-4 | Chronic Disease Management | Quản lý bệnh mãn tính liên tục — AI-first, 4 chương trình song song, Care Plan review mỗi 3 tháng |
| `S5_AsynchronousMessaging/` | Service-5 | Asynchronous Messaging | Nhắn tin bất đồng bộ với AI; AI trả lời < 10 giây; escalate lên bác sĩ khi cần |
| `S6_LabResultsConsultation/` | Service-6 | Lab Results Consultation | Tư vấn kết quả xét nghiệm do KH tự upload; AI OCR + auto-routing; pay-per-use |
| `S7_RoutineCheckup/` | Service-7 | Routine Health Checkup | Quản lý sức khỏe định kỳ — trigger-based; tạo Health Plan cá nhân hóa sau mỗi lần khám |
| `S8_CMR/` | Service-8 | Comprehensive Medication Review (CMR) | Đánh giá thuốc toàn diện sau onboarding; bao gồm CMR đầy đủ và Micro-CMR |
| `S9_TMR/` | Service-9 | Targeted Medication Review (TMR) | Đánh giá thuốc mục tiêu — được kích hoạt từ S8 khi phát hiện rủi ro Làn Vàng/Đỏ |
| `S10_LabImagingManagement/` | Service-10 | Lab & Imaging Management | Quản lý toàn bộ vòng đời xét nghiệm và chẩn đoán hình ảnh — từ tạo đơn đến tích hợp kết quả vào EHR |

Mỗi thư mục dịch vụ thường chứa:
- File `IT_service-X-*-chitiet.md` — luồng nghiệp vụ chi tiết
- File `IT_high-level-flow-service-X-*.md` — luồng tổng quan (một số service)
- File `CS_*Checklist.md` — checklist vận hành cho đội CS
- File `CS_*UIUX.md` — tài liệu thiết kế UI/UX

> ⚠️ **Lưu ý:** S8 (CMR) và S9 (TMR) là 2 service liên kết chặt chẽ. S9 không hoạt động độc lập — luôn được kích hoạt từ S8.

---

### MODULE 04 — Quản lý vận hành & KPI (`04_Service_Management/`)

**Mô tả:** Vùng thông tin tập trung các tài liệu giám sát, đo lường và quản trị chất lượng vận hành toàn hệ thống — bao gồm SOP, dashboard bác sĩ, KPI tổng thể và quy trình CS.

**Mục đích:** Hỗ trợ Ban Quản lý và đội CS theo dõi hiệu quả hoạt động, phát hiện điểm cần cải tiến và duy trì chất lượng dịch vụ ổn định. Vùng này phục vụ quản trị nội bộ, **không dùng để mô tả dịch vụ cho khách hàng**.

> Thư mục này đang trong quá trình bổ sung tài liệu.

---

## 4. TÀI LIỆU ĐANG ĐƯỢC CÁCH LY (`_Loai_Khoi_He_Thong/`)

**Mô tả:** Vùng cách ly dự kiến sẽ chứa các tài liệu bị tạm thời đưa ra khỏi hệ thống chính vì chưa hoàn thiện, đang chỉnh sửa, hoặc có nội dung có thể gây nhiễu khi AI phân tích.

> ⚠️ **Lưu ý:** Thư mục `_Loai_Khoi_He_Thong/` **chưa được tạo trong hệ thống**. Cấu trúc này được dự trữ theo kế hoạch quản trị và sẽ được kích hoạt khi có tài liệu cần cách ly.

**Mục đích:** Bảo vệ tính chính xác của toàn hệ thống — ngăn AI tiếp cận dữ liệu chưa được phê duyệt hoặc có thể dẫn đến kết luận sai. Tài liệu trong vùng này chỉ được đưa trở lại hệ thống sau khi được Ban Quản trị xem xét và phê duyệt.

AI **không được** sử dụng nội dung trong thư mục `_Loai_Khoi_He_Thong/` để phân tích hoặc trả lời (khi thư mục được tạo).

---

## 5. QUY ƯỚC ĐẶT TÊN (NAMING CONVENTION)

| Prefix | Viết tắt | Mô tả | Mục đích |
|---|---|---|---|
| `BO_` | Back Office / Governance | Tài liệu quản trị nội bộ, pháp lý, quy chế tổ chức | Phân biệt tài liệu quản trị với tài liệu nghiệp vụ và dịch vụ |
| `BD_` | Business Development | Tài liệu phát triển kinh doanh, marketing, sản phẩm | Nhận diện nhanh các tài liệu thuộc phạm vi sales và chiến lược thị trường |
| `CS_` | Customer Service | Tài liệu quy trình và checklist dành cho đội chăm sóc khách hàng | Hỗ trợ CS triển khai đúng quy trình khi phục vụ khách hàng |
| `IT_` | Information Technology | Tài liệu kỹ thuật, luồng hệ thống, thiết kế UI/UX | Phân biệt tài liệu kỹ thuật với tài liệu vận hành và nghiệp vụ |
| `MD_` | Medical Doctor | Tài liệu hướng dẫn lâm sàng và quy trình dành cho bác sĩ | Đảm bảo bác sĩ có tài liệu chuẩn để thực hiện dịch vụ đúng giao thức y tế |

---

*Tài liệu này được tạo ngày 02/04/2026 theo chỉ đạo của Ban Quản trị Compass Vitals. Cập nhật lần cuối: 09/04/2026.*
