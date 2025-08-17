# BÁO CÁO QUÁ TRÌNH THỰC HIỆN RECONNAISSANCE site *paylution.com trên H1

---

Mục tiêu (target) thực hiện quá trình Reconnaissance là site *paylution.com* trên platform Hackerone:

<img width="1892" height="943" alt="image" src="https://github.com/user-attachments/assets/1daa9eb2-9c18-495c-be5c-642855b7cb93" />

<img width="1914" height="889" alt="image" src="https://github.com/user-attachments/assets/2867fbbb-cbd7-4a20-bfc3-b727332d8178" />

## 1. Thu thập, xác định tên miền, IP

Sử dụng công cụ Nslookup để tra cứu thông tin DNS của site *paylution.com*:

<img width="924" height="253" alt="image" src="https://github.com/user-attachments/assets/413d074d-99c0-4828-9143-72741b99e3df" />

Tra cứu bản ghi MX (Mail Server) và NS (Name Server):

<img width="965" height="372" alt="image" src="https://github.com/user-attachments/assets/4309f59e-8022-4d25-9a6a-cc0ee7cce9f0" />

<img width="847" height="259" alt="image" src="https://github.com/user-attachments/assets/d8a2cfac-35fe-4680-ba4a-0624f73d209e" />

Chi tiết hơn ta sử dụng một công cụ khác là *whois* (một công cụ dòng lệnh dùng để truy vấn thông tin đăng ký tên miền từ cơ sở dữ liệu WHOIS – thường chứa: Tên chủ sở hữu tên miền (Registrant), Tổ chức đăng ký (Registrar), Ngày tạo và hết hạn tên miền, Thông tin liên hệ kỹ thuật/quản trị, DNS đang sử dụng (Name Servers).

Kết quả từ *whois*:

<img width="1677" height="594" alt="image" src="https://github.com/user-attachments/assets/bd18cf52-2fd0-4e6a-916c-069c8e9b903e" />

<img width="1671" height="701" alt="image" src="https://github.com/user-attachments/assets/be030ab3-95c4-4d12-89fd-0bb87d3214e5" />








