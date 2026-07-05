# Hướng Dẫn Chạy Dự Án Pockie (Local Environment)

Tài liệu này hướng dẫn chi tiết cách khởi chạy toàn bộ hệ sinh thái Pockie trên môi trường Local.

## Bước 1: Khởi chạy Database & Storage (Docker)

Hệ thống yêu cầu **PostgreSQL** (Database) và **MinIO** (Object Storage để lưu ảnh/file eKYC). Các cấu hình này đã có sẵn trong thư mục `deploy-vps/docker-compose.yml`.

1. Mở terminal và di chuyển vào thư mục `deploy-vps`:
   ```bash
   cd deploy-vps
   ```
2. Chạy ngầm các service hạ tầng (chỉ chạy Database và MinIO để dev ở local):
   ```bash
   docker-compose up -d pockie-postgres pockie-minio
   ```
*(Lưu ý: Đảm bảo máy bạn đã cài Docker và Docker Compose).*

---

## Bước 2: Khởi chạy Backend API (`core-api-pockie`)

Backend được viết bằng **NestJS** và dùng **Prisma ORM**.

1. Di chuyển vào thư mục Backend:
   ```bash
   cd core-api-pockie
   ```
2. Cài đặt các thư viện:
   ```bash
   npm install
   ```
3. Khởi tạo Prisma Client và chạy Migration để tạo các bảng trong Database:
   ```bash
   npx prisma generate
   npx prisma migrate dev
   ```
4. Khởi chạy Backend ở chế độ development (tự động reload khi sửa code):
   ```bash
   npm run start:dev
   ```
*(Backend thường sẽ chạy ở cổng `http://localhost:3000` hoặc cổng được chỉ định trong file `.env`).*

---

## Bước 3: Khởi chạy các Frontend Web (React / Vite)

Dự án có 3 Repo Frontend, tất cả đều dùng Vite. Bạn cần mở **3 terminal khác nhau** cho từng web:

### 1. Ứng dụng cho người dùng (User Web)
```bash
cd user-web-pockie
npm install
npm run dev
```

### 2. Ứng dụng cho khách hàng doanh nghiệp (Customer Web)
```bash
cd customer-web-pockie
npm install
npm run dev
```

### 3. Ứng dụng quản trị nội bộ (Internal Web)
```bash
cd internal-web-pockie
npm install
npm run dev
```

*Khi chạy lệnh `npm run dev`, Vite sẽ hiển thị URL ở terminal (thường là `http://localhost:5173`, `5174`, `5175`).*

---

## Bước 4: Khởi chạy Mobile App (`app_pockie`)

Mobile app được viết bằng **Flutter**.

1. Đảm bảo bạn đang mở máy ảo (Android Emulator / iOS Simulator) hoặc đã cắm thiết bị thật.
2. Di chuyển vào thư mục app:
   ```bash
   cd app_pockie
   ```
3. Cài đặt và tải các thư viện về:
   ```bash
   flutter pub get
   ```
4. Khởi chạy ứng dụng:
   ```bash
   flutter run
   ```

---

## Lưu Ý Quan Trọng
- **Biến môi trường (`.env`)**: Nếu dự án không chạy được, hãy kiểm tra xem bạn đã copy file `.env.example` thành `.env` cho từng repo chưa (đặc biệt là repo backend cần cấu hình URL kết nối với Postgres và MinIO).
- **Lỗi Port**: Đảm bảo các cổng mặc định không bị chiếm dụng trước khi chạy (ví dụ `5432` cho Postgres, `9000` cho Minio).
