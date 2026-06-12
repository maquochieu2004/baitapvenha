# BÀI TẬP 04

## TỰ ĐỘNG HÓA QUÁ TRÌNH ĐĂNG BÀI WORDPRESS BẰNG N8N

**Học phần:** Phát triển ứng dụng với mã nguồn mở (TEE0421)
**Lớp:** 58KTPM
**Hạn nộp:** 23h59 ngày 25/05/2026

---

## Mục tiêu

Tiếp tục sử dụng hệ thống đã triển khai ở Bài tập 03 và mở rộng thêm dịch vụ N8N trong môi trường Docker Compose. Hệ thống hoàn chỉnh phải bao gồm:

* MariaDB
* phpMyAdmin
* WordPress
* Cloudflared
* N8N

Sau khi hoàn thành, người dùng có thể gửi nội dung qua Telegram, N8N sẽ sử dụng Google Gemini để tạo nội dung HTML và tự động xuất bản bài viết lên WordPress.

---

## Yêu cầu triển khai Docker Compose

Tạo file `docker-compose.yml` trên Ubuntu với các dịch vụ sau:

### 1. MariaDB

Sử dụng image:

```yaml
mariadb:latest
```

Khai báo các biến môi trường:

* TZ=Asia/Ho_Chi_Minh
* MARIADB_ROOT_PASSWORD
* MARIADB_DATABASE
* MARIADB_USER
* MARIADB_PASSWORD

Giá trị được tự lựa chọn.

---

### 2. phpMyAdmin

Sử dụng image:

```yaml
phpmyadmin:latest
```

Khai báo:

* PMA_HOST=<tên dịch vụ MariaDB>
* PMA_ARBITRARY=1

Mục đích chính là quản lý và kiểm tra cơ sở dữ liệu. Việc tạo bảng sẽ do WordPress tự thực hiện trong quá trình cài đặt.

---

### 3. WordPress

Sử dụng image:

```yaml
wordpress:latest
```

Thiết lập kết nối tới MariaDB bằng các biến:

* WORDPRESS_DB_HOST
* WORDPRESS_DB_NAME
* WORDPRESS_DB_USER
* WORDPRESS_DB_PASSWORD

Thông tin phải đồng nhất với phần cấu hình MariaDB.

---

### 4. Cloudflared

Sử dụng image:

```yaml
cloudflare/cloudflared:latest
```

Cấu hình command và Tunnel Token lấy từ Cloudflare Dashboard, sau đó chuyển đổi sang định dạng Docker Compose.

---

### 5. N8N

Sử dụng image:

```yaml
n8nio/n8n:latest
```

Thiết lập biến môi trường:

```text
WEBHOOK_URL=https://subdomain-n8n.domain.com/
```

Tên miền phải trùng với hostname đã khai báo trong Cloudflare Tunnel.

---

## Khởi động hệ thống

Sau khi hoàn thiện file cấu hình:

1. Tải toàn bộ image cần thiết.
2. Chạy hệ thống bằng Docker Compose.
3. Kiểm tra trạng thái container.
4. Đảm bảo các dịch vụ hoạt động ổn định và không xuất hiện tình trạng restart liên tục.

---

## Cấu hình Cloudflare Tunnel

Thiết lập các route công khai:

### Subdomain 1

Dùng để truy cập WordPress.

Ví dụ:

```text
wp.domain.com
```

### Subdomain 2

Dùng để truy cập phpMyAdmin.

Ví dụ:

```text
db.domain.com
```

### Subdomain 3

Dùng để truy cập N8N.

Ví dụ:

```text
n8n.domain.com
```

---

## Kiểm tra cơ sở dữ liệu

### Bước 1

Truy cập phpMyAdmin thông qua Subdomain 2.

Xác nhận cơ sở dữ liệu WordPress chưa có bảng nào được tạo.

### Bước 2

Truy cập WordPress thông qua Subdomain 1 và thực hiện cài đặt theo hướng dẫn.

### Bước 3

Quay lại phpMyAdmin để kiểm tra.

Xác nhận WordPress đã tự động sinh các bảng dữ liệu cần thiết.

---

## Tạo nội dung trên WordPress

Tạo tối thiểu hai bài viết:

### Bài viết số 1

Giới thiệu bản thân:

* Thông tin cá nhân
* Sở thích
* Kỹ năng
* Thành tích
* Có thể bổ sung hình ảnh, âm thanh hoặc video

### Bài viết số 2

Trình bày các kiến thức đã tiếp thu trong học phần Phát triển ứng dụng với mã nguồn mở.

---

## Thiết lập N8N

### Tạo tài khoản

Truy cập Subdomain 3.

Tạo tài khoản quản trị và sử dụng email hợp lệ.

---

### Kích hoạt License

Sau khi đăng ký:

1. Nhận License Key qua email.
2. Mở:

Settings → Usage & Plan

3. Nhập License Key.
4. Thực hiện kích hoạt.
5. Kiểm tra thông báo xác nhận thành công.

---

## Xây dựng Workflow

### Telegram Trigger

Thêm node:

```text
Telegram Trigger (On Message)
```

Tạo bot mới thông qua BotFather và lấy Access Token.

Sau khi tạo bot:

* Sao chép Token
* Nhắn ít nhất một tin nhắn tới bot

Bước này là bắt buộc.

---

### Google Gemini

Thêm node:

```text
Google Gemini - Message a Model
```

Tạo API Key tại:

https://aistudio.google.com/api-keys

Sau đó cấu hình API Key trong N8N.

Ví dụ Prompt:

```text
{{ $json.message.text }}

Kết quả trả về ở định dạng HTML + CSS để sử dụng trực tiếp cho bài viết WordPress.
```

Bật tùy chọn:

```text
Output Content as JSON
```

Có thể thử nghiệm thêm:

* System Message
* Temperature
* Max Output Tokens

để cải thiện chất lượng nội dung.

---

### JavaScript Code Node

Kết nối sau Gemini.

Ví dụ xử lý dữ liệu:

```javascript
const rawText = $input.first().json.content.parts[0].text;

const cleanData = JSON.parse(rawText);

return {
  title: cleanData.post_title,
  content: cleanData.post_content
};
```

Node này có nhiệm vụ tách riêng tiêu đề và nội dung từ phản hồi JSON.

---

### WordPress Create Post

Thêm node:

```text
WordPress → Create a Post
```

#### Tạo Application Password

Đăng nhập:

```text
https://subdomain1/wp-admin
```

Vào:

Users → Profile → Application Passwords

Tạo mật khẩu mới và sao chép chuỗi ký tự được cấp.

#### Thiết lập Credential

* URL WordPress: https://subdomain1/
* Username: tài khoản WordPress
* Password: Application Password
* Ignore SSL Issues: Enabled

#### Cấu hình bài viết

Liên kết:

* title → tiêu đề
* content → nội dung

Thêm thuộc tính:

```text
Status = Publish
```

để bài viết được xuất bản ngay sau khi tạo.

---

## Kích hoạt Workflow

Nhấn nút:

```text
Publish
```

Sau khi Publish, Workflow sẽ tự động chạy khi có tin nhắn mới từ Telegram.

---

## Kết quả mong đợi

Quy trình hoạt động như sau:

1. Người dùng gửi tin nhắn cho Telegram Bot.
2. Telegram Trigger tiếp nhận nội dung.
3. Gemini sinh nội dung HTML/CSS theo yêu cầu.
4. JavaScript Node tách dữ liệu JSON.
5. WordPress Node tự động đăng bài.
6. Làm mới trang WordPress để kiểm tra bài viết mới.

---

## Minh chứng cần nộp

Báo cáo phải có hình ảnh thể hiện:

* Cấu hình Docker Compose
* Các container đang hoạt động
* Cloudflare Tunnel
* phpMyAdmin trước và sau khi cài đặt WordPress
* Workflow N8N
* Cấu hình Telegram
* Cấu hình Gemini
* Cấu hình WordPress Node
* Kết quả đăng bài tự động

---

## Đánh giá kết quả

Trình bày nhận xét cá nhân về:

* Khả năng tự động hóa của N8N
* Sự tích hợp giữa Telegram, Gemini và WordPress
* Những khó khăn gặp phải
* Kiến thức và kinh nghiệm thu được trong quá trình thực hiện
# KẾT QUẢ DEMO CUỐI CÙNG

## 1. Gửi yêu cầu từ Telegram Bot

Người dùng gửi nội dung yêu cầu tới Telegram Bot đã được tạo thông qua BotFather.

Ví dụ nội dung gửi:

> Viết bài 600 chữ về chủ đề: Hè này đi uống beer với bạn thì tuyệt vời, nâng cao tinh thần đoàn kết, nâng cao sức khỏe.

Sau khi nhận được tin nhắn, Telegram Trigger trong N8N sẽ tự động kích hoạt quy trình xử lý.

**Hình 1. Tin nhắn gửi tới Telegram Bot**

<img width="1363" height="394" alt="f2e79d1a-3e80-4c72-8212-8d4f8a434140(1)(1)" src="https://github.com/user-attachments/assets/f7218893-f6eb-4532-bd56-896f050cb9bb" />


---

## 2. Luồng xử lý tự động trên N8N

Workflow được xây dựng gồm các thành phần:

* Telegram Trigger
* Google Gemini – Message a Model
* Code in JavaScript
* WordPress – Create a Post

Quá trình hoạt động:

1. Telegram Trigger nhận nội dung từ người dùng.
2. Google Gemini tiếp nhận yêu cầu và sinh nội dung bài viết ở định dạng JSON.
3. Node JavaScript tách dữ liệu thành tiêu đề và nội dung bài viết.
4. Node WordPress tự động tạo và xuất bản bài đăng mới.

Toàn bộ quy trình diễn ra tự động, không cần thao tác thủ công.

**Hình 2. Workflow tự động trên N8N**

<img width="1363" height="344" alt="f2e79d1a-3e80-4c72-8212-8d4f8a434140(1)(1)" src="https://github.com/user-attachments/assets/602caca5-b5cb-408f-9e20-4bcf46888b67" />


---

## 3. Kết quả bài viết trên WordPress

Sau khi workflow thực thi thành công, WordPress tự động tạo một bài đăng mới với:

* Tiêu đề được sinh bởi Gemini.
* Nội dung bài viết định dạng HTML.
* Trạng thái bài viết là Publish.
* Có thể truy cập trực tiếp trên website WordPress.

Ví dụ kết quả:

**Tiêu đề:** Hè Này Đi Uống Beer Với Bạn: Sự Kết Hợp Hoàn Hảo Giữa Gắn Kết Và Sức Khỏe

Bài viết đã được xuất bản thành công trên website và hiển thị đầy đủ nội dung do AI tạo ra.

**Hình 3. Bài viết tự động được đăng trên WordPress**

<img width="1363" height="422" alt="f2e79d1a-3e80-4c72-8212-8d4f8a434140(1)(1)" src="https://github.com/user-attachments/assets/34f87d8a-dd1b-4317-8143-9fee08977e6f" />


---

## 4. Đánh giá kết quả đạt được

Qua bài thực hành này, em đã triển khai thành công hệ thống tự động đăng bài lên WordPress bằng N8N. Workflow đã kết nối và hoạt động ổn định giữa Telegram, Google Gemini và WordPress.

Một số kiến thức và kỹ năng đạt được:

* Sử dụng Docker Compose để triển khai nhiều dịch vụ cùng lúc.
* Cấu hình Cloudflare Tunnel để công khai dịch vụ ra Internet.
* Quản lý cơ sở dữ liệu bằng MariaDB và phpMyAdmin.
* Cài đặt và vận hành WordPress.
* Xây dựng workflow tự động bằng N8N.
* Tích hợp Telegram Bot với AI Gemini.
* Tự động hóa việc tạo và xuất bản nội dung trên WordPress.

Kết quả cuối cùng cho thấy hệ thống có thể nhận yêu cầu từ Telegram, sử dụng AI để sinh nội dung và tự động đăng bài lên WordPress mà không cần sự can thiệp của người dùng.

