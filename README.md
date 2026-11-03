# Bài 7: Gỡ lỗi luồng yêu cầu kết quả xét nghiệm và sai lệch mã trạng thái HTTP

## Phần 1: Phân tích vấn đề

### 1. Bốn điểm sai lệch trong bản giải trình của thực tập sinh

| # | Chủ đề | Điểm sai lệch | Nguyên lý đúng |
|---|---|---|---|
| 1 | Vai trò DNS | Trình duyệt gửi trực tiếp chuỗi URL tới Web Server | Trình duyệt phải nhờ DNS dịch tên miền thành địa chỉ IP của máy chủ trước, sau đó mới gửi được yêu cầu |
| 2 | Quy trình TCP/TLS | Cho rằng kết nối được mở "không điều kiện" | HTTPS bắt buộc bắt tay TCP/TLS để thiết lập kết nối an toàn rồi mới tải được dữ liệu |
| 3 | Mã lỗi HTTP | Cho rằng hồ sơ bị xóa vĩnh viễn nên server trả mã 404 | Lỗi do phía máy chủ (backend, cơ sở dữ liệu) phải trả mã 500 Internal Server Error; hồ sơ vẫn còn nguyên |
| 4 | Sửa DOM trên tab Elements | Cho rằng sửa chữ trên Elements là sửa dữ liệu thật | Chỉnh sửa chỉ thay đổi tạm thời trên máy cá nhân, không ghi ngược lại máy chủ; nhấn F5 là hiển thị lại nội dung gốc |

### 2. Vì sao máy chủ cơ sở dữ liệu sập phải trả mã 500, không phải 404?

- **404 Not Found** dùng khi đường dẫn hoặc tài nguyên được yêu cầu không tồn tại.
- Khi cơ sở dữ liệu hoặc dịch vụ backend bị sập, lỗi nằm ở phía máy chủ, đường dẫn vẫn đúng và hồ sơ vẫn còn, nên mã chuẩn là **500 Internal Server Error**.

**Hậu quả nếu báo sai thành 404:**

- Bệnh nhân hiểu nhầm rằng hồ sơ bệnh án hoặc kết quả xét nghiệm đã bị xóa vĩnh viễn.
- Gây hoang mang, lo lắng và làm mất niềm tin nghiêm trọng vào hệ thống.
- Mã 500 cho thấy đây là sự cố tạm thời phía hệ thống, bệnh nhân có thể thử lại sau.

---

## Phần 2: Sửa lỗi và hoàn thiện giải pháp

### 3. Biên bản giải trình kỹ thuật theo 4 bước vòng đời HTTP Request

1. **DNS Lookup:** khi bệnh nhân bấm xem kết quả, trình duyệt gửi yêu cầu đến hệ thống DNS để chuyển tên miền thành địa chỉ IP của Web Server.
2. **Kết nối TCP/TLS:** sau khi có IP, trình duyệt thực hiện bắt tay TCP/TLS để thiết lập đường truyền HTTPS an toàn giữa điện thoại của bệnh nhân và Web Server.
3. **HTTP Request:** trình duyệt gửi yêu cầu lấy nội dung từ Web Server. Do máy chủ gặp sự cố, hệ thống trả về mã **500 Internal Server Error** (hồ sơ của bệnh nhân vẫn còn nguyên vẹn).
4. **Browser Render:** trình duyệt nhận phản hồi lỗi từ máy chủ và hiển thị trang thông báo lỗi lên màn hình.

### 4. Vì sao sửa chữ trên tab Elements không có giá trị lưu trữ trên máy chủ CarePulse?

- Tab Elements chỉ can thiệp vào **cây DOM** do trình duyệt dựng ra và giữ tạm trong **bộ nhớ RAM** của thiết bị cá nhân.
- Thay đổi này không gửi yêu cầu nào về máy chủ, nên dữ liệu trong cơ sở dữ liệu CarePulse không bị ảnh hưởng.
- Khi nhấn F5, trình duyệt tải lại HTML từ máy chủ và dựng lại DOM, toàn bộ chỉnh sửa tạm trên RAM bị xóa và nội dung gốc hiển thị lại.
