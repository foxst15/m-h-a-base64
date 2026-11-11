# HỆ THỐNG QUẢN LÝ SINH VIÊN – KIẾN TRÚC XÁC THỰC

**Xây dựng hệ thống đăng nhập an toàn với cơ chế bảo mật nhiều lớp, chống dò mật khẩu tự động (Brute-force) và phân quyền người dùng.**

---

## 1. Giới thiệu dự án

Hệ Thống Quản Lý Sinh Viên – Kiến Trúc Xác Thực là dự án tập trung xây dựng cơ chế đăng nhập, quản lý tài khoản và phân quyền người dùng.

Dự án hướng đến việc hạn chế các rủi ro thường gặp trong quá trình xác thực như lưu trữ mật khẩu dạng văn bản thuần, dò mật khẩu tự động và truy cập trái phép vào khu vực quản trị.

Hệ thống được thiết kế theo mô hình Serverless, sử dụng JavaScript phía trình duyệt và Cloud Firestore để lưu trữ dữ liệu.

## 2. Giới thiệu nhóm thực hiện

**NHÓM 4 – GỒM 6 THÀNH VIÊN**

| STT | Họ và tên | Vai trò |
|---|---|---|
| 1 | Nguyễn Văn Hải | Trưởng nhóm |
| 2 | Nguyễn Thanh Bông | Thành viên |
| 3 | Lâm Duy Kha | Thành viên |
| 4 | Hồ Văn Siêu | Thành viên |
| 5 | Trần Minh Tú | Thành viên |
| 6 | Đinh Thiên Tú | Thành viên |

## 3. Mục tiêu dự án

- Xây dựng quy trình đăng nhập và đổi mật khẩu có bước xác minh.
- Hạn chế lưu mật khẩu gốc dưới dạng văn bản thuần.
- Giới hạn số lần đăng nhập sai.
- Phân quyền giữa quản trị viên và sinh viên.
- Quản lý phiên đăng nhập.
- Lưu trữ dữ liệu tài khoản trên Cloud Firestore.

## 4. Công nghệ sử dụng

| Công nghệ | Chức năng |
|---|---|
| JavaScript | Xử lý logic xác thực |
| Web Crypto API | Băm dữ liệu SHA-256 |
| Base64 / Unicode | Tiền xử lý chuỗi đầu vào |
| Homoglyph | Làm rối biểu diễn ký tự |
| LocalStorage | Lưu trạng thái phiên và thời gian khóa |
| Cloud Firestore | Lưu trữ dữ liệu NoSQL |

## 5. Kiến trúc hệ thống

Hệ thống hoạt động theo mô hình Serverless:

1. Trình duyệt Client tiếp nhận thông tin và xử lý thuật toán.
2. Cloud Firestore lưu trữ dữ liệu tài khoản.
3. Client kết nối trực tiếp với Firestore, không thông qua Backend trung gian.

## 6. Quy trình xử lý mật khẩu

Hệ thống sử dụng ba bước xử lý:

**Bước 1: Base64**
- Chuẩn hóa chuỗi đầu vào.
- Hỗ trợ xử lý ký tự Unicode và tiếng Việt có dấu.

**Bước 2: Homoglyph Substitution**
- Thay thế một số ký tự bằng ký tự Unicode có hình dạng tương tự.

**Bước 3: SHA-256**
- Sử dụng Web Crypto API để tạo mã băm một chiều 256-bit.

### Sơ đồ xử lý

```text
Mật khẩu đầu vào
       |
       v
    Base64
       |
       v
   Homoglyph
       |
       v
    SHA-256
       |
       v
   Mã băm cuối cùng
```

## 7. Cơ chế chống Brute-force

- Theo dõi số lần nhập sai mật khẩu.
- Khi nhập sai quá 5 lần, khóa đăng nhập trong 60 giây.
- Lưu thời điểm khóa trong LocalStorage.
- Kiểm tra trạng thái khóa khi tải lại trang.
- Vô hiệu hóa nút đăng nhập trong thời gian khóa.

## 8. Quản lý tài khoản và phân quyền

### Quản trị viên (Admin)

- Khởi tạo tài khoản.
- Quản lý và xóa tài khoản.
- Theo dõi thông tin tài khoản.
- Quản lý dữ liệu người dùng.

### Sinh viên (Student)

- Truy cập cổng thông tin sinh viên.
- Kiểm tra phiên đăng nhập.
- Đổi mật khẩu sau khi xác minh mật khẩu hiện tại.
- Truy cập các chức năng được cấp quyền.

## 9. Quy trình đổi mật khẩu

1. Người dùng nhập mật khẩu hiện tại.
2. Hệ thống xử lý và đối chiếu mã băm.
3. Nếu xác minh thành công, hệ thống cập nhật mã băm mật khẩu mới.

## 10. Quản lý phiên đăng nhập

Thông tin phiên đăng nhập được lưu trong LocalStorage theo cấu trúc:

```json
{
  "uid": "BONGNL123",
  "role": "sinh_vien"
}
```

Khi tải lại trang, hệ thống kiểm tra phiên và quyền truy cập để điều hướng người dùng phù hợp.

## 11. Cấu trúc dữ liệu

Dữ liệu tài khoản trên Cloud Firestore có các trường:

- `uid`: Mã định danh tài khoản.
- `role`: Vai trò tài khoản (`admin` hoặc `sinh_vien`).
- `passwordHash`: Mã băm mật khẩu.
- `createdBy`: Thông tin người tạo tài khoản.

## 12. Lưu ý bảo mật

Base64 và Homoglyph chỉ là kỹ thuật biến đổi hoặc làm rối dữ liệu, không phải phương pháp mã hóa bảo mật. SHA-256 đơn lẻ cũng không phải lựa chọn phù hợp để lưu mật khẩu trong môi trường thực tế vì đây là hàm băm nhanh và không tích hợp salt.

LocalStorage có thể bị người dùng chỉnh sửa nên không được xem là nguồn xác thực hoặc phân quyền đáng tin cậy.

Khi triển khai thực tế, nên sử dụng Firebase Authentication hoặc cơ chế xác thực phía máy chủ phù hợp, kết hợp Firestore Security Rules, thuật toán băm mật khẩu chuyên dụng như Argon2id hoặc bcrypt và cơ chế giới hạn đăng nhập ở phía đáng tin cậy.

## 13. Tổng kết

Dự án xây dựng mô hình xác thực cho hệ thống quản lý sinh viên với các nội dung chính:

- Xử lý mật khẩu nhiều bước.
- Giới hạn đăng nhập sai.
- Quản lý phiên đăng nhập.
- Phân quyền người dùng.
- Lưu trữ dữ liệu trên Cloud Firestore.

Đây là nền tảng giúp tìm hiểu cách xây dựng hệ thống đăng nhập và quản lý người dùng theo kiến trúc Serverless.

---

**HỆ THỐNG QUẢN LÝ SINH VIÊN – NHÓM 4**

*Authentication Architecture*
