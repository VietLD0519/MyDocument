## **1. Yêu cầu**
### **1.1. Môi trường server**
- Máy chủ Windows với **IIS 8.0+** đã cài đặt.
- **Node.js** phiên bản tương thích với Next.js (khuyến nghị: Node.js 18.x hoặc 20.x).
- **PM2** được cài đặt để quản lý tiến trình Node.js.
- Ứng dụng Next.js đã được build (`next build`).
- Địa chỉ server: `10.8.8.88:7777` (Next.js).
- IIS truy cập qua `http://10.8.8.88` (cổng 80 hoặc cổng khác).

### **1.2. Công cụ bổ sung**
- **IIS URL Rewrite Module**: Để xử lý chuyển tiếp yêu cầu.
- **Application Request Routing (ARR)**: Hỗ trợ reverse proxy.
- **Quyền quản trị**: Để cấu hình IIS, PM2, và phân quyền thư mục.

---

## **2. Các bước cấu hình**

### **2.1. Chuẩn bị ứng dụng Next.js**
1. **Build ứng dụng Next.js**:
   - Trong thư mục dự án Next.js, chạy:
     ```bash
     npm run build
     ```
   - Thư mục `.next` sẽ được tạo, chứa các tệp tối ưu hóa.

2. **Cài đặt PM2**:
   - Cài đặt PM2 toàn cục trên server:
     ```bash
     npm install -g pm2
     ```

3. **Sao chép dự án vào server**:
   - Sao chép thư mục dự án (bao gồm `.next`, `node_modules`, `package.json`) vào server, ví dụ: `C:\inetpub\wwwroot\my-next-app`.
   - Cài đặt dependencies:
     ```bash
     cd C:\inetpub\wwwroot\my-next-app
     npm install
     ```

4. **Chạy ứng dụng Next.js với PM2**:
   - Trong thư mục dự án, chạy:
     ```bash
     pm2 start npm --name "next-app" -- start -- -p 7777
     ```
   - Lưu cấu hình PM2:
     ```bash
     pm2 save
     ```
   - Cấu hình PM2 khởi động cùng hệ thống:
     ```bash
     pm2 startup
     ```
   - Kiểm tra ứng dụng chạy tại `http://10.8.8.88:7777` bằng trình duyệt hoặc `curl http://10.8.8.88:7777`.

### **2.2. Cài đặt và cấu hình IIS**
1. **Cài đặt IIS (nếu chưa có)**:
   - Mở **Server Manager** > **Add Roles and Features** > Chọn **Web Server (IIS)**.
   - Cài đặt các thành phần: **HTTP Redirection**, **Application Development** (nếu cần).

2. **Cài đặt URL Rewrite Module và ARR**:
   - Tải **URL Rewrite Module** từ: [https://www.iis.net/downloads/microsoft/url-rewrite](https://www.iis.net/downloads/microsoft/url-rewrite).
   - Tải **Application Request Routing (ARR)** từ: [https://www.iis.net/downloads/microsoft/application-request-routing](https://www.iis.net/downloads/microsoft/application-request-routing).
   - Cài đặt cả hai và xác nhận chúng xuất hiện trong **IIS Manager**.

3. **Tạo website trong IIS**:
   - Mở **IIS Manager**, nhấp chuột phải vào **Sites** > **Add Website**.
   - Cấu hình:
     - **Site name**: `NextJSApp`.
     - **Physical path**: Chọn thư mục trống, ví dụ: `C:\inetpub\wwwroot\nextjs`.
     - **Binding**:
       - **Type**: `http`.
       - **IP Address**: `10.8.8.88` hoặc `All Unassigned`.
       - **Port**: `80` (hoặc 8080 nếu cần).
       - **Host name**: Để trống hoặc nhập domain.
   - Nhấn **OK**.

### **2.3. Cấu hình Reverse Proxy**
1. **Thêm quy tắc URL Rewrite**:
   - Trong **IIS Manager**, chọn website `NextJSApp`.
   - Nhấp đúp vào **URL Rewrite** > **Add Rule(s)** > Chọn **Reverse Proxy**.
   - Cấu hình:
     - **Inbound Rules**:
       - **Pattern**: `.*` (chấp nhận tất cả yêu cầu).
       - **Rewrite URL**: `http://10.8.8.88:7777/{R:0}`.
     - **Enable Proxy**: Đảm bảo proxy được bật trong **Application Request Routing Cache** > **Server Proxy Settings** > Check **Enable proxy**.
   - Nhấn **Apply**.

2. **Cấu hình file `web.config`**:
   - Tạo file `web.config` trong `C:\inetpub\wwwroot\nextjs`:
     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <configuration>
         <system.webServer>
             <rewrite>
                 <rules>
                     <rule name="ReverseProxyToNextJS" stopProcessing="true">
                         <match url="(.*)" />
                         <action type="Rewrite" url="http://10.8.8.88:7777/{R:0}" />
                     </rule>
                 </rules>
             </rewrite>
         </system.webServer>
     </configuration>
     ```

3. **Phân quyền thư mục**:
   - Nhấp chuột phải vào `C:\inetpub\wwwroot\nextjs` > **Properties** > **Security**.
   - Thêm user **IIS_IUSRS** và cấp quyền **Full Control**.

### **2.4. Cấu hình firewall và kiểm tra**
1. **Mở cổng trong Windows Firewall**:
   - Mở **Windows Defender Firewall** > **Advanced Settings** > **Inbound Rules** > **New Rule**.
   - Cấu hình:
     - **Rule Type**: Port.
     - **Protocol and Ports**: TCP, Specific Ports: `80, 7777`.
     - **Action**: Allow the connection.
     - **Profile**: Chọn tất cả (Domain, Private, Public).
     - **Name**: `NextJSApp`.

2. **Kiểm tra ứng dụng**:
   - Truy cập `http://10.8.8.88` trên trình duyệt.
   - Nếu thành công, giao diện Next.js sẽ hiển thị.
   - Nếu lỗi, kiểm tra:
     - PM2 có đang chạy (`pm2 list`).
     - Ứng dụng Next.js có hoạt động tại `http://10.8.8.88:7777` (`curl http://10.8.8.88:7777`).
     - File `web.config` và quy tắc URL Rewrite trong IIS.

### **2.5. Tối ưu hóa và bảo mật**
1. **Cấu hình SSL**:
   - Lấy chứng chỉ SSL (ví dụ: Let's Encrypt).
   - Trong **IIS Manager**, vào **Bindings**, thêm binding `https` (cổng 443) và chọn chứng chỉ.
   - Cập nhật `web.config` để chuyển hướng HTTP sang HTTPS:
     ```xml
     <rule name="RedirectToHTTPS" stopProcessing="true">
         <match url="(.*)" />
         <conditions>
             <add input="{HTTPS}" pattern="^OFF$" />
         </conditions>
         <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />
     </rule>
     ```

2. **Tối ưu hóa hiệu suất**:
   - Bật nén Gzip trong IIS:
     ```xml
     <httpCompression directory="%SystemDrive%\inetpub\temp\IIS Temporary Compressed Files">
         <scheme name="gzip" dll="%Windir%\system32\inetsrv\gzip.dll" />
         <dynamicTypes>
             <add mimeType="text/*" enabled="true" />
             <add mimeType="message/*" enabled="true" />
             <add mimeType="application/javascript" enabled="true" />
             <add mimeType="*/*" enabled="false" />
         </dynamicTypes>
         <staticTypes>
             <add mimeType="text/*" enabled="true" />
             <add mimeType="message/*" enabled="true" />
             <add mimeType="application/javascript" enabled="true" />
             <add mimeType="*/*" enabled="false" />
         </staticTypes>
     </httpCompression>
     ```
   - Cấu hình caching tĩnh trong `next.config.js`:
     ```javascript
     module.exports = {
         async headers() {
             return [
                 {
                     source: '/:all*(css|js|png|jpg|jpeg|gif|svg)',
                     headers: [
                         {
                             key: 'Cache-Control',
                             value: 'public, max-age=31536000, immutable',
                         },
                     ],
                 },
             ];
         },
     };
     ```

3. **Giám sát và bảo trì**:
   - Kiểm tra trạng thái PM2: `pm2 list` hoặc `pm2 logs`.
   - Khởi động lại ứng dụng nếu cần: `pm2 restart next-app`.
   - Đảm bảo PM2 tự khởi động cùng hệ thống (`pm2 startup`).

---

## **3. Lưu ý quan trọng**
- **Tương thích**: Đảm bảo phiên bản Node.js và Next.js tương thích (kiểm tra [tài liệu Next.js](https://nextjs.org/docs)).
- **Hiệu suất**: Reverse proxy với IIS và PM2 là phương pháp hiệu quả cho các ứng dụng Next.js, đặc biệt khi cần mở rộng.
- **Xử lý lỗi**:
  - Kiểm tra log PM2: `pm2 logs`.
  - Kiểm tra log IIS trong `C:\inetpub\logs`.
  - Đảm bảo quyền truy cập thư mục và cấu hình `web.config` chính xác.
- **Bảo mật**: Sử dụng SSL và cập nhật Node.js/Next.js để vá lỗ hổng bảo mật.

---

## **4. Tài liệu tham khảo**
- Next.js: [https://nextjs.org/docs](https://nextjs.org/docs)
- PM2: [https://pm2.keymetrics.io/docs](https://pm2.keymetrics.io/docs)
- IIS URL Rewrite: [https://www.iis.net/downloads/microsoft/url-rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)
- Application Request Routing: [https://www.iis.net/downloads/microsoft/application-request-routing](https://www.iis.net/downloads/microsoft/application-request-routing)