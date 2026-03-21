# 🏗️ Selenium Chrome Controller (SCC) - Technical Documentation

Tài liệu này chi tiết các cơ chế kỹ thuật nội bộ của thư viện SCC (version 2.2.0), được thiết kế để hỗ trợ việc bảo trì và nâng cấp trong tương lai.

---

## 🚀 1. Kiến trúc tổng thể (Architecture)

Thư viện được xây dựng theo mô hình **Wrapper Pattern** quanh Selenium WebDriver, tập trung vào việc giảm thiểu boilerplate code và tăng điện kiện tự phục hồi (self-healing).

### Các module chính:
*   `controller.py`: Chứa lớp `ChromeController`, quản lý vòng đời trình duyệt và các tương tác.
*   `helpers.py`: Chứa các hàm tiện ích dùng chung (Logger, Retry Decorator, URL Decoder).
*   `constants.py`: Định nghĩa các biến hằng số, phím tắt (`SPECIAL_KEYS`) và phiên bản.
*   `models.py`: Chứa các dataclass định nghĩa cấu trúc dữ liệu (như `DownloadItem`).

---

## 🛠️ 2. Cơ chế quản lý Driver (Flexible Driver Search)

SCC triển khai một hệ thống tìm kiếm ChromeDriver linh hoạt để giảm thiểu các lỗi "Driver not found".

### Thứ tự ưu tiên (Priority):
1.  **Manual Path**: Đường dẫn `driver_path` được người dùng truyền trực tiếp vào hàm khởi tạo.
2.  **Project Root**: Tìm file `chromedriver.exe` hoặc `chromedriver` ngay tại thư mục gốc của dự án (thư mục cha của `scc/`).
3.  **Persistent Storage**: Đường dẫn thành công từ lần sử dụng trước đó (được lưu tại `scc_config.json`).
4.  **Interactive Fallback**:
    *   Yêu cầu người dùng nhập đường dẫn qua hộp thoại `pymsgbox`.
    *   Yêu cầu người dùng chọn file trực tiếp qua Windows File Dialog (`tkinter`).

### Cấu hình lưu trữ (`scc_config.json`):
Sử dụng file JSON để lưu lại đường dẫn driver làm việc gần nhất, giúp tự động hóa quá trình khởi động trong các phiên sau mà không cần cấu hình lại.

---

## 🔍 3. Kiểm tra tương thích phiên bản (Version Check)

Một trong những lỗi phổ biến nhất là sự mất cân bằng phiên bản giữa Chrome và ChromeDriver. SCC tích hợp cơ chế kiểm tra tự động trước khi khởi chạy (`begin()`):

*   **Chrome Version**: Truy xuất từ Windows Registry (`HKEY_CURRENT_USER\Software\Google\Chrome\BLBeacon`).
*   **Driver Version**: Thực thi lệnh `chromedriver --version` từ đường dẫn driver tìm thấy.
*   **Compatiblity Logic**: So sánh **Major Version** (Ví dụ: 124.x.x vs 124.y.y). Nếu khác nhau, thư viện sẽ đưa ra cảnh báo (`Warning`) nhưng không chặn thực thi để người dùng vẫn có thể ghi đè nếu cần.

---

## ⚡ 4. Hợp nhất cơ chế tương tác (Consolidated XPath Engine)

Để tối ưu hóa mã nguồn, toàn bộ các hàm "tìm kiếm và chờ đợi" đã được gộp lại xung quanh lõi `find_element`.

### Nguyên lý hoạt động:
```python
def find_element(self, xpath: str, timeout: float = 10, visible: bool = False) -> Optional[Any]:
    # Sử dụng WebDriverWait với logic linh hoạt (Presence vs Visibility)
```

Các phương thức công khai (`public methods`) như `wait_xpath`, `get_xpath`, `click_xpath`, `send_keys_xpath` đều sử dụng chung lõi này, đảm bảo tính nhất quán trong việc xử lý **Timeout** và **Exception**. Các hàm cũ bị trùng lặp như `_find_presence` và `_find_visible` đã bị loại bỏ.

---

## 📸 5. Hệ thống xử lý lỗi và Chụp ảnh màn hình (Error Handling)

SCC tích hợp cơ chế chụp ảnh màn hình tự động (`Self-Snapshot`) vào các phương thức tương tác chính.

### Cơ chế kích hoạt:
*   Khi `capture_on_error=True` (mặc định), bất kỳ ngoại lệ nào trong `click_xpath` hoặc `send_keys_xpath` đều kích hoạt hàm `capture_error`.
*   Ảnh được lưu với định dạng: `prefix_YYYYMMDD_HHMMSS.png`.
*   **Thư mục lưu trữ**: Tự động lấy tham số `screenshot_dir` hoặc tạo thư mục `logs/screenshots` tại thư mục gốc dự án nếu chưa tồn tại.

---

## 📈 6. Tính năng mở rộng mới (Từ bản Scratch)

1.  **Hỗ trợ Linux & Raspberry Pi OS (`Chromium`)**:
    *   Truy suất phiên bản `chromium-browser` trên Raspberry Pi.
    *   Quét tự động các đường dẫn `/usr/bin/chromedriver`, `/usr/lib/chromium-browser/chromedriver`.
2.  **Khởi tạo linh hoạt với `extra_args`**:
    *   Cho phép truyền mảng cấu hình (`List[str]`) khi khởi tạo lớp (ví dụ: `extra_args=['--no-sandbox', '--disable-dev-shm-usage']`) rất cần thiết khi chạy trên server / Pi.
3.  **Hàm chủ động `screenshot(filename)`**:
    *   Bên cạnh việc tự động chụp ảnh khi có lỗi (`capture_error`), có thể chủ động gọi `ctrl.screenshot("test.png")` và sẽ trả về chuỗi đường dẫn ảnh.

---

## 🔮 7. Định hướng phát triển tương lai (Roadmap)

1.  **Shadow DOM Engine**: Nâng cấp khả năng tìm kiếm xuyên qua các lớp Shadow DOM sâu hơn thay vì chỉ giới hạn ở trang Downloads.
2.  **Multi-Profile Manager**: Hỗ trợ quản lý và chuyển đổi nhanh giữa các Profile Chrome khác nhau một cách chuyên nghiệp.
3.  **Proxy Rotation**: Tích hợp cơ chế xoay vòng Proxy trực tiếp vào lớp `Options`.
4.  **Auto-Update Driver**: Tích hợp logic tự động tải bản driver mới nhất từ Google API khi phát hiện version mismatch nghiêm trọng.
