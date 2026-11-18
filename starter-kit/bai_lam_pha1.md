## 4.1. Bảo vệ dữ liệu và chống ảo giác

### a) Ghi chú đã ẩn danh hoá

**GHI CHU DA AN DANH**

```text
Khán giả [TEN_KHACH_HANG], số điện thoại [SDT_CA_NHAN], số CCCD [SO_CCCD_CA_NHAN], email quan.nguyen@example.com, đặt 3 vé gói SVT trận bán kết Cúp Sao Vàng ngày 18/12/2026 cho 2 người lớn và 1 trẻ em 9 tuổi. Khán giả đã chuyển khoản đặt cọc 1.500.000 đồng. Khán giả yêu cầu 3 ghế ngồi cạnh nhau gần lối đi.
```

### b) Hai câu lệnh chống ảo giác

**CAU LENH**

```text
Câu 1: Dựa trên file chính sách hoàn/hủy vé mà tôi gửi hãy trả lời câu hỏi trên của tôi. Nếu các thông tin không có trong chính sách thì không tự bịa chính sách mà trả lời trực tiếp là "Chính sách về điều khoản này không tồn tại"

Câu 2: Khi bạn trả lời cho tôi thì ghi rõ câu trả lời lấy từ mục hoặc điều khoản nào trong file chính sách hoàn hủy đó
```

## 4.2. Trích xuất tin nhắn đặt vé ra JSON

**PROMPT**

```text
C-R-E-A-T-E

[Context] Bạn là một chuyên gia phân tích tin nhắn khách hàng, bạn có nhiệm vụ chuyển các tin nhắn thành định dạng JSON

[Request] Bạn hãy chuyển dữ liệu TN1, TN2, TN3 trong thẻ <tin_nhan> thành một mảng JSON 3 đối tượng, đúng thứ tự, đúng 6 trường như sau  "ma_goi", "ngay_su_kien", "nguoi_lon", "tre_em", "loai_ve", "ghi_chu", như tin nhắn mẫu

[Explanation] Khi chuyển thành JSON bạn cần bám theo quy tắc chuyển hóa trong thẻ <quy_chuan> . Bạn có thể xem qua thử 2 Ví dụ mẫu mà tôi đã thực hiện trong thẻ <vi_du>
    <quy_chuan>
        BẢNG MÃ GÓI
        SVT – Sân Vận Động: vé trận đấu
        FZ – Fan Zone: khu cổ vũ
        VIP – Khán đài VIP

        QUY TẮC CHUẨN HOÁ (5 quy tắc)
        1. ma_goi lấy theo bảng mã gói.
        2. ngay_su_kien dạng YYYY-MM-DD; tin nhắn không ghi năm thì lấy năm 2026; không có ngày cụ thể thì null.
        3. Khách không nhắc trẻ em thì tre_em là 0 và mọi khách là người lớn.
        4. loai_ve chỉ nhận "thuong", "vip" hoặc null.
        5. ghi_chu chỉ ghi yêu cầu đặc biệt về chỗ ngồi, hỗ trợ di chuyển hoặc ăn uống; không có thì null.
        
        Câu trả lời không chào hỏi, không giải thích chỉ có dữ liệu JSON trả về
    </quy_chuan>

    <vi_du>
        Tin nhắn mẫu 1:
        Đặt giúp mình 2 vé VIP trận chung kết ngày 15/11/2026, 2 người lớn và 1 bé. Cần chỗ ngồi cạnh lối đi.
        Kết quả đúng:
        {"ma_goi": "VIP", "ngay_su_kien": "2026-11-15", "nguoi_lon": 2, "tre_em": 1, "loai_ve": "vip", "ghi_chu": "chỗ ngồi cạnh lối đi"}

        Tin nhắn mẫu 2:
        Cho mình một vé Fan Zone, ngày nào cũng được, mình đi một mình.
        Kết quả đúng:
        {"ma_goi": "FZ", "ngay_su_kien": null, "nguoi_lon": 1, "tre_em": 0, "loai_ve": "thuong", "ghi_chu": null}
    </vi_du>

[Action] Đọc dữ liệu của TN Khách hàng mà tôi đưa => Chuẩn hóa về đúng các trường dữ liệu => Trả về đúng định dạng JSON

[Tone] Chuyên nghiệp, ngắn gọn, xúc tích

[Extras] Định dạng đầu ra chỉ là dữ liệu JSON

    <tin_nhan>
        TN1: Mình muốn đặt gói Sân Vận Động trận tứ kết ngày 20/12 nha, 2 người lớn với 1 bé 8 tuổi, vé thường.

        TN2: Cho 3 người đi Fan Zone trận ngày 05/01/2027 ạ. Một người cần hỗ trợ xe lăn.

        TN3: Trận chung kết Cúp Sao Vàng tháng 12 còn vé không ạ? Nhà em 4 người lớn, có một người khiếm thị cần hỗ trợ tìm chỗ, lấy vé VIP.
    </tin_nhan>
```

## 4.3. Báo giá vé bằng Chain-of-Thought

### a) Prompt

**PROMPT**

```text
[Role]
Bạn là một chuyên viên báo giá và kế toán dịch vụ chính xác, cẩn trọng. Nhiệm vụ của bạn là tính toán tổng chi phí cho yêu cầu báo giá của khách hàng dựa trên bảng giá và các quy tắc được cung cấp.

[Request]
Hãy đọc kỹ dữ liệu trong các cặp thẻ phân tách dưới đây:

<bang_gia>
SVT – Sân Vận Động: 1.200.000
FZ – Fan Zone: 800.000
VIP – Khán đài VIP: 2.500.000
</bang_gia>

<quy_tac>
1. Trẻ em từ 5 đến 11 tuổi tính 70% giá người lớn. Trẻ dưới 5 tuổi miễn phí.
2. Nhóm từ 5 người lớn trở lên được giảm 5% trên tiền vé (tiền người lớn + tiền trẻ em).
3. Phụ thu khu ghế ưu tiên: 300.000 đồng/người. Phụ thu không được giảm giá.
4. Tổng tiền = tiền vé sau giảm + phụ thu.
5. Câu trả lời không chào hỏi, không giải thích chỉ có dữ liệu trả về
</quy_tac>

<yeu_cau_bao_gia>
Nhóm cổ động viên đi xem trận bán kết, mua gói SVT, gồm 5 người lớn, 2 trẻ em (8 tuổi và 10 tuổi) và 1 bé 3 tuổi. Có 1 người lớn chọn khu ghế ưu tiên.
</yeu_cau_bao_gia>

[Tone] Chuyên nghiệp, ngắn gọn, xúc tích

[Instructions]
Hãy suy luận và trình bày chi tiết từng bước tính toán theo đúng thứ tự 6 bước bắt buộc sau đây. Ở mỗi bước, phải viết rõ phép tính cụ thể trước khi ra kết quả của bước đó:
1. **Tiền người lớn:** [Ghi phép tính] = [Kết quả]
2. **Tiền trẻ em:** [Ghi phép tính] = [Kết quả]
3. **Tiền vé (tạm tính):** [Ghi phép tính: Tiền người lớn + Tiền trẻ em] = [Kết quả]
4. **Giảm giá nhóm:** [Đối chiếu quy tắc, ghi phép tính mức giảm] = [Kết quả]
5. **Phụ thu:** [Đối chiếu quy tắc, ghi phép tính phụ thu] = [Kết quả]
6. **Tổng tiền:** [Ghi phép tính: Tiền vé - Giảm giá nhóm + Phụ thu] = [Kết quả]

[RESULT: <tổng tiền> đồng]. Không viết thêm bất cứ ký tự nào phía sau.
[Extras] Chỉ xuất dữ liệu theo đúng mẫu trên. Kết thúc ngay tại dòng [RESULT], không thêm câu hỏi gợi ý, lời chào hay bất kỳ nội dung nào khác.
```

## 4.4. Dây chuyền xử lý khiếu nại (Prompt Chaining)

### a) Prompt Trạm 1

**PROMPT**

```text
[Context] Bạn là một chuyên gia phân tích tin nhắn khách hàng, bạn có nhiệm vụ chuyển các tin nhắn thành định dạng JSON

```

### b) Hai điều cần kiểm tra ở JSON đầu ra của Trạm 1

1. (Viết câu trả lời vào đây – xoá dòng này)

2.

### c) Câu lệnh viết tiếp

**CAU LENH**

```text
(Viết câu lệnh vào đây – xoá dòng này)
```
