# BÁO CÁO QUÁ TRÌNH THỰC HIỆN RECONNAISSANCE TRANG WEB TH SCHOOL

---

Trinh sát (hay còn gọi là Trinh sát) là một quá trình thiết yếu trong pentesting, đặc biệt là Pentesting hộp đen, nơi ta không có thông tin cụ thể về mục tiêu của mình. Trước khi bắt đầu khai thác mục tiêu, điều quan trọng là phải thu thập càng nhiều thông tin càng tốt về mục tiêu của bạn để xác định khu vực bề mặt, phạm vi tấn công. "Bề mặt tấn công" là một thuật ngữ ưa thích được sử dụng để xác định "Điểm tấn công khả thi" cho mục tiêu của ta.

Mục tiêu (target) là trang web TH School (link: *https://thschool.edu.vn*), giao diện trang chủ chính thức của trang web:

<img width="1919" height="804" alt="image" src="https://github.com/user-attachments/assets/1ec89d8b-8b8c-4cd5-b87e-dc3454710305" />

Một số workflow quan trọng ta sẽ thực hiện trong phạm vi trang web này:

- Thu thập, xác định tên miền, IP
- Thu thập subdomain
- Quét cổng, dịch vụ
- Technology Fingerprinting
- Crawling (URL Enumeration)
- Thu thập, liệt kê đường dẫn, thu mục, tệp tin ẩn
- Parameter Discovery
- Google Dorking

---

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

---

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

---

## 3. Quét cổng, dịch vụ

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

---

## 4. Techlonogy Fingerprinting

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

Sử dụng extension ***Wappalyzier*** để kiểm tra trên website: *https://thschool.edu.vn*

<img width="1885" height="940" alt="image" src="https://github.com/user-attachments/assets/dc8ceb74-48ca-492f-923b-1430f4def94f" />

Một số kết quả của các subdomains của thschool.edu.vn:

<img width="1919" height="957" alt="image" src="https://github.com/user-attachments/assets/8d5e8917-e053-4931-882f-267b7d2e660f" />

<img width="1918" height="946" alt="image" src="https://github.com/user-attachments/assets/addd1e80-8bdc-4d28-99cd-7c846a52dcfa" />

<img width="1876" height="940" alt="image" src="https://github.com/user-attachments/assets/14d4bea1-2014-49a0-9b4b-53b81e8d7c14" />

<img width="1919" height="952" alt="image" src="https://github.com/user-attachments/assets/10969d43-3bac-4cf2-8186-c7ec4dcb1a65" />

---

## 5. Crawling  (URL Enumeration)

Đây là bước tìm tất cả các liên kết bên trong site, giúp phát hiện như: Endpoint ẩn, file .js, .css, .xml, html ... , trang con tiềm năng, API. Quá trình này có thể thực hiện để đầu vào cho fuzzing, tạo wordlist hoặc một loại test/analys khác.

Ta sử dụng công cụ *katana* (một công cụ web crawler) để quét:

- Quét trên domain chính: thschool.edu.vn

  ```bash
  katana -u https://thschool.edu.vn -d 5 -o urls_main_domain.txt
  ```

  <img width="1673" height="661" alt="image" src="https://github.com/user-attachments/assets/21a41cca-5491-4f5c-b3d6-a8b4b7df9a2a" />

  Kết quả:

  <img width="1664" height="661" alt="image" src="https://github.com/user-attachments/assets/67b2cc9f-fff9-48a0-b15e-c74cda743978" />

- Quét trên subdomain: chuaboc.thschool.edu.vn

  ```bash
  katana -u https://chuaboc.thschool.edu.vn -d 5 -o urls_chuaboc.txt
  ```

  <img width="1643" height="575" alt="image" src="https://github.com/user-attachments/assets/df463051-6050-4fea-bbef-f2c51fd5c771" />

  <img width="1668" height="660" alt="image" src="https://github.com/user-attachments/assets/10634dee-1b2c-4903-afe3-a29c9cfbce7e" />

- Quét trên subdomain: hoalac.thschool.edu.vn

  ```bash
  katana -u https://hoalac.thschool.edu.vn -d 5 -o urls_hoalac.txt
  ```

  <img width="1662" height="652" alt="image" src="https://github.com/user-attachments/assets/9714ae0a-1b52-4634-8051-69f68031712c" />

  <img width="1674" height="641" alt="image" src="https://github.com/user-attachments/assets/14f57c87-0859-4df0-a3f2-48b178ec5520" />


- Quét trên subdomain: admin.photos.thschool.edu.vn

  ```bash
  katana -u https://admin.photos.thschool.edu.vn -d 5 -o urls_adminphoto.txt
  ```

  <img width="1611" height="652" alt="image" src="https://github.com/user-attachments/assets/b7616ad0-92a5-448d-a65e-e837b1529a2d" />

  <img width="1512" height="647" alt="image" src="https://github.com/user-attachments/assets/33eb7309-3bda-4b25-b824-f611a29655af" />

- Quét trên subdomain: photos.thschool.edu.vn

  ```bash
  katana -u https://photos.thschool.edu.vn -d 5 -o urls_photos.txt
  ```

  <img width="1632" height="657" alt="image" src="https://github.com/user-attachments/assets/d181db39-9e41-4154-abf1-537e4a30d9ad" />

  <img width="1624" height="646" alt="image" src="https://github.com/user-attachments/assets/8d4b2d0a-cd6c-456b-a8ee-819032d4d9a0" />

- Quét trên subdomain: library.thschool.edu.vn

  ```bash
  katana -u https://library.thschool.edu.vn -d 5 -o urls_library.txt
  ```

  <img width="1291" height="456" alt="image" src="https://github.com/user-attachments/assets/5dd48209-ac23-41f9-a063-2667f145c799" />

- Quét trên subdomain: vinh.thschool.edu.vn

  ```bash
  katana -u https://vinh.thschool.edu.vn -d 5 -o urls_vinh.txt
  ```

  <img width="1594" height="647" alt="image" src="https://github.com/user-attachments/assets/5413bae1-1513-4d80-b73d-4f84a3c58ae4" />

  <img width="1634" height="644" alt="image" src="https://github.com/user-attachments/assets/058fa5d0-34c6-43e3-89e9-53af65aa14c6" />

---

## 6. Directory, File Enumeration (Thu thập, liệt kê đường dẫn, thu mục, tệp tin ẩn)

Đây là bước liệt kê các đường dẫn, thư mục và tệp tin ẩn hoặc không được hiển thị công khai trên website/server.

Sử dụng wordlist chung cho các công cụ là: */usr/share/wordlists/dirb/common.txt* hoặc */usr/share/wordlists/dirb/big.txt*

Thực hiện quét bằng công cụ ***Gobuster*** (một công cụ quét thư mục và tệp tin ẩn trên web server, hoạt động bằng cách gửi hàng loạt HTTP request đến server web, dựa trên wordlist):

```bash
gobuster dir -u https://thschool.edu.vn/ -w /usr/share/wordlists/dirb/common.txt -t 50 
```

Kết quả thực hiện:

<img width="1690" height="733" alt="Screenshot 2025-07-19 225301" src="https://github.com/user-attachments/assets/70bbbcfe-4660-409e-8b37-ee02713e02b1" />

<img width="1694" height="801" alt="image" src="https://github.com/user-attachments/assets/7bdbbaea-12a0-4ccc-a60e-23e389705729" />

&rarrr; có thể thấy một số File nhạy cảm như: *.htaccess, .hta, .htpasswd* bị chặn truy cập (403), tuy nhiên có thể thấy một số đường dẫn thư mục tiềm năng như: /admin, /index.php, /robots.txt, ...

Sử dụng công cụ khác là ***ffuf*** (một công cụ fuzzing phổ biến, có thể dùng Directory/file brute-force để quét):

```bash
ffuf -u https://thschool.edu.vn/FUZZ -w /usr/share/wordlists/dirb/big.txt -t 100
```

Kết quả thực hiện:

<img width="1691" height="743" alt="image" src="https://github.com/user-attachments/assets/b3f3186d-6e0a-4b7a-b584-8a95fd58e74d" />

<img width="1676" height="751" alt="image" src="https://github.com/user-attachments/assets/a9326bfa-0fb0-4e05-983a-2bc8969343b3" />

<img width="1675" height="680" alt="image" src="https://github.com/user-attachments/assets/efcb233a-4dff-4540-a55c-88de4b271afe" />

Một công cụ khác để liệt kê đường dẫn, thư mục, file ẩn là ***dirserch*** (tương đói khó dùng):

```bash
python3 /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -e php,html,js -u https://thschool.edu.vn /usr/share/wordlists/dirb/common.txt
```

<img width="1687" height="737" alt="image" src="https://github.com/user-attachments/assets/deebee96-a7cf-4a69-b71e-8ba2566ae1fc" />

<img width="1680" height="795" alt="image" src="https://github.com/user-attachments/assets/b6f5a5c3-d94e-4866-b1c1-dfd9843e3097" />

<img width="1695" height="805" alt="image" src="https://github.com/user-attachments/assets/24407993-e399-49ae-8284-138f1caa95ea" />

&rarr; Kết quá tương đối giống với các công cụ đã quét ở trên.

---

## 7. Parameter Discovery

Đây là bước tìm kiếm các tham số đầu vào (parameters) mà ứng dụng web chấp nhận, từ đó kiểm tra xem chúng có dễ bị tấn công như SQL Injection, XSS, SSRF, LFI... hay không.

Thực hiện sử dụng công cụ ***Arjun*** (một công cụ mã nguồn mở bằng ngôn ngữ python, dùng để tự động tìm ra các tham số (parameter) ẩn mà các ứng dụng web có thể xử lý, hay là tìm tham số HTTP GET/POST tiềm ẩn).

- Quét URL chính: *https://thschool.edu.vn*

```bash
arjun -u https://thschool.edu.vn
```

<img width="1349" height="239" alt="image" src="https://github.com/user-attachments/assets/007b10a2-6cf0-4081-9fed-81879f63167a" />

&rarr; Đã phát hiện 13 tham số có thể sử dụng trong tấn công khai thác lỗ hổng, tuy nhiên bị chặn sau tiếp tục quét.

- Quét một số các đường dẫn, subdomains khác đã thu thập được từ trên:
  
  - *https://thschool.edu.vn/admin/login.php*
    
    ```bash
    arjun -u https://thschool.edu.vn/admin/login.php
    ```
    
    <img width="1006" height="273" alt="image" src="https://github.com/user-attachments/assets/97aedcb5-4b7c-4c6e-9add-da74fe7bfcd2" />

  - *https://thschool.edu.vn/phpMyAdmin*
 
    ```bash
    arjun -u https://thschool.edu.vn/phpMyAdmin
    ```
    
    <img width="1372" height="253" alt="image" src="https://github.com/user-attachments/assets/203d616a-260d-422b-a08b-2f71ec12a1ef" />
    
  - *https://admin.photos.thschool.edu.vn/login*
 
    ```bash
    arjun -u https://admin.photos.thschool.edu.vn/login
    ```
 
    <img width="1314" height="240" alt="image" src="https://github.com/user-attachments/assets/645cc183-9076-4b19-9c9e-14c00720b1c3" />

    &rarr; Phát hiện một parameter "based on: body length", tức nội dung trả về dài ngắn khác nhau → tham số có ảnh hưởng.

  - *https://photos.thschool.edu.vn*
 
    ```bash
    arjun -u https://admin.photos.thschool.edu.vn/login
    ```

    <img width="1680" height="269" alt="image" src="https://github.com/user-attachments/assets/a48e3aa4-1279-401c-a105-6dd72f256511" />

  - *https://hoalac.thschool.edu.vn*
 
    ```bash
    arjun -u https://hoalac.thschool.edu.vn
    ```

    <img width="1663" height="733" alt="image" src="https://github.com/user-attachments/assets/f019f6c6-f5e7-415b-89ab-80fe3bcf43d9" />

    <img width="1672" height="629" alt="image" src="https://github.com/user-attachments/assets/16c8cea0-7e15-4584-8926-30a01dcb3c58" />

    &rarr; Có thể thấy một số các parameter "based on: http code", tức mã phản hồi (status code) HTTP khác nhau → server xử lý khác nhau dựa vào tham số. Ngoài ra "based on: param name reflection" là tham số được phản chiếu lại trong nội dung HTML (dễ dẫn đến XSS).

  - *https://chuaboc.thschool.edu.vn*
 
    ```bash
    arjun -u https://chuaboc.thschool.edu.vn
    ```

    <img width="1684" height="701" alt="image" src="https://github.com/user-attachments/assets/f7042a42-6d8f-4dd6-8f76-a1863d25788b" />

    <img width="1675" height="594" alt="image" src="https://github.com/user-attachments/assets/ae69cb18-59be-40de-94f6-5e402b22a614" />

    &rarr; Các parrameter phát hiện "based on" tương đồng với url: https://hoalac.thschool.edu.vn

  - *https://vinh.thschool.edu.vn*
 
    ```bash
    arjun -u https://vinh.thschool.edu.vn
    ```

    <img width="1680" height="365" alt="image" src="https://github.com/user-attachments/assets/8665729e-caf5-4de7-97ff-2d715ea38a15" />

---

## 8. Google Dorking

Đây là bước tìm kiếm các thông tin nhạy cảm bị rò rỉ công khai trên internet hoặc là mã nguồn hoặc thông tin bị rò rỉ liên quan đến domain trên GitHub.

Một trang hữu ích và phổ biến là *https://www.exploit-db.com/google-hacking-database*

<img width="1911" height="885" alt="image" src="https://github.com/user-attachments/assets/f6d89799-79ae-4b13-a864-1b721c0d94f1" />

Một trang web khác tìm kiếm tiện hơn là: *https://nitinyadav00.github.io/Bug-Bounty-Search-Engine/*

<img width="1898" height="882" alt="image" src="https://github.com/user-attachments/assets/bc001d03-844a-49a3-b0b2-5ae6d972c786" />

---

## ✅ Kết luận:

Thu được một số kết quả trong quá trình Reconnaissance.

---
