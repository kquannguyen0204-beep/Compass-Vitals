# BO_TongChiPhiVendor

---

## Chi Phí Năm 1


## BÁO CÁO CHI PHÍ VENDOR - NĂM ĐẦU TIÊN


### Compass Vitals (CV) \| Telemedicine Platform \| ~5,000 users năm đầu → 30,000 users/3 năm \| HIPAA Compliant

| Dịch Vụ | Vendor Đề Xuất | Chi Phí Set Up<br>(USD) | Chi Phí Vận Hành<br>Năm 1 (USD) | Chi Phí Khác<br>(USD) | Tổng Chi Phí<br>Năm 1 (USD) | Ghi Chú |
|---|---|---|---|---|---|---|
| Video Calling | Daily.co | 0 | 20400 | 0 | 20400 | Chi phí Năm 1 = $20,400 (theo TCO báo cáo gốc). HIPAA add-on bắt buộc $500/tháng = $6,000/năm. Graduated pricing khi scale. Flutter SDK chính thức, BAA included. |
| SMS | Twilio | 0 | 13994 | 300 | 14294 | Chi phí SMS Năm 1 = $13,994: SMS $13,080 + phone number $14 + BAA/Enterprise tier $600 + 10DLC registration $300 (one-time). Giả định 5,000 users × 20 SMS/tháng = 1.2M SMS/năm. |
| Email | Paubox | 0 | 1688 | 0 | 1688 | Chi phí Năm 1 = $1,688: API/Gửi email $1,188 ($99/tháng × 12) + Chi phí tích hợp Dev $500 (one-time). HIPAA-native end-to-end encryption, BAA included. Giả định 5,000 users × 35 email/tháng = 175,000 emails/tháng = 2,100,000 emails/năm. |
| Fax | Documo | 500 | 2880 | 0 | 3380 | Setup $500. Chi phí vận hành $2,880/năm (Business API ~$0.008/page × 300,000 trang/năm). Năm 1: 5,000 users × 5 trang fax/tháng × 12 = 300,000 trang. API-first, HIPAA compliant, BAA available. |
| Payment Processing | Square | 0 | 18720 | 0 | 18720 | Chi phí Năm 1 = $18,720 (theo TCO báo cáo gốc). Ước tính ~2.6% + $0.10/giao dịch, ~1 giao dịch/user/tháng × $50 trung bình × 5,000 users. HIPAA compliant, BAA available, Flutter SDK chính thức. |
| E-Prescribing | DoseSpot | 5000 | 6300 | 0 | 11300 | Tổng phí setup $5,000: Onboarding $2,000 + Surescripts $1,500 + EPCS $1,500. Phí vận hành: $525/tháng × 12 = $6,300/năm (~500 Rx/tháng, Năm 1). HIPAA-certified, BAA available, EPCS compliant. |
| Lab Ordering | Health Gorilla | 2000 | 18000 | 0 | 20000 | Setup $2,000 (API onboarding fee). Chi phí vận hành $18,000/năm (~500 orders/tháng × per-API-call fee, Năm 1 – theo TCO analysis). HIPAA compliant, BAA available, FHIR-based API, extensive lab network. Lưu ý: Junction là lựa chọn thay thế với setup $1,500 và Flutter SDK native. |
| Cloud PACS / Imaging | Medicai | 500 | 6000 | 0 | 6500 | Setup $500. Chi phí vận hành $6,000/năm (Starter ~$500/tháng × 12, scale theo imaging studies). Giả định 2-5 studies/user/tháng. HIPAA compliant, cloud-native PACS, BAA available. |
| Hạ Tầng Cloud (AWS) | Amazon Web Services | 0 | 4284 | 300 | 4584 | EC2 2×t3.medium ~$120/th, RDS PostgreSQL db.t3.medium Multi-AZ ~$100/th, S3 500GB ~$12/th, ALB+CloudFront ~$30/th, NAT Gateway ~$35/th, CloudWatch+Logs ~$20/th, WAF+KMS+Secrets Manager ~$40/th → ~$357/tháng × 12 = $4,284/năm. Chi phí khác: AWS Support (Developer Plan) $300/năm. Không có setup fee — chi phí triển khai là nội bộ. HIPAA: sử dụng AWS HIPAA-eligible services, cần ký BAA với AWS (miễn phí). |
| TỔNG CỘNG |  | 8000 | 92266 | 600 | 100866 | Ước tính năm đầu. Thực tế phụ thuộc vào volume sử dụng và thương lượng hợp đồng. |
| LƯU Ý QUAN TRỌNG |  |  |  |  |  |  |
| • Chi phí trên là ước tính dựa trên pricing công khai tính đến Q1 2026. Cần xác nhận lại với vendor trước khi ký hợp đồng. |  |  |  |  |  |  |
| • Số liệu vận hành tính cho ~5,000 users năm đầu. Chi phí sẽ tăng tuyến tính khi scale lên 30,000 users. |  |  |  |  |  |  |
| • Chi phí giao dịch (Square, Health Gorilla) phụ thuộc hoàn toàn vào volume — cần dự báo số lượng giao dịch thực tế. |  |  |  |  |  |  |
| • DoseSpot và Health Gorilla thường có thương lượng giá enterprise — khuyến nghị thương lượng trực tiếp để giảm chi phí. |  |  |  |  |  |  |
| • Tất cả vendor đã ký BAA (Business Associate Agreement) — đáp ứng yêu cầu HIPAA bắt buộc của dự án. |  |  |  |  |  |  |


