# BÁO CÁO QUÁ TRÌNH KHAI THÁC FILE INCLUSION (LOW, MEDIUM, HIGH)

---

## 1. FILE INCLUSION (LOW)

Đầu tiên, giao diện là một trang với ba đường dẫn file: *file1, file2, file3*. cho phép người dẫn click để truy cập nội dung của file đó:

<img width="1440" height="606" alt="image" src="https://github.com/user-attachments/assets/b5e05f15-f9b4-4ee2-aa40-4d4f3f257b0d" />

Khi click một trong các link file, trang sẽ direct để truy cập nội dung của file đó. Ta có thể thấy ở đây, trang web thực hiện truyền vào tên file và nhận giá trị thông qua *page=...* Do đó mà có thể tồn tại lỗ hổng File inclusion cho phép chèn và thực thi nội dung của file (mã PHP) vào trong quá trình xử lý của server.

<img width="1441" height="614" alt="image" src="https://github.com/user-attachments/assets/49fe71ec-d2c1-4c00-a453-141d6928a0f0" />

Kiểm tra source PHP:

<img width="1442" height="535" alt="image" src="https://github.com/user-attachments/assets/696a53f4-a50e-4bbc-8dee-c21b1d67543a" />

&rarr; Ta có thể thấy, tên file truyền vào được lấy giá trị từ URL (tham số page) và gán vào biến $file. Do không có kiểm tra đầu vào gì cả, nên có thể tận dụng để khai thác lỗ hổng File inclusion dạng LFI hoặc RFI.

**Khai thác LFI:**

Thử chỉnh sửa giá trị tên file truyền vào của tham số page là *file4*.

<img width="1433" height="526" alt="image" src="https://github.com/user-attachments/assets/2af8a1d9-1140-4680-ba35-17e9c48ca694" />

&rarr; Có thể thấy server đã trả về nội dung file không được phép truy cập.

Tạo các payload sử dụng directory traversal ../ để thực hiện đọc các file hệ thống khác như: */etc/passwd, /etc/hosts*

<img width="1446" height="529" alt="image" src="https://github.com/user-attachments/assets/9b9e475f-90db-4c9b-91a9-328874264e36" />

<img width="1433" height="527" alt="image" src="https://github.com/user-attachments/assets/cd225153-2de6-48a7-a7e5-582847ae3b60" />

**Khai thác RFI:**

Tạo một file *test.php* với nội dung:

<img width="1435" height="588" alt="image" src="https://github.com/user-attachments/assets/2bc3fbd1-09c2-4169-97f7-ab24061fb3fc" />

Thực hiện tạo một payload để tải file vừa tạo từ server HTTP cổng 1337 từ máy mình:

<img width="1438" height="578" alt="image" src="https://github.com/user-attachments/assets/15a16814-ac74-4b58-8d3d-205c1a5e94f7" />

Server từ máy đã nhận request:

<img width="1444" height="386" alt="image" src="https://github.com/user-attachments/assets/724019c5-95b9-4759-8a01-04aab9197af1" />

---

## 2. FILE INCLUSION (MEDIUM)

Kiểm tra source PHP:

<img width="1440" height="527" alt="image" src="https://github.com/user-attachments/assets/a890d097-bc80-48c4-bffb-a10cdbd1fe4e" />

&rarr; Có thể thấy, tên file truyền vào vẫn được lấy giá trị từ URL (tham số page) và gán vào biến $file. Tuy nhiên, nó đã thực hiện lọc đầu vào, gỡ bỏ các chuỗi "http://" và "https://" → nhằm ngăn Remote File Inclusion (RFI) và Gỡ bỏ "../" và "..\\" → nhằm ngăn Directory Traversal, là kỹ thuật thường dùng trong Local File Inclusion (LFI).

Tạo payload bypass filter “../” bằng cách tạo ra: ….// bởi khi filer thực hiện lọc bỏ được một lần:

<img width="1430" height="467" alt="image" src="https://github.com/user-attachments/assets/79f19491-1694-4a76-85d5-3fc1c90e2718" />

<img width="1433" height="469" alt="image" src="https://github.com/user-attachments/assets/322f19d6-a5c8-4723-8826-84b70c785ec2" />

Tạo payload bypass filter “http://” bằng cách tạo ra các đường dẫn khác như: “Http://…” hoặc “HTTP://…” bởi khi filer sử dụng str_replace(), và đây là hàm phân biệt giữa uppercase và lowercase nên ta có thể tận dụng để bypass. Hoặc sử dụng các contents khác như: file://, ftp://

<img width="1434" height="526" alt="image" src="https://github.com/user-attachments/assets/742bbf85-45c7-4371-8c09-f863a84af32a" />

<img width="1437" height="499" alt="image" src="https://github.com/user-attachments/assets/1f61a5ce-51d4-4240-9b75-ee22434599c3" />

<img width="1444" height="486" alt="image" src="https://github.com/user-attachments/assets/e412590a-41c9-4233-a0b5-901e55d3a37f" />

<img width="1439" height="315" alt="image" src="https://github.com/user-attachments/assets/792149f9-fbcb-428b-8c24-f05213decd20" />

---

## 3. FILE INCLUSION (HIGH)

Kiểm tra source PHP:

<img width="1431" height="599" alt="image" src="https://github.com/user-attachments/assets/18641a18-057c-4c45-905e-468ebdd2a6c7" />

&rarr; Ta có thể thấy được rằng, nó sử dụng cơ chế lọc *fnmatch*, chỉ cho phép tên file bắt đầu bằng "file" và file được gọi là phải là *include.php*.

Ta có thể lợi dụng content file:// để thực hiện tạo payload bypass filter này nó vẫn match vì tên bắt đầu là “file”:

<img width="1440" height="486" alt="image" src="https://github.com/user-attachments/assets/22080cbf-3b33-4d87-afcd-4aa5a85ceb26" />

<img width="1454" height="544" alt="image" src="https://github.com/user-attachments/assets/6fa34a92-f8a2-463c-a1ec-0c5a60441670" />

<img width="1442" height="488" alt="image" src="https://github.com/user-attachments/assets/f7af5ead-4828-4f0e-8487-48e063fbda98" />

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng File Inlusion trên lab DVWA.

---

