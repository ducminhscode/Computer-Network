# 🛡️ Hướng Dẫn Cài Đặt và Bảo Vệ Web Server với ModSecurity

Hướng dẫn này cung cấp các bước chi tiết để cài đặt môi trường kiểm tra bảo mật web sử dụng **XAMPP** và **DVWA (Damn Vulnerable Web Application)**, triển khai **ModSecurity** để bảo vệ server khỏi các cuộc tấn công như **SQL Injection** và **XSS**, cùng với hướng dẫn thực hiện các cuộc tấn công để kiểm tra hiệu quả bảo vệ.

---

## 🚀 Tính năng chính

- 🛠️ **Cài đặt XAMPP**: Thiết lập môi trường phát triển web với Apache và MySQL.  
- 📂 **Triển khai DVWA**: Tạo một ứng dụng web dễ bị tấn công để kiểm tra bảo mật.  
- 🛡️ **Bảo vệ với ModSecurity**: Cấu hình tường lửa ứng dụng web (WAF) để chặn các cuộc tấn công SQL Injection và XSS.  
- 📊 **Kiểm tra tấn công**: Thực hiện các cuộc tấn công SQL Injection và XSS để đánh giá hiệu quả bảo vệ.  
- 🔄 **Cấu hình linh hoạt**: Hướng dẫn thay đổi port và thiết lập rules cho ModSecurity.  

---

## 🏗️ Kiến trúc hệ thống

- **Web Server**:  
  - [XAMPP](https://www.apachefriends.org/) (Apache, MySQL, PHP)  
  - [ModSecurity](https://www.modsecurity.org/) (Tường lửa ứng dụng web)  

- **Ứng dụng kiểm tra**:  
  - [DVWA](https://github.com/digininja/DVWA) (Damn Vulnerable Web Application)  

- **Cơ sở dữ liệu**:  
  - [MySQL](https://www.mysql.com/) (Quản lý qua phpMyAdmin)  

---

## 📋 Hướng dẫn cài đặt

### Bước 1: Cài đặt XAMPP
1. Tải XAMPP từ [trang chính thức](https://www.apachefriends.org/download.html), chọn phiên bản phù hợp với hệ điều hành.
2. Mở file cài đặt, chọn đường dẫn lưu trữ (mặc định: `C:\xampp`), sau đó nhấn **Next** để hoàn tất.
3. Đổi port mặc định của Apache để tránh xung đột:
   - Mở **XAMPP Control Panel**, chọn **Config** của Apache, sau đó chọn **Apache (httpd.conf)**.
   - Sử dụng Notepad, nhấn **Ctrl + H**, thay thế tất cả số `80` bằng `8080` (hoặc một port khác), sau đó lưu.
   - Tương tự, mở **Apache (httpd-ssl.conf)**, thay thế số `443` bằng `4443` (hoặc một port khác), sau đó lưu.

### Bước 2: Cài đặt DVWA
1. Tải DVWA từ [GitHub](https://github.com/digininja/DVWA) và giải nén.
2. Tạo thư mục `DVWA` trong `C:\xampp\htdocs` và sao chép toàn bộ nội dung giải nén vào đây.
3. Đổi tên file `C:\xampp\htdocs\DVWA\config\config.inc.php.dist` thành `config.inc.php`.

### Bước 3: Khởi động và cấu hình cơ sở dữ liệu
1. Mở **XAMPP Control Panel**, khởi động **Apache** và **MySQL**.
2. Truy cập `http://localhost:8080/phpmyadmin/` trên trình duyệt.
3. Tạo một database mới, vào tab **Privileges**, chọn **Add user account**, thiết lập username và password (giống thông tin trong `config.inc.php`).
4. Truy cập `http://localhost:8080/dvwa/setup.php`, nhấn **Create/Reset Database** để khởi tạo cơ sở dữ liệu.
5. Đăng nhập DVWA với tài khoản `admin/password` để kiểm tra giao diện.

### Bước 4: Cài đặt ModSecurity
1. Tải ModSecurity từ [Apache Lounge](https://www.apachelounge.com/download/).
2. Giải nén và sao chép:
   - File `mod_security2.so` vào `C:\xampp\apache\modules`.
   - File `yail.dll` vào `C:\xampp\apache\bin`.
3. Mở file `C:\xampp\apache\conf\httpd.conf` và thêm các dòng sau:
   ```
   LoadModule security2_module modules/mod_security2.so
   LoadModule unique_id_module modules/mod_unique_id.so
   SecRuleEngine DetectionOnly

   <IfModule security2_module>
       SecRuleEngine On
       SecDefaultAction "phase:2,deny,log,status:403"
   </IfModule>
   ```

### Bước 5: Thiết lập rules cho ModSecurity
1. Tạo hai file `sqlinjection.conf` và `xss.conf` trong `C:\xampp\apache\conf`.
2. Tải nội dung rules từ:
   - [SQL Injection Rules](https://raw.githubusercontent.com/SEC642/modsec/master/rules/base_rules/modsecurity_crs_41_sql_injection_attacks.conf) và sao chép vào `sqlinjection.conf`.
   - [XSS Rules](https://raw.githubusercontent.com/SEC642/modsec/master/rules/base_rules/modsecurity_crs_41_xss_attacks.conf) và sao chép vào `xss.conf`.
3. Mở lại `httpd.conf` và thêm:
   ```
   Include conf/sqlinjection.conf
   Include conf/xss.conf
   ```
4. Khởi động lại **XAMPP Control Panel**.

---

## 🧪 Kiểm tra tấn công

### 1. Tấn công SQL Injection
- Truy cập DVWA, đặt mức bảo mật (**Security Level**) về **Low**.
- Trong mục **SQL Injection**, nhập `%’ or ‘0’=’0` và nhấn **Submit**.
- **Kết quả không bảo vệ (trước B4)**: Hiển thị toàn bộ thông tin users trong cơ sở dữ liệu.
- **Kết quả sau khi cài ModSecurity**: Tấn công bị chặn, trả về lỗi 403.

### 2. Tấn công XSS
- **XSS (DOM)**:
  - Truy cập mục **XSS (DOM)**, nhập:
    ```html
    </select><img src=x onerror=alert("Hi!Attacker")>
    ```
    hoặc
    ```html
    </option></select><img src=x onerror=alert("Hello")>
    ```
- **XSS (Reflected)**:
  - Truy cập mục **XSS (Reflected)**, nhập:
    ```html
    <script>alert("Hello_Attacker")</script>
    ```
    hoặc
    ```html
    <img src=x onerror=alert("Good_Morning_Sir")>
    ```
- **Kết quả không bảo vệ**: Các đoạn mã chạy và hiển thị thông báo.
- **Kết quả sau khi cài ModSecurity**: Tấn công bị chặn, trả về lỗi 403.

---

## ✅ Kết luận
Sau khi hoàn thành các bước từ B1 đến B5, server sẽ được bảo vệ bởi ModSecurity, chặn hiệu quả các cuộc tấn công SQL Injection và XSS. Có thể kiểm tra thêm ở các mức bảo mật khác nhau của DVWA để đánh giá hiệu quả.
