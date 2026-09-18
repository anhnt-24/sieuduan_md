# Lõi 1 đổi vai: có hợp với đồ án không

Cập nhật 19/9/2026. Đi kèm [`NGHIEN-CUU-3-LOI.md`](NGHIEN-CUU-3-LOI.md) (paper và cách thế giới làm).

## Kết luận

Hợp, và hợp hơn thiết kế hiện tại. Chỉ phần **đo năng lực** chuyển sang Elo/CAT; ba trên bốn tác vụ vẫn dùng LLM. Có một điều kiện là ngân hàng câu hỏi, và một câu hỏi mở: đề tài có bắt buộc LLM API làm lõi cho mọi bước không.

## Đổi gì, giữ gì

| Tác vụ | Hiện tại | Sau khi đổi vai |
|---|---|---|
| A. Test đầu vào | Sonnet 5 sinh từng câu lúc test, độ khó ±1 theo đúng/sai | CAT chọn câu từ ngân hàng theo `abs(d − θ)` nhỏ nhất. Ngân hàng do Sonnet 5 sinh trước theo lô, Lecturer duyệt. Chấm short/code vẫn Sonnet 5 |
| B. Ước lượng proficiency | Sonnet 5 đọc lịch sử, trả `p` 0–100 bằng lời | Elo cập nhật `θ` sau mỗi câu, `p = clamp((θ − 1000) / 800, 0, 1) · 100`. Không gọi API |
| C. Roadmap | Sonnet 5 | Giữ nguyên |
| D. Bài tập hằng ngày | Haiku 4.5; cứ 10 bài gọi lại B bằng Sonnet 5 | Giữ Haiku 4.5; phần ước lượng lại thay bằng Elo |
| Lịch ôn | Cố định 1, 3, 7, 14, 30 ngày | Giữ nguyên; SM-2/FSRS vẫn là đề xuất |

Công thức, code và điều kiện của Elo/CAT đã có sẵn ở mục 7.1 (E1) của [README lõi 1](../core/01-adaptive-learning/README.md). Việc đổi vai chủ yếu là chuyển E1 lên mục 3, không phải nghiên cứu mới.

## Vì sao hợp

1. **Con số proficiency đi tiếp sang lõi 3 và tới Recruiter.** Lõi 3 lấy `verified_skills` (skill có `p ≥ 60`) để so khớp. Recruiter xếp ứng viên "theo năng lực thực". Hiện mục 9 lõi 1 tự ghi `p` "không có sai số chuẩn", "lệch ±10", "không cấp chứng chỉ". Dùng số đó để xếp ứng viên là tự mâu thuẫn.
2. **Đã làm gần xong.** `demo.html` lõi 1 đã chạy Elo/CAT thật, có mô phỏng hội tụ. MAE đã đo: khoảng 117 điểm `θ` tại n = 20.
3. **Có paper để bảo vệ.** Bốn nghiên cứu 2024–2026 cho thấy LLM ước lượng năng lực kém mô hình chuyên dụng: Neshaei et al. (EDM 2024), Srivatsa et al. (BEA 2025), Hooshyar et al. (2025), Bhattacharyya et al. (EDM 2026). Nghiên cứu thứ năm của Borchers et al. (2026) cho thấy GPT-4.1 luôn đánh giá người học cao hơn thực tế. Duolingo và Khan Academy đều không dùng LLM để chấm năng lực.
4. **Đề tài vẫn là ứng dụng LLM.** LLM vẫn sinh câu hỏi, chấm câu tự luận và code, sinh roadmap, gợi ý, flashcard. Đây đúng là cách Duolingo English Test làm: máy sinh câu, người duyệt, IRT chấm.
5. **Rẻ và nhanh hơn.** Mỗi lượt test hiện tốn khoảng $0,15 và thêm 20–60 s chờ sinh câu. Sau khi đổi còn khoảng $0,04, chủ yếu là chấm short/code và roadmap, không còn chờ sinh câu. Ngân hàng tốn một lần, khoảng $1,4 cho 240 câu mỗi career.

## Rủi ro và cách xử lý

| Rủi ro | Cách xử lý |
|---|---|
| E1 cần ≥ 8 câu mỗi skill mỗi mức: 6 skill × 5 mức × 8 = 240 câu mỗi career | MVP làm 1 career (Frontend). Sonnet 5 sinh theo lô, Lecturer duyệt mẫu |
| Độ khó ban đầu do LLM hoặc Lecturer gán, chưa hiệu chỉnh | `d = 900 + 180 · difficulty` làm giá trị khởi đầu; Elo tự chỉnh `d` sau ≥ 30 lượt (đã có trong E1) |
| Đồ án không có người dùng thật để hiệu chỉnh | Chứng minh bằng mô phỏng như demo đang làm. Nếu cần thêm, chạy trên bộ dữ liệu công khai (ASSISTments) |
| 15–20 câu cho 6 skill chỉ được 2–4 câu mỗi skill; sai số `θ` khoảng ±100 | Ghi thẳng con số này trong báo cáo. Đo được sai số là lợi thế so với LLM, vốn không đo được |
| Giảng viên dẫn Li et al. 2024 (GPT-4 few-shot ngang DKT) | Paper đó chỉ trên vài bộ dữ liệu; các nghiên cứu 2025–2026 cho kết quả ngược lại |
| Phải đồng bộ lại tài liệu đã viết theo hướng LLM làm lõi (commit `1419702`) | Làm theo checklist bên dưới |

## Câu hỏi mở

Đề tài hoặc giảng viên có yêu cầu "LLM API làm lõi" cho mọi bước không?

- **Không:** đổi vai như trên.
- **Có:** vẫn đổi được. Báo cáo viết "LLM API là lõi của hệ thống; riêng phần đo năng lực dùng Elo/CAT vì cần sai số đo được", dẫn các paper ở mục 3.

## Việc cần làm nếu chốt

- [ ] `core/01-adaptive-learning/README.md`: đoạn mở đầu, mục 2 (lời giải thích sơ đồ), mục 3 (3.1 bảng luồng, 3.2 tác vụ A, 3.3 tác vụ B thành Elo, 3.6 chi phí, 3.7 quy tắc rơi về), mục 7 (E1 chuyển lên mục 3), mục 9 (hạn chế)
- [ ] `core/01-adaptive-learning/so-do-llm.svg` và `.png`: khối B thành Elo, khối A lấy câu từ ngân hàng
- [ ] `core/01-adaptive-learning/bao-cao.pdf`: xuất lại bằng headless Chrome như các bản trước
- [ ] `README.md` dòng 37: bỏ Elo/CAT khỏi danh sách "đề xuất bổ sung"
- [ ] `BAO-CAO-DE-TAI-DEVROOM.md`, `BAO-CAO-SIEU-DO-AN.md`, `BAO-CAO-KY-THUAT-CORE.md`: rà khối "phương án đã chọn" và mục lõi 1, rồi xuất lại PDF
- [ ] `demo.html` lõi 1: không cần sửa, đã chạy Elo/CAT
