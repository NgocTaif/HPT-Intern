# BÁO CÁO QUÁ TRÌNH KHAI THÁC CSRF (LOW, MEDIUM, HIGH)

---

## 1. CSRF (LOW)

Đầu tiên, giao diện là một trang cho phép người dùng thực hiện thay đổi mật khẩu của tài khoản hiện tại đang đăng nhập:

<img width="1439" height="504" alt="image" src="https://github.com/user-attachments/assets/4659f1f8-0a57-43b4-b2ac-b14b9ac572af" />

Kiểm tra source PHP:

<img width="1487" height="457" alt="image" src="https://github.com/user-attachments/assets/0754b3bc-2693-4025-b6ba-f79a782b8f5e" />

&rarr; Ta có thể thấy, nó không hề kiểm tra xem form gửi có chứa mã csrf_token nào không và Dùng $_GET thay vì $_POST khiến yêu cầu dễ bị giả mạo từ một link độc hại, nó cũng không kiểm tra Referer, Origin, hay xác minh bất cứ gì từ phía client. Do đó mà nếu nếu người dùng đang đăng nhập (có cookie), và kẻ tấn công có thể tạo một link đổi mật khẩu trên trình duyệt để user thực hiện click, lệnh đổi mật khẩu sẽ chạy thành công. Đó là lý do lỗ hổng này hay thường dùng cùng với lỗ hổng XSS.

Do đó mà giả sử nếu người dùng đang có phiên đăng nhập thực hiện truy cập vào đường link do attacker gửi/chèn/cài/phising: 

```bash
/dvwa/vulnerabilities/csrf/?password_new=123&password_conf=123&Change=Change 
```

Lúc này lệnh thay đổi mật khẩu sẽ được thực hiện vì nó chỉ cần xác minh dựa trên session cookie lúc này của người dùng (PHPSESSID).

<img width="1442" height="508" alt="image" src="https://github.com/user-attachments/assets/1a6b47db-1949-4d75-8612-5a01391ca39e" />

Do đó, mật khẩu cũ là “password” sẽ bị thay đổi và không thể thực hiện đăng nhập:

<img width="1433" height="575" alt="image" src="https://github.com/user-attachments/assets/87d9c3e4-32c4-41bd-ab70-7c4964cfb383" />

Đăng nhập với mật khẩu là “123” thì thành công:

<img width="1439" height="586" alt="image" src="https://github.com/user-attachments/assets/9de931a6-5d35-461b-bde2-6eacc68f2e73" />

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng CSRF trên lab DVWA.

---
