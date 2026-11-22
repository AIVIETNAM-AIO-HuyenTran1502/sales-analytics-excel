# Dữ liệu

## Lưu ý quan trọng

Thư mục này **chỉ chứa dữ liệu mẫu** — 500 dòng đầu của mỗi bảng, đủ để hiểu cấu trúc và kiểu dữ liệu.

Bộ dữ liệu đầy đủ (9 file CSV, ~40 MB) thuộc về **AI VIETNAM – End-to-End Data Analytics (eDA Conquer 2026)** và không được đăng lại tại đây. Dữ liệu đã được ẩn danh và quy đổi về một đơn vị tiền quy ước, không phải số liệu thật của doanh nghiệp nào.

Toàn bộ số liệu trong [báo cáo](../reports/Sales_Analytics_2017-2022_Report.pdf) và [dashboard](../dashboard/Sales_Analytics_Dashboard.xlsx) được tính trên bộ dữ liệu đầy đủ, không phải trên mẫu trong thư mục này.

---

## 9 bảng nguồn

Giai đoạn dữ liệu: **01/01/2017 – 31/12/2022**.

| Bảng | Số dòng | Mô tả | Cột |
|---|---:|---|---|
| `orders` | 292.531 | Một dòng một đơn hàng | `order_id`, `order_date`, `customer_id`, `zip`, `order_status`, `payment_method`, `device_type`, `order_source` |
| `order_items` | 315.803 | Một dòng một sản phẩm trong đơn | `order_id`, `product_id`, `quantity`, `unit_price`, `discount_amount`, `promo_id`, `promo_id_2` |
| `customers` | 121.930 | Hồ sơ khách hàng | `customer_id`, `zip`, `city`, `signup_date`, `gender`, `age_group`, `acquisition_channel` |
| `products` | 2.412 | Danh mục sản phẩm | `product_id`, `product_name`, `category`, `segment`, `size`, `color`, `price`, `cogs` |
| `geography` | 39.948 | Tra cứu mã vùng | `zip`, `city`, `region`, `district` |
| `promotions` | 50 | Chương trình khuyến mãi | `promo_id`, `promo_name`, `promo_type`, `discount_value`, `start_date`, `end_date`, `applicable_category`, `promo_channel`, `stackable_flag`, `min_order_value` |
| `returns` | 17.454 | Lượt trả hàng | `return_id`, `order_id`, `product_id`, `return_date`, `return_reason`, `return_quantity`, `refund_amount` |
| `web_traffic` | 2.191 | Tổng hợp theo ngày | `date`, `sessions`, `unique_visitors`, `page_views`, `bounce_rate`, `avg_session_duration_sec`, `traffic_source` |
| `sales` | 2.191 | Tổng hợp theo ngày | `Date`, `Revenue`, `COGS` |

### Giá trị phân loại thường gặp

| Cột | Giá trị |
|---|---|
| `order_status` | `delivered`, `returned`, `cancelled`, `created` |
| `order_source` | `direct`, `referral`, `social_media`, `organic_search`, `paid_search`, `email_campaign` |
| `category` | `Streetwear`, `Outdoor`, `Casual`, `GenZ` |
| `region` | `East`, `Central`, `West` (Miền Đông / Trung / Tây) |
| `promo_type` | `percentage`, `fixed_amount` |
| `return_reason` | `wrong_size`, `defective`, `not_as_described`, `changed_mind`, `late_delivery` |
| `device_type` | `desktop`, `mobile`, `tablet` |

---

## Sơ đồ quan hệ

```
                        ┌──────────────┐
                        │   DimDate    │  bảng lịch 01/2017–12/2022
                        │  (tự tạo)    │  cầu nối giữa đơn hàng và
                        └──────┬───────┘  hai bảng tổng hợp theo ngày
             ┌─────────────────┼─────────────────┐
             │                 │                 │
       order_date            date              Date
             │                 │                 │
       ┌─────▼──────┐   ┌──────▼──────┐   ┌──────▼─────┐
       │   orders   │   │ web_traffic │   │   sales    │
       └─────┬──────┘   └─────────────┘   └────────────┘
             │
    ┌────────┼─────────┬──────────────┐
    │        │         │              │
customer_id zip     order_id       order_id
    │        │         │              │
┌───▼─────┐ ┌▼────────┐ │        ┌────▼────┐
│customers│ │geography│ │        │ returns │
└─────────┘ └─────────┘ │        └────┬────┘
                   ┌────▼────────┐    │
                   │ order_items │    │ product_id
                   └────┬────┬───┘    │
            product_id  │    │ promo_id
                   ┌────▼────┐   ┌────▼───────┐
                   │products │◄──┤ promotions │
                   └─────────┘   └────────────┘
```

Mô hình trong Power Pivot gồm **11 bảng nối với nhau qua 12 quan hệ** — 9 bảng nguồn ở trên, cộng bảng lịch `DimDate` và các bảng đã làm sạch (`OrderItems_Clean`, `Products_Clean`, `Returns_Clean`).

---

## Những gì đã thay đổi khi làm sạch

Số dòng sau xử lý: `orders` còn **261.839** (từ 292.531), `order_items` còn **282.671** (từ 315.803). Các bảng khác giữ nguyên.

| Phát hiện | Cách xử lý |
|---|---|
| 27.039 đơn `cancelled` và 3.653 đơn `created` | Loại khỏi phạm vi vì chưa phát sinh giao dịch. Giữ 16.240 đơn `returned` vì đó là đơn đã bán. |
| 33.132 dòng hàng thuộc các đơn vừa bị loại | Loại theo. **Thứ tự thao tác quan trọng**: loại đơn trước, loại dòng hàng sau. Nếu giữ các dòng này, tổng doanh thu và tổng theo tháng lệch nhau 824 triệu. |
| `promo_id` trống ở 59,8% dòng hàng | Gắn nhãn `Không khuyến mãi` và thêm một dòng tương ứng vào bảng `promotions`. Đây là nhóm đối chứng cho Q2 và Q4. |
| `promo_id_2` trống ở 99,98% dòng | Bỏ cột, nên không đo được tác động của khuyến mãi thứ hai trên cùng một dòng hàng. |
| 6 giá trị `order_source` | Gom thành 4 nhóm kênh: `Organic Search`, `Paid Search`, `Email Marketing`, `Direct/Referral` (gồm `direct`, `referral`, `social_media`). |

Sau xử lý, các con số tổng khớp với số tham chiếu trong tài liệu đề bài: doanh thu **6.955.621.865**, biên lợi nhuận **8,87%**, **261.839** đơn.

---

## Nguồn

AI VIETNAM – End-to-End Data Analytics (eDA Conquer 2026), *Sales Analytics Dataset*.
Trang khóa học: <https://eda.aivietnam.edu.vn/>
