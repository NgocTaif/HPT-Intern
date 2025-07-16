# BÁO CÁO QUÁ TRÌNH KHAI THÁC XSS DOM (LOW, MEDIUM, HIGH)

---

## 1. XSS DOM (LOW)

Đầu tiên, giao diện ban đầu là một page select option:

<img width="1539" height="649" alt="image" src="https://github.com/user-attachments/assets/696b643a-197d-427d-835b-da16d0188a68" />

Kiểm tra page source, và có thể thấy điểm đặc biệt của đoạn mã sau:

<img width="1533" height="361" alt="image" src="https://github.com/user-attachments/assets/16d3fe20-622e-4ac7-8d0f-5648737e8058" />

Đầu tiên, nó kiểm tra xem trong URL hiện tại có chứa chuỗi *"default="* hay không, tức là kiểm tra để lấy giá trị trong URL, và nếu có, thì giá trị ở phần sau *“default=”* sẽ được cắt và gán vào giá trị của biến *lang*.

Tiếp theo, nó sẽ chèn giá trị lang vào trong thẻ <option> trong DOM mà không bất kỳ bộ lọc, hay kiểm tra nào thông qua: 

```bash
document.write("<option value='" + lang + "'>" + decodeURI(lang) + "</option>");
```

Lúc này, document.write() sẽ ghi trực tiếp vào HTML, và trình duyệt sẽ render và  thực thi bất kỳ đoạn mã JS nào được chèn vào.

Trên URL, ta viết đoạn mã JS lấy cookie hiện tại sau phần “default=”:

<img width="1536" height="686" alt="image" src="https://github.com/user-attachments/assets/a49669e1-d02f-42cd-9bf0-7fc26a1516bb" />
<img width="1534" height="624" alt="image" src="https://github.com/user-attachments/assets/8b1ea9ba-f4f2-4855-aec1-77cc70492c4f" />

&rarr; Kết quả có thể thấy đoạn mã đã được chèn vào trực tiếp trong trang trong phần value:

<img width="1540" height="612" alt="image" src="https://github.com/user-attachments/assets/e5d72ed1-a0c3-407b-87e6-ec186bc3d61d" />

---

## 2. XSS DOM (MEDIUM)

Kiểm tra source PHP, ta thấy nó ngăn người dùng chèn mã <script> vào tham số default trong URL, nếu dữ liệu có <script> thì sẽ redirect về trang với default=English:

<img width="1538" height="686" alt="image" src="https://github.com/user-attachments/assets/4c0f1b9c-eac9-48c3-90d8-b2d67be3c173" />

&rarr; Do đó ta có thể thực chèn mã thẻ khác với thẻ <script> như: <img …>, <svg …>, ...

<img width="1534" height="632" alt="image" src="https://github.com/user-attachments/assets/34f8483a-9ada-43cc-880b-9e1407fc2987" />

<img width="1540" height="627" alt="image" src="https://github.com/user-attachments/assets/7f7b64b1-b28d-4f4c-9363-f6cdd5d0c971" />

Đoạn mã được chèn trực tiếp vào value trong phần <option> của trang.

<img width="1534" height="336" alt="image" src="https://github.com/user-attachments/assets/e37ec2d4-da88-436d-8dd8-521e0dd4ff36" />

Để hợp thức hóa cú pháp HTML hơn, ta có thể thêm </select> trước đoạn mã JS, để thẻ <svg> thoát khỏi thẻ <select>, giúp chèn thẻ <svg> độc lập ngoài DOM form control:

<img width="1535" height="338" alt="image" src="https://github.com/user-attachments/assets/b51c93cd-7490-4926-b0b9-99f03622b0c9" />

<img width="1535" height="361" alt="image" src="https://github.com/user-attachments/assets/5f278c79-f8ad-4d97-af1f-108ea45b8038" />

Kết quả có thể thấy thẻ <svg> đã độc lập ra khỏi thẻ <select>

<img width="1539" height="337" alt="image" src="https://github.com/user-attachments/assets/a21479ac-7784-4a2b-b49e-ab7b00e6d989" />

---

## 3. XSS DOM (HIGH)

Kiểm tra source PHP, ta thấy nó sử dụng whitelist để kiểm tra giá trị, và chỉ chấp nhận các giá trị cụ thể: English, French, German, Spanish. Nếu không đúng giá trị nào trong whitelist, redirect về default=English:

<img width="1533" height="747" alt="image" src="https://github.com/user-attachments/assets/52553ee6-6197-4863-bd4b-cfa7cc1bc264" />

Do đó cần tìm lỗi phía client-side (JavaScript).
Sử dụng kỹ thuật thêm dấu # vào sau giá trị hợp lệ và trước mã JS ta cần chèn:

```bash
?default=French#<img src=x onerror=alert(1)>
```

PHP sẽ chỉ thấy phần **?default=French**, vì nội dung sau dấu # là fragment (hash) không được gửi lên server.

Tức là server vẫn sẽ nhận: 

```bash
GET /vulnerabilities/xss_d/?default=French HTTP/1.1 
```

Tuy nhiên phía client JS, vẫn sẽ đọc toàn bộ *document.location.href*, lấy chuỗi sau default và gán toàn bộ cho biến lang và chèn thẳng vào DOM.

```bash
var lang = "French#<img src=x onerror=alert(1)>"
```

<img width="1437" height="764" alt="image" src="https://github.com/user-attachments/assets/0ac9888c-21e0-4af3-aaf1-62fcce2abd0f" />

&rarr; Kết quả thu được:
<img width="1448" height="600" alt="image" src="https://github.com/user-attachments/assets/8540db54-15ba-45db-af7f-e4ddbd78c794" />

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng XSS DOM trên lab DVWA.

---
