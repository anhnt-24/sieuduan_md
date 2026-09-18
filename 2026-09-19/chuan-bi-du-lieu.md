# Cách chuẩn bị dữ liệu — DevRoom MVP (Frontend)

Số lượng và người cung cấp ở [bao-cao.md §5](bao-cao.md#5-dữ-liệu-cần-thiết), bảng/cột ở §4. File mẫu ở [data/](data/). Tài liệu này nêu thứ tự, cách làm, cách kiểm.

## 1. Thứ tự chuẩn bị

| Bước | Dữ liệu | Phụ thuộc | Ai làm | File/bảng đích |
|---|---|---|---|---|
| 0 | Role, user mẫu, tham số | — | Admin | `roles`, `users`, `settings` ← `data/settings.json` |
| 1 | Career `fe` | — | Admin | `careers` ← `data/skills.json` |
| 2 | 10 skill (6 MVP + TypeScript, HTML + 2 nhóm cha) | 1 | Admin | `skills` ← `data/skills.json` |
| 3 | Alias | 2 | Admin; LLM đề xuất | `skill_aliases` ← `data/skill_aliases.csv` |
| 4 | Skill thuộc career, trọng số | 1, 2 | Admin | `career_skills` ← `data/skills.json` |
| 5 | Học phần | 1, 2 | Lecturer | `modules` ← `data/modules.json` |
| 6 | Tiên quyết | 5 | Lecturer | `module_prerequisites` ← `data/modules.json` |
| 7 | Lộ trình base | 5, 6 | Lecturer | `base_roadmaps` ← `data/modules.json` |
| 8 | Câu test | 2 | Sonnet 5 sinh; Lecturer duyệt | `questions` ← `data/questions.json` |
| 9 | Lựa chọn MCQ | 8 | Sonnet 5; Lecturer sửa | `question_options` ← `data/questions.json` |
| 10 | Bài code, bài tập | 2, 5, 8 | Lecturer | `code_problems`, `exercises` |
| 11 | Câu phỏng vấn + ý chính | 1, 2, 10 | Sonnet 5 nháp; Lecturer duyệt | `interview_questions` ← `data/interview_questions.json` |
| 12 | JD + skill của JD | 0, 1, 2, 3 | Admin (crawl), Recruiter | `jd_postings`, `jd_skills` ← `data/jd_samples.json` |
| 13 | CV | 0, 2, 3 | Nhóm đồ án | `cvs` ← `data/cv_samples.json` |
| 14 | Golden set G1–G4, biến thể CV | 11, 12, 13 | Nhóm đồ án, Lecturer | File nhãn riêng, không nạp CSDL production (đề xuất) |

## 2. Cách làm từng nhóm dữ liệu

### 2a. Taxonomy skill + alias

- **Nguồn:** tự soạn (bao-cao §5). Đối chiếu tên với [O\*NET Software Skills](https://www.onetcenter.org/database.html), cột Hot Technology (CC BY 4.0). [Stack Overflow Survey](https://survey.stackoverflow.co) (ODbL) chỉ tham khảo tên. Bỏ ESCO: không có tiếng Việt, không có React.
- **Các bước:**
  1. Ghi career `fe` và 10 skill: `code`, `name_en`, `name_vi`, `parent_id`.
  2. Ghi `career_skills`: 6 skill MVP có `is_core = true`, `weight` 1–3.
  3. Viết tay 5–10 alias mỗi skill MVP, gồm alias tiếng Việt, `source = admin`.
  4. LLM đề xuất thêm (`source = llm`); Admin duyệt rồi mới nạp.
- **Kiểm tra:** không `lower(alias)` nào trỏ 2 skill. Cây `parent_id` không vòng, không mồ côi. `name_vi` không NULL.

### 2b. Học phần, tiên quyết, lộ trình base

- **Nguồn:** tự soạn theo seed §4.2.3 [báo cáo kỹ thuật](../BAO-CAO-KY-THUAT-CORE.md). roadmap.sh chỉ tham khảo: chỉ cho dùng cá nhân, cấm sao chép nội dung.
- **Các bước:**
  1. Soạn 6 học phần: `code`, `skill_ids`, `hours`, `content`.
  2. Khai `module_prerequisites`, ví dụ `m_react` cần `m_js_core`.
  3. Ghi 1 `base_roadmaps`: `module_ids` theo thứ tự, `is_current = true`.
  4. Import upsert theo `code`. Skill resolve qua alias; không resolve được thì dừng, in danh sách.
- **Kiểm tra:** đồ thị tiên quyết không chu trình. Trong lộ trình, học phần đứng sau học phần nó cần. Mỗi skill MVP có ≥ 1 học phần.

### 2c. Ngân hàng câu test

- **Nguồn:** Sonnet 5 sinh theo lô qua Batch API (giảm 50% giá, §3.6 báo cáo kỹ thuật). ≈ $1,4 cho 240 câu (ước tính, DANH-GIA-LOI-1). Mẫu văn phong: [lydiahallie/javascript-questions](https://github.com/lydiahallie/javascript-questions) (MIT, có `vi-VI/`), [sudheerj/reactjs-interview-questions](https://github.com/sudheerj/reactjs-interview-questions) (MIT), [duyet/vietnamese-frontend-interview-questions](https://github.com/duyet/vietnamese-frontend-interview-questions) (MIT). Tránh `Ebazhanov/linkedin-skill-assessments-quizzes` (AGPL-3.0) và `sudheerj/javascript-interview-questions` (không license).
- **Các bước:**
  1. Chia 30 ô = 6 skill × 5 mức `difficulty`. Mỗi ô 1 request, xin 10–12 câu để dư khi loại (ước tính).
  2. Ép output bằng JSON schema theo cột `questions` + `question_options`: `kind`, `stem`, lựa chọn, `answer_key`, `explanation`, `difficulty`.
  3. Tính `content_hash = sha256(normalize(stem))`, trùng thì bỏ. Bỏ MCQ thiếu đáp án hoặc < 2 lựa chọn.
  4. Gán `elo_d = 900 + 180·difficulty` (1080…1800), `source = llm_batch`, `review_status = draft`.
  5. Lecturer duyệt `draft → approved | rejected`, ghi `reviewed_by`. Ô chưa đủ 8 câu `approved` thì sinh lô bù cho riêng ô đó.
- **Kiểm tra:** 30 ô × ≥ 8 câu `approved` = 240. MCQ có đúng 1 `is_correct`. `short`/`code` có `answer_key`. CAT chỉ lấy câu `approved`. `elo_d` chỉ tự chỉnh khi `times_served ≥ 30`.
- **Bài code:** ≈ 10 bài, Lecturer tự viết đề và test; Blind 75, NeetCode chỉ tham khảo. Mỗi bài 4–8 test, bắt buộc có ca biên.

### 2d. Câu phỏng vấn + ý chính

- **Nguồn:** Sonnet 5 sinh nháp. Tham khảo hai repo MIT ở 2c (React, frontend tiếng Việt) và [yangshun/tech-interview-handbook](https://github.com/yangshun/tech-interview-handbook) (MIT, câu behavioral).
- **Các bước:**
  1. Chia ô theo skill × `difficulty` (easy/medium/hard) × `kind` (technical/behavioral/code), tổng ≈ 50 câu.
  2. Mỗi câu kèm `key_points` 4–5 ý như file mẫu. Đây là căn cứ chấm `key_points_hit`, `missing_points`.
  3. Câu `kind = code` trỏ `code_problem_id` tới bài ở 2c.
  4. Lecturer duyệt như 2c. Chỉ câu `approved` được chọn làm câu chính.
- **Kiểm tra:** `key_points` không rỗng. Câu `technical` có `skill_id`. Câu `code` có `code_problem_id`. Mỗi skill MVP đủ 3 mức.

### 2e. JD

| Nguồn | Link, license, điều kiện | Dùng cho |
|---|---|---|
| ① Crawl TopCV, ITviec | Tôn trọng robots.txt; 1 request/2–4 s/site; chỉ lưu link + tóm tắt + skill, không hiện nguyên văn | Feed chạy thật |
| ① thay thế: VietJobs | [VinNLP/VietJobs](https://github.com/VinNLP/VietJobs), HF `dinhieufam/VietJobs`; MIT / CC BY 4.0; 1.906 JD IT từ TopCV, 7/2025 | Seed 200 JD cho G2, không cần crawler |
| ② Agent web search | Tavily/Brave, whitelist domain việc làm; Haiku 4.5 lọc, bỏ khi `is_jd = false` hoặc `confidence < 0,7` | On-demand, ≤ 10 query/ngày/candidate |
| ③ Recruiter form | Recruiter xác nhận skill do AI bóc | Tin thật, được giữ khi trùng |
| Đối chiếu | [Kaggle arshkon/linkedin-job-postings](https://www.kaggle.com/datasets/arshkon/linkedin-job-postings), CC BY-SA 4.0 | Chỉ đo extractor, không nạp CSDL production |
| License hạn chế | HF `tinixai/vietnamese-job-descriptions` (CC BY-NC), Kaggle `quocnguyenx43/...`, `halhuynh/it-jobs-dataset` | Chỉ đo nội bộ |

- **Các bước:**
  1. Lọc JD Frontend từ VietJobs; thiếu thì bù bằng crawl.
  2. Haiku 4.5 bóc JSON: `title`, `company`, `level`, skill kèm `is_required`, `weight`, `years_min`.
  3. Map skill 3 tầng: alias exact → cosine ≥ 0,85 → hàng đợi unmapped, giữ `raw_name`.
  4. Khử trùng: `content_hash = sha256(title|company|normalized_desc)`. Sau đó cosine ≥ 0,95 và trigram tiêu đề ≥ 0,8, cùng công ty/career trong 30 ngày.
  5. Bản trùng: `status = duplicate`, ghi `duplicate_of`, `dup_method`, không xoá. Giữ theo ưu tiên ③ > ① > ②.
- **Kiểm tra:** mỗi JD ≥ 1 `jd_skills` có `is_required`. `source_url`, `content_hash` UNIQUE. Tỷ lệ skill unmapped < 10%.

### 2f. CV

- **Nguồn:** [Kaggle snehaanbhawal/resume-dataset](https://www.kaggle.com/datasets/snehaanbhawal/resume-dataset) (CC0; 2.484 CV, chỉ 120 CV IT, tiếng Anh). CV tiếng Việt không có bộ công khai: 10 CV thật có đồng ý + 20 CV do LLM sinh (GHEP-NOI §6.2).
- **Các bước:**
  1. Gom 100 CV cho G1: ≈ 70 CV IT Kaggle + 10 thật + 20 sinh (tỷ lệ ước tính).
  2. CV thật: xin đồng ý, ẩn danh. Che tên, email, SĐT, số CMND trước khi lưu hoặc gửi LLM.
  3. CV sinh: `origin = ai_generated`, tên và domain giả ("Ứng viên Mẫu", `example.com`).
  4. Mỗi CV trong bộ kiểm thiên lệch: tạo 1 bản `name_swap`, 1 bản `gender_swap`, giữ nguyên nội dung khác.
  5. Chạy lại prefilter và judge, ghi `bias_audits`. Đạt khi `|delta| ≤ bias_max_delta` (5 điểm).
- **Kiểm tra:** không còn PII thật trong `extracted`. Nhãn G1 do người gán, trộn CV thật với CV sinh. Mỗi CV gốc có đủ 2 biến thể.

### 2g. Golden set G1–G4

| Bộ | Cỡ | Ai gán | Đo, ngưỡng (§3.2 báo cáo kỹ thuật) |
|---|---|---|---|
| G1 CV | 100 CV | 2 thành viên, gán độc lập | Bóc skill: F1 ≥ 0,85, recall ≥ 0,88 |
| G2 JD | 200 JD | 2 thành viên, gán độc lập | Như G1 |
| G3 chấm phỏng vấn | 150 transcript (50 câu × 3 mức), ghi thử nội bộ, có câu cố tình thiếu từng chiều S/T/A/R | 2 Lecturer | MAE mỗi chiều ≤ 0,7/5, kappa ≥ 0,6 |
| G4 cặp CV × JD | 300 cặp | Nhóm đồ án | `% match` người so với LLM; ngưỡng chưa chốt |

Tên G3 do tài liệu này đặt; bao-cao §5 không đánh số bộ này. §3.2 báo cáo kỹ thuật chia bộ khác (chấm CV 60, map skill 300 chuỗi).

- **Các bước:**
  1. Viết hướng dẫn gán theo `skill_id`, kèm ví dụ.
  2. 20 mẫu đầu mỗi bộ: cả hai người cùng gán, đo Cohen's kappa.
  3. Kappa dưới ngưỡng (0,75 bóc skill; 0,6 chấm) thì sửa hướng dẫn, gán lại.
  4. Gán phần còn lại; chỗ lệch chốt bằng thảo luận.
- **Kiểm tra:** nhãn luôn do người gán. Không lấy output Claude làm nhãn: Claude vừa sinh vừa chấm là vòng tròn.

### 2h. Settings

- **Nguồn:** `data/settings.json`, 19 khóa, lấy từ mục 5 README ba lõi.
- **Các bước:**
  1. Nạp nguyên file vào `settings` theo `key`.
  2. Code đọc tham số từ bảng, không hard-code.
  3. Admin sửa qua UI, ghi `updated_by`.
- **Kiểm tra:** `elo_d_init` (900, 180) khớp `elo_d` trong `questions`. `match_weights` cộng lại bằng 1. `srs_intervals` = 1, 3, 7, 14, 30. `pass_threshold_p` = 60.

## 3. Nạp vào CSDL

1. Chạy đúng thứ tự mục 1. Mỗi bước 1 transaction; lỗi FK thì dừng cả bước.
2. Upsert `INSERT … ON CONFLICT DO UPDATE` theo khóa tự nhiên: `skills.code`, `lower(alias)`, `modules.code`, `questions.content_hash`, `(question_id, idx)`, `code_problems.slug`, `jd_postings.source_url` và `content_hash`, `settings.key`. Chạy lại N lần vẫn ra 1 bản ghi; nội dung không đổi thì không tốn LLM.
3. File mẫu tham chiếu bằng id chuỗi (`S_js`, `m1`). Script resolve qua `code`/alias; không resolve được thì dừng.
4. Embedding bge-m3 `vector(1024)`, chạy local qua Celery sau insert: `skills` (`name_vi` + `name_en`), `jd_postings` (`title` + skill + mô tả), `cvs` (toàn văn rút gọn).
5. Dataset đối chiếu và golden set để thư mục riêng, script riêng, không nạp.

## 4. Checklist trước khi chạy MVP

- [ ] 1 career, 10 skill, 6 dòng `career_skills` có `is_core`.
- [ ] Mỗi skill MVP có 5–10 alias; không alias nào trỏ 2 skill.
- [ ] 6 học phần, mỗi skill MVP có ≥ 1; tiên quyết không chu trình; lộ trình base tôn trọng tiên quyết, đúng 1 bản `is_current`.
- [ ] 240 câu `approved`, mỗi ô skill × mức ≥ 8; mọi câu `approved` có `reviewed_by` (file mẫu đang để NULL).
- [ ] Mỗi MCQ đúng 1 lựa chọn `is_correct = true`; `short`/`code` có `answer_key`.
- [ ] Không trùng `content_hash` trong `questions` và `jd_postings`.
- [ ] `elo_d = 900 + 180·difficulty` với mọi câu có `times_served < 30`.
- [ ] ≈ 10 bài code, mỗi bài 4–8 test có ca biên.
- [ ] ≈ 50 câu phỏng vấn `approved`, `key_points` không rỗng; câu `code` có `code_problem_id` (file mẫu đang NULL).
- [ ] 200 JD, mỗi JD ≥ 1 `jd_skills`; unmapped < 10%.
- [ ] 100 CV; CV thật có đồng ý và đã che PII.
- [ ] Mọi FK hợp lệ: anti-join từng bảng con ra 0 dòng.
- [ ] `embedding` không NULL ở `skills`, `jd_postings`, `cvs`.
- [ ] Mỗi CV kiểm thiên lệch có bản `name_swap` và `gender_swap`.
- [ ] G1–G4 đủ cỡ; 20 mẫu đầu mỗi bộ đạt kappa.
- [ ] Đủ 19 khóa `settings`.
- [ ] Kaggle LinkedIn không có trong CSDL production.
