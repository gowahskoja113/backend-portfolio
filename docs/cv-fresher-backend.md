# Nguyễn Hoàng Ngọc Sơn

**Fresher Backend Developer · Java / Spring Boot**

- Ngày sinh: 15/05/2003
- Điện thoại: 0352393203
- Email: sonnguyen.150503@gmail.com
- Địa chỉ: Phường Đông Hòa, Thành phố Hồ Chí Minh
- GitHub: [github.com/gowahskoja113](https://github.com/gowahskoja113)

## Mục tiêu

Fresher backend Java / Spring Boot. Đã làm REST API có JWT, phân quyền theo role và theo bản ghi, JPA trên PostgreSQL và SQL Server, và luồng nghiệp vụ có trạng thái (nhà, hợp đồng, cọc, kích hoạt). Sẵn sàng đi làm sau khi bảo vệ đồ án tốt nghiệp.

## Kỹ năng

- Java 21, Spring Boot 4 và 3.5 (Web, Security, Data JPA, Validation, `@Scheduled`), Maven, Gradle
- REST `/api/v1`, JWT HMAC (`jjwt`), BCrypt, `@PreAuthorize`, kiểm tra đúng nhà hoặc khu trong service
- PostgreSQL, SQL Server, Hibernate, MapStruct, SpringDoc OpenAPI
- PayOS (webhook ký HMAC), Twilio Verify, SMTP, Apache POI, XDocReport, iText 7
- OCR.space, Google Cloud Vision, Gemini, ONNX Runtime (fallback trong JVM)
- Docker (image hai giai đoạn), Git, GitHub Actions

## Dự án

### Sub-Leasing Management System (SLMS2026) — Đồ án tốt nghiệp SEP490

05/2026 – 10/2026 · Backend · Nhóm 4  
Java 21 · Spring Boot 4 · PostgreSQL · Docker  
[github.com/gowahskoja113/Sub-leasing-managemant-system](https://github.com/gowahskoja113/Sub-leasing-managemant-system)

Một API cho công ty thuê nhà nguyên căn, cải tạo, rồi cho thuê lại theo phòng hoặc nguyên căn. Ảnh do frontend đẩy Cloudinary; API chỉ giữ URL.

Em phụ trách onboard nhà và phòng, hợp đồng thuê, cọc PayOS, chỉ số đồng hồ và bảo trì. Thành viên khác giữ cổng tài chính chủ đầu tư, gia hạn hợp đồng, API mobile và một phần trả phòng.

- REST JWT, năm role (`ADMIN`, `OWNER`, `MANAGER`, `TENANT`, `USER`). `@PreAuthorize` chặn loại user; service kiểm tra đúng nhà hoặc khu
- Onboard nhà: phòng, thiết bị, import Excel/ZIP, xuất hợp đồng DOCX/PDF
- Cọc PayOS: webhook public đối chiếu chữ ký, ghi `PAID` theo `orderCode`, chuyển hợp đồng sang chờ xác nhận. Gọi lại không tạo thêm hoá đơn cọc
- Kích hoạt hợp đồng: sau khi đã trả, khách thuê yêu cầu gửi hai OTP (khách và quản lý). Hợp đồng sang `ACTIVE` khi cả hai mã đúng. Khóa dòng khi ghi nhận để hai lần xác nhận không kích hoạt đúp
- Chỉ số đồng hồ qua OCR. Nhãn thiết bị: Google Vision, lỗi thì ONNX trong JVM. Bảo trì gắn QR và yêu cầu sửa
- Docker hai giai đoạn, Postgres có volume, cron múi `Asia/Ho_Chi_Minh`, cấu hình qua biến môi trường. Schema do Hibernate `ddl-auto: update`

### EV Dealer System — Đồ án chuyên ngành SWP391

09/2025 – 11/2025 · Backend · Nhóm 4  
Java 21 · Spring Boot 3.5 · Gradle · SQL Server  
[github.com/gowahskoja113/EV-Dealer-System](https://github.com/gowahskoja113/EV-Dealer-System)

API một process cho đại lý xe điện: mẫu xe, tồn kho theo VIN, khách hàng, cọc, giao xe, lịch lái thử và bảo dưỡng. Em viết phần lớn API.

- Kho và phạm vi đại lý: mẫu xe, tồn theo model, chuyển kho, VIN. Nhân viên không phải admin chỉ bán VIN thuộc đại lý của mình
- Đơn `PROCESSING` giữ VIN `HOLD`, cọc hoặc trả nốt bằng tiền mặt hoặc VNPay sandbox, giao xe đến `COMPLETED` / VIN `DELIVERED`. Callback VNPay public, đối chiếu chữ ký trước khi đổi trạng thái
- PDF hợp đồng đặt cọc và phiếu giao xe (iText 7). Import mẫu xe từ Excel (Apache POI)
- Quên mật khẩu qua token và email SMTP. Job định kỳ tự hoàn tất đơn đang giao khi đến ngày giao

## Học vấn

**FPT University — Cơ sở TP. Hồ Chí Minh** · Kỹ thuật phần mềm  
2021 – nay

- On-the-job training tại FPT Japan Academy (09/2024 – 12/2024)
- SWP391: EV Dealer System · SEP490: Sub-Leasing Management System

## Chứng chỉ

Java Enterprise Edition Specialization (02/2024) · JLPT N4 (08/2025)

---

## Phỏng vấn

Phần dưới để ôn nói, không đưa vào CV nộp. Đồ án kể chính là SLMS, bản đang chạy là một API. EV Dealer dùng khi bị hỏi lỗ hổng. Không kể microservices hay RabbitMQ như việc đã làm: hướng tách User Access Management là kế hoạch nâng cấp sau, repo chưa có.

### Năm câu sơ vấn

**Nhóm và phần em làm.** Nhóm 4. Em giữ onboard nhà/phòng, hợp đồng, PayOS, đồng hồ/OCR, bảo trì và vision. Cổng tài chính chủ đầu tư, gia hạn, API mobile và phần lớn trả phòng là người khác. Nói đúng phạm vi này.

**Vì sao chưa tách service.** Một nhóm, một miền đang chạy ổn trong một API. Tách User Access Management trước chỉ khi nào bắt đầu nâng cấp, vì mọi luồng đều cần danh tính. Hiện chưa làm.

**PayOS và hai OTP.** Đây là hai bước, không phải một bước.

1. Khách thanh toán cọc. PayOS gọi `POST /api/v1/payos/webhook` (public). API kiểm tra chữ ký HMAC. `code = 00` thì `markDepositPaid`: hợp đồng `PAID`, trạng thái `AWAITING_CONFIRM`, tạo hoá đơn cọc nếu chưa có mã hoá đơn đó.
2. Webhook không gửi OTP. Khách thuê, khi hợp đồng đã `PAID` và đang `AWAITING_CONFIRM`, gọi API gửi mã. Hệ thống tạo hai mục đích: `CONTRACT_CONFIRM_TENANT` và `CONTRACT_CONFIRM_MANAGER`.
3. Mỗi bên nhập mã của mình. `confirmContract` ghi `managerOtpVerifiedAt`, `confirmContractByTenant` ghi `tenantOtpVerifiedAt`.
4. `tryActivateAfterDualOtp` đọc lại dòng hợp đồng có khóa. Cả hai mốc đã có thì `activateContract` một lần: `ACTIVE`, sinh tiền nhà kỳ đầu, bắn sự kiện.

Câu nên nói: “Webhook chỉ xác nhận tiền. Chữ ký điện tử là hai OTP sau đó. Hợp đồng chỉ `ACTIVE` khi đủ hai mốc.”

Điểm phải nhận nếu bị hỏi code: `OtpDeliveryOverride` đang gửi mọi OTP về một số cố định (demo, tiết kiệm Twilio), chưa gửi đúng số trên hợp đồng. Webhook sai chữ ký hoặc lỗi xử lý vẫn trả HTTP 200. Nhánh đã trả tiền thì không tạo hoá đơn cọc lần hai, vì mã hoá đơn là khóa duy nhất.

**JWT đang chạy.** HMAC, hạn mặc định 10 ngày (`JWT_EXPIRATION` = 864000000 ms), không refresh, không thu hồi. Đủ cho đồ án. Cách làm production nằm ở mục dưới.

**Schema.** `ddl-auto: update`, không Flyway/Liquibase. Hibernate sửa bảng lúc khởi động. Đủ cho đồ án một máy. Cách làm production nằm ở mục dưới.

### JWT theo hướng production

Mục tiêu: request vẫn stateless, token bị lấy cắp chỉ sống ngắn, đăng xuất và đổi mật khẩu có hiệu lực.

- Access token JWT: 15–60 phút. Claim chỉ những gì service cần để phân quyền (`sub`, role). Ký HMAC bằng secret đọc từ biến môi trường, secret đủ dài và không commit.
- Refresh token: chuỗi ngẫu nhiên dài, không phải JWT sống nhiều ngày. Database chỉ lưu hash (SHA-256). Hạn vài ngày đến vài tuần. Mỗi lần đổi access thì phát refresh mới và vô hiệu refresh cũ (xoay vòng).
- Bảng gợi ý: `user_id`, `token_hash`, `expires_at`, `revoked_at`, `replaced_by`. Unique trên `token_hash`.
- Đăng nhập trả cặp token. Gọi refresh: hash không tồn tại, hết hạn hoặc đã thu hồi thì từ chối. Nếu một refresh đã xoay vẫn bị dùng lại, thu hồi cả chuỗi của user đó (dấu hiệu token bị đánh cắp).
- Đổi mật khẩu và đăng xuất hết thiết bị: thu hồi mọi refresh của user. Access còn sống tối đa bằng hạn access, nên hạn phải ngắn.
- Cần chặn access ngay (đăng xuất tức thì): denylist `jti` trong Redis, TTL bằng thời gian access còn lại. Không làm denylist nếu chưa cần, vì mỗi request sẽ đụng store.
- Client web: refresh để trong cookie `HttpOnly`, `Secure`, `SameSite`. Client mobile: có thể nhận refresh trong body và giữ ở kho bảo mật của máy. Access để header `Authorization: Bearer`. CSRF vẫn tắt trên API Bearer; cookie refresh thì endpoint refresh cần chống CSRF hoặc gắn cookie `SameSite` chặt và chỉ gửi đúng site.

Câu chốt: “Đồ án để access 10 ngày cho tiện demo. Production em tách access ngắn và refresh xoay vòng, chỉ lưu hash, đổi mật khẩu thì xoá refresh.”

### Migration theo hướng production

Mục tiêu: schema đổi có review, chạy một lần, nhiều instance khởi động không sửa bảng cạnh nhau.

- Prod để `ddl-auto: validate` (hoặc tắt generate DDL). App chỉ kiểm tra entity khớp bảng. Lệch thì không lên, thay vì tự `ALTER`.
- Flyway (hoặc Liquibase) chạy trước khi nhận request. File `V1__init.sql`, `V2__add_column.sql` trong git. Bảng `flyway_schema_history` ghi version đã chạy. File đã chạy trên môi trường chung thì không sửa; đổi tiếp bằng version mới.
- Seed demo tách khỏi schema. Prod không chạy tài khoản demo.
- Đổi cột đang có dữ liệu theo hai pha: thêm cột mới cho phép null và code ghi cả hai nơi, deploy, backfill, rồi mới siết `NOT NULL` hoặc xoá cột cũ. Tránh một script khoá bảng lớn và phá bản app đang chạy.
- Local vẫn được để Hibernate cập nhật khi thử entity. Trước khi merge, đối chiếu diff thành một file migration và bật `validate` ở profile chạy chung.
- Unique và khóa ngoại đặt trong SQL, không chỉ trong Java. Idempotency của PayOS dựa vào unique `orderCode` / mã hoá đơn trên database, không chỉ câu `if` trong service.

Câu chốt: “Đồ án dùng `update` cho nhanh. Môi trường chung em chuyển `validate` và để Flyway áp version. Không sửa migration đã chạy.”

### Webhook thanh toán theo hướng production

Phần SLMS đã có: endpoint public, kiểm tra chữ ký, ghi nhận theo `orderCode`, hoá đơn cọc không tạo lần hai.

Phần cần nói thêm khi bị hỏi “đưa lên thật thì sửa gì”:

- Sai chữ ký trả 401. Trả 200 làm nhà cung cấp hiểu là đã xử lý.
- Lỗi sau khi đã xác thực chữ ký, nếu transaction chưa commit, trả 5xx để PayOS gửi lại. Trả 200 khi exception làm mất lần retry.
- Retry an toàn vì unique trên mã đơn / mã hoá đơn. Hết tiền và đã `PAID` thì lần sau chỉ thoát, không cộng tiền lần hai.
- Đối soát thêm số tiền và mã đơn trong payload với bản ghi mình tạo lúc phát QR, rồi mới đổi trạng thái, trong cùng một transaction.
- Không gửi OTP, không kích hoạt hợp đồng trong webhook. Webhook chỉ là sự kiện “tiền đã vào”.

### Điểm yếu khác, nói ngắn

| Điểm yếu | Câu nhận |
|---|---|
| EV Dealer phân quyền không đều. Quên mật khẩu còn đòi JWT. `ROLE_EVMMANAGER` gần như chưa gắn method security. | Forgot/reset để public. API ghi dữ liệu gắn `@PreAuthorize`. Service vẫn kiểm tra đúng đại lý. |
| EV Dealer để mật khẩu database, JWT, VNPay, SMTP trong `application.properties`. | Secret đọc từ biến môi trường. Key đã lọt git thì xoay. |
| Cron, STOMP, SSE, ONNX nằm một JVM. Broker STOMP trong RAM. Ảnh upload vào đĩa container, không volume riêng. | Một instance đủ cho demo. Nhiều bản thì cron cần lock, socket qua broker dùng chung, ảnh lên object storage. |
| Cron EV Dealer chạy mỗi phút trong khi comment ghi mỗi ngày. Giờ job theo JVM, Jackson để UTC. | Một lần mỗi ngày, gắn timezone, gọi service để rule trạng thái nằm một chỗ. |

Việc làm trước nếu được hỏi “cải thiện gì trên chính code này”: test webhook PayOS (sai chữ ký, gọi lại, lệch số tiền), rồi Flyway + `validate`, rồi rút hạn access và thêm refresh. Tách service để sau ba việc đó.
