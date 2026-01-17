# Hướng dẫn thiết lập Google Apps Script cho Form

## Bước 1: Mở Google Sheets

1. Mở file Google Sheets của bạn:
   https://docs.google.com/spreadsheets/d/1LhoUj30_-fVGA9rsFYn6c6rGEt10IIuHtxtJ8sibTzM/edit

2. Đảm bảo sheet có tên (hoặc đổi tên sheet đầu tiên) thành **"Orders"** (hoặc tên bạn muốn)

3. Tạo header cho các cột (dòng 1):
   - Cột A: **Họ và tên**
   - Cột B: **Email**
   - Cột C: **Số điện thoại**
   - Cột D: **Thời gian**

## Bước 2: Tạo Apps Script

1. Trong Google Sheets, click **Extensions** (Tiện ích mở rộng) → **Apps Script**

2. Xóa code mặc định và paste đoạn code sau:

```javascript
function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    var doc = SpreadsheetApp.getActiveSpreadsheet();
    var sheet = doc.getSheetByName('Orders');

    if (!sheet) {
      sheet = doc.insertSheet('Orders');
      sheet.appendRow(['Thời gian', 'Họ và tên', 'Số điện thoại', 'Chi tiết đơn hàng', 'Tổng tiền']);
    }

    var data = JSON.parse(e.postData.contents);

    sheet.appendRow([
      "'" + data.timestamp,
      data.fullName,
      "'" + data.phone,
      data.quantity,
      data.totalMoney
    ]);

    return ContentService.createTextOutput(JSON.stringify({ result: 'success' }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (e) {
    return ContentService.createTextOutput(JSON.stringify({ result: 'error', error: e }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}

// Test function (không bắt buộc)
function doGet(e) {
  return ContentService
    .createTextOutput('Google Apps Script đang hoạt động!')
    .setMimeType(ContentService.MimeType.TEXT);
}
```

3. Click **Save** (biểu tượng đĩa mềm) hoặc Ctrl+S
4. Đặt tên project: **"Order Form Handler"**

## Bước 3: Deploy Web App

1. Click nút **Deploy** (phía trên bên phải) → chọn **New deployment**

2. Trong cửa sổ mới:
   - Click biểu tượng **⚙️ Settings** bên cạnh "Select type"
   - Chọn **Web app**

3. Điền thông tin:
   - **Description**: "Order Form API v1"
   - **Execute as**: **Me** (địa chỉ email của bạn)
   - **Who has access**: **Anyone** (Quan trọng!)

4. Click **Deploy**

5. Có thể cần xác thực:
   - Click **Authorize access**
   - Chọn tài khoản Google của bạn
   - Click **Advanced** → **Go to Order Form Handler (unsafe)** (an toàn, chỉ là cảnh báo)
   - Click **Allow**

6. **SAO CHÉP WEB APP URL** - URL có dạng:
   ```
   https://script.google.com/macros/s/AKfycby.../exec
   ```

## Bước 4: Cập nhật URL vào Website

1. Mở file `public/index.html`

2. Tìm dòng (khoảng dòng 150):
   ```javascript
   const GOOGLE_SCRIPT_URL = 'YOUR_WEB_APP_URL_HERE';
   ```

3. Thay thế bằng URL bạn vừa copy:
   ```javascript
   const GOOGLE_SCRIPT_URL = 'https://script.google.com/macros/s/AKfycby.../exec';
   ```

4. Save file

## Bước 5: Test Form

1. Mở file `public/index.html` trong trình duyệt

2. Điền thông tin vào form và click **"Mua ngay"**

3. Kiểm tra:
   - Có thông báo "Thông tin của bạn đã được gửi shop sẽ liên hệ với bạn ngay bây giờ"
   - Dữ liệu xuất hiện trong Google Sheets

## Xử lý sự cố

### Lỗi: "Script function not found: doPost"
- Đảm bảo bạn đã save code Apps Script
- Redeploy lại Web app

### Lỗi: "Authorization required"
- Execute as phải là **Me** (email của bạn)
- Who has access phải là **Anyone**
- Có thể cần xóa deployment cũ và tạo mới

### Form submit nhưng không thấy data
- Mở Developer Console (F12) kiểm tra lỗi
- Kiểm tra tên sheet phải là "Orders"
- Kiểm tra URL đã paste đúng chưa

### Dữ liệu không hiển thị trong sheet
- Đảm bảo sheet đầu tiên có tên "Orders"
- Hoặc sửa code `getSheetByName('Orders')` thành tên sheet của bạn

## Cập nhật sau này

Nếu sửa code Apps Script:
1. Save code mới
2. Click **Deploy** → **Manage deployments**
3. Click **✏️ Edit** deployment hiện tại
4. Chọn **New version** trong dropdown "Version"
5. Click **Deploy**

URL sẽ không đổi, không cần update lại website!

---

## Bonus: Gửi Email tự động (Tùy chọn)

Thêm vào cuối hàm `doPost`, trước `return`:

```javascript
// Gửi email thông báo cho admin
MailApp.sendEmail({
  to: 'Anhkhai0709@gmail.com',
  subject: '🛒 Đơn hàng mới từ website',
  body: `Khách hàng mới đặt hàng:
  
Họ tên: ${data.fullName}
Email: ${data.email}
SĐT: ${data.phone}
Thời gian: ${data.timestamp}

Vui lòng liên hệ khách hàng sớm nhất!`
});
```

Lưu ý: Gmail giới hạn ~100 email/ngày cho Apps Script.
