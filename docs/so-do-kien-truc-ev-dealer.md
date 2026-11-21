# Sơ đồ kiến trúc — EV Dealer System

Backend quản lý đại lý xe điện (đồ án SWP391). Hệ thống theo dõi mẫu xe, số VIN trong kho, khách hàng, đơn cọc và thanh toán, giao xe, lịch lái thử / bảo dưỡng. Repo này là **một API monolith** (không có frontend trong source).

Tài liệu này tóm tắt kiến trúc đang chạy trong code, không phải bản vẽ UML có sẵn trong repo.

## 1. Bối cảnh hệ thống

Ba tầng: client → API Spring Boot → SQL Server. Thanh toán đi qua VNPay sandbox. Email đi qua SMTP Gmail. PDF hợp đồng và phiếu giao xe được sinh trong process.

```mermaid
flowchart LR
    subgraph clients [Client]
        Web["Web / Postman<br/>localhost"]
    end

    subgraph app [Server]
        API["EV Dealer API<br/>Spring Boot 3.5 · Java 21<br/>Gradle · :8080"]
    end

    subgraph data [Database]
        MSSQL[("SQL Server<br/>evdealer1 · :1433")]
    end

    subgraph external [Dịch vụ ngoài]
        VNPay["VNPay sandbox<br/>vpcpay.html"]
        SMTP["SMTP Gmail<br/>:587 STARTTLS"]
    end

    Web --> API
    API --> MSSQL
    API -->|tạo URL thanh toán| VNPay
    VNPay -->|return + IPN| API
    API -->|welcome · reset mật khẩu| SMTP
```

Không có reverse proxy, Docker hay WebSocket trong repo. Ảnh xe chỉ lưu `image_url` trên entity, API không nhận file ảnh.

## 2. Kiến trúc phân lớp trong API

Một process Spring Boot, stateless. Request đi từ controller xuống service, MapStruct, repository, entity. Không có message broker.

```mermaid
flowchart TB
    subgraph presentation [Tầng giao tiếp]
        REST["REST /api<br/>17 controller"]
        Docs["Swagger / OpenAPI<br/>springdoc"]
    end

    subgraph cross [Cắt ngang]
        JWT["JwtAuthenticationFilter<br/>@Secured · @PreAuthorize"]
        EX["GlobalExceptionHandler<br/>ApiError"]
        VAL["Bean Validation"]
    end

    subgraph application [Tầng nghiệp vụ]
        SVC["service + *ServiceImpl"]
        PDF["PdfGenerationService<br/>iText 7 + font Noto Sans"]
        MAIL["MailService<br/>JavaMail @Async"]
        XLS["Apache POI<br/>import mẫu xe"]
        CRON["OrderScheduledTasks<br/>mỗi phút"]
        UTIL["VnPayUtil · VinGenerator<br/>AuthenticationHelper"]
    end

    subgraph persistence [Tầng dữ liệu]
        MAP["MapStruct mapper"]
        REPO["Spring Data JPA"]
        ENT["JPA entity<br/>ddl-auto: update<br/>open-in-view: false"]
    end

    REST --> JWT --> SVC
    SVC --> PDF
    SVC --> MAIL
    SVC --> XLS
    SVC --> UTIL
    SVC --> MAP --> REPO --> ENT
    CRON --> REPO
```

Luồng một request đã đăng nhập:

1. `JwtAuthenticationFilter` đọc `Authorization: Bearer`, `JwtService` lấy email (subject), `JpaUserDetailsService` nạp user.
2. Method security (`@Secured` hoặc `@PreAuthorize`) kiểm tra role trên một số controller. Nhiều API chỉ yêu cầu đã đăng nhập; phạm vi đại lý được kiểm tra trong service.
3. Controller nhận DTO request, gọi interface trong `service`.
4. `*ServiceImpl` áp rule nghiệp vụ, ghi entity qua repository.
5. MapStruct trả DTO response.
6. Lỗi đi qua `GlobalExceptionHandler` và body `ApiError`.

Phiên HTTP là **stateless** (`SessionCreationPolicy.STATELESS`). CSRF tắt vì client gửi JWT.

Endpoint công khai: `/api/auth/**` (đăng ký, đăng nhập), Swagger (`/swagger-ui/**`, `/v3/api-docs/**`), callback VNPay `/api/payments/vnpay/**`, và preflight `OPTIONS`. `VnpayController` (`/api/vnpay`) đang comment, không đăng ký bean.

`/api/password/forgot` và `/api/password/reset` **không** nằm trong danh sách public, nên filter hiện tại vẫn đòi JWT.

## 3. Nhóm nghiệp vụ

Các module nằm chung một codebase, tách theo controller và service, không phải microservice.

```mermaid
flowchart TB
    API["EV Dealer API"]

    API --> Auth["Tài khoản<br/>auth · user · role · mật khẩu"]
    API --> Catalog["Danh mục xe<br/>model · electric vehicle"]
    API --> Stock["Kho<br/>dealership · warehouse · VIN"]
    API --> Sale["Bán hàng<br/>customer · order · payment"]
    API --> Paper["Chứng từ<br/>hợp đồng cọc · phiếu giao xe"]
    API --> After["Hậu mãi<br/>slot · appointment · service record"]

    Catalog --> Stock
    Stock --> Sale
    Sale --> Paper
    Sale --> After
```

| Nhóm | Việc chính | Điểm vào API |
|---|---|---|
| Tài khoản | Đăng ký, đăng nhập JWT, CRUD user, role, quên mật khẩu qua email | `/api/auth`, `/api/users`, `/api/roles`, `/api/password` |
| Danh mục xe | Mẫu xe, cấu hình xe điện, import Excel | `/api/models`, `/api/electric-vehicles` |
| Kho | Đại lý, kho, tồn theo model, chuyển kho, VIN | `/api/dealerships`, `/api/warehouses` |
| Bán hàng | Khách, đơn cọc, trả nốt, giao xe | `/api/customers`, `/api/orders` |
| Thanh toán | Tiền mặt hoặc VNPay (cọc / phần còn lại), lịch sử | `/api/payments` |
| Chứng từ | PDF hợp đồng đặt cọc và phiếu giao xe | `/api/contracts`, `/api/orders/{id}/delivery-slip` |
| Hậu mãi | Khung giờ, lịch hẹn lái thử / bảo dưỡng, phiếu dịch vụ | `/api/slots`, `/api/appointments`, `/api/service-entitys`, `/api/service-records` |

Role lưu trong bảng `roles`, tên kèm prefix: `ROLE_USER`, `ROLE_ADMIN`, `ROLE_EVMSTAFF`, `ROLE_EVMMANAGER`. JWT nhét claim `role`.

- `ROLE_ADMIN`: CRUD user, model, xe; bỏ qua kiểm tra đại lý khi bán xe và xem kho.
- `ROLE_EVMSTAFF`: xem model và xe; bán trong đại lý mình thuộc.
- `ROLE_EVMMANAGER`: role có trong seed; phân quyền method chủ yếu chưa gắn role này.
- `ROLE_USER`: role mặc định lúc đăng ký.

Phạm vi dữ liệu: nhân viên không phải admin chỉ bán VIN thuộc đúng `dealership` của mình (`OrderServiceImpl`). Kho cũng lọc theo role trong `WarehouseServiceImpl`.

## 4. Dữ liệu

SQL Server, database `evdealer1`. Hibernate `ddl-auto: update` cập nhật bảng lúc khởi động. Script tạo bảng và seed nằm ở `src/main/resources/db/` (`evdealer_table.sql`, `evdealer_key.sql`, `evdealer_data.sql`), chạy tay, không gắn vào startup.

| Nhóm | Bảng / entity |
|---|---|
| Tổ chức | `Role`, `User`, `Dealership`, `PasswordResetToken` |
| Sản phẩm | `Model`, `ElectricVehicle` |
| Kho | `Warehouse`, `WarehouseStock`, `VehicleSerial` |
| Bán hàng | `Customer`, `Order`, `Payment` |
| Hậu mãi | `Slot`, `Appointment`, `ServiceEntity`, `ServiceRecord` |

Trạng thái đơn (`OrderStatus`): `PROCESSING` → `ORDER_PAID` → `DELIVERING` → `COMPLETED`, hoặc `CANCELED`.

Trạng thái VIN (`VehicleStatus`): `AVAILABLE`, `HOLD`, `UNDELIVERED`, `DELIVERING`, `DELIVERED`, `SOLD_OUT`.

Thanh toán: `PaymentMethod` là `CASH` hoặc `VNPAY`; `PaymentPurpose` là `DEPOSIT` hoặc `REMAINING`. `OrderPaymentStatus` theo dõi phần tiền: `UNPAID`, `DEPOSIT_PAID`, `PAID`, `OVERDUE`.

## 5. Luồng cắt ngang

### Xác thực

```mermaid
sequenceDiagram
    participant C as Client
    participant A as AuthController
    participant S as AuthService
    participant DB as SQL Server

    C->>A: POST /api/auth/login
    A->>S: email + password
    S->>DB: User + BCrypt
    S-->>C: JWT (subject = email, claim role, hạn 60 phút)
    C->>A: request kèm Bearer
    Note over A: JwtAuthenticationFilter<br/>@Secured hoặc @PreAuthorize
```

Mật khẩu hash bằng BCrypt. JWT ký HMAC-SHA256 (`jjwt` 0.11.5). Hạn đọc từ `jwt.exp-minutes` (mặc định 60).

Quên mật khẩu: `PasswordService` tạo `PasswordResetToken`, `MailService.sendResetPasswordEmail` gửi link (method đánh `@Async`). Tạo user admin / staff cũng gửi email chào mừng qua SMTP.

### Đơn hàng và thanh toán

```mermaid
sequenceDiagram
    participant C as Client
    participant O as OrderService
    participant P as PaymentService
    participant V as VNPay
    participant DB as SQL Server

    C->>O: POST /api/orders/deposit
    O->>DB: đơn PROCESSING, VIN = HOLD
    C->>P: POST /api/payments/vnpay/start hoặc /cash
    alt VNPay
        P-->>C: URL sandbox
        V->>P: GET /api/payments/vnpay/return và /ipn
    else Tiền mặt
        P->>DB: ghi Payment PAID
    end
    Note over P,DB: Cọc xong: VIN HOLD<br/>Trả nốt: đơn ORDER_PAID, VIN UNDELIVERED
    C->>O: PUT /orders/{id}/delivery hoặc POST deliver-now
    O->>DB: DELIVERING rồi COMPLETED, VIN DELIVERED
```

`VNPAYService` ký query theo `vnpay.tmnCode`, `vnpay.hashSecret`, `vnpay.url`. Return URL và IPN trỏ `http://localhost:8080/api/payments/vnpay/return|ipn`. Callback public, xác thực bằng chữ ký VNPay trong service.

### Chứng từ PDF

`PdfGenerationServiceImpl` (iText 7) vẽ hợp đồng đặt cọc và phiếu giao xe, nhúng font `src/main/resources/fonts/NotoSans-*.ttf`. Thông tin bên bán đọc từ `evdealer.company.*`. API trả `application/pdf`.

### Import Excel

`POST /api/electric-vehicles/import-vehicle-type-excel` đọc workbook bằng Apache POI và tạo `ElectricVehicle` (status mặc định `AVAILABLE` nếu ô trạng thái không hợp lệ).

### Job định kỳ

`@EnableScheduling` trên `EvdealersystemApplication`. Một job:

| Job | Lịch | Việc |
|---|---|---|
| `autoCompleteDeliveringOrders` | `0 * * * * ?` (mỗi phút) | Đơn `DELIVERING` có `deliveryDate` ≤ hôm nay → đơn `COMPLETED`, VIN `DELIVERED`, xóa `holdUntil` |

Comment trong code nói 01:00 mỗi ngày; cron đang chạy mỗi phút. Múi giờ job là giờ JVM (`LocalDate.now()`). Jackson và Hibernate đặt `UTC`.

## 6. Triển khai

```mermaid
flowchart LR
    Dev["Máy dev<br/>gradle bootRun<br/>:8080"] --> DB[("SQL Server local<br/>localhost:1433<br/>database evdealer1")]
    VNPay["VNPay sandbox"] -->|return / IPN về localhost| Dev
    Dev --> SMTP["smtp.gmail.com:587"]
```

Không có `Dockerfile` hay workflow CI trong repo. Chạy local bằng Gradle, JDK 21.

Cấu hình nằm thẳng trong `src/main/resources/application.properties` (không tách profile, không đọc `.env`):

| Nhóm key | Việc |
|---|---|
| `spring.datasource.*` | JDBC SQL Server, user `sa`, pool Hikari tối đa 10 |
| `jwt.secret`, `jwt.exp-minutes` | Ký và hạn JWT |
| `vnpay.*` | Sandbox VNPay |
| `spring.mail.*` | SMTP Gmail |
| `evdealer.company.*` | Thông tin in trên PDF |

`MailConfig` trong package `config` đang comment; mail dùng auto-config của `spring-boot-starter-mail`.

## 7. Ràng buộc của kiến trúc hiện tại

- Một deployable: API, cron, sinh PDF và gửi mail cùng một JVM. Không có broker, cache hay hàng đợi ngoài.
- Schema vừa có script SQL tay, vừa để Hibernate `ddl-auto: update`. Hai nguồn có thể lệch nhau.
- Phân quyền không đều: catalog và user gắn `@Secured` / `@PreAuthorize`; đơn, thanh toán, kho, lịch hẹn chủ yếu chỉ cần đã đăng nhập, rồi service tự so sánh đại lý.
- Quên mật khẩu và đặt lại mật khẩu chưa được `permitAll`, trong khi login mới là endpoint public.
- Callback VNPay và cron giả định một instance trên `localhost`. IPN sandbox không tới được máy dev nếu không có tunnel.
- `Thymeleaf` có trong `build.gradle` nhưng không có template nào dùng nó. PDF vẽ bằng iText.
- `group` trong Gradle vẫn để `HeathCareSystem.example` — dấu vết project gốc, không ảnh hưởng runtime.
