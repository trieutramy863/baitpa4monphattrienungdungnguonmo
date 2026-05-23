# baitpa4monphattrienungdungnguonmo
# Đề bài 
SỬ DỤNG KẾT QUẢ ĐÃ LÀM Ở BÀI TẬP 3, BỔ SUNG VÀO DOCKER COMPOSE ĐỂ CÓ THÊM SERVICE 8N8:
SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file docker-compose.yml chứa:
Mariadb: sử dụng image: mariadb:latest để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh", MARIADB_ROOT_PASSWORD, MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)
Phpmyadmin: sử dụng image: phpmyadmin:latest để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết), khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1
WordPress: sử dụng image: wordpress:latest, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin, khai báo biến môi trường: WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã khai báo)
Cloudflared: sử dụng image: cloudflare/cloudflared:latest , full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker compose
N8n : sử dụng image: n8nio/n8n:latest, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )
Yêu cầu: sau khi có 5 service này trong file docker-compose.yml :
pull các images về và chạy chúng (up -d)
Kiểm tra các service đã running ok (ko bị restart liên tục)
Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)
Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)
Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)
Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào!
Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)
Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp
Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...
Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn Phát triển ứng dụng với mã nguồn mở
Truy cập sub-domain3 để cấu hình n8n:
tạo tài khoản admin : nhớ điền đúng email
Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY
Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.
Create workflow (home page => overview => Create workflow)
Add trigger node: tìm node: Telegram => OnMessage ; cấu hình Credential: Set up Credential => cần Nhập Access Token
Access Token thì lấy ở Telegram qua việc chát với @BotFather
Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp
Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)
Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY
Lấy API KEY tại trang: https://aistudio.google.com => https://aistudio.google.com/api-keys
cần tạo project mới, sẽ lấy được API KEY
Nhập API Key lên giao diện n8n
kéo thả nội dung đã chát với bot của telegram (phía bên trái) vào nội dung phần PROMPT kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)
Turn on Output Content as JSON : để kết quả trả về dạng json
Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?
Add (nối tiếp vào sau node Message a model) node: Code in JavaScript
Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
  title: cleanData.post_title,
  content: cleanData.post_content
};
Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post

Set up Credential: vào wp tại url: https://sub-domain1/wp-admin => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential
Wordpress URL: điền giá trị https://sub-domain1/ (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)
Ignore SSL Issues (Insecure): TURN ON
Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content
Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)
PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger

Kết quả cuối cùng cần đặt được:

từ điện thoại, chát với telegram bot
nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.
f5 wordpress để thấy bài viết mới đã lên sóng.
Chụp ảnh quá trình thao tác/cấu hình/các kết quả trung gian đạt được

Nhận xét thành quả đạt được!!!

demo kết quả cuối cùng:

chát với bot:
# Bài Làm

- TẠO THƯ MỤC
  <img width="884" height="338" alt="Screenshot 2026-05-22 162231" src="https://github.com/user-attachments/assets/33054803-033e-417e-b718-58e0853314b3" />

- Tạo file:
``` text
nano docker-compose.yml
```
<img width="1910" height="1117" alt="Screenshot 2026-05-22 161628" src="https://github.com/user-attachments/assets/485c687e-511d-4c76-bf4a-f54f7fa4b66d" />

<img width="1013" height="867" alt="Screenshot 2026-05-22 215836" src="https://github.com/user-attachments/assets/6f0cfff5-dce8-4a98-9691-078508cb4941" />

- CHẠY DOCKER
```text
   chạy container
     docker compose pull
     docker compose up -d
  kiểm tra
     docker ps
```
<img width="1634" height="581" alt="Screenshot 2026-05-22 173527" src="https://github.com/user-attachments/assets/a9fb2578-9e08-4001-8350-1334c02d7876" />

Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)

<img width="1920" height="1200" alt="Screenshot 2026-05-22 181638" src="https://github.com/user-attachments/assets/76a006d2-d2d5-4cd6-89e0-1914e3573cda" />

Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)

<img width="1920" height="1200" alt="Screenshot 2026-05-22 181754" src="https://github.com/user-attachments/assets/ae7e6449-881f-4f75-9552-3aa54a18a0c4" />

Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)

<img width="1920" height="1200" alt="Screenshot 2026-05-22 181851" src="https://github.com/user-attachments/assets/98cfcb0c-4d69-494d-b8ef-f1c0676dc0cd" />

Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào

- https://pma.trieutramy04.id.vn
  
<img width="1920" height="1200" alt="Screenshot 2026-05-22 182536" src="https://github.com/user-attachments/assets/57e795fc-9a4d-42e5-a684-8195a7eeb3a5" />

Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)

- https://wp.trieutramy04.id.vn

<img width="1920" height="1200" alt="Screenshot 2026-05-22 182144" src="https://github.com/user-attachments/assets/47adaf8a-3e69-499c-b683-0d8cb1e93e21" />

<img width="1920" height="1200" alt="Screenshot 2026-05-22 182324" src="https://github.com/user-attachments/assets/73461d40-8b4b-4c89-bbc9-b53352b35dd1" />

Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp

- https://pma.trieutramy04.id.vn

<img width="1920" height="1200" alt="Screenshot 2026-05-22 182601" src="https://github.com/user-attachments/assets/903ba073-0d25-49be-a945-b368bcffc02c" />

Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...

<img width="1920" height="1200" alt="Screenshot 2026-05-22 183113" src="https://github.com/user-attachments/assets/b405c0e0-a75e-480b-9807-249a2c88763e" />

<img width="1920" height="1200" alt="Screenshot 2026-05-22 183132" src="https://github.com/user-attachments/assets/be0fd03b-a785-41ed-995d-c3a7ba1446ca" />

Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn Phát triển ứng dụng với mã nguồn mở

<img width="1920" height="1200" alt="Screenshot 2026-05-22 183303" src="https://github.com/user-attachments/assets/e8b09e7c-9a82-4cb1-b436-927f1da5ba99" />

Truy cập sub-domain3 để cấu hình n8n

https://n8n.trieutramy04.id.vn

tạo tài khoản admin : nhớ điền đúng email

<img width="950" height="1199" alt="Screenshot 2026-05-22 183509" src="https://github.com/user-attachments/assets/05c2f316-d8e8-434f-9466-4d7663ac2af9" />

Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY

<img width="828" height="1792" alt="image" src="https://github.com/user-attachments/assets/fa12b6b1-54ed-4bfb-b43c-4f603466b076" />

Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.

<img width="1920" height="1200" alt="Screenshot 2026-05-22 183811" src="https://github.com/user-attachments/assets/a4cceb38-2daa-4614-9b6d-44d3fdd005b7" />

<img width="1920" height="1200" alt="Screenshot 2026-05-22 183834" src="https://github.com/user-attachments/assets/c6560770-6e0b-496c-8946-537e257349fc" />

Create workflow (home page => overview => Create workflow)

Add trigger node: tìm node: Telegram => OnMessage ; cấu hình Credential: Set up Credential => cần Nhập Access Token

<img width="1441" height="847" alt="Screenshot 2026-05-22 190919" src="https://github.com/user-attachments/assets/f7d10ec1-2736-42a3-b64e-af61c1b8345b" />

Access Token thì lấy ở Telegram qua việc chát với @BotFather

<img width="828" height="1792" alt="image" src="https://github.com/user-attachments/assets/aaf29916-5987-4613-9114-da584c8296a8" />

Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp

<img width="828" height="1792" alt="image" src="https://github.com/user-attachments/assets/025345ec-e453-40ad-8a2d-bba4c389b500" />

Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)

<img width="1920" height="1200" alt="Screenshot 2026-05-22 222252" src="https://github.com/user-attachments/assets/90bdb70b-5efc-4288-9b6f-b11c16b02f60" />

Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY

<img width="1372" height="822" alt="Screenshot 2026-05-22 222842" src="https://github.com/user-attachments/assets/ed40e497-5aea-4b8d-9e23-c23e5f53db81" />

Lấy API KEY tại trang: https://aistudio.google.com => https://aistudio.google.com/api-keys

Cần tạo project mới, sẽ lấy được API KEY

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/96ee9be0-7110-407c-9a8b-f5a128d8c74a" />

Nhập API Key lên giao diện n8n

<img width="1372" height="822" alt="Screenshot 2026-05-22 222842" src="https://github.com/user-attachments/assets/745f9476-13a0-4325-973f-b93c05ca72b2" />

- kéo thả nội dung đã chát với bot của telegram (phía bên trái) vào nội dung phần PROMPT kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)

- Turn on Output Content as JSON : để kết quả trả về dạng json

- Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?

- Bấm nút Execute step và chờ Gemini xử lý. Khi thành công, ở cột Output bên phải sẽ thấy AI trả về một chuỗi text thô lộn xộn nhưng chứa cấu trúc post_title và post_content.

<img width="1920" height="1200" alt="Screenshot 2026-05-22 224019" src="https://github.com/user-attachments/assets/d9d62e50-91ce-4a01-8105-ac7c6ff29836" />

- Add (nối tiếp vào sau node Message a model) node: Code in JavaScript

  + Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.

```text
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
 title: cleanData.post_title,
 content: cleanData.post_content
};
```
<img width="1920" height="1200" alt="Screenshot 2026-05-22 224230" src="https://github.com/user-attachments/assets/5155ee55-212d-462f-aa24-b551fd972346" />

- Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post

<img width="614" height="969" alt="Screenshot 2026-05-22 224309" src="https://github.com/user-attachments/assets/ca1dcaec-81a1-4678-a172-66ce220f3050" />

<img width="1886" height="1022" alt="Screenshot 2026-05-22 224504" src="https://github.com/user-attachments/assets/41e5c28a-446f-421c-93c9-e38e905ea16d" />

Set up Credential: vào wp tại url: https://wp.trieutramy04.id.vn/wp-admin => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential

Wordpress URL: điền giá trị https://wp.trieutramy04.id.vn/ (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)

Ignore SSL Issues (Insecure): TURN ON

<img width="1329" height="814" alt="image" src="https://github.com/user-attachments/assets/8102fb2e-ecd8-466c-9fa4-f6622d3bfeb2" />

Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content

<img width="1920" height="1200" alt="Screenshot 2026-05-22 224928" src="https://github.com/user-attachments/assets/f01ea154-2e02-4950-b4d8-65280fa454e3" />
Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)

<img width="1920" height="1200" alt="Screenshot 2026-05-22 225021" src="https://github.com/user-attachments/assets/3a5db863-8179-475f-95f1-1eda91c955a1" />

PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger

<img width="1920" height="1200" alt="Screenshot 2026-05-23 103631" src="https://github.com/user-attachments/assets/254b3a33-94f7-449f-a1a0-34e91405894b" />
