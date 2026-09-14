# DevRoom — Báo cáo đề tài

Nền tảng học thích ứng, luyện phỏng vấn và kết nối việc làm cho lập trình viên.

Sinh viên thực hiện: …………………… · Giảng viên hướng dẫn: ……………………

## 1. Giới thiệu đề tài

Sinh viên ngành công nghệ thông tin thường học dàn trải, không xác định được mình còn thiếu kỹ năng nào so với yêu cầu tuyển dụng thực tế, không có môi trường luyện phỏng vấn có phản hồi, và viết CV không khớp với mô tả công việc (JD). Về phía nhà tuyển dụng, việc sàng lọc dựa trên CV dạng văn bản không phản ánh năng lực thật của ứng viên.

DevRoom là nền tảng dành riêng cho lập trình viên và các vị trí IT (Frontend, Backend, Mobile, DevOps, Data, QA), gộp ba hoạt động vốn tách rời thành một chu trình khép kín: **học đúng lỗ hổng → luyện phỏng vấn → tối ưu CV và tìm việc → kết quả quay lại điều chỉnh lộ trình học**.

Đóng góp chính của đề tài nằm ở hai mắt xích nối các phân hệ:

1. Câu trả lời yếu trong phỏng vấn thử và kỹ năng còn thiếu so với JD được tự động chuyển thành đề xuất học bù. Người học quyết định nhận hay bỏ qua.
2. Năng lực đã được kiểm chứng qua quá trình học và phỏng vấn thử trở thành bằng chứng cung cấp cho nhà tuyển dụng, thay cho thông tin tự khai trên CV.

## 2. Nghiệp vụ

### 2.1 Vai trò

| Vai trò | Chức năng |
|---|---|
| **Candidate** | Tạo phòng học theo career, làm test đầu vào, học theo lộ trình thích ứng, xây dựng CV, so khớp việc làm, phỏng vấn thử với AI |
| **Lecturer** | Thiết kế học phần, bài tập có gắn tag, lộ trình mẫu cho từng career |
| **Recruiter** | Đăng JD, xem danh sách ứng viên xếp theo năng lực thực, mời phỏng vấn |
| **Admin** | Theo dõi nhật ký hệ thống và chi phí AI, quản lý tài chính |

### 2.2 Chu trình của Candidate

![Luồng tổng quan](LUONG-TONG-QUAN.svg)

| # | Bước | Kết quả |
|---|---|---|
| 1 | Tạo phòng học, mỗi phòng ứng với một career | Có thể học nhiều career song song |
| 2 | Test đầu vào thích ứng | Điểm năng lực từng kỹ năng, danh sách kỹ năng còn thiếu |
| 3 | Roadmap học | Hệ thống gợi ý từ lộ trình mẫu và kỹ năng thiếu; người học tự thêm, bớt học phần |
| 4 | Học thích ứng | Bài tập theo ngày hoặc tuần đúng độ khó, flashcard ôn ngắt quãng, sandbox chạy code |
| 5 | CV | Tạo tự động hoặc tự viết; hệ thống chấm điểm và gợi ý sửa theo JD mục tiêu |
| 6 | So khớp việc làm | JD từ ba nguồn: thu thập tự động, agent tìm trên web, Recruiter đăng. Tính tỉ lệ khớp và kỹ năng thiếu |
| 7 | Phỏng vấn thử | Hỏi đáp bằng giọng nói, hỏi thêm khi trả lời chưa đủ sâu, chấm theo khung STAR |
| 8 | Đánh giá phỏng vấn thật | Tùy chọn, ghi lại sau khi phỏng vấn với Recruiter |

Sau bước 6 và 7, hệ thống sinh đề xuất cải thiện lộ trình. Người học chọn nhận hoặc bỏ qua; roadmap ở bước 3 được cập nhật tương ứng.

### 2.3 Từ điển kỹ năng chung

Toàn bộ hệ thống dùng một từ điển kỹ năng thống nhất, xây dựng từ bộ phân loại ESCO của Liên minh châu Âu (hơn 13.000 kỹ năng, có quan hệ phân cấp). CV, JD, học phần, câu hỏi kiểm tra và câu hỏi phỏng vấn đều được gắn về cùng một mã kỹ năng. Đây là cơ sở để kết quả phỏng vấn có thể tác động ngược lên lộ trình học và kỹ năng đã học có thể so khớp với JD.

## 3. Các lõi xử lý

| Lõi | Chức năng | Nguyên lý hoạt động |
|---|---|---|
| **C1. Đánh giá năng lực** | Đo trình độ từng kỹ năng qua bài test ngắn | Mô hình ngôn ngữ sinh câu hỏi theo kỹ năng và độ khó, chấm câu trả lời; kết quả các câu trước được đưa vào để điều chỉnh độ khó câu sau. Sau 15–20 câu trả về điểm năng lực 0–100 từng kỹ năng và danh sách kỹ năng thiếu |
| **C2. Roadmap và bài tập thích ứng** | Sắp xếp học phần, chọn bài đúng độ khó, lên lịch ôn | Mô hình nhận catalog học phần của Lecturer, lộ trình mẫu và kỹ năng thiếu, trả roadmap có thứ tự kèm lý do; người học tự thêm, bớt. Hằng ngày mô hình chọn bài theo kỹ năng thiếu, chấm và gợi ý ba mức. Flashcard ôn theo khoảng cách cố định |
| **C3. Sandbox** | Chạy code người học trong môi trường cô lập | Container giới hạn CPU, bộ nhớ, thời gian, không có mạng. So kết quả với bộ test. Mô hình chỉ sinh đề và bộ test |
| **C4. Xử lý CV** | Bóc tách kỹ năng, chấm điểm, gợi ý sửa | Đọc PDF/DOCX ra văn bản; mô hình bóc tách kỹ năng, số năm, vai trò thành JSON gắn mã kỹ năng trong từ điển; chấm theo rubric; viết gợi ý sửa theo JD mục tiêu |
| **C5. Thu thập JD và so khớp** | Thu JD từ ba nguồn, tính tỉ lệ khớp | Thu thập theo lịch, theo yêu cầu hoặc do Recruiter nhập; mô hình bóc tách kỹ năng; lọc trùng lặp. So khớp bằng cách đưa hồ sơ CV và JD dạng JSON cho mô hình trả tỉ lệ khớp, kỹ năng thiếu và lý do |
| **C6. Phỏng vấn thử** | Phỏng vấn bằng giọng nói, chấm điểm | Chuyển giọng nói thành văn bản; mô hình đóng vai người phỏng vấn, chấm STAR theo rubric, quyết định hỏi thêm hay chuyển câu; đọc câu hỏi tiếp bằng giọng tổng hợp. Kết thúc có bảng điểm và kỹ năng yếu |
| **C7. Vòng phản hồi** | Chuyển kết quả C5, C6 thành đề xuất học | Mô hình nhận kỹ năng thiếu hoặc yếu cùng catalog học phần, sinh đề xuất kèm lý do; người học nhận hoặc bỏ qua |
| **C8. Trợ lý AI** | Trả lời về tiến độ, bước tiếp theo, việc phù hợp | Mô hình có công cụ đọc dữ liệu của chính người dùng và trả lời có dẫn số liệu. Chỉ đọc, không sửa dữ liệu |

Trừ C3, mọi lõi đều gọi mô hình ngôn ngữ lớn qua API. Phương án, mô hình và chi phí từng lõi ở mục 4.

## 4. Phương án công nghệ

Đề tài triển khai theo phương án **LLM API**: mọi thành phần cần hiểu, sinh hoặc đánh giá nội dung đều gọi mô hình ngôn ngữ lớn có sẵn qua API, không huấn luyện mô hình. Các thuật toán tất định đã thiết kế được trình bày ở mục 4.3 làm đề xuất bổ sung.

### 4.1 Phương án chính: LLM API

Mô hình sử dụng: Claude Sonnet 5 cho tác vụ chính, Claude Haiku 4.5 cho tác vụ nhẹ và xử lý hàng loạt; có thể thay bằng GPT hoặc Gemini tương đương. Đầu ra luôn ép theo JSON schema, temperature 0. Chi phí tính theo giá công bố: Sonnet 5 là 2 USD mỗi triệu token vào và 10 USD mỗi triệu token ra; Haiku 4.5 là 1 và 5 USD.

| Lõi | LLM API đảm nhiệm | Mô hình | Chi phí ước tính |
|---|---|---|---|
| **C1. Đánh giá năng lực** | Sinh câu hỏi theo career, kỹ năng và độ khó; chấm câu trả lời; sau 15–20 câu ước lượng proficiency 0–100 từng kỹ năng và kỹ năng thiếu. Thích ứng độ khó bằng cách đưa kết quả các câu trước vào prompt | Sonnet 5 | 0,03–0,05 USD mỗi bài test |
| **C2. Roadmap và bài tập** | Sinh roadmap từ catalog học phần của Lecturer, lộ trình mẫu và kỹ năng thiếu, chỉ được chọn học phần có trong catalog; chọn hoặc sinh bài tập hằng ngày, chấm, gợi ý ba mức; sinh flashcard. Lịch ôn dùng khoảng cách cố định 1, 3, 7, 14, 30 ngày | Sonnet 5 (roadmap), Haiku 4.5 (bài tập) | 0,01 USD mỗi roadmap; 0,002 USD mỗi bài tập |
| **C3. Sandbox** | Chỉ sinh đề bài và bộ test. Việc chạy code vẫn dùng container cô lập | Haiku 4.5 | Không đáng kể |
| **C4. Xử lý CV** | Bóc tách kỹ năng, số năm, vai trò thành JSON gắn mã kỹ năng trong từ điển; chấm theo rubric; viết gợi ý sửa theo JD mục tiêu | Sonnet 5 | 0,01–0,02 USD mỗi CV |
| **C5. Thu thập JD và so khớp** | Bóc tách JD hàng loạt; so khớp theo cách đưa hồ sơ CV và JD dạng JSON cho mô hình chấm tỉ lệ khớp, kỹ năng thiếu và lý do; agent tìm JD trên web bằng công cụ tìm kiếm; phán xét cặp JD nghi trùng lặp | Haiku 4.5 (bóc tách, hàng loạt), Sonnet 5 (so khớp) | 0,001 USD mỗi JD; 0,005 USD mỗi lượt so khớp |
| **C6. Phỏng vấn thử** | Nhận dạng giọng nói qua API; mô hình đóng vai người phỏng vấn theo JD hoặc career, chấm STAR theo rubric, quyết định hỏi thêm hay chuyển câu, sinh câu hỏi thêm; tổng hợp giọng nói qua API; cuối buổi tổng hợp kỹ năng yếu và chọn học phần từ catalog | Sonnet 5, Whisper, TTS | 0,05–0,10 USD mỗi buổi ba câu |
| **C7. Vòng phản hồi** | Từ kỹ năng thiếu hoặc yếu và catalog học phần, sinh đề xuất học kèm lý do; người học nhận hoặc bỏ qua | Haiku 4.5 | Không đáng kể |
| **C8. Trợ lý AI** | Mô hình có công cụ đọc tiến độ, roadmap, CV, việc đã so khớp của chính người dùng; chỉ đọc, không sửa | Sonnet 5 | 0,002 USD mỗi câu hỏi |

Ước tính tổng: 0,3–0,6 USD mỗi người dùng hoạt động mỗi tháng, chưa tính nhận dạng giọng nói.

### 4.2 Thành phần không dùng LLM

| Thành phần | Công nghệ |
|---|---|
| Chạy code | Piston hoặc Judge0, container giới hạn tài nguyên, không có mạng |
| Nhận dạng và tổng hợp giọng nói | Whisper qua API; edge-tts hoặc OpenAI TTS |
| Thu thập JD theo lịch | Crawler Playwright hoặc Crawlee, lưu HTML thô, khử trùng lặp sơ bộ bằng hash nội dung |
| Lưu trữ | PostgreSQL, Redis, object storage; bảng nhật ký mọi lần gọi mô hình (token vào, ra, độ trễ, kết quả parse) |

### 4.3 Đề xuất thuật toán bổ sung

Các thuật toán sau đã được thiết kế chi tiết và cài đặt trong bản demo tĩnh. Chúng thay thế hoặc bổ sung cho từng bước LLM khi cần giảm chi phí, tăng tính nhất quán hoặc cần giải thích kết quả.

| Thuật toán | Bổ sung cho bước | Lợi ích | Điều kiện áp dụng |
|---|---|---|---|
| Elo kết hợp test thích ứng (CAT) | C1 ước lượng năng lực | Điểm năng lực có công thức, đo được sai số, không tốn token, tự hiệu chỉnh độ khó câu hỏi | Có kho câu hỏi gắn kỹ năng và độ khó |
| Đồ thị tiên quyết và sắp xếp topo | C2 sinh roadmap | Luôn tôn trọng tiên quyết, cùng đầu vào cho cùng kết quả | Lecturer khai báo quan hệ tiên quyết giữa học phần |
| Chọn bài theo dải Elo và tag | C2 bài tập hằng ngày | Không tốn token, độ khó bám sát năng lực | Bài tập có tag kỹ năng và độ khó |
| Ôn ngắt quãng SM-2 | C2 lịch ôn flashcard | Khoảng cách ôn giãn theo mức nhớ thực tế thay vì cố định | Không cần điều kiện |
| Chuẩn hóa kỹ năng ba tầng (tên, alias, độ tương đồng) | C4, C5 gắn mã kỹ năng | Không cần đưa từ điển vào prompt, xử lý được kỹ năng ngoài từ điển qua hàng đợi duyệt | Có bảng alias và embedding cho từ điển |
| Chấm CV theo bộ quy tắc | C4 chấm điểm | Điểm giải thích được từng tiêu chí, không tốn token | Rubric cố định |
| Công thức so khớp có trọng số kết hợp độ tương đồng vector | C5 so khớp | Xếp hạng 1 JD với hàng nghìn ứng viên bằng một truy vấn, không tốn token, nhất quán | Kỹ năng đã gắn mã, có embedding |
| Khử trùng lặp bằng hash và độ tương đồng | C5 thu thập JD | Lọc trước khi gọi mô hình, giảm số lần gọi | Không cần điều kiện |
| Chấm STAR theo quy tắc | C6 chấm điểm | Chạy khi mô hình lỗi, dùng kiểm tra chéo kết quả mô hình | Rubric có anchor |
| Ánh xạ kỹ năng yếu sang học phần | C7 đề xuất | Tất định, không trùng lặp | Học phần gắn kỹ năng |

### 4.4 Hạn chế của phương án LLM API và cách xử lý

| Hạn chế | Cách xử lý |
|---|---|
| Kết quả có thể lệch giữa các lần gọi | Temperature 0, ép JSON schema, chấm hai lần khi độ tin cậy thấp, bộ golden set để đo định kỳ |
| Chi phí tăng theo số lần gọi, nhất là so khớp 1 JD với nhiều ứng viên | Lọc sơ bằng giao kỹ năng trước, chỉ gọi mô hình cho nhóm đầu; hạn mức token theo người dùng |
| Độ trễ 1–3 giây mỗi lần gọi | Gộp nhiều việc vào một lần gọi, cache phần prompt cố định, chạy nền các tác vụ hàng loạt |
| Lỗi parse JSON, từ chối trả lời, quá hạn mức | Gọi lại một lần, sau đó rơi về thuật toán đề xuất tương ứng |
| Khó giải thích vì sao ra điểm | Yêu cầu mô hình trả kèm bằng chứng trích từ đầu vào; đối chiếu với thuật toán đề xuất |

## 5. Kết luận

DevRoom giải quyết bài toán học và tìm việc rời rạc của sinh viên IT bằng một chu trình khép kín, trong đó kết quả phỏng vấn thử và so khớp JD quay lại điều chỉnh lộ trình học, và năng lực đã kiểm chứng trở thành thông tin cho nhà tuyển dụng. Đề tài triển khai bằng LLM API để có chất lượng xử lý ngôn ngữ và hội thoại ngay từ đầu, đồng thời đề xuất bộ thuật toán tất định làm phương án bổ sung cho các bước cần chi phí thấp, kết quả nhất quán và giải thích được. Hiện đề tài đã hoàn thành sơ đồ nghiệp vụ, thiết kế các lõi theo cả hai phương án, và bản demo giao diện với các thuật toán đề xuất chạy trực tiếp trong trình duyệt, xem tại `anhnt-24.github.io/sieuduan_md`.
