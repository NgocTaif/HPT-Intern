# BÁO CÁO QUÁ TRÌNH KHAI THÁC SQL INJECTION (LOW, MEDIUM, HIGH)

---

## 1. SQL INJECTION (LOW)

Đầu tiên, giao diện xuất hiện một text field cho phép người dùng nhập ID của người dùng:

<img width="1532" height="630" alt="image" src="https://github.com/user-attachments/assets/af3086f2-6e75-4f0a-ba01-fff78b0e1746" />

Kiểm tra source PHP:

<img width="1542" height="369" alt="image" src="https://github.com/user-attachments/assets/24b51fdf-2785-4102-89f3-ee9aeba46f58" />

&rarr; Ta thấy giá trị dữ liệu nhập từ người dùng được nhận thông qua: *$id = $_REQUEST['id'];*

Sau đó, dùng giá trị $id này chèn thẳng vào truy vấn SQL, Sau đó thực thi truy vấn mà không có bất kỳ kiểm tra, lọc hay escape nào.

Tận dụng điều này ta có thể tạo ra các untrusted data để tạo ra các câu truy vấn SQL lỗi.

Tạo ra truy vấn luôn đúng để xem toàn bộ ID, tên user của toàn bộ:

```SQL
' OR '1'='1
```

<img width="1540" height="747" alt="image" src="https://github.com/user-attachments/assets/813358be-44e5-41a1-bbff-4c7164e62a5a" />

Sử dụng Order By để xác định số cột trong câu truy vấn phía trước (mặc dù đã biết là 2 cột trong source PHP :v), nhận thấy order by 2 thì trả về nhưng order by 3 đã bị lỗi trả về → câu truy vấn phía trước có 2 cột:

<img width="1545" height="753" alt="image" src="https://github.com/user-attachments/assets/bc94a93a-0c7a-4f8a-a9b4-a9c60045cb50" />
<img width="1536" height="699" alt="image" src="https://github.com/user-attachments/assets/850110c7-bcd1-4e32-885d-41b9b448ac95" />

Nhập payload sau để xác định table có trong CSDL. Kết quả có hai table là users và guestbook:

```SQL
' UNION SELECT table_name, null FROM information_schema.tables WHERE table_schema=database() -- - 
```

<img width="1534" height="718" alt="image" src="https://github.com/user-attachments/assets/ac824016-b00a-49c8-b246-8c26e26c38d8" />

Nhập payload để lấy tên các cột của table users:

```SQL
' UNION SELECT column_name, null FROM information_schema.columns where table_name = 'users' -- -
```

<img width="1543" height="716" alt="image" src="https://github.com/user-attachments/assets/42fd49cc-d967-45aa-8eb1-4093497ff625" />

Nhập payload để lấy tên tài khoản và mật khẩu trong table users. 

```SQL
 ‘ UNION SELECT user, password FROM users -- -
```

Kết quả thu được:

<img width="1536" height="701" alt="image" src="https://github.com/user-attachments/assets/ffb3a58a-90e3-4de2-af57-4a5a8fa84804" />

---

## 2. SQL INJECTION (MEDIUM)

Kiểm tra source PHP: 

<img width="1905" height="763" alt="image" src="https://github.com/user-attachments/assets/2137573e-58f5-42be-b394-2f38269421b5" />

Ta thấy giá trị dữ liệu nhập từ người dùng được nhận thông qua: *$id = $_POST['id'];*

Sau đó thực hiện escaping bằng *mysqli_real_escape_string*, đây là hàm thực hiện thêm dấu \ vào trước các ký tự đặc biệt như ', ", \, NULL.

Tuy nhiên một điểm đặc biệt là nó không đặt biến *$id* trong dấu nháy '...' → do đó nếu ta nhập các dữ liệu payload bình thường (không cần bypass dấu ‘’) thì sẽ hợp lệ.

Nhập payload dưới đây, sau đó tiến hành gửi:

```SQL
1 OR 1 = 1
```

<img width="1718" height="699" alt="image" src="https://github.com/user-attachments/assets/dba99303-6ae2-46e5-9316-e9254ec94632" />

&rarr; Kết quả thu được toàn bộ thông tin ID, tên của user.

Nhập các payload tương tự phần SQLi low.

Nhập payload sau để lấy tên tài khoản và mật khẩu trong table users. 

```SQL
1 UNION SELECT user, password FROM users
```

Kết quả thu được:

<img width="1727" height="708" alt="image" src="https://github.com/user-attachments/assets/5995ae61-f6f1-4dc1-8044-294c3fcb5b3c" />

---

## 3. SQL INJECTION (HIGH)

Giao diện xuất hiện một *link*, khi click sẽ hiện một form khác cho phép user nhập session id và gửi:

<img width="1714" height="724" alt="image" src="https://github.com/user-attachments/assets/918d94e7-8b6c-4ce3-9be3-8fe3c75446b8" />

Kiểm tra source PHP:

<img width="1718" height="654" alt="image" src="https://github.com/user-attachments/assets/73a973ef-a0fe-4ab2-bb28-dede379a3335" />

&rarr; Ta thấy ở mức lab này có xuất hiện một giao diện form nhỏ để nhập Session ID, và form session-input.php sẽ gửi id thông qua phương thức POST.

<img width="1721" height="726" alt="image" src="https://github.com/user-attachments/assets/ee9e38d3-17d5-4146-8100-9b7413d04132" />

Do đó mà về cơ bản lỗ hổng SQLi vẫn xảy ra tương tự như trong bài lab ở mức low khi ta nhập payload (untrusted data) ở trong form session-input.php:

<img width="1723" height="748" alt="image" src="https://github.com/user-attachments/assets/a36c20af-030d-4e55-bb83-cecb3e927296" />

<img width="1724" height="733" alt="image" src="https://github.com/user-attachments/assets/1fd561ca-3ea6-4dbf-9da3-6cd03a3977df" />

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng SQLi trên lab DVWA.

---

