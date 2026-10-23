# Xây dựng repository thân thiện với agent

[![en](https://img.shields.io/badge/lang-en-blue.svg)](README.md)

Buổi chia sẻ về những tinh chỉnh nhỏ trong repository giúp coding agent làm việc hiệu quả hơn: `AGENTS.md`, scripts, skills, hooks, MCP, isolation và orchestration. Trình bày tại **Codegym Tech Talk** bởi Nguyễn Văn Đức.

Slide: [vanduc2514.github.io/presentation-repository-agent-ready](https://vanduc2514.github.io/presentation-repository-agent-ready/) · Bản tiếng Việt: [index.vi.html](https://vanduc2514.github.io/presentation-repository-agent-ready/index.vi.html)

## Bối cảnh

Coding agent vốn dĩ đã làm việc được trên một repository bình thường. Nó đọc file, chạy lệnh, sửa code và báo cáo lại; với một thay đổi nhỏ ở local, nó thường thành công mà không cần thiết lập gì đặc biệt. Vấn đề không phải là nó không chạy được, mà là nó lãng phí bao nhiêu công sức để tự tìm hiểu lại những gì repository đã biết.

Mỗi câu hỏi không được trả lời sẽ trở thành một phép suy đoán: lệnh nào chạy test, tầng nào xử lý validation, những đường dẫn nào không được chạm vào. Suy đoán tốn token, tốn thời gian và làm giảm độ chính xác. Buổi chia sẻ gọi đây là những điểm ma sát và xem việc cấu hình repository là cách loại bỏ chúng, không phải cách làm model thông minh hơn.

Khán giả: sinh viên và lập trình viên mới đi làm, những người đã dùng coding agent và không ít lần thấy nó gặp lỗi khó hiểu. Hình thức: thuyết trình, không live demo, nên không cần cài đặt gì để theo dõi.

## Nội dung trình bày

| # | Chủ đề | Mô tả |
|---|--------|-------|
| 1 | Những "biểu hiện lạ" thường gặp khi sử dụng Agent | Code nhìn qua tưởng đúng nhưng lỗi khi deploy, sửa vào file generated, session cần can thiệp liên tục, chi phí token không tương xứng với thay đổi |
| 2 | Vì sao lại có những biểu hiện này? | Agent bắt đầu mỗi session mà không có hình dung về repository; câu hỏi không được trả lời sẽ thành phép đoán |
| 3 | Các kỹ thuật để tối ưu môi trường cho agent | Tổng quan bảy hạng mục và thời điểm nên áp dụng từng hạng mục |
| 4 | AGENTS.md | Một file ngắn do con người viết: lệnh chạy, quy ước, ranh giới cấm chạm và tiêu chuẩn hoàn thành |
| 5 | Scripts | Mỗi mục đích một lệnh, không tương tác, exit code ổn định, output có giới hạn |
| 6 | Skills | Quy trình lặp lại được đóng gói với cơ chế nạp lũy tiến, chỉ đọc phần chi tiết khi thật sự cần |
| 7 | Hooks | Ép buộc quy tắc theo sự kiện của harness, không phụ thuộc vào việc model có đọc hay không |
| 8 | MCP | Đưa vào những dữ liệu nằm ngoài checkout, với chi phí context cao nhất trong bảy hạng mục |
| 9 | Isolation | Git worktree cho mỗi task song song, kết hợp PR stack để giữ review ở quy mô nhỏ |
| 10 | Orchestration | Điều phối nhiều agent cùng lúc; chỉ đáng đầu tư khi nút thắt nằm ở review chứ không phải sinh code |
| 11 | Những sai lầm phổ biến (Anti-patterns) | Cài đặt mọi thứ trước khi biết ma sát ở đâu, tự sinh context file, viết hướng dẫn quá dài, hook chặn quá mức |
| 12 | DevEx: Từ lập trình viên đến coding agent | Mọi cải thiện giúp con người làm việc tốt hơn đều ánh xạ sang môi trường agent cần |
| 13 | Take Aways | Nên bắt đầu từ đâu và làm sao biết nó có tác dụng |

## Render

```sh
npm install
npm run build      # render cả hai ngôn ngữ vào output/
npm run preview    # mở output/index.html trên trình duyệt
```

Yêu cầu Node.js 20 (được pin trong `mise.toml`). Chạy `mise install` trước nếu bạn dùng mise.

| File | Ngôn ngữ |
|---|---|
| `output/index.html` | Tiếng Anh (mặc định) |
| `output/index.vi.html` | Tiếng Việt |

Cả hai file đều có nút chuyển ngôn ngữ nổi, dẫn tới phiên bản còn lại.

## Điều khiển từ điện thoại

Bản trình bày có kèm remote control. Mở `output/remote.html` trên điện thoại để điều khiển việc chuyển slide và đọc speaker notes theo thời gian thực qua relay ntfy.sh.

## Cấu trúc dự án

```
slides/
  presentation.en.md  # nội dung slide tiếng Anh
  presentation.vi.md  # nội dung slide tiếng Việt
  images/             # ảnh chụp và mã QR dùng trong slide
build.cjs             # script build và xử lý hậu kỳ slide
public/
  presentation/       # presentation.css và presentation.js
  remote/             # remote control trên điện thoại
.github/workflows/
  deploy-pages.yml    # deploy lên GitHub Pages mỗi khi push vào main
.agents/skills/       # agent skills cho nội dung, style và release notes
```

Phần giao diện nằm trong `public/presentation/presentation.css`; màu nhấn riêng cho từng slide và hiệu ứng chuyển động vào slide là các rule CSS tại đó. Xem [`.agents/skills/markpress-styling/SKILL.md`](.agents/skills/markpress-styling/SKILL.md).
