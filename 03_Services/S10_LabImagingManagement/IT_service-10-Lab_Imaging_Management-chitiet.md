# SERVICE 10 (Lab_Imaging_Management): LAB & IMAGING MANAGEMENT (Quản lý Xét nghiệm & Chẩn đoán Hình ảnh) — BẢN CHI TIẾT

## Thông tin chung

- **Mô tả:** Quy trình quản lý toàn bộ vòng đời xét nghiệm (Lab) và chẩn đoán hình ảnh (Imaging) — từ tạo đơn, tìm đối tác, thông báo KH, nhận kết quả, đến review và follow-up
- **INPUT Lab:** KH có nhu cầu xét nghiệm (từ tư vấn y tế, theo dõi bệnh lý, hoặc kiểm tra định kỳ)
- **INPUT Imaging:** KH có nhu cầu chẩn đoán hình ảnh (từ tư vấn y tế, theo dõi bệnh lý, hoặc kiểm tra sức khỏe)
- **OUTPUT:** Kết quả được tích hợp vào EHR, sẵn sàng cho BS và KH xem
- **Target User:** Bác sĩ (MD), Khách hàng (KH), Customer Service (CS)
- **Duration:** Lab: 3-7 ngày / Imaging: 5-14 ngày / STAT: < 24 giờ
- **Compliance:** HIPAA + JCI AOP.6 + USPSTF Guidelines

## Nguyên tắc cốt lõi

- **Partner Network:** CHỈ sử dụng Lab/Imaging Center đã ký hợp đồng BAA với CVH
- **Tìm đối tác tự động** theo ưu tiên: (1) Bảo hiểm, (2) Vị trí địa lý, (3) Giá cả
- **HIPAA Compliance:** Tất cả dữ liệu y tế được mã hóa và bảo mật
- **Critical Values Protocol:** Giá trị bất thường được thông báo ngay lập tức cho BS qua cuộc gọi điện
- **Chia nhánh Lab/Imaging:** Mỗi flow xác định rõ điểm khác biệt giữa Lab và Imaging

> ⚠️ **ĐIỂM KHÁC BIỆT QUAN TRỌNG:**
> - **Lab:** Gửi đơn tự động qua kênh tích hợp điện tử | Nhận kết quả tự động qua HL7/FHIR
> - **Imaging:** CS gửi đơn thủ công qua FAX/email có mã hóa | CS nhận kết quả (fax/email/CD) và upload thủ công

---

## Danh sách luồng

- **FLOW-01:** Tạo đơn yêu cầu (Lab + Imaging)
- **FLOW-02:** Tìm đối tác và gửi yêu cầu (Lab + Imaging)
- **FLOW-03:** Thông báo và hướng dẫn khách hàng (Lab + Imaging)
- **FLOW-04:** Nhận kết quả (Lab + Imaging)
- **FLOW-05:** Review kết quả và Follow-up (Lab + Imaging)
- **STAT Flow:** Quy trình xét nghiệm/hình ảnh khẩn cấp

> Giữa FLOW-03 và FLOW-04 có quy trình bên đối tác (Lab PSC / Imaging Center) — không phải CVH touchpoint

---

## FLOW-01: Tạo đơn yêu cầu

**1. BS — Xác định loại dịch vụ**
- ⛔ Lab (Xét nghiệm) → Nhánh LAB | Imaging (Chẩn đoán hình ảnh) → Nhánh IMAGING

| Điều kiện | Hành động |
|-----------|-----------|
| BS chọn Lab | Chuyển sang Nhánh LAB (Bước 2L) |
| BS chọn Imaging | Chuyển sang Nhánh IMAGING (Bước 2I) |

### Nhánh LAB

**2L. BS/Hệ thống — Xác định nhu cầu xét nghiệm**
- ⛔ 3 nguồn phát sinh:
  - Nhánh A: Sau tư vấn y tế → BS tạo đơn trực tiếp
  - Nhánh B: Protocol quản lý bệnh → Hệ thống tự động tạo đơn → BS phê duyệt
  - Nhánh C: KH yêu cầu kiểm tra định kỳ → BS review và phê duyệt

| Điều kiện | Hành động |
|-----------|-----------|
| Nhánh A: Sau tư vấn y tế | BS mở form, chọn test panel, nhập chỉ định, xác nhận |
| Nhánh B: Protocol trigger | Hệ thống tạo đơn tự động → BS review và phê duyệt |
| Nhánh C: KH yêu cầu định kỳ | KH gửi yêu cầu qua app → BS phê duyệt và tạo đơn |

**Nhánh A — Sau tư vấn y tế:**

| Bước | Đối tượng | Mô tả |
|------|-----------|-------|
| 1A | BS | Sau video visit hoặc chat, BS xác định KH cần xét nghiệm |
| 2A | BS | Click "Tạo đơn xét nghiệm" → Form hiện ra với thông tin KH đã điền sẵn |
| 3A | BS | Chọn test panels cần thiết từ danh sách |
| 4A | BS | Ghi lý do xét nghiệm (VD: "Diabetes monitoring", "Annual checkup") |
| 5A | Hệ thống | Tạo đơn, hiển thị xác nhận: "Đơn xét nghiệm đã được tạo" |

**Nhánh B — Từ chương trình quản lý bệnh:**

| Bước | Đối tượng | Mô tả |
|------|-----------|-------|
| 1B | Hệ thống | Phát hiện KH đến lịch xét nghiệm theo protocol (VD: A1C mỗi 3 tháng cho KH tiểu đường) |
| 2B | Hệ thống | Tự động tạo đơn xét nghiệm theo protocol đã cấu hình |
| 3B | Hệ thống | BS nhận thông báo để review và phê duyệt |
| 4B | BS | Review và phê duyệt (hoặc chỉnh sửa nếu cần) → Đơn được kích hoạt |

**Nhánh C — Yêu cầu kiểm tra định kỳ:**

| Bước | Đối tượng | Mô tả |
|------|-----------|-------|
| 1C | KH | Gửi yêu cầu xét nghiệm định kỳ qua app |
| 2C | Hệ thống | Tiếp nhận và gửi thông báo cho BS/Care Coordinator |
| 3C | BS | Xem xét yêu cầu, chọn test panel phù hợp, tạo đơn |
| 4C | Hệ thống | KH nhận thông báo: "Yêu cầu xét nghiệm đã được phê duyệt" |

**3L. BS — Chọn loại xét nghiệm**
- Chọn test panel: BMP, CMP, CBC, Lipid Panel, A1C, Thyroid, Urinalysis...

**Danh sách Test Panel phổ biến (LAB):**

| Test Panel | Mã code | Mô tả |
|------------|---------|-------|
| Basic Metabolic Panel (BMP) | 80048 | Na, K, Cl, CO2, BUN, Creatinine, Glucose |
| Comprehensive Metabolic (CMP) | 80053 | BMP + Liver enzymes, Albumin, Total Protein |
| Complete Blood Count (CBC) | 85025 | WBC, RBC, Hemoglobin, Hematocrit, Platelets |
| Lipid Panel | 80061 | Total Cholesterol, LDL, HDL, Triglycerides |
| Hemoglobin A1C | 83036 | Theo dõi đường huyết dài hạn |
| Thyroid Panel (TSH, T4, T3) | 84443 | Đánh giá chức năng tuyến giáp |
| Urinalysis | 81003 | Phân tích nước tiểu |

**4L. BS — Nhập thông tin chỉ định**
- Clinical indication + Độ ưu tiên (Routine/Urgent/STAT) + Hướng dẫn đặc biệt
- ⚠️ Hệ thống validate real-time, reject nếu thiếu trường bắt buộc

- INPUT: patient_id (String, required), test_panels (Array, required — min 1 item), clinical_indication (String, required), priority (Enum: Routine/Urgent/STAT, required), special_instructions (String, optional)
- OUTPUT: lab_order_id (String — VD: LAB-20260130-00001), order_created (Boolean)

**VALIDATION:**

| Field | Rule | Message lỗi |
|-------|------|-------------|
| test_panels | Bắt buộc, ít nhất 1 item | "Vui lòng chọn ít nhất 1 loại xét nghiệm" |
| clinical_indication | Bắt buộc | "Vui lòng nhập chỉ định lâm sàng" |
| priority | Bắt buộc | "Vui lòng chọn độ ưu tiên" |

**UI STATES:**

| State | Hiển thị |
|-------|----------|
| Default | Form trống, Button disabled |
| Valid | Ít nhất 1 test + indication → Button enabled |
| Loading | Button hiện spinner "Đang tạo đơn..." |
| Success | Toast "Đơn xét nghiệm đã được tạo" + Redirect |
| Error | Toast lỗi với message cụ thể |

**WIREFRAME — Màn hình Tạo đơn xét nghiệm (BS View):**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              TẠO ĐƠN XÉT NGHIỆM                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  👤 THÔNG TIN BỆNH NHÂN                                         │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Họ tên: Nguyễn Văn A                                        ││
│  │ DOB: 15/03/1985     MRN: CVH-20260115-00001                 ││
│  │ Insurance: BlueCross BlueShield (In-network)                ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  🧪 CHỌN LOẠI XÉT NGHIỆM                                        │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ☐ Basic Metabolic Panel (BMP)                               ││
│  │ ☐ Comprehensive Metabolic Panel (CMP)                       ││
│  │ ☑ Complete Blood Count (CBC)                                ││
│  │ ☑ Lipid Panel                                               ││
│  │ ☐ Hemoglobin A1C                                            ││
│  │ ☐ Thyroid Panel (TSH, T4, T3)                               ││
│  │ ☐ Urinalysis                                                ││
│  │ [+ Thêm xét nghiệm khác]                                    ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📋 CHỈ ĐỊNH LÂM SÀNG *                                         │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Annual wellness exam - screening for cardiovascular risk    ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ⏰ ĐỘ ƯU TIÊN                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ (●) Routine    ( ) Urgent    ( ) STAT                       ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📝 HƯỚNG DẪN ĐẶC BIỆT (Không bắt buộc)                         │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Nhịn đói 8-12 giờ trước xét nghiệm                          ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    TẠO ĐƠN XÉT NGHIỆM                       ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**5L. Hệ thống — Xác nhận và tạo đơn**
- Tạo Lab Order ID (VD: LAB-20260130-00001), lưu vào EHR

### Nhánh IMAGING

**2I. BS — Chọn loại CĐHA và vùng chụp**
- Loại: X-ray, CT, MRI, Ultrasound, Mammography, DEXA + Body part

**Danh sách loại CĐHA hỗ trợ:**

| Loại CĐHA | Mã code | Mô tả | Chuẩn bị đặc biệt |
|-----------|---------|-------|-------------------|
| X-ray | XRAY | Chụp X-quang các vùng cơ thể | Không |
| CT Scan | CT | Chụp cắt lớp vi tính | Nhịn ăn nếu có contrast |
| CT with Contrast | CT_C | CT có chất tương phản | Nhịn ăn 6h, kiểm tra creatinine |
| MRI | MRI | Chụp cộng hưởng từ | Tháo kim loại |
| MRI with Contrast | MRI_C | MRI có chất tương phản | Như MRI + kiểm tra dị ứng |
| Ultrasound | US | Siêu âm | Tùy vùng (bụng: nhịn ăn) |
| Mammography | MAMMO | Chụp X-quang tuyến vú | Không dùng deodorant |
| DEXA Scan | DEXA | Đo mật độ xương | Không |

**3I. BS — Nhập thông tin lâm sàng**
- Lý do chỉ định + Mức độ ưu tiên (Routine/Urgent) + Có cần contrast không

- INPUT: patient_id (String, required), imaging_type (Enum: XRAY/CT/CT_C/MRI/MRI_C/US/MAMMO/DEXA, required), body_part (String, required), clinical_indication (String, required), priority (Enum: Routine/Urgent, required), contrast_required (Boolean, required), special_notes (String, optional)
- OUTPUT: imaging_order_id (String — VD: IMG-20260130-00001), order_created (Boolean)

> ⚠️ **LƯU Ý:** Nếu KH đang mang thai, hệ thống CHỈ cho phép tạo đơn Ultrasound. Các loại CĐHA có tia xạ (X-ray, CT) hoặc contrast PHẢI bị từ chối tự động.

**4I. BS — Kiểm tra contraindication (nếu cần)**
- Dị ứng contrast → Warning, BS phải confirm
- MRI + implant kim loại → Warning, suggest CT thay thế
- KH đang mang thai → Chỉ cho phép US, từ chối X-ray/CT/MRI có contrast
- Creatinine bất thường → Warning cho contrast

| Điều kiện | Hành động |
|-----------|-----------|
| KH có tiền sử dị ứng contrast | Warning hiển thị, BS phải confirm hoặc chọn không dùng contrast |
| KH có implant kim loại | Warning cho MRI, suggest CT thay thế nếu phù hợp |
| KH đang mang thai | Warning, chỉ cho phép US, từ chối X-ray/CT/MRI có contrast |
| Creatinine bất thường | Warning cho contrast, yêu cầu consult chuyên khoa nếu cần |

**WIREFRAME — Màn hình Tạo đơn CĐHA (BS View):**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]             TẠO ĐƠN CHẨN ĐOÁN HÌNH ẢNH                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  👤 BỆNH NHÂN: Nguyễn Văn A (CVH-20260115-00001)                │
│                                                                 │
│  🏥 LOẠI CĐHA *                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ CT Scan                                                 [▼]││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📍 VÙNG CHỤP *                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Chest (Ngực)                                            [▼]││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📋 LÝ DO CHỈ ĐỊNH *                                            │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Đánh giá phổi - Triệu chứng: ho kéo dài, khó thở.          ││
│  │ R/O: Lung mass, Pneumonia                                  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ⏰ MỨC ĐỘ ƯU TIÊN *                                            │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ (●) Routine (3-5 ngày)    ( ) Urgent (24-48h)              ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  💉 CẦN CHẤT TƯƠNG PHẢN? *                                      │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ( ) Không    (●) Có                                        ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ⚠️ LƯU Ý VỀ CONTRAST:                                         │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ • KH sẽ cần nhịn ăn 6 giờ trước chụp                       ││
│  │ • Kiểm tra tiền sử dị ứng: ☐ Không có dị ứng               ││
│  │ • Creatinine gần nhất: 0.9 mg/dL (15/01/2026) ✅           ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────┐  ┌─────────────────────────────────────────┐│
│  │      HỦY        │  │          XÁC NHẬN VÀ TẠO ĐƠN            ││
│  └─────────────────┘  └─────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**5I. Hệ thống — Xác nhận và tạo đơn**
- Tạo Imaging Order ID (VD: IMG-20260130-00001), lưu vào EHR

---

## FLOW-02: Tìm đối tác và gửi yêu cầu

**1. Hệ thống — Kiểm tra thông tin bảo hiểm**
- ⛔ Có bảo hiểm → Lọc đối tác in-network | Không có bảo hiểm → Hiển thị tất cả đối tác
- ⚠️ Không tìm thấy in-network trong bán kính 10 dặm → Auto mở rộng lên 25 dặm

- INPUT: lab_order_id hoặc imaging_order_id (String, required), patient_insurance (Object, optional), patient_address (String, required)
- OUTPUT: selected_lab hoặc selected_center (Object), lab_confirmation_status hoặc dispatch_status (Enum)

| Điều kiện | Hành động |
|-----------|-----------|
| Có bảo hiểm | Lọc đối tác in-network → Ưu tiên hiển thị |
| Không có bảo hiểm | Hiển thị tất cả đối tác với giá self-pay |
| Không tìm thấy in-network trong 10 dặm | Auto mở rộng bán kính lên 25 dặm + thông báo KH |
| Vẫn không có kết quả | Auto gửi alert: "Không tìm thấy lab phù hợp. CS đang hỗ trợ." |

**Tiêu chí tìm kiếm:**

| Ưu tiên | Tiêu chí | Mô tả |
|---------|----------|-------|
| 1 | **Bảo hiểm** | Lab/Center nằm trong mạng lưới bảo hiểm KH (in-network) |
| 2 | **Vị trí** | Khoảng cách từ địa chỉ KH, bán kính 15 dặm |
| 3 | **Giá cả** | Giá tốt nhất trong số các đối tác đủ điều kiện |
| Phụ | **Năng lực** | Có thiết bị và chứng chỉ thực hiện test cụ thể |
| Phụ | **Khả dụng** | Giờ hoạt động và appointment slots |
| Phụ | **Chất lượng** | Rating và turnaround time |

**2. Hệ thống — Tìm đối tác theo vị trí**
- Sắp xếp theo khoảng cách từ địa chỉ KH (bán kính 15 dặm)

**3. Hệ thống — So sánh giá cả**
- Hiển thị giá ước tính: Co-pay (in-network) hoặc Self-pay (cash price)

**4. Chọn đối tác và gửi yêu cầu**
- ⛔ **Lab:** Hệ thống tự động gửi qua kênh tích hợp điện tử → Lab confirm qua hệ thống
- ⛔ **Imaging:** CS gửi thủ công qua FAX/email có mã hóa → CS xác nhận đã gửi trong hệ thống → CS gọi điện Imaging Center xác nhận (BẮT BUỘC cho Urgent)
- ⚠️ Không tìm được đối tác → CS nhận alert, tìm kiếm thủ công

| Điều kiện | Hành động |
|-----------|-----------|
| Lab — gửi thành công | Hệ thống tự động gửi qua kênh tích hợp → Lab confirm → FLOW-03 |
| Imaging — gửi thành công | CS gửi FAX/email mã hóa → CS xác nhận "Đã gửi" → FLOW-03 |
| Lab reject yêu cầu | Tự động tìm Lab backup, thông báo KH |
| Imaging — FAX fail | CS thử gửi email, hoặc gọi trực tiếp |
| Không tìm được đối tác | CS nhận alert, tìm kiếm thủ công |

**Nhánh IMAGING — CS gửi yêu cầu thủ công:**

| Bước | Đối tượng | Mô tả |
|------|-----------|-------|
| 1 | Hệ thống | Tìm Imaging Center theo: (1) Bảo hiểm, (2) Vị trí, (3) Giá → Hiển thị danh sách gợi ý |
| 2 | CS | Xem danh sách gợi ý, chọn center phù hợp nhất |
| 3 | CS | In đơn CĐHA từ hệ thống (thông tin KH, loại CĐHA, chỉ định lâm sàng) |
| 4 | CS | Gửi đơn đến Imaging Center qua FAX/email có mã hóa (HIPAA compliant) |
| 5 | CS | Cập nhật trạng thái "Đã gửi", ghi chú: ngày giờ gửi, phương thức, người nhận |
| 6 | CS | Gọi điện Imaging Center xác nhận (BẮT BUỘC cho Urgent) |

**Thông tin cần gửi đến Imaging Center:**

| Mục | Bắt buộc | Mô tả |
|-----|----------|-------|
| Thông tin bệnh nhân | Có | Họ tên, DOB, giới tính, số điện thoại |
| Mã đơn CĐHA | Có | IMG Order ID (VD: IMG-20260130-00001) |
| Loại CĐHA + Vùng chụp | Có | VD: CT Chest with Contrast |
| Chỉ định lâm sàng | Có | Lý do chụp, triệu chứng, nghi ngờ bệnh lý |
| Thông tin bảo hiểm | Nếu có | Tên bảo hiểm, số thẻ, authorization number |
| Lưu ý đặc biệt | Nếu có | Dị ứng, thai kỳ, claustrophobia, v.v. |
| Thông tin BS chỉ định | Có | Tên BS, số điện thoại/email để gửi kết quả về |

**WIREFRAME — Màn hình CS gửi yêu cầu CĐHA (CS View):**

```
┌─────────────────────────────────────────────────────────────────┐
│  [CVH CS Portal]        GỬI YÊU CẦU CĐHA                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📋 THÔNG TIN ĐƠN CĐHA                                          │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Mã đơn: IMG-20260130-00001                                  ││
│  │ KH: Nguyễn Văn A          SĐT: 0901-234-567                 ││
│  │ Loại: CT Chest with Contrast                                ││
│  │ BS chỉ định: Dr. Trần Văn B                                 ││
│  │ Ưu tiên: 🟠 Urgent                                          ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  🏥 IMAGING CENTER ĐÃ CHỌN                                      │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Medic Center Q.1                                            ││
│  │ 123 Nguyễn Du, Q.1, TP.HCM                                  ││
│  │ FAX: (028) 3822-1234                                        ││
│  │ Email: order@mediccenter.vn                                 ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📤 PHƯƠNG THỨC GỬI *                                           │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ (●) FAX         ( ) Email có mã hóa                        ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📎 TÀI LIỆU GỬI ĐI                                             │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ☑ Đơn yêu cầu CĐHA (đã in)                                  ││
│  │ ☑ Thông tin bệnh nhân                                       ││
│  │ ☑ Thông tin bảo hiểm                                        ││
│  │ ☑ Chỉ định lâm sàng                                         ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📝 GHI CHÚ                                                     │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Đã gửi fax lúc 10:30 AM. Người nhận: Ms. Hoa               ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              ✅ XÁC NHẬN ĐÃ GỬI THÀNH CÔNG                  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**5. Xác nhận từ đối tác**
- ⛔ Confirm → FLOW-03 | Reject → Tự động tìm đối tác backup

**Prior Authorization Check (nếu cần):**
- Hệ thống tự động tra cứu yêu cầu Prior Auth qua API bảo hiểm (≤ 2 phút)
- ⛔ Auth approved → tiếp tục | Auth rejected → Alert MD cung cấp thêm clinical documentation
- ⚠️ Auth pending > 48h → CS follow up trực tiếp với payer

| Điều kiện | Hành động |
|-----------|-----------|
| Auth approved | Ghi nhận auth number vào order → tiếp tục |
| Auth rejected | Alert MD kèm lý do → MD cung cấp thêm documentation / thay thế order |
| Auth pending > 48h | CS follow up trực tiếp với payer |
| API payer không phản hồi | Auto retry 3 lần (cách 30 giây) |

---

## FLOW-03: Thông báo và hướng dẫn khách hàng

**1. Hệ thống — Tạo thông tin thông báo**
- Địa chỉ đối tác, giờ hoạt động, mã đơn, hướng dẫn chuẩn bị theo loại dịch vụ

- INPUT: lab_order_id hoặc imaging_order_id (String, required), selected_lab hoặc selected_center (Object, required), preparation_guide (Object, required)
- OUTPUT: notification_sent (Boolean), customer_acknowledged (Boolean)

**Nội dung thông báo theo loại dịch vụ:**

| Mục | LAB | IMAGING |
|-----|-----|---------|
| Thông tin đối tác | Tên Lab, địa chỉ, SĐT, giờ hoạt động, link Maps | Tên Center, địa chỉ, SĐT, giờ hoạt động, link Maps |
| Mã đơn | Lab Order ID | Imaging Order ID |
| Hướng dẫn chuẩn bị | Nhịn đói (nếu cần), ngừng thuốc, mang ID + insurance card | Nhịn ăn (CT/MRI contrast), tháo kim loại (MRI), không dùng deodorant (Mammography) |
| Lịch hẹn | Appointment time hoặc Walk-in window | Appointment time hoặc Walk-in window |
| Chi phí ước tính | Co-pay hoặc Self-pay price | Co-pay hoặc Self-pay price |

**Hướng dẫn chuẩn bị theo loại CĐHA:**

| Loại CĐHA | Hướng dẫn chuẩn bị |
|-----------|-------------------|
| X-ray | Không cần chuẩn bị đặc biệt. Tháo trang sức vùng chụp |
| CT without contrast | Không cần chuẩn bị đặc biệt |
| CT with contrast | Nhịn ăn 6 giờ trước chụp. Được uống nước lọc. Thông báo nếu có dị ứng |
| MRI | Tháo tất cả kim loại (trang sức, kẹp tóc, răng giả). Thông báo nếu có implant |
| MRI with contrast | Như MRI + nhịn ăn 4 giờ. Thông báo nếu có bệnh thận hoặc dị ứng |
| Ultrasound (bụng) | Nhịn ăn 6-8 giờ trước. Được uống nước lọc |
| Mammography | Không dùng deodorant, phấn, hoặc lotion vùng ngực và nách |

**2. Hệ thống — Gửi thông báo đa kênh (SMS + Email + Push)**
- Lab: Lab PSC address + Mã xét nghiệm + Hướng dẫn (nhịn ăn, ngừng thuốc...)
- Imaging: Imaging Center + Mã CĐHA + Hướng dẫn chuẩn bị theo loại
- ⚠️ Gửi fail sau 3 lần retry → CS gọi điện trực tiếp đọc hướng dẫn cho KH

| Điều kiện | Hành động |
|-----------|-----------|
| Gửi thành công | Log timestamp + delivery status vào database |
| Email bị trả lại | Auto retry qua SMS |
| SMS fail | Auto retry qua App notification |
| Tất cả kênh fail sau 3 lần | CS gọi điện trực tiếp đọc hướng dẫn cho KH |

**WIREFRAME — Màn hình Thông báo xét nghiệm (KH View - App):**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]            THÔNG TIN XÉT NGHIỆM                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🧪 ĐƠN XÉT NGHIỆM CỦA BẠN                                      │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Mã xét nghiệm: LAB-20260130-00001                           ││
│  │ Loại: CBC, Lipid Panel                                      ││
│  │ Trạng thái: Chờ thực hiện                                   ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📍 THÔNG TIN PHÒNG XÉT NGHIỆM                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Quest Diagnostics - Downtown                                ││
│  │ 123 Health Street, Suite 100                                ││
│  │ San Francisco, CA 94102                                     ││
│  │ 📞 (415) 555-0123                                           ││
│  │ 🕐 Thứ 2-6: 7:00 AM - 5:00 PM                               ││
│  │    Thứ 7: 8:00 AM - 12:00 PM                                ││
│  │ [🗺️ XEM BẢN ĐỒ]                                             ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📋 HƯỚNG DẪN CHUẨN BỊ                                          │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ⚠️ QUAN TRỌNG:                                              ││
│  │ • Nhịn đói 8-12 giờ trước xét nghiệm                        ││
│  │ • Được uống nước lọc                                        ││
│  │ • Mang theo: ID + Insurance card                            ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  💰 CHI PHÍ ƯỚC TÍNH                                            │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Co-pay: $15.00                                              ││
│  │ (Đã áp dụng bảo hiểm BlueCross BlueShield)                  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              XÁC NHẬN ĐÃ ĐỌC THÔNG TIN                      ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**3. KH — Xác nhận đã đọc**
- Click link xác nhận (logged trong hệ thống)

**Appointment Reminder:**
- Email: 3h trước | App: 30 phút trước + 15 phút trước
- KH không confirm + lịch sử no-show → CS gọi điện nhắc nhở

| Điều kiện | Hành động |
|-----------|-----------|
| 3h trước hẹn | Email reminder kèm hướng dẫn chuẩn bị tóm tắt |
| 30 phút trước hẹn | App notification: "Lịch hẹn còn 30 phút. [Tên Lab/Center] - [Địa chỉ]" |
| 15 phút trước hẹn | App notification: "Nhắc nhở: Lịch hẹn còn 15 phút." |
| KH không confirm sau reminder 30p | SMS: "Bạn có đến lịch hẹn hôm nay không? Phản hồi C/K. Hotline: 1900xxxx" |
| KH không confirm + lịch sử no-show | CS gọi điện nhắc nhở trực tiếp |

---

> **[Quy trình bên đối tác — External, không phải CVH touchpoint]**

**Quy trình tại LAB:**

```
┌─────────────────────────────────────────────────────────────────┐
│          QUY TRÌNH BÊN LAB (External - Reference Only)          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │  KH đến Lab │ →  │  Check-in   │ →  │  Xác minh   │          │
│  │     PSC     │    │  (mã/tên)   │    │  danh tính  │          │
│  └─────────────┘    └─────────────┘    └─────────────┘          │
│                                              │                  │
│                                              ▼                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │  Kết quả    │ ←  │  Xử lý &    │ ←  │  Lấy mẫu    │          │
│  │  phê duyệt  │    │  phân tích  │    │  (specimen) │          │
│  └─────────────┘    └─────────────┘    └─────────────┘          │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Lab gửi kết quả về CVH qua kênh tích hợp → FLOW-04     │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

| Giai đoạn | Mô tả | Thời gian |
|-----------|-------|-----------|
| Check-in | KH cung cấp mã xét nghiệm hoặc tên + DOB, xác minh danh tính | 5-15 phút |
| Lấy mẫu | Phlebotomist thu thập mẫu, gắn nhãn mã vạch | 10-20 phút |
| Xử lý & phân tích | Lab xử lý mẫu, chạy xét nghiệm, kiểm soát chất lượng | 1-7 ngày |
| Phê duyệt | Pathologist/Lab Director review và phê duyệt | Cùng ngày |

> Timeline: Routine (1-2 ngày), STAT (1-4 giờ), Send-out (3-7 ngày)

**Quy trình tại IMAGING CENTER:**

```
┌─────────────────────────────────────────────────────────────────┐
│        QUY TRÌNH TẠI IMAGING CENTER (External - Reference)      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │  KH đến     │ →  │  Check-in   │ →  │  Chuẩn bị   │          │
│  │  Center     │    │  (mã/tên)   │    │  (thay đồ)  │          │
│  └─────────────┘    └─────────────┘    └─────────────┘          │
│                                              │                  │
│                                              ▼                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │  Kết quả    │ ←  │ Radiologist │ ←  │  Thực hiện  │          │
│  │  phê duyệt  │    │ đọc kết quả │    │  chụp CĐHA  │          │
│  └─────────────┘    └─────────────┘    └─────────────┘          │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Center gửi kết quả về CVH (fax/email/CD) → FLOW-04     │    │
│  │  ⚠️ CS sẽ upload thủ công vào hệ thống                  │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

| Giai đoạn | Mô tả | Thời gian |
|-----------|-------|-----------|
| Check-in | KH cung cấp mã đơn CĐHA hoặc tên + DOB | 5-15 phút |
| Chuẩn bị | Thay đồ (nếu cần), tháo trang sức kim loại | 5-10 phút |
| Thực hiện chụp | Kỹ thuật viên thực hiện CĐHA theo yêu cầu | 15-60 phút |
| Đọc kết quả | Radiologist đọc và phân tích hình ảnh | 1-24 giờ |
| Phê duyệt & gửi | Radiologist ký duyệt, gửi về CVH qua fax/email/CD | Cùng ngày |

> Timeline: X-ray (15-20 phút), CT (20-40 phút), MRI (45-90 phút), US (20-30 phút)

**Xử lý trường hợp đặc biệt tại đối tác:**

| Trường hợp | Áp dụng | Xử lý |
|-----------|---------|-------|
| KH không có mã đơn | Lab+Imaging | Lookup bằng tên + DOB, hoặc gọi CVH support |
| KH đến sai địa chỉ | Lab+Imaging | Hướng dẫn đến đúng địa chỉ đã chỉ định |
| KH chưa chuẩn bị đúng | Lab+Imaging | Hướng dẫn quay lại sau khi chuẩn bị đúng |
| Critical value/finding | Lab+Imaging | Gọi điện ngay cho CVH physician trước khi release kết quả |
| KH có contraindication | Imaging | Từ chối thực hiện, liên hệ BS CVH để thay đổi order |

---

## FLOW-04: Nhận kết quả

**1. Nhận kết quả từ đối tác**
- ⛔ **Lab:** Hệ thống tự động nhận qua HL7/FHIR → Import vào EHR
- ⛔ **Imaging:** CS nhận từ center (fax/email/CD) → CS upload thủ công vào hệ thống
- ⚠️ Data parsing fail → Auto retry + yêu cầu partner gửi lại. Interface sập → CS nhận kết quả qua email secure (backup)

**Nhánh LAB — Nhận kết quả tự động:**

| Bước | Đối tượng | Mô tả |
|------|-----------|-------|
| 1 | Lab | Tự động gửi kết quả điện tử về CVH EHR qua kênh bảo mật tích hợp |
| 2 | Hệ thống | Nhận, validate, match với original order, import vào patient chart |
| 3 | Hệ thống | Gửi notification cho BS (Dashboard + Email) và KH (SMS + Email + Push) |
| 4 | BS + KH | Kết quả có trong EHR để BS review, KH xem trong Patient Portal |

- INPUT: test_results (Array, required), lab_order_id (String, required)
- OUTPUT: results_imported (Boolean), notifications_sent (Boolean)

**Nhánh IMAGING — CS upload kết quả thủ công:**

| Bước | Đối tượng | Mô tả |
|------|-----------|-------|
| 1 | CS | Nhận kết quả từ Imaging Center qua: fax, email có mã hóa, CD/DVD, hoặc patient portal |
| 2 | CS | Verify: đúng bệnh nhân, đúng mã đơn, đầy đủ thông tin (báo cáo + hình ảnh) |
| 3 | CS | Upload file vào CVH EHR: PDF báo cáo + hình ảnh DICOM (nếu có). Gắn với IMG Order ID gốc |
| 4 | Hệ thống | Validate file, match với original order, import vào patient chart |
| 5 | Hệ thống | Gửi notification cho BS (Dashboard + Email) và KH (SMS + Email + Push) |
| 6 | BS + KH | BS xem trong EHR (hình ảnh + PDF), KH xem PDF summary trong Patient Portal |

- INPUT: radiologist_report (File/PDF, required), dicom_images (File, optional), imaging_order_id (String, required)
- OUTPUT: results_imported (Boolean), notifications_sent (Boolean)

**WIREFRAME — Màn hình CS upload kết quả CĐHA (CS View):**

```
┌─────────────────────────────────────────────────────────────────┐
│  [CVH CS Portal]        UPLOAD KẾT QUẢ CĐHA                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📋 THÔNG TIN ĐƠN CĐHA                                          │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Mã đơn: IMG-20260130-00001                                  ││
│  │ KH: Nguyễn Văn A          MRN: CVH-20260115-00001           ││
│  │ Loại: CT Chest with Contrast                                ││
│  │ Trạng thái: ⏳ Chờ kết quả                                  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📥 NGUỒN NHẬN KẾT QUẢ *                                        │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ (●) FAX    ( ) Email    ( ) CD/DVD    ( ) Patient Portal   ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📎 UPLOAD FILE KẾT QUẢ                                         │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  📄 PDF Báo cáo (Bắt buộc)                                  ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │  [📤 CHỌN FILE]  CT_Report_NguyenVanA.pdf (2.3 MB) ✅ │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  │                                                             ││
│  │  🖼️ DICOM Images (Không bắt buộc)                           ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │  [📤 CHỌN FILE]  Chưa upload                          │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ✅ KIỂM TRA THÔNG TIN                                          │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ☑ Tên bệnh nhân khớp với đơn                                ││
│  │ ☑ Mã đơn khớp                                               ││
│  │ ☑ Loại CĐHA khớp                                            ││
│  │ ☑ Báo cáo có đầy đủ (findings + conclusion)                 ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  🚨 MỨC ĐỘ KẾT QUẢ *                                            │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ (●) 🟢 Normal    ( ) 🟡 Abnormal    ( ) 🔴 Critical        ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────┐  ┌─────────────────────────────────────────┐│
│  │      HỦY        │  │          UPLOAD VÀ LƯU KẾT QUẢ          ││
│  └─────────────────┘  └─────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**2. Hệ thống — Kiểm tra giá trị kết quả**
- ⛔ Rẽ nhánh theo kết quả:
  - Bình thường → Tự động lưu EHR + Thông báo KH qua app
  - Bất thường → Alert cho BS → BS review → Liên hệ KH tư vấn
  - **Critical value** → Alert khẩn cấp cho BS trong < 15 phút → Gọi điện KH ngay lập tức

| Điều kiện | Hành động |
|-----------|-----------|
| Bình thường | Tự động lưu EHR + Thông báo KH qua app |
| Bất thường | Alert cho BS → BS review → Liên hệ KH tư vấn |
| Critical value | Alert khẩn cấp cho BS trong < 15 phút → Gọi điện KH ngay lập tức |

**WIREFRAME — Màn hình Xem kết quả LAB (KH View - Patient Portal):**

```
┌─────────────────────────────────────────────────────────────────┐
│  [←]              KẾT QUẢ XÉT NGHIỆM                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🧪 KẾT QUẢ XÉT NGHIỆM CỦA BẠN                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Mã: LAB-20260130-00001                                      ││
│  │ Ngày lấy mẫu: 31/01/2026                                    ││
│  │ Ngày có kết quả: 01/02/2026                                 ││
│  │ Lab: Quest Diagnostics - Downtown                           ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📊 COMPLETE BLOOD COUNT (CBC)                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Test           │ Kết quả  │ Tham chiếu     │ Trạng thái     ││
│  │────────────────┼──────────┼────────────────┼────────────────││
│  │ WBC            │ 7.5      │ 4.5-11.0 K/uL  │ ✅ Bình thường ││
│  │ RBC            │ 4.8      │ 4.5-5.5 M/uL   │ ✅ Bình thường ││
│  │ Hemoglobin     │ 14.2     │ 13.5-17.5 g/dL │ ✅ Bình thường ││
│  │ Hematocrit     │ 42.5     │ 38.8-50.0 %    │ ✅ Bình thường ││
│  │ Platelets      │ 250      │ 150-400 K/uL   │ ✅ Bình thường ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📊 LIPID PANEL                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Test           │ Kết quả  │ Tham chiếu     │ Trạng thái     ││
│  │────────────────┼──────────┼────────────────┼────────────────││
│  │ Total Chol.    │ 195      │ <200 mg/dL     │ ✅ Bình thường ││
│  │ LDL            │ 120      │ <100 mg/dL     │ ⚠️ Cao        ││
│  │ HDL            │ 55       │ >40 mg/dL      │ ✅ Bình thường ││
│  │ Triglycerides  │ 140      │ <150 mg/dL     │ ✅ Bình thường ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  💬 GHI CHÚ TỪ BÁC SĨ                                           │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Kết quả tổng thể tốt. LDL vẫn cao hơn mục tiêu nhưng đã     ││
│  │ cải thiện. Tiếp tục chế độ ăn và tập luyện hiện tại.        ││
│  │ Xét nghiệm lại sau 6 tháng.                                 ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              ĐẶT LỊCH TƯ VẤN VỚI BÁC SĨ                     ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Xử lý ngoại lệ FLOW-04:**

| Trường hợp | Áp dụng | Xử lý |
|-----------|---------|-------|
| Transmission/Upload fails | Lab+Imaging | Lab: Tự động retry 3 lần. Imaging: CS thử upload lại |
| Order ID không match | Lab+Imaging | Cách ly dữ liệu, alert nhân viên lâm sàng |
| File corrupted/unreadable | Imaging | CS liên hệ Center yêu cầu gửi lại |
| Critical value/finding | Lab+Imaging | Urgent alert cho BS, gọi điện KH ngay lập tức |
| Kết quả không đầy đủ | Imaging | CS liên hệ Center yêu cầu bổ sung |

**3. Hệ thống — Cập nhật EMR**
- Cập nhật kết quả vào đúng hồ sơ, link với Order ID + Appointment ID
- Notify MD: "Kết quả Order [ID] đã sẵn sàng để review"
- ⚠️ EMR update fail sau 3 lần → CS manual upload theo quy trình backup

| Điều kiện | Hành động |
|-----------|-----------|
| Update thành công | Notify MD: "EMR đã cập nhật kết quả Order [ID]. Sẵn sàng review." |
| EMR update fail | Auto retry 3 lần (cách 30 giây) |
| Vẫn fail sau 3 lần | Alert đội kỹ thuật + CS: "EMR update fail - Order [ID] - Cần xử lý thủ công." |
| Mapping error (sai patient) | Auto rollback + alert ngay lập tức |

---

## FLOW-05: Review kết quả và Follow-up

**1. BS — Review kết quả (≤ 24h, STAT ≤ 1h)**
- Hệ thống hỗ trợ: highlight giá trị bất thường, so sánh kết quả trước, AI analysis xu hướng
- ⚠️ BS chưa review sau 24h (hoặc 1h STAT) → Auto reminder → Sau 2 lần → CS gọi điện escalate
- ⚠️ Critical value chưa review sau 2h → STAT escalation

- INPUT: test_results hoặc imaging_result (Object, required), patient_id (String, required)
- OUTPUT: review_completed (Boolean), follow_up_actions (Array), documentation (Object)

| Điều kiện | Hành động |
|-----------|-----------|
| BS chưa review sau 24h (thường) | Auto reminder cho MD |
| BS chưa review sau 1h (STAT) | Auto reminder cho MD |
| MD chưa phản hồi sau 2 lần reminder | Auto alert cho CS để escalate |
| Critical value chưa review sau 2h | Auto STAT escalation |

**WIREFRAME — Màn hình Review kết quả (BS View - EHR):**

```
┌─────────────────────────────────────────────────────────────────┐
│  [CVH EHR]            REVIEW KẾT QUẢ                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  👤 Nguyễn Văn A    MRN: CVH-20260115-00001    DOB: 15/03/1985  │
│                                                                 │
│  🔔 NEW RESULTS - Cần review                                    │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Order: LAB-20260130-00001                                   ││
│  │ Ngày có kết quả: 01/02/2026                                 ││
│  │ ⚠️ 1 giá trị bất thường                                     ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📊 KẾT QUẢ CHI TIẾT                                            │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ CBC: Tất cả trong giới hạn bình thường                      ││
│  │ Lipid Panel:                                                ││
│  │   - Total Cholesterol: 195 mg/dL ✅                         ││
│  │   - LDL: 120 mg/dL ⚠️ (Target: <100)                        ││
│  │   - HDL: 55 mg/dL ✅                                        ││
│  │   - Triglycerides: 140 mg/dL ✅                             ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📝 HÀNH ĐỘNG                                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ( ) Kết quả bình thường - Không cần follow-up               ││
│  │ (●) Gửi message cho bệnh nhân                               ││
│  │ ( ) Đặt lịch follow-up consultation                         ││
│  │ ( ) Order xét nghiệm thêm                                   ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  💬 GHI CHÚ CHO BỆNH NHÂN                                       │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Kết quả tổng thể tốt. LDL vẫn cao hơn mục tiêu nhưng đã     ││
│  │ cải thiện. Tiếp tục chế độ ăn và tập luyện hiện tại.        ││
│  │ Xét nghiệm lại sau 6 tháng.                                 ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌───────────────────┐  ┌───────────────────────────────────────┐│
│  │      HỦY          │  │       XÁC NHẬN REVIEW HOÀN TẤT        ││
│  └───────────────────┘  └───────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**2. BS — Phân loại và ra quyết định**
- ⛔ Kết quả bình thường → BS ghi nhận, gửi message cho KH, tiếp tục theo dõi định kỳ
- ⛔ Cần follow-up → BS liên hệ KH → Giải thích → Đặt lịch tư vấn hoặc xét nghiệm/CĐHA thêm

| Loại kết quả | Hành động |
|-------------|-----------|
| Bình thường | Document "Labs reviewed, WNL", gửi message cho KH xác nhận |
| Bất thường nhẹ | Schedule follow-up consultation để thảo luận |
| Bất thường đáng lo ngại | Gọi điện/message KH ngay, recommend xét nghiệm thêm hoặc treatment |
| Critical value | Đã được notify từ đối tác, urgent intervention, document immediately |

**Follow-up actions có thể:**

| Action | Mô tả |
|--------|-------|
| Repeat testing | Yêu cầu xét nghiệm lại sau một khoảng thời gian |
| Additional diagnostics | Order thêm xét nghiệm khác để làm rõ |
| Treatment adjustment | Điều chỉnh phác đồ điều trị dựa trên kết quả |
| Specialist referral | Chuyển đến bác sĩ chuyên khoa nếu cần |
| Follow-up consultation | Đặt lịch tư vấn để thảo luận kết quả và kế hoạch |

**3. Hệ thống — Thông báo KH kết quả**
- Gửi kết quả qua App + Email + SMS

---

## STAT Flow: Quy trình khẩn cấp

**1. BS Mỹ — Tạo STAT Order**
- Chọn loại xét nghiệm/hình ảnh STAT + Nhập lý do lâm sàng khẩn cấp + Xác nhận mức ưu tiên
- Hệ thống tạo STAT Order ID, gắn flag STAT

- INPUT: patient_id (String, required), test_type (String, required), stat_reason (String, required — lý do lâm sàng khẩn cấp), priority_level (Enum: STAT, required)
- OUTPUT: stat_order_id (String), stat_flag (Boolean: true)

| Điều kiện | Hành động |
|-----------|-----------|
| BS chọn STAT | Auto bật form với trường bắt buộc highlight đỏ |
| Form chưa hợp lệ (thiếu trường bắt buộc) | Block Submit |
| Tạo thành công | Auto notification + ghi nhật ký timestamp |
| Database error | Auto retry 3 lần (cách 2 giây), nếu fail → thông báo chờ 5 phút |

**2. Hệ thống — Priority Validation (≤ 15 giây)**
- Kiểm tra CPT code, bảo hiểm, chống chỉ định
- Bỏ qua Prior Auth thông thường cho STAT (ghi nhận billing sau)
- ⛔ Hợp lệ → Gán Priority = 1, đẩy vào hàng đợi ưu tiên | Không hợp lệ → Gửi lỗi chi tiết cho BS

| Điều kiện | Hành động |
|-----------|-----------|
| Hợp lệ | Gán Priority = 1, đẩy vào hàng đợi ưu tiên ngay |
| Không hợp lệ | Gửi thông báo lỗi chi tiết cho BS, giữ order trạng thái "Cần sửa" |
| Lỗi kết nối bảo hiểm | Retry 3 lần, nếu failed → Auto bypass + gắn cờ "Cần verify sau" |
| Timeout > 30 giây | Auto alert kỹ thuật + tự động escalate sang MA Pickup |

**3. MA — Pickup (≤ 5 phút)**
- Auto-assign MA đang trực → Push + SMS + Desktop alert
- MA liên hệ KH ngay: xác nhận tình trạng, giải thích quy trình khẩn cấp
- ⚠️ Không có MA online sau 5 phút → CS gọi điện KH thay MA

| Điều kiện | Hành động |
|-----------|-----------|
| MA available | Auto assign (round-robin/workload-based), hiện dashboard STAT |
| MA chính không phản hồi sau 2 phút | Auto assign MA thứ 2 |
| Sau 5 phút chưa có MA pickup | Auto alert cấp trên + ghi log cảnh báo |
| Không có MA nào online | CS gọi điện KH thay MA |
| MA liên hệ KH thành công | Auto ghi timestamp → Chuyển sang Urgent Scheduling |
| MA không liên hệ được KH sau 3 lần gọi | Auto gửi SMS → escalate nếu critical |

**4. MA — Urgent Scheduling (≤ 15 phút)**
- Đặt lịch same-day hoặc next-day tại cơ sở có hợp đồng STAT
- Gửi xác nhận qua App + SMS cho KH
- ⚠️ Tất cả partner API sập → CS gọi trực tiếp cơ sở đặt lịch thủ công

| Điều kiện | Hành động |
|-----------|-----------|
| Partner API phản hồi | Auto tạo Appointment ID + gửi thông báo App + SMS cho KH |
| Partner API không phản hồi | Retry 3 lần, sau đó hiện cơ sở thay thế cho MA |
| Tất cả partner API sập | CS gọi trực tiếp cơ sở đặt lịch thủ công |
| SMS failed | Auto retry qua App notification |

**5. KH — STAT Execution tại cơ sở**
- Hệ thống gửi prep instructions 1h trước lịch
- Theo dõi check-in qua webhook từ partner
- ⚠️ No-show sau 30 phút → Auto SMS nhắc → Không phản hồi → MA gọi khẩn

| Điều kiện | Hành động |
|-----------|-----------|
| KH check-in | Auto cập nhật trạng thái "Đang thực hiện" |
| Không có check-in sau 30 phút từ giờ hẹn | Auto gắn cờ "No-Show nguy cơ" + SMS nhắc |
| Không phản hồi sau 15 phút | Auto alert MA để gọi điện khẩn |
| Nghi ngờ tình huống khẩn cấp | CS escalate cho BS đánh giá tình trạng lâm sàng |

**6. Hệ thống — STAT Results (≤ 24h)**
- Nhận kết quả qua HL7/API/Fax
- Auto scan giá trị, flag Critical Value ngay lập tức
- ⚠️ Sau 22h chưa có kết quả → CS gọi partner khẩn

| Điều kiện | Hành động |
|-----------|-----------|
| Nhận kết quả thành công | Parse, kiểm tra định dạng, ghi database, lưu DICOM nếu có |
| Phát hiện Critical Value | Gắn cờ đỏ CriticalAlert + gửi đồng thời cho BS: SMS + App + Desktop trong 5 phút |
| Kết quả sai định dạng | Auto retry request từ partner |
| Sau 22h chưa có kết quả | Auto alert MA + CS liên hệ partner khẩn |

**7. BS — Immediate MD Review (≤ 1h, Critical Value ≤ 30 phút)**
- Dashboard tổng hợp: kết quả + EMR + AI analysis + DICOM viewer
- ⛔ Ổn định → Ghi nhận, tiếp tục theo dõi | Cần can thiệp → Chỉ định xử lý | Khẩn cấp → Emergency Protocol
- ⚠️ BS không review sau 45 phút (20 phút Critical) → CS gọi trực tiếp

| Điều kiện | Hành động |
|-----------|-----------|
| Ổn định | Ghi nhận vào EMR, tiếp tục theo dõi |
| Cần can thiệp | Chỉ định hướng xử lý cụ thể → Chuyển sang Intervention |
| Khẩn cấp | Kích hoạt Emergency Protocol |
| Chưa mở dashboard sau 15 phút | Gửi SMS nhắc |
| Chưa review sau 45 phút (20 phút Critical) | Auto alert CS escalate |

**8. BS — Intervention (≤ 2h)**
- Gọi điện/nhắn tin KH → Thông báo kết quả + giải thích + hướng dẫn
- ⛔ Kê đơn thuốc | Đặt lịch tái khám | Chuyển viện khẩn | Hướng dẫn theo dõi tại nhà
- Đóng ca STAT với trạng thái: Ổn định / Chuyển viện / Cần theo dõi
- ⚠️ BS không hoàn tất sau 2h → CS gọi escalate. KH không liên hệ được → Emergency Protocol

| Điều kiện | Hành động |
|-----------|-----------|
| BS submit "Cần can thiệp" | Auto tạo Intervention task với deadline 2h |
| Cần chuyển viện | Auto chuẩn bị form chuyển viện, điền sẵn từ EMR |
| Cần đơn thuốc | Auto mở form Rx Order |
| BS submit Intervention hoàn tất | Auto lưu EMR + gửi tóm tắt cho KH qua App + Email + đóng ca STAT |
| Chưa bắt đầu sau 1 giờ | Gửi SMS nhắc BS |
| Ca chưa đóng sau 24 giờ | Auto alert CS leo thang |
| KH không liên hệ được | CS phối hợp BS kích hoạt Emergency Protocol |

---

## Bảng thông báo tự động (mở rộng có template)

### LAB Templates

**SMS-LAB-001 — Thông báo có đơn xét nghiệm mới:**
- Kênh: SMS → KH
- Thời điểm: Ngay sau khi Lab confirm yêu cầu
- Nội dung: `[CVH] Bạn có đơn xét nghiệm mới. Lab: [Tên Lab], [Địa chỉ]. Mã: [LAB-ID]. Chi tiết: [link]`

**SMS-LAB-002 — Nhắc nhở đến Lab:**
- Kênh: SMS → KH
- Thời điểm: 3 ngày sau khi tạo đơn nếu chưa thực hiện
- Nội dung: `[CVH] Nhắc nhở: Bạn có đơn xét nghiệm chưa thực hiện. Lab: [Tên Lab]. Mã: [LAB-ID]. Vui lòng đến Lab sớm.`

**SMS-LAB-003 — Kết quả xét nghiệm đã có:**
- Kênh: SMS → KH
- Thời điểm: Ngay sau khi kết quả import vào EHR
- Nội dung: `[CVH] Kết quả xét nghiệm của bạn đã sẵn sàng. Bác sĩ sẽ review và liên hệ nếu cần. Xem chi tiết: [link]`

**EMAIL-LAB-001 — Thông tin chi tiết xét nghiệm:**
- Kênh: Email → KH
- Thời điểm: Ngay sau khi Lab confirm yêu cầu
- Subject: `[CVH] Thông tin xét nghiệm của bạn - Mã: [LAB-ID]`
- Body: Tên KH, Thông tin Lab (tên, địa chỉ, SĐT, giờ hoạt động), Mã xét nghiệm, Loại xét nghiệm, Hướng dẫn chuẩn bị, Chi phí ước tính

**EMAIL-LAB-002 — Kết quả xét nghiệm đã có:**
- Kênh: Email → KH
- Thời điểm: Ngay sau khi kết quả import vào EHR
- Subject: `[CVH] Kết quả xét nghiệm của bạn đã sẵn sàng`
- Body: Tên KH, Mã đơn, Ngày lấy mẫu, Ngày có kết quả, [Link xem kết quả]

**PUSH-LAB-001 — Đơn xét nghiệm mới:**
- Kênh: Push → KH
- Thời điểm: Ngay sau khi Lab confirm
- Title: `🧪 Bạn có đơn xét nghiệm mới`
- Body: `Bác sĩ đã tạo đơn xét nghiệm. Nhấn để xem chi tiết và hướng dẫn.`

**PUSH-LAB-002 — Kết quả xét nghiệm đã có:**
- Kênh: Push → KH
- Thời điểm: Ngay sau khi kết quả import vào EHR
- Title: `📊 Kết quả xét nghiệm đã sẵn sàng`
- Body: `Kết quả xét nghiệm của bạn đã có. Nhấn để xem chi tiết.`

**PUSH-LAB-003 — BS notification (Kết quả mới):**
- Kênh: Push → BS
- Thời điểm: Ngay sau khi kết quả import vào EHR
- Title: `🔔 Kết quả xét nghiệm mới cần review`
- Body: `Bệnh nhân [Tên] có kết quả xét nghiệm mới. [⚠️ X giá trị bất thường nếu có]`

### IMAGING Templates

**SMS-IMG-001 — Thông báo có đơn CĐHA mới:**
- Kênh: SMS → KH
- Thời điểm: Ngay sau khi CS gửi yêu cầu thành công
- Nội dung: `[CVH] Bạn có đơn CĐHA mới: [Loại CĐHA]. Trung tâm: [Tên center], [Địa chỉ]. Mã: [IMG-ID]. Chi tiết: [link]`

**SMS-IMG-002 — Kết quả CĐHA đã có:**
- Kênh: SMS → KH
- Thời điểm: Sau khi BS approve kết quả
- Nội dung: `[CVH] Kết quả CĐHA [Loại] của bạn đã sẵn sàng. Bác sĩ đã review. Xem chi tiết: [link]`

**SMS-IMG-003 — Kết quả KHẨN CẤP (Critical):**
- Kênh: SMS → KH
- Thời điểm: Ngay khi phát hiện Critical finding
- Nội dung: `🚨 [CVH KHẨN] [Tên KH], kết quả CĐHA của bạn cần được bác sĩ trao đổi NGAY. Vui lòng GỌI LẠI: [hotline] hoặc chờ cuộc gọi từ CVH.`

**EMAIL-IMG-001 — Thông tin chi tiết CĐHA:**
- Kênh: Email → KH
- Thời điểm: Ngay sau khi CS gửi yêu cầu thành công
- Subject: `[CVH] Thông tin chẩn đoán hình ảnh của bạn - Mã: [IMG-ID]`
- Body: Tên KH, Loại CĐHA, Vùng chụp, Thông tin Imaging Center, Hướng dẫn chuẩn bị, Chi phí ước tính

**EMAIL-IMG-002 — Kết quả CĐHA đã có:**
- Kênh: Email → KH
- Thời điểm: Sau khi BS approve kết quả
- Subject: `[CVH] Kết quả chẩn đoán hình ảnh của bạn đã sẵn sàng`
- Body: Tên KH, Loại CĐHA, Ngày thực hiện, Mã đơn, BS review, [Link xem kết quả]

**PUSH-IMG-001 — Đơn CĐHA mới:**
- Kênh: Push → KH
- Thời điểm: Ngay sau khi CS gửi yêu cầu thành công
- Title: `🏥 Bạn có đơn CĐHA mới`
- Body: `Bác sĩ đã tạo đơn [Loại CĐHA]. Nhấn để xem chi tiết và hướng dẫn.`

**PUSH-IMG-002 — Kết quả CĐHA đã có:**
- Kênh: Push → KH
- Thời điểm: Sau khi BS approve kết quả
- Title: `📊 Kết quả CĐHA đã sẵn sàng`
- Body: `Kết quả [Loại CĐHA] của bạn đã có. Nhấn để xem chi tiết.`

**PUSH-IMG-003 — BS notification (Kết quả CĐHA mới):**
- Kênh: Push → BS
- Thời điểm: Ngay sau khi CS upload kết quả vào EHR
- Title: `🔔 Kết quả CĐHA mới cần review`
- Body: `Bệnh nhân [Tên] có kết quả [Loại CĐHA]. [🔴 CRITICAL nếu critical]`

### Thông báo nội bộ

| Sự kiện | Kênh | Người nhận | Nội dung |
|---------|------|------------|----------|
| Order tạo thành công | App | MD | "Order [ID] đã lưu. Đang chuyển xác thực." |
| Validation pass | App | MD | "Order [ID] hợp lệ. Đang phân loại." |
| Critical Value | SMS + App + Desktop | MD | "⚠️ CRITICAL VALUE - Order [ID] - Review ngay." |
| STAT order | SMS + App + Desktop | MA | "STAT Order [ID] đã vào Priority Queue. Xử lý ngay." |
| MD chưa review quá hạn | SMS | MD | Nhắc nhở review |
| Mẫu đã lấy | App | KH | "Mẫu đã được lấy. Kết quả dự kiến trong [N] ngày." |
| KH không confirm attendance | SMS | KH | "Bạn có đến lịch hẹn hôm nay không?" |

---

## Bảng giám sát CS-MD đầy đủ

| Trạm | KPI | Auto handling | CS (Fatal) | MD |
|------|-----|---------------|------------|-----|
| **Order Creation** | Thời gian: ≤ 5 phút; Đầy đủ Mã CPT, chỉ định lâm sàng, mã chẩn đoán | Auto checklist bắt buộc; Validate real-time; CPT code không hợp lệ → highlight đỏ + gợi ý; Order chưa hoàn tất >5 phút → auto cảnh báo; Database error → retry 3 lần | Hệ thống validate sập >10 phút → CS ghi nhận order thủ công tạm thời, báo IT | ✅ Nhập đầy đủ CPT, chỉ định, chẩn đoán, ưu tiên; Review nếu hệ thống cảnh báo |
| **Order Validation** | Thời gian: ≤ 20 giây; Kiểm tra đầy đủ trường, CPT, chống chỉ định, duplicate 30 ngày | Auto validate 100%; Thiếu info → reject + danh sách lỗi; Duplicate → popup confirm; Chống chỉ định → cảnh báo đỏ | ❌ Không can thiệp (chỉ xem report tổng hợp) | ✅ Review order bị reject; Xác nhận duplicate; Đánh giá cảnh báo chống chỉ định |
| **Lab Routing** | Thời gian: ≤ 10 giây; Phân loại đúng order type; Tracking ID tạo đúng | Auto phân loại 100%; Routing fail → retry 3 lần → alert kỹ thuật | ❌ Không can thiệp (chỉ nhận alert nếu routing fail 3 lần → báo IT) | ❌ Không can thiệp |
| **Search Lab** | Thời gian: ≤ 3 giây; Lọc in-network, tối thiểu 3 options | Auto query database; Filter theo insurance + capabilities; Không tìm thấy trong 10 dặm → mở rộng 25 dặm | Không tìm được lab sau mở rộng tối đa → CS tìm thủ công qua database nội bộ | ❌ Không can thiệp |
| **Send Lab List** | Thời gian: ≤ 30s; Gửi đủ kênh App/Email/SMS | Auto gửi template chuẩn; Email fail → retry SMS; SMS fail → App; Tất cả fail → retry 3 lần | Fail sau 3 lần → CS gọi điện chia sẻ danh sách thủ công | ❌ Không can thiệp |
| **KH Select Lab** | Thời gian: N/A (tùy KH); Ghi nhận đúng lab đã chọn | Auto hiển thị xác nhận; Không chọn trong 24h → reminder; Lỗi selection → retry + reload | ❌ Không can thiệp (CS chỉ giải thích khác biệt giữa lab nếu KH hỏi) | ✅ Có thể recommend lab nếu KH hỏi trong tư vấn |
| **Schedule Appointment** | Slot available real-time; Appointment ID liên kết Order ID; Confirmation qua 3 kênh | Auto query slot; Double booking → detect + load slot mới; API fail → retry 3 lần | Double booking sau xử lý → CS liên hệ lab giải quyết; Lab reject → CS hỗ trợ đổi lịch | ❌ Không can thiệp |
| **Prior Auth Check** | Thời gian: ≤ 2 phút qua API bảo hiểm; Submit auth request tự động | Auto tra cứu + submit auth; API fail → retry 3 lần (30s); Auth pending → theo dõi nhắc 24h | Auth reject → CS liên hệ payer appeal; Auth pending >48h → CS follow up payer | ✅ Review auth reject + cung cấp clinical documentation; Quyết định thay thế order |
| **Send Prep Instructions** | Thời gian: ≤ 2 phút sau confirm; Đúng loại test; Song ngữ Việt-Anh | Auto match prep với CPT code; Auto-fill info; Gửi Email + App; Fail → retry 3 lần | Fail sau 3 lần → CS gọi đọc hướng dẫn; Template không có → CS lấy template đã duyệt | ✅ Review/approve prep cho xét nghiệm phức tạp; Điều chỉnh nếu KH có điều kiện đặc biệt |
| **Appointment Reminder** | Email 3h trước; App 30p + 15p trước | Auto lên lịch nhắc; Fail → retry + switch kênh; KH không confirm → SMS hỏi | KH không confirm + lịch sử no-show → CS gọi nhắc trực tiếp | ❌ Không can thiệp |
| **Lab Execution** | Thời gian: ≤ 30 phút check-in → hoàn tất lấy mẫu | Auto nhận status qua API/HL7; Validate data integrity; Không có confirmation sau 1h → alert | Không confirmation sau 1h → CS liên hệ lab trực tiếp | ❌ Không can thiệp (review nếu có complaint chất lượng) |
| **Radiologist Read** | Thời gian: ≤ 48h (thường), ≤ 4h (STAT) | Auto track thời gian; Auto-flag báo cáo trễ; Parse fail → yêu cầu gửi lại | Báo cáo >48h → CS liên hệ imaging center thúc đẩy | ✅ Review báo cáo radiologist; Liên hệ radiologist nếu findings bất thường |
| **Results Received** | Thời gian: ≤ 5 phút; Đúng format HL7/FHIR/DICOM; Critical values detected | Auto monitor 24/7; Validate data; Auto flag critical; Critical → alert MD <15 phút | Data parsing fail >2h → CS nhận kết quả qua email secure backup | ✅ Review critical values alert; Xác nhận kết quả bất thường |
| **Update EMR** | Thời gian: ≤ 2 phút; Đúng hồ sơ bệnh nhân; Audit trail đầy đủ | Auto update; Mapping error → rollback + alert; Fail → retry 3 lần (30s) | EMR fail sau 3 lần + IT không giải quyết 1h → CS manual update backup | ✅ Verify kết quả trong EMR trước review |
| **MD Review** | Thời gian: ≤ 24h (thường), ≤ 1h (STAT); Highlight bất thường; Trend analysis | Auto lấy kết quả trước; Highlight đỏ/vàng; Chưa review 24h → reminder; 2 lần → CS escalate | MD chưa review sau 24h (4h STAT/critical) → CS gọi trực tiếp MD | ✅ Review đầy đủ kết quả; So sánh trend; Ghi assessment + kế hoạch vào EMR |
| **MA Contact Patient** | Thời gian: ≤ 15 phút cho STAT; Ghi nhận call đầy đủ | Auto-assign MA; Chuẩn bị call script; Không liên hệ được sau 3 lần → SMS + escalate | ✅ MA thực hiện outbound call; Giải thích kết quả theo script; Book follow-up | ✅ Review STAT urgency; Cung cấp clinical context cho MA |
| **STAT Order Creation** | 100% đầy đủ mã STAT + lý do khẩn cấp; STAT Order ID tạo thành công | Auto form STAT; Validate real-time; Database error → retry 3 lần | Database sập >10 phút → CS ghi nhận order qua điện thoại, nhập hệ thống dự phòng | ✅ BS chọn loại test STAT + nhập lý do + submit |
| **Priority Validation** | Thời gian: ≤ 15 giây; 100% validate hoặc reject có lý do | Auto validate 100%; Bỏ qua Prior Auth cho STAT; Lỗi kết nối → retry 3 lần → bypass + cờ verify | Hệ thống validation sập >10 phút → CS chuyển order sang checklist dự phòng | ❌ Không can thiệp |
| **MA Pickup** | Thời gian: ≤ 5 phút; 100% STAT order có MA pickup | Auto assign MA (round-robin); MA không phản hồi 2 phút → assign MA thứ 2; Countdown 5 phút | Không có MA online → CS gọi KH thay MA; Notification sập → CS gọi MA trực tiếp | ❌ Không can thiệp |
| **Urgent Scheduling** | Thời gian: ≤ 15 phút; 100% tại cơ sở có hợp đồng STAT; 100% KH nhận xác nhận | Auto hiện danh sách STAT partner; Auto check in-network; Auto gửi request API | Tất cả partner API sập → CS gọi cơ sở đặt lịch thủ công; App+SMS fail → CS gọi KH xác nhận | ❌ Không can thiệp |
| **STAT Execution** | Check-in đúng; STAT order đồng bộ partner; 0% lỗi đồng bộ | Auto gửi prep 1h trước; Gửi order qua HL7/API 30 phút trước; No-show → SMS nhắc | KH đến nhưng partner không có order → CS gọi cung cấp info; No-show khẩn → CS escalate BS | ❌ Không can thiệp |
| **STAT Results** | Thời gian: ≤ 24h; 100% nhận đầy đủ; Auto flag Critical Value | Auto lắng nghe HL7/API 24/7; Parse + validate; Critical → alert BS trong 5 phút | Sau 22h chưa có → CS gọi partner gửi Fax/Email backup; Parse fail → CS nhận thủ công + upload | ❌ Không can thiệp |
| **Immediate MD Review** | Thời gian: ≤ 1h (≤ 30 phút Critical); Dashboard đầy đủ | Auto tổng hợp dashboard; Auto assign BS; SMS + App + Desktop alert; Countdown timer | BS không review sau 45 phút (20 phút Critical) → CS gọi BS; BS báo lỗi truy cập → CS escalate IT | ✅ Mở dashboard review ngay; Xem kết quả + DICOM; Đối chiếu EMR; Đưa ra quyết định |
| **Intervention** | Thời gian: ≤ 2h; 100% KH được thông báo; 100% ca đóng rõ ràng | Auto tạo task 2h; Chuẩn bị info liên lạc + checklist; Auto lưu EMR; Chưa bắt đầu 1h → SMS | BS không hoàn tất 2h → CS gọi escalate; KH không liên hệ được → CS + BS Emergency Protocol | ✅ Gọi/nhắn KH; Thông báo kết quả; Kê đơn/tái khám/chuyển viện; Đóng ca STAT |

---

## Data Specification

### Lab Order

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| lab_order_id | String | Có | LAB-YYYYMMDD-XXXXX | Mã duy nhất đơn xét nghiệm |
| patient_id | String | Có | CVH-YYYYMMDD-XXXXX | Reference to Patient |
| ordering_physician_id | String | Có | | Reference to Physician |
| test_panels | Array | Có | Min 1 item | Danh sách mã test panel |
| clinical_indication | String | Có | | Lý do chỉ định |
| priority | Enum | Có | Routine / Urgent / STAT | Mức ưu tiên |
| special_instructions | String | Không | | Hướng dẫn chuẩn bị đặc biệt |
| status | Enum | Có | Created / Sent / Confirmed / InProgress / Completed | Trạng thái đơn |
| selected_lab_id | String | Không | | Reference to Lab (sau khi chọn) |
| created_at | DateTime | Có | ISO 8601 | Thời điểm tạo đơn |
| updated_at | DateTime | Có | ISO 8601 | Thời điểm cập nhật gần nhất |

### Lab Result

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| result_id | String | Có | Auto-generated | Mã kết quả duy nhất |
| lab_order_id | String | Có | | Reference to Lab Order |
| accession_number | String | Có | | Mã tracking của Lab |
| test_code | String | Có | CPT code | Mã test panel |
| test_name | String | Có | | Tên xét nghiệm |
| result_value | String | Có | | Giá trị kết quả |
| result_unit | String | Có | | Đơn vị đo |
| reference_range_low | Decimal | Không | | Giới hạn dưới khoảng tham chiếu |
| reference_range_high | Decimal | Không | | Giới hạn trên khoảng tham chiếu |
| abnormal_flag | Enum | Không | H (High) / L (Low) / C (Critical) / null | Cờ bất thường |
| performing_lab | String | Có | | Lab thực hiện xét nghiệm |
| specimen_collected_at | DateTime | Có | ISO 8601 | Thời điểm lấy mẫu |
| result_available_at | DateTime | Có | ISO 8601 | Thời điểm có kết quả |

### Imaging Order

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| imaging_order_id | String | Có | IMG-YYYYMMDD-XXXXX | Mã duy nhất đơn CĐHA |
| patient_id | String | Có | CVH-YYYYMMDD-XXXXX | Reference to Patient |
| ordering_physician_id | String | Có | | Reference to Physician |
| imaging_type | Enum | Có | XRAY, CT, CT_C, MRI, MRI_C, US, MAMMO, DEXA | Loại CĐHA |
| body_part | String | Có | Chest, Abdomen, Head, Spine, etc. | Vùng chụp |
| clinical_indication | String | Có | | Lý do chỉ định |
| priority | Enum | Có | ROUTINE / URGENT | Mức ưu tiên |
| contrast_required | Boolean | Có | true/false | Có/Không cần chất tương phản |
| special_notes | String | Không | | Ghi chú đặc biệt |
| status | Enum | Có | CREATED / DISPATCHED / IN_PROGRESS / COMPLETED | Trạng thái đơn |
| selected_center_id | String | Không | | Reference to Imaging Center |
| dispatch_method | Enum | Không | FAX / EMAIL | Phương thức CS gửi |
| dispatched_at | DateTime | Không | ISO 8601 | Thời điểm CS gửi đơn |
| dispatched_by | String | Không | | CS thực hiện gửi |
| created_at | DateTime | Có | ISO 8601 | Thời điểm tạo đơn |
| updated_at | DateTime | Có | ISO 8601 | Thời điểm cập nhật gần nhất |

### Imaging Result

| Field | Type | Required | Format/Rule | Mô tả |
|-------|------|----------|-------------|-------|
| result_id | String | Có | Auto-generated | Mã kết quả duy nhất |
| imaging_order_id | String | Có | | Reference to Imaging Order |
| imaging_date | DateTime | Có | ISO 8601 | Ngày thực hiện chụp |
| radiologist_name | String | Có | | Radiologist đọc kết quả |
| findings | Text | Có | | Mô tả chi tiết phát hiện |
| conclusion | Text | Có | | Kết luận tóm tắt |
| critical_flag | Enum | Có | NORMAL / ABNORMAL / CRITICAL | Mức độ kết quả |
| performing_center | String | Có | | Imaging Center thực hiện |
| pdf_report_url | String | Có | URL | URL đến PDF báo cáo |
| dicom_url | String | Không | URL | URL đến hình ảnh DICOM |
| received_via | Enum | Có | FAX / EMAIL / CD / PORTAL | CS nhận qua kênh nào |
| uploaded_by | String | Có | | CS upload kết quả |
| uploaded_at | DateTime | Có | ISO 8601 | Thời điểm upload |

---

## So sánh Lab vs Imaging

| Tiêu chí | Lab | Imaging |
|----------|-----|---------|
| Gửi đơn | Tự động qua kênh tích hợp | CS gửi thủ công (FAX/email mã hóa) |
| Nhận kết quả | Tự động qua HL7/FHIR | CS nhận (fax/email/CD) + upload thủ công |
| Loại dịch vụ | BMP, CMP, CBC, Lipid, A1C, Thyroid, Urinalysis | X-ray, CT, MRI, US, Mammography, DEXA |
| Duration | 3-7 ngày | 5-14 ngày |
| STAT TAT | 2-4 giờ | 24-48 giờ |

---

## Nguồn tài liệu gốc

- `docs/02-product/ba-specs/service-10-Lab_Imaging_Management/base/Lab_Imaging_Management_Touchpoint_Flow_v3.md`
- `docs/02-product/ba-specs/service-10-Lab_Imaging_Management/checklist-cs-md/checklist-cs-md.md`
- `docs/02-product/tomtat-chitiet/tomtat/service-10-Lab_Imaging_Management-tomtat.md`
