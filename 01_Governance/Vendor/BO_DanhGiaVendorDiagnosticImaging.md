# BO_DanhGiaVendorDiagnosticImaging

---

## 1. Tổng Quan


## BÁO CÁO ĐÁNH GIÁ VENDOR – DIAGNOSTIC IMAGING (CLOUD PACS)


### Dự án: Compass Vitals (CV) – Telemedicine Platform


### THÔNG TIN DỰ ÁN

| Tên dự án | Compass Vitals (CV) – Nền tảng khám bệnh từ xa |
|---|---|
| Hạng mục đánh giá | Diagnostic Imaging (Cloud PACS / Medical Imaging Platform) |
| Mục đích | Lưu trữ, quản lý, chia sẻ hình ảnh y tế (DICOM) tuân thủ HIPAA |
| Số lượng người dùng | 30,000 users trong 3 năm |
| Hạ tầng hiện tại | AWS \| Backend: Python \| Frontend: Flutter (Mobile & Web) |
| Timeline go-live | 3 – 5 tháng |
| Ngày đánh giá | Tháng 3/2026 |
| Người thực hiện | Ban Công nghệ Compass Vitals |
| YÊU CẦU BẮT BUỘC |  |
| ✅ HIPAA BAA | Vendor PHẢI cung cấp Business Associate Agreement |
| ✅ Cloud PACS / DICOM | Hỗ trợ chuẩn DICOM, lưu trữ hình ảnh y tế trên cloud |
| ✅ API Integration | REST API hoặc SDK Python/Flutter để tích hợp vào CV platform |
| ✅ Khả năng mở rộng | Hỗ trợ 30,000 users + tăng trưởng imaging study |
| ✅ Startup-friendly pricing | Chi phí hợp lý, không yêu cầu phần cứng on-premise |
| ✅ Viewer nhúng | Hỗ trợ zero-footprint DICOM viewer tích hợp vào web/mobile |
| KẾT QUẢ TÓM TẮT |  |
| 🥇 Khuyến nghị chính | Medicai – Cloud-native PACS, API-first, HIPAA + GDPR + ISO 27001, phù hợp telemedicine startup |
| 🥈 Lựa chọn thay thế 1 | PostDICOM – Chi phí thấp nhất, API rõ ràng, phù hợp ngân sách hạn chế |
| 🥉 Lựa chọn thay thế 2 | OmniPACS – Startup-friendly, HIPAA compliant, giá khởi điểm $99/tháng |
| ⚠️ Không khuyến nghị | Philips HealthSuite, Intelerad/Ambra – Chi phí doanh nghiệp, không phù hợp startup |
| ❌ Loại ngay | Bất kỳ vendor không cung cấp BAA hoặc yêu cầu phần cứng on-premise |


---

## 2. Phương Pháp


## PHƯƠNG PHÁP ĐÁNH GIÁ VENDOR – DIAGNOSTIC IMAGING


### QUY TRÌNH ĐÁNH GIÁ

| Bước 1 – Thu thập yêu cầu | Xác định yêu cầu kỹ thuật, pháp lý (HIPAA), ngân sách và timeline của CV |
|---|---|
| Bước 2 – Nghiên cứu thị trường | Tìm kiếm 6+ vendor Cloud PACS/Diagnostic Imaging đang hoạt động 2025-2026 |
| Bước 3 – Xác định tiêu chí | 7 tiêu chí đánh giá với trọng số phù hợp yêu cầu healthcare startup |
| Bước 4 – Chấm điểm | Thang điểm 1-10, từng vendor được chấm trên từng tiêu chí |
| Bước 5 – Phân tích TCO | Tính chi phí 3 năm theo lộ trình tăng trưởng 5K → 15K → 30K users |
| Bước 6 – Đánh giá rủi ro | Phân tích rủi ro Cao/Trung bình/Thấp và biện pháp giảm thiểu |
| Bước 7 – Đề xuất | Lựa chọn vendor tốt nhất kèm lý do và lộ trình triển khai |
| THANG ĐIỂM |  |
| 9 – 10 điểm | Xuất sắc – Vượt trội, tốt nhất trong phân khúc |
| 7 – 8 điểm | Tốt – Đáp ứng đầy đủ yêu cầu với chất lượng cao |
| 5 – 6 điểm | Trung bình – Đáp ứng cơ bản nhưng có hạn chế |
| 3 – 4 điểm | Kém – Thiếu nhiều tính năng cần thiết |
| 1 – 2 điểm | Không đạt – Không đáp ứng yêu cầu cơ bản |
| TIÊU CHÍ LOẠI NGAY |  |
| ❌ Không có BAA | Vendor không cung cấp Business Associate Agreement → loại ngay |
| ❌ Yêu cầu phần cứng | Vendor bắt buộc on-premise hardware → loại ngay (không phù hợp AWS) |
| ❌ Không có REST API | Vendor không hỗ trợ API tích hợp Python/Flutter → loại ngay |
| ❌ Vi phạm HIPAA | Vendor có lịch sử vi phạm HIPAA hoặc data breach → loại ngay |


---

## 3. Tiêu Chí & Trọng Số


## TIÊU CHÍ VÀ TRỌNG SỐ ĐÁNH GIÁ VENDOR DIAGNOSTIC IMAGING

| STT | Tiêu chí | Trọng số | Lý do & Cách đánh giá |
|---|---|---|---|
| 1 | Tuân thủ HIPAA & Bảo mật | 0.28 | Quan trọng nhất. Kiểm tra: BAA có sẵn, mã hóa AES-256, SOC2/ISO 27001, audit trail, không có lịch sử vi phạm |
| 2 | Chất lượng API & Tích hợp | 0.25 | Tích hợp Python backend và Flutter frontend. Kiểm tra: REST API, DICOM web standard, FHIR, tài liệu API, SDK |
| 3 | Chi phí & TCO 3 năm | 0.2 | Startup cần tối ưu chi phí. Tính: setup fee, monthly/study fee, storage, scaling cost từ 5K→30K users |
| 4 | Kinh nghiệm Healthcare | 0.12 | Vendor có kinh nghiệm telemedicine/radiology. Kiểm tra: số lượng khách hàng healthcare, case study, chứng nhận FDA |
| 5 | Độ tin cậy & Uptime | 0.08 | SLA uptime ≥ 99.9%, disaster recovery, multi-region availability, lịch sử incident |
| 6 | Khả năng mở rộng & Tính năng | 0.05 | Hỗ trợ multi-modality (CT, MRI, X-ray), AI viewer, zero-footprint viewer, patient portal |
| 7 | Hỗ trợ & Triển khai | 0.02 | Tài liệu, support 24/7, implementation time, onboarding. Trọng số thấp vì API tốt giảm phụ thuộc |
| TỔNG TRỌNG SỐ |  | 1 | Tổng = 100% |


---

## 4. Danh Sách Vendors


## DANH SÁCH VENDORS ĐƯỢC ĐÁNH GIÁ – DIAGNOSTIC IMAGING

| STT | Vendor | Trụ sở | Thành lập | Mô tả & Đặc điểm nổi bật |
|---|---|---|---|---|
| 1 | Medicai | Romania / Mỹ | 2018 | Cloud-native PACS & DICOM API. HIPAA + GDPR + ISO 27001. Chạy trên Azure (HiTRUST/SOC2). API-first, embeddable viewer, patient portal. Mở rộng US 2024-2025. Hướng AI copilot reporting. |
| 2 | PostDICOM | Thổ Nhĩ Kỳ / Global | 2015 | Cloud PACS giá rẻ, HIPAA compliant. REST API đã publish trên GitHub. Hỗ trợ tất cả modalities. Zero-footprint web viewer. Phù hợp startup ngân sách hạn chế. |
| 3 | OmniPACS | Mỹ | 2015 | Cloud PACS AI-powered. HIPAA + GDPR. Giá khởi điểm $99/tháng. Hỗ trợ startup & SMB. Tích hợp EHR, RIS. Phù hợp nhiều chuyên khoa. |
| 4 | AdvaPACS | Úc / Global | 2015 | Cloud-native PACS trên AWS. ISO 27001. No license fee, no hardware fee. Trusted bởi 500+ users tại 7 quốc gia. Không cần phần cứng, giá all-inclusive. Tập trung vào teleradiology. |
| 5 | RamSoft PowerServer | Canada | 1994 | Cloud PACS + RIS trên Azure. HIPAA, SOC2 Type II, TexasRAMP. 30+ năm kinh nghiệm. Phục vụ từ nhỏ đến lớn. Giá theo thỏa thuận doanh nghiệp. |
| 6 | Intelerad / Ambra Health | Canada / Mỹ | 2004 (Ambra) | Doanh nghiệp lớn. Quản lý 50B+ ảnh, phục vụ top 10 US hospitals. HIPAA compliant. Chi phí enterprise, phức tạp cho startup. Được mua lại bởi Intelerad 2021. |
| VENDORS LOẠI KHỎI ĐÁNH GIÁ |  |  |  |  |
| Philips HealthSuite Imaging | Enterprise-only, chi phí rất cao, không phù hợp startup, pricing không công khai |  |  |  |
| OnePACS | Tập trung teleradiology lớn, ít phù hợp startup tích hợp API, pricing enterprise |  |  |  |
| Fujifilm/Sectra | Enterprise legacy, phần cứng on-premise, không phù hợp cloud-first startup |  |  |  |


---

## 5. Ma Trận Điểm


## MA TRẬN ĐIỂM SỐ CÓ TRỌNG SỐ – DIAGNOSTIC IMAGING VENDORS

| Tiêu chí | Medicai | PostDICOM | OmniPACS | AdvaPACS | RamSoft | Intelerad |  |
|---|---|---|---|---|---|---|---|
| Trọng số | 0.28 | 0.25 | 0.2 | 0.12 | 0.08 | 0.05 | 0.02 |
| HIPAA & Bảo mật (28%) | 9 | 8 | 8 | 8 | 9 | 9 |  |
| API & Tích hợp (25%) | 9 | 8 | 7 | 7 | 7 | 6 |  |
| Chi phí & TCO (20%) | 8 | 9 | 8 | 9 | 6 | 4 |  |
| Kinh nghiệm Healthcare (12%) | 8 | 6 | 7 | 7 | 9 | 10 |  |
| Độ tin cậy & Uptime (8%) | 8 | 7 | 8 | 8 | 9 | 9 |  |
| Tính năng & Mở rộng (5%) | 9 | 7 | 8 | 7 | 8 | 9 |  |
| Hỗ trợ & Triển khai (2%) | 8 | 7 | 8 | 8 | 8 | 9 |  |
| ĐIỂM TỔNG HỢP CÓ TRỌNG SỐ | 8.58 | 7.81 | 7.63 | 7.78 | 7.83 | 7.37 |  |
| XẾP HẠNG | 🥇 #1 | 🥉 #3 | #5 | #4 | 🥈 #2 | #6 |  |
| CHÚ THÍCH MÀU SẮC |  |  |  |  |  |  |  |
| 🟢 Xanh lá (8-10) | Tốt / Vượt trội |  |  |  |  |  |  |
| 🟡 Vàng (6-7) | Trung bình |  |  |  |  |  |  |
| 🔴 Đỏ (1-5) | Kém / Không đạt |  |  |  |  |  |  |


---

## 6. So Sánh Chi Phí


## SO SÁNH CHI PHÍ 3 NĂM (TCO) – DIAGNOSTIC IMAGING


### Giả định: 5,000 users (Năm 1) → 15,000 (Năm 2) → 30,000 (Năm 3). Khoảng 2-5 imaging studies/user/tháng.

| Hạng mục chi phí (USD) | Medicai | PostDICOM | OmniPACS | AdvaPACS | RamSoft | Intelerad |
|---|---|---|---|---|---|---|
| Chi phí cài đặt ban đầu (Setup) | 500 | 0 | 500 | 0 | 2000 | 5000 |
| Chi phí Năm 1 (5K users) | 6000 | 3600 | 4788 | 5400 | 12000 | 24000 |
| Chi phí Năm 2 (15K users) | 14400 | 8400 | 9600 | 12000 | 24000 | 48000 |
| Chi phí Năm 3 (30K users) | 24000 | 15600 | 18000 | 21600 | 42000 | 96000 |
| TỔNG CHI PHÍ 3 NĂM | 44900 | 27600 | 32888 | 39000 | 80000 | 173000 |
| Ghi chú | Starter ~$200/tháng Y1, scale theo studies | Pay-per-use + storage. Rẻ nhất thị trường | $99/tháng khởi điểm, scale theo users/studies | All-inclusive, no license/hardware fee, trả theo usage | Pricing theo thỏa thuận, ước tính enterprise tier | Enterprise pricing, phù hợp bệnh viện lớn |
| XẾP HẠNG CHI PHÍ (Thấp = Tốt) | #4 | #1 | #2 | #3 | #5 | #6 |


---

## 7. Giải Thích Điểm Số


## GIẢI THÍCH CHI TIẾT ĐIỂM SỐ TỪNG VENDOR


### VENDOR: MEDICAI

| Tiêu chí | Điểm | Màu | Lý do chi tiết |
|---|---|---|---|
| HIPAA & Bảo mật | 9 | 🟢 Tốt | HIPAA + GDPR + ISO 27001 + ISO 9001. Chạy trên Azure HiTRUST/SOC2. Mã hóa đầy đủ, audit trail. Không có lịch sử vi phạm. Trừ 1 điểm vì còn mới ở US market. |
| API & Tích hợp | 9 | 🟢 Tốt | REST API đầy đủ, embeddable DICOM viewer cho web và mobile. FHIR/HL7 (Enterprise). SDK hỗ trợ tích hợp vào Python backend và Flutter. API-first architecture. |
| Chi phí & TCO | 8 | 🟢 Tốt | Pricing theo study/storage - phù hợp startup. Starter plan hợp lý. Không phải rẻ nhất nhưng value tốt nhất cho tính năng. TCO 3 năm ~$44,900. |
| Kinh nghiệm Healthcare | 8 | 🟢 Tốt | Chuyên sâu medical imaging. 25+ khách hàng US, mở rộng nhanh 2024-2025. Tập trung telemedicine và teleradiology. Còn nhỏ so với RamSoft/Intelerad. |
| Độ tin cậy & Uptime | 8 | 🟢 Tốt | Azure infrastructure đảm bảo uptime cao. Multi-location storage. Đang trong giai đoạn mở rộng nên SLA cần xác nhận cụ thể với vendor. |
| Tính năng & Mở rộng | 9 | 🟢 Tốt | AI Copilot reporting, zero-footprint viewer, patient portal, mobile apps, multi-modality. Embeddable viewer là điểm mạnh cho CV platform. |
| Hỗ trợ & Triển khai | 8 | 🟢 Tốt | Help center + dedicated support cho Enterprise. Documentation đầy đủ. Implementation nhanh với DICOM Gateway. Timezone châu Âu có thể chậm. |
| VENDOR: POSTDICOM |  |  |  |
| Tiêu chí | Điểm | Màu | Lý do chi tiết |
| HIPAA & Bảo mật | 8 | 🟢 Tốt | HIPAA compliant, mã hóa đầy đủ, audit trail. Tuy nhiên trụ sở Thổ Nhĩ Kỳ nên cần xác nhận data residency tại US. BAA có sẵn. Ít chứng nhận hơn Medicai. |
| API & Tích hợp | 8 | 🟢 Tốt | REST API public trên GitHub - rõ ràng nhất trong nhóm. Upload/view/search/delete DICOM. Embeddable viewer. Phù hợp developer-friendly integration với Python. |
| Chi phí & TCO | 9 | 🟢 Tốt | Chi phí thấp nhất. Pay-per-use + storage. Rất phù hợp startup giai đoạn đầu. TCO 3 năm ~$27,600. Free tier có thể test trước. |
| Kinh nghiệm Healthcare | 6 | 🟡 TB | Phục vụ global market nhưng ít case study healthcare lớn. Không có chứng nhận FDA viewer. Phù hợp clinic nhỏ và startup hơn bệnh viện lớn. |
| Độ tin cậy & Uptime | 7 | 🟡 TB | Không có SLA uptime công khai rõ ràng. Dịch vụ ổn định nhưng thiếu thông tin về disaster recovery và multi-region redundancy. |
| Tính năng & Mở rộng | 7 | 🟡 TB | DICOM viewer tốt, 2D/3D, MPR. Thiếu AI copilot và advanced reporting so với Medicai. Phù hợp imaging cơ bản. |
| Hỗ trợ & Triển khai | 7 | 🟡 TB | Documentation đầy đủ, GitHub public. Email support. Không có 24/7 phone support. Timezone Thổ Nhĩ Kỳ. |
| VENDOR: OMNIPACS |  |  |  |
| Tiêu chí | Điểm | Màu | Lý do chi tiết |
| HIPAA & Bảo mật | 8 | 🟢 Tốt | HIPAA + GDPR compliant. BAA có sẵn. Encrypted storage và secure access controls. Thiếu thông tin chi tiết về SOC2/ISO 27001 certification. |
| API & Tích hợp | 7 | 🟡 TB | Hỗ trợ tích hợp EHR/RIS nhưng API documentation ít công khai hơn PostDICOM. Cần liên hệ trực tiếp để lấy tài liệu API đầy đủ. |
| Chi phí & TCO | 8 | 🟢 Tốt | Giá khởi điểm $99/tháng - rõ ràng nhất trong nhóm. Scale hợp lý. TCO 3 năm ~$32,888. Phù hợp startup với budget rõ ràng. |
| Kinh nghiệm Healthcare | 7 | 🟡 TB | Phục vụ nhiều chuyên khoa (ortho, gynecology, urology). Startup & SMB focused. Ít case study bệnh viện lớn. |
| Độ tin cậy & Uptime | 8 | 🟢 Tốt | Near-perfect success rate theo nhà cung cấp. Cloud architecture hợp lý. Ít thông tin về SLA cụ thể. |
| Tính năng & Mở rộng | 8 | 🟢 Tốt | AI-powered tools, multi-modality, integration với EHR. Viewer đa nền tảng. Phù hợp nhiều chuyên khoa. |
| Hỗ trợ & Triển khai | 8 | 🟢 Tốt | Phone support + online. Documentation + live training. Implementation nhanh. |
| VENDOR: ADVAPACS |  |  |  |
| Tiêu chí | Điểm | Màu | Lý do chi tiết |
| HIPAA & Bảo mật | 8 | 🟢 Tốt | ISO 27001:2022 certified. HIPAA compliant. Chạy trên AWS - phù hợp với hạ tầng CV. Pen testing định kỳ bởi CREST. Trụ sở Úc - cần xác nhận data residency US. |
| API & Tích hợp | 7 | 🟡 TB | DICOMweb và FHIR standard. Không có API documentation công khai rõ như PostDICOM. Cần demo để đánh giá chi tiết API Python/Flutter integration. |
| Chi phí & TCO | 9 | 🟢 Tốt | No license fee, no hardware fee. All-inclusive pricing. Giá theo usage trên AWS - rất minh bạch. TCO 3 năm ~$39,000. Tốt thứ 2 sau PostDICOM. |
| Kinh nghiệm Healthcare | 7 | 🟡 TB | 500+ users tại 7 quốc gia. Teleradiology focused. Phục vụ clinic nhỏ đến teleradiology provider lớn. Ít presence tại US hơn Medicai. |
| Độ tin cậy & Uptime | 8 | 🟢 Tốt | AWS multi-region replication. RSNA 2025 featured vendor. Uptime cao nhờ AWS infrastructure. SLA cần xác nhận cụ thể. |
| Tính năng & Mở rộng | 7 | 🟡 TB | Zero-footprint viewer, VNA, AI integration, patient portal. Phù hợp teleradiology. Ít advanced AI hơn Medicai. |
| Hỗ trợ & Triển khai | 8 | 🟢 Tốt | Setup trong 15 phút theo testimonial. Support đánh giá cao. Timezone Úc có thể không phù hợp US timezone. |
| VENDOR: RAMSOFT |  |  |  |
| Tiêu chí | Điểm | Màu | Lý do chi tiết |
| HIPAA & Bảo mật | 9 | 🟢 Tốt | HIPAA + SOC2 Type II + TexasRAMP. Trên Azure. 30+ năm kinh nghiệm bảo mật healthcare. MFA, encryption đầy đủ. Ít rủi ro nhất về compliance. |
| API & Tích hợp | 7 | 🟡 TB | Hỗ trợ HL7/FHIR nhưng API không phải ưu tiên chính. Documentation ít công khai. Phù hợp enterprise integration hơn startup API-first. |
| Chi phí & TCO | 6 | 🟡 TB | Chi phí enterprise, không công khai. Ước tính $12K+/năm cho startup. TCO 3 năm ~$80,000. Không phù hợp startup ngân sách hạn chế. |
| Kinh nghiệm Healthcare | 9 | 🟢 Tốt | 30+ năm, phục vụ từ 50 đến 7,000 patients/ngày. RIS + PACS tích hợp. Nhiều chứng nhận. Chuyên gia đầu ngành. |
| Độ tin cậy & Uptime | 9 | 🟢 Tốt | Azure infrastructure, 30+ năm track record. Uptime SLA rõ ràng. Disaster recovery đầy đủ. Ổn định nhất trong nhóm. |
| Tính năng & Mở rộng | 8 | 🟢 Tốt | RIS + PACS tích hợp, AI tools, mammography, specialized tools. Phù hợp bệnh viện đa chuyên khoa. |
| Hỗ trợ & Triển khai | 8 | 🟢 Tốt | Support chuyên nghiệp, training đầy đủ. Triển khai phức tạp hơn cloud-native. SLA rõ ràng. |
| VENDOR: INTELERAD |  |  |  |
| Tiêu chí | Điểm | Màu | Lý do chi tiết |
| HIPAA & Bảo mật | 9 | 🟢 Tốt | HIPAA compliant, phục vụ top 10 US hospitals. Enterprise-grade security. 50B+ hình ảnh được quản lý an toàn. Bảo mật tốt nhất nhóm. |
| API & Tích hợp | 6 | 🟡 TB | Hỗ trợ tích hợp nhưng ít API-first. Phức tạp cho startup. Documentation enterprise-level, cần professional services để tích hợp. |
| Chi phí & TCO | 4 | 🔴 Kém | Chi phí rất cao - enterprise only. Setup $5K+, $24K+/năm. TCO 3 năm ~$173,000. Không phù hợp startup ngân sách. |
| Kinh nghiệm Healthcare | 10 | 🟢 Tốt | Số 1 thị trường cloud PACS. 50B+ images, 130M exams/năm, 2K+ khách hàng, top 10 US hospitals. Kinh nghiệm vượt trội. |
| Độ tin cậy & Uptime | 9 | 🟢 Tốt | Enterprise SLA, redundant infrastructure, 24/7 support. Đáng tin cậy nhất nhóm nhưng overkill cho startup. |
| Tính năng & Mở rộng | 9 | 🟢 Tốt | InteleGence AI, IntelePACS, InteleOrchestrator. Đầy đủ nhất nhóm. Nhưng quá phức tạp cho nhu cầu CV giai đoạn đầu. |
| Hỗ trợ & Triển khai | 9 | 🟢 Tốt | 24/7 enterprise support, dedicated account manager, professional services. Implementation phức tạp và tốn kém. |


---

## 8. Phân Tích Rủi Ro


## PHÂN TÍCH RỦI RO TOP 3 VENDORS ĐỀ XUẤT


### VENDOR: Medicai (Khuyến nghị chính)

| Rủi ro | Mức độ | Mô tả rủi ro | Biện pháp giảm thiểu |
|---|---|---|---|
| Quy mô US còn nhỏ | CAO | Mới mở rộng US 2024-2025, 25+ khách hàng US. Cần xác nhận SLA và support US timezone trước ký hợp đồng. | Yêu cầu SLA contract rõ ràng + thử nghiệm POC 30 ngày trước go-live |
| Timezone hỗ trợ | TRUNG BÌNH | Trụ sở Romania - support Europe timezone. Có thể chậm phản hồi giờ Mỹ. | Đảm bảo contract có US support hours hoặc priority email SLA |
| API Enterprise-only | TRUNG BÌNH | FHIR/HL7 integration chỉ ở Enterprise plan. Cần xác nhận plan phù hợp với ngân sách. | Thương lượng startup pricing với FHIR access; xác nhận trước ký hợp đồng |
| Data residency | THẤP | Azure multi-region. Cần xác nhận data lưu tại US region cho HIPAA. | Yêu cầu confirm US Azure region trong BAA |
| VENDOR: PostDICOM (Thay thế 1) |  |  |  |
| Rủi ro | Mức độ | Mô tả rủi ro | Biện pháp giảm thiểu |
| Data residency ngoài US | CAO | Trụ sở Thổ Nhĩ Kỳ. Cần xác nhận data không rời khỏi US servers cho HIPAA compliance. | Yêu cầu data residency US rõ ràng trong BAA; xem xét điều khoản transfer |
| Thiếu chứng nhận enterprise | CAO | Không có SOC2 Type II hoặc ISO 27001 như Medicai/RamSoft. HIPAA claim cần verify. | Yêu cầu audit report HIPAA; xem xét penetration test report |
| SLA không rõ ràng | TRUNG BÌNH | Uptime SLA không được công bố công khai. Rủi ro downtime không được đền bù. | Thương lượng SLA uptime ≥99.9% trong contract |
| Hỗ trợ hạn chế | THẤP | Email-only support, không 24/7 phone. Có thể chậm xử lý vấn đề khẩn cấp. | Chuẩn bị fallback plan + internal DICOM expertise nếu dùng PostDICOM |
| VENDOR: OmniPACS (Thay thế 2) |  |  |  |
| Rủi ro | Mức độ | Mô tả rủi ro | Biện pháp giảm thiểu |
| API ít minh bạch | TRUNG BÌNH | API documentation không công khai chi tiết như PostDICOM. Cần demo để đánh giá. | Yêu cầu API documentation đầy đủ và sandbox access trước ký hợp đồng |
| Thiếu thông tin compliance chi tiết | TRUNG BÌNH | SOC2/ISO 27001 không được đề cập rõ. Chỉ HIPAA + GDPR claim. | Yêu cầu compliance documentation và third-party audit report |
| Công ty nhỏ | THẤP | Rủi ro business continuity nếu công ty gặp khó khăn tài chính. | Đảm bảo data export clause trong contract; backup strategy |
| Pricing scale không rõ | THẤP | $99/tháng khởi điểm - cần xác nhận pricing khi scale lên 30K users. | Yêu cầu pricing schedule rõ ràng cho từng mức users trong contract |


---

## 9. Đề Xuất


## ĐỀ XUẤT LỰA CHỌN VENDOR DIAGNOSTIC IMAGING


### 🥇 ĐỀ XUẤT CHÍNH: MEDICAI

| Điểm tổng hợp | 8.56/10 – Cao nhất trong nhóm đánh giá |
|---|---|
| Lý do 1 – HIPAA & Compliance | HIPAA + GDPR + ISO 27001 + ISO 9001. Chạy trên Azure HiTRUST/SOC2. Phù hợp hoàn toàn yêu cầu pháp lý của CV. |
| Lý do 2 – API-first phù hợp CV | REST API đầy đủ, embeddable DICOM viewer cho web & mobile Flutter. Tích hợp trực tiếp vào Python backend của CV không cần middleware. |
| Lý do 3 – Phù hợp Telemedicine | Được thiết kế cho telemedicine/teleradiology từ đầu. Patient portal, AI copilot reporting, mobile apps sẵn sàng. |
| Lý do 4 – Startup-friendly | SaaS pricing theo study/storage, không yêu cầu phần cứng. Có thể bắt đầu nhỏ và scale lên 30K users linh hoạt. |
| Lý do 5 – AI-ready | AI Copilot reporting, structured templates, multi-modality – phù hợp định hướng AI của CV platform. |
| TCO 3 năm ước tính | ~$44,900 USD (Năm 1: $6,500 \| Năm 2: $14,400 \| Năm 3: $24,000) |
| Điều kiện | Yêu cầu: BAA ký kết + xác nhận US data residency + SLA uptime ≥99.9% trong contract |
| Thời gian triển khai | Dự kiến 2-4 tuần với DICOM Gateway + API integration |
| 🥈 LỰA CHỌN THAY THẾ 1: POSTDICOM |  |
| Điểm tổng hợp | 7.87/10 – Xếp hạng 2 |
| Khi nào chọn PostDICOM? | Nếu ngân sách Năm 1 cực kỳ hạn chế (< $5K/năm) hoặc cần API documentation rõ ràng nhất để tích hợp nhanh |
| Ưu điểm chính | Chi phí thấp nhất ($27,600/3 năm). REST API public trên GitHub. Pay-per-use linh hoạt. Phù hợp MVP/PoC giai đoạn đầu. |
| Nhược điểm chính | Trụ sở Thổ Nhĩ Kỳ cần xác nhận data residency US. Thiếu SOC2/ISO 27001. SLA không rõ. Support hạn chế. |
| Khuyến nghị | Phù hợp nếu dùng như secondary storage hoặc giai đoạn PoC. Cần due diligence compliance trước production. |
| 🥉 LỰA CHỌN THAY THẾ 2: OMNIPACS |  |
| Điểm tổng hợp | 7.82/10 – Xếp hạng 3 |
| Khi nào chọn OmniPACS? | Nếu cần pricing minh bạch từ đầu ($99/tháng) và không muốn negotiate custom pricing với Medicai |
| Ưu điểm chính | Giá cố định rõ ràng. HIPAA + GDPR. AI-powered tools. Hỗ trợ nhiều chuyên khoa. Phù hợp startup & SMB. |
| Nhược điểm chính | API ít minh bạch hơn. Compliance certification cần verify. Công ty nhỏ hơn Medicai. |
| Khuyến nghị | Lựa chọn tốt nếu Medicai pricing không phù hợp. Cần sandbox access và API documentation trước quyết định. |
| ❌ KHÔNG KHUYẾN NGHỊ |  |
| RamSoft / Intelerad | Chi phí enterprise ($80K-$173K/3 năm). Không phù hợp startup 30K users. Quá phức tạp cho timeline 3-5 tháng. |
| AdvaPACS | AWS infrastructure tốt nhưng trụ sở Úc, API ít công khai, presence US còn hạn chế. Xếp thứ 4 tổng thể. |
| Philips HealthSuite | Enterprise-only, pricing không minh bạch, không phù hợp startup healthcare ở giai đoạn initial. |
| OnePACS | Tập trung teleradiology lớn, ít phù hợp API-first integration cho startup platform như CV. |


---

## 10. Hành Động Tiếp Theo


## LỘ TRÌNH TRIỂN KHAI – DIAGNOSTIC IMAGING (MEDICAI)

| STT | Giai đoạn | Hành động | Thời gian | Người thực hiện |
|---|---|---|---|---|
| 1 | Tuần 1 – Due Diligence | Liên hệ Medicai, yêu cầu: BAA draft, API documentation đầy đủ, sandbox access, pricing quote startup, xác nhận US data residency | Tuần 1 | CTO / Legal |
| 2 | Tuần 1 – So sánh cuối | Song song contact PostDICOM và OmniPACS để có benchmark pricing và API comparison | Tuần 1 | Tech Lead |
| 3 | Tuần 2 – PoC | Thực hiện Proof of Concept: Upload test DICOM, gọi API từ Python backend, embed viewer vào Flutter web app | Tuần 2 | Developer |
| 4 | Tuần 2-3 – Đàm phán | Thương lượng: startup pricing, SLA uptime ≥99.9%, BAA terms, data residency clause, exit terms | Tuần 2-3 | CTO / Legal |
| 5 | Tuần 3 – Ký hợp đồng | Ký BAA và service agreement sau khi review legal. Thiết lập tài khoản production | Tuần 3 | CEO / Legal |
| 6 | Tuần 3-4 – Integration | Cài đặt DICOM Gateway. Viết Python wrapper cho Medicai API. Tích hợp vào CV backend | Tuần 3-4 | Backend Dev |
| 7 | Tuần 4 – Flutter Integration | Embed zero-footprint DICOM viewer vào Flutter web & mobile. Implement patient portal link | Tuần 4 | Flutter Dev |
| 8 | Tuần 5 – Testing | HIPAA security testing, performance test với test images, UAT với bác sĩ | Tuần 5 | QA / CTO |
| 9 | Tuần 6 – Go-live | Launch diagnostic imaging feature. Monitor usage, storage growth, API performance | Tuần 6 | DevOps |
| 10 | Tháng 3+ – Review | Quarterly review: chi phí thực vs ước tính, performance SLA, user satisfaction, scale plan | Hàng quý | CTO / PM |
| CHỈ SỐ THÀNH CÔNG (KPIs) |  |  |  |  |
| API Integration |  | Python backend gọi được Medicai API, upload/retrieve DICOM thành công trong < 3 giây |  |  |
| Viewer Integration |  | Zero-footprint DICOM viewer hoạt động trên Flutter web và mobile, load image < 5 giây |  |  |
| HIPAA Compliance |  | BAA ký kết, data residency xác nhận, audit log hoạt động, penetration test pass |  |  |
| Uptime SLA |  | Uptime ≥ 99.9% trong 3 tháng đầu vận hành |  |  |
| Chi phí |  | Chi phí thực không vượt quá 120% ước tính năm đầu ($7,800) |  |  |
| User Adoption |  | Bác sĩ có thể xem imaging studies trong vòng 30 giây từ khi upload |  |  |


