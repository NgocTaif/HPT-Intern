# Lỗ hổng trong WebSockets

---

## WebSockets là gì?

- WebSockets là một giao thức truyền thông hai chiều, full duplex được khởi tạo qua HTTP.
  
- Chúng thường được sử dụng trong các ứng dụng web hiện đại để truyền tải dữ liệu và các lưu lượng không đồng bộ khác.

---

## Sự khác biệt giữa HTTP và WebSockets

- HTTP:

  - Cơ chế: request → response, client - server, xong là hết, 1 giao dịch (transaction) độc lập. Mỗi lần cần dữ liệu mới, client phải gửi request mới. Kể cả khi connection       TCP còn mở (HTTP keep-alive), thì nó vẫn chỉ phục vụ từng request → response riêng lẻ.
 
- WebSockets:

  - Cơ chế: ban đầu cũng đi qua HTTP (handshake). Sau đó “nâng cấp” connection thành WebSocket.
 
  - Đặc điểm:

    - Connection lâu dài (long-lived).
    
    - 2 chiều (full-duplex): cả client và server đều có thể gửi message bất kỳ lúc nào, không cần chờ request.
   
    - Không mang tính “transaction” request/response như HTTP.

    - Rất hiệu quả khi cần real-time.
   
  - Ví dụ:
 
    - Ứng dụng chứng khoán: server push giá cổ phiếu ngay khi thay đổi.

    - Chat app: tin nhắn từ bạn bè hiển thị tức thì, không cần polling.
   
---

## How are WebSocket connections established?

- Kết nối WebSocket thường được tạo ra bằng cách sử dụng JavaScript phía client như sau:

  ```
  var ws = new WebSocket("wss://normal-website.com/chat");
  ```

  Giao thức ws thiết lập WebSockets sử dụng một kết nối không được mã hóa. Còn wss sử dụng TLS.

- Để thiết lập kết nối, trình duyệt và máy chủ thực hiện một cuộc bắt tay WebSocket qua HTTP. Trình duyệt phát đi một yêu cầu bắt tay WebSocket như sau:

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

  Tại thời điểm này, kết nối mạng vẫn đang mở và có thể được sử dụng để gửi các tin nhắn WebSocket theo cả hai hướng.

**NOTE**:

- Một vài đặc điểm của WebSocket handshake messages đáng chú ý:

  - Các header như Connection và Upgrade trong cả request và response cho biết rằng đây là một cuộc bắt tay WebSocket.
 
  - Header Sec-WebSocket-Version request chỉ định phiên bản giao thức WebSocket mà client muốn sử dụng. Thông thường, đây là phiên bản 13.
 
  - Header Sec-WebSocket-Key request chứa một giá trị ngẫu nhiên được mã hóa Base64, cái mà nên được tạo ngẫu nhiên trong mỗi yêu cầu bắt tay.
 
  - Header Sec-WebSocket-Accept request chứa một mã hash của giá trị được gửi trong header Sec-WebSocket-Key request, nối với một chuỗi cụ thể được định nghĩa trong đặc       tả giao thức. Điều này được thực hiện để ngăn chặn các phản hồi gây hiểu lầm do cấu hình sai máy chủ hoặc lưu trữ proxy.

---

## What do WebSocket messages look like?

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

---

## Điều khiển lưu lượng WebSocket

- Tìm kiếm lỗ hổng bảo mật WebSockets thường liên quan đến việc thực hiện các thao tác với chúng theo những cách mà ứng dụng không mong đợi.

- Ta có thể sử dụng Burp Suite để:

  - Intercept and modify WebSocket messages.
    
  - Replay and generate new WebSocket messages.
    
  - Manipulate WebSocket connections.
 
---
 
### Intercepting and modifying WebSocket messages

- Ta có thể sử dụng Burp Suite để thực hiện intercept and modify WebSocket messages, theo flow như dưới:

  - Mở trình duyệt của Burp, hoặc trình duyệt có Burp là proxy.
 
  - Duyệt đến chức năng ứng dụng sử dụng WebSockets. Ta có thể xác định rằng WebSockets đang được sử dụng bằng cách sử dụng ứng dụng và tìm kiếm các mục xuất hiện trong      tab lịch sử WebSockets trong Burp Proxy.
 
  - Trong tab Intercept của Burp Proxy, đảm bảo rằng việc intercpet đã được bật.
 
  - Khi một tin nhắn WebSocket được gửi từ trình duyệt hoặc máy chủ, nó sẽ được hiển thị trong tab Intercept để ta có thể xem hoặc chỉnh sửa. Nhấn nút Forward để chuyển      tiếp tin nhắn.
 
---

### Replaying and generating new WebSocket messages

- Ta có thể replay các tin nhắn riêng lẻ và tạo ra các tin nhắn mới. Bạn có thể thực hiện điều này bằng cách sử dụng Burp Repeater:

  - Trong Burp Proxy, chọn một tin nhắn trong WebSockets history, hoặc trong tab Interception, và chọn "Send to Repeater".
 
  - Tại đây ta có thể chỉnh sửa tin nhắc được chọn, và gửi nó qua lại.
 
  - Ta có thể nhập một tin nhắn mới và gửi nó theo cả hai hướng, đến client hoặc server.
 
  - Trong bảng "History" của Burp Repeater, ta có thể xem lịch sử các tin nhắn đã được truyền qua kết nối WebSocket. Điều này bao gồm các tin nhắn mà ta đã tạo trong         Burp Repeater, cũng như bất kỳ tin nhắn nào được tạo bởi trình duyệt hoặc máy chủ thông qua cùng một kết nối.
 
  - Nếu muốn chỉnh sửa và gửi lại bất kỳ tin nhắn nào trong bảng History, ta có thể làm bằng cách chọn tin nhắn và chọn "Edit and resend".
 
---

### Manipulating WebSocket connections

- Đôi khi có nhiều tình huống mà việc thao tác với quá trình bắt tay WebSocket có thể là cần thiết:

  - Nó có thể giúp bạn tiếp cận nhiều bề mặt tấn công hơn.
 
  - Một số cuộc tấn công có thể khiến kết nối của bạn bị ngắt, vì vậy bạn cần thiết lập một kết nối mới.
 
  - Các tokens hoặc dữ liệu khác trong yêu cầu bắt tay ban đầu có thể đã lỗi thời và cần được cập nhật.
 
- Ta có thể thao tác với quy trình bắt tay WebSocket bằng Burp Repeater:

  - Gửi WebSockets message tới Burp Repeater.
 
  - Trong Burp Repeater, nhấp vào biểu tượng bút chì bên cạnh URL WebSocket. Điều này mở ra một wizard cho phép bạn kết nối với một WebSocket đã kết nối, sao chép một        WebSocket đã kết nối, hoặc kết nối lại với một WebSocket đã ngắt kết nối.
 
  - Nếu chọn sao chép một WebSocket đã kết nối hoặc kết nối lại với một WebSocket đã ngắt kết nối, thì trình hướng dẫn sẽ hiển thị chi tiết đầy đủ về yêu cầu bắt tay         WebSocket, mà ta có thể chỉnh sửa theo nhu cầu trước khi thực hiện bắt tay.
 
  - Khi ta nhấp vào "Connect", Burp sẽ cố gắng thực hiện quy trình bắt tay đã được cấu hình và hiển thị kết quả. Nếu một kết nối WebSocket mới được thiết lập thành công,     ta có thể sử dụng nó để gửi các tin nhắn mới trong Burp Repeater.
 
---

## WebSockets security vulnerabilities

- Về nguyên tắc, hầu hết mọi lỗ hổng bảo mật web đều có thể phát sinh liên quan đến WebSockets:

  - Dữ liệu từ người dùng được cung cấp cho máy chủ có thể bị xử lý theo những cách không an toàn, dẫn đến các lỗ hổng như SQLi hoặc XEE.

  - Một số lỗ hổng mù có thể tiếp cận qua WebSockets chỉ có thể phát hiện bằng kỹ thuật ngoài băng (OAST).
  
  - Nếu dữ liệu do kẻ tấn công kiểm soát được truyền qua WebSockets tới những người dùng ứng dụng khác, điều đó có thể dẫn đến XSS hoặc các lỗ hổng client-side khác.

---

### Manipulating WebSocket messages to exploit vulnerabilities

- Đa số các lỗ hổng dựa trên input ảnh hưởng đến WebSocket có thể được tìm thấy và khai thác bằng cách can thiệp vào nội dung của các tin nhắn WebSocket.

- Ví dụ, giả sử một ứng dụng chat sử dụng WebSockets để gửi tin nhắn trò chuyện giữa trình duyệt và máy chủ. Khi một người dùng gõ một tin nhắn trò chuyện, một tin nhắn WebSocket như sau sẽ được gửi đến máy chủ:

  ```json
  {"message":"Hello Carlos"}
  ```

  Nội dung của tin nhắn được truyền (một lần nữa qua WebSockets) đến một người dùng trò chuyện khác, và được hiển thị trong trình duyệt của người dùng như sau:

  ```
  <td>Hello Carlos</td>
  ```

- Trong tình huống này, nếu không có bất kỳ xử lý đầu vào hoặc biện pháp bảo vệ nào khác đang hoạt động, một kẻ tấn công có thể thực hiện một cuộc tấn công XSS bằng cách gửi tin nhắn WebSocket sau:

  ```
  {"message":"<img src=1 onerror='alert(1)'>"}
  ```

### Lab Manipulating WebSocket messages to exploit vulnerabilities

- Giao diện trang web shop online ta thực hiện khai thác, để ý có thể thấy có chức năng live chat:

  <img width="1919" height="940" alt="image" src="https://github.com/user-attachments/assets/37095237-2cde-41af-bf5a-01e594ee3d11" />

  Chức năng live chat cho phép chat với chatbot:

  <img width="1916" height="870" alt="image" src="https://github.com/user-attachments/assets/7a254c17-f95b-4a2a-b45a-159749fd53f5" />

- Trong Burp Suite, tại WebSocket History, ta có thể thấy chat message được gửi thông qua một WebSocket message:

  <img width="1919" height="918" alt="image" src="https://github.com/user-attachments/assets/36214f49-370d-4698-b279-bd5177987b3a" />

- Thực hiện gửi một tin nhắn mới với nội dung là dấu "<", quan sát trong Burp proxy ở phần WS ta thấy dấu "<" đã được HTML-encoded ở client trước khi gửi đi:

  <img width="1919" height="925" alt="image" src="https://github.com/user-attachments/assets/102e3468-f658-46ef-bef2-1fd2cc7176db" />

- Nếu ta thực hiện gửi một tin nhắn với nội dung _<script>alert(“ngoctai”)</script>_ và gửi, ta có thể thấy nó được hiển thị trong trình duyệt của người dùng như sau:

  <img width="1919" height="947" alt="image" src="https://github.com/user-attachments/assets/062e18f2-8aa0-4bf9-a141-98c260455bb8" />

  Tuy nhiên, nó sẽ được HTML-endcoded trước khi gửi đi như đã đề cập ở trên:
  
  <img width="1919" height="929" alt="image" src="https://github.com/user-attachments/assets/b2ed51da-00fd-4254-b0bf-ee184eb51cf6" />

- Thực hiện bật Burp Interception, sau đó gửi một tin nhắn với nội dung _<img src=1 onerror='alert(1)'>_, sau đó gửi, ta có thể thấy WebSocket message bắt được:

  <img width="1919" height="930" alt="Screenshot 2025-09-10 171608" src="https://github.com/user-attachments/assets/d3c4de14-6167-463c-9101-65ea50d5053c" />

  Sửa lại message dưới dạng sau và forward các messages sau:

  <img width="1919" height="919" alt="Screenshot 2025-09-10 171637" src="https://github.com/user-attachments/assets/da7efacb-f00f-43c1-9aed-859980a83978" />

  <img width="1912" height="934" alt="image" src="https://github.com/user-attachments/assets/e20e4c09-1c02-4bc0-b4ae-13cddab60b2c" />

- Quay lại trình duyệt, ta có thể thấy một thông báo đã được kích hoạt trong trình duyệt &rarr; kích hoạt được lỗi XSS và đương nhiên điều này cũng sẽ xảy ra trong trình duyệt của nhân viên hỗ trợ chatbot:

  <img width="1919" height="940" alt="Screenshot 2025-09-10 171709" src="https://github.com/user-attachments/assets/1209e5e2-56cf-4b84-85a1-880a238378d0" />

**NOTE**: Thử với <svg onload=alert(1)> hay <sctipt></script> thì méo được, chắc do cách server xử lý thẻ tag.

---

### Manipulating the WebSocket handshake to exploit vulnerabilities

- Some WebSockets vulnerabilities can only be found and exploited by manipulating the WebSocket handshake. These vulnerabilities tend to involve design flaws, such as:

  - Misplaced trust in HTTP headers to perform security decisions, such as the X-Forwarded-For header.
  
  - Flaws in session handling mechanisms, since the session context in which WebSocket messages are processed is generally determined by the session context of the handshake message.

  - Attack surface introduced by custom HTTP headers used by the application.
 
### Lab Manipulating the WebSocket handshake to exploit vulnerabilities

<img width="1919" height="942" alt="Screenshot 2025-09-11 143429" src="https://github.com/user-attachments/assets/f0de653f-4c05-4f49-bf70-5a634d3baafe" />

<img width="1919" height="937" alt="Screenshot 2025-09-11 143549" src="https://github.com/user-attachments/assets/a3bdbfd8-7d11-4c8f-a511-680efdc42ed9" />

<img width="1918" height="924" alt="Screenshot 2025-09-11 143841" src="https://github.com/user-attachments/assets/cd3eaa21-ba65-495f-bde1-c979ad2a0c35" />

<img width="1919" height="940" alt="Screenshot 2025-09-11 144755" src="https://github.com/user-attachments/assets/1a5a6d76-d839-437f-933a-cec694c91e35" />

<img width="1919" height="1005" alt="Screenshot 2025-09-11 145258" src="https://github.com/user-attachments/assets/7fdf009c-d994-437e-8c63-917e67f16794" />

<img width="1919" height="1012" alt="Screenshot 2025-09-11 145841" src="https://github.com/user-attachments/assets/fb2bf6b4-bb61-4696-8ea4-ca64549c5cd8" />

<img width="1919" height="1006" alt="Screenshot 2025-09-11 145901" src="https://github.com/user-attachments/assets/692b0edd-8e9a-49e6-8450-ed2c8b281713" />

<img width="1919" height="956" alt="Screenshot 2025-09-11 150835" src="https://github.com/user-attachments/assets/22901038-78ac-427e-acef-f808ae02b057" />

<img width="1911" height="875" alt="Screenshot 2025-09-11 150906" src="https://github.com/user-attachments/assets/fae10202-f634-4802-b2a3-86ca685dac14" />

---

### Cross-site WebSocket hijacking

- Cross-site WebSocket hijacking (also known as cross-origin WebSocket hijacking) involves a cross-site request forgery (CSRF) vulnerability on a WebSocket handshake.

- It arises when the WebSocket handshake request relies solely on HTTP cookies for session handling and does not contain any CSRF tokens or other unpredictable values.

- An attacker can create a malicious web page on their own domain which establishes a cross-site WebSocket connection to the vulnerable application. The application will handle the connection in the context of the victim user's session with the application.

- The attacker's page can then send arbitrary messages to the server via the connection and read the contents of messages that are received back from the server. This means that, unlike regular CSRF, the attacker gains two-way interaction with the compromised application.

- Since a cross-site WebSocket hijacking attack is essentially a CSRF vulnerability on a WebSocket handshake, the first step to performing an attack is to review the WebSocket handshakes that the application carries out and determine whether they are protected against CSRF.

- In terms of the normal conditions for CSRF attacks, you typically need to find a handshake message that relies solely on HTTP cookies for session handling and doesn't employ any tokens or other unpredictable values in request parameters.

- For example, the following WebSocket handshake request is probably vulnerable to CSRF, because the only session token is transmitted in a cookie:

  ```
  GET /chat HTTP/1.1
  Host: normal-website.com
  Sec-WebSocket-Version: 13
  Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
  Connection: keep-alive, Upgrade
  Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
  Upgrade: websocket
  ```

  NOTE: The Sec-WebSocket-Key header contains a random value to prevent errors from caching proxies, and is not used for authentication or session handling purposes.

### Lab: Cross-site WebSocket hijacking

- Tương tự các bài lab trước, ta cần để ý chức năng Live chat của website shopping này:
  
  <img width="1919" height="932" alt="image" src="https://github.com/user-attachments/assets/20d1d4f9-8baa-4c3a-b8d9-06f6d5244875" />

- Thực hiện gửi một message bất kỳ lên live chat. Sau đó load lại trang Live chat.

- Tiến hành kiểm tra trong WebSocket history của Burp Suite, ta nhận thấy có điểm đặc biết là client app gửi một lệnh là "READY" lên server để lấy lại 

