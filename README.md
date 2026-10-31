# Bài 1 — Quản trị cấu hình tập trung cho chuỗi hiệu thuốc

Đề bài: https://portal.rikkei.edu.vn/learn/159/homework/8288

Hai ứng dụng Spring Boot dùng cùng một kho Git: Config Server (cổng 8888) đọc
`pharmacy-service.properties` ở gốc kho, còn Pharmacy Service lấy cấu hình đó
khi khởi động. Cấu hình PostgreSQL và thông tin chi nhánh không nằm trong mã nguồn
Pharmacy Service.

## Yêu cầu

- JDK 21, PostgreSQL, Git.
- Spring Boot 3.5.6, Spring Cloud 2025.0.0, Gradle Wrapper 8.14.3.
- PostgreSQL có database `pharmacy_db` và tài khoản có quyền kết nối.

Máy đang mặc định dùng JDK 26 cần chọn JDK 21 trước khi chạy Gradle, ví dụ
trên macOS: `export JAVA_HOME=$(/usr/libexec/java_home -v 21)`.

## Chạy

Clone hoặc tải repository này, rồi chạy lệnh từ thư mục gốc của repository.
Config Server mặc định đọc chính Git repository này; file cấu hình phải được
commit trên nhánh `main` trước khi chạy. Để đọc repository Git khác, đặt biến
`CONFIG_REPO_URI` thành URL HTTPS hoặc `file:///duong/dan/repo`.

Terminal 1:

```bash
./gradlew :config-server:bootRun
```

Xác nhận Config Server trả dữ liệu từ Git:

```bash
curl -fsS http://localhost:8888/pharmacy-service/default
```

JSON phải có `spring.datasource.url`, `app.branch-name`, `app.hotline` trong
`propertySources` và `version` là mã commit Git. Có thể dùng `python3 -m json.tool`
để đọc JSON dễ hơn. Không đưa mật khẩu thật lên Git; giá trị `${DB_PASSWORD}`
được Pharmacy Service lấy từ biến môi trường của chính nó.

Terminal 2:

```bash
DB_PASSWORD='mat-khau-postgresql-cua-ban' ./gradlew :pharmacy-service:bootRun
```

Nếu tên tài khoản, database hoặc cổng khác mặc định, đặt thêm `DB_USERNAME`,
`DB_NAME` hoặc `DB_PORT` trong môi trường Terminal 2. Khi chạy thành công,
console in `PostgreSQL connected`, `Branch: Nha Thuoc So 1` và
`Hotline: 1900xxxx`. Service cố ý mở một kết nối thật lúc khởi động để báo lỗi
ngay khi cấu hình database sai.

## Cấu trúc

```text
pharmacy-service.properties       Cấu hình tập trung, được commit trong Git
config-server/                    Đọc Git và phục vụ cấu hình trên cổng 8888
pharmacy-service/                Config Client và kiểm tra kết nối PostgreSQL
```

Đổi tên chi nhánh hoặc hotline trong `pharmacy-service.properties`, commit và
khởi động lại Pharmacy Service để thấy giá trị mới. Nếu Config Server đọc
GitHub từ xa, push commit trước khi khởi động lại service.

## Kiểm tra build

```bash
./gradlew clean test bootJar
```

Nộp URL của repository GitHub chứa toàn bộ project này trên hệ thống Rikkei.
