# Setup "Bộ não" IDE: Mớm Context chuẩn cho AI

*Đừng để AI phải đoán mò cách build project của bạn.*

## Tại sao phải cấu hình Workspace IDE cho AI?

Anh em xài các AI Editor (Cursor, Windsurf, Copilot, hay JetBrains AI) chắc hay gặp cảnh này: Bảo nó build project hoặc chạy test, nó lại đi lấy mấy cái command chung chung ở đâu đâu (như gõ `npm run build` thay vì `pnpm build`, hay gọi sai script docker). Đơn giản là vì nó thiếu **context** (ngữ cảnh) của dự án.

Thư mục cấu hình workspace (như `.vscode` hay `.idea`) và các file rule chính là "bộ quy tắc" chuẩn nhất để mớm ngữ cảnh cho AI. Khi AI vào project, nếu thấy cấu hình rõ ràng, nó sẽ "đọc vị" được workflow của team ngay lập tức. Dưới đây là các chiêu thiết lập chuẩn chỉ nhất, được sắp xếp từ quan trọng nhất trở xuống:

---

## Global Rule (Luật toàn cục): Ép AI "thông minh đột xuất"

Muốn AI không viết code rác (boilerplate) như một ông thực tập sinh, bạn phải cung cấp cho nó một "Global Rule" (thường là file `.cursorrules`, `.windsurfrules` hoặc nhập vào phần *Custom Instructions* của IDE). Đây là cái System Prompt sẽ áp dụng lên MỌI câu chat của bạn trong dự án này.

**Ví dụ một Global Rule "cơm gạo":**
```markdown
Bạn là một Senior Frontend Architect cực kỳ khắt khe.
- KHÔNG BAO GIỜ dùng `any` trong TypeScript.
- Chỉ sử dụng Tailwind CSS 4.0, cấm dùng CSS Modules.
- Bắt buộc dùng `pnpm` thay cho `npm`.
- Code không vượt qua ESLint thì không được nhả ra.
- Luôn sử dụng TanStack Query cho việc gọi API, không dùng `useEffect` thuần.
```
Cài cái rule này vào xong, bạn chat một câu bâng quơ: *"Tạo cho anh component danh sách user"*, nó sẽ tự động sinh code TypeScript chuẩn chỉnh, dùng Tailwind và bọc sẵn TanStack Query. Trông như có phép thuật, nhưng thực chất là do mình đã "trấn yểm" luật lệ từ trước.

---

## Tối giản Extensions (`extensions.json`)

Nhiều anh em có thói quen nhồi nhét cả đống extensions vào IDE nhìn cho "ngầu". Nhưng thật lòng mà nói, đi làm cần tốc độ cao thì mình ưu tiên sự nhẹ nhàng. IDE khởi động chậm hay lag là rất bực mình.

Trong phần Extensions của project, mình gần như chả cài gì nặng nề để AI Editor chạy mượt nhất có thể. Tuy nhiên, có một "bảo bối" duy nhất mình khuyên anh em sống chết cũng phải cài: **Mermaid** (ví dụ `bierner.markdown-mermaid` trên VSCode).

**Tại sao lại là Mermaid?** 
Nó là công cụ vẽ biểu đồ (flowchart, architecture) bằng text (Markdown) mà bọn AI lại cực kỳ giỏi viết cái này.
- **Nắm kiến thức từ AI:** Khi bắt AI giải thích một luồng code cũ phức tạp, mình luôn bảo nó: *"Vẽ cho tao cái luồng bằng Mermaid"*. Có sẵn extension, mình xem được luôn biểu đồ ngay trong IDE cực kỳ trực quan, lười đọc chữ là hiểu ngay vấn đề.
- **Mớm kiến thức cho AI:** Ngược lại, khi thiết kế database hay luồng chạy mới, mình phác thảo vài dòng Mermaid ném vào context. AI đọc chữ là tự tưởng tượng ra được cả cấu trúc hệ thống. Giao tiếp giữa người và AI vừa nhanh vừa không sợ bị lệch sóng.

---

## "Thao túng" Build Tools qua `tasks.json` thần thánh

Nhiều anh em không nhận ra sự linh hoạt khủng khiếp của file `tasks.json`. Kể cả với những dự án Backend cực nặng như **Java (Spring Boot)**, thay vì phải mở IntelliJ hay Eclipse cồng kềnh, mình hoàn toàn có thể ôm trọn project chạy mượt mà ngay trên VSCode / Cursor chỉ bằng cách cấu hình chuẩn file này. Sự nhẹ nhàng này là thứ vô giá giúp tăng tốc độ làm việc.

Hơn nữa, AI Agent cực kỳ nhạy bén với file cấu hình tasks. Thay vì mất công chat dặn dò *"mày gõ lệnh mvn clean install để build nhé"*, mình vứt mọi thứ vào `tasks.json`. Khi AI cần chạy lệnh hay test code, nó tự chui vào đây đọc cấu trúc và lấy đúng script ra "giã".

**Ví dụ một file `.vscode/tasks.json` thực chiến (Build Java):**
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Build & Run Java Spring Boot",
      "type": "shell",
      "command": "mvn",
      "args": ["clean", "spring-boot:run"],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "detail": "Khi tôi bảo chạy hoặc test app Backend, hãy gọi task này bằng Maven. Nhớ đọc log terminal để xem app có start up thành công ở port 8080 không nhé."
    }
  ]
}
```
**Mẹo ăn tiền:** Hãy để ý trường `"detail"`. Đây là chỗ mình viết "lời nhắn gửi" bằng ngôn ngữ tự nhiên dành riêng cho AI. Nó đọc là hiểu ngay mục đích của task này dùng để làm gì, khi nào nên gọi, và đặc biệt là biết tự giác đọc log để bắt lỗi nếu build xịt.

---

## Đồng bộ Formatting và Rules với `settings.json`

Nhiều lúc AI sinh code ra bị lệch chuẩn indent, quên import, hoặc dùng single quote thay vì double quote. Thay vì dặn dò đi dặn lại bằng prompt rát cả cổ, mình ép rule trực tiếp vào Workspace Settings. AI tự nhìn vào đây và học theo luật của nhà mình.

**Ví dụ:** Dặn AI **lờ đi** các thư mục rác thông qua `ai.context.ignoredFiles` (hoặc `search.exclude`). Các IDE AI hiện tại đọc cái này rất tốt, giúp context window luôn sạch sẽ, AI không bị nhiễu thông tin khi đọc code toàn project.

---

## Quản lý luồng Debug với `launch.json`

Khi bảo AI *"tìm bug cho tao đoạn này"*, nếu project có setup file cấu hình debug, AI có thể hiểu được môi trường (Environment Variables) mà app đang chạy. Mở `launch.json` ra, AI biết luôn app chạy bằng Node, file entry là `index.ts` và PORT là `3000`. Cực kỳ tường minh!

---

## Khai mở "Siêu năng lực" với Skills & Skills Trigger

Các IDE/Editor tích hợp AI Agent xịn (như Cursor, Windsurf) bây giờ không chỉ dừng ở việc chat, mà nó có khả năng gọi các **Skills** (các quy trình, scripts tự động) thông qua **Skills Trigger**. Việc nhúng thiết lập này thẳng vào Workspace là một trong những thủ thuật "ăn tiền" nhất.

- **Skills là gì?** Là những quy trình nghiệp vụ được bạn đóng gói sẵn. Ví dụ: Skill "Check Quality" sẽ bao gồm chạy linter, chạy unit test, và format code.
- **Skills Trigger:** Là những từ khóa (keywords) hoặc hoàn cảnh khiến AI tự động kích hoạt Skill đó thay vì ngồi chat luyên thuyên.

**Thực chiến đi làm:**
Thay vì dặn dò AI bằng miệng, bạn khai báo hẳn các triggers vào file cấu hình (thường đặt ở `.cursorrules`, `.windsurfrules` hoặc config nội bộ của project):
```yaml
# Ví dụ cấu hình Trigger cho AI Agent
skills:
  - name: "Database Migration"
    trigger: ["migrate", "update schema", "thêm bảng"]
    action: "Chạy lệnh 'pnpm run db:migrate'. Đợi hoàn tất rồi đọc file error.log báo cáo kết quả."
```
Nhờ chiêu này, mỗi khi bạn chat: *"Anh vừa sửa entity, em chạy update schema nhé"*, AI sẽ quét thấy trigger `update schema`, ngay lập tức hiểu nó cần kích hoạt skill `Database Migration`, tự động mở terminal chạy script và báo cáo kết quả lại như một đồng nghiệp thực sự.

---

**Tóm lại:** Việc setup kĩ file cấu hình Workspace giống như bạn dọn dẹp nhà cửa gọn gàng trước khi mời đội ngũ AI vào làm việc. Nhà càng quy củ, nó làm việc càng chuẩn xác, ít bug và đặc biệt là không bao giờ bị "ngáo" khi dự án dần phình to ra.
