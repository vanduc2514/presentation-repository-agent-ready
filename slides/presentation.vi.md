<!--markpress-opt
{
  "autoSplit": false,
  "sanitize": false,
  "title": "Xây dựng repository thân thiện với agent"
}
markpress-opt-->

<!--slide-attr x=0 y=0 scale=1.2 -->

# Xây dựng repository thân thiện với agent
## Những tinh chỉnh nhỏ giúp coding agent làm việc hiệu quả hơn trong codebase của bạn

Tên của bạn · Tên sự kiện · 2026

<!-- SPEAKER NOTES
Mở đầu bằng việc làm rõ phạm vi: đây là một chủ đề cụ thể và rất thực tế. Coding agent vốn dĩ đã có thể làm việc trên một repo bình thường mà không cần thiết lập gì đặc biệt. Những tối ưu ở đây chỉ giúp nó làm việc hiệu quả và mượt mà hơn.
Khán giả hướng tới: các bạn sinh viên và lập trình viên mới đi làm, những người đã từng dùng coding agent và không ít lần thấy nó gặp lỗi rất khó hiểu.
Hình thức: thuyết trình chia sẻ kinh nghiệm, không live demo, không bài tập thực hành. Mọi người không cần cài đặt gì trước; các thiết lập này nhằm mục đích để các bạn có thể tự áp dụng lại sau buổi nói chuyện.
Ngoài phạm vi: không bàn về việc lựa chọn model hay kỹ thuật viết prompt. Model được giữ cố định xuyên suốt buổi chia sẻ.
Môi trường chạy (Harness): các ví dụ minh họa dùng Copilot trong VS Code. Tên hook và frontmatter của skill sẽ dùng chuẩn của Copilot; các harness khác đều có những khái niệm tương đương dù tên gọi có thể khác.
-->

------

<!--slide-attr x=3600 y=0 rotate=2 scale=1.0 -->

# Những "biểu hiện lạ" thường gặp khi sử dụng Agent

- Code nhìn qua tưởng đúng, nhưng deploy lên hoặc đến tay người dùng thật mới lỗi
- Tự ý sửa vào file generated, file `.env`, hoặc những phần code không liên quan
- Session cần con người can thiệp và nắn chỉnh liên tục mới đi đúng hướng
- Session kéo dài lê thê, phần lớn thời gian chỉ để tìm kiếm và đoán lệnh
- Lượng token tiêu tốn quá lớn, không tương xứng với một thay đổi nhỏ
- Thời gian ngồi review và vá lỗi triệt tiêu luôn phần thời gian agent tiết kiệm được

<!-- SPEAKER NOTES
Bắt đầu bằng việc thừa nhận một điều mà khán giả có thể đang băn khoăn: agent thực ra vốn đã chạy được trên một repo bình thường mà không cần cài cắm gì cao siêu. Nó vẫn đọc file, chạy lệnh, sửa code và báo cáo lại; với một thay đổi nhỏ và cục bộ, nó thường làm xong. Không có điều gì trong buổi chia sẻ này là điều kiện tiên quyết để agent có thể bắt đầu chạy. Một repo mặc định không có nghĩa là một repo có vấn đề.
Vì vậy, câu hỏi ở đây không phải là "agent có chạy được không". Câu hỏi là bao nhiêu công sức của agent đang bị lãng phí chỉ để mò mẫm những thông tin mà repo vốn dĩ đã biết sẵn. Chủ đề hôm nay là ma sát và hao tổn tài nguyên, chứ không phải năng lực của mô hình.
Sự thất bại của agent hiếm khi là một cú crash rõ ràng. Nó diễn ra âm thầm hơn nhiều, và mỗi gạch đầu dòng ở đây đều là trải nghiệm mà phần lớn anh em trong phòng đều đã từng gặp phải.
Có hai nghiên cứu đã công bố rất đáng chú ý về bối cảnh này: một file AGENTS.md do chính repo duy trì giúp agent tiêu tốn ít token hơn và session kết thúc nhanh hơn (Lulla et al., 2026). Ngược lại, việc dùng công cụ tự động sinh file context không hề cải thiện tỉ lệ thành công mà còn làm chi phí đội lên (Gloaguen et al., 2026).
Cũng cần nói rõ: cả hai nghiên cứu không đo đạc chính xác các dạng lỗi cụ thể này và cũng không chạy trên chính repo của các bạn. Chúng mang tính cung cấp bối cảnh tham khảo, không phải là kết quả thực nghiệm tại chỗ.
-->

------

<!--slide-attr x=5400 y=0 rotate=-1 scale=1.0 -->

# Vì sao lại có những biểu hiện này ?

Khi bắt đầu session, agent hoàn toàn chưa có hình dung gì về repo của bạn.

- Không nói rõ lệnh build: agent đoán mò, sai lại đoán tiếp
- Feedback loop quá chậm: agent sửa đi sửa lại mà không biết có đang đi đúng hướng không
- Không nêu rõ conventions: agent tiện đâu copy đấy theo ví dụ gần nhất
- Không vạch rõ ranh giới: không có gì cản agent sửa vào bất kỳ file nào hay file nhạy cảm hay chạy nhầm lệnh
- Kiến thức nằm ngoài repository: agent hoàn toàn không thể nhìn thấy

> Mỗi câu hỏi chưa được codebase giải đáp đều biến thành một lần suy đoán của agent.
> Mà suy đoán thì đốt token, tốn thời gian và dễ đẻ ra bug.

<!-- SPEAKER NOTES
Một đồng nghiệp mới vào team sẽ tích lũy hiểu biết dần qua từng tuần. Còn agent thì khác: bước vào session, nó chỉ có chỉ dẫn (instructions), lịch sử chat và các tools, ngoài ra không có gì trong đầu cả. Mọi thứ nó cần đều phải đến từ file trong repo, từ hội thoại hoặc qua tool calls.
Điểm qua năm gạch đầu dòng trên: mỗi ý thực chất là một câu hỏi mà model buộc phải tự trả lời bằng cách suy đoán khi repo không nói rõ cho nó biết.
Đây chính là cơ chế nền tảng mà toàn bộ phần còn lại của buổi nói chuyện dựa vào. Luận điểm này được đúc rút từ cơ chế vận hành của agent và được kiểm chứng qua quá trình theo dõi thực tế, chứ không phải một giả định lý thuyết.
-->

------

<!--slide-attr x=7200 y=0 rotate=2 scale=1.0 -->

# Tối ưu môi trường làm việc thay vì đổi model

- Không làm model thông minh hơn, mà giải phóng nó khỏi những việc phải mò mẫm lặp đi lặp lại
- Model sẽ hiệu quả hơn khi làm việc trong một môi trường "lý tưởng"

<!-- SPEAKER NOTES
Model trước hay sau khi tối ưu vẫn là một. Thứ thay đổi duy nhất là: bao nhiêu phần trong "ngân sách" (token và sự chú ý) của nó bị tiêu tốn cho việc tìm hiểu repo, và bao nhiêu được dành trọn vẹn để giải quyết task.
Buổi chia sẻ này không bàn về: đổi sang model xịn hơn hay học mẹo viết prompt hay hơn — cả hai điều đó đều quan trọng nhưng không phải trọng tâm hôm nay. Chúng ta cũng không khẳng định rằng cứ thiết lập xong là agent sẽ luôn chạy đúng. Đây không phải một bài nghiên cứu đo đạc số liệu, càng không phải bảng checklist bắt bạn phải cài đặt mọi thứ. Một vài hạng mục chỉ nên áp dụng khi có điều kiện phát sinh, và có những thứ hoàn toàn là tùy chọn.
-->

------

<!--slide-attr x=1800 y=1300 rotate=-2 scale=1.0 -->

# Các kỹ thuật để tối ưu môi trường cho agent

| Hạng mục | Khi nào nên áp dụng? |
|---|---|
| AGENTS.md | Khi agent hay phải đoán lệnh hoặc conventions |
| Scripts | Khi một quy trình thao tác bị lặp lại |
| Skills | Khi quy trình vừa lặp lại, vừa cần hướng dẫn chi tiết từng bước |
| Hooks | Khi có nguy cơ agent chạm vào những vùng không được phép sửa |
| MCP | Khi thông tin cần thiết nằm ngoài phạm vi checkout |
| Orchestration | Khi điểm nghẽn nằm ở khâu review chứ không phải khâu sinh code |

<!-- SPEAKER NOTES
Đây là bản đồ ánh xạ từ 6 câu hỏi ở slide trước sang từng hạng mục giải pháp, kèm theo điều kiện khi nào thì nên triển khai. Không hề có một thứ tự bắt buộc nào ở đây.
Các giải pháp này xếp tầng theo một tư duy logic: AGENTS.md đưa ra câu trả lời; scripts tự động hóa các thao tác đó; skills đóng gói toàn bộ quy trình; còn hooks là rào chắn kỹ thuật siết chặt quy định. Cách xếp tầng giúp chúng ta hiểu cách các thành phần ăn khớp với nhau, chứ không có nghĩa là bạn phải làm tuần tự từ trên xuống dưới.
Bốn hạng mục đầu tiên là các file nằm gọn trong checkout. MCP là một server dịch vụ kết hợp cấu hình phía client. Còn Orchestration là công cụ mà con người dùng để điều phối các agent, chứ không làm thay đổi gì trong repo cả. Khi đi vào chi tiết từng phần, chúng ta sẽ làm rõ điều này.
-->

------

<!--slide-attr x=3600 y=1300 rotate=2 scale=1.0 -->

# AGENTS.md

![Ảnh chụp: AGENTS.md bên cạnh session đã dùng nó.](images/placeholder-agents-md.svg)

<!-- SPEAKER NOTES
Một file ngắn gọn, do chính tay con người viết: tổng hợp lệnh chạy, cấu trúc thư mục vắn tắt, conventions, ranh giới cấm chạm, và tiêu chuẩn thế nào là xong (Done).
Chỉ vào các dòng cụ thể trong file và hành động tương ứng của agent trong session.
Về chi phí context: mỗi session agent sẽ phải đọc file này, nên hãy giữ file thật ngắn gọn, nếu viết quá dài thì chi phí token sẽ lấn át lợi ích.
Đặc biệt lưu ý: hãy tự tay viết, tránh dùng tool tự động sinh context file vì nghiên cứu cho thấy nó chỉ làm tăng chi phí mà không giúp ích gì. Không đưa vào: bản copy README, bài luận kiến trúc dài dòng, secret hay các bản tóm tắt tự động.
-->

------

<!--slide-attr x=7200 y=1300 rotate=2 scale=1.0 -->

# Scripts

- Mỗi mục đích một lệnh rõ ràng: `setup`, `verify`, `test-one <path>`, `reproduce <issue>`
- Chạy không tương tác (non-interactive), exit code chuẩn xác, giới hạn lượng output, fail-fast
- Hỗ trợ cờ `--json` khi cần dữ liệu có cấu trúc, tự tắt màu khi output không phải TTY

![Ảnh so sánh: Lệnh verify khi thất bại bên trái, thành công bên phải.](images/placeholder-scripts.svg)

<!-- SPEAKER NOTES
Scripts giúp giảm bớt số vòng lặp agent phải loay hoay thử sai để xem dự án này chạy thế nào, đồng thời cắt ngắn những session dài lê thê mà phần lớn chỉ để xử lý các câu lệnh chạy lỗi.
Chi phí context ở đây rất thấp. Agent chỉ cần biết tên lệnh, và tên lệnh thì đã được nêu rõ ràng trong AGENTS.md. Tuy nhiên, output của script sẽ được đưa thẳng vào context của session, vì thế việc khống chế dung lượng output in ra là cực kỳ quan trọng.
Bạn sẽ thấy ngay giá trị của scripts ngay khi một quy trình thao tác bắt đầu có tần suất lặp lại.
-->

------

<!--slide-attr x=0 y=2600 rotate=-1 scale=1.0 -->

# Skills

- Đóng gói quy trình lặp lại vào một thư mục riêng kèm file `SKILL.md`
- Cơ chế nạp lũy tiến (progressive loading): đọc name và description trước, chỉ tải phần nội dung khi thật sự cần
- Tách riêng template và fixtures ra các file độc lập
- Kiểm thử kỹ từng skill với task mới mà agent chưa từng gặp
- Không đóng gói những việc chỉ làm một lần

![Ảnh chụp: Một skill được kích hoạt, hiển thị name và description trước khi tải toàn bộ nội dung.](images/placeholder-skills.svg)

<!-- SPEAKER NOTES
Skills giúp khắc phục tình trạng thiếu nhất quán khi thực hiện một quy trình — vốn dĩ chạy rất mượt khi có người nhớ đủ các bước, nhưng lại hay bị sót việc khi giao cho agent.
Chi phí context cho skills rất thấp nhờ cơ chế nạp lũy tiến: ban đầu agent chỉ đọc lướt tên và mô tả ngắn; chỉ khi nào gặp task phù hợp, nó mới tải toàn bộ nội dung chi tiết vào session.
Chỉ nên đóng gói skill khi cùng một quy trình lặp đi lặp lại qua nhiều task và đòi hỏi nhiều chi tiết cụ thể. Nếu không cần quá nhiều bước hướng dẫn, chỉ một dòng lệnh trong AGENTS.md hoặc một đoạn script là đã đủ.
-->

------

<!--slide-attr x=1800 y=2600 rotate=2 scale=1.0 -->

# Hooks

- Câu lệnh kích hoạt tự động theo sự kiện của harness, không phụ thuộc vào model
- Vòng đời sự kiện trong Copilot: `preToolUse`, `postToolUse`, `sessionStart`, `sessionEnd`
- Bắt đầu nhẹ nhàng bằng audit hook: chỉ ghi log lại tên các tools được gọi
- Khi đã có dữ liệu, mới siết chặt rào chắn (enforce) ở những paths và câu lệnh nhạy cảm hoặc dễ gây lỗi

![Ảnh chụp: Một thao tác sửa file bị hook chặn lại và thông báo phản hồi mà agent nhận được.](images/placeholder-hooks.svg)

<!-- SPEAKER NOTES
Hooks giúp ngăn chặn những lỗi mà các hướng dẫn bằng văn bản thường bất lực — chẳng hạn như việc agent tự ý sửa vào các path được bảo vệ hoặc commit file chưa format.
Hook là cơ chế thay đổi duy nhất trong repo không phụ thuộc vào việc model có đọc hiểu hay có chịu tuân thủ hay không. Chúng ta không thể giao phó ranh giới an toàn cho tính tự giác của model.
Chi phí context của hook rất thấp: Hook chạy hoàn toàn bên ngoài model, nhưng khi một hành động bị chặn lại, thông báo phản hồi sẽ được gửi ngược lại cho model để nó biết đường xử lý.
Một anti-pattern rất hay gặp cần tránh: viết hook chặn quá đà mọi thứ, hoặc hook bị lỗi âm thầm (fail silently) mà không báo gì cho agent.
-->

------

<!--slide-attr x=3600 y=2600 rotate=-2 scale=1.0 -->

# MCP

- Dựng server cung cấp những dữ liệu mà checkout không có: issues, CI logs, database
- Tốn context nhất trong cả 6 hạng mục: cả định nghĩa tool lẫn output trả về đều đổ vào session
- Luôn ưu tiên dùng script nội bộ (local script) nếu script tự giải quyết được
- Kiểm tra thực tế xem tool có thực sự được agent gọi hay không trước khi quyết định duy trì server

![Ảnh chụp: Một lượt gọi tool qua MCP và kết quả trả về ngay trong session.](images/placeholder-mcp.svg)

<!-- SPEAKER NOTES
MCP giúp agent bớt phải đoán mò về những thông tin nghiệp vụ nằm ngoài phạm vi repository.
Tuy nhiên, nếu câu trả lời không nhất thiết phải nằm ngoài checkout, việc dựng một MCP server chỉ làm phát sinh thêm chi phí token mà không đem lại giá trị gì.
-->

------

<!--slide-attr x=5400 y=2600 rotate=1 scale=1.0 -->

# Orchestration

- Bộ công cụ giúp lập trình viên quản lý và điều phối nhiều agent cùng lúc
- Không làm thay đổi repository: toàn bộ code trong checkout vẫn giữ nguyên
- Chi phí đánh đổi ở đây là sự tập trung của con người, chứ không phải context của model
- Tận dụng Git worktree: mỗi task chạy song song một checkout riêng, tách biệt port và database
- Chỉ mở rộng ra nhiều session khi quy trình chạy đơn lẻ với một session đã vượt qua lệnh `verify` ổn định

![Ảnh chụp: Hai session chạy song song kèm danh sách file thay đổi (giao diện Agents trong VS Code).](images/placeholder-orchestration.svg)

<!-- SPEAKER NOTES
Orchestration giúp chúng ta tránh rơi vào tình trạng "review mù" khi có quá nhiều session chạy cùng lúc. Nó chỉ thật sự đáng đầu tư khi điểm nghẽn của bạn nằm ở khâu review code chứ không phải khâu sinh code.
Nó đóng vai trò khuếch đại: repo của bạn đang có ưu điểm hay nhược điểm gì, nó sẽ nhân bản lên bấy nhiêu lần. Cửa sổ Agents trong hình hiện đang ở bản Preview của VS Code, mang tính giới thiệu để các bạn tham khảo thử sau này. Các harness khác trên thị trường cũng có cách làm tương tự với những tên gọi khác nhau.
Khi thiết lập custom agent và subagents, chúng ta thường chia theo các vai trò chuyên biệt: lên kế hoạch (planner), viết code (implementer), review (reviewer), hay nghiên cứu (researcher). Đặc biệt, các agent làm nhiệm vụ review và research chỉ nên được cấp tools chỉ đọc (read-only tools).
Hai lưu ý quan trọng: các session chạy trên worktree thường được cấp quyền thao tác khá rộng, vì vậy sandbox và hook bảo vệ lại càng trở nên cần thiết. Nếu nhiều session cùng lúc báo "đã xong" nhưng test vẫn fail hàng loạt, thì vấn đề nằm ở repo của bạn, chứ không phải do model kém.
-->

------

<!--slide-attr x=7200 y=3900 rotate=2 scale=1.0 -->

# Những sai lầm phổ biến (Anti-patterns)

- Chưa biết mình đau ở đâu đã vội cài đặt đủ mọi công cụ
- Dùng công cụ tự động sinh file `AGENTS.md` rồi bỏ mặc, không bao giờ đọc lại
- File hướng dẫn viết quá dài, lặp lại những tài liệu vốn đã có sẵn
- Viết hooks chặn quá đà mọi thứ, hoặc hooks lỗi âm thầm mà không báo lý do
- Cài thêm MCP server chỉ vì linh cảm "biết đâu sau này sẽ cần"

<!-- SPEAKER NOTES
Năm ý trên slide là những sai lầm phổ biến nhất khi áp dụng trên thực tế; các trường hợp khác được tổng hợp trong tài liệu phát kèm slide: chỉ chăm chăm nhìn vào số lượng token mà không kiểm tra độ đúng của code, cài tools nhưng không bao giờ đo xem agent có thực sự gọi hay không, chạy song song nhiều agent mà không cô lập port và dependencies, vội vã triển khai orchestration khi trường hợp một agent đơn lẻ còn chưa chạy ổn định, dựng dashboard theo dõi nhưng không có thói quen hành động theo số liệu, hay ngộ nhận rằng cứ có file config là tính năng đó tự khắc hoạt động trơn tru.
-->

------

<!--slide-attr x=0 y=5200 rotate=-1 scale=1.0 -->

# DevEx: Từ lập trình viên đến coding agent

| Cải thiện | Ý nghĩa cho con người | Ý nghĩa cho Coding Agent |
|---|---|---|
| Tài liệu & Onboarding rõ ràng | Bắt nhịp dự án nhanh, đỡ phải hỏi đồng nghiệp | Biến kiến thức ngầm thành runtime input (`AGENTS.md`, `skills`) |
| Chuẩn hóa script thao tác | Giảm việc thủ công, hạn chế gõ sai lệnh | Cung cấp tập lệnh tất định, chạy non-interactive (`scripts`, `MCP`) |
| Rút ngắn feedback loop | Test local và CI chạy nhanh, giữ nhịp flow | Agent biết ngay đúng hay sai, fail-fast sau mỗi lần sửa |
| Giảm tải nhận thức | Tập trung vào logic nghiệp vụ, đỡ phải nhớ vặt | Giữ context gọn gàng, tránh context rot và hạn chế đoán mò |
| Rào chắn an toàn (Guardrails) | Linter và quy ước nhóm ngăn commit nhầm code | Hooks chặn thẳng tay các hành vi sai phạm, không phụ thuộc model |
| Môi trường cô lập | Làm việc đa nhiệm không sợ đụng độ port hay DB | Cho phép nhiều session chạy song song an toàn qua Git worktree |

<!-- SPEAKER NOTES
DevEx (Developer Experience) không chỉ có ý nghĩa với con người. Mọi nỗ lực chúng ta đầu tư để repo dễ thở hơn cho lập trình viên — viết tài liệu rõ ràng, chuẩn hóa script, đẩy nhanh tốc độ test, giảm tải nhận thức, dựng guardrails bảo vệ và cô lập môi trường — thực chất đều ánh xạ trực tiếp sang việc dọn đường cho coding agent.
Điểm khác biệt cốt lõi: Con người có khả năng tích lũy kiến thức ngầm (tacit knowledge) qua thời gian, còn agent thì hoàn toàn "mù" bối cảnh ở từng session mới. Vì vậy, một repo có DevEx tốt cho con người chính là tiền đề vững chắc nhất để coding agent phát huy tối đa năng lực.
-->

------

<!--slide-attr x=1800 y=5200 rotate=2 scale=1.0 -->

# Take Aways

- Agent vốn dĩ đã tự chạy được; repository sẽ quyết định nó làm việc hiệu quả đến đâu
- Tinh chỉnh từng bước một, chỉ can thiệp khi gặp một "biểu hiện lạ" trong thực tế
- Sau khi tối ưu môi trường chạy, hãy theo dõi tiếp và tối ưu tiếp cho những lần chạy sau.

> Lập trình viên chỉ cần học cách thiết lập một lần.
> Còn agent phải trả giá bằng token và thời gian ở từng session.

<!-- SPEAKER NOTES
Tóm lại bằng luận điểm cốt lõi và giới hạn của nó: khi model và độ khó của task được giữ nguyên, việc thay đổi repo sẽ thay đổi trực tiếp cách agent làm việc. Điều này hoàn toàn dựa trên cơ chế vận hành thực tế. Và chính những session trong công việc hàng ngày của các bạn sẽ là nơi kiểm chứng rõ nhất.
Nếu sau khi bạn thay đổi repo mà các session không hề bớt nghẽn hay hiệu quả hơn, điều đó có nghĩa là giải pháp đó chưa thực sự phù hợp với repo của bạn.
-->

------

<!--slide-attr x=3600 y=5200 rotate=-2 scale=1.0 -->

# Tài liệu tham khảo

- Lulla et al., [On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents](https://arxiv.org/html/2601.20404v2)
- Gloaguen et al., [Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988)
- [How Do AI Agents Spend Your Money?](https://arxiv.org/html/2604.22750v3)
- METR, [Measuring the Impact of Early-2025 AI on Experienced Developer Productivity](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study)
- Google Cloud, [Báo cáo DORA 2024](https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report)
- Spotify, [Coding Is No Longer the Constraint](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint)
- Modal, [Agents need good developer experience too](https://modal.com/blog/agents-devex)
- incident.io, [DevEx matters for coding agents, too](https://incident.io/blog/ai-developer-tools)

<!-- SPEAKER NOTES
Các bài báo khoa học và bài phân tích kỹ thuật tiêu biểu từ các nhóm kỹ sư tại Spotify, Modal, incident.io và các tổ chức nghiên cứu AI. Mọi người có thể nhấp trực tiếp vào từng liên kết trên slide để đọc bài viết gốc.
-->

------

<!--slide-attr x=5400 y=5200 rotate=1 scale=1.1 -->

# Cảm ơn mọi người đã theo dõi!

| GitHub | Website |
|---|---|
| ![GitHub QR](images/github-qr.svg) | ![Website QR](images/nvduc-qr.svg) |
| ![GitHub icon](images/github-icon.svg) | ![Website icon](images/website-icon.svg) |

<!-- SPEAKER NOTES
Mười phút dành cho phần hỏi đáp và thảo luận. Một số chủ đề mọi người thường quan tâm: làm thế nào để đo lường hiệu quả cụ thể, cách áp dụng cho các harness khác ngoài Copilot, hay khi nào thì việc đầu tư dựng MCP server mới thực sự xứng đáng với chi phí context của nó.
-->
