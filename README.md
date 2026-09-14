# DevRoom — Nền tảng học, luyện phỏng vấn AI và tìm việc cho lập trình viên

Đồ án: một nền tảng khép kín dành riêng cho ngách IT (Frontend, Backend, Mobile, DevOps, Data, QA).
Chu trình: **học bù lỗ hổng → luyện phỏng vấn AI → tối ưu CV và match việc làm**, kết quả từ hai bước đầu quay lại cập nhật lộ trình học.

![Luồng tổng quan](LUONG-TONG-QUAN.png)

## Bốn vai trò

| Vai trò | Làm gì |
|---|---|
| **Candidate** | Tạo phòng học theo career, test đầu vào, roadmap do AI gợi ý và tự chỉnh, học adaptive, CV, matching việc làm, mock interview |
| **Lecturer** | Thiết kế học phần, bài tập gắn tag, lộ trình base cho từng career |
| **Recruiter** | Đăng JD, tìm ứng viên xếp theo năng lực thực, mời phỏng vấn |
| **Admin** | Logging & tracking, giám sát chi phí AI, quản lý tiền |

## Nội dung repo

| File | Nội dung |
|---|---|
| `BAO-CAO-DE-TAI-DEVROOM.md` / `.pdf` | Báo cáo ngắn cho giảng viên: đề tài, nghiệp vụ, 8 lõi, ba hướng công nghệ (LLM API / local / hybrid) chưa chốt, câu hỏi xin ý kiến |
| `BAO-CAO-SIEU-DO-AN.md` / `.pdf` | Báo cáo tổng quan: luồng nghiệp vụ, tính năng, công nghệ, thuật toán, MVP, rủi ro |
| `BAO-CAO-KY-THUAT-CORE.md` / `.pdf` | Tài liệu kỹ thuật lõi 64 trang: 11 engine chạy thế nào, hiệu chỉnh và đo chất lượng, pipeline dữ liệu, lược đồ CSDL, API, luồng tuần tự, 30 màn hình web |
| `LUONG-TONG-QUAN.png` / `.svg` | Sơ đồ luồng tổng quan 4 vai trò |
| `SLIDE-SIEU-DO-AN.pptx` / `.pdf` | Slide thuyết trình |
| `demo/index.html` | Web demo mock, 1 file, không cần backend |
| `Siêu đồ án.pdf` | Tài liệu ý tưởng gốc |

## Chạy demo

**Bản online**: <https://anhnt-24.github.io/sieuduan_md/>

Mở thẳng `demo/index.html` bằng trình duyệt. Hoặc:

```bash
cd demo && python3 -m http.server 8080
```

rồi vào <http://localhost:8080>.

Demo chạy hoàn toàn trong trình duyệt, dữ liệu giả lưu ở `localStorage`, nút **Reset demo** seed lại từ đầu.

**Logic thật trong demo**: Elo/CAT đánh giá năng lực, SM-2 spaced repetition, chạy code JavaScript qua test case, công thức % match CV–JD, chấm CV rule-based, chấm STAR rule-based.

**Chỉ mock**: speech-to-text, text-to-speech, câu trả lời của trợ lý AI, số token AI.

## Công nghệ dự kiến

Next.js · FastAPI · PostgreSQL + pgvector · Redis · Celery · Piston sandbox · Claude Sonnet 5 và Haiku 4.5 · bge-m3 embedding · Whisper STT · edge-tts TTS · Playwright crawler.

Chi tiết và lý do chọn nằm trong báo cáo.
