# Hướng Dẫn Khởi Chạy Dự Án Pockie (Môi trường Local)

Tài liệu này cung cấp các bước chi tiết để khởi chạy toàn bộ hệ sinh thái Pockie trên môi trường phát triển (Local), phục vụ cho quá trình đánh giá và chấm thi. Hệ thống bao gồm 1 Backend (NestJS), 3 Frontend Web (React/Vite), 1 Mobile App (Flutter), cùng hệ thống cơ sở hạ tầng (PostgreSQL, MinIO) được đóng gói qua Docker.

## 0. Yêu Cầu Hệ Thống (Prerequisites)
Trước khi bắt đầu, cần đảm bảo hệ thống đã cài đặt sẵn các công cụ sau:
- **Node.js**: Phiên bản `v18.x` trở lên (Khuyến nghị dùng công cụ quản lý phiên bản như `nvm`).
- **Docker** & **Docker Compose**: Dành cho việc khởi chạy Database và Object Storage độc lập.
- **Flutter SDK**: Phiên bản `3.19.x` trở lên (Dành riêng cho việc biên dịch và chạy Mobile App).
- Trình duyệt web (Google Chrome, Microsoft Edge, hoặc Safari).

---

## Bước 1: Khởi Chạy Hạ Tầng (Database & Storage)
Hệ thống yêu cầu **PostgreSQL** (cơ sở dữ liệu chính) và **MinIO** (lưu trữ file, ảnh chụp eKYC, tài liệu). Cấu hình đã được cung cấp sẵn thông qua file `docker-compose.yml`.

1. Di chuyển vào thư mục chứa cấu hình Docker:
   ```bash
   cd deploy-vps
   ```
2. Khởi chạy các dịch vụ hạ tầng ở chế độ ngầm (detached mode):
   ```bash
   docker-compose up -d pockie-postgres pockie-minio
   ```
3. Kiểm tra trạng thái các container để đảm bảo hệ thống đã chạy:
   ```bash
   docker ps
   ```
   *Kết quả mong đợi: Có 2 container trạng thái `Up`, trong đó Postgres lắng nghe ở cổng `5432` và MinIO lắng nghe ở cổng `9000` & `9001`.*

---

## Bước 2: Cấu Hình & Khởi Chạy Backend API (`core-api-pockie`)
Backend API đóng vai trò xử lý nghiệp vụ chính, giao tiếp cơ sở dữ liệu và tích hợp các API của VNPT.

1. Di chuyển vào thư mục Backend:
   ```bash
   cd core-api-pockie
   ```
2. Cài đặt các gói thư viện phụ thuộc:
   ```bash
   npm install
   ```
3. Cấu hình biến môi trường:
   Dự án đã đính kèm file cấu hình mẫu `.env.example`. Cần tạo file `.env` chính thức bằng lệnh sau:
   ```bash
   cp .env.example .env
   ```
   *Lưu ý: File `.env.example` đã được cấu hình sẵn các chuỗi kết nối (`DATABASE_URL`) trỏ tới PostgreSQL và MinIO vừa được khởi chạy ở Bước 1. Không cần thiết lập thủ công trừ khi có thay đổi cổng mạng ở máy cục bộ.*
4. Khởi tạo Prisma Client và đồng bộ cấu trúc Database (Migration):
   ```bash
   npx prisma generate
   npx prisma migrate dev
   ```
5. Khởi chạy Backend ở chế độ phát triển:
   ```bash
   npm run start:dev
   ```
   *Kết quả mong đợi: Terminal hiển thị thông báo khởi tạo thành công và ứng dụng lắng nghe tại `http://localhost:3000`. Có thể truy cập `http://localhost:3000/api` để xem tài liệu chi tiết (Swagger UI).*

---

## Bước 3: Khởi Chạy Hệ Sinh Thái Frontend Web (React / Vite)
Hệ thống gồm 3 ứng dụng Web độc lập, đóng vai trò phục vụ 3 nhóm đối tượng khác nhau. Cần mở **3 cửa sổ Terminal riêng biệt** để khởi chạy từng dự án.

### 3.1. User Web (Dành cho Người dùng cuối)
Hệ thống ví cá nhân, quản lý chi tiêu và tích hợp AI Chat, Smart Scan.
```bash
cd user-web-pockie
npm install
npm run dev
```
*Truy cập ứng dụng tại: `http://localhost:5173`*

### 3.2. Customer Web (Dành cho Khách hàng Doanh nghiệp B2B)
Hệ thống đăng ký, tích hợp và quản lý dịch vụ dành cho đối tác.
```bash
cd customer-web-pockie
npm install
npm run dev
```
*Truy cập ứng dụng tại: `http://localhost:5174`*

### 3.3. Internal Web (Dành cho Quản trị viên Pockie)
Hệ thống nội bộ phục vụ kiểm duyệt hồ sơ eKYC, phân tích chỉ số rủi ro, vận hành.
```bash
cd internal-web-pockie
npm install
npm run dev
```
*Truy cập ứng dụng tại: `http://localhost:5175`*

---

## Bước 4: Khởi Chạy Mobile App (`app_pockie`)
Ứng dụng di động đa nền tảng mang lại trải nghiệm tương tự User Web nhưng hỗ trợ sâu hơn các tính năng phần cứng (Camera, NFC).

1. Bật trình mô phỏng (Android Emulator / iOS Simulator) hoặc kết nối thiết bị vật lý (đã bật chế độ USB Debugging).
2. Di chuyển vào thư mục chứa mã nguồn App:
   ```bash
   cd app_pockie
   ```
3. Cài đặt các package Dart:
   ```bash
   flutter pub get
   ```
4. Biên dịch và khởi chạy ứng dụng lên thiết bị:
   ```bash
   flutter run
   ```

---

## Các Vấn Đề Thường Gặp (Troubleshooting)
- **Xung đột cổng mạng (Port Conflict)**: Cần đảm bảo các cổng `3000`, `5432`, `9000` không bị chiếm dụng bởi các phần mềm khác đang chạy ngầm trên máy (ví dụ: máy đã cài sẵn PostgreSQL local từ trước).
- **Lỗi kết nối cơ sở dữ liệu ở Backend**: Kiểm tra lại tiến trình Docker ở Bước 1. Đảm bảo container `pockie-postgres` đang hoạt động bình thường.
- **Lỗi thiếu Prisma Client**: Nếu tiến trình Backend báo lỗi `Cannot find module '@prisma/client'`, yêu cầu dừng Backend và chạy lại lệnh `npx prisma generate` trong thư mục `core-api-pockie`.
