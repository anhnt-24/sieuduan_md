# DevRoom — Báo cáo đề tài

> Báo cáo ngắn để xin ý kiến giảng viên. **Nhóm chưa chốt công nghệ.** Mục 4 đặt ba hướng triển khai cạnh nhau cho từng lõi, mục 5 là các câu hỏi nhóm cần thầy/cô định hướng trước khi quyết định.

## 1. Đề tài

DevRoom là nền tảng dành riêng cho lập trình viên và các vị trí IT (Frontend, Backend, Mobile, DevOps, Data, QA). Nó gộp ba việc vốn rời rạc thành một chu trình khép kín:

**Học đúng lỗ hổng → luyện phỏng vấn → tối ưu CV và tìm việc → kết quả quay lại sửa lộ trình học.**

**Bài toán.** Sinh viên IT học dàn trải, không biết mình thiếu gì so với yêu cầu tuyển dụng thật. Luyện phỏng vấn không có ai phản hồi. CV viết không khớp mô tả công việc (JD). Nhà tuyển dụng chỉ đọc CV chữ, không thấy năng lực thật.

**Điểm mới** không nằm ở từng phân hệ (mỗi phân hệ đều đã có sản phẩm riêng lẻ trên thị trường) mà ở hai mắt xích nối chúng lại:

1. Câu trả lời yếu trong phỏng vấn thử và kỹ năng còn thiếu so với JD tự động trở thành đề xuất học bù. Người học chọn nhận hay bỏ qua.
2. Năng lực đã kiểm chứng qua học và phỏng vấn thử trở thành bằng chứng cho nhà tuyển dụng, thay cho chữ trên CV.

## 2. Nghiệp vụ

### 2.1 Bốn vai trò

| Vai trò | Làm gì trên hệ thống |
|---|---|
| **Candidate** (sinh viên, ứng viên IT) | Tạo phòng học theo career, làm test đầu vào, học theo lộ trình thích ứng, làm CV, match việc, phỏng vấn thử với AI |
| **Lecturer** (giảng viên) | Thiết kế học phần, bài tập có gắn tag, lộ trình mẫu cho từng career |
| **Recruiter** (nhà tuyển dụng) | Đăng JD, xem ứng viên xếp theo năng lực thực, mời phỏng vấn thật |
| **Admin** | Theo dõi log và chi phí AI, quản lý tiền |

### 2.2 Chu trình của Candidate

![Luồng tổng quan](LUONG-TONG-QUAN.png)

| # | Bước | Kết quả |
|---|---|---|
| 1 | Tạo phòng học, mỗi phòng là một career | Có thể có nhiều phòng song song |
| 2 | Test đầu vào thích ứng | Điểm năng lực từng kỹ năng, danh sách kỹ năng còn thiếu |
| 3 | Roadmap học | AI chỉ gợi ý từ lộ trình mẫu của Lecturer và kỹ năng thiếu. Candidate tự thêm, bớt học phần |
| 4 | Học thích ứng | Bài tập theo ngày hoặc tuần đúng độ khó, flashcard ôn ngắt quãng, sandbox code kiểu LeetCode |
| 5 | CV | AI tạo hoặc tự viết. AI chấm điểm và gợi ý sửa theo JD mục tiêu |
| 6 | Job / JD matching | JD từ ba nguồn: crawl, agent tìm trên web, Recruiter đăng. Tính % khớp và kỹ năng thiếu |
| 7 | Mock interview | Hỏi đáp bằng giọng nói, hỏi vặn khi trả lời nông, chấm theo STAR |
| 8 | Đánh giá phỏng vấn thật | Tùy chọn, ghi lại sau khi phỏng vấn với Recruiter |

**Vòng phản hồi**: sau bước 6 và 7, hệ thống sinh đề xuất cải thiện lộ trình. Candidate chọn nhận hoặc bỏ qua, roadmap ở bước 3 cập nhật theo.

### 2.3 Cái gì giữ ba phân hệ dính vào nhau

Một **từ điển kỹ năng chung** (lấy từ bộ ESCO của EU, hơn 13.000 kỹ năng, có quan hệ cha con). CV, JD, học phần, câu hỏi test, câu hỏi phỏng vấn đều gắn về cùng một mã kỹ năng. Nhờ vậy điểm phỏng vấn mới "chảy" được về roadmap, và kỹ năng đã học mới so được với JD. Nếu thiếu từ điển này, ba phân hệ chỉ là ba ứng dụng rời dùng chung đăng nhập.

## 3. Các lõi và cách hoạt động

| Lõi | Làm gì | Hoạt động ra sao | Vào → Ra |
|---|---|---|---|
| **C1. Đánh giá năng lực** | Đo trình độ từng kỹ năng bằng test ngắn | Mỗi kỹ năng có một điểm năng lực, mỗi câu hỏi có một điểm độ khó. Chọn câu có độ khó gần năng lực hiện tại, trả lời đúng thì năng lực tăng, sai thì giảm, biên độ giảm dần khi đã có nhiều dữ liệu. Dừng khi điểm ổn định | Bài làm → điểm năng lực 0–100 theo kỹ năng, danh sách kỹ năng thiếu |
| **C2. Roadmap và bài tập thích ứng** | Xếp học phần đúng thứ tự tiên quyết, chọn bài đúng độ khó, nhắc ôn đúng lúc | Học phần là đồ thị có tiên quyết. Roadmap là các học phần chưa đạt, xếp theo thứ tự đồ thị. Bài tập chọn trong dải độ khó quanh năng lực hiện tại, theo tag trùng kỹ năng thiếu. Flashcard lên lịch ôn giãn dần theo mức nhớ | Năng lực + lộ trình mẫu → roadmap; năng lực + kho bài tập → bài hôm nay |
| **C3. Code sandbox** | Chạy code người học trong môi trường cô lập | Gửi code và test case vào container giới hạn CPU, RAM, thời gian, không có mạng. So kết quả với đáp án | Code → số test đạt, lỗi nếu có |
| **C4. CV** | Bóc tách kỹ năng, chấm điểm, gợi ý sửa | Đọc PDF/DOCX ra chữ, nhận diện kỹ năng và số năm kinh nghiệm, quy về từ điển chung. Chấm theo bộ quy tắc (có số liệu trong mô tả không, có link GitHub không, thiếu kỹ năng nào so với JD mục tiêu). Sinh gợi ý sửa | File CV → kỹ năng, điểm 0–100, danh sách gợi ý |
| **C5. JD và matching** | Thu JD từ ba nguồn, tính độ khớp | Crawl trang việc làm theo lịch, hoặc agent tìm trên web theo vị trí mục tiêu, hoặc Recruiter nhập. Bóc kỹ năng, khử trùng lặp. % khớp = phần kỹ năng trùng có trọng số, cộng độ giống tổng thể giữa CV và JD | CV + JD → % khớp, kỹ năng thiếu; một JD → danh sách ứng viên xếp hạng |
| **C6. Mock interview** | Phỏng vấn thử bằng giọng nói, chấm điểm | Ghi âm câu trả lời → chuyển thành chữ → "người phỏng vấn" chấm theo rubric STAR và quyết định hỏi vặn hay chuyển câu → đọc câu hỏi tiếp bằng giọng máy. Kết thúc có bảng điểm từng câu, kỹ năng yếu | Giọng nói → transcript, điểm STAR, kỹ năng yếu |
| **C7. Vòng phản hồi** | Biến kết quả C5, C6 thành việc học | Kỹ năng thiếu hoặc yếu → tìm học phần dạy kỹ năng đó → tạo đề xuất, không trùng với cái đã có. Candidate quyết | Kỹ năng yếu → đề xuất chờ duyệt |
| **C8. Trợ lý AI riêng** | Trả lời câu hỏi về tiến độ, bước tiếp theo, việc phù hợp | Đọc dữ liệu của chính người đó (roadmap, năng lực, CV, việc đã match) rồi trả lời. Chỉ đọc, không tự sửa dữ liệu | Câu hỏi → câu trả lời có dẫn số liệu thật |

Lưu ý: **C1, C2, C3, C7 là thuật toán tất định**, không có phần "hiểu ngôn ngữ", nên không cần model AI nào ở lõi. C4, C5, C6, C8 có phần đọc văn bản tự do hoặc hội thoại, là nơi câu hỏi chọn công nghệ đặt ra.

## 4. Ba hướng công nghệ

### 4.1 Định nghĩa

| Hướng | Nội dung | Điểm mạnh | Điểm yếu |
|---|---|---|---|
| **A. LLM API** | Gọi model ngôn ngữ có sẵn (Claude, GPT, Gemini) qua API cho mọi việc cần hiểu hoặc sinh văn bản. Không train | Chất lượng cao ngay, tiếng Việt ổn, không cần máy mạnh, phát triển nhanh | Trả tiền theo lượng dùng, phụ thuộc dịch vụ ngoài, kết quả có thể thay đổi giữa các lần gọi, khó giải thích vì sao ra kết quả đó |
| **B. Local** | Tự cài đặt thuật toán, dùng model nhỏ chạy trên máy mình: nhận diện thực thể, embedding, nhận dạng giọng nói tự host, hoặc LLM mã nguồn mở (Qwen, Llama) chạy qua Ollama | Không tốn tiền theo lượt, không phụ thuộc ngoài, kiểm soát và giải thích được, thể hiện được kiến thức tự cài đặt | Chất lượng hội thoại và đọc văn bản tự do thấp hơn rõ. Cần GPU nếu muốn nhận dạng giọng nói nhanh hoặc chạy LLM. Công sức phát triển cao |
| **C. Hybrid** | Thuật toán và model nhỏ chạy local. Chỉ gọi LLM API ở đúng những chỗ local làm không tới | Giữ được phần tự cài đặt, chi phí thấp, chất lượng cao ở chỗ cần | Hai nhóm kỹ năng phải làm cùng lúc, phải có ranh giới rõ gọi API ở đâu |

### 4.2 Từng lõi theo từng hướng

| Lõi | Hướng A: LLM API | Hướng B: Local | Nhận xét |
|---|---|---|---|
| **C1 Đánh giá năng lực** | LLM tự đặt câu hỏi và tự chấm. Không ổn định, không đo được | Thuật toán xếp hạng kiểu Elo kết hợp test thích ứng (CAT). Khoảng 50 dòng code, không cần dữ liệu train | Bản chất là bài toán thuật toán, LLM không có vai trò ở lõi. LLM chỉ hữu ích một lần lúc gán độ khó ban đầu cho kho câu hỏi |
| **C2 Roadmap, bài tập** | LLM sinh lộ trình từ mô tả. Khó nhất quán, không kiểm soát được tiên quyết | Đồ thị tiên quyết + sắp xếp topo + thuật toán ôn ngắt quãng SM-2. Tất định | Local. Riêng gợi ý khi làm sai: LLM sinh gợi ý theo ngữ cảnh, local thì dùng gợi ý Lecturer viết sẵn theo từng bài |
| **C3 Sandbox** | Không áp dụng | Piston hoặc Judge0 chạy container cô lập | Chỉ có local |
| **C4 CV** | LLM đọc CV trả về JSON kỹ năng, chấm và viết gợi ý sửa tự nhiên. Chất lượng cao, khoảng một cent mỗi CV | Bóc chữ bằng PyMuPDF, dò kỹ năng bằng từ điển ESCO và mô hình nhận diện thực thể (spaCy, GLiNER), chấm bằng bộ quy tắc | Điểm số làm bằng quy tắc local được và giải thích được. Bỏ sót cách viết lạ ("ReactJS" và "React") và không viết được gợi ý sửa tự nhiên nếu không có LLM |
| **C5 JD, matching** | LLM bóc kỹ năng từ JD, embedding qua API | Crawler + từ điển + embedding bge-m3 chạy CPU + công thức khớp tự viết | Công thức khớp là local. Bóc kỹ năng từ JD viết tự do là chỗ LLM hơn hẳn. Agent tìm JD trên web bắt buộc cần LLM |
| **C6 Mock interview** | Nhận dạng giọng nói và đọc giọng máy qua API, LLM đóng vai người phỏng vấn, hỏi vặn thật, chấm STAR. Khoảng 5 đến 10 cent mỗi buổi | Whisper tự host (PhoWhisper cho tiếng Việt, cần GPU để không chậm), kho câu hỏi cố định, hỏi vặn theo quy tắc, chấm theo từ khóa, giọng máy Piper. Chạy được nhưng cứng, chấm thô. Có thể dùng LLM mở 7B chạy máy mình, tiếng Việt kém hơn API rõ | Lõi chênh lệch chất lượng lớn nhất giữa hai hướng. Bản demo hiện tại chính là hướng B của lõi này |
| **C7 Vòng phản hồi** | Không cần | Logic tất định | Local |
| **C8 Trợ lý AI** | LLM có công cụ đọc dữ liệu người dùng | Trả lời theo kịch bản từ khóa, hoặc LLM mở local | Không có LLM thì không phải trợ lý, chỉ là menu có chữ |

### 4.3 So sánh tổng thể

| Tiêu chí | A. LLM API | B. Local | C. Hybrid |
|---|---|---|---|
| Chất lượng trải nghiệm | Cao ở phần ngôn ngữ | Tốt ở thuật toán, yếu ở hội thoại và đọc văn bản tự do | Cao ở chỗ cần, tốt ở phần còn lại |
| Chi phí vận hành | Theo lượng dùng. Ước thô 0,2 đến 0,5 USD mỗi người dùng mỗi tháng nếu dùng model tầm trung | Gần bằng không nếu có sẵn máy | Thấp, vì chỉ gọi API ở ba lõi |
| Công sức phát triển | Thấp nhất | Cao nhất | Trung bình |
| Phụ thuộc ngoài | Cao | Không | Chỉ ở phần gọi API, có thể đổi nhà cung cấp |
| Tiếng Việt | Ổn với các model lớn | Nhận dạng giọng nói cần PhoWhisper. LLM mở tiếng Việt yếu | Như A ở chỗ gọi API |
| Hạ tầng | Máy thường | Cần GPU nếu muốn nhận dạng giọng nói nhanh hoặc chạy LLM local | Máy thường |
| Phần tự cài đặt thể hiện kiến thức | Thấp, chủ yếu là viết prompt | Cao | Cao ở lõi thuật toán, thấp ở phần gọi API |
| Kiểm soát, giải thích được | Thấp | Cao | Cao ở lõi, thấp ở phần gọi API |

Con số chi phí là ước lượng thô theo giá công bố của model tầm trung, chưa tính nhận dạng giọng nói. Sẽ tính lại khi chốt hướng.

## 5. Câu hỏi xin ý kiến giảng viên

1. Với yêu cầu của đồ án, thầy/cô ưu tiên **phần tự cài đặt** (hướng B) hay **chất lượng trải nghiệm** (hướng A)?
2. Lõi mock interview: bản chấm theo quy tắc, kho câu hỏi cố định (hướng B) có được chấp nhận làm phiên bản đầu không, hay cần LLM ngay từ đầu?
3. Đồ án có ràng buộc **không dùng dịch vụ trả phí** không? Nếu có, hướng A bị loại.
4. Nhóm có được dùng **GPU của trường hoặc lab** không? Nếu có, hướng B mở rộng được sang nhận dạng giọng nói và LLM local.
5. Phạm vi: làm **đủ ba phân hệ ở mức chạy được đầu cuối**, hay làm **sâu một phân hệ**?

## 6. Hiện trạng

| Đã có | Chưa có |
|---|---|
| Sơ đồ luồng nghiệp vụ bốn vai trò | Chốt hướng công nghệ |
| Báo cáo tổng quan và tài liệu kỹ thuật lõi. Hai tài liệu này viết theo một phương án sơ bộ nghiêng về hybrid để nhóm hình dung, **chưa phải quyết định** | Backend và cơ sở dữ liệu thật |
| Web demo chạy được, mọi lõi đều là thuật toán local, chưa gọi model AI nào. Xem tại `anhnt-24.github.io/sieuduan_md`. Đây chính là hình hài của hướng B | Kho dữ liệu thật: từ điển kỹ năng, kho câu hỏi, JD |
