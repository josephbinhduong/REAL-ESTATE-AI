# Client Home Search (client-search)

## Mục tiêu

Mỗi ngày, tìm nhà mới phù hợp cho từng nhóm khách mua nhà của Joseph Binh Duong (theo `reports/client-search/clients.csv`), rồi gửi **một thông báo tổng hợp** cho Joseph (không gửi thẳng cho khách — Joseph tự follow up qua phone/SMS/Zalo).

Khác với skill `review-home` (chọn nhà để **quay video**), skill này chọn nhà để **giới thiệu cho khách mua thật** — không chấm điểm "viral angle", mà đánh giá mức độ khớp với tiêu chí từng khách.

## Nguồn dữ liệu

1. **Gmail trước** — tìm email trong 24h qua từ `reply-to-agent@alerts3.realscout.com` và `CRMLS@mlsmatrix.com`. Đọc toàn bộ nội dung, lọc theo tiêu chí từng nhóm khách.
2. **WebSearch bổ sung** — nếu Gmail không đủ (ít hơn ~3 ứng viên cho 1 nhóm), tìm trên Redfin/Zillow/Realtor.com/Trulia. Ghi rõ nguồn nào cho mỗi tin (Gmail = đáng tin hơn, WebSearch = cần xác minh lại trước khi dùng).
3. Nếu Gmail đã stale nhiều ngày (không có mail mới 24h), báo rõ cho Joseph trong report — đây là dấu hiệu cần re-activate RealScout/CRMLS auto-email.

## Danh sách khách

Đọc `reports/client-search/clients.csv`. Mỗi dòng là **một nhóm tìm kiếm** (search group), có thể gồm nhiều người nhận (cột `recipients`, format `Ten:Phone;Ten:Phone`). Các cột lọc:

- `cities` — danh sách thành phố, phân cách `;`
- `price_min` / `price_max` — có thể để trống (không giới hạn)
- `property_type` — thường là SFR (single-family residence)
- `min_beds` / `min_baths`
- `min_lot_sqft` / `preferred_lot_sqft` — dùng cho khách muốn xây ADU; nếu trống nghĩa là không quan trọng
- `adu_interest` — Yes / No / Soft interest
- `special_notes` — yêu cầu riêng (hướng nhà, gần công viên, move-in ready, v.v.) — đọc kỹ, đây thường là yếu tố quyết định ai nên liên hệ trước

## Quy trình mỗi lần chạy

1. Đọc `clients.csv`.
2. Với mỗi nhóm, search Gmail trước, WebSearch sau (theo thứ tự ở trên).
3. Loại nhà không đạt: sai loại property, ngoài khoảng giá, dưới min beds/baths, (nếu có) dưới min_lot_sqft, hoặc trạng thái Pending/Sold/Off Market.
4. So sánh với `reports/client-search/<group_id>/history.csv` — **không lặp lại nhà đã báo trong 7 ngày gần nhất**, trừ khi giá hoặc status đổi (Price Drop, Status Change).
5. Với các khách ADU (`adu_interest = Yes`): ưu tiên nhà có sân sau đủ rộng (không chỉ tổng lot size) — nêu rõ đây là suy đoán từ mô tả/ảnh, chưa xác minh zoning/setback/utilities/permit.
6. Ghi báo cáo từng nhóm vào `reports/client-search/<group_id>/YYYY-MM-DD.md`:
   - Danh sách nhà đạt tiêu chí: địa chỉ, giá, bed/bath, sqft/lot, status, DOM, điểm khớp tiêu chí (ngắn gọn, không cần thang 100 như review-home), lý do phù hợp, điều cần xác minh, nguồn.
   - Nếu không có nhà nào đạt: ghi rõ "Không có ứng viên nào đạt tiêu chí hôm nay" — không được bịa nhà.
7. Cập nhật `reports/client-search/<group_id>/history.csv` với các nhà vừa báo.
8. Tổng hợp **một thông báo duy nhất** cho Joseph (không gửi cho khách): theo từng nhóm, liệt kê người nhận (kèm số phone) và top nhà nên gửi hôm đó, hoặc "không có nhà mới" nếu không có.
9. Nếu nhóm `quy-cuong-diep` có kết quả, nhắc rõ thứ tự gửi: **Chị Diệp xem trước, rồi mới gửi Chị Quý & Anh Cường**.

## Quy tắc nghề nghiệp

- Không xác nhận ADU được phép xây chỉ vì lot rộng — luôn ghi "cần kiểm tra zoning, setback, utilities, permit".
- Không đoán hướng nhà (Bắc/Đông/Nam/Tây) nếu nguồn không ghi rõ — ghi "chưa xác minh, cần hỏi agent hoặc xem nhà".
- Dùng ngôn ngữ Fair Housing trung lập.
- Phân biệt rõ dữ kiện đã xác minh (Gmail/MLS) và suy đoán/WebSearch chưa xác minh.
