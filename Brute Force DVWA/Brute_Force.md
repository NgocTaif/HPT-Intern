# BÁO CÁO QUÁ TRÌNH KHAI THÁC BRUTE FORCE (LOW, MEDIUM, HIGH)

---

## 1. BRUTE FORCE (LOW)

Đầu tiên, giao diện là một trang login cho phép người dùng nhập username và password:

<img width="1538" height="638" alt="image" src="https://github.com/user-attachments/assets/942c6bc3-5031-4f98-95a5-03b6b4d38fe6" />

Kiểm tra source PHP:

<img width="1540" height="633" alt="image" src="https://github.com/user-attachments/assets/f20c7a8c-edc0-495f-8048-b15a5e3ebeeb" />

&rarr; Có thể thấy trong đoạn code trên, ta thấy không có bất kỳ biện pháp bảo vệ nào được đưa ra, qua đó cho phép bất kỳ ai cũng có thể thực hiện thử login bao lần tùy thích, và login vào bất kỳ user nào mà không có sự ảnh hưởng nào.

Username và password người dùng nhập được gửi đi bằng phương thức POST tới server:

<img width="1544" height="691" alt="image" src="https://github.com/user-attachments/assets/bdd6cc61-5a5a-44ba-8232-7b7ee0554b85" />

Thực hiện brute force password của tài khoản admin bằng công cụ Intruder ở chế độ Sniper:

<img width="1629" height="799" alt="image" src="https://github.com/user-attachments/assets/70405c1a-0e62-458f-a201-449bd8b165db" />

Thực hiện nhập payloads là danh sách top 100 password thông dụng, có thể lấy từ các nguồn trên internet như Github:

<img width="1434" height="788" alt="image" src="https://github.com/user-attachments/assets/7d00ecbb-e23b-4532-99b3-02bac83bbc70" />

Thực hiện tấn công brute force, nhận thấy với payload password là password, thì Length trả về có độ dài bất thường (5073) hơn so với các paylaod password khác là 5030:

<img width="1438" height="501" alt="image" src="https://github.com/user-attachments/assets/41794292-df6b-42a5-986b-a8e194ecfba3" />

Kết quả thực hiện brute force password thành công:

<img width="1445" height="556" alt="image" src="https://github.com/user-attachments/assets/3812f8ad-e13e-4f17-b9a4-df675c2f9b13" />

---

## 2. BRUTE FORCE (MEDIUM)

Kiểm tra source PHP, ta thấy thêm một hàm sleep(2) để khiến mỗi lần đăng nhập sẽ delay 2s, khiến cho quá trình brute force trở lên lâu hơn:: 

<img width="1441" height="630" alt="image" src="https://github.com/user-attachments/assets/32c64161-ab98-4893-8497-ab48229c54c7" />

---

## 3. BRUTE FORCE (HIGH)

Kiểm tra source PHP:

<img width="1449" height="549" alt="image" src="https://github.com/user-attachments/assets/c92cf643-dfab-46ed-b8d2-dacc9a69aa83" />

&rarr; Ta có thể thấy được rằng, nó sẽ kiểm tra xem token của form gửi lên có trùng khớp với token lưu trong session, do đó ta buộc phải trích xuất được token mỗi lần trước khi gửi request, giúp ngăn chặn các cuộc tấn công brute force tự động.

<img width="1451" height="633" alt="image" src="https://github.com/user-attachments/assets/7487c89a-3519-4406-b5fe-4bf40bf60c2b" />

Ý tưởng sử dụng công cụ Intruder, thực hiện trích xuất giá trị user_token động được ẩn từ phản hồi (response) và tự động chèn giá trị đó vào các request tiếp theo. Ta có thể thực hiện bằng tính năng “Grep-extract” của công cụ Intruder.
Do đó mà về cơ bản lỗ hổng SQLi vẫn xảy ra tương tự như trong bài lab ở mức low khi ta nhập payload (untrusted data) ở trong form session-input.php:

Sử dụng chế độ Pitchfork (thực hiện brute force theo dạng song song giữa các payload), và add hai payload để brute force là password và user_token:

<img width="1447" height="634" alt="image" src="https://github.com/user-attachments/assets/443ada1d-1fa5-456e-a1d2-f2f823a68398" />

Cấu hình cho payload password:

<img width="927" height="816" alt="image" src="https://github.com/user-attachments/assets/c065ca77-acab-4fc2-a766-e27b6e007bc7" />

Đổi với payload user_token ta để payload type là *Recursive grep*.

Trong phần setttings, mục Grep-extract ta thực hiện add grep extract item như sau:

<img width="982" height="785" alt="image" src="https://github.com/user-attachments/assets/d7725ce9-aa36-49e2-9fda-c911bdd589f0" />

Mục Redirections để là Always.

Kết quả brute force thành công:

<img width="1446" height="587" alt="image" src="https://github.com/user-attachments/assets/a5f4b4dc-01a6-48d3-b221-b3d58f04c0c4" />

---

## ✅ Kết luận:

Thành công khai thác lỗ hổng Brute Force trên lab DVWA.

---

