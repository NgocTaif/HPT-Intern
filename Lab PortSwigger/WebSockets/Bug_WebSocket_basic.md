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

  - **Vulnerability Detection**: kiểm tra xem máy chủ WebSockets có bị lỗ hổng WebSockets đã biết hay không, ví dụ như: CSWSH, ... cũng như các CVE khác nhau tùy thuộc       vào các thư viện được phát hiện.
 
    <img width="1536" height="587" alt="image" src="https://github.com/user-attachments/assets/b8d5a1e1-3928-4132-a3f2-9edd10c6cdb6" />
 
- Link github: _https://github.com/PalindromeLabs/STEWS_


