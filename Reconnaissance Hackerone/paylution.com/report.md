# BÁO CÁO QUÁ TRÌNH THỰC HIỆN RECONNAISSANCE site *paylution.com trên H1

---

Mục tiêu (target) thực hiện quá trình Reconnaissance là site *paylution.com* trên platform Hackerone:

<img width="1892" height="943" alt="image" src="https://github.com/user-attachments/assets/1daa9eb2-9c18-495c-be5c-642855b7cb93" />

<img width="1914" height="889" alt="image" src="https://github.com/user-attachments/assets/2867fbbb-cbd7-4a20-bfc3-b727332d8178" />

---

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

---

## 2. Thu thập subdomain 

### Passive Enumeration (Thụ động) kết hợp công cụ OSINT (open source intelligence)

--> quá trình thu thập thông tin mà không tương tác trực tiếp với hệ thống mục tiêu.


> Lấy subdomain từ certificate transparency logs (crt.sh, certspotter…)

 crt.sh – Certificate Transparency Search - 1 nền tảng tra cứu các chứng chỉ số (SSL certificate) đã được cấp cho một domain/subdomain.

Mỗi khi một website được cấp chứng chỉ SSL, thông tin đó được ghi lại công khai trong hệ thống Certificate Transparency Log.

--> ta có thể tra cứu tất cả subdomain nào đã từng được dùng để đăng ký SSL. Kể cả những subdomain đã bị xóa, nếu từng có chứng chỉ, vẫn hiện lên ở [crt.sh](https://crt.sh/?q=paydiant.com)

 ![](./images_recon/im2.3.png) 


> Tra cứu DNS thông qua các API public (Shodan, VirusTotal, AlienVault…)

 ![](./images_recon/im2.5.png) 

--> thu thập được các subdomain liên quan đến domain
[ở virustotal](https://www.virustotal.com/gui/domain/paydiant.com/relations)
+ Hoặc tìm dữ liệu bị lộ từ GitHub, Wayback Machine, Google Dork…

Ở đây, t sử dụng chaos.projectdiscovery.io – Chaos Dataset - kho dữ liệu subdomain public được thu thập bởi ProjectDiscovery từ các nguồn bug bounty & các chương trình công khai. Nó tập hợp subdomain của hàng ngàn domain lớn, đã được xác minh trong các chương trình như HackerOne, Bugcrowd, v.v.

 ![](./images_recon/im2.4.png) 

-->  có thể tải toàn bộ .txt danh sách subdomain đã được index ở [chaos](https://chaos.projectdiscovery.io/?trk=public_post-text)

Mục đích chính : Nhanh chóng tìm được subdomain đã tồn tại trước đó. Bây giờ vẫn chưa tạo traffic tới target.

> Cùng với đó, ta có thể kết hợp với công cụ subfinder, assetfinder để tìm thêm subdomain.


```bash
sudo subfinder -d paydiant.com -all > domain1.txt
```
 ![](./images_recon/im2.6.png) 

--> ra 512 subdomain có khả năng tồn tại

```bash
sudo assetfinder --subs-only paydiant.com > domain2.txt
```
 ![](./images_recon/im2.7.png) 

```
cat domain2.txt | wc -l
```
(wc word count ; -l line)

--> ra 164 subdomain có khả năng tồn tại


Cả 2 công cụ trên đều : 

+ Tìm tất cả subdomain liên quan đến paydiant.com bằng cách query các nguồn dữ liệu công khai.

+ Kết quả sẽ được lưu vào file domain2.txt.

+ Không kiểm tra subdomain có “sống” hay không = không gửi DNS query để kiểm tra subdomain có tồn tại không -->  chỉ là danh sách có khả năng tồn tại.

--> sau này, cho hết các subdomain tìm được vào 1 file --> dùng nó kết hợp httpx xem subdomain nó tồn tại hay ko 

Lí do sử dụng 2 công cụ này : 
subfinder được dùng rất nhiều public APIs để thu thập subdomain.(VirusTotal, Shodan, Crt.sh, SecurityTrails, DNSDB, v.v.)

Các API này có thể chứa:
+ Subdomain được index từ chứng chỉ SSL.
+ Subdomain bị lộ từ leaks, threat intel.
+ Subdomain từng được ai đó scan/truy cập

Còn assetfinder chủ yếu dựa vào chứng chỉ SSL công khai qua dịch vụ certspotter (tương tự crt.sh, ở trên ta đã tra thủ công). Rồi rút trích các subdomain từng dùng để đăng ký HTTPS certificate.

---

## 3. Kiểm tra Hosts phản hồi lại không ?
### Brute force

Bởi vì brute force subdomain được xếp vào loại Active Reconnaise, tức là có tương tác với target.

Có hai loại:
+ Brute force thông thường
+ Dùng wordlist.

--> Khuyến khích nên dùng kiểu thứ hai vì kiểu thứ nhất có ghi lại log.

T lấy wordlist chuyển dụng cho việc brute force tìm kiếm subdomain [n0kovo_subdomains](https://github.com/n0kovo/n0kovo_subdomains) 

(tìm thấy wordlist liên quan cập nhật gần nhất nữa)

Khi chạy : 
```
ffuf -u https://FUZZ.paydiant.com -w /home/kali/n0kovo_subdomains/n0kovo_subdomains_medium.txt -o domain3.txt

```

Lấy từng dòng trong wordlist.txt, ví dụ: dev, test, staging, uat, v.v. Ghép thành URL:
```
https://dev.paydiant.com  
https://test.paydiant.com  
```
Sau đó, nó gửi HTTP request đến từng URL.

+ Nếu subdomain tồn tại, DNS phân giải thành IP, và server phản hồi → kết quả sẽ hiện ra.

+ Nếu subdomain không tồn tại (NXDOMAIN) hoặc không có web server chạy, request sẽ fail.


Mục tiêu của lệnh này:
Tìm ra các subdomain ẩn/chưa bị lộ của paydiant.com bằng cách brute-force trên subdomain prefix.


Tại sao dùng ffuf để fuzz subdomain?

--> Vì các công cụ như subfinder, assetfinder, crt.sh... chỉ thu thập dữ liệu có sẵn. Nhưng nhiều subdomain:

+ Không có trong public logs
+ Không ai từng index
+ Nhưng lại vẫn đang hoạt động

→ Fuzz bằng ffuf giúp khai phá thêm subdomain bí mật.

Nhưng đợi hơi lâu, thì đồng thời t lấy tạm domain1.txt và domain2.txt từ 2 công cụ trước đó gộp thành 1 file domain.txt dùng cho httpx luôn.


```
sort -u domain1.txt domain2.txt > domain.txt
cat domain.txt | wc -l

```
 ![](./images_recon/im2.8.png) 

 609 subdomain thu được từ 2 công cụ assetfinder và subfinder, kết hợp thêm cả subdomain list từ chaos và crt.sh

```
 cat domain.txt | sudo httpx > live_domains.txt
```
Mỗi dòng (subdomain) được chuyển qua cho httpx, rồi httpx sẽ thực hiện:
+ Gửi HTTP/HTTPS request tới từng subdomain đó.

+ Kiểm tra xem host có phản hồi hay không.

+ Nếu host “sống” (có phản hồi), thì in ra kết quả (thường là URL có status 200/301...).

--> Mặc định httpx kiểm tra trên cổng 80 (HTTP) và 443 (HTTPS).

 ![](./images_recon/im2.9.png) 

Đợi hơi lâu, nên tôi dùng công cụ subzy cùng lúc đó : 
```
subzy run --targets domain.txt
```

 ![](./images_recon/im2.10.png) 

Sử dụng subzy để kiểm tra tất cả (kể cả host sống hoặc die) xem có cái nào takeover được không

Subdomain Takeover là tình huống khi một subdomain của website vẫn còn tồn tại (DNS còn trỏ),
nhưng dịch vụ mà nó trỏ đến đã bị xóa, bị bỏ hoặc chưa được cấu hình đúng.

--> Điều này cho phép người khác chiếm quyền kiểm soát subdomain đó, bằng cách tạo lại đúng dịch vụ mà subdomain đang trỏ về.

Giả dụ có subdomain 'blog.example.com'
Subdomain này trỏ đến dịch vụ GitHub Pages:
```
CNAME → user123.github.io
```
Nhưng người dùng user123 đã xóa GitHub repo hoặc không còn sở hữu repo đó nữa. Mà subdomain vẫn trỏ về user123.github.io, nhưng không có gì ở đó. Vậy hacker chỉ cần tạo GitHub Pages có tên user123.github.io, thì:
blog.example.com → hiển thị nội dung hacker upload
--> takeover thành công subdomain blog.example.com

Ta sử dụng công cụ nuclei  để scan các loại lỗ hổng khác, có thể kết hợp takeover vào cùng một flow
 ![](./images_recon/imp.1.png) 

 hoặc dùng 
 ```
git clone https://github.com/projectdiscovery/nuclei-templates.git 
 ```

 Sau khi cập nhập template, thường nó có mục takeover nhưng tôi đang ko thấy đâu 
 ```
nuclei -l domain.txt -t nuclei-templates/detect-all-takeovers.yaml
 ```

 dùng nuclei để check lỗi khác cx được.

---

## 4. Crawling (URL Enumeration)
Sau khi có live_domain.txt, ta dùng công cụ katana thu thập endpoints (đường dẫn, param, JS, API...)

 ```
 go install https://github.com/projectdiscovery/katana.git 
 ```
Katana là một công cụ URL crawler (thu thập các đường dẫn) từ ProjectDiscovery, dùng để quét URL từ một trang web (bao gồm link ẩn, JS file, iframe, params…)
Check từng path một để cào dữ liệu.

```
katana -u http://auto5.qa.paydiant.com -o urls.txt
```

Dù đã dùng katana, nuclei để tìm ra các url có thể chứa các link ẩn nhưng không tìm ra được.

[OneforAll](https://github.com/shmilylty/OneForAll/blob/master/docs/en-us/README.md)

```
git clone https://github.com/shmilylty/OneForAll.git
cd OneForAll
pip install -r requirements.txt
```
 ![](./images_recon/imp.2.png)

 --> tập trung vào các câu lệnh trên

 ```
python3 oneforall.py --targets ../live_domains.txt run
 ```

  ![](./images_recon/imp.3.png)  

chỉ định file chứa danh sách tên miền cần quét (ở đây là domain.txt, nằm ở thư mục cha), rồi yêu cầu tool bắt đầu chạy với các target đã chỉ định.

  ``` bash
00:29:15,883 [ALERT] wildcard:47 - b579af57.paydiant.com resolve to: b579af57.paydiant.com. IP: {'3.33.139.32'} TTL: 3600
00:29:15,928 [ALERT] wildcard:47 - 2ce49aab.paydiant.com resolve to: 2ce49aab.paydiant.com. IP: {'3.33.139.32'} TTL: 3600
00:29:15,975 [ALERT] wildcard:47 - 55e5d03e.paydiant.com resolve to: 55e5d03e.paydiant.com. IP: {'3.33.139.32'} TTL: 3600

00:29:21,265 [INFOR] module:63 - HackerTargetQuery module took 1.1 seconds found 67 subdomains
00:29:21,273 [INFOR] module:63 - CertSpotterQuery module took 1.1 seconds found 0 subdomains
00:29:21,294 [ALERT] utils:273 - GET https://fullhunt.io/api/v1/domain/paydiant.com/subdomains 403 - Forbidden 52
00:29:21,297 [ALERT] utils:282 - {'message': 'Missing X-API-KEY header', 'status': 400}
00:29:21,299 [INFOR] module:63 - AnubisQuery module took 1.1 seconds found 486 subdomains
00:29:21,303 [INFOR] module:63 - FullHuntAPIQuery module took 1.1 seconds found 0 subdomains
00:29:21,353 [INFOR] module:63 - Sublist3rQuery module took 1.2 seconds found 1 subdomains
00:29:21,386 [ALERT] utils:273 - GET https://transparencyreport.google.com/transparencyreport/api/v3/httpsreport/ct/certsearch?include_expired=true&include_subdomains=true&domain=paydiant.com 404 - Not Found 1621               
  ```

 + subdomain b579af57.paydiant.com được resolve (phân giải DNS) thành IP 3.33.139.32.

 Wildcard alert là cảnh báo cho thấy domain sử dụng DNS wildcard – tức là bất kỳ chuỗi nào.paydiant.com cũng trả về IP hợp lệ (thường là một địa chỉ mặc định như trên).

 --> bất kỳ subdomain nào (dù tồn tại hay không) cũng sẽ resolve (phân giải) về cùng một IP.
→ Điều này che giấu số lượng subdomain thực sự tồn tại

-  Những keyword chính : 
    + HackerTargetQuery: tìm được 67 subdomain.

    + CertSpotterQuery: không tìm thấy gì.

    + AnubisQuery: tìm được 486 subdomain.

    + Sublist3rQuery: tìm được 1.

=> Kết quả từ nhiều nguồn sẽ được tổng hợp lại.

Có những đường dẫn khi chạy hiện ra cảnh báo khi thiếu API Key:
```bash
[ALERT] utils:273 - GET https://fullhunt.io/api/v1/domain/paydiant.com/subdomains 403 - Forbidden
[ALERT] utils:282 - {'message': 'Missing X-API-KEY header', 'status': 400}
```
--> Tool cố gọi API fullhunt.io nhưng bị 403 Forbidden vì ta chưa cung cấp API Key.

```bash
00:34:08,163 [ALERT] wildcard:47 - 8c145ec1.paydiant.com resolve to: 8c145ec1.paydiant.com. IP: {'3.33.139.32'} TTL: 3600
00:34:08,206 [ALERT] wildcard:47 - faa2b4da.paydiant.com resolve to: faa2b4da.paydiant.com. IP: {'3.33.139.32'} TTL: 3600
00:34:08,251 [ALERT] wildcard:47 - 6c94f671.paydiant.com resolve to: 6c94f671.paydiant.com. IP: {'3.33.139.32'} TTL: 3600
00:34:08,252 [INFOR] utils:700 - Attempting to request http://8c145ec1.paydiant.com
00:34:09,740 [ALERT] wildcard:77 - Request: http://8c145ec1.paydiant.com Status: 200 Size: 278538
00:34:09,741 [INFOR] utils:700 - Attempting to request http://faa2b4da.paydiant.com
00:34:11,300 [ALERT] wildcard:77 - Request: http://faa2b4da.paydiant.com Status: 200 Size: 278346
00:34:11,301 [INFOR] utils:700 - Attempting to request http://6c94f671.paydiant.com
00:34:12,939 [ALERT] wildcard:77 - Request: http://6c94f671.paydiant.com Status: 200 Size: 293396
00:34:13,078 [ALERT] wildcard:121 - The domain paydiant.com enables wildcard
00:34:13,079 [INFOR] collect:44 - Start collecting subdomains of paydiant.com

```

Ở trên, ta thấy công cụ tiếp tục gửi request HTTP đến subdomain giả đó. Nếu server vẫn trả về status 200 OK và có nội dung HTML trả về (có size), thì khẳng định thêm rằng wildcard DNS đang trả về một trang mặc định.

Rồi sau đó lại dùng katana để xét từng đường dẫn. Nhưng mà katana t tra từng link không ra j cả nên dùng waybackurls.

> Lưu ý : 

katana là công cụ crawler — truy cập vào website theo thời gian thực để phân tích HTML source để tìm link (từ href=, src=, form action=, v.v.)

Vấn đề xảy ra nếu:
+ Website dùng JavaScript render (SPA như React/Vue) ➜ katana không thấy gì vì nó không thực thi JS.

+ Website chặn bot/crawler ➜ như dùng robots.txt, Cloudflare, WAF.

+ Website không public nhiều link trong homepage ➜ katana không có gì để theo.

waybackurls không crawl trang hiện tại, mà lấy dữ liệu đã lưu từ Wayback Machine (Internet Archive) .Tức là các URL đã từng tồn tại, kể cả hiện tại bị ẩn, redirect, hay xoá

Vậy nên dù site có chặn bot, dùng JS, hay minimal page ➜ waybackurls vẫn tìm được.

Chạy lệnh waybackurls : 
```
echo "https://paydiant.com" | waybackurls > urls.txt
```
 ![](./images_recon/imp.4.png) 


Dựa vào đây ta có thể đã bắt đầu kiềm thử ở bước này , tất nhiên vẫn sẽ còn bước thu thập thông tin về hệ điều hành, csdl mà nền tảng web đang sử dụng
```
cat urls.txt | grep =
```
--> các URL có dấu = thường là mục tiêu tốt cho kiểm tra XSS, LFI, SQLi, v.v. trong pentest.

--> kiểm tra từng URL trên trình duyệt

---

## 5. Thu thập thông tin về OS, fingerprint

> Dựa vào gói tin HTTP Request khi check trong Burp Suite 

> Dựa vào extension wappalyzer

<img width="473" height="508" alt="image" src="https://github.com/user-attachments/assets/74acd34b-7dc8-4511-b83b-f6e2ccd4acaf" />

> Dựa vào Censys (https://censys.io)

Một công cụ tìm kiếm chuyên dụng duy trì dữ liệu nghiên cứu Internet công khai và cung cấp giao diện người dùng dựa trên web để truy vấn các bộ dữ liệu như địa chỉ IPv4, trang web và chứng chỉ web. 

---

## 6. Thu thập thêm các thông tin ẩn bằng Google Dork - GitHub Recon
- Endpoint ẩn, file config, backup, lỗi… liên quan đến domain mục tiêu

```
intitle:"Index of" ".htpasswd" htpasswd.bak site:paydiant.com

filetype:sql "insert into" (pass|passwd|password) site:paydiant.com

inurl:"build.xml" intext:"tomcat.manager.password" site:paydiant.com

site:paydiant.com inurl:staging

```

- Có thể sử dụng trang web dưới để check : 

<img width="1656" height="618" alt="image" src="https://github.com/user-attachments/assets/ffbb61b4-7f5b-4d18-bd97-72e3f60dfe20" />

 - Git hub Recon 
 
 <img width="1432" height="627" alt="image" src="https://github.com/user-attachments/assets/514c7ae2-faee-482d-b948-9df72c623c47" />




