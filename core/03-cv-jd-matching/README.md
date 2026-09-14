# Lõi 3 — CV và so khớp JD

Lõi 3 biến CV và tin tuyển dụng (JD) thành hai tập `skill_id` trên cùng một từ điển kỹ năng, rồi tính tỉ lệ khớp giữa chúng. CV, JD, học phần và câu hỏi phỏng vấn đều trỏ tới bảng `skills`, nên gap từ JD khớp thẳng với học phần mà không cần bước dịch. Lõi thuộc con đường sự nghiệp của Candidate (bước 5 và 6 trong chu trình DevRoom) và là nguồn dữ liệu cho Recruiter tìm ứng viên. Đầu ra là % match, danh sách kỹ năng thiếu (gap) và đề xuất học phần đẩy ngược về lõi học thích ứng. Ba engine của tài liệu kỹ thuật gộp trong lõi này: E6 (bóc tách và chấm CV), E7 (thu thập và chuẩn hoá JD), E8 (matching).

## 1. Nghiệp vụ

Hai luồng vào, một trục chung, hai chiều ra.

| # | Bước | Ai | Kết quả |
|---|---|---|---|
| 1 | Upload CV PDF/DOCX hoặc dùng CV do AI tạo | Candidate | `cvs` (raw_text, cv_json, embedding), `cv_skills` theo `skill_id` |
| 2 | Chấm CV theo JD mục tiêu | Hệ thống | `score_rule` 0–100, gợi ý sửa, cờ ATS |
| 3 | Thu JD từ ① crawler theo lịch, ② agent tìm web theo yêu cầu, ③ Recruiter nhập | Hệ thống, Recruiter | `jd_postings` chuẩn hoá, `jd_skills` theo `skill_id`, bản trùng gắn `duplicate_of` |
| 4 | Xem feed việc làm, chọn một JD | Candidate | % match, skill_match, cosine, gap từng skill |
| 5 | Tạo đề xuất cải thiện từ gap | Candidate | `suggestions(status='pending')` chờ nhận hoặc bỏ qua ở roadmap |
| 6 | Tìm ứng viên cho một JD | Recruiter | Danh sách ẩn danh xếp theo `recruiter_score`, lọc `min_match` |
| 7 | Admin duyệt hàng đợi `unmapped_skills`, chỉnh trọng số | Admin | Alias mới, `settings.match_weights` |

## 2. Sơ đồ

![Sơ đồ Lõi 3 — CV và so khớp JD](so-do.svg)

Đọc từ trên xuống. Hàng đầu là hai luồng vào: CV đi qua upload → bóc chữ → trích xuất LLM; JD đi từ ba nguồn vào một pipeline chuẩn hoá chung. Cả hai đổ chuỗi kỹ năng thô vào dải màu navy `map_skill(raw)` ba tầng và chỉ đi tiếp khi đã có `skill_id`. Hàng ba là phần riêng của mỗi luồng sau chuẩn hoá: CV được embedding và chấm điểm, JD được khử trùng lặp và embedding phần yêu cầu. Khối so khớp ở giữa nhận `cv_skills`, `emb_cv`, `jd_skills`, `emb_jd` và thêm `skill_states` từ lõi học (mũi tên teal). Từ đó toả ra chiều thuận cho Candidate (% match, gap, đề xuất pending) và chiều ngược cho Recruiter (xếp hạng N ứng viên). Mũi tên teal dưới cùng là vòng phản hồi về roadmap.

## 3. Nguyên lý hoạt động

### 3.1 Pipeline trích xuất CV (E6)

```text
upload (≤ 5 MB, .pdf/.docx, magic bytes) → object storage private
→ PyMuPDF page.get_text("blocks") | python-docx paragraphs
→ len(text) < 200 → parse_status='scanned_unsupported' (không OCR ở MVP)
→ cluster block theo cột, sort theo y (CV 2 cột)
→ Claude Sonnet 5, temperature 0, JSON schema
   {full_name, headline, email, phone, links, years_total,
    skills[{raw, years, level, evidence_span}], experiences[bullets{text, has_metric}],
    projects, education, page_count, ats_flags}
→ map raw → skill_id (3 tầng, §3.2) → cv_skills
→ bge-m3(toàn văn rút gọn) → cvs.embedding vector(1024)
```

Số năm kinh nghiệm và vai trò do LLM trả về theo schema; demo thay bằng regex `(\d+)\s*\+?\s*(năm|years?)` lấy giá trị lớn nhất và bảng `title_map` trên dòng headline.

### 3.2 Chuẩn hoá về `skill_id` ba tầng (dùng chung CV và JD)

| Tầng | Điều kiện | Kết quả |
|---|---|---|
| 1 | `lower(trim(raw)) = lower(skills.name_en | name_vi)` | `skill_id`, `map_method='exact'`, `map_cos=1` |
| 2 | `lower(trim(raw)) = lower(skill_aliases.alias)` (UNIQUE `lower(alias)`) | `skill_id`, `map_method='alias'` |
| 3 | `cosine(embed(raw), skills.embedding) ≥ 0,85` trên top-3 HNSW, và cặp (top-1, top-2) không thuộc `skill_confusion_blocklist` | `skill_id`, `map_method='cosine'`, `map_cos` |
| – | Không đạt | `unmapped_skills(raw_text, freq += 1, best_guess, best_cos)` → Admin thêm alias |

```python
def map_skill(raw):
    k = norm(raw)
    if hit := exact_lookup(k):  return hit, "exact", 1.0
    if hit := alias_lookup(k):  return hit, "alias", 1.0
    rows = knn_skills(embed(raw), k=3)          # 1 - (a <=> b) trong pgvector
    if rows and rows[0].cos >= CFG.cos_threshold:
        if (rows[0].skill_id, rows[1].skill_id) not in BLOCKLIST:
            return rows[0].skill_id, "cosine", rows[0].cos
    enqueue_unmapped(raw)                        # giữ raw, không tạo skill mới
    return None, "unmapped", rows[0].cos if rows else 0.0
```

Bất biến: mọi dòng `cv_skills` và `jd_skills` có `skill_id NOT NULL` hoặc `raw_text` tồn tại trong `unmapped_skills`. Chuỗi gốc không bao giờ bị bỏ.

### 3.3 Chấm CV theo quy tắc

| # | Quy tắc | Điểm |
|---|---|---|
| R0 | Điểm nền | +40 |
| R1 | Có `full_name` và `headline` | +10 |
| R2 | Mỗi bullet kinh nghiệm có số liệu | +6/bullet, cap +18 |
| R3 | Bullet dài > 140 ký tự | −5/bullet |
| R4 | ≥ 3 bullet kinh nghiệm/dự án | +8 |
| R5 | Link GitHub hợp lệ | +8 |
| R6 | Liệt kê ≥ 4 skill cụ thể | +6 |
| R7 | Mỗi skill của JD mục tiêu không có trong `cv_skills` | −4/skill |
| R8 | `page_count ∈ {1, 2}` | +5, ngoài dải −5 |
| R9 | Không có `ats_flags` | +4 |
| R10 | Có email và phone | +3 |

`score_rule = clamp(round(ΣR), 0, 100)`. Bản thật cộng thêm rubric LLM 4 mục (rõ ràng, tác động, phù hợp JD, gọn) mỗi mục 0–5: `score_llm = Σ/20·100`, `score_final = 0,5·score_rule + 0,5·score_llm`, chấm 2 lần lấy trung bình. Gợi ý sửa từ LLM chỉ được giữ khi `evidence_span` là chuỗi con nguyên văn của `cv_text`; skill mới phải có `needs_user_confirm = true`.

### 3.4 Thu JD từ ba nguồn và chuẩn hoá (E7)

| Nguồn | `source` | `trust_weight` | Cách thu |
|---|---|---|---|
| ① Crawler | `crawler` | 0,8 | Crawlee + Playwright, TopCV và ITviec, Celery beat 07:00 và 19:00, ≤ 200 tin/lần, lưu HTML thô trước khi parse |
| ② Agent tìm web | `web_search` | 0,6 | Tavily/Brave khi Candidate bấm tìm thêm; Haiku 4.5 lọc `is_job_posting`, bỏ nếu `confidence < 0,7`; 10 query/ngày |
| ③ Recruiter | `recruiter` | 1,0 | Form; LLM bóc skill rồi Recruiter sửa `required`, `weight_1_3` và xác nhận |

Pipeline chung: parse (trafilatura/PyMuPDF/form) → Haiku 4.5 JSON `{title, company, level, city, salary, skills[{name, required, weight, years}]}` → map skill 3 tầng → bge-m3 trên `title + skills + responsibilities` → dedupe → `status='published'` → `recompute_match(jd_id)`. Chỉ embed phần yêu cầu, không embed phúc lợi để cosine không bị boilerplate kéo lên.

### 3.5 Khử trùng lặp

1. `content_hash = sha256(lower(trim(title)) || '|' || lower(trim(company)) || '|' || normalized_desc)`; `normalized_desc` bỏ HTML, hạ chữ, gộp khoảng trắng, bỏ số điện thoại, email, số. Trùng hash → cùng tin.
2. Gần đúng: cùng `company` hoặc cùng `career`, cách nhau ≤ 30 ngày, `cosine(embedding) ≥ 0,95` và `similarity(title) ≥ 0,8` (pg_trgm).
3. Chọn bản giữ theo thứ tự: nguồn ③ > ① > ②; mô tả dài hơn; `posted_at` mới hơn; `id` nhỏ hơn.
4. Bản bị loại: `status='duplicate'`, `duplicate_of = id giữ lại`, ghi `jd_duplicates(kept_id, dropped_id, reason, cos)`. Không xoá.

### 3.6 Công thức so khớp (E8)

```text
match       = w_skill · skill_match + w_cos · cosine(emb_cv, emb_jd_requirements)     (0,7 / 0,3)
skill_match = Σᵢ wᵢ·cᵢ / Σᵢ wᵢ                        i chạy qua mọi skill của JD
wᵢ          = weight_1_3(i) · (1,0 nếu required, 0,5 nếu nice_to_have)
Have        = {skill_id ∈ cv_skills} ∪ {skill_id ∈ skill_states : proficiency ≥ 60}
```

| `cᵢ` | Điều kiện | Ý nghĩa |
|---|---|---|
| 1 | `skill_id(i) ∈ Have` | Có sẵn, từ CV tự khai hoặc từ lõi học đo được |
| 0,5 | Tồn tại `j ∈ Have` với `broader_id(i) = broader_id(j)` hoặc `cosine(emb_i, emb_j) ≥ 0,8` | Họ hàng gần, học nhanh nhưng không bằng có sẵn |
| 0 | Còn lại | Thiếu hẳn, đưa vào `gap` |

```sql
-- pgvector: <=> là cosine distance, cosine_sim = 1 - (a <=> b); index HNSW vector_cosine_ops
WITH cfg AS (SELECT (v->>'w_skill')::float ws, (v->>'w_cos')::float wc FROM settings WHERE k='match_weights'),
have AS (SELECT skill_id FROM cv_skills WHERE cv_id = $1
         UNION SELECT skill_id FROM skill_states WHERE room_id = $2 AND p >= 60),
sm AS (SELECT js.jd_id,
         SUM(js.weight_1_3 * (CASE WHEN js.required THEN 1.0 ELSE 0.5 END) *
             CASE WHEN js.skill_id IN (SELECT skill_id FROM have) THEN 1.0
                  WHEN EXISTS (SELECT 1 FROM have h JOIN skills a ON a.skill_id = js.skill_id
                               JOIN skills b ON b.skill_id = h.skill_id
                               WHERE a.broader_id = b.broader_id
                                  OR 1 - (a.embedding <=> b.embedding) >= 0.8) THEN 0.5
                  ELSE 0.0 END)
         / NULLIF(SUM(js.weight_1_3 * (CASE WHEN js.required THEN 1.0 ELSE 0.5 END)), 0) AS skill_match
       FROM jd_skills js GROUP BY js.jd_id)
SELECT j.id, ROUND((cfg.ws * sm.skill_match + cfg.wc * (1 - (c.embedding <=> j.embedding)))::numeric, 4) AS match
FROM jd_postings j JOIN sm ON sm.jd_id = j.id JOIN cvs c ON c.id = $1 CROSS JOIN cfg
WHERE j.status = 'active' ORDER BY match DESC LIMIT 50;
```

Công thức tuyến tính được chọn thay LightGBM vì chưa có nhãn "được mời/được tuyển", vì giải thích được từng skill mất bao nhiêu điểm, và vì đổi trọng số chỉ là một dòng `settings`.

### 3.7 Chiều ngược: 1 JD × N ứng viên

```text
recruiter_score = match_score + 0,1 · (mock_score / 10)     mock cùng career
tie-break: last_active_at mới hơn
```

Precompute vào bảng `matches` khi JD tạo/sửa (`UPSERT ON CONFLICT (jd_id, candidate_id)`) và refresh 02:00 hằng ngày. Recruiter nhận danh sách ẩn danh, lọc `min_match`; mở hồ sơ mới ghi `audit_logs` và tính phí.

### 3.8 Sinh đề xuất từ gap

```python
def suggest_from_gap(candidate, room, jd, match):
    n = 0
    for s in match.gap:                                   # chỉ skill có c_i = 0
        m = first_module_with_skill(s)                    # module_skills.skill_id = s
        if m is None or m.id in roadmap_items(room):      continue
        if pending_exists(room, m.id):                    continue   # UNIQUE partial index
        insert_suggestion(room, m.id, origin="jd_match",
            reason=f"Match {jd.id} {jd.title} ({match.pct}%): thiếu {s.name}",
            priority=match.w[s], status="pending")
        n += 1
    return n
```

Candidate nhận → `roadmap_items(source='from_suggestion', position=0)`; bỏ qua → không đổi.

### 3.9 Phần mô phỏng trong demo

Tầng 3 dùng độ giống chuỗi Jaro–Winkler giữa `raw` và tên/alias, ngưỡng 0,85, thay cho cosine embedding bge-m3. Cosine giữa CV và JD dùng vector bag-of-skills (`v[s] = p/100`, tối thiểu 0,6 nếu có trong CV; `y[s] = 1` cho skill của JD) thay cho embedding văn bản. Cosine dedupe dùng bag-of-words trên `normalized_desc`; hash dùng cyrb53 64 bit thay sha256. Tiêu chí `cosine(emb_i, emb_j) ≥ 0,8` của `cᵢ = 0,5` không mô phỏng, chỉ dùng cùng nhánh cha.

## 4. Dữ liệu

Đầu vào so khớp (rút gọn):

```json
{"cv_id":"cv_u1","room_id":"r1",
 "cv_skills":["S_react","S_ts","S_css","S_cicd","S_statemgmt","S_testing"],
 "skill_states":{"S_js":82,"S_react":76,"S_css":70,"S_http":41,"S_testing":52,"S_git":68},
 "jd":{"id":"j1","skills":[{"skill_id":"S_react","required":true,"weight_1_3":3},
                           {"skill_id":"S_http","required":true,"weight_1_3":2},
                           {"skill_id":"S_ts","required":false,"weight_1_3":1}]}}
```

Đầu ra:

```json
{"jd_id":"j1","skill_match":0.810,"cos_sim":0.830,"match_score":0.816,"match_pct":82,
 "rows":[{"skill_id":"S_http","w":2,"c":0,"why":"không có → gap"}],
 "gap":["S_http"],"suggestions_created":1}
```

| Bảng | Cột chính |
|---|---|
| `skills` | `skill_id`, `name_en`, `name_vi`, `broader_id`, `branch`, `embedding` |
| `skill_aliases` | `id`, `skill_id`, `alias` (UNIQUE `lower(alias)`), `lang`, `added_by` |
| `unmapped_skills` | `id`, `raw_text`, `freq`, `best_guess_skill_id`, `best_cos`, `status`, `resolved_by` |
| `cvs` | `id`, `candidate_id`, `source`, `file_url`, `raw_text`, `cv_json`, `page_count`, `parse_status`, `embedding` |
| `cv_skills` | `cv_id`, `skill_id`, `raw_text`, `years`, `level`, `map_method`, `map_cos`, `confirmed_by_user` |
| `cv_scores` | `id`, `cv_id`, `jd_id`, `score_rule`, `score_llm`, `score_final`, `rubric_json`, `tips_json`, `run_no` |
| `jd_postings` | `id`, `source`, `trust_weight`, `url_canonical`, `content_hash`, `company_id`, `title_normalized`, `seniority`, `province_code`, `salary_min/max`, `requirements_text`, `embedding`, `status`, `duplicate_of`, `posted_at` |
| `jd_skills` | `jd_id`, `skill_id`, `raw_text`, `required`, `weight_1_3`, `years_min`, `map_method`, `map_cos` |
| `jd_duplicates` | `kept_id`, `dropped_id`, `reason`, `cos` |
| `matches` | `jd_id`, `candidate_id`, `skill_match`, `cos_sim`, `match_score`, `gap`, `computed_at` |
| `suggestions` | `id`, `room_id`, `module_id`, `skill_id`, `origin`, `reason`, `priority`, `status` |
| `settings`, `settings_history` | `k='match_weights'`, `v={"w_skill":0.7,"w_cos":0.3}`, CHECK `w_skill + w_cos = 1` |

## 5. Tham số cấu hình

| Tên | Mặc định | Ảnh hưởng |
|---|---|---|
| `skill_map_cos_threshold` | 0,85 | Hạ → nhiều false positive ở tầng 3; nâng → `unmapped` phình |
| `cv_bullet_max_len` | 140 | R3 |
| `cv_metric_bonus_cap` | 18 | R2 |
| `cv_missing_skill_penalty` | −4 | R7, ảnh hưởng lớn khi JD nhiều skill |
| `cv_rule_llm_weight` | 0,5 / 0,5 | Ổn định (rule) vs tinh tế (LLM) |
| `jd_neardup_cos` | 0,95 (§4.3; §2.7 ghi 0,93) | Gộp sai JD khác nhau nếu hạ |
| `jd_neardup_window_days` | 30 | Tin đăng lại theo mùa |
| `trust_weight` | 1,0 / 0,8 / 0,6 | Bản nào được giữ khi trùng |
| `w_skill` / `w_cos` | 0,7 / 0,3 | Nghiêng về skill rời hay ngữ nghĩa toàn văn |
| `c_half_cosine` | 0,8 | Khi nào tính khớp một nửa |
| `p_pass_for_have` | 60 | Skill đo được nào tính là "có" |
| `nice_to_have_factor` | 0,5 | Trọng số skill không bắt buộc |
| `recruiter_mock_bonus` | 0,1 | Mock ảnh hưởng thứ hạng |

## 6. Kiểm chứng

| # | Đầu vào | Đầu ra mong đợi |
|---|---|---|
| T1 | CV A, dòng kỹ năng "ReactJS, Typescript, Java Script, CSS3, Github Action, Redux Toolkit, Figma" | 11 cụm từ (kể cả quét nội dung): exact 2, alias 6, cosine 1 ("Github Action" ≈ "GitHub Actions" 0,986 → `S_cicd`), unmapped 2 ("Java Script": top-1 JavaScript 0,982, top-2 Java 0,873, cặp blocklist; "Figma": 0,606 < 0,85). 6 `skill_id`, `years_total = 2` |
| T2 | Admin thêm alias "Java Script" → JavaScript | Trích xuất lại: "Java Script" khớp tầng 2, hàng đợi unmapped giảm 1, CV A có thêm `S_js` |
| T3 | CV A với JD j1 | R7 trừ 12 (thiếu JavaScript, HTTP/REST, Git), R3 trừ 5 (1 bullet 214 ký tự) → `score_rule = 73` |
| T5 | j1 và j2 (cùng công ty, khác 1 từ) | cos 0,968 ≥ 0,95, title 0,833 ≥ 0,8 → j2 `duplicate_of = j1` (j1 dài hơn 3 ký tự) |
| T6 | j5 (Recruiter) và j6 (agent, cùng văn bản) | `content_hash` bằng nhau → giữ j5 vì nguồn ③ > ② |
| T7 | Dán JD crawler gần giống j12 (Recruiter) | cos 0,966, title 0,882 → bản dán `duplicate_of = j12`, lý do nguồn ③ > ① |
| T8 | CV A × j1 | `skill_match = 8,5/10,5 = 0,810`, `cos = 0,830`, `match = 0,7·0,810 + 0,3·0,830 = 0,816` → 82 %, gap = HTTP/REST |
| T9 | CV A × j4 | HTML `cᵢ = 0,5` (cùng nhánh Giao diện với CSS), Vue `cᵢ = 0,5` (cùng nhánh Framework với React); `skill_match = 8/9` |
| T10 | Đổi `w_skill` 0,7 → 0,5 → 0,3 | CV A × j4: 81 → 75 → 70; CV B × j11: 74 → 71 → 67. JD có cos cao, skill thấp leo hạng |
| T11 | Thêm 1 skill vào Have (u3 thêm `S_http`) | match 0,611 → 0,745, không bao giờ giảm (đơn điệu) |
| T12 | Candidate 0 skill, CV rỗng | `skill_match = 0`, `cos = 0` → match = 0 ≤ 0,3 |
| T13 | Bấm "Tạo đề xuất" hai lần cho CV A × j1 | Lần 1 tạo 1 đề xuất (học phần HTTP, REST API và caching); lần 2 tạo 0, lý do "đang pending" |
| T14 | SQL và hàm Python trên cùng dữ liệu | Lệch ≤ 1e-6 (golden test hai đường code) |
| T15 | Golden set map skill 300 chuỗi, quét ngưỡng 0,75 → 0,95 | Chọn ngưỡng có precision ≥ 0,95; top-1 accuracy ≥ 0,90; `unmapped_rate < 10 %` |

## 7. Demo

Chạy [demo.html](demo.html). Một file, không backend, trạng thái lưu trong `localStorage`, nút Reset seed lại.

| Thao tác | Quan sát được |
|---|---|
| Mục 1: kéo ngưỡng tầng 3 xuống 0,70 | "Terraform" và "Prometheus" bị map sai (false positive); kéo lên 0,99 thì "Github Action", "Dockerr" rơi vào unmapped |
| Mục 1: gán alias cho cụm trong hàng đợi | Ba CV được trích xuất lại, hàng đợi giảm, nhật ký ghi alias mới |
| Mục 2: chọn CV A/B/C, sửa văn bản, bấm Trích xuất | Bảng cụm từ → tầng khớp → `skill_id`; top-3 và điểm JW cho cụm unmapped; years bắt bằng regex kèm ngữ cảnh |
| Mục 3: đổi JD mục tiêu | R7 đổi theo skill thiếu, tổng điểm và gợi ý đổi ngay |
| Mục 4: bấm "Chạy pipeline chuẩn hóa" rồi "Chạy khử trùng lặp" | Hash rút gọn, 8 cặp cosine cao nhất, cặp bị coi là trùng, bản được giữ và lý do |
| Mục 5: chọn CV và JD, kéo trọng số | Bảng wᵢ, cᵢ, lý do từng skill; hai vector số; cosine tính ra; % đổi theo trọng số |
| Mục 6: đổi JD | 10 ứng viên xếp lại theo `recruiter_score`, cột mock và Verified |
| Mục 7: bấm "Tạo đề xuất từ gap" | Đề xuất pending theo học phần, lần bấm sau không tạo trùng |
| Mục 8 | Nhật ký từng bước với số |

| Phần | Thật hay mô phỏng |
|---|---|
| Tầng 1, 2 (tên, alias), blocklist, hàng đợi unmapped | Logic thật |
| Tầng 3 | Mô phỏng bằng Jaro–Winkler; bản thật dùng cosine bge-m3 qua pgvector |
| Trích xuất cụm từ, years, headline | Mô phỏng bằng regex và quét n-gram; bản thật dùng Sonnet 5 JSON schema |
| Bộ quy tắc R0–R10 | Logic thật, `page_count` ước từ độ dài, `ats_flags` chỉ kiểm ký tự tab |
| `score_llm`, gợi ý sửa từ LLM | Không có trong demo |
| `content_hash`, quy tắc giữ bản, cửa sổ 30 ngày, title trigram | Logic thật; hash cyrb53 thay sha256; cosine bag-of-words thay embedding |
| Công thức match, wᵢ, cᵢ, Have, trọng số Admin | Logic thật; cosine dùng vector bag-of-skills |
| Chiều ngược, `recruiter_score`, sinh đề xuất, chặn trùng pending | Logic thật trên 10 ứng viên và roadmap seed cố định |

## 8. Hạn chế

| Hạn chế | Hệ quả |
|---|---|
| Điểm rule là heuristic tự đặt, đo hình thức CV | Không đo năng lực thật; R7 khuyến khích nhồi keyword, cần giới hạn `cv_missing_skill_penalty` |
| CV nhiều cột, chữ trong ảnh | Bóc chữ sai thứ tự; MVP không OCR |
| Cosine tầng 3 nhầm cặp gần nghĩa (React / React Native, Java / JavaScript) | Bắt buộc blocklist dựng từ golden set; cụm hợp lệ như "Java Script" vẫn phải qua Admin |
| `cᵢ = 1` không phân biệt biết sơ và 3 năm kinh nghiệm | `years_min` của JD chưa vào công thức |
| Trọng số 0,7/0,3 chọn tay, chưa có nhãn | Chỉ chứng minh được nhất quán và giải thích được; nhãn đầu tiên là tỉ lệ candidate `match ≥ 0,7` được mời |
| Weight của skill JD nguồn ①② do LLM gán | Trọng số sai thì `skill_match` sai; nguồn ③ có Recruiter xác nhận nên đáng tin hơn |
