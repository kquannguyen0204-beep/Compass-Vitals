# BD_KPIs_CustomerJourney

---

## KPIs Tham khảo

| Chỉ số | Mô tả | Target (KPIs) | Công thức |
|---|---|---|---|
| Day-1 Onboarding Start Rate | Tỉ lệ khách hàng thực hiện Onboarding block 1 ngay trong 24h đầu tiên từ khi đăng ký tài khoản | ≥70% | = (Số user bắt đầu onboarding trong 24h / Tổng user đăng ký mới) * 100%<br>👉 Tracking:<br>Event: signup_time, onboarding_start_time<br>Điều kiện: onboarding_start_time ≤ signup_time + 24h |
| Day-7 Return Rate | Tỉ lệ khách hàng mở app lại trong vòng 7 ngày kể từ khi đăng ký (không tính lần mở app để đăng ký đầu tiên) | ≥50% | = AVG(Số user mở app trong 7 ngày/Tổng user đăng ký) *100%<br>Điều kiện: (active time lần 2 - active time lần 1) < 7 |
| Onboarding completion rate | Tỉ lệ khách hàng hoàn thành các field required để gen ra Health Overview trong 7 ngày | ≥60% | = (User có Health Overview / Tổng user đăng ký) * 100% |
| Feature Adoption Velocity | Tỉ lệ khách hàng có sử dụng ít nhất 1 dịch vụ trong 14 ngày kể từ khi đăng ký tài khoản | ≥40% | = (Số user dùng ≥1 feature trong 14 ngày / Tổng user đăng ký) * 100%<br>👉 Feature = chat 24/7/ book video / tư vấn lab... |
| Time-to-First-Consult — TFC | Mất bao nhiêu ngày từ khi đăng ký đến khi người dùng đặt lịch hoặc chat với bác sĩ lần đầu. | ≤7 ngày | = AVG (Ngày first consult - ngày signup) |
| Monthly Active Users — MAU | Số member có ít nhất 1 hành động có nghĩa trong tháng: chat bác sĩ, xem kết quả xét nghiệm, đặt lịch tái khám. | ≥20% | = (Số user có ≥1 meaningful action trong tháng / Tổng active users hoặc total users) * 100%<br>👉 Meaningful action: chat 24/7/book video... |
| In-App Reminder Response Rate | Bao nhiêu người click hoặc đặt lịch sau khi nhận thông báo tự động (chuỗi R#1–R#10 trong journey map). | ≥15% | = (Số user click / book sau reminder / Tổng user nhận reminder) * 100% |
| Net Promoter Score — NPS | Khả năng member giới thiệu Compass Vitals cho người Việt khác trong gia đình hoặc cộng đồng (thang 0–10). - Đo bằng khảo sát | ≥20% | Khảo sát 1 câu: "Bạn có sẵn sàng giới thiệu CV cho người thân/bạn bè không? (0–10)"<br>Gửi sau tháng 1 và tháng 3. Tính NPS = %Promoters (9–10) − %Detractors (0–6). |
| Trial-to-Paid CVR (%) | Tỉ lệ khách hàng từ Trial chuyển đổi sang Paid member | 15-35% | = (Số user bắt đầu trial trong tháng T convert sang paid / Tổng user bắt đầu trial trong tháng T) * 100% |
| Retention rate (tái ký) - tháng | Tỉ lệ khách hàng gia hạn/nâng cấp gói hàng tháng | ≥40% | = (Số user gia hạn tháng sau / Tổng user hết hạn tháng trước) * 100% |


---

## Rule gửi reminder

| Phase | Reminder | Timeline | Mốc thời gian | Timeline của khách |  | Ngày | Danh sách reminder KH nhận được |
|---|---|---|---|---|---|---|---|
| 1 | Reminder 1 | 2026-03-02 00:00:00 | Đăng ký + 1 | Đăng ký tài khoản | 2026-03-01 00:00:00 | 2026-03-02 00:00:00 | Reminder 1 |
|  | Reminder 2 | 2026-03-03 00:00:00 | Đăng ký + 2 | Onboarding 1 | 2026-03-16 00:00:00 | 2026-03-03 00:00:00 | Reminder 2 |
|  | Reminder 3 | 2026-03-06 00:00:00 | Đăng ký + 5 | Onboarding 2 | 2026-03-18 00:00:00 | 2026-03-06 00:00:00 | Reminder 3 |
|  | Reminder 4 | 2026-03-11 00:00:00 | Đăng ký + 10 | Onboarding 3 | 2026-03-19 00:00:00 | 2026-03-11 00:00:00 | Reminder 4 |
|  | Reminder 5 | 2026-03-15 00:00:00 | Đăng ký + 14 | Onboarding 4 | 2026-03-30 00:00:00 | 2026-03-15 00:00:00 | Reminder 5 |
|  | Reminder 6 | 2026-03-21 00:00:00 | Đăng ký + 20 | Ngày hết hạn | 2026-04-01 00:00:00 | x | Không có noti |
|  | Reminder 7 | 2026-03-31 00:00:00 | Đăng ký + 30 | Phiên bản |  | x | Không có noti |
| 2 | Reminder 8 | 2026-03-17 00:00:00 | Onboarding 1 +1 | Ver 1 | Push điền onboarding 2 | 2026-03-17 00:00:00 | Reminder 8 Ver 1 |
|  | Reminder 9 | 2026-03-18 00:00:00 | Onboarding 1 +2 | Ver 2 | Push điền onboarding 3 | 2026-03-18 00:00:00 | Reminder 9 Ver 1 |
|  | Reminder 10 | 2026-03-21 00:00:00 | Onboarding 1 +5 | Ver 3 | Push điền onboarding 4 | 2026-03-21 00:00:00 | Reminder 10 Ver 3 |
|  | Reminder 11 | 2026-03-26 00:00:00 | Onboarding 1 +10 |  |  | 2026-03-26 00:00:00 | Reminder 11 Ver 3 |
|  | Reminder 12 | 2026-03-30 00:00:00 | Onboarding 1 + 14 |  |  | 2026-03-30 00:00:00 | Reminder 12 Ver 3 |
|  | Reminder 13 | 2026-04-05 00:00:00 | Onboarding 1 +20 |  |  | x | Không có noti |
|  | Reminder 14 | 2026-04-15 00:00:00 | Onboarding 1 +30 |  |  | x | Không có noti |
| 3 | Reminder 15 | 2026-04-02 00:00:00 | Onboarding 4 + 3 |  |  | 2026-04-02 00:00:00 | Reminder 15 |
| 4 | Reminder 16 | 2026-04-11 00:00:00 | Ngày hết hạn + 10 |  |  | 2026-04-11 00:00:00 | Reminder 16 |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |
|  |  |  |  |  |  | 0 |  |


---

## Sumary

| Platform | Total hành trình khách hàng | Tỷ lệ | Avg Cost/User/tháng  (trung bình lấy mốc 500 user) $ | Total chi phí $ | Số lượng tối đa/tháng |
|---|---|---|---|---|---|
| Email | 8 | 0.4444444444444444 | 1.8 | 9 | 5 |
| Inapp | 10 | 0.5555555555555556 | 2.7 | 16.200000000000003 | 6 |
| SMS (hệ thống  cho account/billing/alerts ) |  |  | 0.25 |  |  |
| TOTAL Communication (Email+app) |  |  | 4.5 |  |  |
| Player | Email | SMS | In-app | Lý do mix chính |  |
| HealthTap | 0.45 | 0.1 | 0.45 | AI/async nudges + message/updates trong app; SMS tiết chế (opt-in) (HealthTap) |  |
| Amwell | 0.6 | 0.25 | 0.15 | B2B2C + visit logistics: email reminder/confirmation, SMS khi đổi lịch/nhắc hẹn (patients.amwell.com) |  |
| Teladoc Health | 0.35 | 0.25 | 0.4 | Visit + chronic programs: cho chọn nhận text/email/push; app dùng mạnh cho “in-flow” (teladochealth.com) |  |
| Doctor on Demand | 0.55 | 0.25 | 0.2 | Visit-based + billing/admin: email mạnh (thanh toán/nhắc), SMS chủ yếu reminder (Doctor On Demand) |  |
| Doctronic | 0.25 | 0.1 | 0.65 | AI-first: trải nghiệm trong app là trung tâm; terms có email/SMS nhưng thường chỉ cho account/billing/alerts (doctronic.ai) |  |
| One Medical | 0.4 | 0.2 | 0.4 | PCP longitudinal: push/tasks + message continuity; có email confirm & reminder + SMS reminder opt-in (onemedical.com) |  |
| K Health | 0.3 | 0.15 | 0.55 | AI + async chat: ưu tiên in-app; họ nêu rõ có thể notify bằng email/push/SMS cho visit, tasks, billing (K Health) |  |


