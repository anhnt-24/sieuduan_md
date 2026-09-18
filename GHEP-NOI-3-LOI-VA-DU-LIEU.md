# DevRoom — Ghép nối 3 lõi, thuật toán và dữ liệu

Cập nhật 19/09/2026. Nguồn dữ liệu ở mục 6 đã kiểm tra tồn tại và license vào ngày này.

## 1. Kết luận chính

Ba lõi (học thích ứng, phỏng vấn thử, CV–JD) nối nhau qua hai thứ dùng chung: mã kỹ năng `skill_id` và bảng đề xuất `suggestions`. Thiếu một trong hai thì DevRoom chỉ là ba app rời dùng chung đăng nhập.

- Phương án đã chốt (commit `1419702`): LLM API làm lõi; thuật toán tất định là đề xuất bổ sung.
- Không cần train model. Việc cần làm: golden set để đo, hiệu chỉnh ngưỡng, mô phỏng Elo.
- Repo chưa có file dữ liệu nào. Khoảng một nửa lấy được từ nguồn công khai tiếng Việt/tiếng Anh; phần còn lại tự tạo.
- Điểm nối phỏng vấn thử → năng lực đang hở trong phương án LLM (mục 3).

## 2. Kiến trúc triển khai

Một frontend Next.js gọi một backend FastAPI. Trình duyệt không bao giờ gọi thẳng database hay Claude.

| Tầng | Thành phần | Vai trò |
|---|---|---|
| Giao diện | Next.js, Monaco, React Flow | REST + WebSocket cho mock interview |
| Backend | FastAPI (Python 3.12) | Router, service, kiểm JSON trả về từ LLM |
| Việc nền | Celery + Redis | Bóc CV, tính embedding, tính lại match |
| Dữ liệu có cấu trúc | PostgreSQL + pgvector | Mọi bảng + vector embedding |
| File | Object storage private | CV gốc, audio phỏng vấn (giữ 30 ngày), HTML JD thô (giữ 14 ngày) |
| Dịch vụ ngoài | Claude Sonnet 5 / Haiku 4.5, Whisper (Groq), edge-tts, Piston, bge-m3 | LLM, giọng nói → chữ, chữ → giọng nói, chạy code, embedding |

Chi tiết ở `BAO-CAO-KY-THUAT-CORE.md` §5.1.

## 3. Ba luồng và điểm nối

```text
L1 HỌC THÍCH ỨNG              L2 CV–JD                  L3 PHỎNG VẤN THỬ
test 15–20 câu                CV → cv_skills            câu hỏi theo skill mục tiêu
→ skill_states → gap          JD → jd_skills            → chấm STAR từng câu
→ roadmap → bài hằng ngày     → match% + gap            → skill yếu

                 gap(JD) ──────►  E10  ◄────── skill yếu
                          suggestions (pending)
                                   │ candidate bấm accept
                                   ▼
                     roadmap_items (source = ai_suggested) → quay về L1
```

| Điểm nối | Chiều | Dữ liệu chảy | Trạng thái |
|---|---|---|---|
| A | L1 → L2 | `skill_states` có p ≥ 60 thành `verified_skills` khi so khớp | Có (`core/03` §3.4) |
| B | L2 → L3 | `jd_skills` của JD mục tiêu quyết định câu hỏi phỏng vấn | Có |
| E10 | L2 + L3 → L1 | gap từ JD + skill yếu từ mock → `suggestions` → accept → `roadmap_items` | Có |
| C | L3 → L1 | điểm STAR cập nhật `skill_states` | **Chưa có** trong phương án LLM, chỉ nằm ở `core/02` §7.3 |

Hệ quả của điểm C bị hở: phỏng vấn kém nhưng năng lực không đổi, Recruiter vẫn thấy "verified" ở skill vừa trượt.

## 4. Thuật toán

### 4.1 Đang áp dụng: kỹ thuật LLM + luật ở backend

| Kỹ thuật | Dùng ở đâu |
|---|---|
| Ép đầu ra đúng JSON Schema (structured output) | Mọi lần gọi LLM |
| Chấm theo rubric có mô tả mốc 1/3/5 | Chấm STAR, chấm CV |
| LLM làm giám khảo so khớp (LLM-as-judge) | Match CV–JD |
| Tool `lookup_skill` | Bóc CV khi từ điển > 300 mục |
| RAG nhẹ: lấy top-k câu từ ngân hàng theo `skill_id` | Mở phiên mock |
| Chấm 2 lần khi confidence < 0,6, lấy trung bình | Mock interview |
| Cache prompt, Batch API | Giảm chi phí |
| Luật backend (không LLM) | Kiểm `module_id ∈ catalog`; `total_score = Σ STAR / (n × 20) × 10`; chống trùng đề xuất (UNIQUE pending, cooldown 14 ngày) |

### 4.2 Thuật toán tất định (đang là đề xuất bổ sung)

**Lõi 1 — Học thích ứng** (`core/01-adaptive-learning/README.md` §7)

| Thuật toán | Làm gì | Cốt lõi |
|---|---|---|
| Elo + CAT | Đo năng lực θ theo skill | `E = 1/(1+10^((d−θ)/400))`, `θ' = θ + K(S−E)`, K = 64 → 32 → 16; chọn câu có \|d−θ\| nhỏ nhất; sàn đoán mò g = 0,25 |
| Sắp xếp topo Kahn trên DAG tiên quyết | Xếp thứ tự học phần | `module_score = Σ gap_score + 0,3 · unlock_count` |
| Chọn bài theo dải Elo | Bài hằng ngày | \|d − θ\| ≤ 100, lấy mẫu có trọng số không hoàn lại |
| SM-2 | Lịch ôn flashcard | `ef' = max(1,3; ef + 0,1 − (5−q)(0,08 + (5−q)·0,02))` |

**Lõi 2 — Phỏng vấn thử** (`core/02-mock-interview/README.md` §7)

| Thuật toán | Làm gì |
|---|---|
| Chấm STAR bằng luật (regex + từ khoá) | Đường dự phòng khi LLM lỗi |
| Luật hỏi thêm | Hỏi thêm khi `A + R ≤ 3` hoặc câu trả lời < 60 ký tự, tối đa 2 lần/câu |
| Cập nhật θ bằng Elo | `S = STAR/20`, K = 16 — chính là điểm nối C |
| Chọn học phần theo điểm | Ít giờ nhất, đã đủ tiên quyết |

**Lõi 3 — CV và JD** (`core/03-cv-jd-matching/README.md` §7)

| Thuật toán | Làm gì | Cốt lõi |
|---|---|---|
| Chuẩn hoá skill 3 tầng | Chuỗi thô → `skill_id` | Trùng tên → trùng alias → cosine bge-m3 ≥ 0,85 (kNN trên index HNSW) + blocklist cặp dễ nhầm |
| Chấm CV theo luật R0–R10 | Điểm hình thức | `score_final = 0,5 · rule + 0,5 · llm` |
| Công thức so khớp | % khớp CV–JD | `0,7 · skill_match + 0,3 · cosine`; skill cùng nhánh cha được 0,5 |
| Khử trùng JD | Gộp tin trùng | Hash nội dung → cosine ≥ 0,95 + trigram tiêu đề ≥ 0,8 |
| Điểm cho Recruiter | Xếp ứng viên | `match + 0,1 · mock / 10` |

**Điểm nối E10:** `score = weight_jd · (1 − p/100) + 0,2 · n_weak`, tối đa 5 đề xuất mỗi lần.

## 5. Dữ liệu và train

### 5.1 Không train model

- Claude, bge-m3, Whisper, TTS dùng nguyên. Chỉ prompt + vài ví dụ mẫu + JSON schema, không fine-tune.
- Thay cho train: golden set để đo, hiệu chỉnh ngưỡng (cosine 0,85; độ khó câu hỏi tự hiệu chỉnh sau ≥ 30 lượt), mô phỏng Elo bằng học viên ảo (có sẵn trong `core/01-adaptive-learning/demo.html`).
- Train thật chỉ làm khi có nhãn từ người dùng thật (BKT, LightGBM, FSRS). Ngoài phạm vi đồ án.

Chọn LLM làm lõi chính là cách né bài toán thiếu dữ liệu.

### 5.2 Dữ liệu cần có (phương án LLM)

| Dữ liệu | Lõi | Bắt buộc? | Hiện trạng |
|---|---|---|---|
| Từ điển skill + alias | Cả 3 | Có — thiếu là gãy trục `skill_id` | Chưa có |
| `career_skills` (skill thuộc career, trọng số) | L1, L3 | Có | Chưa có |
| Catalog học phần + quan hệ tiên quyết | L1, E10 | Có | Chưa có |
| Ngân hàng câu hỏi test | L1 | Không — LLM sinh được; bắt buộc nếu dùng Elo cho test đầu vào (≥ 8 câu/skill/mức) | Chưa có |
| Bài code + test case | L1 | Chỉ khi làm sandbox | Chưa có |
| Câu hỏi phỏng vấn | L2 | Không — LLM sinh được | — |
| Rubric STAR | L2 | Có | Đã có trong README |
| JD | L3 | Có | Chưa có |
| CV mẫu | L3 | Có (để test) | Chưa có |

### 5.3 Golden set

Không train, nhưng phải chứng minh LLM làm đúng. Hội đồng sẽ hỏi "sao biết LLM chấm đúng?".

| Bộ | Tài liệu kỹ thuật đề xuất | Mức tối thiểu | Đo |
|---|---|---|---|
| Bóc skill từ CV/JD | 100 CV + 200 JD | 30 CV + 50 JD | F1, recall |
| Chấm CV | 60 CV | 30 CV | MAE, Spearman |
| Chấm STAR | 50 câu trả lời | 30 câu trả lời | MAE, kappa |
| Map skill | 300 chuỗi | 150 chuỗi (chỉ khi dùng tầng cosine) | Accuracy |

Mức tối thiểu đủ để có số trong báo cáo, nhưng sai số rộng; cần ghi rõ. Chi phí API ước lượng: gán nhãn seed $3–8, mỗi lần chạy eval 100 mẫu bằng Sonnet ≈ $0,6.

## 6. Nguồn dữ liệu tiếng Việt / tiếng Anh, chủ đề CNTT

### 6.1 Lấy sẵn được

| Dữ liệu | Nguồn | Ngôn ngữ | License | Ghi chú |
|---|---|---|---|---|
| Từ điển skill | [O\*NET Software Skills](https://www.onetcenter.org/database.html) (cột Hot Technology) | EN | CC BY 4.0 | Có React, Docker, Kubernetes. Lọc 150–300 skill IT |
| Danh sách công nghệ phổ biến | [Stack Overflow Developer Survey](https://survey.stackoverflow.co) | EN | ODbL (share-alike) | Chỉ tham khảo tên công nghệ, không phải cây skill |
| JD tiếng Việt | [VietJobs](https://github.com/VinNLP/VietJobs) (dữ liệu trên HF `dinhieufam/VietJobs`) | VI | MIT / CC BY 4.0 | 1.906 JD IT từ TopCV, 7/2025 — không cần crawler |
| JD tiếng Anh | [Kaggle arshkon/linkedin-job-postings](https://www.kaggle.com/datasets/arshkon/linkedin-job-postings) | EN | CC BY-SA 4.0 | 124k tin mọi ngành, lọc IT |
| Câu hỏi JavaScript | [lydiahallie/javascript-questions](https://github.com/lydiahallie/javascript-questions) | VI + EN | MIT | 155 câu, có bản dịch tiếng Việt `vi-VI/` |
| Câu hỏi React | [sudheerj/reactjs-interview-questions](https://github.com/sudheerj/reactjs-interview-questions) | EN | MIT | ~444 câu |
| Câu hỏi frontend tiếng Việt | [duyet/vietnamese-frontend-interview-questions](https://github.com/duyet/vietnamese-frontend-interview-questions) | VI | MIT | |
| CV tiếng Anh | [Kaggle snehaanbhawal/resume-dataset](https://www.kaggle.com/datasets/snehaanbhawal/resume-dataset) | EN | CC0 | 2.484 CV, chỉ 120 CV nhóm INFORMATION-TECHNOLOGY |
| Giọng nói → chữ tiếng Việt | PhoWhisper (VinAI) | VI | BSD-3 | So sánh với Whisper |
| Embedding | [BAAI/bge-m3](https://huggingface.co/BAAI/bge-m3) | 100+ ngôn ngữ | MIT | Vector tiếng Việt và tiếng Anh chung một không gian |

Bỏ ESCO: không có tiếng Việt, không có React/Docker/Kubernetes. Thay bằng O\*NET.

Tham khảo thêm, license chưa rõ hoặc hạn chế (chỉ dùng để đo nội bộ): HF `tinixai/vietnamese-job-descriptions` (CC BY-NC), Kaggle `quocnguyenx43/vietnamese-job-description-dataset`, Kaggle `halhuynh/it-jobs-dataset` (ITviec 2021).

### 6.2 Phải tự tạo

| Dữ liệu | Lý do | Cách làm nhanh |
|---|---|---|
| CV tiếng Việt | Không có bộ công khai (dữ liệu cá nhân) | 10 CV thật có đồng ý, ẩn danh + 20 CV do LLM sinh |
| Câu trả lời phỏng vấn | Không có bộ nào chấm STAR | Tự viết 30 câu, cố tình thiếu từng chiều S/T/A/R |
| Catalog học phần + lộ trình | Nghiệp vụ của Lecturer; roadmap.sh cấm dùng lại nội dung | Tự soạn 3 lộ trình FE/BE/Data, ~20 học phần; roadmap.sh chỉ tham khảo |
| Alias tiếng Việt | O\*NET chỉ có tiếng Anh | Viết tay ~50 alias, ví dụ "lập trình hướng đối tượng" → OOP |
| Nhãn golden set | Không bộ nào gắn nhãn theo `skill_id` của mình | Luôn do người gán |

LLM sinh CV được, nhưng nhãn phải do người gán. Claude vừa sinh vừa chấm là vòng tròn, số F1 sẽ đẹp giả. Trộn CV thật với CV sinh.

### 6.3 Xử lý song ngữ

- `skill_id` không phụ thuộc ngôn ngữ: CV tiếng Việt và JD tiếng Anh quy về cùng mã, so khớp chéo được.
- Tên skill giữ tiếng Anh (thói quen ngành IT ở Việt Nam), thêm `name_vi` và alias tiếng Việt.
- Claude đọc được cả hai ngôn ngữ; bge-m3 đa ngôn ngữ.
- Rủi ro chính là giọng nói tiếng Việt chen thuật ngữ tiếng Anh ("em dùng useEffect để…"). PhoWhisper chỉ train trên tiếng Việt, có thể nghe sai thuật ngữ. Thu 20 đoạn ghi âm của nhóm, chạy thử Whisper và PhoWhisper rồi chọn.

### 6.4 Tránh dùng

- `Ebazhanov/linkedin-skill-assessments-quizzes`: AGPL-3.0, lại chép đề LinkedIn.
- `sudheerj/javascript-interview-questions`: không có license.
- roadmap.sh: chỉ cho dùng cá nhân, cấm sao chép nội dung.

## 7. Vấn đề cần chốt và thứ tự làm

### 7.1 Cần chốt

1. **Điểm nối C.** Đề xuất: test đầu vào giữ LLM (không cần ngân hàng câu hỏi); mock cập nhật năng lực bằng công thức Elo K = 16. Đổi qua lại hai thang bằng `θ = 1000 + 8 · p`. Không cần ngân hàng vì độ khó lấy từ tag của câu phỏng vấn.
2. **Một thang đo.** `BAO-CAO-KY-THUAT-CORE.md` §5.4 vẫn mô tả Elo θ = 1200, còn README các lõi dùng proficiency 0–100 do LLM trả. Chốt một thang lưu trong `skill_states`.
3. **Tên cột lệch giữa các tài liệu:** `suggestions.source` / `origin`; `ai_suggested` / `from_suggestion`; `matches` / `match_scores`; `cv_skills` / `cvs.skill_ids`.
4. **Tài liệu kỹ thuật chưa cập nhật:** vẫn ghi ESCO và crawler TopCV/ITviec.

### 7.2 Thứ tự làm

1. Từ điển skill (O\*NET + alias tiếng Việt) + `career_skills` + 3 lộ trình. Kiểm: mỗi skill có ≥ 1 học phần.
2. L1: test → `skill_states` → roadmap. Kiểm: làm test xong có `skill_states` và `roadmap_items`.
3. 50 JD từ VietJobs + 10–20 CV mẫu, chạy L2. Kiểm: `cv_skills`, `jd_skills` có FK hợp lệ; `match_pct` hợp lý.
4. E10. Kiểm: JD thiếu `[docker, k8s]` → 2 đề xuất; accept 1 → roadmap thêm đúng 1 dòng.
5. L3 + điểm nối C. Kiểm: mock yếu `http` → có đề xuất `http` và `skill_states` của `http` giảm.
6. Golden set nhỏ. Kiểm: có F1 / MAE đưa vào báo cáo.
