# Lõi 1 — Học thích ứng

Lõi 1 xác định học viên đang ở đâu và nên học gì tiếp theo trong phòng học của một career (Frontend, Backend, Mobile, DevOps, Data, QA) bằng bốn engine nối tiếp: đánh giá năng lực Elo/CAT (E1), sinh roadmap theo đồ thị tiên quyết (E2), đề xuất bài tập theo dải Elo (E3) và ôn ngắt quãng SM-2 (E4). Lõi thuộc bước 2–4 trong chu trình của Candidate (test đầu vào → roadmap → học thích ứng). Người dùng trực tiếp là Candidate; Lecturer cung cấp đầu vào: câu hỏi có độ khó, học phần, tiên quyết, bài tập có tag, lộ trình base. Toàn bộ lõi là công thức tất định, không gọi LLM.

## 1. Nghiệp vụ

| # | Bước | Kết quả |
|---|---|---|
| 1 | Test đầu vào 15–20 câu, câu sau chọn theo kết quả câu trước | `θ` từng skill, proficiency `p`, tập gap (`p < 60`) |
| 2 | Hệ thống gợi ý roadmap từ lộ trình base và tập gap | Danh sách học phần đúng thứ tự tiên quyết, kèm lý do |
| 3 | Candidate thêm hoặc bỏ học phần | Roadmap cá nhân; cảnh báo vàng khi thiếu tiên quyết, không chặn |
| 4 | Mỗi ngày nhận 5 bài tập đúng lỗ hổng, đúng tầm; ôn flashcard theo lịch SM-2 | Làm bài → `θ` cập nhật → dải bài ngày sau dịch theo; lịch ôn giãn 1, 6, 15, 38 ngày |
| 5 | Vòng lặp: `θ` mới quay về bước 1–2 | Gap thu hẹp, roadmap và bộ bài tự điều chỉnh |

## 2. Sơ đồ

![Sơ đồ Lõi 1 — Học thích ứng](so-do.svg)

Sơ đồ đọc từ trái sang phải theo bốn khối. Khối 1 là vòng lặp CAT: chọn skill ít câu nhất, chọn câu có `|d − θ|` nhỏ nhất, cập nhật `θ` bằng công thức Elo, kiểm tra điều kiện dừng; chưa dừng thì quay lại chọn skill. Khối 2 chuyển `θ` thành `p` và tách tập gap. Khối 3 nhận gap cùng đầu vào Lecturer (xanh lá), chấm điểm học phần, sắp thứ tự bằng topo sort Kahn, rồi để người học thêm hoặc bỏ. Khối 4 lọc bài tập theo tag ∩ gap và dải `[θ − W, θ + W]`, nới dải khi thiếu bài; kết quả làm bài cập nhật `θ` và quay về khối 2 theo mũi tên vòng phía trên. Nhánh flashcard SM-2 nằm trong khối 4 và không tác động lên `θ`.

## 3. Nguyên lý hoạt động

### 3.1 E1 — Đánh giá năng lực Elo/CAT

`θ` là năng lực ẩn của candidate ở một skill trong một phòng, khởi tạo 1200. `d` là độ khó câu hỏi, cùng thang; `d = θ` nghĩa là 50% cơ hội làm đúng.

```
E  = 1 / (1 + 10^((d − θ) / 400))
θ' = θ + K · (S − E)          S = 1 đúng, S = 0 sai
d' = d − K_q · (S − E)        chỉ khi câu đã có ≥ 30 lượt, K_q = 8
```

| Thành phần | Giá trị | Ý nghĩa |
|---|---|---|
| Hằng số 400 | cố định | Thang đo Elo cờ vua: lệch 400 điểm thì `E = 1/11 = 0.0909`, tỷ lệ 10:1. Dải 1000–1800 (5 mức khó) phủ đúng khoảng "gần chắc sai" đến "gần chắc đúng" |
| `K` | 64 khi `n < 3`, 32 khi `n < 6`, 16 còn lại | Learning rate giảm dần. Đầu phiên `θ` còn là giá trị mặc định, cần bước lớn để vào đúng vùng; cuối phiên bước lớn chỉ làm `θ` nhảy loạn vì một câu đoán mò. `n` là số câu đã trả lời trong phiên |
| `E_adj` | `g + (1 − g)·E`, `g = 0.25` | Sàn đoán mò của trắc nghiệm 4 đáp án: đúng một câu rất khó không đẩy `θ` bằng đúng câu vừa sức |
| `S = 0.5` | đúng, `answer_ms < 3000`, `d > θ + 200` | Cờ `suspect`: đúng quá nhanh ở câu quá khó |
| Tạm dừng | 5 câu sai liên tiếp | Chống bỏ bừa, hỏi lại candidate |

Chọn câu hai tầng. Tầng 1: skill có ít câu đã hỏi nhất, bảo đảm phủ hết skill của career. Tầng 2: trong skill đó, câu chưa hỏi có `|d − θ|` nhỏ nhất. Tại `d = θ` thì `E = 0.5`, phương sai Bernoulli `E(1 − E)` đạt cực đại 0.25, mỗi câu mang nhiều thông tin nhất; đây là xấp xỉ rẻ của quy tắc maximum-information trong CAT dựa trên IRT. Điều kiện dừng: `n ≥ 15` và (`n ≥ 20` hoặc `|Δθ| < 20` ở 3 câu liên tiếp cùng skill) và mỗi skill có ≥ 2 câu.

Cold start: `θ = 1200`; `d = 900 + difficulty·180` từ tag độ khó 1–5 của Lecturer (1080 / 1260 / 1440 / 1620 / 1800). `d` chỉ tự hiệu chỉnh khi câu đã có ≥ 30 lượt; trước đó `K_q = 0` để hệ không mất tính xác định.

Proficiency: `p = clamp((θ − 1000) / 800, 0, 1) · 100`. Đạt khi `p ≥ 60`, tương đương `θ ≥ 1480`. Khoá lưu là `(candidate_id, room_id, skill_id)`: cùng `S_sql` nhưng phòng Data và phòng Backend có ngân hàng câu và `θ` khác nhau.

```python
def pick_question(room, career_skills, asked):
    counts = {s: room.answered.get(s, 0) for s in career_skills}
    for s in sorted(career_skills, key=lambda s: (counts[s], s)):
        th = room.theta.get(s, 1200)
        pool = [q for q in bank(s) if q.id not in asked]
        if pool:
            return min(pool, key=lambda q: (abs(q.d - th), q.id))
    return None

def elo_update(room, q, correct, answer_ms, n, g=0.25):
    th = room.theta.get(q.skill_id, 1200)
    E  = g + (1 - g) / (1 + 10 ** ((q.d - th) / 400))
    S  = 1.0 if correct else 0.0
    if correct and answer_ms < 3000 and q.d > th + 200:
        S = 0.5                                   # nghi đoán mò
    K  = 64 if n < 3 else 32 if n < 6 else 16
    room.theta[q.skill_id] = round(th + K * (S - E))
    room.answered[q.skill_id] = room.answered.get(q.skill_id, 0) + 1
    if q.attempts >= 30:
        q.d = round(q.d - 8 * (S - E))
    return room.theta[q.skill_id]
```

### 3.2 E2 — Sinh roadmap

Đầu vào: tập gap `[{skill_id, p, weight}]`, lộ trình base `base_tracks(career)`, DAG `module_prerequisites`, `module_skills`.

```
gap_score(s)    = weight_career(s) · (1 − p/100)          skill p ≥ 60 bị loại
module_score(m) = Σ_{s ∈ m.skills} gap_score(s) + 0.3 · unlock_count(m)
```
`unlock_count(m)` là out-degree của `m` trong DAG (số học phần mà `m` mở ra); cộng vào để học phần nền được đẩy lên trước. Tập ứng viên `C = base_tracks(career) ∪ {m : m.skills ∩ gap ≠ ∅}` trừ module đã `done`. Đây là bước chèn học phần theo gap: học phần dạy skill thiếu được thêm dù không nằm trong lộ trình base.

Thứ tự là topological sort Kahn có ưu tiên: ở mỗi bước, giữa các module đã đủ tiên quyết, lấy module có `module_score` cao nhất; hoà thì lấy `hours` nhỏ hơn, rồi `module_id`. Hàm tất định.

```python
def build_roadmap(career_id, gap, done):
    cand = set(base_track(career_id)) | {m.id for m in modules_touching(gap)}
    cand -= done
    prereq = {m: set(prereqs(m)) & cand for m in cand}      # chỉ prereq trong tập
    score  = {m: module_score(m, gap) for m in cand}
    out, ready = [], {m for m in cand if not prereq[m]}
    while ready:
        m = max(ready, key=lambda x: (score[x], -hours(x), x))
        ready.remove(m); out.append(m)
        for n in cand:
            if m in prereq[n]:
                prereq[n].discard(m)
                if not prereq[n] and n not in out:
                    ready.add(n)
    if len(out) != len(cand):
        raise CycleError(sorted(cand - set(out)))
    return out

def check_prereq(roadmap_ids, module_id):
    return [p for p in prereqs(module_id) if p not in roadmap_ids]   # cảnh báo, không chặn
```
Candidate thêm hoặc bỏ học phần tự do. `source` là `ai_suggested` hoặc `user_added`; bỏ học phần đặt `removed_at` thay vì xoá. Khi bỏ một học phần tiên quyết hoặc thêm học phần mà tiên quyết chưa có trong roadmap, engine trả `warnings[] = [{module_id, missing_prereqs[]}]`, giao diện hiện banner vàng. Không chặn vì luồng nghiệp vụ đã chốt "AI chỉ gợi ý".

### 3.3 E3 — Đề xuất bài tập

```
d_ex      = 900 + difficulty_1_5 · 180
center(s) = θ_s + streak_shift(s)     +100 nếu 3 đúng liên tiếp, −100 nếu 2 sai liên tiếp
chọn e    khi tags(e) ∩ gap ≠ ∅  và  |d_ex − center(s)| ≤ W
```
`W = 100` cho bộ daily (5 bài), `150` cho weekly (15 bài + 1 bài code). Khi số bài đủ điều kiện nhỏ hơn số bài cần, nới dải theo bậc `100 → 250 → ∞`; mỗi lần nới ghi `band_relaxed_level`. Tỷ lệ lần gọi có nới `> 30%` là tín hiệu ngân hàng bài mỏng ở skill đó.

Chống lặp: bài `pass` nghỉ 14 ngày, bài `fail` nghỉ 3 ngày, không quá 1 lần cùng bài trong 7 ngày, không quá 2 bài cùng skill trong một bộ, xoay vòng `type` (mcq → code → essay → flashcard). Bước chọn cuối là weighted sampling không hoàn lại với trọng số `gap_score(skill)`; không lấy top-k cứng vì top-k làm bộ bài lặp gần y nguyên mỗi ngày. Kết quả làm bài cập nhật `θ` bằng đúng công thức E1 với `n` là số bài đã làm ở skill đó.

```python
def suggest_exercises(room, gap, mode="daily"):
    n_target, W0 = (5, 100) if mode == "daily" else (15, 150)
    for level, W in enumerate([W0, 250, None]):
        pool = []
        for g in gap:
            c = room.theta.get(g.skill_id, 1200) + streak_shift(room, g.skill_id)
            for e in exercises_with_tag(g.skill_id):
                if in_cooldown(room, e) or seen_within(room, e, days=7): continue
                if W is None or abs(e.d - c) <= W: pool.append((e, gap_score(g)))
        if len(pool) >= n_target: break
    picked, per_skill, last_type = [], {}, None
    for e, w in weighted_shuffle(pool):                    # sampling theo gap_score
        if per_skill.get(e.skill_id, 0) >= 2: continue
        if e.type == last_type and len(pool) > n_target: continue
        picked.append(e); per_skill[e.skill_id] = per_skill.get(e.skill_id, 0) + 1
        last_type = e.type
        if len(picked) == n_target: break
    return picked, level
```

### 3.4 E4 — Spaced repetition SM-2

Thẻ mới: `ef = 2.5`, `interval = 0`, `reps = 0`. Sau mỗi lần ôn với điểm `q ∈ 0..5`:

```
q < 3:  reps = 0, interval = 1
q ≥ 3:  interval = 1 (reps = 0) | 6 (reps = 1) | round(interval · ef) (reps ≥ 2);  reps += 1
ef'     = max(1.3, ef + 0.1 − (5 − q) · (0.08 + (5 − q) · 0.02))
due     = today + interval
```
Chuỗi `interval` khi luôn `q = 4`: `1, 6, 15, 38, 95, 238`. `ef` không đổi tại `q = 4`, tăng 0.1 tại `q = 5`, giảm 0.14 tại `q = 3`.

Quy ước `q` (tự đặt, không có trong SM-2 gốc). Flashcard có 4 nút tự chấm: Quên = 1, Khó = 3, Tốt = 4, Dễ = 5. Quiz tự chấm suy `q` từ `(correct, answer_ms)` với `t_ref` là median thời gian các lượt đúng của chính thẻ đó, sàn 8 s, fallback 20 s khi chưa đủ 5 lượt:

| Điều kiện | `q` |
|---|---|
| sai và `t ≥ 2·t_ref` | 0 |
| sai và `t < 2·t_ref` | 1 |
| sai nhưng chọn đáp án `near_miss` | 2 |
| đúng và `t > 2·t_ref` | 3 |
| đúng và `0.5·t_ref < t ≤ 2·t_ref` | 4 |
| đúng và `t ≤ 0.5·t_ref` | 5 |

Tồn đọng: mỗi ngày tối đa 30 thẻ, xếp theo `overdue_ratio = (today − due) / max(interval, 1)` giảm dần. Thẻ quá hạn hơn `2·interval` mà `q ≥ 3` không reset về 1 mà `interval ← max(1, round(interval · 0.5))`, cờ `lapsed`. Thẻ `lapses ≥ 8` là `leech`, gỡ khỏi lịch.

```python
def sm2(card, q, today):
    ef, itv, reps = card.ef or 2.5, card.interval or 0, card.reps or 0
    lapsed = False
    if q < 3:
        reps, itv, card.lapses = 0, 1, card.lapses + 1
    else:
        if card.due and (today - card.due).days > 2 * max(itv, 1):
            itv, lapsed = max(1, round(itv * 0.5)), True      # tồn đọng, phạt nhẹ
        else:
            itv = 1 if reps == 0 else 6 if reps == 1 else round(itv * ef)
        reps += 1
    ef = max(1.3, ef + 0.1 - (5 - q) * (0.08 + (5 - q) * 0.02))
    return {"ef": round(ef, 2), "interval": itv, "reps": reps,
            "due": today + timedelta(days=itv), "lapsed": lapsed, "leech": card.lapses >= 8}
```

## 4. Dữ liệu

Một lượt trả lời trong test đầu vào (E1), vào và ra:

```json
{"room_id":"r1","skill_id":"S_react","question_id":"q77","d":1450,"correct":true,"answer_ms":8400,"n_answered_in_session":4}
{"skill_id":"S_react","theta_before":1200,"theta_after":1229,"E":0.0909,"K":32,"p":28,"question_d_after":1449}
```
E3 vào `{"room_id":"r1","mode":"daily","gap":[{"skill_id":"S_docker","p":18}],"theta":{"S_docker":1150},"streak":{"S_docker":[1,1,1]}}`, ra `{"items":[{"exercise_id":"e91","skill_id":"S_docker","d":1080,"type":"mcq","reason":"gap Docker (p=18), d trong [1050,1250]"}],"band_relaxed":false}`. E4 vào `{"card_id":"f12","ef":2.5,"interval":6,"reps":2,"q":4}`, ra `{"ef":2.5,"interval":15,"reps":3,"due":"2026-09-29","lapsed":false}`.

| Bảng | Cột chính |
|---|---|
| `rooms` | `id`, `candidate_id`, `career_id`, `tested_at` |
| `questions` | `id`, `skill_id`, `career_id`, `difficulty_1_5`, `d`, `attempts`, `choices`, `answer_key` |
| `question_attempts` | `id`, `room_id`, `question_id`, `correct`, `answer_ms`, `theta_before`, `theta_after`, `k_used`, `suspect` |
| `skill_states` | `room_id`, `skill_id`, `theta`, `answered`, `p`, `updated_at` — PK `(room_id, skill_id)` |
| `modules` / `module_skills` / `module_prerequisites` | `id`, `career_id`, `name`, `hours` / `module_id`, `skill_id`, `weight` / `module_id`, `prereq_id` |
| `base_tracks` | `id`, `career_id`, `module_id`, `order_index` |
| `roadmap_items` | `id`, `room_id`, `module_id`, `order_index`, `source`, `reason`, `status`, `removed_at` |
| `exercises` / `exercise_tags` | `id`, `type`, `difficulty_1_5`, `d`, `attempts` / `exercise_id`, `tag_id` |
| `exercise_attempts` / `exercise_assignments` | `room_id`, `exercise_id`, `correct`, `answer_ms` / `mode`, `assigned_date`, `band_relaxed_level`, `done_at` |
| `flashcards` / `srs_cards` / `review_logs` | `front`, `back`, `skill_id` / `ef`, `interval`, `reps`, `lapses`, `due`, `leech` / `q`, `answer_ms`, `t_ref`, `interval_before`, `interval_after` |

## 5. Tham số cấu hình

| Tên | Mặc định | Ảnh hưởng |
|---|---|---|
| `elo_init_theta` / `elo_scale` | 1200 / 400 | Điểm khởi đầu; độ dốc đường kỳ vọng (không nên đổi) |
| `elo_k_schedule` | `[64, 32, 16]` tại `n < 3`, `n < 6` | Nhanh hội tụ vs ổn định |
| `elo_kq_question` | 8, bật khi `attempts ≥ 30` | Tốc độ tự hiệu chỉnh `d` |
| `cat_min_q` / `cat_max_q` / `cat_stop_delta` | 15 / 20 / 20 trong 3 câu | Độ dài test, dừng sớm |
| `guess_floor_g` | 0.25 | Mức miễn nhiễm đoán mò |
| `pass_threshold_p` (= `gap_p_cutoff`) | 60 | Ranh giới "đạt"; đổi là đổi gap toàn hệ |
| `unlock_bonus` / `warn_missing_prereq` | 0.3 / true | Ưu tiên học phần nền; bật/tắt banner cảnh báo |
| `band_w_daily` / `band_w_weekly` / `band_relax_steps` | 100 / 150 / `[100, 250, off]` | Độ khó cảm nhận; khi nào chấp nhận bài lệch tầm |
| `daily_count` / `weekly_count` / `max_per_skill_in_set` | 5 / 15 + 1 code / 2 | Tải học mỗi ngày; đa dạng bộ bài |
| `cooldown_pass_days` / `cooldown_fail_days` | 14 / 3 | Mức lặp lại |
| `streak_shift_up` / `down` | +100 / −100 | Tốc độ leo độ khó |
| `sm2_ef_init` / `sm2_ef_min` | 2.5 / 1.3 | Độ giãn ban đầu; sàn chặn interval đứng yên |
| `srs_daily_cap` / `srs_overdue_penalty` / `srs_leech_threshold` | 30 thẻ / ×0.5 khi quá `2·interval` / 8 lapses | Chống tồn đọng dội; phạt khi bỏ lâu; khi nào gỡ thẻ |

## 6. Kiểm chứng

| Engine | Đầu vào | Đầu ra mong đợi | Kết quả trên demo |
|---|---|---|---|
| E1 | `θ=1200, d=1200, S=1, K=64, g=0` | `E=0.5`, `θ'=1232` | 🟢 đúng |
| E1 | `θ=1200, d=1600, S=1, K=32, g=0` | `E=0.0909`, `θ'=1229` | 🟢 đúng |
| E1 | 20.000 cặp `θ, d ∈ [800, 2000]`, đúng | `θ' ≥ θ` và `|θ' − θ| ≤ K` | 🟢 giữ; `θ' = θ` xảy ra 10% do làm tròn khi `K·(S − E) < 0.5` |
| E1 §3.4 | 200 học viên ảo, `θ* ~ N(1300, 200)`, `g = 0.25` | MAE giảm đơn điệu theo `n`; CAT tốt hơn chọn ngẫu nhiên | 🟢 giữ; đo ≈ 145 / 134 / 125 / 117 / 107 tại n = 5 / 10 / 15 / 20 / 30 |
| E1 §3.4 | như trên | MAE `< 80` tại `n = 20` | 🔴 không đạt với `σ = 200` (≈ 117); đạt ≈ 80 khi `σ = 100`. `K = 16` đóng khoảng cách chậm khi `θ*` xa 1200 |
| E2 | DAG `m1 → m2`, gap chỉ có React (m2) | `[m1, m2]`, không phải `[m2]` | 🟢 đúng |
| E2 | roadmap có m4 → m5, bỏ m4 | `check_prereq` trả `[m4]` cho m5, m5 vẫn còn | 🟢 đúng, banner vàng |
| E2 | 6 career × 2 cấu hình `θ` | mọi tiên quyết trong tập đứng trước; không chu trình; chạy 2 lần cùng kết quả | 🟢 giữ |
| E3 | gap chỉ CSS, `θ = 1200` | `W=100` → 1 bài, `W=250` → 3 bài, `∞` → 3 bài; `band_relaxed_level = 2` | 🟢 đúng |
| E3 | 120 bộ ngẫu nhiên | `tags ∩ gap ≠ ∅`, không trùng id, `≤ 5` bài, `≤ 2` bài/skill | 🟢 giữ |
| E4 | `{ef:2.5, reps:2, interval:6}`, `q = 4 / 5 / 3 / 2` | `interval=15, ef=2.5` / `ef=2.6` / `ef=2.36` / `reps=0, interval=1, ef=2.18` | 🟢 đúng |
| E4 | `q = 4` liên tiếp 8 lần; `q = 0` liên tiếp 20 lần | `interval` tăng đơn điệu; `ef` chạm sàn 1.3, `interval = 1` | 🟢 giữ |

Chỉ số vận hành khi có dữ liệu thật: tỷ lệ đúng của câu CAT chọn nằm 0.45–0.65; tỷ lệ đúng bài tập theo tuần nằm 0.50–0.80; retention lần ôn kế tiếp nằm 0.80–0.90; nhóm `q = 5` có retention cao hơn nhóm `q = 3`.

## 7. Demo

Demo chạy độc lập tại [demo.html](demo.html): bốn tab và một nhật ký thuật toán chung ở cuối trang.

| Thao tác | Quan sát được |
|---|---|
| Tab 1: chọn career, số câu tối đa 8 hoặc 15, Bắt đầu | Mỗi câu hiện skill, `d`, `θ`, `K` sẽ dùng và lý do chọn: skill ít câu nhất, danh sách `|d − θ|` của các câu chưa hỏi |
| Trả lời (chuột hoặc phím 1–4), Enter sang câu tiếp | Thời gian trả lời, `E`, `E_adj`, `K`, `S`, `θ'`, `Δθ` với số thay vào công thức; cờ nghi đoán mò khi đúng dưới 3 s ở câu `d > θ + 200` |
| Kết thúc test | Lý do dừng; bảng `θ`, `p`, đạt/gap; biểu đồ `θ` theo câu, mỗi skill một đường |
| Tab 2: kéo `θ*`, Mô phỏng 1 học viên / 200 học viên | Đường `θ̂` tiến về đường ngang `θ*`, chấm xanh/đỏ là đúng/sai, bảng 30 bước; đường MAE theo `n` cho CAT và chọn ngẫu nhiên |
| Tab 3 | Chip gap với `gap_score`; đồ thị tiên quyết tô đạt / chưa đạt / khóa; bảng thứ tự topo với `score` và lý do |
| Tab 3: Bỏ, Thêm, Xong | Banner vàng khi thiếu tiên quyết, học phần phụ thuộc vẫn còn; `user_added`; xong mở khóa học phần phụ thuộc; topo sort chạy lại |
| Tab 4: bộ bài hôm nay, trả lời hoặc tự chấm | Số bài ở từng mức nới `W`; mỗi bài ghi `d_ex`, `θ`, `streak_shift`, `|d − tâm|`, tag thuộc gap; sau khi làm: `E`, `K`, `θ` trước → sau, tâm dải lần sau |
| Tab 4: lật thẻ (F), chấm Quên/Khó/Tốt/Dễ (1–4) | `ef'`, `interval`, `reps`, `due` với số thay vào công thức; dải lịch 30 ngày đánh dấu số thẻ đến hạn |
| Reset | Xoá `localStorage`, seed lại |

| Phần | Logic thật hay mô phỏng |
|---|---|
| `E`, `θ'`, lịch `K`, sàn `g`, cờ `suspect`, chọn câu hai tầng | Thật, đúng công thức E1 |
| Điều kiện dừng | Rút gọn: `n ≥ N` (N = 8 hoặc 15) hoặc mỗi skill ≥ 2 câu và 3 câu liên tiếp cùng skill có `|Δθ| < 20`; không có `cat_max_q = 20` |
| Cập nhật `d` của câu hỏi (`K_q`) | Không mô phỏng; ngân hàng chưa có 30 lượt |
| Mô phỏng hội tụ | Thật theo §3.4; ngân hàng 41 câu `d = 950…1750` cách đều là dữ liệu giả |
| `gap_score`, `module_score`, topo sort Kahn, `check_prereq` | Thật; `weight_career = 1` cho mọi skill; 8 cạnh tiên quyết tự đặt trong seed |
| Dải Elo, nới dải, cooldown 14/3 ngày, ≤ 2 bài/skill, xoay type, weighted sampling | Thật; hạt giống ngẫu nhiên theo ngày để bộ bài ổn định trong ngày. Không có quy tắc 7 ngày và chế độ weekly |
| Bài trong bộ daily | Trắc nghiệm mượn câu QBANK cùng skill có `d` gần `d_ex` nhất; code/tự luận tự chấm đúng/sai |
| SM-2 | Thật với 4 nút tự chấm. Không có phạt tồn đọng ×0.5, `leech`, cap 30 thẻ/ngày, ánh xạ thời gian → `q` |

## 8. Hạn chế

| Hạn chế | Hệ quả |
|---|---|
| 15–20 câu cho 6 skill là 2–4 câu/skill | Sai số `θ` từng skill ±100; chỉ dùng xếp ưu tiên gap, không cấp chứng chỉ |
| `K = 16` sau câu thứ 6 | `θ*` cách 1200 trên 300 điểm hội tụ chậm; MAE ≈ 117 tại n = 20, cao hơn kỳ vọng 70 của tài liệu kỹ thuật; `K = 32` cố định cho MAE thấp hơn từ n ≥ 15 |
| Elo không phân biệt "đã nắm" với "đoán trúng", không mô hình hoá quên | Phần quên do E4 gánh; khi đủ ~50k lượt, BKT là lựa chọn tốt hơn |
| `d` do Lecturer gắn sai, câu ít lượt | `θ` lệch theo; cần dashboard câu có `attempts < 30` |
| Tập ứng viên roadmap kéo mọi học phần dạy skill gap, kể cả của career khác | Frontend có gap HTTP nhận cả "Python cho backend"; bản thật cần lọc theo `career_id` |
| Roadmap phụ thuộc DAG và lộ trình base của Lecturer; skill gap không có học phần nào dạy bị bỏ qua | DAG sai thì thứ tự sai; cần ghi `missing_module_demand` |
| `d_ex = 900 + diff·180` là ánh xạ tuyến tính tự đặt; ngân hàng dưới 10 bài/skill | Độ khó của hai Lecturer không cùng thang; cooldown 14 ngày làm cạn bài |
| SM-2 nhét thời gian trả lời vào `q`, coi thẻ độc lập | Phần dễ sai nhất; thẻ cùng khái niệm ôn lệch nhau; FSRS là nâng cấp rẻ nếu retention lệch |
