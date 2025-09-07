# Lỗ hổng Access Control and Privilege escalation

---

### Lab Unprotected admin functionality

<img width="1919" height="882" alt="Screenshot 2025-09-05 145659" src="https://github.com/user-attachments/assets/2b1d2d11-05ac-49a2-99a8-5ba841125150" />

<img width="1919" height="927" alt="Screenshot 2025-09-05 145812" src="https://github.com/user-attachments/assets/2bc0fa4c-029b-4705-ba50-70354ca7069d" />

<img width="1919" height="942" alt="Screenshot 2025-09-05 150049" src="https://github.com/user-attachments/assets/42d46410-02e3-4ce4-90e7-bce3df5a4ba0" />

<img width="1919" height="881" alt="Screenshot 2025-09-05 150141" src="https://github.com/user-attachments/assets/b812dd8c-b506-4992-8357-96910475aad2" />

### Lab Unprotected admin functionality with unpredictable URL

<img width="1916" height="946" alt="Screenshot 2025-09-05 151013" src="https://github.com/user-attachments/assets/bd8c423d-e04e-4060-b781-d1f8df2719e0" />

<img width="1919" height="947" alt="Screenshot 2025-09-05 151053" src="https://github.com/user-attachments/assets/ba2b909b-8686-45f5-82aa-c72a928372f5" />

<img width="1919" height="939" alt="Screenshot 2025-09-05 151328" src="https://github.com/user-attachments/assets/5bb5cfad-cfe7-4645-a1d1-2f2627bb7819" />

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

### Lab URL-based access control can be circumvented

<img width="1917" height="941" alt="Screenshot 2025-09-05 170354" src="https://github.com/user-attachments/assets/9acef41a-d030-4915-8a3e-245464d8165b" />

<img width="1919" height="947" alt="Screenshot 2025-09-05 170551" src="https://github.com/user-attachments/assets/daba4240-da26-4beb-a116-1c275f5846be" />

<img width="1918" height="838" alt="Screenshot 2025-09-05 170937" src="https://github.com/user-attachments/assets/838184e3-816d-4430-afb4-50d7c5835a15" />

<img width="1916" height="852" alt="Screenshot 2025-09-05 171217" src="https://github.com/user-attachments/assets/b1f08029-112f-455c-b2a8-3dab4ff98ff8" />

<img width="1919" height="844" alt="Screenshot 2025-09-05 171935" src="https://github.com/user-attachments/assets/b7ceda05-040c-4f1a-9575-6bbf3808c08c" />

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










