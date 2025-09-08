# Lỗ hổng Access Control and Privilege escalation

  <img width="781" height="461" alt="image" src="https://github.com/user-attachments/assets/fc25e90c-c41a-4477-81a0-5c886ebc6b80" />

---

## Vertical access controls

- Vertical access controls are mechanisms that restrict access to sensitive functionality to specific types of users.

- With vertical access controls, different types of users have access to different application functions.

- For example, an administrator might be able to modify or delete any user's account, while an ordinary user has no access to these actions.

--- 

## Horizontal access controls

- Horizontal access controls are mechanisms that restrict access to resources to specific users.

- With horizontal access controls, different users have access to a subset of resources of the same type.

- For example, a banking application will allow a user to view transactions and make payments from their own accounts, but not the accounts of any other user.

--- 

## Context-dependent access controls (phụ thuộc ngữ cảnh)

- Context-dependent access controls restrict access to functionality and resources based upon the state of the application or the user's interaction with it.

- Context-dependent access controls prevent a user performing actions in the wrong order.

- For example, a retail website might prevent users from modifying the contents of their shopping cart after they have made payment.

---

## Vertical privilege escalation

- If a user can gain access to functionality that they are not permitted to access then this is vertical privilege escalation.

- For example, if a non-administrative user can gain access to an admin page where they can delete user accounts, then this is vertical privilege escalation.

### Unprotected functionality

- _/admin_, _/robots.txt_

### Lab Unprotected admin functionality

<img width="1919" height="882" alt="Screenshot 2025-09-05 145659" src="https://github.com/user-attachments/assets/2b1d2d11-05ac-49a2-99a8-5ba841125150" />

<img width="1919" height="927" alt="Screenshot 2025-09-05 145812" src="https://github.com/user-attachments/assets/2bc0fa4c-029b-4705-ba50-70354ca7069d" />

<img width="1919" height="942" alt="Screenshot 2025-09-05 150049" src="https://github.com/user-attachments/assets/42d46410-02e3-4ce4-90e7-bce3df5a4ba0" />

<img width="1919" height="881" alt="Screenshot 2025-09-05 150141" src="https://github.com/user-attachments/assets/b812dd8c-b506-4992-8357-96910475aad2" />

- In some cases, sensitive functionality is concealed by giving it a less predictable URL. This is an example of so-called "security by obscurity".

- However, the application might still leak the URL to users:

  ```html
  <script>
	var isAdmin = false;
	if (isAdmin) {
		...
		var adminPanelTag = document.createElement('a');
		adminPanelTag.setAttribute('href', 'https://insecure-website.com/administrator-panel-yb556');
		adminPanelTag.innerText = 'Admin panel';
		...
	}
  </script>
  ```

### Lab Unprotected admin functionality with unpredictable URL

<img width="1916" height="946" alt="Screenshot 2025-09-05 151013" src="https://github.com/user-attachments/assets/bd8c423d-e04e-4060-b781-d1f8df2719e0" />

<img width="1919" height="947" alt="Screenshot 2025-09-05 151053" src="https://github.com/user-attachments/assets/ba2b909b-8686-45f5-82aa-c72a928372f5" />

<img width="1919" height="939" alt="Screenshot 2025-09-05 151328" src="https://github.com/user-attachments/assets/5bb5cfad-cfe7-4645-a1d1-2f2627bb7819" />

---

### Parameter-based access control methods

- Some applications determine the user's access rights or role at login, and then store this information in a user-controllable location. This could be:

  A hidden field.
  A cookie.
  A preset query string parameter.
  
- The application makes access control decisions based on the submitted value. For example:

  ```http
  https://insecure-website.com/login/home.jsp?admin=true
  https://insecure-website.com/login/home.jsp?role=1
  ```

### Lab User role controlled by request parameter

<img width="1919" height="878" alt="Screenshot 2025-09-05 152236" src="https://github.com/user-attachments/assets/5af4adde-becf-41b1-94e5-07a7e343ad3d" />

<img width="1919" height="889" alt="Screenshot 2025-09-05 155959" src="https://github.com/user-attachments/assets/d1266501-2975-42e0-95b0-1bcbbdd42e81" />

<img width="1919" height="934" alt="Screenshot 2025-09-05 160305" src="https://github.com/user-attachments/assets/7bc1e6d0-9d71-44d0-ae4a-4898a83b47f0" />

<img width="1919" height="934" alt="Screenshot 2025-09-05 160438" src="https://github.com/user-attachments/assets/6ea486c5-124c-4701-9a16-abd2ca4d9027" />

<img width="1919" height="922" alt="Screenshot 2025-09-05 160521" src="https://github.com/user-attachments/assets/b19e8177-2f21-4e24-935d-0131c6f4a3e0" />

<img width="1919" height="929" alt="Screenshot 2025-09-05 160549" src="https://github.com/user-attachments/assets/e50dfa75-b09d-44dc-894a-01c016a2499b" />

<img width="1919" height="941" alt="Screenshot 2025-09-05 160617" src="https://github.com/user-attachments/assets/e158541b-b3d7-49fe-af5a-8c4051c0c940" />

### Lab User role can be modified in user profile

<img width="1918" height="938" alt="Screenshot 2025-09-05 161327" src="https://github.com/user-attachments/assets/4713e13c-097f-4be3-8796-40226fe471ca" />

<img width="1919" height="953" alt="Screenshot 2025-09-05 162400" src="https://github.com/user-attachments/assets/38dc64ab-67f7-4bc0-9f31-c0b9c98216d8" />

<img width="1919" height="871" alt="Screenshot 2025-09-05 163752" src="https://github.com/user-attachments/assets/a3214738-3b02-438e-aeb0-520362a0cb8d" />

<img width="1919" height="883" alt="Screenshot 2025-09-05 163911" src="https://github.com/user-attachments/assets/14db95e6-64a4-4632-8fed-8738181f2131" />

---

### Broken access control resulting from platform misconfiguration

- Some application frameworks support various non-standard HTTP headers that can be used to override the URL in the original request, such as X-Original-URL and X-Rewrite-URL.

- If a website uses rigorous front-end controls to restrict access based on the URL, but the application allows the URL to be overridden via a request header, then it might be possible to bypass the access controls using a request like the following:

  ```http
  POST / HTTP/1.1
  X-Original-URL: /admin/deleteUser
  ...
  ```

### Lab URL-based access control can be circumvented

<img width="1917" height="941" alt="Screenshot 2025-09-05 170354" src="https://github.com/user-attachments/assets/9acef41a-d030-4915-8a3e-245464d8165b" />

<img width="1919" height="947" alt="Screenshot 2025-09-05 170551" src="https://github.com/user-attachments/assets/daba4240-da26-4beb-a116-1c275f5846be" />

<img width="1918" height="838" alt="Screenshot 2025-09-05 170937" src="https://github.com/user-attachments/assets/838184e3-816d-4430-afb4-50d7c5835a15" />

<img width="1916" height="852" alt="Screenshot 2025-09-05 171217" src="https://github.com/user-attachments/assets/b1f08029-112f-455c-b2a8-3dab4ff98ff8" />

<img width="1919" height="844" alt="Screenshot 2025-09-05 171935" src="https://github.com/user-attachments/assets/b7ceda05-040c-4f1a-9575-6bbf3808c08c" />

---

- An alternative attack relates to the HTTP method used in the request.

- If an attacker can use the GET (or another) method to perform actions on a restricted URL, they can bypass the access control that is implemented at the platform layer.

### Lab Method-based access control can be circumvented

- Có thông tin đăng nhập của admin:

  <img width="1919" height="844" alt="Screenshot 2025-09-05 171935" src="https://github.com/user-attachments/assets/bf98e331-618d-4de2-83aa-064295efa275" />

- Lấy request upgrade user vào Burp Repeater:

  <img width="1919" height="944" alt="Screenshot 2025-09-06 172539" src="https://github.com/user-attachments/assets/7da3170d-00ee-4f0c-8a6d-911b56f3fb05" />

- Đăng xuất user admin, sử dụng user non-admin bình thường, và copy session cookie của user wiener để paste vào session cookie trong request upgrade user trong Burp Repeater:

  <img width="1867" height="632" alt="Screenshot 2025-09-06 172737" src="https://github.com/user-attachments/assets/5e1ff927-05ec-49d2-9917-e6454f574329" />

  &rarr; Ý tưởng là muốn sử dụng user non-admin gửi request của user admin để xem phản ứng của server.

- Kết quả _"Unthorized"_:
  
  <img width="1864" height="691" alt="Screenshot 2025-09-06 172955" src="https://github.com/user-attachments/assets/10f92f35-57be-443b-a1a2-ac2610fffb48" />

- Thử đổi phương thức sang POSTabc và xem phản ứng server:

  <img width="1870" height="743" alt="Screenshot 2025-09-06 173508" src="https://github.com/user-attachments/assets/72c5d8e0-af79-41ee-996e-5996ddd5edf2" />

  &rarr; Báo missing param &rarr; bypass FE filtering, tuy nhiên có thể BE mặc định method không hợp lệ sẽ fallback về POST, GET. Do parse phía BE có thể thấy method lỗi    nên không parse body của request hoặc có thể mặc định về method GET nên thiếu query parame username, do đó báo lỗi 400 Missing param.

- Đổi thành method GET, và gửi request:

  <img width="1866" height="741" alt="Screenshot 2025-09-06 174821" src="https://github.com/user-attachments/assets/ae6adb46-ec35-454c-bfa8-096e992ab3aa" />

- Thành công:

  <img width="1919" height="945" alt="Screenshot 2025-09-06 175130" src="https://github.com/user-attachments/assets/63830619-d516-47f1-8b46-d5f00c821559" />

  &rarr; bypass FE filtering và BE vẫn xử lý method GET để thực hiện chức năng của admin.

---

### Broken access control resulting from URL-matching discrepancies

- Các client app có thể khác nhau trong việc cấu hình chặt chẽ đường dẫn của một request với endpoint được định nghĩa/cấu hình sẵn.

- Ví dụ: một request như /ADMIN/DELETEUSER vẫn có thể được ánh xạ thành endpoint /admin/deleteUser. (nếu cơ chế chặt chẽ sẽ không cho phép)

- Tương tự nếu client app mà devs sử dụng framework Spring đã bật tùy chọn _useSuffixPatternMatch_ tức là cái này cho phép các đường dẫn với phần mở rộng tệp tùy ý được ánh xạ đến một endpoint tương đương không có phần mở rộng tệp.

- Nói cách khác, một yêu cầu đến /admin/deleteUser.anything vẫn sẽ khớp với mẫu /admin/deleteUser. Trước Spring 5.3, tùy chọn này được bật theo mặc định.

- Trên một số các hệ thống, ta có thể né tránh các kiểm soát truy cập bằng cách thêm một dấu gạch chéo vào cuối endpoint, ta có thể kiểm tra sự khác biệt giữa: /admim/deleteuser và /admin/deleteuser/

---

## Horizontal privilege escalation

- Lỗi này xảy ra nếu một người dùng có thể truy cập vào tài nguyên của người dùng khác, thay vì tài nguyên của chính họ.

- Ví dụ nếu một nhân viên có thể truy cập vào hồ sơ của những nhân viên khác cũng như của chính họ, thì đó là horizontal privilege escalation.

- Các cuộc tấn công tăng quyền ngang có thể sử dụng các phương pháp khai thác tương tự như tăng quyền dọc.

- Ví dụ một URL của một user truy cập tài khoản của chính họ với:

  ```http
  https://insecure-website.com/myaccount?id=123
  ```

  Nếu họ thực hiện thay đổi giá trị param _id_ thành của một user khác và có thể thu được quyền truy cập vào account user đó.

### Lab User ID controlled by request parameter

- Tương tự các bài lab trước, ta thực hiện đăng nhập với thông tin đăng nhập là: _wiener:peter_

  <img width="1919" height="874" alt="image" src="https://github.com/user-attachments/assets/60494526-0401-44aa-b7aa-6799645fbf4a" />

  &rarr; Ta thấy mỗi user đều có một API key riêng.

- Trong Burp Suite, ta thấy có request để lấy thông tin của user wiener với query string param _id_:

  <img width="1867" height="633" alt="image" src="https://github.com/user-attachments/assets/e087bdc9-7fb1-4507-b451-174367a4568c" />

- Thực hiện chuyển request trên vào trong Burp Repeater và thử đổi id thành user khác, ở đây là: _carlos_

  <img width="1919" height="842" alt="image" src="https://github.com/user-attachments/assets/db12c0d1-f6ae-43be-b33f-33b4b2fa81fa" />

  &rarr; Ta thấy server phản hổi chuyển ngay sang thông tin tài khoản của user carlos &rarr; lỗi.

- Trong một số ứng dụng, tham số có thể khai thác không dự đoán trước được giá trị.

- Ví dụ, thay vì một số tăng dần, một ứng dụng có thể sử dụng các định danh toàn cầu duy nhất (GUID) để xác định người dùng. Điều này có thể ngăn chặn ta đoán hoặc dự đoán định danh của người dùng khác. Tuy nhiên, các GUID thuộc về người dùng khác có thể bị tiết lộ ở nơi nào đó trong ứng dụng chẳng hạn.

### Lab User ID controlled by request parameter, with unpredictable user IDs

- Tương tự, đăng nhập với thông tin đăng nhập: _wiener:peter_

  <img width="1918" height="953" alt="image" src="https://github.com/user-attachments/assets/74cd23ac-daa5-4d3d-9975-ebc196e7db72" />

- Kiểm tra trong Burp, có thể thấy request sử dụng GUID để lấy thông tin truy cập vào tài khoản của user:

  <img width="1919" height="729" alt="image" src="https://github.com/user-attachments/assets/0a35d571-d4b0-47f7-8908-832fc2390066" />

  &rarr; Không thể dự đoán.

- Trên trang chủ của website blog, ta thử kiểm tra các bài viết trên trang, ta thấy ở mỗi bài viết đều sẽ dán link tác giả của bài viết như sau:

  <img width="1919" height="939" alt="image" src="https://github.com/user-attachments/assets/2cc02484-784f-4bfa-9642-395c660bc470" />

  <img width="1919" height="940" alt="image" src="https://github.com/user-attachments/assets/cbe27d84-279d-4e11-aaf8-074aaccc778e" />

- Để ý rằng khi thực hiện click vào link tác giá được gán, nó sẽ chuyển hướng tới đường dẫn mà ở đó để lộ GUID của user tác giá (trùng với GUID trong request trên):

  <img width="1919" height="954" alt="image" src="https://github.com/user-attachments/assets/56d9e86a-79c6-4b8d-9a58-61d81e176ce7" />

- Do đó nếu ta thử, truy cập vào tác giá carlos &rarr; cũng sẽ lộ luôn GUID của tài khoản này:

  <img width="1919" height="950" alt="image" src="https://github.com/user-attachments/assets/cf3962bf-481f-4b1a-9032-9757177d6668" />

- Thực hiện copy và paste vào id của request lấy thông tin user, vâ gửi request đi:

  <img width="1919" height="931" alt="image" src="https://github.com/user-attachments/assets/7ea183ed-8e34-43f4-99d3-bf2dd08406d7" />

  &rarr; Truy cập thành công vào user carlos &rarr; lỗi.

- Trong một số trường hợp, một ứng dụng có thể phát hiện khi người dùng không được phép truy cập tài nguyên và trả về chuyển hướng tới trang đăng nhập. Tuy nhiên, phản hồi chứa chuyển hướng vẫn có thể bao gồm một số dữ liệu nhạy cảm thuộc về người dùng bị nhắm đến, vì vậy cuộc tấn công vẫn thành công.

### Lab User ID controlled by request parameter with data leakage in redirect

- Tương tự, ta để ý request lấy thông tin của website:

  <img width="1919" height="915" alt="image" src="https://github.com/user-attachments/assets/c6ddad1b-7546-4fcb-a436-edac605ccae4" />

- Thử tháy đổi giá trị param sang _carlos_ và gửi request:

  <img width="1919" height="921" alt="image" src="https://github.com/user-attachments/assets/74e8f51e-d40d-4caa-b44b-43552694ef9e" />

  Ta thấy trước khi nó redirect về trang /login của website nó vẫn để lộ trang thông tin user carlos:

  <img width="1919" height="921" alt="image" src="https://github.com/user-attachments/assets/3400c5de-548e-46f9-a098-58e86414c61f" />

  <img width="1919" height="883" alt="image" src="https://github.com/user-attachments/assets/103c8dd2-a916-4883-b3dd-69cc2f83f760" />

  &rarr; Vẫn bị lộ thông tin, dữ liệu nhạy cảm nếu ta dùng proxy như Burp.

---

## Horizontal to vertical privilege escalation

- Ví dụ với endpoint:

  ```http
  https://insecure-website.com/myaccount?id=456
  ```

  Nếu ta ban đầu ta chỉ có thể làm horizontal escalation (chiếm account của user khác) bằng cách thay đổi giá trị param _id_

  Nhưng nếu trúng user có quyền cao (ví dụ admin), thì từ horizontal nó biến thành vertical escalation luôn.

### Lab User ID controlled by request parameter with password disclosure

- Tương tự, đăng nhập với tài khoản: _wiener:peter_, tuy nhiên ta thấy có xuất hiện thêm chức năng update password, với password được hiện thị dưới dạng: *****

  <img width="1919" height="945" alt="image" src="https://github.com/user-attachments/assets/1ada1042-7312-48dc-ad23-4acc19ec095c" />

- Tại request, lấy thông tin user có query param _id_ ta thử điền một user khác là _carlos_ để check lỗi leo quyền chiều ngang:

  <img width="1919" height="891" alt="image" src="https://github.com/user-attachments/assets/abcf48c2-26e1-4791-a475-2ca7cb7d4475" />

  &rarr; Kết quả lấy được page user profile của carlos &rarr; lỗi leo quyền chiều ngang.

- Thử check với tên user là administrator để kiểm tra tài khoản dạng admin:

  <img width="959" height="462" alt="image" src="https://github.com/user-attachments/assets/fb07f1a5-8aa2-45b9-a07a-39e676294804" />

  &rarr; Kết quả thành công vào được page user của admin.

- Từ phản hổi HTML của server trang user administrator ta có thể thu thập được thành công password của admin là: _lnvjd6tjfyd3x5bd2r5l_, sau đó thực hiện đăng nhập:

  <img width="1919" height="854" alt="image" src="https://github.com/user-attachments/assets/1e22828f-c233-4ce7-a0a2-6958b5de9d61" />

  <img width="1919" height="891" alt="image" src="https://github.com/user-attachments/assets/b260d62e-d154-46d1-b09a-80181756bfeb" />

- Truy cập thành công vào chức năng Admin panel:

  <img width="1919" height="885" alt="image" src="https://github.com/user-attachments/assets/968c7f67-db70-4d00-8905-21b7f7762f78" />

---

## Insecure direct object references

- IDOR xảy ra khi một ứng dụng sử dụng đầu vào do người dùng cung cấp để truy cập trực tiếp vào các đối tượng và ta có thể sửa đổi đầu vào để có được quyền truy cập không được phép.

- Lỗ hổng IDOR thường liên quan đến việc gia tăng quyền hạn theo chiều ngang, nhưng chúng cũng có thể xuất hiện liên quan đến việc gia tăng quyền hạn theo chiều dọc.

### IDOR vulnerability with direct reference to database objects

- Một trang web mà cho phép sử dụng URL như bên dưới để truy cập vào trang account khách hàng, bằng cách lấy nhưng thông tin từ BE database:

  ```http
  https://insecure-website.com/customer_account?customer_number=132355https://insecure-website.com/customer_account?customer_number=132355
  ```

  Ở đây customer_number được dùng trực tiếp để query DB (kiểu SELECT * FROM customers WHERE customer_number=132355).

  Nếu backend không có kiểm tra quyền, attacker chỉ cần đổi số này (132355 → 132356) là có thể truy cập data của khách hàng khác &rarr; Đây là IDOR → horizontal            privilege escalation (user này xem được dữ liệu của user khác) hoặc thành leo thẳng lên vertical escalation nếu có được số admin chẳng hạn.

### IDOR vulnerability with direct reference to static files

- App lưu transcript (log chat, hóa đơn, báo cáo, …) trực tiếp thành file với tên được đánh số tăng dần trong server, ví dụ:

  ```
  /static/12144.txt
  /static/12145.txt
  /static/12146.txt
  ```

  &rarr; Nếu không có kiểm tra access control (ownership), thì bất kỳ ai biết hoặc đoán được số ID đều có thể tải file người khác.

### Lab Insecure direct object references

- Ta thấy giao diện có thêm chức năng Live chat:

  <img width="1919" height="889" alt="image" src="https://github.com/user-attachments/assets/6aaeeb45-3344-4ea3-beb4-6e036610bb91" />

- Có vè như chức năng này cho phép lưu trữ nhật ký trò chuyện của người dùng trực tiếp trên hệ thống tệp của máy chủ:

  <img width="1919" height="950" alt="Screenshot 2025-09-07 231525" src="https://github.com/user-attachments/assets/58539438-ae3d-4884-89db-a29a3b9c3c1a" />

- Khi thực hiện chọn _"View transcipt"_ thì nhận thấy các bản transcript được tải về dưới dạng tên file là số và được dánh dưới dạng tăng dần:

  <img width="1919" height="946" alt="Screenshot 2025-09-07 233310" src="https://github.com/user-attachments/assets/658936fa-0f39-43d8-9724-ee4618000fb6" />

  Tuy nhiên thú vị là bản được tải về đầu tiên là 2.txt &rarr; có đoạn 1.txt trước đó &rarr; tìm cách xem nội dung bản 1.txt

- Trong Burp, kiểm tra thấy có request để thực hiện request tải bản transcript về: /download-transcript/2.txt, cùng với đó là server phản hồi lại nội dung của đoạn transcript:

  <img width="1919" height="843" alt="image" src="https://github.com/user-attachments/assets/7f8160d8-c8ab-4ffd-a40b-3458e1c71e47" />

  Thử để param là /download-transcript/1.txt:

  <img width="1909" height="828" alt="image" src="https://github.com/user-attachments/assets/71b96ee0-5d0c-44bd-8fb2-58ab97daff03" />

  &rarr; Lộ bản 1.txt &rarr; Lộ cả thông tin password người dùng là: _133h0q94f5q4iqs057oe_

- Thành công đăng nhập vào user tài khoản carlos với password ăn cắp được:

  <img width="1919" height="875" alt="image" src="https://github.com/user-attachments/assets/b0404eae-032f-4fb1-84f9-d2262ae17150" />

---

## Access control vulnerabilities in multi-step processes

- Một số chức năng quan trọng trong web không chỉ có 1 request duy nhất, mà chia làm nhiều bước liên tiếp như:

  Bước 1: Load form (hiện thông tin user).
  
  Bước 2: Submit thay đổi.
  
  Bước 3: Review & confirm (xác nhận thay đổi).

- Hệ thống thường đặt kiểm tra access control ở bước đầu (bước 1,2). Nhưng lại không kiểm tra ở bước cuối (bước 3), vì dev “nghĩ rằng” user chỉ có thể tới được bước 3 nếu họ đã đi qua 1 & 2 hợp lệ.

  &rarr; Thực tế attacker có thể bỏ qua UI, tự gửi request của bước 3 với parameter phù hợp.

- Ví dụ: Bình thường flow update user:

  ```url
  /loadUser?id=123 (check quyền admin).
  /submitUpdate?name=newName (check quyền admin).
  /confirmUpdate?name=newName&id=123 (không check gì).
  ```

  Nhưng ta có thể trực tiếp gọi luôn bước 3:

  ```http
  POST /confirmUpdate
  id=456&name=hacked
  ```
  
  &rarr; Nếu endpoint này không check quyền → ta đổi thông tin user khác thành công.

### Lab Multi-step process with no access control on one step

- Đầu tiên, ta có thông tin đăng nhập của user administrator là: _administrator:admin_, sau đó truy cập chức năng Admin panel:

  <img width="1919" height="941" alt="image" src="https://github.com/user-attachments/assets/d0aa7f96-b596-45a0-be0e-e298959c8888" />

  Ta thấy khi thực hiện Upgrade một user lên Admin, website sẽ hỏi lại Admin để confirm lại quyết định vừa rồi:

  <img width="1919" height="933" alt="image" src="https://github.com/user-attachments/assets/71a3ac65-702f-451f-92e9-67613d83a2e8" />

- Kiểm tra trong Burp, có thể thấy request thực hiện upgrade/downgrade và request confirm lại với Admin về việc upgrade/downgrade user:

  <img width="1919" height="814" alt="image" src="https://github.com/user-attachments/assets/2cc9cadf-9da9-447a-a41e-c0737691711b" />

  <img width="1919" height="648" alt="image" src="https://github.com/user-attachments/assets/a6791bd7-eebb-4cff-a4d2-c03c7bc2c009" />

  Thực hiện gửi request trên vào trong Burp Repeater, sau đó đăng xuất user Admin.

- Thực hiện đăng nhập với tài khoản _wiener:peter_ là tài khoản non-admin:

  Trong Burp Repeater, ở request thực hiện quyền downgrade như bên dưới, nếu ta thực hiện gửi request với user đang login là wiener (non-admin) (trong đó session cookie    ta phải set lại về thành session cookie của user non-admin wiener đang login):

  <img width="959" height="443" alt="image" src="https://github.com/user-attachments/assets/a67b3696-d767-44bd-8309-0920aa6682a8" />

  &rarr; "Unthorized" &rarr; Server có kiểm tra quyền admin với request này.

  Tiếp tục thực hiện với request confirm lại hành động upgrade user của admin như bên dưới, sau đó gửi request (trong đó session cookie ta phải set lại về thành session    cookie của user non-admin wiener đang login):

  <img width="1919" height="922" alt="image" src="https://github.com/user-attachments/assets/5ff9bc05-3ed0-4c41-8638-84d459065dcc" />

  <img width="1919" height="926" alt="image" src="https://github.com/user-attachments/assets/3938bf65-39c5-4d75-97e2-b3f1c2cea596" />

  &rarr; Thành công thực hiện đổi quyền user wiener lên admin &rarr; Lỗi server không check endpoint confirm của admin.

---

## Referer-based access control

- Referer header cho server biết request này bắt nguồn từ đâu.

- Giả sử /admin chính là trang quản trị chính → ở đây dev kiểm tra access control chuẩn (chỉ admin mới vào được).

- Nhưng với các sub-pages như /admin/deleteUser hoặc /admin/updateConfig, thay vì check quyền người dùng, server chỉ check:

  Nếu Referer là /admin → cho phép.
  
  Nếu không → chặn.

  &rarr; Chỉ dựa vào Referer để tin rằng request đến từ trang admin.

- Tuy nhiên, Referer header nằm trong HTTP request và hoàn toàn có thể bị ta giả mạo một cách thủ công &rarr; Server vẫn cho phép vì thấy Referer “đúng” &rarr; bypass.

---

## Lab Referer-based access control

- Tương tự lab trước, ta có thông tin đăng nhập tài khoản admin là: _administrator:admin_, có thể thấy giao diện chức năng Admin panel như bên dưới:

  <img width="1919" height="868" alt="image" src="https://github.com/user-attachments/assets/7f139afc-7752-49ad-a787-dd38643eea7e" />

- Thực hiện upgrade user carlos:

  <img width="1919" height="868" alt="image" src="https://github.com/user-attachments/assets/28ee9e6a-61ff-4590-8fae-721172b27fbe" />

- Kiểm tra trong Burp, ta có thể thấy với request chức năng Admin panel, Referer header của request sẽ là URL lấy thông tin acc user admin:

  <img width="1918" height="884" alt="image" src="https://github.com/user-attachments/assets/c027883a-3fd0-415f-9885-9b0b8194fae1" />

  Đối với request upgrade user carlos, Referer header sẽ là URL /admin (chức năng Admin panel)

  <img width="1919" height="885" alt="image" src="https://github.com/user-attachments/assets/f332c5fe-c6cc-4212-b46e-127f75bde9f2" />

- Gửi hai request vào trong Burp Repeater.

- Có thể thấy, đối với request /admin, server đã thực hiện chặn nếu user login hiện tại không phải admin:

  <img width="1919" height="882" alt="image" src="https://github.com/user-attachments/assets/8e9689d2-026d-47d1-abf7-44236040948e" />

- Thực hiện sang request upgrade user của admin, ta đổi username thành _wiener_ và session cookie set lại thành session cookie của user wiener hiện tại, sau đó gửi request:

  <img width="1919" height="882" alt="image" src="https://github.com/user-attachments/assets/235313cf-d112-4a54-9e85-73b30c08357a" />

  &rarr; Thành công upgrade user wiener lên admin.

  Tuy nhiên nếu trong request này nếu ta bỏ đuôi /admin hoặc bỏ hẳn trường Referer header ra khỏi request, thì request sẽ bị server chặn "Unthorized" ngay lập tức:

  <img width="1919" height="848" alt="image" src="https://github.com/user-attachments/assets/6b8fe9d4-30ae-4990-866d-4629549a959f" />

  <img width="1919" height="881" alt="image" src="https://github.com/user-attachments/assets/992e6213-efa9-4ae5-869f-d9f0d475d9d3" />

  &rarr; Lỗi Referer-based access control.

--- 

## Location-based access control

- Một số website chỉ cho phép user truy cập nếu đến từ một khu vực địa lý nhất định.

- Ví dụ:

  Ngân hàng: chỉ cho phép giao dịch từ IP nội địa (VN).

  Dịch vụ streaming: chỉ xem được phim nếu ở US, EU, hoặc Nhật (do luật bản quyền).
  
  Trang chính phủ: chặn mọi IP ngoài quốc gia.

- Cơ chế này thường dựa vào:

  _IP-based geolocation (tra vị trí từ IP)._
  
  _Client-side geolocation API (trình duyệt gửi thông tin vị trí GPS)._

- Tuy nhiên, ta có thể bypass bằng cách:

  _**VPN**: route traffic qua server đặt ở vùng được phép._

  _**Proxy**: dùng web proxy ở location hợp lệ._
  
  _**IP spoofing (ít phổ biến hơn)**: fake địa chỉ IP (nhưng thường khó hơn vì TCP handshake)._
  
  _**Manipulate geolocation API**: trình duyệt chỉ lấy location từ client device → attacker có thể giả vị trí bằng extension, script hoặc fake response._

---
  








    

  









  


  

  










