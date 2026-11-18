> ## So sánh Trade-off: 2 giải pháp prompt (cùng bài toán xử lý đơn hàng)

> ### Bước 1: Hai prompt đề xuất

> #### Giải pháp 1: Đơn giản (1 prompt duy nhất)

> markdown

```
Bạn là nhân viên bán hàng. Đọc tin nhắn khách bên dưới, kiểm tra tồn kho
và trả lời khách bằng tiếng Việt, ngắn gọn.

Danh mục & tồn kho: {{catalog}}
Tin nhắn: {{message}}
```

> #### Giải pháp 2: Chi tiết (Prompt Chaining \+ CoT \+ xử lý lỗi)

> markdown

```
# P1 – Trích xuất (chỉ JSON)
- Trường không có trong tin nhắn → null, KHÔNG đoán.
- Số lượng mơ hồ → qty = null. Mỗi null ghi vào "errors".

# P2 – Kiểm tra + tính tiền (CoT, ghi vào "reasoning")
B1. Item đủ sku/size/qty? → B2. Tồn kho >= qty? Không thì gợi ý size khác
→ B3. SĐT đủ 10 số? → B4. Mã giảm giá có trong chính sách?
→ B5. Chỉ tính tiền item hợp lệ → B6. Chốt status: OK / NEEDS_INFO / REJECTED

# P3 – Soạn phản hồi
- Chỉ dùng dữ liệu trong JSON của P2, không bịa. Tối đa 120 từ.
- NEEDS_INFO → liệt kê từng điểm cần khách bổ sung.
```

> ### Bước 2: Bảng so sánh Trade-off

| Tiêu chí | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) |
| ----- | ----- | ----- |
| Độ dễ hiểu / dễ viết | Cao: 1 prompt, viết trong 2 phút | Trung bình: 3 prompt \+ sơ đồ chuỗi |
| Tốn token / chi phí | Ít: 1 lần gọi | Nhiều hơn: 3 lần gọi, thêm “reasoning” |
| Độ trễ | Thấp | Cao hơn (3 lượt gọi nối tiếp) |
| Chặn lỗi biên (thiếu size, SĐT sai, hết hàng, mã sai) | Trung bình: dễ bỏ sót, dễ tự đoán | Cực tốt: có điều kiện rõ cho từng lỗi |
| Chống bịa thông tin (hallucination) | Yếu | Mạnh: có quy tắc “null, không đoán” |
| Độ chính xác tính tiền | Dễ sai | Cao: tính ở bước riêng, theo từng bước |
| Dễ gỡ lỗi | Khó (không biết sai ở đâu) | Dễ: xem JSON từng bước |
| Dễ bảo trì / mở rộng | Khó (sửa 1 chỗ ảnh hưởng tất cả) | Dễ: sửa riêng từng bước |
| Xử lý đứt gãy | Không có | Có `status` để chuyển người hoặc hỏi lại |

> ### Bước 3: Lựa chọn và chạy thử

> #### Chọn: Giải pháp 2

Lý do:

* Đơn hàng liên quan đến tiền và tồn kho, nên sai sót gây thiệt hại thật. Chi phí token tăng thêm là nhỏ so với rủi ro.  
* Bài toán yêu cầu xử lý dữ liệu bẩn; Giải pháp 1 không có cơ chế chặn.  
* Khi nào dùng Giải pháp 1: chỉ phù hợp tin nhắn hỏi đáp đơn giản (giờ mở cửa, hỏi giá), không tính tiền.

> #### Dữ liệu thử (cùng bẫy ở bài trước)

> *“Chào shop, mình lấy 2 áo thun trắng size XL và 1 quần jean nhé. SĐT 09123\. Mình có mã GIAM10.”*  
>  (Tồn: AT01 XL \= 0; mã hợp lệ duy nhất là `SALE10`)

> #### Kết quả mô phỏng

Giải pháp 1:

> “Dạ shop xác nhận đơn 2 áo thun trắng XL và 1 quần jean, tổng 650.000đ, áp mã GIAM10 giảm 10% còn 585.000đ. Shop sẽ gọi số 09123 để giao hàng nhé\!”

Sai ở 4 điểm: XL đã hết hàng nhưng vẫn xác nhận; tự chọn size quần; chấp nhận SĐT 5 số; áp mã không tồn tại.

Giải pháp 2:

> “Chào bạn, shop cần bạn xác nhận thêm:

> * Áo thun size XL đã hết, shop còn L và M, bạn chọn size nào?  
> * Quần jean bạn lấy size nào (28-32)?  
> * SĐT chưa đủ 10 số, bạn gửi lại giúp shop nhé.  
> * Mã GIAM10 không áp dụng được; shop có mã SALE10 (-10%).”

Bắt đủ 4 lỗi, không tính tiền khi dữ liệu chưa hợp lệ, không bịa thông tin.

> 

