# Phân tích hoạt động bán hàng đa sàn — Python, SQL và Power BI

Dự án thực hành Data Analyst về một công ty gia dụng bán trên Shopee, TikTok Shop và Lazada. Mục tiêu là kết hợp dữ liệu bán hàng, kho, nhân sự, nội dung và quảng cáo để hỗ trợ quyết định quản trị.

**Trạng thái:** đã triển khai quy trình thu thập và làm sạch 11 bảng bằng Python, phân tích bằng 12 truy vấn SQL và xây dựng báo cáo Power BI gồm 5 trang. Quy mô dữ liệu phân tích: **6.450 đơn hàng**, **1.200 SKU** và **1.200 video**.

[Portfolio](https://hieutran642002.github.io/Portfolio-TranTrungHieu/) · [Python Notebook](https://github.com/HieuTran642002/Portfolio-TranTrungHieu/blob/main/projects/da-sales-analysis/notebooks/01_extract_clean_load.ipynb) · [SQL Analysis](https://hieutran642002.github.io/Portfolio-TranTrungHieu/projects/da-sales-analysis/sql-analysis.html) · [Power BI Report](https://app.powerbi.com/view?r=eyJrIjoiZTgzMjMzMjQtZjdhZS00NGIwLWEzNGUtNDRkZDlkOWNiN2MyIiwidCI6ImFmMWYzNzUzLTM5MjUtNGU2Zi05NDliLTk3YzAwNzMyMDgwMyIsImMiOjEwfQ%3D%3D)

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

## 4. Quy mô dữ liệu và kết quả chính

| Bảng | Số dòng dữ liệu sạch |
|---|---:|
| orders | 6.450 |
| order_items | 12.999 |
| payments | 6.450 |
| products | 1.200 |
| content | 1.200 |
| stock_counts | 2.400 |
| receipts | 1.200 |
| attendance | 3.620 |
| warehouse_tasks | 5.940 |
| inventory_movements | 16.701 |
| ads_daily | 8.540 |

Số dòng trên là toàn bộ dữ liệu. Đơn hàng và quảng cáo thuộc quý I; một số tác vụ kho, thanh toán và hoàn hàng phát sinh sang tháng 4. Khi phân tích vận hành Q1, chỉ lấy ngày thực hiện trong 01/01–31/03; tồn cuối quý chốt tại 31/03. Đối soát đơn Q1 giữ cả các khoản thanh toán về sau quý.

| Chỉ số quý I/2026 | Kết quả |
|---|---:|
| Đơn hoàn thành | 5.370 / 6.450 — 83,26% |
| Doanh thu thuần | 5.218.612.000 VND |
| Lãi gộp / biên lãi gộp | 1.679.746.000 VND / 32,19% |
| Contribution trước quảng cáo | 934.712.560 VND |
| Chi quảng cáo | 2.347.858.405 VND |
| Chi sản xuất nội dung | 179.700.000 VND |
| Contribution sau quảng cáo và nội dung | −1.592.845.845 VND |
| Giao dịch lệch thanh toán | 68 giao dịch; lệch ròng +340.000 VND |
| Tồn sổ ngày 31/03 | 182.905 sản phẩm; 29.947.941.000 VND |
| Khớp kiểm kê | 2.335 / 2.400 cặp SKU–kho — 97,29% |
| Task PACK trong quý / task lỗi | 5.878 / 194 — 3,30% |
| Năng suất PACK | 23.503 sản phẩm / 728,8 giờ — 32,25 SP/giờ |
| Video thiếu creator_id | 8 video được giữ trong nhóm UNKNOWN |

Các tổng này là mốc đối chiếu trên dữ liệu sạch và SQL, với toàn bộ kênh/sản phẩm; các chỉ số trên dashboard thay đổi theo bộ lọc.

## 5. Công cụ

Python, pandas, openpyxl, requests, Jupyter Notebook/Anaconda, SQLite và DB Browser for SQLite. Power Query, DAX và Power BI dùng cho mô hình dữ liệu và báo cáo tương tác.

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
| sql/ | 12 truy vấn SQL phục vụ phân tích và đối soát |
| reports/ | Log làm sạch và báo cáo phân tích |
| output/powerbi/ | Dữ liệu cho mô hình Power BI gồm 7 dimension và 10 fact |

Notebook được trình bày tại `notebooks/01_extract_clean_load.ipynb`; bài SQL hiển thị tại `sql-analysis.html` trong thư mục dự án trên GitHub. Các thư mục dữ liệu và công cụ trong bảng trên thuộc bộ thực hành; không mặc định rằng repository trình bày chứa đầy đủ các tệp này.

## 7. Cách chạy bản thực hành hiện tại

1. Giải nén bộ dữ liệu thực hành DA_A_Z. Mở Jupyter trong thư mục có sources/, tools/ và requirements.txt.
2. Trong notebook, cài thư viện bằng `%pip install -r requirements.txt`.
3. Nhập ba CSV mẫu từ sources/sheets/ vào một Google Spreadsheet. Ghi lại ID file và gid của từng tab. Phương án link CSV yêu cầu quyền đọc không đăng nhập; chỉ dùng dữ liệu được phép chia sẻ.
4. Mở Terminal tại DA_A_Z và chạy `python tools/mock_api.py`. Giữ Terminal chạy trong lúc thu thập API. Token training-token là token mô phỏng.
5. Chạy các ô notebook theo thứ tự: thiết lập ROOT, lấy Excel/Sheets/API, lưu dữ liệu nguồn, clean, kiểm tra, xuất CSV và nạp SQLite.
6. Mở database SQLite cần phân tích trong DB Browser for SQLite. Trong bộ SQL bàn giao, chạy `sql/00_setup.sql` trước rồi các truy vấn 01–12 ở tab Execute SQL; dùng database đi kèm gói SQL để đối chiếu các tổng ở mục 4.

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

## 9. Phát hiện và đề xuất

- **Chi phí marketing vượt contribution trước quảng cáo:** tổng Ads và nội dung khoảng 2,528 tỷ, gấp 2,70 lần contribution trước Ads, khiến contribution sau marketing âm 1,593 tỷ. Cần kiểm tra chi phí, tracking và attribution, sau đó thử nghiệm phân bổ ngân sách; chưa kết luận phải cắt toàn bộ quảng cáo.
- **Doanh thu cao chưa đồng nghĩa đóng góp tốt nhất:** TikTok Shop dẫn đầu doanh thu khoảng 1,767 tỷ; Shopee tạo contribution trước Ads cao nhất khoảng 318,6 triệu. Đánh giá kênh cùng biên đóng góp, tỷ lệ hủy/hoàn và chi phí phục vụ.
- **Doanh thu tháng 3 phục hồi:** đạt 1,772 tỷ, tăng 8,19% so với tháng 2; contribution sau marketing vẫn âm 489,48 triệu. Cần tách số đơn, AOV, cơ cấu sản phẩm và chi phí trước khi giải thích nguyên nhân.
- **Tồn kho lớn so với tốc độ bán:** DIO bình quân khoảng 703 ngày. Rà soát theo SKU/kho, tốc độ bán và kế hoạch nhập; không diễn giải DIO thành tuổi hàng.
- **Cần truy vết ngoại lệ:** 68 giao dịch lệch và 65 cặp SKU–kho lệch kiểm kê. Giá trị lệch kiểm kê tuyệt đối khoảng 17,39 triệu, khác lệch ròng +5,29 triệu; ưu tiên đối chiếu chứng từ và kiểm đếm lại.
- **Giới hạn gán nguồn marketing:** 1.032/6.450 đơn có video_id, tương đương 16%. Doanh thu attributed của nền tảng không cùng phạm vi với doanh thu nội bộ; không cộng hai nguồn và không suy ra ROI hoặc tác động nhân quả từ ROAS.

Báo cáo Power BI gồm năm trang: **Tổng quan; Sales & đối soát; Tồn kho & kiểm kê; Hiệu quả kho; Marketing**. Các đề xuất trên là hướng kiểm tra từ dữ liệu mô phỏng, chưa phải kết quả cải thiện đã triển khai tại doanh nghiệp thật.

## 10. Phạm vi và nguồn gốc

Đây là dự án học tập sử dụng dữ liệu giả lập và API mô phỏng local, không phải dữ liệu doanh nghiệp hoặc kết nối API sàn thật. Bộ dữ liệu, API server, mẫu biểu và mã tham chiếu được cung cấp trong bộ thực hành DA_A_Z. Phần triển khai được thực hiện theo hướng dẫn, có điều chỉnh cách lấy Google Sheets qua CSV.

Không đưa khóa JSON, token thật hoặc dữ liệu nhạy cảm vào repository. Dự án chưa có dữ liệu phiên truy cập hoặc customer_id để tính conversion rate website, mua lại hoặc RFM đáng tin cậy.

