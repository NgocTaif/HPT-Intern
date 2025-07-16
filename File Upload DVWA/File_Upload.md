# BÁO CÁO QUÁ TRÌNH KHAI THÁC FILE UPLOAD (LOW, MEDIUM, HIGH)

---

## 1. FILE UPLOAD (LOW)

Đầu tiên, giao diện là một trang cho phép người dùng thực hiện upload một tệp ảnh lên trang web:

<img width="1536" height="642" alt="image" src="https://github.com/user-attachments/assets/290815b4-c5df-4b30-9b29-b48d6a2e9c35" />

Thực hiện upload một file ảnh hợp lệ lên trình duyệt:

<img width="1619" height="586" alt="image" src="https://github.com/user-attachments/assets/184d158a-8db6-4d2a-8962-c09885e0a109" />

Kiểm tra source PHP:

<img width="1619" height="673" alt="image" src="https://github.com/user-attachments/assets/99a8a499-2925-4440-a9b9-235d2d6074b6" />

Ta có thể thấy nó không có bất kỳ kiểm tra định dạng, loại tệp hay nội dung mà người dùng thực hiện upload lên. Người dùng có thể upload bất kỳ loại file nào → có thể thực hiện upload các file php có nội dung độc hại.

Trong công cụ Repeater thực hiện gửi lại request upload một file, trong đó ta đổi tên file upload lên là *image.php*, và có thể thấy việc upload vẫn diễn ra thành công:

<img width="1620" height="651" alt="image" src="https://github.com/user-attachments/assets/772c1f5d-fe47-4754-9900-9f6ac36eb905" />

Tạo các file img.php với nội dung: `*<?php system(‘ls /’);*` hoặc `*<?php system(‘whoami’);*` sau đó thực hiện upload như bình thường:

<img width="1615" height="630" alt="image" src="https://github.com/user-attachments/assets/a0c378aa-a363-4df0-9e4d-c3aae461e7ca" />

<img width="1622" height="635" alt="image" src="https://github.com/user-attachments/assets/5acbe238-73f0-4106-9f33-61f52103e7ac" />

<img width="1618" height="493" alt="image" src="https://github.com/user-attachments/assets/3bf61b09-f631-4e89-b2b7-d30787273bcb" />

Thực hiện kiểm tra lại đường dẫn lưu tệp tin đã upload để kiểm tra kết quá: *../../hackable/uploads/img.php*

<img width="1622" height="664" alt="image" src="https://github.com/user-attachments/assets/c0198af0-2dd7-4501-a347-2ae99c4d9d1e" />

<img width="1624" height="663" alt="image" src="https://github.com/user-attachments/assets/d01a27e7-be2d-4948-93ef-866dbad06d1b" />

---

## 2.FILE UPLOAD (MEDIUM)

Kiểm tra source PHP:

<img width="1619" height="753" alt="image" src="https://github.com/user-attachments/assets/02a124e5-fb2e-4019-806c-0cf42149a9dc" />

&rarr; Có thể thấy nó thực hiện kiểm tra loại MIME type (kiểu nội dung) dựa trên *$_FILES['uploaded']['type']*, cùng với đó nếu là JPEG/PNG hợp lệ và nhỏ hơn 100KB thì cho phép upload. Tuy nhiên, nó không kiểm tra nội dung thật của file và phần mở rộng tên file.

Thực hiện upload một file ảnh hợp lệ:

<img width="1623" height="660" alt="image" src="https://github.com/user-attachments/assets/a298b5df-384e-4293-bcd2-ccd3c9ee8701" />

Thực hiện upload file không hợp lệ:

<img width="1624" height="480" alt="image" src="https://github.com/user-attachments/assets/f02e82f8-5c79-4355-854f-e7f76c877eb6" />

Thực hiện sử dụng Repeater một request hợp lệ, sau đó đổi phần đuôi file thành .php và Content-type vẫn image/png, và nội dung là `*<?php system(‘cat /etc/passwd’);*` (tương tự trong phần LOW):

<img width="1627" height="615" alt="image" src="https://github.com/user-attachments/assets/95e19de2-e610-4a55-ba6a-1362cfa4b097" />

Kiểm tra lại đường dẫn lưu trữ:
  
<img width="1622" height="573" alt="image" src="https://github.com/user-attachments/assets/a8d1aea5-7c1b-43ec-aa3e-de16f5b4c0a4" />

---

## 3. FILE UPLOAD (HIGH)

Kiểm tra source PHP:

<img width="1629" height="686" alt="image" src="https://github.com/user-attachments/assets/bd29faf6-a71c-47f7-b740-07b897c4dbbb" />

&rarr; Ta có thể thấy được rằng, nó thực hiện trích phần mở rộng file và chỉ chấp nhận các phần mở rộng như: .jpg, .png, .jpeg. Cùng với đó sử dụng hàm *getimagesize()* để xác minh nội dung thật của file, hàm sẽ sẽ trả về false nếu file không phải là ảnh thật.

Thực hiện upload một file ảnh hợp lệ:
  
<img width="1632" height="486" alt="image" src="https://github.com/user-attachments/assets/c7713a73-430f-4898-b393-9e90aac5acf0" />

Thực hiện upload không thành công đối với các file có đuôi không hợp lệ và có nội dung không phải ảnh thật:

<img width="1616" height="528" alt="image" src="https://github.com/user-attachments/assets/ef7c500c-e127-4962-b89d-2229115d6799" />

Trong công cụ Repeater ta tiếp tục thực hiện tạo một request hợp lệ của một file “image.png” và nội dung là file ảnh hợp lệ, tuy nhiên ta chèn thêm nội dung mã php ở cuối như sau:

<img width="1620" height="580" alt="image" src="https://github.com/user-attachments/assets/c139988e-2f57-4175-a52f-3900159944ad" />

Kết quả vẫn sẽ upload thành công vì hàm *getimagesize()* đọc phần đầu của file để xác định xem nó có header hợp lệ của một ảnh không. Tuy nhiên do file là định dạng png không phải PHP, do đó mà mã PHP được chèn sẽ không thể thực thi, ta có thể điều đó khi truy cập đường dần lưu file và thấy chỉ hiện thị hình ảnh:

<img width="1625" height="419" alt="image" src="https://github.com/user-attachments/assets/d3c3ddde-439a-4fef-b9f1-a281c7e83167" />

Lúc này ta có thể lợi dụng lỗ hổng File Inclusion của trang web. File Inclusion là lỗ hổng xảy ra khi một ứng dụng PHP cho phép người dùng chỉ định tên file để include (nhúng nội dung file đó vào mã nguồn và thực thi), nhưng không kiểm tra đầu vào một cách an toàn, ở đây là `*page=*`

Nếu ta truy cập như sau: 

```bash
http://127.0.0.1/dvwa/vulnerabilities/fi/?page=../../hackable/uploads/image.png
```

Lúc này PHP sẽ thực hiện: 

```bash
include("../../hackable/uploads/image.png");
```

Khi include một file, PHP thực thi toàn bộ đoạn mã PHP bên trong file, kể cả nó không có đuôi .php.

Kết quả thu được:

<img width="1625" height="667" alt="image" src="https://github.com/user-attachments/assets/9a3bdd9a-09e1-4c0f-9ca5-6b79181a696a" />

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng File Upload trên lab DVWA.

---

