# Reflection — Day 17 (≤ 200 words)

Answer briefly, in your own words. This is graded on reasoning, not length.

1. **The flywheel.** Day 13 emitted agent traces; today you turned them into an
   eval set and DPO pairs that Day 22 will train on. Which step in
   `traces → Bronze → datasets` would break most silently in production if you
   got it wrong — and how would you detect it?

2. **Decontamination.** Your run dropped 2 of 3 preference pairs because their
   prompts were in the eval set. What concretely goes wrong if you *skip* this
   step and train on those pairs? How would the lie show up in your metrics?

3. **Point-in-time.** The naive join leaked a future `lifetime_spend` into the
   training row. Describe one feature in a system you know that would be
   dangerous to join without an `ASOF`/point-in-time guard.

4. **Graph vs vector.** From `kg_demo.py`, name one question the knowledge graph
   answers well that flat chunk retrieval (`embed.py`) would struggle with, and
   one where the graph is overkill.

_Trả lời._

1. **Flywheel.** Bước **decontamination** dễ hỏng ngầm nhất. Nhãn `split='eval'` gán
   sai hoặc `_norm` lệch sẽ để prompt eval lọt vào tập train mà **không báo lỗi** —
   run vẫn in "ALL PASS". Phát hiện bằng assertion CI: giao prompt eval và train phải
   rỗng (∩ = ∅), kèm canary đếm dòng (run của em: 3 cặp thô → 1 cặp sạch).

2. **Decontamination.** Bỏ bước này là train DPO trên 2 cặp có prompt nằm **trong**
   eval set. Model học thuộc đáp án bị chấm, nên điểm eval offline tăng còn tổng quát
   hóa thật thì không — khoảng cách train–serve nới rộng: benchmark cao nhưng metric
   production đứng yên, eval không còn khớp đánh giá của người.

3. **Point-in-time.** `lifetime_spend`, hoặc `total_defaults_to_date` (mô hình chống
   gian lận). Join "giá trị mới nhất" làm rò một lần vỡ nợ xảy ra *sau* sự kiện
   vào dòng ngày sớm hơn — 2/3 dòng của em bị rò (300 thay vì 50/120). Đẹp offline,
   vô dụng khi chạy thật; ASOF sửa được.

4. **Graph vs vector.** Graph thắng câu 2-hop "widget ship từ đâu?"
   (widget→accessory→Hà Nội) — không chunk nào chứa cả hai fact. Graph là thừa với
   câu "bảo hành gadget bao lâu?" — một chunk trả lời đủ, vector lookup rẻ hơn.
