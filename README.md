# Hệ Thống Quản Lý Sinh Viên -- Kiến Trúc Xác Thực

```{=html}
<p align="center">
```
`<strong>`{=html}Hệ thống đăng nhập an toàn với cơ chế xác thực nhiều
lớp`</strong>`{=html}`<br>`{=html} Ứng dụng kiến trúc Serverless và
Cloud Firestore
```{=html}
</p>
```

------------------------------------------------------------------------

## Giới thiệu

**Hệ Thống Quản Lý Sinh Viên -- Kiến Trúc Xác Thực** là dự án tập trung
xây dựng cơ chế đăng nhập, quản lý tài khoản và phân quyền người dùng
cho hệ thống quản lý sinh viên.

Dự án hướng đến việc hạn chế các rủi ro thường gặp trong quá trình xác
thực như lưu trữ mật khẩu dạng văn bản thuần, dò mật khẩu tự động
(Brute-force) và truy cập trái phép vào các khu vực quản trị.

Hệ thống sử dụng mô hình Serverless, trong đó trình duyệt xử lý các bước
tiền xử lý và băm mật khẩu, còn Cloud Firestore đảm nhiệm việc lưu trữ
dữ liệu.

## Mục tiêu dự án

-   Xây dựng quy trình đăng nhập và đổi mật khẩu có bước xác minh.
-   Không lưu mật khẩu gốc dưới dạng văn bản thuần trong cơ sở dữ liệu.
-   Bổ sung cơ chế giới hạn số lần đăng nhập sai.
-   Phân quyền rõ ràng giữa quản trị viên và sinh viên.
-   Quản lý phiên đăng nhập và điều hướng theo quyền truy cập.
-   Tổ chức dữ liệu tài khoản trên Cloud Firestore.

## Công nghệ và kiến trúc

  Thành phần         Vai trò
  ------------------ -----------------------------------------------------
  JavaScript         Xử lý logic xác thực phía trình duyệt
  Web Crypto API     Thực hiện băm dữ liệu bằng SHA-256
  Base64 / Unicode   Tiền xử lý chuỗi đầu vào
  Homoglyph          Làm rối biểu diễn ký tự
  LocalStorage       Lưu trạng thái đăng nhập và thông tin khóa tạm thời
  Cloud Firestore    Lưu trữ dữ liệu tài khoản theo mô hình NoSQL

### Mô hình hoạt động

1.  **Trình duyệt Client:** tiếp nhận thông tin đăng nhập và xử lý thuật
    toán ở phía người dùng.
2.  **Cloud Firestore:** lưu trữ và cung cấp dữ liệu tài khoản cho ứng
    dụng.
3.  **Không sử dụng Backend trung gian:** dự án hướng đến mô hình kết
    nối trực tiếp từ Client đến Firestore.

## Cơ chế xử lý mật khẩu

Quy trình xử lý mật khẩu được trình bày qua ba lớp:

1.  **Base64:** chuẩn hóa chuỗi đầu vào thành dạng biểu diễn ASCII.
2.  **Homoglyph Substitution:** thay thế một số ký tự bằng ký tự Unicode
    có hình dạng tương tự.
3.  **SHA-256:** tạo mã băm một chiều thông qua Web Crypto API.

Ví dụ luồng xử lý:

``` text
Mật khẩu người dùng nhập
          |
          v
     Mã hóa Base64
          |
          v
   Thay thế Homoglyph
          |
          v
       SHA-256
          |
          v
      Mã băm lưu trữ
```

Dự án cũng xử lý đầu vào Unicode, bao gồm tiếng Việt có dấu và biểu
tượng, thông qua bước chuyển đổi UTF-8 trước khi mã hóa Base64.

## Chống đăng nhập sai liên tục

Hệ thống triển khai cơ chế giới hạn số lần đăng nhập:

-   Theo dõi số lần nhập sai liên tiếp.
-   Khi nhập sai quá 5 lần, hệ thống khóa đăng nhập tạm thời 60 giây.
-   Lưu thời điểm kết thúc khóa trong LocalStorage.
-   Kiểm tra trạng thái khóa khi tải lại trang.
-   Vô hiệu hóa nút đăng nhập trong thời gian bị khóa.

## Quản lý tài khoản và phân quyền

Hệ thống có hai nhóm người dùng chính:

### Quản trị viên (Admin)

-   Khởi tạo tài khoản.
-   Quản lý và xóa tài khoản.
-   Theo dõi thông tin tài khoản và thời gian tạo.
-   Quản lý dữ liệu người dùng.

### Sinh viên (Student)

-   Truy cập cổng thông tin dành cho sinh viên.
-   Kiểm tra phiên đăng nhập.
-   Cập nhật mật khẩu sau khi xác minh mật khẩu hiện tại.
-   Chỉ truy cập những chức năng được cấp quyền.

## Quy trình đổi mật khẩu

Việc đổi mật khẩu được thực hiện theo quy trình:

1.  Người dùng nhập mật khẩu hiện tại.
2.  Hệ thống xử lý và đối chiếu mã băm với dữ liệu đã lưu.
3.  Nếu xác minh thành công, hệ thống cập nhật mã băm của mật khẩu mới.

## Quản lý phiên đăng nhập

Thông tin phiên được lưu trong LocalStorage theo cấu trúc tham khảo:

``` json
{
  "uid": "BONGNL123",
  "role": "sinh_vien"
}
```

Khi tải lại trang, hệ thống kiểm tra thông tin phiên và quyền truy cập
để điều hướng người dùng đến khu vực phù hợp.

## Cấu trúc dữ liệu

Dữ liệu tài khoản được tổ chức trên Cloud Firestore, bao gồm các thông
tin như:

-   `uid`: mã định danh tài khoản.
-   `role`: vai trò của tài khoản (`admin` hoặc `sinh_vien`).
-   `passwordHash`: mã băm mật khẩu.
-   `createdBy`: thông tin người tạo tài khoản.
-   `createdAt`: thời điểm tạo tài khoản (nếu được triển khai trong dữ
    liệu).

## Lưu ý về bảo mật

Các kỹ thuật Base64 và Homoglyph chỉ là bước biến đổi hoặc làm rối dữ
liệu, **không phải mã hóa bảo mật**. SHA-256 đơn lẻ cũng không phải lựa
chọn phù hợp để lưu mật khẩu trong hệ thống triển khai thực tế vì đây là
hàm băm nhanh và không có cơ chế salt tích hợp.

Ngoài ra, dữ liệu LocalStorage có thể bị người dùng chỉnh sửa, nên không
được xem là nguồn xác thực hoặc phân quyền đáng tin cậy. Với hệ thống
thực tế, cần sử dụng Firebase Authentication hoặc cơ chế xác thực phía
máy chủ phù hợp, kết hợp Firestore Security Rules, thuật toán băm mật
khẩu chuyên dụng như Argon2id hoặc bcrypt và cơ chế giới hạn đăng nhập
được thực thi ở phía đáng tin cậy.

Vì vậy, các cơ chế trong dự án nên được xem là nội dung triển khai và
minh họa kiến trúc xác thực, không phải cam kết rằng hệ thống đã chống
được mọi hình thức tấn công.

## Tổng kết

Dự án xây dựng một mô hình xác thực cho hệ thống quản lý sinh viên với
các nội dung chính: xử lý mật khẩu nhiều bước, giới hạn đăng nhập sai,
quản lý phiên, phân quyền tài khoản và lưu trữ dữ liệu trên Cloud
Firestore.

Đây là nền tảng để tìm hiểu cách tổ chức một hệ thống đăng nhập và quản
lý người dùng theo hướng Serverless.

------------------------------------------------------------------------

```{=html}
<p align="center">
```
`<strong>`{=html}HỆ THỐNG QUẢN LÝ SINH
VIÊN`</strong>`{=html}`<br>`{=html} Authentication Architecture
```{=html}
</p>
```
