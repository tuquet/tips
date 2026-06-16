# Đồ nghề "Cơm Gạo" - Tối ưu tốc độ Build sản phẩm

Thú thật với mọi người, làm SaaS hay AI app thời điểm này mà cứ lóc cóc code từ con số 0 thì thời gian đâu mà validate thị trường hay chốt sale. Trải qua không ít project "sứt đầu mẻ trán", cá nhân mình nhận ra: **Tốc độ là sống còn, và bí quyết nằm ở việc biết dùng đúng "đồ nghề".**

Gần đây, "vũ khí bí mật" giúp mình đẩy tốc độ dev lên bàn thờ chính là các **MCP Server (Model Context Protocol)**. Thay vì phải tự viết script tích hợp lằng nhằng, mình cắm thẳng các server này vào AI Assistant để nó tự động tương tác với các công cụ bên ngoài. Cảm giác như mình thực sự có một đội ngũ đệ tử làm việc thay mình vậy.

Dưới đây là 3 MCP Server mà mình dùng nhiều nhất, như những người bạn đồng hành "vào sinh ra tử", được đúc kết lại theo cấu trúc 5 câu hỏi để anh em dễ cảm nhận:

---

## [Figma MCP Server](https://github.com/mcp/com.figma.mcp/mcp)

*Mỗi lần nhìn AI tự "hút" design rồi nhả ra code, cảm giác thực sự rất "phê"!*

- **Nó là gì?:** Cầu nối trực tiếp giữa AI và các bản thiết kế UI/UX trên Figma.
- **Tại sao lại chọn nó?:** Đi làm dự án, sợ nhất là cảnh khách hàng bắt sửa UI liên tục. Cái việc ngồi đo từng pixel, xuất từng mã màu hay viết đống CSS/Tailwind làm mình phát ốm. Có cái này, mình bắt AI tự đọc hiểu Design System và sinh ra code chuẩn luôn. Khách sửa một cái trên Figma, mình bảo AI update lại code phát một. Quá nhàn!
Đặc biệt, nó cứu mình những bàn thua trông thấy trong các "edge cases" cực kỳ thực tế khi làm outsource:
  - **Trị các bản design "vô kỷ luật":** Gặp khách tự vẽ hoặc UI/UX nghiệp dư vứt layer lộn xộn, không thèm gom nhóm (Auto-layout). Bình thường mình phải ngồi dọn file mất cả buổi, giờ quăng cho MCP, AI tự quét hình học, tự phân tích layout và nhả ra code HTML/Tailwind siêu sạch, bỏ qua sự bừa bộn của file gốc.
  - **Trích xuất Design Tokens siêu tốc:** Khởi tạo base dự án mệt nhất là đi copy mã màu, typography của khách vào `tailwind.config.js`. Giờ chỉ việc gắn link Figma, bảo AI: *"Lấy hết bộ màu chuẩn và font size cho tao"*, dán một phát vào config là code tẹt ga.
  - **Cân mọi trạng thái (Variants) phức tạp:** Cùng 1 cái Component nhưng khách vẽ đủ trạng thái: hover, disabled, outline, dark mode. Khỏi cần code chay từng class, AI đọc hiểu pattern và viết luôn một React Component hoàn chỉnh với đầy đủ biến thể (có thể dùng `cva`), bế thẳng vào project chạy ngay.
- **Khi nào lôi ra xài?:** Ngay khoảnh khắc chốt xong wireframe hoặc vừa nhận màn hình mới từ team UI/UX (hoặc lúc khách đổi ý bắt sửa layout).
- **Dùng ở khâu nào?:** Khâu Frontend, dựng Layout và tái cấu trúc UI Components.
- **Thực chiến xài ra sao?:** Mình ném thẳng link Figma cho AI qua MCP và bảo: *"Mày đọc màn hình này, bóc tách cho tao các component và viết bằng Tailwind 4.0 chuẩn React nhé"*. Xong mình đi pha ly cafe, quay lại là đã có đồ ngon để xài.

---

## [Supabase MCP Server](https://github.com/supabase/mcp)

*Thằng đệ mẫn cán lo toàn bộ phần Backend và Database.*

- **Nó là gì?:** Cổng giao tiếp cho phép AI can thiệp trực tiếp vào project Supabase (PostgreSQL, Auth, Storage). Điểm ăn tiền cực mạnh là thằng này có thể chạy trên Cloud, hoặc anh em tự kéo về deploy dưới Local hoàn toàn miễn phí vì kiến trúc nền tảng của nó được đóng gói bằng Docker.
- **Tại sao lại chọn nó?:** Anh em đi làm outsourcing chắc sẽ cực kỳ thấm cảnh này: Khách đòi xem tính năng gấp, mình thì cần mô phỏng một cái database dưới local nhanh nhất có thể để code và test. Thay vì tốn cả buổi cài cắm config, mở DBeaver gõ từng dòng SQL, mình dùng MCP kết nối thẳng với Supabase. Cần thao tác, tạo bảng, hay giao tiếp gì với DB thì cứ chat bằng mồm.
Chưa hết, dựng DB local bằng Supabase còn cực kỳ nhiều cái hay ho mà ít ai khai thác hết:
  - **Bơm dữ liệu giả (Seed Data):** Dựng DB xong rỗng tuếch thì test tính năng kiểu gì? Chỉ việc bảo MCP: *"Tạo cho tao 1000 dòng data giả cho bảng Users trông y như thật"*, AI tự sinh file `seed.sql` và bơm vào DB trong 5 giây.
  - **Tự động sinh Typescript Types:** Mỗi lần sửa schema, tự động kéo type mới nhất về. Frontend xài bao chặt chẽ, không bao giờ lo lỗi sai tên trường.
  - **Test Triggers / Functions (PL/pgSQL):** Viết mấy cái logic db ngầm, RLS lằng nhằng hay webhook... nhờ MCP test thẳng tay dưới local, hỏng thì reset, không sợ toang data khách.
- **Khi nào lôi ra xài?:** Từ ngày 1 của project lúc dựng Schema, hoặc những lúc đang code dở cần thêm gấp một table, trigger, hay function mới để demo cho khách.
- **Dùng ở khâu nào?:** Database Management, Backend Logic và Authentication.
- **Thực chiến xài ra sao?:** Quy trình chuẩn khi làm cho khách thường sẽ thế này: Đầu tiên, mình tải schema từ DB staging của khách hàng về, sau đó dùng Supabase MCP import vào để dựng lại y chang cấu trúc DB dưới local cực nhanh. Khi có yêu cầu mới (thêm bảng, sửa cột), mình yêu cầu AI phân tích và xử lý. Input là prompt, còn Output mình bắt AI nhả ra là những bản file `.sql` migration cực kỳ chuẩn chỉnh để gửi thẳng cho khách cập nhật trên DB của họ. Vừa an toàn, vừa vô cùng chuyên nghiệp.

---

## [Memplace MCP Server](https://github.com/mempalace/mempalace)

*Trạm gác "não bộ" - Nơi lưu trữ ký ức dài hạn để AI không bị "ngáo".*

- **Nó là gì?:** Bộ nhớ phụ (Knowledge Base / Context Management) dành riêng cho AI khi làm việc với project của mình.
- **Tại sao lại chọn nó?:** Nhận mấy cái project to to hay maintain source code cũ là hay bị tình trạng AI "mất não". Chat một lúc là nó quên béng convention của dự án hay những rule mình đã dặn. Memplace giải quyết triệt để vụ này. Mọi quyết định kiến trúc, mọi rule của khách hàng mình vứt hết vào đây. AI giống như một ông dev cứng, tự động hiểu bối cảnh.
Nhưng đi làm thực tế, Memplace mới thực sự phát huy sức mạnh cứu mạng ở mấy cái "edge cases" đẫm nước mắt này:
  - **Trị bệnh "bịa code" ở dự án khổng lồ:** Project phình to lên hàng trăm files, AI rất hay ảo giác (hallucinate) tự gọi mấy cái component hoặc package không hề tồn tại. Mình dùng Memplace ép nó phải query lại `package.json` và bảng index thư mục trước khi đẻ ra bất cứ dòng code nào.
  - **Né "bom mìn" từ Legacy Code:** Đi maintain code do mấy "pháp sư" dự án cũ để lại, có những đoạn logic cực dị. Mình dò được bug nào là note ngay vào Memplace. Lần sau AI mà đụng tới module đó, nó tự móc "bí kíp" ra đọc để biết đường né, không đạp mìn lại lần 2.
  - **Chạy show nhiều dự án cùng lúc (Context Isolation):** Sáng code React cho khách A, chiều maintain Next.js cho khách B. Xài AI rất dễ bị "nhập ma" lấy râu ông nọ cắm cằm bà kia. Với Memplace, mình trỏ MCP vào đúng phân vùng của dự án nào thì AI tải đúng "bộ não" của dự án đó xuống. Ngọt lịm và không bao giờ bị lẫn lộn bối cảnh!
- **Khi nào lôi ra xài?:** Bất cứ lúc nào mình join vào một project lớn, nhiều file, hoặc maintain dự án kéo dài từ tháng này sang tháng nọ.
- **Dùng ở khâu nào?:** Quản lý Context, Prompt Engineering và lưu trữ tài liệu kiến trúc.
- **Thực chiến xài ra sao?:** Mình ném toàn bộ tài liệu cấu trúc thư mục, coding conventions, và các yêu cầu "trời ơi đất hỡi" của khách vào Memplace. Mỗi lần bắt đầu task mới, AI tự động query qua MCP để lôi đúng "ký ức" ra đọc trước khi viết code. Code ra đúng ý luôn ngay từ nhát đầu tiên.

---
*Hy vọng những chia sẻ "ruột gan" này sẽ giúp anh em tìm được workflow ưng ý và bứt tốc trong những project sắp tới!*
