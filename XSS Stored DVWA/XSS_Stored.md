# BÁO CÁO QUÁ TRÌNH KHAI THÁC XSS Stored (LOW, MEDIUM, HIGH)

---

## 1. XSS STORED (LOW)

Đầu tiên, giao diện xuất hiện với hai *text field* cho phép người dùng nhập dữ liệu là *name* và *message*:

<img width="1538" height="680" alt="image" src="https://github.com/user-attachments/assets/28a77458-ed02-43cf-a9c8-413f528912aa" />

Kiểm tra page source thì nhận thấy trường name chỉ có thể nhập tối đa 10 ký tự, trong khi message là 50 ký tự:

<img width="1534" height="649" alt="image" src="https://github.com/user-attachments/assets/344d69ca-06bc-445c-b62f-9ba243011b4f" />

Nhập thử dữ liệu hợp lệ cho hai trường name và message, sau đó nhấn nút Sign Guestbook, ta thấy giao diện sẽ xuất hiện một nhãn dán chứa nội dung dữ liệu của name và message, có thể thấy dữ liệu được lưu vào trong CSDL của trang web → có thể tồn tại XSS stored.

<img width="1533" height="661" alt="image" src="https://github.com/user-attachments/assets/98037daa-8b76-4366-aa2f-9e729094f601" />

Xem source PHP nhận thấy rằng, dữ liệu input nhập vào không được kiểm tra và được lưu thẳng vào database mà không lọc, trường message được xử lý bằng hàm *stripslashes($message)*, tuy nhiên nó chỉ loại bỏ ký tự \ và không hề xử lý hay vô hiệu hóa các thẻ HTML hoặc mã JavaScript.

<img width="1539" height="579" alt="image" src="https://github.com/user-attachments/assets/6b04d713-1896-4593-978c-bc3ba426d199" />

Thực hiện nhập dữ liệu trong trường message với đoạn mã Javascript như sau, sau đó ấn Sign Guestbook:

```bash
<script>alert(“ngoctai”)</script>
```

Nhận thấy giao diện một message box cảnh báo 

<img width="1535" height="661" alt="image" src="https://github.com/user-attachments/assets/605f0ee3-dbce-4cc7-9172-e3a3b6d9a64b" />

→ Lỗ hổng XSS stored.

Thử chỉnh sửa độ dài ký tự tối đa của trường message lên thành 250 ký tự, sau đó viết mã Javascript sử dụng window.location để gửi cookie hiện tại về cho máy chủ HTTP local trên port 8080:

<img width="1541" height="719" alt="image" src="https://github.com/user-attachments/assets/9579e7fd-3e0d-49cb-a1b2-0c10360ccf14" />

Kết quả sau khi ấn nút Sign Guestbook:

<img width="1534" height="667" alt="image" src="https://github.com/user-attachments/assets/b46524c4-ae1e-4b68-b4c2-737bc0c582ee" />

<img width="1530" height="719" alt="image" src="https://github.com/user-attachments/assets/344bcdb6-2582-4d5d-9553-b2eb099bbb5f" />

---

## 2. XSS STORED (MEDIUM)

Kiểm tra source PHP:

<img width="1536" height="724" alt="image" src="https://github.com/user-attachments/assets/abbc1b59-0692-447d-89bc-059d45ed80e5" />

&rarr; Có thể thấy trường message được xử lý bằng hàm *strip_tags* xóa tất cả các thẻ HTML như <script>, <b>, <img>,... và hàm *addslashes* thêm ký tự escape (\) trước ', ", \, NULL cùng với *htmlspecialchars* sẽ chuyển các ký tự HTML thành dạng an toàn: < → &lt, > → &gt,...

<img width="1537" height="669" alt="image" src="https://github.com/user-attachments/assets/a68f09bc-eda7-48d2-99ef-70b2fd0033c4" />

Tuy nhiên, trường name chỉ được xử lý bằng *str_replace('<script>')*, dùng để loại bỏ đúng cụm <script> ra khỏi dữ liệu input:

<img width="1534" height="706" alt="image" src="https://github.com/user-attachments/assets/24f8fed7-832e-4eac-ac4f-af8ce290b8f1" />

Do đó ta thực hiện thử bằng các thẻ tag khác như <svg … > hoặc các biến dạng khác của <script> như <scr<script>ipt> để bypass.

<img width="1536" height="698" alt="image" src="https://github.com/user-attachments/assets/14ba131d-34e0-4988-9d57-06d3173471c4" />

<img width="1530" height="708" alt="image" src="https://github.com/user-attachments/assets/e8f163aa-1dc1-4521-a5d9-cdb0ee1fa9fe" />

<img width="1528" height="695" alt="image" src="https://github.com/user-attachments/assets/33cc45b7-402c-42c0-a48a-8ab86b5baba4" />

<img width="1532" height="702" alt="image" src="https://github.com/user-attachments/assets/13658d57-9a3b-4afd-b650-a3ac594bad0b" />

&rarr; Kết quả bypass thành công bằng hai cách trên.

---

## 3. XSS STORED (HIGH)

Kiểm tra source PHP:

<img width="1541" height="709" alt="image" src="https://github.com/user-attachments/assets/e2083337-ba41-4189-ba1f-7daf60d547df" />

&rarr; Nhận thấy rằng dữ liệu trường name được xử lý bằng cách sử dụng preg_replace('/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i'), nó sẽ thực hiện tìm và xóa tất cả biến thể của từ <script>, tức tất cả nội dung khớp với pattern này sẽ bị xóa khỏi biến name, ví dụ như các dạng: <script>, <sCrIpT>, <scri<script>pt>, <s...c...r...i...p…t>,...

Do chỉ xóa mọi dạng <script> viết biến thể nên ta có thể sử dụng các thẻ tag khác như: <img …> hay <svg …>,... để khai thác.

<img width="1535" height="653" alt="image" src="https://github.com/user-attachments/assets/29a6f3f5-1912-43d5-a4fc-59081c7057de" />

<img width="1547" height="681" alt="image" src="https://github.com/user-attachments/assets/12c936c8-cd55-4feb-83a6-935bfd8e4ac8" />

Thực hiện tạo payload chuyển hướng người dùng server HTTP máy mình, hi ảnh không tải được (src="x"), trình duyệt thực hiện onerror:

<img width="1538" height="457" alt="image" src="https://github.com/user-attachments/assets/bad5b34d-0eb4-4550-ab5c-c6705148627b" />

<img width="1533" height="669" alt="image" src="https://github.com/user-attachments/assets/8d7e32e8-1958-4931-8f19-a320a3041534" />

Thử tạo dữ liệu của trường name là một dữ liệu liên kết bằng cách tạo một thẻ HTML <a> và có chứa mã JavaScript trong thuộc tính sự kiện onclick: 

```bash
<a href=”#” onclick=”alert(1)”>Clickme</a>
```

<img width="1538" height="645" alt="image" src="https://github.com/user-attachments/assets/057f9c6d-6b5b-4233-b28c-d6c1a162b00c" />

<img width="1537" height="612" alt="image" src="https://github.com/user-attachments/assets/a85b94a3-5176-4df4-bcb5-ce6aa71f9fa9" />

Khi click vào *click me* lỗ hổng XSS stored sẽ được kích hoạt.

<img width="1531" height="650" alt="image" src="https://github.com/user-attachments/assets/cf85eb98-571f-4679-b25d-15c2783aaa97" />

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng XSS Stored trên lab DVWA.

---
