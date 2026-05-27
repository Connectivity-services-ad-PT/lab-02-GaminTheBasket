# Biên bản đàm phán hợp đồng API - Phân hệ Analytics (A5)

- Cặp đàm phán: Pair 06, 07, 08, 09 (Nhóm A5 đàm phán với A1, A2, A6, A3)
- Product: Product A
- Provider: Analytics (A5)
- Consumer: IoT (A1), Camera (A2), Core Business (A6), Access Gate (A3)
- Phiên: v1.0
- Ngày: 2026-05-27

---

## Issue #1 (Đàm phán với Pair 06 - IoT)
- Raised by: Provider (Analytics)
- Endpoint: Topic `iot.telemetry.ingested`
- Concern: Cần khóa phân cụm để vẽ biểu đồ theo khu vực, nhưng Payload mặc định của IoT không có thông tin vị trí.
- Proposal: Thêm trường tùy chọn `zoneId` vào payload của sự kiện đo lường cảm biến.
- Resolution: Accepted
- Rationale: Giúp Analytics có thể nhóm (group by) các chỉ số tiêu thụ điện/nước theo từng tòa nhà mà không cần gọi ngược lại API của Core Business để tra cứu.
- Impact: Nhóm A1 cần sửa lại schema sinh dữ liệu, thêm trường `zoneId`.

---

## Issue đàm phán với IoT Ingestion (A1)
- **Raised by:** Provider (Analytics - A5)
- **Endpoint:** Topic `iot.telemetry.ingested`
- **Concern:** Analytics cần biết dữ liệu trắc đo thuộc tòa nhà nào để vẽ biểu đồ mật độ sử dụng năng lượng.
- **Proposal:** Nhóm A1 đồng ý bổ sung trường `zoneId` vào payload của sự kiện telemetry.
- **Resolution:** Accepted
- **Rationale:** Việc gán `zoneId` tại nguồn giúp Analytics thực hiện lệnh Group By nhanh chóng, tăng tốc độ xử lý dữ liệu realtime.
- **Impact:** Nhóm A1 đã cập nhật code phát sự kiện để đính kèm thông tin `zoneId`.

## Issue thống nhất chuẩn định dạng thời gian (Cả 2 nhóm)
- **Concern:** Sai lệch múi giờ gây lỗi timeline trên biểu đồ thống kê.
- **Proposal:** Thống nhất định dạng `occurredAt` theo chuẩn ISO 8601, múi giờ UTC (đuôi Z).
- **Resolution:** Accepted
- **Rationale:** Đảm bảo tính tuần tự (Ordering) khi Analytics xử lý đồng bộ chuỗi dữ liệu lịch sử.
- **Impact:** Nhóm A1 đồng ý format lại timestamp trước khi gửi tin nhắn vào Queue.

**Provider sign-off:** Nguyễn Hữu Tuấn Minh (Leader A5)
**Consumer sign-off:** Nguyễn Tuấn Anh (Leader A1)

---

## Issue #3 (Đàm phán chung cho cả 4 Pair)
- Raised by: Provider (Analytics)
- Endpoint: Tất cả các Event
- Concern: Không thống nhất định dạng thời gian dẫn đến sai lệch biểu đồ thống kê khi các service chạy ở các server có múi giờ khác nhau.
- Proposal: Trường `timestamp` bắt buộc sử dụng định dạng ISO 8601 và múi giờ UTC (đuôi Z).
- Resolution: Accepted
- Rationale: UTC là chuẩn quốc tế, giúp Analytics đồng bộ timeline chính xác tuyệt đối từ mọi nguồn phát.
- Impact: Cả 4 nhóm Consumer phải format lại datetime trước khi đẩy event.

---

## Issue đàm phán với Access Gate (A3)
- **Raised by:** Provider (Analytics - A5)
- **Endpoint:** Topic `gate.access.logged`
- **Concern:** Rủi ro trùng lặp dữ liệu (Duplicate) khi thiết bị cổng an ninh mất kết nối mạng và gửi lại tin nhắn sau khi phục hồi, làm sai lệch báo cáo lưu lượng ra/vào.
- **Proposal:** Nhóm A3 (Access Gate) bắt buộc tích hợp thư viện sinh UUID v4 làm `eventId` cho mỗi sự kiện quẹt thẻ.
- **Resolution:** Accepted
- **Rationale:** Phân hệ Analytics (A5) sẽ sử dụng mã này làm Idempotency Key để đảm bảo tính toàn vẹn dữ liệu thống kê (Exactly-once processing).
- **Impact:** Nhóm A3 cập nhật logic code tại module phát sự kiện, đính kèm `eventId` vào gói tin.

## Issue thống nhất chuẩn định dạng thời gian (Cả 2 nhóm)
- **Concern:** Sai lệch thời gian trên biểu đồ do múi giờ local của các thiết bị cổng khác nhau.
- **Proposal:** Thống nhất toàn bộ trường `occurredAt` (timestamp) phải định dạng ISO 8601 và ép múi giờ UTC (đuôi Z).
- **Resolution:** Accepted
- **Rationale:** Đảm bảo tính tuần tự (Ordering) khi Analytics xử lý đồng bộ chuỗi dữ liệu lịch sử.
- **Impact:** Nhóm A3 cập nhật hàm xử lý thời gian trước khi publish tin nhắn lên Broker.

**Provider sign-off:** Nguyễn Hữu Tuấn Minh (Leader A5)
**Consumer sign-off:** Lương Duy Chiến (Leader A3)

---

## Issue đàm phán với Camera Stream (A2)
- **Raised by:** Consumer (Camera Stream - A2)
- **Endpoint:** Topic `camera.motion.detected`
- **Concern:** Tần suất phát hiện chuyển động quá lớn (hàng chục frame/giây) gây nguy cơ nghẽn Broker.
- **Proposal:** Nhóm A5 chấp thuận cho nhóm A2 thực hiện Batching sự kiện (gộp các sự kiện trong 5s thành mảng) trước khi gửi.
- **Resolution:** Modified
- **Rationale:** Việc này giúp hệ thống ổn định, giảm số lượng message overhead trên Broker.
- **Impact:** Analytics Worker đã cập nhật logic để parse dữ liệu nhận được ở dạng mảng (Array).

## Issue thống nhất chuẩn định dạng thời gian (Cả 2 nhóm)
- **Concern:** Sai lệch thời gian trên biểu đồ do múi giờ local của camera khác nhau.
- **Proposal:** Thống nhất định dạng `occurredAt` theo chuẩn ISO 8601, múi giờ UTC (đuôi Z).
- **Resolution:** Accepted
- **Rationale:** Đảm bảo tính tuần tự (Ordering) khi Analytics xử lý dữ liệu.
- **Impact:** Nhóm A2 đồng ý format lại timestamp trước khi gửi tin nhắn vào Queue.

**Provider sign-off:** Nguyễn Hữu Tuấn Minh (Leader A5)
**Consumer sign-off:** Bùi Đình Phúc (Leader A2)

---

## Issue đàm phán với Core Business (A6)
- **Raised by:** Provider (Analytics - A5)
- **Endpoint:** Topic `core.alert.published`
- **Concern:** Analytics cần biết mức độ nghiêm trọng để phân loại KPI cảnh báo trên Dashboard.
- **Proposal:** Nhóm A6 (Core) bắt buộc cung cấp trường `severity` với tập giá trị Enum: `[LOW, MEDIUM, HIGH, CRITICAL]`.
- **Resolution:** Accepted
- **Rationale:** Đảm bảo Analytics có đủ dữ liệu để tính toán KPI vận hành và hiển thị biểu đồ phân loại sự cố chính xác.
- **Impact:** Nhóm A6 bổ sung ràng buộc dữ liệu tại tầng phát sự kiện.

## Issue thống nhất chuẩn định dạng thời gian (Cả 2 nhóm)
- **Concern:** Sai lệch múi giờ gây lỗi timeline trên biểu đồ thống kê KPI.
- **Proposal:** Thống nhất định dạng `occurredAt` theo chuẩn ISO 8601, múi giờ UTC (đuôi Z).
- **Resolution:** Accepted
- **Rationale:** Giúp hệ thống Analytics sắp xếp thứ tự sự cố (Ordering) chính xác tuyệt đối.
- **Impact:** Nhóm A6 đồng ý format lại timestamp trước khi gửi tin nhắn vào Queue.

**Provider sign-off:** Nguyễn Hữu Tuấn Minh (Leader A5)
**Consumer sign-off:** Nguyễn Văn Hưởng (Leader A6)

---

# Chốt hợp đồng v1.0

Provider sign-off: Nguyễn Hữu Tuấn Minh (Leader A5)  
Consumer sign-off: Leader các nhóm A1, A2, A3, A6 (Đã xác nhận qua group chung)    
Date: 2026-05-27