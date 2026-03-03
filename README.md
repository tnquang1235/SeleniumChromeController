# 🚀 Selenium Chrome Controller (SCC)

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![Selenium](https://img.shields.io/badge/selenium-4.x-green.svg)](https://www.selenium.dev/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **SCC** là một lớp điều khiển (abstraction layer) tối ưu hóa Selenium, tập trung vào độ ổn định, tính chuyên nghiệp và khả năng mở rộng trong các dự án tự động hóa trình duyệt.

**Phiên bản:** `2.1.0` | **Cập nhật:** `2026-03-03` | **Trạng thái:** Active

---

## 📋 Mục lục
- [🤖 Phân nhiệm: Con người & AI](#-phân-nhiệm-con-người--ai)
- [⚖️ Tại sao nên dùng SCC thay vì Selenium thuần?](#️-tại-sao-nên-dùng-scc-thay-vì-selenium-thuần)
- [📦 Cài đặt](#-cài-đặt)
- [🗂 Cấu trúc dự án đề xuất](#-cấu-trúc-dự-án-đề-xuất)
- [🔥 Tính năng nổi bật](#-tính-năng-nổi-bật)
- [💻 Ví dụ sử dụng nhanh](#-ví-dụ-sử-dụng-nhanh)
- [🎯 Cấu trúc lớp (Class Architecture)](#-cấu-trúc-lớp-class-architecture)

---

## 🤖 Phân nhiệm: Con người & AI (Gemini & ChatGPT)

Dự án được xây dựng dựa trên sự phối hợp rõ ràng về vai trò để tối ưu hiệu suất và đảm bảo khả năng bảo trì lâu dài:

*   **Con người (Tác giả)**: Định hướng chiến lược, thiết lập luồng nghiệp vụ (business logic), đưa ra các yêu cầu kỹ thuật đặc thù (như giải quyết Shadow DOM) và kiểm duyệt/ra quyết định cuối cùng cho mọi thay đổi.
*   **AI (Gemini & ChatGPT)**: Thực thi chi tiết mã nguồn, đảm bảo tiêu chuẩn cấu trúc (PEP8), tối ưu hóa các hàm bổ trợ, xử lý các trường hợp biên (edge cases) và soạn thảo hệ thống tài liệu.

> [!NOTE]
> Việc xác định rõ vai trò giúp việc đánh giá vấn đề và định hướng nâng cấp trong tương lai trở nên chính xác hơn. Tác giả sử dụng phương pháp **kiểm tra chéo (cross-checking)** giữa các công cụ AI để khai thác tối đa ưu điểm của từng model và tăng tính chuẩn xác cho mã nguồn.

---

## ⚖️ Tại sao nên dùng SCC thay vì Selenium thuần?

| Tính năng | Selenium thuần | SCC |
|------------|----------------|------|
| Độ trễ website | ❌ Tự xử lý | ✅ Tích hợp wait, retry |
| Tự cuộn chuột | ❌ Dễ lỗi | ✅ Tự động trước khi diễn ra |
| Click Force | ❌ Click thường | ✅ Fallback sang JS (Click force) |
| Quản lý tab | ❌ Dùng handle phức tạp | ✅ Quản lý bằng nickname |
| Theo dõi tải file | ❌ Khó triển khai | ✅ Shadow DOM (Đã tích hợp) |
| Debug | ❌ Khó truy vết | ✅ Chụp ảnh tự động khi lỗi |

---

## 📦 Cài đặt thư viện bổ sung

```bash
# Cách 1: Sử dụng requirements.txt
pip install -r requirements.txt

# Cách 2: Cài đặt thủ công
pip install selenium webdriver-manager pymsgbox retry python-dotenv pyyaml
```

---

## 🗂 Cấu trúc thư viện

Thư viện được tách nhỏ thành các module chuyên nghiệp:

```
scc/                    # Thư mục gói (Package)
├── __init__.py         # Export các thành phần chính
├── controller.py       # Lớp ChromeController chính (Trái tim của thư viện)
├── helpers.py          # Các công cụ hỗ trợ (logging, retry deco, decode url...)
├── models.py          # Định nghĩa các cấu trúc dữ liệu (DownloadItem...)
└── constants.py       # Lưu trữ VERSION, phím tắt, cấu hình
```

---

## 🔥 Tính năng nổi bật

*   **⚡ Smart Action Engine**: Tự động cuộn chuột và chờ đợi phần tử hiển thị.
*   **🛡️ Click-Force Technology**: Click cưỡng bức bằng JavaScript khi các phương pháp tiêu chuẩn bị chặn.
*   **📑 Alias-based Tab Management**: Quản lý đa tab bằng `name` chuyên nghiệp, không cần quản lý handle thủ công.
*   **📥 Shadow DOM Download Tracker**: Theo dõi file tải xuống qua `chrome://downloads` và trả về đường dẫn (path).
*   **📸 Error Auto-Capture**: Chụp ảnh màn hình tự động vào thư mục `/logs` khi gặp lỗi.
*   **🧩 Anti-Detection Bypassing**: Tích hợp các cấu hình giả lập người dùng và ẩn danh.

---

## 💻 Ví dụ sử dụng nhanh

```python
from selenium_chrome_controller import ChromeController

# Khởi tạo với cấu hình tối ưu
ctrl = ChromeController(headless=False, disable_images=True)
ctrl.begin()

# Mở tab với nickname chuyên nghiệp
ctrl.open_new_tab("https://finance.vietstock.vn", name="vietstock")

# Tương tác mượt mà (wait_visible_xpath trả về bool)
if ctrl.wait_visible_xpath("//input[@id='txtSearch']"):
    ctrl.send_keys_xpath("//input[@id='txtSearch']", ["VN30", "ENTER"])

# Chờ và lấy file vừa tải về tự động
filepath = ctrl.wait_for_download_complete(timeout=30)
print(f"✅ Data ready at: {filepath}")

ctrl.close()
```

---

## 🎯 Cấu trúc lớp (Class Architecture)

Việc refactor đã phân tác rõ rệt trách nhiệm của từng nhóm hàm:
1.  **Lifecycle**: `begin()`, `close()`.
2.  **Navigation**: `open_new_tab()`, `switch_to_tab()`, `openUrl()`.
3.  **Wait Helpers (Bool)**: `wait_xpath()`, `wait_visible_xpath()`.
4.  **Discovery (Retr./Fast)**: `get_xpath()`, `check_xpath()`, `is_visible()`.
5.  **Actions**: `click_xpath()`, `click_force()`, `send_keys_xpath()`.
6.  **Advanced**: `execute_script()`, `set_zoom()`, `fetch_json_via_js()`.

---
*Phát triển bởi 🌱Quang Amateur✨, hỗ trợ bởi AI. Hướng tới sự chuyên nghiệp và tin cậy.*
