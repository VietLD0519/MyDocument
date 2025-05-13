## **1. Yêu cầu**
### **1.1. Môi trường server**
- Máy chủ Windows với **IIS 8.0+** đã cài đặt.
- **Node.js** phiên bản tương thích với Next.js (khuyến nghị: Node.js 18.x hoặc 20.x).
- Module **iisnode** đã cài đặt trên server.
- Ứng dụng Next.js đã được build (`next build`).
- Địa chỉ server: `10.8.8.88`.
- Ứng dụng Next.js chạy thông qua iisnode, truy cập qua `http://10.8.8.88`.

### **1.2. Công cụ bổ sung**
- **IIS URL Rewrite Module**: Để xử lý các yêu cầu HTTP.
- **Quyền quản trị**: Để cấu hình IIS và phân quyền thư mục.

---

## **2. Các bước cấu hình**

### **2.1. Cài đặt iisnode**
1. **Tải và cài đặt iisnode**:
   - Tải từ [https://github.com/tjanczuk/iisnode/releases](https://github.com/tjanczuk/iisnode/releases) (chọn bản x64 hoặc x86 phù hợp).
   - Chạy file `.msi` và làm theo hướng dẫn cài đặt.
   - Kiểm tra thư mục `C:\Program Files\iisnode` để xác nhận cài đặt thành công.

2. **Cấu hình IIS để sử dụng iisnode**:
   - Mở **IIS Manager**.
   - Vào **Modules** tại cấp server, kiểm tra module `iisnode` có xuất hiện không.
   - Nếu không, đăng ký module thủ công:
     ```cmd
     %programfiles%\iisnode\install.bat
     ```

3. **Cài đặt URL Rewrite Module**:
   - Tải từ [https://www.iis.net/downloads/microsoft/url-rewrite](https://www.iis.net/downloads/microsoft/url-rewrite).
   - Cài đặt và xác nhận module xuất hiện trong **IIS Manager**.

### **2.2. Chuẩn bị ứng dụng Next.js**
1. **Build ứng dụng Next.js**:
   - Trong thư mục dự án, chạy:
     ```bash
     npm run build
     ```
   - Thư mục `.next` sẽ được tạo, chứa các tệp tối ưu hóa.

2. **Tạo file khởi động cho iisnode**:
   - Tạo file `server.js` trong thư mục gốc của dự án:
     ```javascript
     const next = require('next');
     const { createServer } = require('http');

     const port = parseInt(process.env.PORT, 10) || 7777;
     const dev = process.env.NODE_ENV !== 'production';
     const app = next({ dev });
     const handle = app.getRequestHandler();

     app.prepare().then(() => {
         createServer((req, res) => {
             handle(req, res);
         }).listen(port, (err) => {
             if (err) throw err;
             console.log(`> Ready on http://localhost:${port}`);
         });
     });
     ```
   - Cài đặt dependencies:
     ```bash
     npm install next
     ```

3. **Sao chép dự án vào server**:
   - Sao chép thư mục dự án (bao gồm `.next`, `node_modules`, `package.json`, `server.js`) vào `C:\inetpub\wwwroot\my-next-app`.

### **2.3. Cấu hình website trong IIS**
1. **Tạo website trong IIS**:
   - Mở **IIS Manager**, nhấp chuột phải vào **Sites** > **Add Website**.
   - Cấu hình:
     - **Site name**: `NextJSApp`.
     - **Physical path**: `C:\inetpub\wwwroot\my-next-app`.
     - **Binding**:
       - **Type**: `http`.
       - **IP Address**: `10.8.8.88` hoặc `All Unassigned`.
       - **Port**: `80` (hoặc 8080 nếu cần).
       - **Host name**: Để trống hoặc nhập domain.
   - Nhấn **OK**.

2. **Cấu hình file `web.config`**:
   - Tạo file `web.config` trong `C:\inetpub\wwwroot\my-next-app`:
     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <configuration>
         <system.webServer>
             <handlers>
                 <add name="iisnode" path="server.js" verb="*" modules="iisnode" />
             </handlers>
             <rewrite>
                 <rules>
                     <rule name="NextJSApp" stopProcessing="true">
                         <match url="^(.*)$" />
                         <conditions logicalGrouping="MatchAll" trackAllCaptures="false" />
                         <action type="Rewrite" url="server.js" />
                     </rule>
                 </rules>
             </rewrite>
             <iisnode nodeProcessCommandLine="&quot;%programfiles%\nodejs\node.exe&quot;" />
         </system.webServer>
     </configuration>
     ```

3. **Phân quyền thư mục**:
   - Nhấp chuột phải vào `C:\inetpub\wwwroot\my-next-app` > **Properties** > **Security**.
   - Thêm user **IIS_IUSRS** và cấp quyền **Full Control**.

### **2.4. Cấu hình firewall và kiểm tra**
1. **Mở cổng trong Windows Firewall**:
   - Mở **Windows Defender Firewall** > **Advanced Settings** > **Inbound Rules** > **New Rule**.
   - Cấu hình:
     - **Rule Type**: Port.
     - **Protocol and Ports**: TCP, Specific Ports: `80`.
     - **Action**: Allow the connection.
     - **Profile**: Chọn tất cả (Domain, Private, Public).
     - **Name**: `NextJSApp`.

2. **Kiểm tra ứng dụng**:
   - Truy cập `http://10.8.8.88` trên trình duyệt.
   - Nếu thành công, giao diện Next.js sẽ hiển thị.
   - Nếu lỗi, kiểm tra:
     - Cú pháp file `web.config`.
     - Chạy thử `node server.js` trong thư mục dự án.
     - Log lỗi trong `C:\inetpub\wwwroot\my-next-app\iisnode`.

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
   - Kiểm tra log iisnode trong thư mục `iisnode`.
   - Sử dụng **Windows Task Scheduler** để tự động khởi động lại ứng dụng nếu server khởi động lại.

---

## **3. Lưu ý quan trọng**
- **Tương thích**: Đảm bảo phiên bản Node.js và Next.js tương thích (kiểm tra tài liệu Next.js).
- **Hiệu suất**: iisnode phù hợp cho ứng dụng đơn giản. Với ứng dụng Next.js phức tạp, cân nhắc sử dụng **reverse proxy** (thay vì iisnode) để tối ưu hiệu suất.
- **Xử lý lỗi**:
  - Kiểm tra log iisnode trong `C:\inetpub\wwwroot\my-next-app\iisnode`.
  - Đảm bảo quyền truy cập thư mục và cấu hình `web.config` chính xác.
- **Bảo mật**: Luôn sử dụng SSL và cập nhật Node.js/Next.js để vá các lỗ hổng bảo mật.

---

## **4. Tài liệu tham khảo**
- iisnode: [https://github.com/tjanczuk/iisnode](https://github.com/tjanczuk/iisnode)
- Next.js: [https://nextjs.org/docs](https://nextjs.org/docs)
- IIS URL Rewrite: [https://www.iis.net/downloads/microsoft/url-rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)