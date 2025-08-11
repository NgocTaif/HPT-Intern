# LỖ HỔNG OAuth 2.0 authentication (PORTSWIGGER)

---

Khi thực hiện lướt web, chúng ta gần như chắc chắn đã bắt gặp các trang web cho phép đăng nhập bằng tài khoản mạng xã hội của mình như: facebook, google, ... Rất có thể tính năng này được xây dựng bằng cách sử dụng framework OAuth 2.0 phổ biến. OAuth 2.0 rất dễ bị để ý với những kẻ tấn công vì nó cực kỳ phổ biến và vốn dĩ dễ mắc lỗi triển khai. Dưới đây ta sẽ tìm hiểu về lỗ hổng trong OAuth authentication.

## 1. OAuth là gì?

- OAuth là một khung ủy quyền thường được sử dụng cho phép các trang web và ứng dụng web yêu cầu quyền truy cập hạn chế/nhất định vào tài khoản của người dùng trên một ứng dụng khác. Điều quan trọng là OAuth cho phép người dùng cấp quyền truy cập này mà không tiết lộ thông tin đăng nhập của họ cho ứng dụng yêu cầu.

- Ví dụ, trên một website là: abc.com, thay vì tạo một tài khoản đăng nhập mới ta có thể chọn mục "Đăng nhập bằng Google", lúc này trang web sẽ phải thực hiện xin cấp quyền từ Google để có thể đăng nhập vào trang web abc.com

## 2. Cách thức hoạt động của OAuth 2.0

- Như đã nói, bản chất của OAuth 2.0 là cho phép ứng dụng bên thứ ba truy cập một phần dữ liệu của người dùng từ một dịch vụ khác, mà không cần chia sẻ mật khẩu của người dùng.

- Việc này xung quanh ba chủ thể:

  - Client Application (Ứng dụng khách): Ứng dụng hoặc website muốn truy cập dữ liệu người dùng.

  - Resource Owner (Chủ sở hữu tài nguyên): Chính là người dùng – người sở hữu dữ liệu. Ví dụ: người có tài khoản Google.

  - OAuth Service Provider (Nhà cung cấp dịch vụ OAuth): Nơi lưu dữ liệu và cho phép truy cập qua OAuth API. Ví dụ: Google, Facebook, GitHub... Gồm 2 thành phần chính:

      - Authorization Server – xử lý việc đăng nhập, cấp quyền, phát access token.

      - Resource Server – API chứa dữ liệu thật, chỉ trả dữ liệu khi có token hợp lệ.

- Tập trung vào hai dạng grant phổ biển nhất là: Authorization Code Flow & Implicit Flow

- Cả hai dạng grant đều theo 4 dạng cơ bản như sau:

<img width="1189" height="1200" alt="image" src="https://github.com/user-attachments/assets/6f29f687-17fc-43a4-9345-787c8c8d2600" />

- Ví dụ: ta vào trang web: hpt.com và chọn "Đăng nhập bằng Google"

  - Đầu tiên, hpt.com gửi request đến Google OAuth:
 
    ```http
    https://accounts.google.com/o/oauth2/v2/auth?client_id=abc123&redirect_uri=https://englishmaster.com/callback&response_type=cod&scope=email
    ```

  - Tiếp đó, Google hỏi: “Cho phép hpt.com xem email và hồ sơ của bạn?" và chọn Allow.
 
  - Google trả về:
 
    ```http
    https://englishmaster.com/callback?code=XYZ987
    ```

  - hpt.com gửi code này kèm client_secret lên Google để lấy:
 
    ```json
    {
      "access_token": "ya29.A0AfH6SMB...",
      "expires_in": 3600,
      "scope": "email"
    }
    ```

  - Web gọi API Google dung access_token để lấy thông tin user.
 
    ```json
    {
      "id": "123456789",
      "email": "user@gmail.com",
      "name": "Nguyễn Văn A",
      "picture": "https://lh3.googleusercontent.com/a-/abc123"
    }
    ```

    &rarr; Đăng nhập thành công.

## 3. OAuth Authentication

- Mặc dù ban đầu không nhằm mục đích này, OAuth cũng đã phát triển thành một công cụ để xác thực người dùng.

- Bản chất là dùng chính quy trình OAuth để xác thực người dùng (cho phép đăng nhập), chỉ thay đổi mục tiêu từ “truy cập dữ liệu” → dùng chúng để “đăng nhập”.

- Ví dụ: Giả sử bạn đăng nhập vào shopabc.com bằng Google. Lúc này:

  - Bạn không cần tạo tài khoản mới, không cần nhớ mật khẩu cho shopabc.

  - Shopabc chỉ lưu thông tin do Google cung cấp (email, tên).

  - Lần sau bạn chọn "Sign in with Google", shopabc chỉ cần xác minh token hợp lệ là bạn được vào.
 
### Lab: Authentication bypass via OAuth implicit flow

- Giao diện trang web cho phép sử dụng mạng xã hội để đăng nhập:

  <img width="1919" height="943" alt="image" src="https://github.com/user-attachments/assets/63448710-444b-4990-9fc6-4ff666c776df" />

- Trong lab này, ta tiến hành đăng nhập với tài khoản social media có sẵn là: *wiener:peter*. Kết quả sau khi đăng nhập thành công:

  <img width="1919" height="880" alt="image" src="https://github.com/user-attachments/assets/5674761a-5561-4835-86a5-b2081efc2ed4" />

- Sau đó trang thực hiện redirect về trang chủ.

- Thực hiện kiểm tra Burp Suite để xem quy trình OAuth trên trang web. Và khi thực hiện nhập thành công tài khoản mạng xã hội ta thấy nó bắt đầu bằng một request xác thực *GET /auth?client_id=[...]* như sau:

  <img width="1431" height="516" alt="image" src="https://github.com/user-attachments/assets/157bdbc4-6c3e-4236-a179-b5c197278ad7" />

  Trong request trên, ta sẽ thấy:

  *client_id=ab8tzl43phra0tq2jlil* là	mã định danh của ứng dụng (client app) đã được đăng ký với OAuth provider.

  *redirect_uri=https://0a6f00d403a19fc80793a4005200e9.web-security-academy.net/oauth-callback*:	Địa chỉ callback mà Authorization Server sẽ redirect người dùng về sau khi xác thực xong.

  *response_type=token*: Đây là Implicit Flow (yêu cầu access token trực tiếp trong URL fragment thay vì code).

  *scope=openid&profile&email*:	Yêu cầu quyền truy cập thông tin OpenID, profile và email của user.

- Ta sẽ thấy OAuth Server sẽ lệnh redirect trình duyệt về redirect_uri mà client đã đăng ký (ở đây là /oauth-callback).

  <img width="1869" height="567" alt="image" src="https://github.com/user-attachments/assets/09a9a28a-bcae-4129-9f62-d8a61df498e6" />

  Fragment #access_token=... → là nơi token được trả về trực tiếp trên URL vì bạn đang dùng Implicit Flow (response_type=token ở request trước).

  





