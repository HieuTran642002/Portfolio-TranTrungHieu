# Phân tích hoạt động bán hàng đa sàn — Python, SQL và Power BI

Dự án thực hành Data Analyst về một công ty gia dụng bán trên Shopee, TikTok Shop và Lazada. Mục tiêu là kết hợp dữ liệu bán hàng, kho, nhân sự, nội dung và quảng cáo để hỗ trợ quyết định quản trị.

**Trạng thái:** đã thu thập và làm sạch cơ bản 11 bảng, nạp vào SQLite; đang bắt đầu phân tích SQL. Dashboard Power BI và kết luận kinh doanh chưa hoàn thành.

## 1. Bài toán kinh doanh

Dự án hướng đến năm câu hỏi:

- Sàn nào có doanh thu cao nhưng đóng góp sau giá vốn và phí thấp?
- SKU nào bán tốt, SKU nào giữ nhiều vốn trong tồn kho?
- Năng suất đóng gói và tỷ lệ lỗi của nhân viên kho thay đổi thế nào?
- Video và người phụ trách nào có hiệu quả quảng cáo tốt?
- Đơn hàng nào có chênh lệch giữa thanh toán thực tế và dự kiến?

Kỳ dữ liệu kinh doanh: quý I/2026. Ngày thanh toán và ngày cập nhật được xử lý riêng với ngày đặt đơn.

## 2. Nguồn dữ liệu và cách thu thập

| Nguồn | Bảng | Cách lấy |
|---|---|---|
| Excel | receipts, attendance, inventory_movements | pandas.read_excel |
| Google Sheets | products, content, stock_counts | pandas.read_csv từ link xuất CSV của từng tab |
| API mô phỏng ba sàn | orders | requests.get, xử lý cấu trúc JSON riêng theo sàn và phân trang |
| API mô phỏng nội bộ | order_items, payments, warehouse_tasks, ads_daily | requests.get và phân trang |

Google Sheets API với service account đã được thử nghiệm như phương án bổ sung cho Sheet riêng tư. Phương án chính được chọn cho bài này là đọc link CSV; đây là điều chỉnh so với hướng dẫn ban đầu.

## 3. Quy trình thực hiện

Thu thập → Lưu raw và staging → Profiling → Làm sạch và kiểm tra → SQLite → SQL → Power BI → Insight và đề xuất.

Các bước đã thực hiện:

- Chuẩn hóa mã: bỏ khoảng trắng đầu/cuối và chuyển thành chữ hoa.
- Loại bản sao giống hoàn toàn; chặn bản ghi cùng khóa nhưng khác nội dung.
- Chuyển số, ngày và thời điểm cập nhật về đúng kiểu; giữ múi giờ cho updated_at.
- Giữ video thiếu người phụ trách trong nhóm UNKNOWN; không suy đoán mã từ tên.
- Kiểm tra khóa, SKU/video/đơn hàng tham chiếu và một số quy tắc nghiệp vụ.
- Tách dữ liệu nguồn, dữ liệu xử lý trung gian và dữ liệu sạch.
- Nạp 11 bảng sạch vào company.db.

Kiểm tra cơ bản chưa thay thế đối soát tài chính, tồn kho và nghiệm thu báo cáo cuối cùng.

## 4. Kết quả kỹ thuật hiện tại

| Hạng mục | Kết quả |
|---|---|
| orders | 6.015 dòng raw → 6.000 đơn sau loại 15 bản sao |
| attendance | 3.606 dòng raw → 3.600 dòng sau loại 6 bản sao |
| order_items | 12.105 dòng |
| warehouse_tasks | 5.530 dòng |
| inventory_movements | 15.788 dòng |
| content thiếu creator_id | 8 video được giữ lại với mã UNKNOWN |
| SQLite | Đã mở được company.db với 11 bảng trong DB Browser |

Đây là kết quả xử lý dữ liệu, chưa phải phát hiện kinh doanh. Doanh thu, đóng góp, hiệu quả Ads và đề xuất sẽ được bổ sung sau khi tính bằng SQL và đối soát.

## 5. Công cụ

Python, pandas, openpyxl, requests, Jupyter Notebook/Anaconda, SQLite và DB Browser for SQLite. Power BI được dự kiến dùng cho phần báo cáo trên Windows.

## 6. Cấu trúc dự án

| Thư mục hoặc file | Nội dung |
|---|---|
| README.md | Giới thiệu và hướng dẫn dự án |
| requirements.txt | Thư viện Python |
| sources/ | Dữ liệu mẫu Excel, Sheets và JSON phục vụ API mô phỏng |
| tools/mock_api.py | Server API chạy local |
| Notebook .ipynb | Thu thập, profiling, làm sạch và nạp database |
| output/raw/ | Bản dữ liệu chưa clean theo lần lấy |
| output/staging/ | CSV trung gian |
| output/clean/ | 11 CSV sạch |
| output/company.db | Database SQLite |
| sql/ | Câu lệnh SQL sẽ được lưu khi phân tích |
| reports/ | Log làm sạch và báo cáo phân tích |
| output/powerbi/ | Dữ liệu xuất cho Power BI — phần cần hoàn thiện |

Notebook hiện đang dùng tên Untitled.ipynb; tên và hướng dẫn chạy sẽ được cập nhật khi tổ chức lại dự án để bàn giao.

## 7. Cách chạy bản thực hành hiện tại

1. Giải nén bộ dữ liệu thực hành DA_A_Z. Mở Jupyter trong thư mục có sources/, tools/ và requirements.txt.
2. Trong notebook, cài thư viện bằng `%pip install -r requirements.txt`.
3. Nhập ba CSV mẫu từ sources/sheets/ vào một Google Spreadsheet. Ghi lại ID file và gid của từng tab. Phương án link CSV yêu cầu quyền đọc không đăng nhập; chỉ dùng dữ liệu được phép chia sẻ.
4. Mở Terminal tại DA_A_Z và chạy `python tools/mock_api.py`. Giữ Terminal chạy trong lúc thu thập API. Token training-token là token mô phỏng.
5. Chạy các ô notebook theo thứ tự: thiết lập ROOT, lấy Excel/Sheets/API, lưu dữ liệu nguồn, clean, kiểm tra, xuất CSV và nạp SQLite.
6. Mở output/company.db trong DB Browser for SQLite. Viết câu lệnh ở tab Execute SQL và lưu các query thành file .sql trong sql/.

Quy trình đang chạy thủ công, chưa có một lệnh pipeline hoàn chỉnh. Khi khởi động lại Kernel, cần chạy lại các ô thiết lập và tạo biến. Bước nạp SQLite dùng replace có chủ đích để thay bảng, tránh cộng thêm dòng khi chạy lại.

## 8. Các định nghĩa cần giữ đúng

- Doanh thu ròng: đơn không hủy tính qty × unit_price − discount − refund; đơn hủy bằng 0.
- Giá vốn ròng: chỉ tính cho đơn completed.
- Đóng góp trước Ads: doanh thu ròng − giá vốn − phí sàn − vận chuyển.
- Phí cấp đơn không được cộng lặp theo số dòng sản phẩm.
- Tồn cuối ngày là số lượng lũy kế đến ngày đó; không cộng tồn snapshot qua nhiều ngày.
- ROAS tổng = tổng doanh thu attributed / tổng spend.
- Doanh thu Ads tự báo không cộng vào doanh thu bán hàng.
- Chi phí sản xuất tính một lần mỗi video; đóng góp không được gọi là lợi nhuận ròng khi thiếu OPEX.

## 9. Phần sẽ bổ sung

- Query và kết quả phân tích SQL, cùng các kiểm tra bảo toàn số liệu sau JOIN.
- Mô hình fact/dimension, dữ liệu xuất và dashboard Power BI năm trang.
- Đối soát Python, SQL và Power BI.
- Năm insight có số liệu, bằng chứng và giới hạn.
- Ba hành động có owner, thời hạn và KPI đo thành công.
- Hướng dẫn chạy lại từ đầu và ảnh minh họa báo cáo.

## 10. Phạm vi và nguồn gốc

Đây là dự án học tập sử dụng dữ liệu giả lập và API mô phỏng local, không phải dữ liệu doanh nghiệp hoặc kết nối API sàn thật. Bộ dữ liệu, API server, mẫu biểu và mã tham chiếu được cung cấp trong bộ thực hành DA_A_Z. Phần triển khai được thực hiện theo hướng dẫn, có điều chỉnh cách lấy Google Sheets qua CSV.

Không đưa khóa JSON, token thật hoặc dữ liệu nhạy cảm vào repository. Mức độ đóng góp và kết quả cuối sẽ được mô tả lại khi dự án hoàn thành.

