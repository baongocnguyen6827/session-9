# BÁO CÁO KIỂM THỬ ĐƠN VỊ (UNIT TESTING) CHO 02 CUSTOM SKILLS
**Dự án:** Agent Giám sát Giá Đối thủ và Tối ưu Khuyến mãi – FPT Retail  
**Mã tài liệu:** AI301_MVP_Unit_Test_Report  
**Kỹ năng kiểm thử:** `price_gap_analyzer` và `margin_guard_calculator`  

---

## 1. Cấu trúc chuẩn 4 thành phần của 2 Custom Skills (Nhiệm vụ 1)

Hai kỹ năng nghiệp vụ được đóng gói độc lập theo chuẩn 4 thành phần trên Google Antigravity:
```text
skills/
├── price_gap_analyzer/
│   ├── SKILL.md (YAML Frontmatter) | instructions/
│   ├── references/price_rules.md (Quy tắc ngưỡng cảnh báo giá)
│   └── scripts/analyzer.py (Script tính độ lệch giá)
└── margin_guard_calculator/
    ├── SKILL.md (YAML Frontmatter) | instructions/
    ├── references/margin_thresholds.md (Khung biên lợi nhuận sàn)
    └── scripts/calculator.py (Script bảo vệ giá sàn)
```

### YAML Frontmatter trong SKILL.md:
```yaml
---
name: margin_guard_calculator
description: Kiểm tra và tính toán biên lợi nhuận gộp nhằm bảo vệ giá sàn, ngăn chặn bán dưới giá vốn cho FPT Retail.
---
```

Tài liệu `references/margin_thresholds.md` quy định ngưỡng sàn: thiết bị công nghệ tối thiểu 10%, phụ kiện tối thiểu 30%; nếu vi phạm sẽ kích hoạt cơ chế can thiệp của con người (Human-in-the-Loop).

---

## 2. Kịch bản Prompting điều phối công cụ tính toán (Nhiệm vụ 2)

Nhằm loại bỏ ảo giác (Hallucination) tính toán số học, System Prompt ép Agent ủy quyền tính cho script:

```text
[VAI TRÒ]: Pricing Execution Agent phụ trách thẩm định giá FPT Retail.
[QUY TẮC CÔNG CỤ BẮT BUỘC]:
1. Tuyệt đối KHÔNG tự nhẩm tính phép trừ giá hay tỷ lệ biên lợi nhuận.
2. Bắt buộc gọi `calculate_price_gap(internal_price, competitor_price)` để xác định độ lệch giá.
3. Bắt buộc gọi `evaluate_margin_guard(proposed_price, cost_price, min_margin)` để thẩm định an toàn tài chính.
4. Nếu `action == "REQUIRE_HITL"`, lập tức dừng quy trình tự động và gửi phiếu trình duyệt lên Giám đốc Thương mại.
```

---

## 3. Mã nguồn Script và Kết quả Kiểm thử đơn vị (Nhiệm vụ 3)

### Mã nguồn hàm tính toán nghiệp vụ (Python Scripts):
```python
def calculate_price_gap(internal_price: float, competitor_price: float) -> dict:
    if internal_price <= 0 or competitor_price < 0:
        raise ValueError("Giá trị đầu vào không hợp lệ.")
    gap_abs = round(internal_price - competitor_price)
    gap_pct = round((gap_abs / internal_price) * 100, 2)
    status = "ALERT_CRITICAL" if gap_pct >= 10.0 else ("ALERT_WARNING" if gap_pct >= 5.0 else "STATUS_NORMAL")
    return {"gap_absolute": gap_abs, "gap_percent": gap_pct, "status": status}

def evaluate_margin_guard(proposed_price: float, cost_price: float, min_margin: float = 0.10) -> dict:
    if proposed_price <= 0 or cost_price <= 0:
        raise ValueError("Giá bán và giá vốn phải lớn hơn 0.")
    margin = round(((proposed_price - cost_price) / proposed_price) * 100, 2)
    is_safe = (margin >= round(min_margin * 100, 2))
    return {"actual_margin_pct": margin, "is_safe": is_safe, "action": "AUTO_APPROVE" if is_safe else "REQUIRE_HITL"}
```

### Bộ ca kiểm thử và Nhật ký thực thi (Execution Log):
Bộ kiểm thử tự động `tests/test_skills.py` bao phủ 10 ca kiểm thử chuẩn và dữ liệu biên:
```text
test_standard_critical_gap ... ok (Đối thủ phá giá 10% -> ALERT_CRITICAL)
test_standard_normal_gap ... ok (Chênh lệch 2% -> STATUS_NORMAL)
test_edge_zero_internal_price ... ok (Giá nội bộ = 0 -> Ném ValueError)
test_edge_negative_competitor_price ... ok (Giá đối thủ âm -> Ném ValueError)
test_edge_equal_price ... ok (Giá bằng nhau 0% -> STATUS_NORMAL)
test_standard_safe_margin ... ok (Lãi gộp 20% >= 10% -> AUTO_APPROVE)
test_standard_unsafe_margin_hitl ... ok (Lãi gộp 4.76% < 10% -> REQUIRE_HITL)
test_edge_loss_selling ... ok (Bán dưới vốn, lỗ -> REQUIRE_HITL)
test_edge_zero_or_negative_price ... ok (Giá bán <= 0 -> Ném ValueError)
test_edge_exact_floor_price ... ok (Lãi gộp đúng 10% -> AUTO_APPROVE)
----------------------------------------------------------------------
Ran 10 tests in 0.000s | OK (10/10 PASS - 100% SUCCESS)
```

**Kết luận:** Cả 02 Custom Skills hoạt động chính xác tuyệt đối, xử lý trơn tru mọi ngoại lệ biên, sẵn sàng tích hợp an toàn vào Agent Orchestrator.

