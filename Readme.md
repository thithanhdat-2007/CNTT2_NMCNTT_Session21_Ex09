# Công thức Prompt 3 Thành phần & Xử lý lỗi Kiosk VoltCharge

## 1. Bối cảnh & Nhật ký lỗi
* **Bối cảnh**: Trụ sạc nhanh số 04 tại hệ thống Kiosk **VoltCharge** (chạy Windows 11) đột ngột dừng cấp điện và báo lỗi `System.UnauthorizedAccessException` với mã thoát `exit code 5 (0x00000005)`.
* **Vấn đề**: Nhân viên mới chỉ hỏi cụt ngủn *"AI ơi phần mềm trạm sạc của tôi bị hỏng rồi làm sao đây?"* khiến AI đoán mò và khuyên tháo vỏ sắt kiểm tra dây điện.

## 2. Phần 1: Thiết kế Prompt 3 Thành phần Chuẩn hóa

| Yêu cầu | Nội dung giải quyết ngắn gọn |
| :--- | :--- |
| **Câu 1: Tại sao câu hỏi cụt ngủn lại gây ra hiện tượng ảo giác (AI Hallucination)?** | • **Thiếu hoàn toàn Ngữ cảnh (Context) và Mã lỗi (Raw Error)**: Cụm từ *"phần mềm trạm sạc bị hỏng"* quá mơ hồ (không rõ hệ điều hành gì, lỗi mạng, lỗi phần cứng hay lỗi phân quyền).<br>• **Cơ chế đoán từ theo xác suất của LLM**: Khi không có dữ liệu đầu vào cụ thể, AI tự động suy diễn theo các kịch bản phổ biến nhất liên quan đến từ khóa *"trạm sạc hỏng"* (như đứt dây, mất nguồn điện), dẫn đến câu trả lời phỏng đoán sai lệch hoàn toàn với lỗi phần mềm thực tế. |
| **Câu 2: Mẫu Prompt 3 Thành phần chuẩn kỹ sư** | Xem khung mẫu Prompt chuẩn hóa ngay bên dưới (tách rõ `[Context]`, `[Raw Error]`, `[Instruction]`). |

### Mẫu Prompt 3 Thành phần gửi cho AI

```text
[Context - Ngữ cảnh]
Tôi là kỹ thuật viên đang vận hành hệ thống Kiosk trạm sạc xe điện thông minh VoltCharge (Trụ sạc số 04) chạy trên hệ điều hành Windows 11. Khi khởi động tiến trình điều khiển trạm sạc (ControllerDaemon), phần mềm đột ngột dừng hoạt động và không cấp điện được.

[Raw Error - Lỗi nguyên văn]
[TERMINAL ERROR LOG - VOLTCHARGE CHARGER #04]
2026-09-21 21:15:30 [FATAL] ControllerDaemon failed to start.
System.UnauthorizedAccessException: Access to the path 'C:\Program Files\VoltCharge\data\session.log' is denied.
   at System.IO.FileStream.ValidateFileHandle(SafeFileHandle handle)
   at VoltCharge.Core.SessionManager.Initialize()
InnerException: None. Process terminated with exit code 5 (0x00000005).
Suggestion: Run the application with elevated administrator privileges or verify NTFS file permissions.

[Instruction - Yêu cầu]
1. Hãy dịch thông báo lỗi trên và giải thích nguyên nhân gốc rễ bằng tiếng Việt bình dân, dễ hiểu nhất cho kỹ thuật viên mới.
2. Hướng dẫn chi tiết 3 bước khắc phục trực tiếp trên Windows 11 (cách chạy quyền Administrator và cấp quyền ghi file cho thư mục data) để trạm sạc hoạt động lại ngay.
```

## 3. Phần 2: Giải mã Lỗi & Kịch bản Khắc phục

| Yêu cầu | Nội dung giải quyết chi tiết & Dễ hiểu |
| :--- | :--- |
| **Câu 3: Dịch lỗi & Giải thích nguyên nhân gốc rễ bằng tiếng Việt đời thường** | • **Dịch thông báo lỗi**: *"Lỗi truy cập trái phép (`UnauthorizedAccessException`): Quyền truy cập vào đường dẫn `C:\Program Files\VoltCharge\data\session.log` bị từ chối (`Access is denied`). Tiến trình kết thúc với mã lỗi số 5 (`exit code 5`)."*<br>• **Giải thích dễ hiểu**: Trạm sạc **không hề bị hỏng phần cứng hay đứt dây điện**. Nguyên nhân là khi phần mềm trạm sạc bắt đầu chạy, nó cần mở cuốn sổ nhật ký (`session.log`) nằm trong thư mục `C:\Program Files\VoltCharge\data\` để ghi lại lịch sử ca sạc. Tuy nhiên, trên Windows, thư mục `C:\Program Files` là "khu vực bảo vệ nghiêm ngặt", tài khoản nhân viên bình thường chỉ được phép **Đọc (Read)** chứ không được phép **Ghi/Sửa (Write)**. Vì bị Windows chặn không cho ghi sổ, phần mềm tự động tắt luôn để đảm bảo an toàn. |
| **Câu 4: 3 bước xử lý cụ thể tại chỗ trên Windows 11** | • **Bước 1 – Chạy phần mềm bằng quyền Quản trị viên (Khắc phục tức thời)**:<br>  Nhấp chuột phải vào biểu tượng phần mềm **VoltCharge (ControllerDaemon)** trên màn hình Desktop (hoặc trong thư mục cài đặt) $\rightarrow$ Chọn **Run as administrator** $\rightarrow$ Nhấn **Yes** khi hộp thoại UAC hiện lên để cấp quyền tạm thời.<br>• **Bước 2 – Cấp quyền ghi (NTFS Permissions) vĩnh viễn cho thư mục `data`**:<br>  1. Mở File Explorer, vào `C:\Program Files\VoltCharge\`, nhấp chuột phải vào thư mục **`data`** $\rightarrow$ Chọn **Properties**.<br>  2. Chuyển sang thẻ **Security** $\rightarrow$ Nhấn nút **Edit...** (để đổi quyền).<br>  3. Tại khung *Group or user names*, chọn nhóm **Users** (hoặc tài khoản chạy Kiosk) $\rightarrow$ Ở khung bên dưới, tích chọn ô **Allow** tại dòng **Modify** và **Write** $\rightarrow$ Nhấn **Apply** và **OK**.<br>• **Bước 3 – Thiết lập tự động chạy quyền Admin & Kiểm tra lại**:<br>  Nhấp chuột phải vào file chạy `.exe` của VoltCharge $\rightarrow$ **Properties** $\rightarrow$ Thẻ **Compatibility** $\rightarrow$ Tích chọn **Run this program as an administrator** $\rightarrow$ **OK**. Khởi động lại phần mềm và kiểm tra xem tệp `session.log` đã cập nhật giờ mới chưa. |
| **Câu 5: Xử lý Bẫy an ninh khi log chứa `API_KEY: volt_secret_token_889922`** | • **Cách xử lý trước khi dán vào AI**: Kỹ thuật viên bắt buộc phải **xóa hoặc thay thế (Masking/Redacting)** chuỗi mã thật bằng ký tự giả lập trước khi gửi, ví dụ sửa thành: `API_KEY: [REDACTED_SECRET_KEY]` hoặc `API_KEY: ****************`.<br>• **Lý do an toàn thông tin**:<br>  1. Mọi nội dung gửi lên cửa sổ chat AI công cộng đều đi qua máy chủ bên thứ ba và có thể được lưu trữ trong lịch sử hội thoại hoặc dùng để huấn luyện mô hình.<br>  2. Nếu để lộ `volt_secret_token_889922`, kẻ xấu hoặc người truy cập được lịch sử chat có thể dùng khóa này để chiếm quyền điều khiển trạm sạc từ xa, giả mạo giao dịch nạp điện hoặc đánh cắp dữ liệu hệ thống VoltCharge. |
