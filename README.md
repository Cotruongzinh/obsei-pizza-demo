# Obsei Pizza 4P's -- Customer Feedback Analysis Demo

Demo phân tích phản hồi khách hàng cho Pizza 4P's bằng **Obsei 1.1.1**,
sử dụng dữ liệu CSV mô phỏng. Dự án minh họa quy trình nạp dữ liệu, phân
loại phản hồi theo sentiment/intent, khám phá Theme, lưu kết quả và truy
vấn qua Obsei MCP.

> **Lưu ý quan trọng:** Các bộ dữ liệu trong repository là dữ liệu tổng
> hợp/mô phỏng phục vụ thực nghiệm và demo. Không xem các thống kê hoặc
> ví dụ trong demo là phản ánh thực tế khách hàng Pizza 4P's.

## 1. Mục tiêu

-   Nạp phản hồi khách hàng từ CSV.
-   Phân loại phản hồi theo **Sentiment** và **Intent** bằng mô hình
    local.
-   Khám phá các nhóm chủ đề (**Themes**) từ nội dung phản hồi.
-   Lưu dữ liệu và kết quả phân tích bằng DuckDB và Parquet.
-   Dùng Obsei Studio để quan sát dashboard và Obsei MCP để truy vấn dữ
    liệu bằng Codex.

## 2. Công nghệ sử dụng

| Thành phần | Công nghệ |
|---|---|
| Nền tảng phân tích | Obsei 1.1.1 |
| Ngôn ngữ chạy | Python 3.11.9 |
| Dữ liệu đầu vào | CSV |
| Cơ sở dữ liệu | DuckDB |
| Dữ liệu xuất | Parquet |
| Phân loại | Obsei `classify` enricher, model local |
| Khám phá chủ đề | Obsei Themes |
| Tích hợp trợ lý lập trình | Obsei MCP Plugin / Codex |

## 3. Cấu trúc repository

``` text
obsei-pizza-demo/
├── .gitattributes
├── .gitignore
├── AGENTS.md
├── obsei.yaml
├── obsei_pizza_demo_50.csv
└── obsei_pizza_dataset/
    ├── README_OBSEI_DATA.txt
    ├── obsei_pizza_original_800.csv
    ├── obsei_pizza_augmented_980.csv
    └── obsei_eval_reference_180.csv
```

Các thành phần phát sinh khi chạy được giữ ở máy local và không commit
lên GitHub:

-   `.venv/`: Python virtual environment.
-   `.obsei/`: trạng thái và secrets cục bộ.
-   `*.duckdb`, `*.duckdb.wal`: database local.
-   `exports/`, `exports_demo_50/`, `exports_demo_50_tuned/`: kết quả
    Parquet.
-   `feedback.csv`: CSV mẫu do `obsei init` tạo.

## 4. Dataset

| File | Số dòng dữ liệu | Mục đích |
|---|---:|---|
| `obsei_pizza_demo_50.csv` | 50 | Dataset demo chính, được cấu hình trong pipeline hiện tại |
| `obsei_pizza_dataset/obsei_pizza_original_800.csv` | 800 | Dataset tổng hợp ban đầu |
| `obsei_pizza_dataset/obsei_pizza_augmented_980.csv` | 980 | Dataset tổng hợp mở rộng |
| `obsei_pizza_dataset/obsei_eval_reference_180.csv` | 180 | Dataset tham chiếu phục vụ đánh giá |

Các cột của dataset 50 được ánh xạ trong `obsei.yaml`. Với các dataset
khác, hãy kiểm tra header trước khi cấu hình source; không giả định tất
cả CSV có cùng schema.

## 5. Cấu hình hiện tại

File `obsei.yaml` là cấu hình chạy chính. Các thiết lập quan trọng:

| Hạng mục | Giá trị hiện tại |
|---|---|
| DuckDB | `obsei_pizza_demo_50_tuned.duckdb` |
| Pipeline | `feedback` |
| CSV source | `pizza_feedback` |
| CSV path | `obsei_pizza_demo_50.csv` |
| Enricher | `classify` |
| Model | `MoritzLaurer/bge-m3-zeroshot-v2.0` (local) |
| Classification threshold | `0.4` |
| Theme embedder | `auto` |
| Theme similarity | `0.50` |
| Duplicate similarity | `0.9` |
| k-anonymity | `2` |
| Parquet output | `exports_demo_50_tuned/` |

Các Intent được cấu hình:

-   `booking`: đặt bàn, thay đổi, hủy hoặc tra cứu thông tin đặt bàn.
-   `chatbot_support`: vấn đề hoặc yêu cầu hỗ trợ liên quan đến chatbot.
-   `staff_service`: phản hồi về nhân viên và chất lượng phục vụ.
-   `food_menu`: món ăn, menu, nguyên liệu và dị ứng thực phẩm.
-   `branch_experience`: chi nhánh, địa chỉ, không gian và tiện ích.
-   `takeaway_delivery`: đơn mang về, giao hàng và trạng thái nhận món.
-   `other`: nội dung không thuộc các nhóm nghiệp vụ trên.

Không đặt API key, token hoặc thông tin bí mật trực tiếp trong
`obsei.yaml`. Secrets được quản lý riêng trong môi trường local và không
được commit.

## 6. Cài đặt

### Yêu cầu

-   Windows 10/11 hoặc môi trường tương thích.
-   Python 3.11.x.
-   Git.
-   Kết nối Internet khi cần tải package/model lần đầu.

### Clone repository

``` powershell
git clone https://github.com/Cotruongzinh/obsei-pizza-demo.git
cd obsei-pizza-demo
```

### Tạo virtual environment và cài Obsei

Ví dụ tạo môi trường mới bằng Python 3.11:

``` powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install "obsei[local]==1.1.1"
```

Nếu project đã có môi trường `.venv` hoạt động, có thể kích hoạt và kiểm
tra thay vì tạo lại:

``` powershell
.\.venv\Scripts\Activate.ps1
python --version
obsei --version
```

Tải model cần thiết theo hướng dẫn của Obsei:

``` powershell
obsei models download
```

Nếu Obsei báo thiếu một embedding model cụ thể, tải đúng model được
thông báo trong lỗi, theo cú pháp `obsei models download <model-name>`.

> Thời gian tải model phụ thuộc tốc độ mạng và cấu hình máy. Không cần
> tải lại model nếu model đã có trong cache.

## 7. Chạy pipeline với dataset demo 50

Đảm bảo terminal đang ở thư mục gốc project và virtual environment đã
được kích hoạt.

### Bước 1: Kiểm tra cấu hình và source

``` powershell
obsei check
```

Kết quả mong đợi: source CSV và sink Parquet được xác nhận sẵn sàng.

### Bước 2: Chạy pipeline

``` powershell
obsei run
```

Một lần chạy thành công trước đây với dataset demo ghi nhận:

``` text
feedback: fetched 50, stored 50, enriched 50, sent parquet=50
```

### Bước 3: Khám phá Theme

``` powershell
obsei themes
```

Lệnh này thực hiện embedding và gán phản hồi vào các Theme. Model
embedding có thể cần được tải trước. Theme được khám phá có thể thay đổi
tùy dữ liệu, model và cấu hình.

> **Cảnh báo:** `obsei run` và `obsei themes` là các thao tác ghi/cập
> nhật dữ liệu phân tích. Không chạy chúng nếu mục tiêu chỉ là xem dữ
> liệu hiện có. Trước khi thử dataset mới, hãy xác định rõ database và
> thư mục export đích để tránh trộn kết quả giữa các lần thực nghiệm.

## 8. Xem kết quả bằng Obsei Studio và MCP

### Obsei Server (tùy chọn)

Có thể khởi động server local trong VS Code Terminal:

``` powershell
obsei serve --config .\obsei.yaml --host 127.0.0.1 --port 8765
```

Giữ terminal này mở để server chạy ở foreground. Không dùng `0.0.0.0`
nếu chỉ cần demo trên máy cá nhân.

Các endpoint theo CLI Obsei 1.1.1:

-   Health: `http://127.0.0.1:8765/healthz`
-   MCP over HTTP: `http://127.0.0.1:8765/mcp`
-   Obsei Studio: `http://127.0.0.1:8765/studio/`

Mở Terminal thứ hai để kiểm tra health:

``` powershell
Invoke-RestMethod http://127.0.0.1:8765/healthz
```

Server chỉ nên được mở trên localhost cho demo cá nhân. Trước khi dùng
trên môi trường chia sẻ, cần kiểm tra và cấu hình xác thực phù hợp theo
tài liệu Obsei.

### Obsei MCP Plugin trong Codex

Obsei có plugin cho Codex. Các lệnh cài đặt marketplace/plugin đã được
sử dụng trong môi trường demo:

``` powershell
codex plugin marketplace add obsei/obsei
codex plugin add obsei@obsei
```

Sau khi cài đặt, kiểm tra trạng thái plugin/MCP trong Codex. Khả năng
gọi công cụ còn phụ thuộc cấu hình MCP và server/database mà plugin đang
kết nối.

Các công cụ MCP hữu ích trong quá trình demo:

-   `feedback_stats`: thống kê phản hồi.
-   `list_themes`: liệt kê Theme đã khám phá.
-   `search_feedback`: tìm phản hồi theo điều kiện.
-   `get_feedback`: xem chi tiết một phản hồi theo Record ID.

Ví dụ yêu cầu Codex:

> Dùng Obsei MCP để thống kê sentiment và intent của dữ liệu hiện tại,
> liệt kê các Theme và nêu nhận xét ngắn. Không chạy lại pipeline hoặc
> themes, không đọc CSV trực tiếp và không chỉnh sửa database.

**Phân biệt rõ:** MCP dùng để truy vấn/phân tích dữ liệu hiện có qua các
công cụ được cung cấp; việc nạp CSV và chạy pipeline là thao tác
ingest/processing. Chỉ yêu cầu Codex thực hiện các thao tác ghi dữ liệu
khi plugin/công cụ đang kết nối thực sự hỗ trợ và bạn đã xác nhận đúng
database đích.

## 9. Thống kê tham khảo của lần demo 50 phản hồi

Một lần truy vấn Obsei MCP trước đây ghi nhận:

### Sentiment

| Sentiment | Số phản hồi |
|---|---:|
| Neutral | 31 |
| Positive | 11 |
| Negative | 7 |
| Mixed | 1 |
| **Tổng** | **50** |

### Intent

| Intent | Số phản hồi |
|---|---:|
| `other` | 14 |
| `booking` | 10 |
| `takeaway_delivery` | 10 |
| `staff_service` | 5 |
| `branch_experience` | 4 |
| `chatbot_support` | 4 |
| `food_menu` | 3 |
| **Tổng** | **50** |

### Theme

Một lần chạy Theme trước đây tạo 19 Theme nội bộ và dashboard hiển thị 8
Theme đủ điều kiện hiển thị theo thiết lập riêng tư. Số lượng và tên
Theme có thể thay đổi khi chạy lại hoặc thay đổi cấu hình.

> Đây là số liệu của một lần chạy cụ thể trên dataset mô phỏng 50 hội
> thoại, không phải số liệu cố định được đảm bảo cho mọi lần chạy.

## 10. Nạp một CSV mới

Trước khi nạp CSV mới:

1.  Đặt file CSV trong project hoặc thư mục dataset.
2.  Kiểm tra tên cột và xác định trường nội dung phản hồi (`text`), ID,
    ngôn ngữ và thời gian nếu có.
3.  Xác định rõ mục tiêu: thay dataset demo hiện tại hay tạo một lần
    thực nghiệm độc lập.
4.  Kiểm tra `store.path` và `sinks[].config.directory` trong cấu hình
    trước khi chạy.
5.  Giữ bản sao cấu hình và dữ liệu hiện tại. Không thay đường dẫn
    database/export nếu chưa hiểu tác động của việc thay đổi.

Để bảo toàn kết quả demo 50, không chạy dataset mới vào cùng database và
export hiện tại nếu chưa có phương án tách biệt hoặc backup được xác
nhận. Không đưa file database, secrets hoặc dữ liệu nhạy cảm lên GitHub.

## 11. An toàn dữ liệu và Git

Repository dùng `.gitignore` để loại trừ:

-   Virtual environment và cache Python.
-   `.obsei/` và các file `.env`.
-   DuckDB và WAL.
-   Thư mục Parquet export.
-   File CSV mẫu do `obsei init` tạo.

Trước mỗi lần commit/push:

``` powershell
git status --short
git status --short --ignored
git diff --cached --check
```

Không commit `.obsei/secrets.env`, API token, database local hoặc dữ
liệu khách hàng thật. Repository hiện được thiết lập Public; chỉ đẩy dữ
liệu mà bạn có quyền công khai.

## 12. Tài liệu tham khảo

-   [Obsei Documentation](https://docs.obsei.com/)
-   [Obsei Configuration](https://docs.obsei.com/configuration/)
-   [Obsei GitHub Repository](https://github.com/obsei/obsei)
-   [Project Repository --
    obsei-pizza-demo](https://github.com/Cotruongzinh/obsei-pizza-demo)

## 13. Giới hạn của demo

-   Dataset là dữ liệu tổng hợp, không đại diện cho toàn bộ khách hàng
    thực tế.
-   Kết quả sentiment, intent và Theme phụ thuộc model, threshold,
    embedding và chất lượng dữ liệu.
-   Theme có thể chứa nhiều phản hồi chưa hoàn toàn đồng nhất; cần kiểm
    tra các phản hồi đại diện trước khi đưa ra kết luận nghiệp vụ.
-   Các thống kê trong README là kết quả tham khảo của lần chạy demo đã
    ghi nhận, không phải cam kết đầu ra cố định.
