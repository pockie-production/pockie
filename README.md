# Hướng Dẫn Chạy Dự Án Pockie (Local Environment)

Tài liệu này hướng dẫn chi tiết từ A-Z cách khởi chạy toàn bộ hệ sinh thái Pockie trên môi trường Local, phục vụ cho mục đích phát triển và chấm thi.

## 0. Yêu Cầu Hệ Thống (Prerequisites)
Để đảm bảo dự án chạy ổn định, máy tính của bạn cần được cài đặt sẵn:
- **Node.js**: Phiên bản `v18.x` trở lên.
- **Docker** & **Docker Compose**: Dành cho Database và Object Storage.
- **Flutter SDK**: Phiên bản `3.19.x` trở lên (nếu muốn chạy app).
- Trình duyệt Chrome hoặc các trình duyệt nhân Chromium.

---

## Bước 1: Khởi chạy Database & Storage (Docker)

Hệ thống yêu cầu **PostgreSQL** (Database chính) và **MinIO** (Object Storage để lưu trữ file, ảnh eKYC). Các cấu hình này đã có sẵn trong thư mục `deploy-vps/docker-compose.yml`.

1. Mở terminal và di chuyển vào thư mục `deploy-vps`:
   ```bash
   cd deploy-vps
   ```
2. Khởi chạy ngầm Database và MinIO:
   ```bash
   docker-compose up -d pockie-postgres pockie-minio
   ```
3. *(Optional)* Để chắc chắn, bạn có thể chạy `docker ps` để kiểm tra. Sẽ có 2 container đang chạy ở port `5432` (Postgres) và `9000` (MinIO).

---

## Bước 2: Cấu hình và chạy Backend API (`core-api-pockie`)

Backend được phát triển bằng **NestJS** kết hợp **Prisma ORM**.

1. Mở một terminal mới, di chuyển vào thư mục Backend:
   ```bash
   cd core-api-pockie
   ```
2. Cài đặt các thư viện phụ thuộc:
   ```bash
   npm install
   ```
3. **Cấu hình biến môi trường**: 
   Dự án đã có sẵn file `.env.example`. Hãy tạo một file `.env` bằng cách copy nội dung từ file example:
   ```bash
   cp .env.example .env
   ```
   *(Trong file `.env` đã thiết lập sẵn URL kết nối tới Postgres và Minio ở Bước 1. Mặc định bạn không cần sửa gì thêm).*
4. Khởi tạo Prisma Client và chạy Migration để tạo/cập nhật cấu trúc database:
   ```bash
   npx prisma generate
   npx prisma migrate dev
   ```
5. Khởi chạy Backend ở chế độ development (tự động reload code khi có thay đổi):
   ```bash
   npm run start:dev
   ```
✅ Backend lúc này sẽ lắng nghe tại **`http://localhost:3000`**. Bạn có thể kiểm tra xem API đã sẵn sàng chưa bằng cách truy cập Swagger UI: `http://localhost:3000/api`.

---

## Bước 3: Khởi chạy các Frontend Web (React / Vite)

Hệ sinh thái Pockie bao gồm 3 ứng dụng web độc lập, tất cả đều được build bằng Vite. Bạn cần mở **3 terminal khác nhau** cho từng web. Cách khởi chạy cho cả 3 hoàn toàn giống nhau:

### 3.1. Ứng dụng Web dành cho End-User (User Web)
Đây là màn hình cho người dùng cuối quản lý chi tiêu.
```bash
cd user-web-pockie
npm install
npm run dev
```
✅ Ứng dụng thường sẽ khởi chạy tại: **`http://localhost:5173`**

### 3.2. Ứng dụng Web cho B2B / Doanh nghiệp (Customer Web)
Dành cho đối tác đăng ký dịch vụ của Pockie.
```bash
cd customer-web-pockie
npm install
npm run dev
```
✅ Ứng dụng thường sẽ khởi chạy tại: **`http://localhost:5174`**

### 3.3. Ứng dụng Web Quản trị (Internal Web)
Hệ thống cho Admin Pockie phê duyệt eKYC, theo dõi chỉ số.
```bash
cd internal-web-pockie
npm install
npm run dev
```
✅ Ứng dụng thường sẽ khởi chạy tại: **`http://localhost:5175`**

---

## Bước 4: Khởi chạy Mobile App (`app_pockie`)

Mobile app đa nền tảng được phát triển bằng **Flutter**.

1. Đảm bảo bạn đang bật máy ảo (Android Emulator / iOS Simulator) hoặc đã cắm thiết bị thật (có bật chế độ USB Debugging).
2. Mở một terminal mới và di chuyển vào thư mục app:
   ```bash
   cd app_pockie
   ```
3. Tải tất cả các package/thư viện của Dart:
   ```bash
   flutter pub get
   ```
4. Build và khởi chạy ứng dụng lên thiết bị:
   ```bash
   flutter run
   ```

---

## Những Vấn Đề Thường Gặp (Troubleshooting)
- **Lỗi Port bị trùng**: Đảm bảo cổng `5432` (nếu bạn có cài sẵn Postgres ở máy), cổng `9000` và `3000` không bị ứng dụng khác chiếm dụng trước khi chạy.
- **Lỗi không kết nối được Database**: Kiểm tra file `.env` ở `core-api-pockie` đã trỏ đúng tới địa chỉ `localhost:5432` theo chuẩn của Prisma chưa.
- **Backend báo lỗi thiếu Prisma Client**: Chắc chắn bạn đã chạy lệnh `npx prisma generate` trước khi start Backend.
