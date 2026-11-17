# BẢN THIẾT KẾ AGENT BLUEPRINT MVP THEO KHUNG GIWO
**Tên dự án:** Agent Giám sát Giá Đối thủ và Tối ưu Khuyến mãi (Competitive Pricing Agent)  
**Doanh nghiệp khảo sát:** Chuỗi Bán lẻ Công nghệ FPT Retail  
**Lĩnh vực:** Bán hàng (Sales) & Phân tích dữ liệu (Data Analytics)  
**Mã tài liệu:** AI301_MVP_Blueprint  

---

## 1. Bối cảnh và Nỗi đau vận hành (Nhiệm vụ 1)

FPT Retail đối mặt với áp lực cạnh tranh giá gay gắt từ Thế Giới Di Động và CellphoneS. Quy trình định giá thủ công hiện tại gặp 2 nút thắt lớn:

- **Thu thập dữ liệu chậm trễ:** Nhân viên phải tra cứu thủ công giá và khuyến mãi của 500 SKU mỗi ngày trên web đối thủ, tốn 4-5 giờ/ngày và phản ứng trễ 12-24 giờ so với thị trường.
- **Rủi ro xói mòn lợi nhuận:** Khi đối thủ hạ giá sốc, nhân viên thường giảm giá theo cảm tính mà không kịp đối soát giá vốn (COGS), dẫn đến việc bán lỗ biên lợi nhuận.

---

## 2. Đặc tả Blueprint MVP theo Khung GIWO (Nhiệm vụ 2)

Hệ thống Agent được thiết kế trên Google Antigravity theo khung chuẩn GIWO:

### G - Goal (Mục tiêu SMART)
Tự động thu thập giá bán 500 SKU từ 2 đối thủ vào 07:00 hàng ngày; phân tích chênh lệch và đề xuất mức giá tối ưu (bảo vệ biên lợi nhuận gộp >= 10%) trước 08:30 sáng, giúp giảm 85% thời gian thủ công và loại bỏ việc bán dưới giá vốn.

### I - Inputs (Khế ước dữ liệu đầu vào)
- `internal_catalog`: Danh mục 500 SKU gồm `sku_id`, `cost_price` (giá vốn), `current_price`, `stock_level` và `min_margin` (10%).
- `competitor_sources`: Danh sách URL sản phẩm đối thủ và chính sách thu thập dữ liệu an toàn.

### W - Workflow (Luồng tác tử xử lý)
1. **Scraping Subagent:** Bóc tách giá niêm yết, flash-sale và quà tặng từ trang đối thủ.
2. **Analysis Subagent:** Ghép mã SKU, tính độ lệch giá (Price Delta) so với giá vốn nội bộ.
3. **Pricing Orchestrator:** Tính giá bán tối ưu bảo toàn biên lãi sàn; kích hoạt điểm dừng phê duyệt (Breakpoint) nếu mức giảm vượt quá 8%.

### O - Outputs (Định dạng sản phẩm bàn giao)
Báo cáo đề xuất giá định dạng Markdown/JSON xuất bản lúc 08:30 sáng:
- Bảng tổng hợp giá khuyến nghị kèm biên lợi nhuận dự kiến từng SKU.
- Danh sách cảnh báo khẩn cấp (Critical Alerts) với các SKU bị đối thủ bán phá giá.

---

## 3. Thiết lập Hàng rào An toàn - Guardrails (Nhiệm vụ 3)

Hệ thống áp dụng 2 hàng rào kiểm soát an toàn bắt buộc:

1. **Hàng rào Bảo vệ Giá sàn Tuyệt đối (Hard Price Floor Guardrail):**  
   Nghiêm cấm Agent đề xuất giá thấp hơn công thức: `Min_Price = Cost_Price * 1.10`. Khi đối thủ phá giá sâu, Agent chỉ đề xuất tặng quà dịch vụ hoặc chuyển quyền duyệt lên Giám đốc Thương mại (Human-in-the-Loop).
2. **Hàng rào Kiểm chứng Dữ liệu Thực tế (Data Groundedness Guardrail):**  
   Mọi dữ liệu giá đối thủ phải gắn kèm nhãn thời gian và bằng chứng HTML trích xuất. Cấm Agent tự suy diễn hoặc ước lượng giá khi nguồn dữ liệu bị gián đoạn.

