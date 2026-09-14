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
| **C1. Đánh giá năng lực** | Đo trình độ từng kỹ năng qua bài test ngắn | Mỗi kỹ năng có một điểm năng lực, mỗi câu hỏi có một điểm độ khó. Hệ thống chọn câu có độ khó gần năng lực hiện tại; trả lời đúng thì năng lực tăng, sai thì giảm, biên độ điều chỉnh giảm dần. Dừng khi điểm ổn định |
| **C2. Roadmap và bài tập thích ứng** | Sắp xếp học phần theo tiên quyết, chọn bài đúng độ khó, lên lịch ôn | Học phần tổ chức thành đồ thị có tiên quyết. Bài tập chọn trong dải độ khó quanh năng lực hiện tại, theo tag trùng kỹ năng thiếu. Lịch ôn giãn dần theo mức ghi nhớ |
| **C3. Sandbox** | Chạy code người học trong môi trường cô lập | Container giới hạn CPU, bộ nhớ, thời gian, không có mạng. So kết quả với bộ test |
| **C4. Xử lý CV** | Bóc tách kỹ năng, chấm điểm, gợi ý sửa | Đọc PDF/DOCX, nhận diện kỹ năng và kinh nghiệm, quy về từ điển chung. Chấm theo bộ tiêu chí. Sinh gợi ý sửa theo JD mục tiêu |
| **C5. Thu thập JD và so khớp** | Thu JD từ ba nguồn, tính tỉ lệ khớp | Thu thập theo lịch hoặc theo yêu cầu, bóc tách kỹ năng, khử trùng lặp. Tỉ lệ khớp tính từ phần kỹ năng trùng có trọng số và độ tương đồng tổng thể giữa CV và JD |
| **C6. Phỏng vấn thử** | Phỏng vấn bằng giọng nói, chấm điểm | Chuyển giọng nói thành văn bản, chấm theo khung STAR, quyết định hỏi thêm hay chuyển câu, đọc câu hỏi tiếp bằng giọng tổng hợp. Kết thúc có bảng điểm và kỹ năng yếu |
| **C7. Vòng phản hồi** | Chuyển kết quả C5, C6 thành đề xuất học | Kỹ năng thiếu hoặc yếu được ánh xạ sang học phần tương ứng, tạo đề xuất chờ người học duyệt |
| **C8. Trợ lý AI** | Trả lời về tiến độ, bước tiếp theo, việc phù hợp | Đọc dữ liệu của chính người dùng và trả lời có dẫn số liệu. Chỉ đọc, không sửa dữ liệu |

Các lõi C1, C2, C3, C7 là thuật toán tất định, không phụ thuộc mô hình AI. Các lõi C4, C5, C6, C8 có thành phần xử lý ngôn ngữ tự nhiên hoặc hội thoại.

## 4. Phương án công nghệ

Đề tài xem xét ba hướng triển khai. Phương án cuối cùng được xác định ở giai đoạn thiết kế chi tiết.

### 4.1 Ba hướng

| Hướng | Nội dung | Ưu điểm | Hạn chế |
|---|---|---|---|
| **A. LLM API** | Gọi mô hình ngôn ngữ lớn có sẵn (Claude, GPT, Gemini) qua API cho mọi thành phần cần hiểu hoặc sinh văn bản | Chất lượng cao, hỗ trợ tiếng Việt tốt, không cần phần cứng mạnh, thời gian phát triển ngắn | Chi phí theo lượng sử dụng, phụ thuộc dịch vụ ngoài, kết quả có thể thay đổi giữa các lần gọi |
| **B. Local** | Tự cài đặt thuật toán; dùng mô hình nhỏ chạy trên hạ tầng riêng: nhận diện thực thể, embedding, nhận dạng giọng nói, hoặc LLM mã nguồn mở (Qwen, Llama) | Không phát sinh chi phí theo lượt, không phụ thuộc ngoài, kiểm soát và giải thích được, khối lượng tự cài đặt lớn | Chất lượng hội thoại và xử lý văn bản tự do thấp hơn; cần GPU cho nhận dạng giọng nói thời gian thực hoặc LLM; công sức phát triển cao |
| **C. Hybrid** | Thuật toán và mô hình nhỏ chạy local; chỉ gọi LLM API ở các thành phần local không đáp ứng được | Giữ phần tự cài đặt, chi phí thấp, chất lượng cao ở thành phần cần thiết | Phải xác định rõ ranh giới gọi API |

### 4.2 Từng lõi theo từng hướng

| Lõi | A. LLM API | B. Local | C. Hybrid |
|---|---|---|---|
| **C1** | LLM tự đặt câu hỏi và chấm; kết quả không ổn định, khó đo lường | Thuật toán Elo kết hợp test thích ứng (CAT); không cần dữ liệu huấn luyện | Như B; LLM chỉ dùng một lần để gán độ khó ban đầu cho kho câu hỏi |
| **C2** | LLM sinh lộ trình từ mô tả; khó đảm bảo ràng buộc tiên quyết | Đồ thị tiên quyết, sắp xếp topo, thuật toán ôn ngắt quãng SM-2 | Như B; LLM sinh gợi ý khi người học làm sai |
| **C3** | Không áp dụng | Piston hoặc Judge0 | Như B |
| **C4** | LLM bóc tách kỹ năng, chấm điểm và viết gợi ý sửa | PyMuPDF, từ điển ESCO, nhận diện thực thể (spaCy, GLiNER), chấm theo bộ quy tắc | Bóc tách và chấm điểm local; LLM viết gợi ý sửa và xử lý kỹ năng không khớp từ điển |
| **C5** | LLM bóc tách kỹ năng từ JD; embedding qua API | Crawler, từ điển, embedding bge-m3, công thức so khớp tự cài đặt | Công thức so khớp và embedding local; LLM bóc tách kỹ năng từ JD và agent tìm việc trên web |
| **C6** | Nhận dạng và tổng hợp giọng nói qua API; LLM đóng vai người phỏng vấn, hỏi thêm, chấm STAR | Whisper tự triển khai (PhoWhisper cho tiếng Việt), kho câu hỏi cố định, hỏi thêm theo quy tắc, chấm theo từ khóa, tổng hợp giọng Piper | Nhận dạng và tổng hợp giọng nói chọn theo hạ tầng; LLM đảm nhiệm hội thoại và chấm điểm |
| **C7** | Logic tất định | Logic tất định | Logic tất định |
| **C8** | LLM có công cụ đọc dữ liệu người dùng | Trả lời theo kịch bản từ khóa, hoặc LLM mã nguồn mở chạy local | Như A |

### 4.3 So sánh

| Tiêu chí | A. LLM API | B. Local | C. Hybrid |
|---|---|---|---|
| Chất lượng trải nghiệm | Cao ở thành phần ngôn ngữ | Tốt ở thuật toán, hạn chế ở hội thoại | Cao ở thành phần cần thiết |
| Chi phí vận hành | Theo lượng sử dụng, ước tính 0,2–0,5 USD mỗi người dùng mỗi tháng | Gần bằng không nếu có sẵn hạ tầng | Thấp |
| Công sức phát triển | Thấp | Cao | Trung bình |
| Phụ thuộc dịch vụ ngoài | Cao | Không | Thấp, có thể thay nhà cung cấp |
| Hỗ trợ tiếng Việt | Tốt | Nhận dạng giọng nói cần PhoWhisper; LLM mã nguồn mở còn hạn chế | Như A ở thành phần gọi API |
| Yêu cầu phần cứng | Máy thông thường | Cần GPU cho giọng nói thời gian thực hoặc LLM local | Máy thông thường |
| Khối lượng tự cài đặt | Thấp | Cao | Cao ở lõi thuật toán |
| Khả năng giải thích kết quả | Thấp | Cao | Cao ở lõi thuật toán |

Chi phí ước tính theo giá công bố của các mô hình tầm trung, chưa bao gồm nhận dạng giọng nói.

## 5. Kết luận

DevRoom giải quyết bài toán học và tìm việc rời rạc của sinh viên IT bằng một chu trình khép kín, trong đó kết quả phỏng vấn thử và so khớp JD quay lại điều chỉnh lộ trình học, và năng lực đã kiểm chứng trở thành thông tin cho nhà tuyển dụng. Bốn trong tám lõi là thuật toán tất định, độc lập với lựa chọn công nghệ, nên có thể triển khai trước. Hiện đề tài đã hoàn thành sơ đồ nghiệp vụ, thiết kế các lõi và bản demo giao diện với các lõi thuật toán chạy trực tiếp trong trình duyệt, xem tại `anhnt-24.github.io/sieuduan_md`.
