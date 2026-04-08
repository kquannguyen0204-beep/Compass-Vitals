# BO_DanhGiaVendorDuoc

---

## 1. Tổng Quan


## BÁO CÁO ĐÁNH GIÁ VENDOR DỊCH VỤ DƯỢC (E-PRESCRIBING & PHARMACY API)


### Dự án: Compass Vitals (CV) – Nền tảng khám bệnh từ xa


### Ngày lập báo cáo: Tháng 3/2026


### THÔNG TIN DỰ ÁN

| Tên dự án | Compass Vitals (CV) – Telemedicine Platform |
|---|---|
| Loại dịch vụ cần đánh giá | E-Prescribing API / Pharmacy Fulfillment Service |
| Hạ tầng | AWS (Python Backend, Flutter Mobile & Web Frontend) |
| Số lượng người dùng mục tiêu | 30.000 users trong 3 năm đầu |
| Yêu cầu tuân thủ | HIPAA bắt buộc + BAA với vendor |
| Thời gian go-live | 3 – 5 tháng |
| Mức độ ưu tiên ngân sách | Cao – chi phí là yếu tố quan trọng |
| EHR | Tự xây dựng |
| MỤC TIÊU ĐÁNH GIÁ |  |
| 1. Xác định các vendor cung cấp dịch vụ E-Prescribing / Pharmacy API phù hợp cho nền tảng telemedicine. |  |
| 2. Đảm bảo tuân thủ HIPAA với BAA được ký kết chính thức. |  |
| 3. Đánh giá khả năng tích hợp API với Python backend và Flutter mobile app. |  |
| 4. Phân tích TCO 3 năm để lựa chọn phương án tối ưu ngân sách. |  |
| 5. Đánh giá rủi ro pháp lý, kỹ thuật và vận hành cho từng vendor. |  |
| 6. Đưa ra khuyến nghị cụ thể phù hợp với timeline go-live 3–5 tháng. |  |
| KẾT QUẢ TỔNG HỢP |  |
| 🥇 Khuyến nghị Chính | DoseSpot – E-Prescribing API tích hợp dành cho telehealth |
| 🥈 Phương án Thay thế | Truepill – Pharmacy Fulfillment API toàn diện (end-to-end) |
| ⚠️ Không Khuyến nghị | Surescripts (direct) – phức tạp, chi phí cao, không phù hợp startup |
| ❌ Loại khỏi đánh giá | Amazon RxPass (B2C, không phải API), CVS/Walgreens API (giới hạn) |


---

## 2. Phương Pháp


## PHƯƠNG PHÁP ĐÁNH GIÁ VENDOR DƯỢC


### KHUNG ĐÁNH GIÁ

| Số lượng vendor đánh giá | 6 vendors |  |
|---|---|---|
| Thang điểm | 1–10 điểm (10 = xuất sắc nhất) |  |
| Số tiêu chí đánh giá | 7 tiêu chí có trọng số |  |
| Phân tích chi phí | TCO 3 năm (Năm 1: ~5.000 users, Năm 2: ~15.000, Năm 3: ~30.000) |  |
| Nguồn dữ liệu | Website chính thức, tài liệu API, đánh giá G2/Capterra, báo cáo ngành 2024–2025 |  |
| Ngôn ngữ báo cáo | Tiếng Việt |  |
| THANG ĐIỂM CHI TIẾT |  |  |
| 9 – 10 điểm | Xuất sắc – Vượt trội so với yêu cầu, tiêu chuẩn ngành hàng đầu |  |
| 7 – 8 điểm | Tốt – Đáp ứng đầy đủ yêu cầu với chất lượng cao |  |
| 5 – 6 điểm | Đạt – Đáp ứng yêu cầu cơ bản nhưng còn hạn chế |  |
| 3 – 4 điểm | Yếu – Đáp ứng một phần, có khoảng cách đáng kể |  |
| 1 – 2 điểm | Không đạt – Không đáp ứng yêu cầu tối thiểu |  |
| CÁC TIÊU CHÍ VÀ TRỌNG SỐ |  |  |
| Tiêu chí | Trọng số | Mô tả đánh giá |
| 1. Tuân thủ HIPAA & Bảo mật | 25% | BAA sẵn có, chứng chỉ SOC 2, mã hóa, lịch sử vi phạm |
| 2. Chất lượng API & Tích hợp | 25% | RESTful API, SDK Python/Flutter, tài liệu, thời gian tích hợp |
| 3. Chi phí & TCO 3 năm | 20% | Chi phí theo đơn/tháng, phí setup, phí EPCS, phí ẩn |
| 4. Kinh nghiệm Healthcare | 10% | Số khách hàng telemedicine, case study, chứng chỉ Surescripts |
| 5. Độ tin cậy & Uptime | 10% | SLA cam kết, lịch sử downtime, khả năng phục hồi |
| 6. Khả năng mở rộng | 5% | Giới hạn transaction, hỗ trợ multi-state, EPCS |
| 7. Hỗ trợ & Dịch vụ | 5% | Thời gian phản hồi, dedicated support, tài liệu tích hợp |


---

## 3. Danh Sách Vendors


## DANH SÁCH VENDORS ĐƯỢC ĐÁNH GIÁ

| Vendor | Loại Dịch vụ | Năm thành lập | Trụ sở | Khách hàng tiêu biểu | Mô tả |
|---|---|---|---|---|---|
| DoseSpot | E-Prescribing API (Tích hợp vào app) | 2009 | Dedham, MA, USA | 300+ clients telehealth | Nền tảng e-prescribing được Surescripts & ONC chứng nhận, chuyên dành cho tích hợp vào EHR/telehealth apps. Có 2 loại tích hợp: Jumpstart (iFrame) và Full Integration (API thuần). |
| DrFirst | E-Prescribing + Medication Management | 2000 | Rockville, MD, USA | Hệ thống bệnh viện lớn | Giải pháp quản lý thuốc và e-prescribing toàn diện, phù hợp cho tổ chức lớn. Tích hợp với các EHR lớn như Epic, Cerner. Giá khởi điểm ~$799/tính năng/năm. |
| Truepill | Pharmacy Fulfillment API (End-to-end) | 2016 | Hayward, CA, USA | Hims, Done, Ro Health | API dược toàn diện: từ e-prescribing đến fulfillment và giao thuốc tại nhà. Phù hợp telemedicine cần D2C pharmacy. HIPAA compliant Covered Entity. |
| MDToolbox | E-Prescribing (Standalone + API) | ~2010 | USA | Phòng khám nhỏ & vừa | Giải pháp e-prescribing giá rẻ, có API nhưng giới hạn. Tốt cho stand-alone, ít phù hợp cho tích hợp sâu vào custom app. Giá từ $28–$38/tháng/prescriber. |
| Surescripts (trực tiếp) | Mạng lưới e-prescribing quốc gia | 2001 | Arlington, VA, USA | 85% Rx toàn quốc Mỹ | Mạng lưới e-Rx lớn nhất Mỹ (85% Rx). Tích hợp trực tiếp rất phức tạp và tốn kém. Phù hợp hơn khi truy cập gián tiếp qua DoseSpot/DrFirst. |
| Rcopia (Allscripts/Veradigm) | E-Prescribing cho EHR | ~2005 | Chicago, IL, USA | Bệnh viện & hệ thống lớn | Giải pháp e-prescribing của Allscripts (nay là Veradigm). Phức tạp, hướng đến enterprise, ít phù hợp startup telemedicine với timeline ngắn. |
| VENDORS LOẠI KHỎI ĐÁNH GIÁ |  |  |  |  |  |
| Vendor | Lý do loại |  |  |  |  |
| Amazon RxPass | Dịch vụ B2C trực tiếp cho bệnh nhân, không có API tích hợp cho telemedicine platform. |  |  |  |  |
| CVS/Walgreens Direct API | Không cung cấp API mở cho third-party telemedicine; hạn chế phạm vi địa lý. |  |  |  |  |
| GoodRx Care | Bị FTC điều tra vi phạm chia sẻ dữ liệu sức khỏe không có sự đồng ý → rủi ro pháp lý cao. |  |  |  |  |


---

## 4. Chi Tiết Đánh Giá


## MA TRẬN ĐÁNH GIÁ CÓ TRỌNG SỐ – VENDOR DƯỢC

| Tiêu chí đánh giá | Trọng số (%) | DoseSpot | DrFirst | Truepill | MDToolbox | Surescripts | Rcopia (Allscripts) |
|---|---|---|---|---|---|---|---|
| 1. Tuân thủ HIPAA & Bảo mật | 25% | 9 | 9 | 9 | 8 | 9 | 8 |
| 2. Chất lượng API & Tích hợp | 25% | 9 | 7 | 8 | 5 | 4 | 5 |
| 3. Chi phí & TCO 3 năm | 20% | 8 | 5 | 7 | 9 | 3 | 4 |
| 4. Kinh nghiệm Healthcare | 10% | 9 | 8 | 8 | 6 | 10 | 7 |
| 5. Độ tin cậy & Uptime | 10% | 8 | 8 | 8 | 7 | 9 | 7 |
| 6. Khả năng mở rộng | 5% | 8 | 8 | 9 | 6 | 9 | 6 |
| 7. Hỗ trợ & Dịch vụ | 5% | 8 | 7 | 8 | 7 | 6 | 6 |
| ĐIỂM CÓ TRỌNG SỐ TỔNG | 100% | 8.6 | 7.35 | 8.1 | 7 | 6.5 | 6.05 |
| XẾP HẠNG |  | 1 | 3 | 2 | 4 | 5 | 6 |
| CHÚ THÍCH MÀU SẮC: |  |  |  |  |  |  |  |
| Xanh lá (8–10): Xuất sắc / Tốt | Vàng (6–7): Đạt yêu cầu | Đỏ (<6): Không đạt / Yếu |  |  |  |  |  |


---

## 5. So Sánh Chi Phí


## SO SÁNH CHI PHÍ TCO 3 NĂM – VENDOR DƯỢC


### Giả định: ~500 prescriptions/tháng (Năm 1) → ~2.000/tháng (Năm 3)

| Hạng mục chi phí (USD) | DoseSpot | DrFirst | Truepill | MDToolbox | Surescripts | Rcopia (Allscripts) |
|---|---|---|---|---|---|---|
| PHÍ SETUP & TÍCH HỢP (1 lần) |  |  |  |  |  |  |
| Phí setup / onboarding | 2000 | 5000 | 3000 | 0 | 50000 | 20000 |
| Phí chứng nhận Surescripts | 1500 | 2000 | 1500 | 800 | 5000 | 3000 |
| Phí EPCS (Electronic Prescription Controlled Substances) | 1500 | 1500 | 1500 | 500 | 3000 | 2000 |
| Tổng phí setup | 5000 | 8500 | 6000 | 1300 | 58000 | 25000 |
| NĂM 1 – 5.000 USERS (~500 Rx/tháng) |  |  |  |  |  |  |
| Phí thuê bao hàng tháng | 525 | 600 | 800 | 168 | 2000 | 500 |
| Phí usage/API (ước tính) | 0 | 100 | 200 | 0 | 500 | 200 |
| Phí hỗ trợ kỹ thuật | 0 | 0 | 0 | 0 | 500 | 200 |
| Tổng chi phí Năm 1 | 525 | 700 | 1000 | 168 | 3000 | 900 |
| NĂM 2 – 15.000 USERS (~1.200 Rx/tháng) |  |  |  |  |  |  |
| Phí thuê bao hàng tháng | 1050 | 1200 | 1500 | 336 | 4000 | 1000 |
| Phí usage/API (ước tính) | 100 | 200 | 400 | 50 | 1000 | 400 |
| Phí hỗ trợ kỹ thuật | 0 | 200 | 0 | 0 | 500 | 300 |
| Tổng chi phí Năm 2 | 1150 | 1600 | 1900 | 386 | 5500 | 1700 |
| NĂM 3 – 30.000 USERS (~2.000 Rx/tháng) |  |  |  |  |  |  |
| Phí thuê bao hàng tháng | 2000 | 2200 | 2800 | 560 | 7000 | 1800 |
| Phí usage/API (ước tính) | 200 | 400 | 700 | 100 | 2000 | 700 |
| Phí hỗ trợ kỹ thuật | 0 | 300 | 0 | 0 | 1000 | 500 |
| Tổng chi phí Năm 3 | 2200 | 2900 | 3500 | 660 | 10000 | 3000 |
| TỔNG TCO 3 NĂM | 8875 | 13700 | 12400 | 2514 | 76500 | 30600 |
| XẾP HẠNG CHI PHÍ | 2 | 4 | 3 | 1 | 6 | 5 |


---

## 6. Giải Thích Điểm Số


## GIẢI THÍCH CHI TIẾT ĐIỂM SỐ TỪNG VENDOR

| Vendor | Tiêu chí | Điểm | Giải thích |
|---|---|---|---|
| DoseSpot | HIPAA & Bảo mật | 9 | Surescripts + ONC certified. Có BAA sẵn sàng. HIPAA compliant đầy đủ. SOC 2. Không có lịch sử vi phạm bảo mật. |
|  | API & Tích hợp | 9 | API mở toàn bộ (~250 endpoints), không tính phí thêm. Tích hợp Jumpstart nhanh (vài ngày). Full integration cho custom app. Hỗ trợ Python/REST/Webhook. |
|  | Chi phí & TCO | 8 | ~$525/tháng cho 500 Rx. Giá cạnh tranh so với thị trường. EPCS thêm ~$100/năm/prescriber. Không có hidden fee. |
|  | Kinh nghiệm Healthcare | 9 | 300+ clients telehealth/EHR. Chuyên telemedicine. Có case study rõ ràng trong lĩnh vực này. |
|  | Độ tin cậy & Uptime | 8 | Uptime 99.9%. Đôi khi API chậm (ghi nhận từ user), nhưng rất hiếm. SLA tốt. |
|  | Khả năng mở rộng | 8 | Hỗ trợ EPCS, multi-state 50 states, không giới hạn số Rx. Phù hợp scale lên 30K users. |
|  | Hỗ trợ & Dịch vụ | 8 | Dedicated Account Manager + Technical Resource. Hỗ trợ trong suốt quá trình chứng nhận Surescripts. |
| DrFirst | HIPAA & Bảo mật | 9 | HIPAA compliant đầy đủ. BAA có sẵn. DEA compliant. Phù hợp enterprise. |
|  | API & Tích hợp | 7 | Tích hợp tốt với EHR lớn (Epic, Cerner), nhưng API ít linh hoạt hơn cho custom app. Documentation trung bình. |
|  | Chi phí & TCO | 5 | Giá cao ~$799/feature/year. Phù hợp enterprise hơn startup. TCO cao nhất nhóm. |
|  | Kinh nghiệm Healthcare | 8 | 25+ năm kinh nghiệm. Phổ biến ở bệnh viện và hệ thống lớn. Ít tập trung vào startup telemedicine. |
|  | Độ tin cậy & Uptime | 8 | Uptime cao. Phổ biến trong hệ thống bệnh viện lớn. SLA tốt. |
|  | Khả năng mở rộng | 8 | Hỗ trợ quy mô lớn, multi-state, EPCS đầy đủ. |
|  | Hỗ trợ & Dịch vụ | 7 | Support tốt nhưng response time đôi khi chậm theo đánh giá người dùng. |
| Truepill | HIPAA & Bảo mật | 9 | Là Covered Entity theo HIPAA. BAA chuẩn có sẵn. TLS 1.2, mã hóa toàn bộ. Không có vi phạm ghi nhận. |
|  | API & Tích hợp | 8 | RESTful API đầy đủ: pharmacy + telehealth. Webhook hỗ trợ. Tích hợp với Surescripts. Tài liệu tốt. |
|  | Chi phí & TCO | 7 | Giá custom quote, không public. Phụ thuộc vào volume. Thường cao hơn DoseSpot cho startup nhỏ. |
|  | Kinh nghiệm Healthcare | 8 | Phục vụ Hims, Ro, Done – các telemedicine startup lớn. D2C pharmacy kinh nghiệm cao. |
|  | Độ tin cậy & Uptime | 8 | Quy mô lớn, 100K+ orders/ngày. Cơ sở hạ tầng mạnh. |
|  | Khả năng mở rộng | 9 | 50 states coverage. Giao thuốc tận nhà. Kiểm soát chất (Schedule II–V). EPCS đầy đủ. |
|  | Hỗ trợ & Dịch vụ | 8 | Support team chuyên nghiệp. Phù hợp cho telemedicine platform. |
| MDToolbox | HIPAA & Bảo mật | 8 | HIPAA compliant, Surescripts certified. BAA có sẵn. Ít thông tin về SOC 2. |
|  | API & Tích hợp | 5 | Theo đánh giá, KHÔNG có API tích hợp cho custom app. Tốt hơn khi dùng standalone. |
|  | Chi phí & TCO | 9 | Rẻ nhất: $28–$38/tháng/prescriber. Phù hợp ngân sách startup nhỏ. |
|  | Kinh nghiệm Healthcare | 6 | Chủ yếu phục vụ phòng khám nhỏ và vừa. Ít case study telemedicine lớn. |
|  | Độ tin cậy & Uptime | 7 | Ổn định nhưng có báo cáo login delays và đôi khi không load được. |
|  | Khả năng mở rộng | 6 | Giới hạn cho custom telemedicine app. Ít phù hợp scale lớn. |
|  | Hỗ trợ & Dịch vụ | 7 | Support qua phone/email. Phù hợp cho phòng khám nhỏ. |
| Surescripts | HIPAA & Bảo mật | 9 | Chuẩn mực ngành. HIPAA đầy đủ. Tuy nhiên, tích hợp trực tiếp đòi hỏi quy trình chứng nhận phức tạp. |
|  | API & Tích hợp | 4 | Tích hợp trực tiếp rất phức tạp. Cần qua quá trình chứng nhận dài. Không phù hợp startup 3–5 tháng go-live. |
|  | Chi phí & TCO | 3 | Chi phí cao nhất nhóm. Setup/certification tốn nhiều thời gian và tiền bạc. |
|  | Kinh nghiệm Healthcare | 10 | Mạng lưới lớn nhất Mỹ: 85% Rx quốc gia. Tuy nhiên quá lớn cho startup. |
|  | Độ tin cậy & Uptime | 9 | Hạ tầng quốc gia. SLA cao nhất. |
|  | Khả năng mở rộng | 9 | Quy mô không giới hạn. Phù hợp cho enterprise. |
|  | Hỗ trợ & Dịch vụ | 6 | Hỗ trợ tốt nhưng chủ yếu cho enterprise. Ít tập trung vào startup. |
| Rcopia (Allscripts) | HIPAA & Bảo mật | 8 | HIPAA compliant. BAA có sẵn. Phù hợp enterprise. |
|  | API & Tích hợp | 5 | Tích hợp phức tạp. Hướng đến EHR lớn. Ít tài liệu cho custom integration. |
|  | Chi phí & TCO | 4 | Chi phí cao, không phù hợp startup. |
|  | Kinh nghiệm Healthcare | 7 | Allscripts có kinh nghiệm lâu năm nhưng tập trung enterprise. |
|  | Độ tin cậy & Uptime | 7 | Ổn định nhưng phức tạp trong vận hành. |
|  | Khả năng mở rộng | 6 | Có thể mở rộng nhưng cần đầu tư lớn. |
|  | Hỗ trợ & Dịch vụ | 6 | Support hướng đến enterprise, ít phù hợp startup. |


---

## 7. Phân Tích Rủi Ro


## PHÂN TÍCH RỦI RO – VENDOR DƯỢC

| Vendor | Mức rủi ro | Rủi ro chính | Biện pháp giảm thiểu |
|---|---|---|---|
| DoseSpot | THẤP | • API đôi khi chậm theo báo cáo user<br>• Chi phí tăng khi volume Rx tăng cao | • Implement timeout/retry logic trong code<br>• Đàm phán volume pricing khi đạt 1000+ Rx/tháng<br>• Monitor API latency bằng DataDog/CloudWatch |
| Truepill | THẤP-TRUNG | • Giá custom quote, khó dự đoán TCO chính xác<br>• Phụ thuộc vào Truepill cho toàn bộ pharmacy chain | • Yêu cầu quote chi tiết và lock pricing trong hợp đồng 1-2 năm<br>• Xây dựng phương án dự phòng với local pharmacy network |
| DrFirst | TRUNG | • Chi phí cao không phù hợp startup<br>• Timeline tích hợp dài<br>• Hướng đến enterprise, ít hỗ trợ startup | • Chỉ xem xét nếu có yêu cầu đặc biệt từ đối tác bệnh viện<br>• Đàm phán gói startup với chiết khấu |
| MDToolbox | TRUNG-CAO | • KHÔNG có API tích hợp cho custom app<br>• Không phù hợp Flutter mobile integration<br>• Giới hạn khả năng mở rộng | • KHÔNG NÊN chọn cho Compass Vitals<br>• Chỉ phù hợp nếu dùng standalone, không tích hợp |
| Surescripts (trực tiếp) | CAO | • Quá phức tạp cho startup<br>• Timeline chứng nhận 6-12 tháng (vượt go-live)<br>• Chi phí setup rất cao ($50K+) | • KHÔNG NÊN tích hợp trực tiếp<br>• Truy cập Surescripts gián tiếp qua DoseSpot hoặc DrFirst |
| Rcopia (Allscripts) | CAO | • Phức tạp, chi phí cao<br>• Ít tài liệu cho custom telemedicine startup<br>• Allscripts có lịch sử tranh chấp khách hàng | • KHÔNG NÊN chọn cho dự án này<br>• Chuyển sang DoseSpot hoặc Truepill |


---

## 8. Đề Xuất


## KHUYẾN NGHỊ VÀ ĐỀ XUẤT – VENDOR DƯỢC CHO COMPASS VITALS


### 🥇 KHUYẾN NGHỊ CHÍNH: DoseSpot

| Điểm có trọng số | 8.45 / 10 (Xếp hạng #1) |
|---|---|
| Lý do chọn | • API tích hợp linh hoạt nhất cho custom telemedicine platform (Python + Flutter)<br>• Surescripts & ONC certified – chuẩn y tế Mỹ<br>• Thời gian onboarding nhanh (Jumpstart: vài ngày)<br>• Giá cạnh tranh: ~$525/tháng cho 500 Rx<br>• 300+ khách hàng telemedicine, nhiều case study<br>• DoseSpot hỗ trợ toàn bộ quá trình chứng nhận Surescripts<br>• Phù hợp timeline go-live 3–5 tháng |
| Ưu điểm nổi bật | E-prescribing chuyên cho telehealth, API mở 100%, EPCS hỗ trợ, không hidden fee |
| Lưu ý triển khai | Bắt đầu bằng Jumpstart Integration để ra mắt nhanh, sau đó nâng cấp lên Full Integration |
| 🥈 PHƯƠNG ÁN THAY THẾ: Truepill |  |
| Điểm có trọng số | 8.20 / 10 (Xếp hạng #2) |
| Khi nào nên chọn Truepill thay DoseSpot? | • Khi CV cần pharmacy fulfillment end-to-end (giao thuốc tận nhà cho bệnh nhân)<br>• Khi cần tích hợp D2C pharmacy model<br>• Khi đã có provider network riêng và chỉ cần pharmacy backend<br>• Khi muốn brand riêng trên packaging thuốc |
| Điểm cần lưu ý | Giá không công khai – cần đàm phán. Phù hợp hơn sau khi đạt 1000+ users. |
| ❌ KHÔNG KHUYẾN NGHỊ |  |
| Surescripts trực tiếp | Quá phức tạp, tốn kém, timeline chứng nhận 6-12 tháng – vượt go-live |
| Rcopia (Allscripts) | Hướng đến enterprise, phức tạp, chi phí cao, ít phù hợp startup |
| MDToolbox (tích hợp API) | KHÔNG có API tích hợp cho custom app – không dùng được cho CV |
| TIÊU CHÍ QUYẾT ĐỊNH CUỐI CÙNG |  |
| Nếu cần tích hợp e-prescribing vào app (khuyến nghị) | → Chọn DoseSpot (Jumpstart trước, Full Integration sau) |
| Nếu cần giao thuốc tận nhà cho bệnh nhân | → Xem xét thêm Truepill bên cạnh DoseSpot |
| Nếu ngân sách rất hạn chế, ít Rx (<100/tháng) | → MDToolbox standalone (nhưng phải thay đổi architecture) |
| Nếu muốn mạng lưới quốc gia nhưng không trực tiếp | → DoseSpot (đã kết nối Surescripts, đơn giản hơn nhiều) |


---

## 9. Hành Động Tiếp Theo


## LỘ TRÌNH TRIỂN KHAI – VENDOR DƯỢC (DOSESPOT)

| Tuần | Hành động | Chi tiết công việc | Người chịu trách nhiệm |
|---|---|---|---|
| Tuần 1 | Liên hệ & Đàm phán | • Liên hệ DoseSpot sales team<br>• Yêu cầu demo + pricing quote<br>• Xem xét điều khoản BAA<br>• Đăng ký tài khoản Sandbox | Tech Lead + BD |
| Tuần 1-2 | Ký kết hợp đồng | • Ký BAA với DoseSpot<br>• Xác nhận gói dịch vụ (Jumpstart hoặc Full)<br>• Thanh toán phí setup<br>• Nhận API credentials | Legal + Tech Lead |
| Tuần 2-4 | Tích hợp Jumpstart | • Tích hợp DoseSpot Jumpstart (iFrame) vào Flutter app<br>• Implement Python backend endpoints<br>• Test các flow: tạo Rx, gửi pharmacy, EPCS<br>• Implement webhook handlers | Backend Dev + Mobile Dev |
| Tuần 4-6 | Chứng nhận Surescripts | • Phối hợp với DoseSpot team qua các buổi certification meeting<br>• Hoàn thành Surescripts certification<br>• Hoàn thành ONC/Drummond certification<br>• Test EPCS với DEA compliance | Tech Lead + DoseSpot Team |
| Tuần 6-8 | UAT & Go-live | • User Acceptance Testing với bác sĩ thử nghiệm<br>• Kiểm tra PDMP integration (nếu cần)<br>• Triển khai production<br>• Monitor và alert setup | QA + Tech Lead |
| Tháng 3-6 | Tối ưu & Mở rộng | • Theo dõi usage và chi phí<br>• Xem xét chuyển sang Full Integration khi cần<br>• Đàm phán volume pricing<br>• Đánh giá thêm Truepill cho delivery | Tech Lead + Finance |
| METRICS THÀNH CÔNG |  |  |  |
| Tích hợp API | DoseSpot API connected và test thành công trong Sandbox trong 2 tuần |  |  |
| Chứng nhận | Surescripts & ONC certification hoàn thành trước go-live |  |  |
| Go-live | Bác sĩ có thể e-prescribe thành công trong vòng 4-6 tuần từ ngày ký hợp đồng |  |  |
| Hiệu suất | API response time < 2 giây cho 95% requests |  |  |
| Tuân thủ | BAA được ký kết, audit logs đầy đủ, EPCS hoạt động |  |  |
| Chi phí | TCO Năm 1 không vượt $15.000 cho DoseSpot |  |  |


---

## 10. Tiêu Chí & Trọng Số


## TIÊU CHÍ ĐÁNH GIÁ VÀ LÝ DO TRỌNG SỐ

| Tiêu chí | Trọng số | Lý do chọn trọng số này |
|---|---|---|
| 1. Tuân thủ HIPAA & Bảo mật | 25% | Yêu cầu bắt buộc (table stakes). Thiếu BAA hoặc vi phạm HIPAA có thể dẫn đến phạt đến $1.5M và đình chỉ hoạt động. Đây là điều kiện tiên quyết cho nền tảng telemedicine. |
| 2. Chất lượng API & Tích hợp | 25% | CV sử dụng Python backend + Flutter app – cần API linh hoạt, tài liệu tốt và thời gian tích hợp ngắn. Timeline 3–5 tháng đòi hỏi API dễ tích hợp. |
| 3. Chi phí & TCO 3 năm | 20% | Ngân sách là ưu tiên cao. Cần tối ưu chi phí cho 30.000 users. Tránh hidden fees và chi phí scale đột ngột. |
| 4. Kinh nghiệm Healthcare | 10% | Vendors có kinh nghiệm telemedicine sẽ giảm rủi ro implementation. Chứng chỉ Surescripts và case study thực tế là chỉ số quan trọng. |
| 5. Độ tin cậy & Uptime | 10% | E-prescribing là critical functionality – downtime ảnh hưởng trực tiếp đến bệnh nhân và uy tín của CV. SLA ≥ 99.9% là yêu cầu tối thiểu. |
| 6. Khả năng mở rộng | 5% | Cần hỗ trợ EPCS, 50 states, tăng volume Rx từ 500 lên 2000/tháng trong 3 năm. Quan trọng nhưng không phải ưu tiên ngay. |
| 7. Hỗ trợ & Dịch vụ | 5% | Hỗ trợ kỹ thuật trong quá trình chứng nhận Surescripts quan trọng. Sau đó ít critical hơn vì có tài liệu API tốt. |
| TỔNG | 100% |  |


