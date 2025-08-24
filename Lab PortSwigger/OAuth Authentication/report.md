# LỖ HỔNG OAuth 2.0 authentication (PORTSWIGGER)

---

Khi thực hiện lướt web, chúng ta gần như chắc chắn đã bắt gặp các trang web cho phép đăng nhập bằng tài khoản mạng xã hội của mình như: facebook, google, ... Rất có thể tính năng này được xây dựng bằng cách sử dụng framework OAuth 2.0 phổ biến. OAuth 2.0 rất dễ bị để ý với những kẻ tấn công vì nó cực kỳ phổ biến và vốn dĩ dễ mắc lỗi triển khai. Dưới đây ta sẽ tìm hiểu về lỗ hổng trong OAuth authentication.

---

## 1. OAuth là gì?

- OAuth là một khung ủy quyền thường được sử dụng cho phép các trang web và ứng dụng web yêu cầu quyền truy cập hạn chế/nhất định vào tài khoản của người dùng trên một ứng dụng khác. Điều quan trọng là OAuth cho phép người dùng cấp quyền truy cập này mà không tiết lộ thông tin đăng nhập của họ cho ứng dụng yêu cầu.

- Ví dụ, trên một website là: abc.com, thay vì tạo một tài khoản đăng nhập mới ta có thể chọn mục "Đăng nhập bằng Google", lúc này trang web sẽ phải thực hiện xin cấp quyền từ Google để có thể đăng nhập vào trang web abc.com

---

## 2. Cách thức hoạt động của OAuth 2.0

- Như đã nói, bản chất của OAuth 2.0 là cho phép ứng dụng bên thứ ba truy cập một phần dữ liệu của người dùng từ một dịch vụ khác, mà không cần chia sẻ mật khẩu của người dùng.

- Việc này xung quanh ba chủ thể:

  - Client Application (Ứng dụng khách): Ứng dụng hoặc website muốn truy cập dữ liệu người dùng.

  - Resource Owner (Chủ sở hữu tài nguyên): Chính là người dùng – người sở hữu dữ liệu. Ví dụ: người có tài khoản Google.

  - OAuth Service Provider (Nhà cung cấp dịch vụ OAuth): Nơi lưu dữ liệu và cho phép truy cập qua OAuth API. Ví dụ: Google, Facebook, GitHub... Gồm 2 thành phần chính:

      - Authorization Server – xử lý việc đăng nhập, cấp quyền, phát access token.

      - Resource Server – API chứa dữ liệu thật, chỉ trả dữ liệu khi có token hợp lệ.

- Cả hai dạng grant đều theo 4 dạng cơ bản như sau:

<img width="1189" height="1200" alt="image" src="https://github.com/user-attachments/assets/6f29f687-17fc-43a4-9345-787c8c8d2600" />

- Tập trung vào hai dạng grant phổ biển nhất là: Authorization Code Flow & Implicit Flow

  - Authorization Code Flow:
 
    Client application và dịch vụ OAuth trước tiên sử dụng chuyển hướng để trao đổi một loạt các yêu cầu HTTP dựa trên trình duyệt nhằm khởi động quy trình.

    Người dùng sẽ được hỏi có đồng ý với yêu cầu quyền truy cập hay không. Nếu họ chấp nhận, client application được cấp một "mã ủy quyền". Ứng dụng khách sau đó đổi mã      này với dịch vụ OAuth để nhận được một "access token", mà họ có thể sử dụng để thực hiện các cuộc gọi API nhằm lấy dữ liệu người dùng liên quan.

    <img width="837" height="607" alt="image" src="https://github.com/user-attachments/assets/e36521c1-79e0-417e-b87d-0d1cb165b525" />

    **Authorization request**

    Client application gửi một request đến endpoint /authorization của dịch vụ OAuth để yêu cầu quyền truy cập dữ liệu người dùng cụ thể, tùy từng loại dịch vụ OAuth, có     thể là: /auth, ...

    ```http
    GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
    Host: oauth-authorization-server.com
    ```

    ***client_id:*** Tham số bắt buộc chứa định danh duy nhất của ứng dụng khách. Giá trị này được tạo ra khi ứng dụng khách đăng ký với dịch vụ OAuth.

    ***redirected_uri:*** URI, cái mà trình duyệt của người dùng nên được chuyển hướng đến khi gửi mã ủy quyền cho ứng dụng khách. Điều này còn được gọi là "URI              callback" hoặc "endpoint callback".

    ***response_type:*** Xác định loại phản hồi mà ứng dụng khách đang mong đợi. Đối với kiểu authorization code grant, response_type là ***code***.

    **Authorization code grant**

    Nếu người dùng đồng ý với yêu cầu truy cập, trình duyệt của họ sẽ chuyển hướng tới endpoint /callback mà được chỉ định cụ thể trong ***redirected_uri***. Kết quả GET     request sẽ chứa "mã ủy quyền" như một tham số truy vấn. Phụ thuộc vào cấu hình, nó cũng có thể gửi tham số ***state*** với giá trị giống với trong authorization          request.

    ```http
    GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1
    Host: client-app.com
    ```

    **Access token request**

    Một khi client application nhận "mã ủy quyền", nó cần đổi nó để yêu cầu lấy "access token", để làm điều này nó sẽ gửi một server-to-server POST request tới endpoint      /token của dịch vụ OAuth.

    ```http
    POST /token HTTP/1.1
    Host: oauth-authorization-server.com
    …
    client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8
    ```

    ***client_secret:*** Client application phải xác thực chính nó bằng cách bao gồm khóa bí mật mà nó đã được cấp khi đăng ký với dịch vụ OAuth.

    ***grant_type:*** Được sử dụng để chắc chắn endpoint mới biết kiểu cấp quyền nào mà client application muốn sử dụng.

    **Access token grant**

    ```http
    {
    "access_token": "z0y9x8w7v6u5",
    "token_type": "Bearer",
    "expires_in": 3600,
    "scope": "openid profile",
    …
    }
    ```

    **API call**

    Tạo một API call tới endpoint /userinfo của dịch vụ OAuth, access token được gửi trong Authorization: Bearer header để chứng minh rằng client application được phép       truy cập user data.

    ```http
    GET /userinfo HTTP/1.1
    Host: oauth-resource-server.com
    Authorization: Bearer z0y9x8w7v6u5
    ```

    **Resource grant**

    Tài nguyên server sẽ xác thực giá trị của token và trả về dữ liệu người dùng dựa theo scope của access token.

    ```
    {
    "username":"carlos",
    "email":"carlos@carlos-montoya.net",
    …
    }
    ```

  - Implicit flow
 
    Thay vì đầu tiên nhận được mã ủy quyền và sau đó đổi mã đó lấy "access token", ứng dụng khách sẽ nhận được "access token" ngay lập tức sau khi người dùng cho phép.

    Câu hỏi đặt ra là vậy tại sao client application không luôn sử dụng dạng này? Đơn giản là nó kém an toàn hơn rất nhiều

    Khi sử dụng dạng này, tất cả các giao tiếp diễn ra thông qua việc chuyển hướng trình duyệt. Loại này thích hợp hơn cho single-page applications và native desktop         applications (loại app chi dành riêng cho 1 nền tảng), không thể dễ dàng lưu trữ client_secret trên máy chủ.

    <img width="837" height="499" alt="image" src="https://github.com/user-attachments/assets/d9b99f34-a8d8-4b17-84f2-6dd2971d7beb" />

    **Authorization request**

    Một điều khác so với Authorization Code Flow là response_type là ***token***.

    **Access token grant**

    Nếu người dùng đồng ý với quyền truy cập được yêu cầu. Dịch vụ OAuth sẽ chuyển hướng trình duyệt của người dùng đến redirect_uri được chỉ định trong yêu cầu ủy           quyền. Tuy nhiên, thay vì gửi một tham số truy vấn chứa mã ủy quyền, nó sẽ gửi token truy cập và các dữ liệu cụ thể về token khác dưới dạng một URL fragment.

---

## 3. OAuth Authentication

- Mặc dù ban đầu không nhằm mục đích này, OAuth cũng đã phát triển thành một công cụ để xác thực người dùng.

- Bản chất là dùng chính quy trình OAuth để xác thực người dùng (cho phép đăng nhập), chỉ thay đổi mục tiêu từ “truy cập dữ liệu” → dùng chúng để “đăng nhập”.

- Ví dụ: Giả sử bạn đăng nhập vào shopabc.com bằng Google. Lúc này:

  - Bạn không cần tạo tài khoản mới, không cần nhớ mật khẩu cho shopabc.

  - Shopabc chỉ lưu thông tin do Google cung cấp (email, tên).

  - Lần sau bạn chọn "Sign in with Google", shopabc chỉ cần xác minh token hợp lệ là bạn được vào.
 
---
 
## 4. Recon OAuth authentication:

  - Việc thực hiện một số khảo sát cơ bản về dịch vụ OAuth đang được sử dụng có thể giúp bạn định hướng đúng khi xác định các lỗ hổng.
 
  - Nếu một dịch vụ OAuth bên ngoài được sử dụng, bạn có thể xác định được nhà cung cấp cụ thể từ tên máy chủ (hostname) mà yêu cầu ủy quyền được gửi đến (trong request      /authorization).

  - Hostname này giúp ta:

    Biết đó là provider nào (Google, Facebook, GitHub, custom OAuth…)
    
    Tìm tài liệu API công khai (documentation) → biết cách hoạt động của các endpoint.
 
  - Hầu hết OAuth provider chuẩn (và cả OpenID Connect) sẽ có endpoint cấu hình public mà không cần login:
 
    ```http
    https://<provider>/.well-known/oauth-authorization-server
    https://<provider>/.well-known/openid-configuration
    ```

  - Ví dụ, một dữ liệu json được trả về:
 
    ```json
    {
      "authorization_endpoint": "https://login.example-oauth.com/authorize",
      "token_endpoint": "https://login.example-oauth.com/token",
      "userinfo_endpoint": "https://login.example-oauth.com/userinfo",
      "scopes_supported": ["openid", "email", "profile", "admin"],
      "grant_types_supported": ["authorization_code", "implicit", "password"]
    }
    ```

    &rarr; Scope admin và grant type password không được app đề cập → Có thể là điểm tấn công tiềm năng.

---

## 5. Khai thác lỗ hổng OAuth authentication

### 5.1. Các lỗ hổng trong OAuth client application

### Improper implementation of the implicit grant type

- Loại implicit grant thường được dùng cho single-page applications, tuy nhiên cũng thường được sử dụng trong các ứng dụng web truyền thống kiểu client-server vì tính đơn giản tương đối của nó.

- Trong quy trình này, "access token" được gửi từ dịch vụ OAuth tới client application qua trình duyệt của người dùng dưới dạng một phần của URL.

- Client application sau đó truy cập mã thông qua JavaScript.

- Vấn để là: nếu client application muốn duy trì phiên sau khi người dùng đóng trang, nó cần phải lưu trữ dữ liệu người dùng ở đâu đó.

- Do đó để giải quyết, client application sẽ thường gửi dữ liệu này đến server thông qua một POST request, sau đó gán cho người dùng một session cookie. Tuy nhiên, trong kịch bản này, server không có bất kỳ cái gì để so sánh với dữ liệu đã được gửi, điều này có nghĩa là dữ liệu này được tin tưởng ngầm định.

- Do đó, hành vi này có thể dẫn đến một lỗ hổng nghiêm trọng nếu client application không kiểm tra đúng cách "access token" khớp với các dữ liệu khác trong request. Trong trường hợp này, ta có thể đơn giản thay đổi các tham số gửi đến server để mạo danh bất kỳ người dùng nào.  

### Lab: Authentication bypass via OAuth implicit flow

- Giao diện trang web cho phép sử dụng mạng xã hội để đăng nhập:

  <img width="1919" height="943" alt="image" src="https://github.com/user-attachments/assets/63448710-444b-4990-9fc6-4ff666c776df" />

- Trong lab này, ta tiến hành đăng nhập với tài khoản social media có sẵn là: *wiener:peter*. Kết quả sau khi đăng nhập thành công:

  <img width="1919" height="880" alt="image" src="https://github.com/user-attachments/assets/5674761a-5561-4835-86a5-b2081efc2ed4" />

- Sau đó trang thực hiện redirect về trang chủ.

- Thực hiện kiểm tra Burp Suite để xem quy trình OAuth trên trang web. Và khi thực hiện nhập thành công tài khoản mạng xã hội ta thấy nó bắt đầu bằng một request xác thực *GET /auth?client_id=[...]* gửi đến cho server của dịch vụ OAUth như sau:

  <img width="1431" height="516" alt="image" src="https://github.com/user-attachments/assets/157bdbc4-6c3e-4236-a179-b5c197278ad7" />

  Trong request trên, ta sẽ thấy:

  *redirect_uri=https://0a6f00d403a19fc80793a4005200e9.web-security-academy.net/oauth-callback*:	Địa chỉ callback mà Authorization Server sẽ redirect người dùng về sau khi xác thực xong.

  *response_type=token*: Đây là Implicit Flow (yêu cầu access token trực tiếp trong URL fragment thay vì code).

  *scope=openid&profile&email*:	Yêu cầu quyền truy cập thông tin OpenID, profile và email của user.

- Ta sẽ thấy OAuth Server thực hiện tạo request redirect trình duyệt về redirect_uri mà client đã đăng ký, đây chính là url mà chứa access token trả về cho về bên phía client application.

  <img width="1869" height="567" alt="image" src="https://github.com/user-attachments/assets/09a9a28a-bcae-4129-9f62-d8a61df498e6" />

  Fragment #access_token=... → là nơi token được trả về trực tiếp trên URL vì bạn đang dùng Implicit Flow (response_type=token ở request trước).

- Tiếp đó, OAuth server gửi request để tải trang /oauth-callback tới client application.

  <img width="1873" height="604" alt="image" src="https://github.com/user-attachments/assets/07182004-9bbc-4bfc-9848-86e83c14ca53" />

  Có thể thấy, sử dụng token vừa lấy để gọi API /me → mục đích: lấy thông tin profile người dùng (OpenID, email, tên, …) từ nhà cung cấp OAuth.

  Sau khi lấy được thông tin user từ OAuth provider, script sẽ gửi POST request đến endpoint /authenticate của OAuth server để lưu thông tin người dùng cho những lần đăng nhập tiếp theo như đã đề cập ở trên.

- Cuối cùng, email và username được lấy từ API /me của OAuth provider và token chính là access token mà ứng dụng đã lấy được từ OAuth server (bước trước), qua đây nó sẽ sẻ dụng POST request để gửi những dữ liệu này nên server cho những lần đăng nhập tiếp theo và sau đó gán cho người dùng một session cookie (để ý phần Set-Cookie trong phản hồi):

  <img width="1866" height="606" alt="image" src="https://github.com/user-attachments/assets/491a16a5-edc4-4cd9-ae19-8338459af3da" />

  Sau đó redirect về trang chính /.

- Tuy nhiên như đã trình bày ở trên, server tin ngay giá trị thông tin user như email trong body request này là thật, không có bất cứ gì để kiểm định lại. Do đó nếu ta dùng Burp Repeater gửi lại request POST /authenticate, sau đó sửa giá trị "email" thành ***carlos@carlos-montoya.net*** và gửi lại request, lúc này server set cookie session cho Carlos. Sau đó, thực hiện chọn "Request in browser" > "In original session" để lấy URL này:

  <img width="1432" height="1001" alt="image" src="https://github.com/user-attachments/assets/6cf0462c-ba04-4d09-a742-45e39f80f630" />

  <img width="1919" height="885" alt="image" src="https://github.com/user-attachments/assets/1ca02820-3302-421e-9a0a-f6dda72e187f" />

  &rarr; Đăng nhập thành công tài khoản *carlos*

### Flawed CSRF protection

- Nhiều thành phần trong luồng OAuth là tùy chọn, tuy nhiên một số được khuyến nghị sử dụng mạnh mẽ trừ khi có lý do quan trọng để không sử dụng chúng.

- Một ví dụ là tham số state trong endpoint /authenticate (/auth,...). Giá trị này được sử dụng để truyền qua lại giữa client application và dịch vụ OAuth như một dạng mã thông báo CSRF cho client application. Do đó nếu request xác thực không có tham số state kèm theo, thì rất có thể có tiềm năng để khai thác, nghĩa là ta có thể tự khởi động một luồng OAuth trước khi lừa trình duyệt của người dùng hoàn tất nó, tương tự như một cuộc tấn công CSRF truyền thống.

- Nếu trong một trang web mà nó cho phép người dùng đăng nhập bằng cơ chế cổ điển dựa trên mật khẩu hoặc bằng cách liên kết tài khoản social từ OAuth. Trong trường hợp này, nếu ứng dụng không sử dụng tham số state, ta có thể có khả năng chiếm đoạt tài khoản của người dùng nạn nhân bằng cách liên kết nó với tài khoản mạng xã hội của chính mình. Lưu ý là nếu trang web cho phép người dùng đăng nhập chỉ qua OAuth, thì tham số state có thể không quan trọng lắm. Tuy nhiên, không sử dụng tham số state vẫn có thể cho phép ta tạo ra các cuộc tấn công CSRF trong đăng nhập, trong đó người dùng bị lừa để đăng nhập vào tài khoản của kẻ tấn công.

### Lab: Forced OAuth profile linking

- Giao diện trang web ta thực hiện khai thác:

  <img width="1919" height="944" alt="image" src="https://github.com/user-attachments/assets/e5bb4365-3be3-4b26-a0fc-eb79a6de0d76" />

- Ta thấy giao diện login bao gốm hai mục là: nhập username password thông thường và sử dụng liên kết với social media.

- Sử dụng thông tin đăng nhập thông thường để login: ***wiener:peter***
  
  <img width="1919" height="878" alt="image" src="https://github.com/user-attachments/assets/0776a110-8373-4ada-b9e5-ba9c9ed77f62" />

- Khi đăng nhập thành công, giao diện sẽ hiện thông tin tài khoản và một phần là ***Attach a social profile***, ta thực hiện liên kết với tài khoản social profile là: wiener.peter:hotdog, giao diện khi thực hiện liên kết thành công:

  <img width="1919" height="883" alt="image" src="https://github.com/user-attachments/assets/5e439155-a334-4d8d-98fe-06b3379a6a08" />

- Thực hiện đăng xuất và đăng nhập tùy chọn "log in with social media", nhận thấy rằng trang thực hiện đăng nhập ngay lập tức qua tài khoản mạng xã hội mới liên kết của mình đó là: wiener.peter

- Để ý trong HTTP proxy của Burp, ta để ý thấy trong GET request /auth?client_id[...] của "attach a social media", thấy redirect_uri cho chức năng này gửi mã xác thực đến /oauth-linking. Quan trọng là request không bao gồm tham số state để bảo vệ chống lại các cuộc tấn công CSRF.

  <img width="1436" height="425" alt="image" src="https://github.com/user-attachments/assets/72f6bafe-9e64-4a25-80c5-7a56de74baec" />

  <img width="1874" height="597" alt="image" src="https://github.com/user-attachments/assets/c817e99a-3e7c-49f0-9f39-c7f24402b771" />

- Trong Burp, thực hiện bật proxy interception, sau đó trên trang web thực hiện chọn lại "Attach a social media", quay lại proxy interception của Burp, thực hiện forward tất cả các request cho đến khi tới request: ***GET /oauth-linking?code=[...]***:

  <img width="1919" height="1009" alt="image" src="https://github.com/user-attachments/assets/42a74dd7-20c3-45a5-bc28-b1f0716450da" />

  Thực hiện chuột phải và chọn Copy URL, sau đó drop request, điều này đồng nghĩa với việc ***code*** không được sử dụng nhưng nó vẫn còn giá trị.

  Tắt proxy interception và thực hiện log out tài khoản.

- Trong exploit server, tạo một thẻ iframe trong đó thuộc tính src trỏ tới URL oauth-linking ... mà ta vừa copy:

  <img width="1919" height="942" alt="image" src="https://github.com/user-attachments/assets/c687cc0c-07bf-4253-b8fc-abe6fa35489d" />

- Thực hiện deliver exploit tới cho victim, khi trình duyệt của họ load iframe, nó sẽ hoàn thành luồng OAuth sử dụng tài khoản social media của mình và liên kết nó tới tài khoản admin của blog website:
  
  <img width="1919" height="929" alt="image" src="https://github.com/user-attachments/assets/e3923c50-5af4-4227-bc92-d82d3a14d041" />

  <img width="1919" height="941" alt="image" src="https://github.com/user-attachments/assets/5007e5d3-47c2-430c-b6bb-354f63662512" />

  <img width="1919" height="877" alt="image" src="https://github.com/user-attachments/assets/b1489c8c-5f96-420c-b870-165cc15ecd06" />

  &rarr; Thành công xóa một user.



  







