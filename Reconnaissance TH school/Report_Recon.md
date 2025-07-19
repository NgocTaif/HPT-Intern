
# BÁO CÁO QUÁ TRÌNH THỰC HIỆN RECONNAISSANCE TRANG WEB TH SCHOOL

---

Trinh sát (hay còn gọi là Trinh sát) là một quá trình thiết yếu trong pentesting, đặc biệt là Pentesting hộp đen, nơi ta không có thông tin cụ thể về mục tiêu của mình. Trước khi bắt đầu khai thác mục tiêu, điều quan trọng là phải thu thập càng nhiều thông tin càng tốt về mục tiêu của bạn để xác định khu vực bề mặt, phạm vi tấn công. "Bề mặt tấn công" là một thuật ngữ ưa thích được sử dụng để xác định "Điểm tấn công khả thi" cho mục tiêu của ta.

Mục tiêu hiện này là trang web TH School (link: *https://thschool.edu.vn*), giao diện trang chủ chính thức của trang web:

<img width="1919" height="804" alt="image" src="https://github.com/user-attachments/assets/1ec89d8b-8b8c-4cd5-b87e-dc3454710305" />

Một số workflow quan trọng ta sẽ thực hiện trong phạm vi trang web này:

- Thu thập, xác định tên miền, IP
- Thu thập subdomain
- Quét cổng, dịch vụ
- Technology Fingerprinting
- Thu thập thông tin từ HTTP
- Thu thập, liệt kê đường dẫn, thu mục, tệp tin ẩn

## 1. Thu thập, xác định tên miền, IP

Sử dụng công cụ ***Nslookup*** để tra cứu thông tin DNS của tên miền thschool.edu.vn

Thực hiện tra cứu IP cảu tên miền thschool.edu.vn:

```bash
nslookup thschool.edu.vn
```

<img width="1207" height="251" alt="image" src="https://github.com/user-attachments/assets/5382f03d-64b6-41f9-bdb9-ccf2d8767c4f" />

&rarr;  101.99.3.34	Đây là địa chỉ IPv4 thực tế (A record) mà tên miền thschool.edu.vn trỏ tới. Đây là IP máy chủ chứa trang web.

Tra cứu bản ghi MX (Mail Server):

```bash
nslookup -type=mx thschool.edu.vn
```

<img width="1143" height="290" alt="image" src="https://github.com/user-attachments/assets/16b8a5fc-bafc-4a84-975b-4aa4cf8230b1" />

&rarr; Danh sách các mail server xử lý email cho tên miền.

Tra cứu bản ghi NS (Name Server):

```bash
nslookup -type=ns thschool.edu.vn
```

<img width="1293" height="454" alt="image" src="https://github.com/user-attachments/assets/56820709-6256-48ef-ad9b-2748f7fa1fd4" />

&rarr; Danh sách các máy chủ DNS chính thức (NS record) của tên miền thschool.edu.vn.

Sử dụng công cụ ***dig*** để kiểm tra lại (tương tự như Nslookup nhưng mạnh và chi tiết hơn):

<img width="1230" height="562" alt="image" src="https://github.com/user-attachments/assets/48a6187c-09c1-46ca-aed7-3d9f13e5ef27" />

&rarr; Kết quả cho thấy địa chỉ IP (A record) của tên miền giống như Nslookup.

Sử dụng trang ***DomainTools*** (*https://whois.domaintools.com*) và *https://vnnic.vn* để tra cứu Whois của tên miền:

<img width="1919" height="813" alt="image" src="https://github.com/user-attachments/assets/3325a6c9-1e1e-4fc9-803e-6eade7c95a2e" />

<img width="1917" height="802" alt="Screenshot 2025-07-19 011559" src="https://github.com/user-attachments/assets/5391ff7e-cb34-494b-babf-f3db4c814924" />

## 2. Thu thập subdomain

Thực hiện tìm, liệt kê các subdomain (tên miền phụ) là phần mở rộng của một tên miền chính: *thschool.edu.vn*

Sử dụng công cụ trang Web ***dnsdumpster*** (một công cụ OSINT dùng để thu thập thông tin về DNS và cơ sở hạ tầng của một tên miền) tìm các subdomains:

<img width="1919" height="654" alt="image" src="https://github.com/user-attachments/assets/b9620cef-58a1-4f4b-bbdf-81382434d1dd" />

Kết quả thu được:

<img width="1917" height="597" alt="image" src="https://github.com/user-attachments/assets/aca741fa-d9d9-4eca-ac30-e449c6e2d00f" />

<img width="1828" height="616" alt="image" src="https://github.com/user-attachments/assets/7b2eae62-adf4-47ce-96d1-4534b8b5e6a4" />

<img width="1695" height="650" alt="image" src="https://github.com/user-attachments/assets/4f161370-7a0b-4663-9177-3ad88649745b" />

Tương tự ta cũng thể sử dụng ***Amass*** để dò quét các subdomains (tuy nhiên quá trình quét diễn ra hơi lâu):

```bash
amass enum -passive/-brute -d thschool.edu.vn
```

<img width="1913" height="868" alt="image" src="https://github.com/user-attachments/assets/609f1b02-53e8-4ed8-8354-53da7cdc4474" />

## 2. Quét cổng, dịch vụ

Sử dụng ***nmap*** (một công cụ port scanner thông dụng và phổ biến) để thực hiện:

Quét các cổng đang mở trên tên miền thschool.edu.vn:

```bash
nmap thshool.edu.vn
```

<img width="1222" height="315" alt="image" src="https://github.com/user-attachments/assets/66674c0a-1300-4776-9f6b-b78788d11b26" />

&rarr; Kết quả cho thấy chỉ có hai cổng đang mở public là 80 (http) và 443 (https)

Quét dịch vụ trên các cổng đang mở:

```bash
nmap -sV thschool.edu.vn
```

<img width="1307" height="539" alt="image" src="https://github.com/user-attachments/assets/6e902766-c6ec-43b0-b309-921d4ab5f4bf" />

&rarr; Kết quả cho thấy trên cổng 80 là phiên bản: Apache/2 (máy chủ web đang chạy Apache), còn cổng 443 có SSL nhưng Nmap không xác định rõ phiên bản dịch vụ.

Quét xác định hệ diều hành của thschool.edu.vn:

```bash
nmap -O thachool.edu.vn
```

<img width="1306" height="506" alt="image" src="https://github.com/user-attachments/assets/3f4584f4-e705-489f-8aae-407f603c82a7" />

&rarr; Kết quả không rõ ràng, có thể là Linux/Crestron 2-Series/ HP embedded.

Sử dụng một công cụ khác là ***naabu*** (một fast port scanner do nhóm ProjectDiscovery phát triển, dùng để quét các cổng mở trên một hoặc nhiều host):

<img width="1286" height="390" alt="image" src="https://github.com/user-attachments/assets/46fb1173-7e36-4e85-9b48-304ed871526b" />

&rarr; Kết quả tương tự.

## 3. Techlonogy Fingerprinting

Đây là bước còn gọi là Web Technology Detection là quá trình xác định các công nghệ được sử dụng trên một website.

Ta sẽ sử dụng hai công cụ xác định fingerprint được sử dụng trên website: Whatweb, Wappalyzer Plugin.

```bash
whatweb https://thschool.edu.vn
```

<img width="1588" height="207" alt="image" src="https://github.com/user-attachments/assets/75d32bb4-80cb-4ceb-8b10-f9248120809c" />

Quét một số subdomains khác của thschool.edu.vn:

```bash
whatweb https://hoalac.thschool.edu.vn
```

<img width="1676" height="192" alt="Screenshot 2025-07-19 174617" src="https://github.com/user-attachments/assets/16d6e0a1-243e-4ae6-a991-afe248109de2" />

```bash
whatweb https://chuaboc.thschool.edu.vn
```

<img width="1669" height="191" alt="image" src="https://github.com/user-attachments/assets/8517749d-2d5b-4566-abba-b7fca40fe3d3" />

```bash
whatweb https://library.thschool.edu.vn
```

<img width="1675" height="183" alt="image" src="https://github.com/user-attachments/assets/98ed87aa-bf94-4d95-a6cb-b8e7c82fcbe6" />

```bash
whatweb https://photos.thschool.edu.vn
```

<img width="1668" height="176" alt="image" src="https://github.com/user-attachments/assets/cf881439-9c7c-43e1-9aba-1a8fa6450aca" />

```bash
whatweb https://admin.photos.thschool.edu.vn
```

<img width="1675" height="366" alt="image" src="https://github.com/user-attachments/assets/5e7ccecf-3eed-4145-85a9-34611f53f1bf" />

```bash
whatweb https://vinh.thschool.edu.vn
```

<img width="1672" height="199" alt="image" src="https://github.com/user-attachments/assets/d1459793-9f91-4f4a-8d68-f0559948d501" />


---

## ✅ Kết luận:

Thành công khai thác lỗ hổng Blind SQLi trên lab DVWA.

---
