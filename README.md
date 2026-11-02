# HCM-KS26-CNTT1_NhapMonCNTT_Session18_BTVN1
# BÀI 1: SỰ CỐ KHÔNG TRUY CẬP ĐƯỢC HỆ THỐNG QUẢN TRỊ

**Họ và tên:** Phạm Thành Đạo  
**Lớp:** HCM-KS26-CNTT1  
**Môn học:** Nhập môn CNTT  
**Session:** 16 – Ex01

---

## PHẦN 1. PHÂN TÍCH

### 1.1. Xác định Input và Output của DNS

DNS (Domain Name System) là hệ thống phân giải tên miền, có chức năng chuyển đổi tên miền mà con người dễ ghi nhớ thành địa chỉ IP mà máy tính sử dụng để kết nối với máy chủ.

- **Input:** Tên miền, ví dụ: `phong-dieu-phoi.greenlog.vn`.
- **Output:** Địa chỉ IP của máy chủ tương ứng, ví dụ minh họa: `203.0.113.10`. Đây là IP giả lập dùng để minh họa, không phải IP thật của hệ thống GreenLogistics.

Có thể hiểu DNS giống như một cuốn danh bạ Internet: người dùng tìm kiếm bằng tên miền, hệ thống tra cứu để tìm địa chỉ IP tương ứng.

### 1.2. Tiến trình DNS Lookup

Khi Nam nhập `phong-dieu-phoi.greenlog.vn` vào trình duyệt, quá trình phân giải DNS diễn ra theo các bước:

**Bước 1: Kiểm tra bộ nhớ đệm trên máy tính và trình duyệt**

Máy tính kiểm tra xem địa chỉ IP của tên miền đã được lưu trong DNS Cache hay chưa. Nếu có bản ghi còn hiệu lực, máy tính có thể sử dụng ngay mà không cần tra cứu lại từ đầu.

**Bước 2: Gửi truy vấn đến DNS Resolver**

Nếu không tìm thấy kết quả trong bộ nhớ đệm, máy tính gửi truy vấn đến DNS Resolver, thường do nhà cung cấp Internet hoặc một dịch vụ DNS công cộng cung cấp.

**Bước 3: DNS Resolver truy vấn hệ thống DNS**

Nếu Resolver chưa có kết quả trong bộ nhớ đệm, nó sẽ lần lượt tìm kiếm thông tin qua hệ thống DNS:

- **Root DNS Server:** Hướng dẫn Resolver đến máy chủ quản lý tên miền cấp cao nhất.
- **TLD DNS Server:** Tìm máy chủ DNS có thẩm quyền quản lý phần mở rộng `.vn`.
- **Authoritative DNS Server:** Tra cứu bản ghi của tên miền `greenlog.vn` và tìm địa chỉ IP tương ứng với máy chủ `phong-dieu-phoi`.

**Bước 4: Trả kết quả về máy tính**

Sau khi tìm được bản ghi phù hợp, DNS Resolver trả địa chỉ IP về cho máy tính. Kết quả có thể được lưu trong bộ nhớ đệm theo thời gian TTL.

**Bước 5: Trình duyệt kết nối đến máy chủ**

Trình duyệt sử dụng địa chỉ IP vừa nhận được để thiết lập kết nối đến máy chủ, sau đó gửi yêu cầu HTTP hoặc HTTPS để lấy nội dung trang quản trị.

### 1.3. Sơ đồ luồng dữ liệu DNS

```text
Nam nhập tên miền vào trình duyệt
              |
              v
      Kiểm tra DNS Cache
              |
       +------+------+
       |             |
   Đã có IP       Chưa có IP
       |             |
       |             v
       |       DNS Resolver
       |             |
       |             v
       |        Root DNS
       |             |
       |             v
       |         TLD DNS
       |             |
       |             v
       |    Authoritative DNS
       |             |
       |             v
       |       Trả về địa chỉ IP
       |             |
       +------+------+
              |
              v
     Trình duyệt kết nối Server
              |
              v
       Gửi HTTP/HTTPS Request
              |
              v
       Nhận HTTP Response
```

### 1.4. Phân tích nguyên nhân lỗi DNS_PROBE_FINISHED_NXDOMAIN

Trong tình huống này, Nam đã nhập sai tên miền:

- Tên miền đúng: `phong-dieu-phoi.greenlog.vn`.
- Tên miền nhập sai: `phong-dieu-phoi.grenlog.vn`.

Nam đã thiếu một chữ `e` trong phần tên miền `greenlog.vn`, khiến trình duyệt tìm kiếm một tên miền khác là `grenlog.vn`.

DNS không tự hiểu rằng người dùng muốn truy cập `greenlog.vn`. Hệ thống chỉ tra cứu chính xác tên miền được yêu cầu. Nếu tên miền sai không tồn tại hoặc không có bản ghi DNS phù hợp, Resolver trả về kết quả **NXDOMAIN (Non-Existent Domain)**.

Hậu quả:

- Trình duyệt không tìm được địa chỉ IP của tên miền đã nhập.
- Quá trình phân giải DNS thất bại.
- Trình duyệt chưa thể gửi yêu cầu HTTP đến máy chủ quản trị dự kiến.
- Nam không thể truy cập hệ thống để cập nhật lộ trình xe tải.

Vì vậy, chưa có căn cứ để kết luận máy chủ công ty bị sập nguồn. Nguyên nhân trực tiếp trong tình huống là nhập sai tên miền.

---

## PHẦN 2. TRIỂN KHAI

### 2.1. Phân biệt lỗi DNS và lỗi HTTP 500

| Tiêu chí | DNS_PROBE_FINISHED_NXDOMAIN | HTTP 500 Internal Server Error |
|---|---|---|
| Bản chất | Không phân giải được tên miền | Máy chủ gặp lỗi khi xử lý yêu cầu |
| Giai đoạn xảy ra | Trước khi trình duyệt kết nối được đến máy chủ qua tên miền đó | Sau khi yêu cầu đã đến máy chủ |
| Nguyên nhân thường gặp | Sai tên miền, tên miền chưa tồn tại, thiếu bản ghi DNS | Lỗi chương trình phía Server, cấu hình hoặc cơ sở dữ liệu |
| Phía cần kiểm tra | Tên miền, DNS Cache, DNS Resolver, cấu hình mạng | Mã nguồn Backend, log máy chủ, Database, cấu hình Server |
| Có HTTP Response không? | Không nhận được HTTP Response từ máy chủ đích do chưa phân giải được tên miền | Có HTTP Response với mã trạng thái 500 |

**Kết luận:**

- Lỗi DNS thường xuất hiện ở giai đoạn phân giải tên miền, trước khi trình duyệt kết nối được đến máy chủ đích. Nguyên nhân có thể nằm ở phía người dùng, cấu hình DNS hoặc hệ thống DNS của tên miền.
- Lỗi HTTP 500 cho thấy máy chủ đã nhận được yêu cầu nhưng gặp lỗi nội bộ trong quá trình xử lý. Đây là lỗi phía Server.

### 2.2. Quy trình kiểm tra HTTP Status Code bằng Chrome DevTools (F12)

Sau khi Nam sửa lại tên miền chính xác, có thể thực hiện các bước sau để kiểm tra yêu cầu đã đến máy chủ hay chưa:

**Bước 1: Mở trang quản trị**

Truy cập:

`http://phong-dieu-phoi.greenlog.vn/`

**Bước 2: Mở công cụ Chrome DevTools**

Nhấn phím `F12` hoặc tổ hợp `Command + Option + I` trên macOS.

**Bước 3: Chọn tab Network**

- Chọn mục **Network**.
- Tích chọn **Preserve log** nếu muốn giữ lại lịch sử yêu cầu khi chuyển trang.
- Nhấn nút tải lại trang (Reload).

**Bước 4: Quan sát danh sách Request**

Chọn yêu cầu chính của trang, thường có loại tài nguyên là `document`.

Kiểm tra các thông tin:

- **Request URL:** Địa chỉ mà trình duyệt đang gửi yêu cầu.
- **Status Code:** Mã trạng thái HTTP máy chủ trả về.
- **Remote Address:** Địa chỉ IP và cổng kết nối của máy chủ khi có thông tin.
- **Response:** Nội dung phản hồi từ máy chủ nếu có.

**Bước 5: Đọc kết quả**

| Mã trạng thái | Ý nghĩa | Nhận định |
|---|---|---|
| 200 OK | Yêu cầu thành công | Máy chủ xử lý và trả nội dung thành công |
| 301/302 | Chuyển hướng | Máy chủ yêu cầu trình duyệt truy cập địa chỉ khác |
| 400 Bad Request | Yêu cầu không hợp lệ | Cần kiểm tra dữ liệu gửi lên |
| 403 Forbidden | Bị từ chối truy cập | Máy chủ hiểu yêu cầu nhưng không cho phép truy cập |
| 404 Not Found | Không tìm thấy tài nguyên | Máy chủ không tìm thấy trang hoặc tài nguyên được yêu cầu |
| 500 Internal Server Error | Lỗi nội bộ máy chủ | Cần kiểm tra Backend hoặc cấu hình Server |
| 502 Bad Gateway | Lỗi phản hồi từ máy chủ trung gian | Cần kiểm tra Gateway hoặc máy chủ phía sau |
| 503 Service Unavailable | Dịch vụ tạm thời không khả dụng | Máy chủ có thể quá tải hoặc đang bảo trì |

**Lưu ý:** Nếu trình duyệt báo `DNS_PROBE_FINISHED_NXDOMAIN`, thường sẽ không có HTTP Status Code của máy chủ đích vì yêu cầu chưa thể được gửi đến máy chủ thông qua tên miền đó.

### 2.3. Một số lệnh kiểm tra DNS trên macOS Terminal

Có thể sử dụng Terminal để kiểm tra tên miền và đối chiếu kết quả:

Kiểm tra tên miền đúng:

```bash
nslookup phong-dieu-phoi.greenlog.vn
```

Kiểm tra tên miền bị nhập sai:

```bash
nslookup phong-dieu-phoi.grenlog.vn
```

Hoặc sử dụng lệnh:

```bash
dig phong-dieu-phoi.greenlog.vn
```

Nếu tên miền không tồn tại hoặc không có bản ghi phù hợp, kết quả có thể hiển thị `NXDOMAIN`.

Lưu ý: Không nên kết luận chỉ dựa vào một lần truy vấn. Cần kiểm tra tên miền, DNS Resolver và trạng thái bản ghi DNS trước khi đưa ra kết luận cuối cùng.

---

## PHẦN 3. KIỂM THỬ

### 3.1. Tình huống giả định

Sau khi Nam sửa tên miền thành:

`phong-dieu-phoi.greenlog.vn`

Trình duyệt không còn báo lỗi DNS nhưng lại nhận được mã trạng thái:

**404 Not Found**

### 3.2. Phân tích nguyên nhân

Mã HTTP 404 Not Found cho biết máy chủ đã nhận được yêu cầu HTTP nhưng không tìm thấy tài nguyên tương ứng với đường dẫn mà trình duyệt yêu cầu.

Một số nguyên nhân có thể xảy ra:

- Đường dẫn `/` chưa được cấu hình để hiển thị trang quản trị.
- Trang quản trị đã bị xóa hoặc di chuyển sang đường dẫn khác.
- Lập trình viên nhập sai URL hoặc Endpoint.
- Cấu hình định tuyến (Routing) của ứng dụng chưa chính xác.
- Tài nguyên được yêu cầu không tồn tại trên máy chủ.

### 3.3. Hướng xử lý

1. Kiểm tra lại Request URL trong tab Network.
2. Xác nhận đường dẫn truy cập có chính xác hay không.
3. Kiểm tra cấu hình Routing của ứng dụng Backend.
4. Kiểm tra tài nguyên hoặc trang quản trị có tồn tại trên máy chủ hay không.
5. Kiểm tra log máy chủ để xác định nguyên nhân cụ thể.

**Kết luận:** Lỗi 404 khác với lỗi DNS. Khi nhận được HTTP 404, quá trình phân giải DNS và kết nối đến máy chủ đã đủ thành công để máy chủ trả về phản hồi HTTP. Vấn đề nằm ở tài nguyên hoặc đường dẫn được yêu cầu, không đồng nghĩa toàn bộ máy chủ bị sập.

---

## KẾT LUẬN

Qua tình huống của Nam tại GreenLogistics, có thể thấy không phải mọi lỗi truy cập website đều xuất phát từ máy chủ. Cần xác định lỗi xảy ra ở giai đoạn nào trước khi đưa ra kết luận.

- **DNS:** Chuyển đổi tên miền thành địa chỉ IP.
- **NXDOMAIN:** Tên miền được truy vấn không tồn tại hoặc không được DNS phân giải.
- **HTTP 404:** Máy chủ không tìm thấy tài nguyên được yêu cầu.
- **HTTP 500:** Máy chủ gặp lỗi nội bộ khi xử lý yêu cầu.

Trong trường hợp này, Nam cần sửa tên miền từ `grenlog.vn` thành `greenlog.vn` trước khi tiếp tục kiểm tra hệ thống. Việc sử dụng Chrome DevTools (F12), tab Network và các lệnh `nslookup`, `dig` giúp xác định lỗi chính xác hơn, tránh nhầm lẫn giữa sự cố DNS và sự cố máy chủ.
