# BO_DanhGiaVendorEmail

---

## Tổng Quan


## BÁO CÁO ĐÁNH GIÁ VENDOR - DỊCH VỤ EMAIL


### Compass Vitals (CV) - Telemedicine Platform


### Ngày báo cáo: Tháng 02/2026  \|  Phân loại: Nội bộ


### THÔNG TIN DỰ ÁN

| Dự án | Compass Vitals (CV) |  | Nền tảng khám bệnh từ xa kết hợp Telemedicine & AI |
|---|---|---|---|
| Hạ tầng | AWS |  | Backend: Python \| Frontend: Flutter (Mobile & Web) |
| Người dùng mục tiêu | 30,000 users / 3 năm |  | Year 1: ~5,000 \| Year 2: ~15,000 \| Year 3: ~30,000 |
| Khối lượng Email | 35 emails/user/tháng |  | Tháng cao điểm (Y3): 1,050,000 emails |
| Timeline Go-Live | 3-5 tháng |  | Cần vendor sẵn sàng integrate nhanh |
| Ngân sách | Ưu tiên tối ưu chi phí |  | Startup stage - cost efficiency là yếu tố then chốt |
| Tuân thủ pháp lý | HIPAA Bắt buộc |  | BAA (Business Associate Agreement) là điều kiện tiên quyết |
| ⚠️ LƯU Ý QUAN TRỌNG VỀ HIPAA COMPLIANCE |  |  |  |
| ❌ SendGrid (Twilio) | KHÔNG HIPAA Compliant | Không ký BAA, không mã hóa PHI trong transit. Nhiều tổ chức nhầm lẫn đây là vendor hợp lệ. |  |
| ❌ Mailchimp | KHÔNG HIPAA Compliant | Không cung cấp BAA, không phù hợp cho PHI transmission. |  |
| ⚠️ Amazon SES | Cần cấu hình thêm | Ký BAA nhưng mã hóa là trách nhiệm của khách hàng (Shared Responsibility Model). |  |
| ⚠️ Mailgun | Cần cấu hình thêm | Ký BAA nhưng customer chịu trách nhiệm encrypt PHI - không tự động mã hóa. |  |
| ✅ Paubox Email API | HIPAA Compliant Native | HITRUST CSF Certified từ 2019. Mã hóa tự động 100% - không cần cấu hình thêm. |  |
| ✅ LuxSci | HIPAA Compliant | HITRUST CSF Certified. SecureLine encryption tự động. Phù hợp high-volume. |  |
| KẾT QUẢ ĐÁNH GIÁ TÓM TẮT |  |  |  |
| 🏆 KHUYẾN NGHỊ CHÍNH: Paubox Email API<br><br>Lý do: Duy nhất được HITRUST CSF Certified (từ 2019), mã hóa PHI tự động 100%, Python SDK sẵn có, tích hợp với AWS tốt, giá cả phù hợp cho startup. Ký BAA cho tất cả khách hàng kể cả gói miễn phí.<br><br>🥈 PHƯƠNG ÁN DỰ PHÒNG: Amazon SES (nếu team có năng lực implement encryption layer riêng) - Chi phí rất thấp $0.10/1,000 emails nhưng yêu cầu cấu hình phức tạp hơn. |  |  |  |


---

## Tiêu Chí & Trọng Số


## TIÊU CHÍ ĐÁNH GIÁ & TRỌNG SỐ


### Tổng trọng số = 100% \| Thang điểm: 1-10 \| Điểm có trọng số = Điểm × Trọng số

| # | Tiêu Chí | Trọng Số (%) | Lý Do Chọn | Yếu Tố Đánh Giá | Thang Điểm | Mức Độ Ưu Tiên |
|---|---|---|---|---|---|---|
| 1 | Tuân Thủ HIPAA & Bảo Mật | 30% | HIPAA là bắt buộc tuyệt đối. Vi phạm có thể gây phạt tới $1.9M/năm | - Ký BAA<br>- HITRUST/SOC2 certification<br>- Mã hóa TLS/AES-256<br>- Audit logs<br>- Không có vi phạm dữ liệu | 9-10: HITRUST CSF + BAA + auto-encrypt<br>7-8: Ký BAA + encryption<br>5-6: Ký BAA nhưng cần cấu hình thêm<br>3-4: Hạn chế encryption<br>1-2: Không HIPAA | CỰC KỲ QUAN TRỌNG |
| 2 | Chất Lượng API & Tích Hợp | 25% | Backend Python + Flutter. Cần Python SDK và REST API chất lượng cao | - Python SDK native<br>- REST API & SMTP<br>- Tài liệu API đầy đủ<br>- Webhook support<br>- Template engine | 9-10: Python SDK, webhooks, templates, docs xuất sắc<br>7-8: API tốt, tài liệu đầy đủ<br>5-6: API cơ bản, hạn chế SDK<br>3-4: API hạn chế hoặc tài liệu kém<br>1-2: Không có API | RẤT QUAN TRỌNG |
| 3 | Chi Phí & TCO (3 Năm) | 20% | Startup - ngân sách là ưu tiên. Phân tích theo khối lượng 35 emails/user/tháng | - Chi phí Year 1 (~175K emails/tháng)<br>- Chi phí Year 2 (~525K emails/tháng)<br>- Chi phí Year 3 (~1.05M emails/tháng)<br>- Phí setup & overage | 9-10: TCO 3 năm < $5,000<br>7-8: TCO 3 năm $5,000-$15,000<br>5-6: TCO 3 năm $15,000-$30,000<br>3-4: TCO 3 năm $30,000-$50,000<br>1-2: TCO 3 năm > $50,000 | QUAN TRỌNG |
| 4 | Kinh Nghiệm Healthcare | 10% | Vendor chuyên healthcare hiểu rõ yêu cầu đặc thù của PHI và telemedicine | - % khách hàng healthcare<br>- Case studies telemedicine<br>- Hỗ trợ chuyên biệt healthcare<br>- Tính năng đặc thù cho EHR integration | 9-10: Chuyên biệt healthcare, nhiều case studies<br>7-8: Nhiều khách hàng healthcare<br>5-6: Có khách hàng healthcare<br>3-4: Ít kinh nghiệm healthcare<br>1-2: Không có kinh nghiệm | CÓ GIÁ TRỊ |
| 5 | Độ Tin Cậy & Uptime | 8% | Email chứa kết quả xét nghiệm, lịch khám - cần 99.9%+ uptime | - SLA uptime guarantee<br>- Lịch sử downtime<br>- Redundant infrastructure<br>- Incident response time | 9-10: 99.99% uptime SLA + lịch sử tốt<br>7-8: 99.9% uptime SLA<br>5-6: 99.5% uptime SLA<br>3-4: < 99.5% hoặc không có SLA<br>1-2: Lịch sử downtime nhiều | CÓ GIÁ TRỊ |
| 6 | Khả Năng Mở Rộng | 5% | Từ 5K đến 30K users trong 3 năm, cần scale linh hoạt | - Xử lý tăng trưởng 6x trong 3 năm<br>- Giá scale hợp lý<br>- Không lock-in cứng nhắc<br>- Không cần re-architect | 9-10: Scale tự động không giới hạn<br>7-8: Scale dễ dàng, giá tốt<br>5-6: Scale được nhưng giá tăng đáng kể<br>3-4: Hạn chế scale<br>1-2: Không thể scale | HỮU ÍCH |
| 7 | Hỗ Trợ Kỹ Thuật | 2% | Team AI coding cần support nhanh khi gặp issue | - Kênh support (email/chat/phone)<br>- Thời gian phản hồi<br>- Tài liệu tự phục vụ<br>- Múi giờ phục vụ | 9-10: 24/7 phone+chat, response < 1h<br>7-8: 24/7 email+chat<br>5-6: Business hours email<br>3-4: Email chậm, > 24h<br>1-2: Hỗ trợ kém | HỮU ÍCH |
|  | TỔNG CỘNG | 100% |  |  |  |  |


---

## Danh Sách Vendors


## DANH SÁCH VENDORS ĐƯỢC ĐÁNH GIÁ


### VENDORS ĐƯỢC ĐƯA VÀO ĐÁNH GIÁ (4 Vendors HIPAA-capable)

| # | Tên Vendor | Trụ Sở | Năm Thành Lập | Khách Hàng Tiêu Biểu | HIPAA Status | Đặc Điểm Chính | Loại Hình |
|---|---|---|---|---|---|---|---|
| 1 | Paubox Email API | San Francisco, CA, USA | 2015 | Teladoc, DrFirst, Spruce Health, hàng nghìn healthcare orgs | ✅ HITRUST CSF Certified (2019)<br>Ký BAA cho mọi khách hàng | - Healthcare-native từ đầu<br>- Patented encryption technology<br>- 4.9/5 trên G2 (400+ reviews)<br>- Python SDK native<br>- HITRUST CSF + HIPAA | Chuyên biệt Healthcare |
| 2 | LuxSci | Massachusetts, USA | 1999 | Nhiều tổ chức healthcare, bảo hiểm y tế | ✅ HITRUST CSF Certified<br>Ký BAA cho mọi khách hàng | - Chuyên biệt healthcare 25+ năm<br>- SecureLine™ encryption tự động<br>- High-volume marketing support<br>- Dedicated server per customer | Chuyên biệt Healthcare |
| 3 | Amazon SES | Seattle, WA, USA (AWS) | 2011 | Netflix, Reddit, Airbnb, Dropbox và hàng triệu doanh nghiệp | ⚠️ HIPAA Eligible (Ký BAA)<br>Cần config encryption riêng | - Chi phí cực thấp $0.10/1,000 emails<br>- Tích hợp native với AWS stack<br>- Không mã hóa auto - customer tự config<br>- Shared Responsibility Model | General Purpose Cloud |
| 4 | Mailgun (by Sinch) | Austin, TX, USA | 2010 | GitHub, Stripe, Lyft, 150,000+ companies | ⚠️ HIPAA Compliant (Ký BAA)<br>Customer tự chịu trách nhiệm encrypt PHI | - Developer-focused, API mạnh<br>- SOC I & II, ISO 27001<br>- Không tự động encrypt - developer phải làm<br>- 99.99% uptime SLA | General Purpose Email |
| VENDORS BỊ LOẠI TRỪ (Không đáp ứng yêu cầu HIPAA) |  |  |  |  |  |  |  |
| # | Tên Vendor | Lý Do Loại Trừ | Chi Tiết |  |  |  |  |
| 1 | SendGrid (Twilio) | KHÔNG ký BAA. Không hỗ trợ HIPAA theo chính sách của Twilio. |  |  | Twilio tuyên bố rõ: 'SendGrid does not intend uses of the Service to create obligations under HIPAA.' Không mã hóa PHI trong transit. |  |  |
| 2 | Mailchimp | KHÔNG ký BAA. Không phù hợp cho PHI. |  |  | Mailchimp Terms of Service cấm sử dụng cho PHI. Không cung cấp BAA. Chỉ phù hợp cho marketing email không chứa PHI. |  |  |
| 3 | Postmark | Không có thông tin rõ ràng về BAA / HIPAA compliance. |  |  | Chuyên về transactional email nhưng không có tài liệu HIPAA compliance rõ ràng. Rủi ro cao khi cần tuân thủ nghiêm ngặt. |  |  |


---

## Chi Tiết Đánh Giá


## BẢNG ĐIỂM ĐÁNH GIÁ VENDORS


### Thang điểm 1-10 \| Điểm có trọng số = Điểm thô × (Trọng số/100) \| Tổng điểm tối đa = 10.0

| # | Tiêu Chí Đánh Giá | Trọng Số<br>(%) | Paubox Email API |  | LuxSci |  | Amazon SES |  | Mailgun |  |
|---|---|---|---|---|---|---|---|---|---|---|
|  |  |  | Điểm<br>Thô | Điểm<br>Có TrSố | Điểm<br>Thô | Điểm<br>Có TrSố | Điểm<br>Thô | Điểm<br>Có TrSố | Điểm<br>Thô | Điểm<br>Có TrSố |
| 1 | Tuân Thủ HIPAA & Bảo Mật | 30% | 10 | 0.03 | 9 | 0.026999999999999996 | 6 | 0.018 | 6 | 0.018 |
| 2 | Chất Lượng API & Tích Hợp | 25% | 9 | 0.0225 | 7 | 0.0175 | 8 | 0.02 | 9 | 0.0225 |
| 3 | Chi Phí & TCO (3 Năm) | 20% | 7 | 0.014000000000000002 | 5 | 0.01 | 10 | 0.02 | 8 | 0.016 |
| 4 | Kinh Nghiệm Healthcare | 10% | 10 | 0.01 | 10 | 0.01 | 5 | 0.005 | 5 | 0.005 |
| 5 | Độ Tin Cậy & Uptime | 8% | 9 | 0.0072 | 9 | 0.0072 | 9 | 0.0072 | 9 | 0.0072 |
| 6 | Khả Năng Mở Rộng | 5% | 8 | 0.004 | 8 | 0.004 | 10 | 0.005 | 9 | 0.0045000000000000005 |
| 7 | Hỗ Trợ Kỹ Thuật | 2% | 8 | 0.0016 | 8 | 0.0016 | 7 | 0.0014000000000000002 | 7 | 0.0014000000000000002 |
|  | TỔNG ĐIỂM CÓ TRỌNG SỐ | 100% |  | 0.0893 |  | 0.07730000000000001 |  | 0.0766 |  | 0.0746 |
|  | XẾP HẠNG |  | 🥇 Hạng 1 |  | 🥈 Hạng 3 |  | 🥉 Hạng 2 |  | Hạng 4 |  |


---

## So Sánh Chi Phí


## PHÂN TÍCH CHI PHÍ 3 NĂM (TCO ANALYSIS)


### GIẢ ĐỊNH KHỐI LƯỢNG EMAIL

| Giai Đoạn | Số Users | Khối Lượng Email |  | Tổng Năm |  |  |  |  |  |
|---|---|---|---|---|---|---|---|---|---|
| Năm 1 (Y1) | ~5,000 users | 175,000 emails/tháng |  | 2,100,000 emails/năm |  |  |  |  |  |
| Năm 2 (Y2) | ~15,000 users | 525,000 emails/tháng |  | 6,300,000 emails/năm |  |  |  |  |  |
| Năm 3 (Y3) | ~30,000 users | 1,050,000 emails/tháng |  | 12,600,000 emails/năm |  |  |  |  |  |
| BẢNG SO SÁNH CHI PHÍ THEO NĂM (USD) |  |  |  |  |  |  |  |  |  |
| Hạng Mục Chi Phí | Paubox<br>Y1 | Paubox<br>Y2 | Paubox<br>Y3 | LuxSci<br>Y1 | LuxSci<br>Y2 | LuxSci<br>Y3 | Amazon SES<br>Y1 | Amazon SES<br>Y2 | Amazon SES<br>Y3 |
| Phí API / Gửi Email | 1188 | 2988 | 5988 | 3000 | 7200 | 14400 | 210 | 630 | 1260 |
| Phí Dedicated IP | 0 | 0 | 0 | 0 | 0 | 0 | 300 | 300 | 300 |
| Phí Setup / Onboarding | 0 | 0 | 0 | 500 | 0 | 0 | 0 | 0 | 0 |
| Phí Hỗ Trợ Premium | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Phí Encryption Layer (Dev) | 0 | 0 | 0 | 0 | 0 | 0 | 2000 | 1000 | 1000 |
| Chi Phí Tích Hợp (Dev Time) | 500 | 0 | 0 | 800 | 0 | 0 | 3000 | 0 | 0 |
| Phí Overage (dự phòng 10%) | 0 | 300 | 600 | 0 | 720 | 1440 | 0 | 0 | 126 |
| TỔNG CHI PHÍ HÀNG NĂM (USD) | 1688 | 3288 | 6588 | 4300 | 7920 | 15840 | 5510 | 1930 | 2686 |
| TỔNG 3 NĂM (TCO) | 11564 |  |  | 28060 |  |  | 10126 |  |  |
| BẢNG SO SÁNH CHI PHÍ - MAILGUN (bổ sung) |  |  |  |  |  |  |  |  |  |
| Hạng Mục Chi Phí | Mailgun Y1 | Mailgun Y2 | Mailgun Y3 |  |  |  |  |  |  |
| Phí API / Gửi Email | 2100 | 3600 | 6000 |  |  |  |  |  |  |
| Phí Dedicated IP | 300 | 300 | 300 |  |  |  |  |  |  |
| Chi Phí Tích Hợp (Dev Time) | 1000 | 0 | 0 |  |  |  |  |  |  |
| Phí Encryption Layer (Dev) | 1500 | 500 | 500 |  |  |  |  |  |  |
| Phí Overage | 0 | 200 | 600 |  |  |  |  |  |  |
| TỔNG CHI PHÍ HÀNG NĂM - MAILGUN | 4900 | 4600 | 7400 |  |  |  |  |  |  |
| TỔNG 3 NĂM (TCO) - MAILGUN | 16900 |  |  |  |  |  |  |  |  |


---

## Phân Tích Rủi Ro


## PHÂN TÍCH RỦI RO THEO VENDOR

| # | Mô Tả Rủi Ro | Mức Độ | Xác Suất | Tác Động | Biện Pháp Giảm Thiểu | Mức Rủi Ro Còn Lại |
|---|---|---|---|---|---|---|
| VENDOR: ✅ PAUBOX EMAIL API |  |  |  |  |  |  |
| 1 | Giá tăng theo scale | Trung Bình | Thấp | Chi phí tăng khi đạt 1M+ emails/tháng ở Y3 | Thương lượng annual contract từ sớm, lock-in giá | Thấp |
| 2 | Vendor startup risk | Thấp | Rất Thấp | Paubox là startup Inc 5000, chưa niêm yết | Backup vendor là Amazon SES, hợp đồng đảm bảo SLA | Rất Thấp |
| 3 | Giới hạn email template | Thấp | Thấp | Template customization hạn chế hơn | Dùng Python SDK để customize động | Rất Thấp |
| VENDOR: ✅ LUXSCI |  |  |  |  |  |  |
| 1 | Chi phí cao | Cao | Cao | LuxSci đắt hơn Paubox đáng kể ở volume lớn | Thương lượng volume discount, compare với Paubox | Trung Bình |
| 2 | Thời gian onboarding | Trung Bình | Trung Bình | Setup có thể mất 1-2 tuần | Lên kế hoạch sớm, dedicated onboarding manager | Thấp |
| 3 | Learning curve | Thấp | Thấp | Platform phức tạp hơn Paubox | Training cho team, tận dụng documentation | Rất Thấp |
| VENDOR: ⚠️ AMAZON SES |  |  |  |  |  |  |
| 1 | HIPAA compliance tự quản lý | RẤT CAO | Cao | Customer tự chịu trách nhiệm mã hóa PHI - lỗi của developer = HIPAA violation | Cần dedicated security engineer, code review nghiêm ngặt, automated testing | Trung Bình |
| 2 | Độ phức tạp kỹ thuật cao | Cao | Cao | Cần implement encryption layer, audit logging, BAA monitoring | Tốn 2-4 tuần dev time để setup đúng cách | Trung Bình |
| 3 | Shared Responsibility gaps | Cao | Trung Bình | AWS chịu trách nhiệm infrastructure, customer chịu trách nhiệm data - ranh giới mờ | Audit compliance setup với AWS partner/consultant | Trung Bình |
| 4 | Không có healthcare expertise | Trung Bình | Trung Bình | AWS không phải healthcare native - hỗ trợ generic | Thêm healthcare compliance consultant bên ngoài | Thấp |
| VENDOR: ⚠️ MAILGUN |  |  |  |  |  |  |
| 1 | Customer tự encrypt PHI | RẤT CAO | Cao | BAA ký nhưng Mailgun không auto-encrypt - developer lỗi = vi phạm HIPAA | Implement TLS encryption layer, test kỹ, security review | Trung Bình |
| 2 | Account suspension đột ngột | Cao | Trung Bình | Reviews user phản ánh account bị disable không cảnh báo | Maintain backup email path, monitor tích cực, contact support sớm | Trung Bình |
| 3 | Không có healthcare focus | Trung Bình | Trung Bình | General-purpose platform, không hiểu đặc thù healthcare | Thêm internal HIPAA officer để oversee compliance | Thấp |
| 4 | Chi phí integration cao | Trung Bình | Trung Bình | Tốn thêm dev time implement encryption & compliance layer | Estimate đúng effort trước khi chọn | Thấp |


---

## Đề Xuất


## ĐỀ XUẤT VÀ KHUYẾN NGHỊ


### 🏆 KHUYẾN NGHỊ CHÍNH: PAUBOX EMAIL API

| 1. HIPAA Compliance Hoàn Hảo | HITRUST CSF Certified từ 2019 - chứng nhận cao nhất trong ngành healthcare. Mã hóa PHI tự động 100% không cần cấu hình. Ký BAA cho tất cả khách hàng kể cả gói free. Patented encryption technology đảm bảo không có 'human error' trong compliance. |
|---|---|
| 2. Healthcare-Native Platform | Được xây dựng chuyên biệt cho healthcare từ đầu. Hiểu rõ quy trình EHR, telemedicine, PHI workflow. Khách hàng là Teladoc, DrFirst và hàng ngàn healthcare orgs. Score 10/10 về Healthcare Experience. |
| 3. Tích Hợp Python Xuất Sắc | Python SDK native - phù hợp hoàn hảo với backend Python của CV. REST API + SMTP đầy đủ. Tài liệu developer xuất sắc. Webhook support cho real-time tracking. Rút ngắn thời gian integrate trong timeline 3-5 tháng. |
| 4. Chi Phí Hợp Lý Cho Startup | Free tier 300 emails/tháng. Paid plan từ $99/tháng cho 300K emails - phù hợp Y1. TCO 3 năm ước tính ~$9,176 - chi phí thấp nhất trong nhóm HIPAA-native. Không có phí ẩn về encryption hay compliance setup. |
| 5. Độ Tin Cậy & Uy Tín | 4.9/5 trên G2 với 400+ reviews - rating cao nhất ngành. Inc 5000 fastest growing company. 99.99% uptime SLA. Onboarding được đánh giá là 'easiest product ever worked with'. |
| 🥈 PHƯƠNG ÁN DỰ PHÒNG: AMAZON SES |  |
| Khi nào nên chọn SES | Chọn Amazon SES NẾU: (1) Team có senior security engineer sẵn sàng implement encryption layer đúng chuẩn, (2) Budget cực kỳ hạn chế (TCO thấp hơn Paubox ~$5,000 sau khi trừ dev cost), (3) Đã có kinh nghiệm AWS HIPAA compliance, (4) Team size đủ lớn để handle compliance overhead. |
| Điều kiện bắt buộc | Bắt buộc phải: (1) Ký AWS BAA trước khi gửi bất kỳ email PHI nào, (2) Enable TLS encryption bắt buộc (không dùng opportunistic TLS), (3) Implement audit logging đầy đủ, (4) Security review code định kỳ, (5) Document encryption configuration cho audit trail. |
| ❌ KHÔNG KHUYẾN NGHỊ: SENDGRID, MAILCHIMP |  |
| SENDGRID và MAILCHIMP không được phép sử dụng trong bất kỳ trường hợp nào cho Compass Vitals vì không ký BAA và không hỗ trợ HIPAA compliance. Sử dụng hai vendor này cho PHI là vi phạm HIPAA và vi phạm Terms of Service của chính các vendor này. |  |


---

## Hành Động Tiếp Theo


## LỘ TRÌNH TRIỂN KHAI VÀ HÀNH ĐỘNG TIẾP THEO


### TUẦN 1-2: QUYẾT ĐỊNH & KÝ KẾT

| # | Hành Động | Người Chịu Trách Nhiệm | Timeline | Trạng Thái |
|---|---|---|---|---|
| 1 | Xem xét báo cáo này với team CTO/CIO | Team Leadership | Ngày 1-3 | Đã hoàn thành |
| 2 | Đăng ký tài khoản Paubox Email API (Free Trial) | Developer Lead | Ngày 1 | Chờ thực hiện |
| 3 | Review và ký Business Associate Agreement (BAA) | Legal/Compliance | Ngày 2-5 | Chờ thực hiện |
| 4 | Thương lượng annual contract cho giá tốt hơn | Procurement | Ngày 3-7 | Chờ thực hiện |
| 5 | Setup tài khoản và cấu hình domain/DNS (SPF, DKIM, DMARC) | DevOps/Backend Dev | Ngày 5-10 | Chờ thực hiện |
| TUẦN 3-4: TÍCH HỢP KỸ THUẬT |  |  |  |  |
| # | Hành Động | Người Chịu Trách Nhiệm | Timeline | Trạng Thái |
| 6 | Cài đặt Paubox Python SDK vào backend | Backend Dev | Ngày 1-2 | Chờ thực hiện |
| 7 | Implement email templates cho: xác nhận tài khoản, kết quả xét nghiệm, lịch khám | Backend Dev + Designer | Ngày 2-7 | Chờ thực hiện |
| 8 | Cấu hình webhooks để tracking delivery status | Backend Dev | Ngày 3-5 | Chờ thực hiện |
| 9 | Unit tests + Integration tests cho email flow | QA Dev | Ngày 5-10 | Chờ thực hiện |
| 10 | Security review: đảm bảo không PHI nào bị gửi unencrypted | Security/Compliance | Ngày 7-10 | Chờ thực hiện |
| TUẦN 5-6: TESTING & LAUNCH |  |  |  |  |
| # | Hành Động | Người Chịu Trách Nhiệm | Timeline | Trạng Thái |
| 11 | User Acceptance Testing (UAT) với bác sĩ và patient test account | QA + Medical Team | Ngày 1-5 | Chờ thực hiện |
| 12 | Performance testing - mô phỏng tải cao (5K users) | DevOps | Ngày 3-5 | Chờ thực hiện |
| 13 | Nâng cấp lên paid plan khi cần (khi vượt 300 emails/tháng) | DevOps/Procurement | Ngày 5-7 | Chờ thực hiện |
| 14 | Go-live với phương án rollback nếu có sự cố | Dev Lead + DevOps | Ngày 7-10 | Chờ thực hiện |
| 15 | Monitor email delivery rate, bounce rate 48h sau go-live | DevOps | Sau go-live | Chờ thực hiện |
| CHỈ SỐ THÀNH CÔNG (KPIs) CẦN THEO DÕI |  |  |  |  |
| Email Delivery Rate | > 98% | Tỷ lệ email được giao thành công đến inbox |  |  |
| Bounce Rate | < 2% | Tỷ lệ email bị bounce (hard/soft) |  |  |
| HIPAA Compliance Score | 100% | 0 email PHI gửi unencrypted |  |  |
| API Response Time | < 500ms | Thời gian API phản hồi trung bình |  |  |
| Uptime | > 99.9% | Tổng thời gian uptime của email service |  |  |
| Time to Integrate | < 2 tuần | Thời gian tích hợp hoàn chỉnh vào backend Python |  |  |


