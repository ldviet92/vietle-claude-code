# vietle-claude-code

Bộ **skills Claude Code** dùng chung cho team Dayladau. Cài một lần, cả team xài chung các workflow đã được đúc kết: build screen, TDD, diagnose bug, planning…

Đây là một **Claude Code plugin marketplace** — cài qua `/plugin`, cập nhật qua `/plugin update`.

## Cài đặt

Trong Claude Code, chạy:

```
/plugin marketplace add ldviet92/vietle-claude-code
/plugin install vietle-claude-code@vietle-claude-code
```

Xong. Các skill sẽ **tự kích hoạt** khi task khớp mô tả của nó — bạn không cần nhớ gọi tay. (Vẫn gọi tay được bằng `/<tên-skill>` nếu muốn.)

Cập nhật khi repo có thay đổi:

```
/plugin marketplace update vietle-claude-code
/plugin update vietle-claude-code
```

## Các skill

> Skill chỉ bung ra khi Claude thấy task khớp mô tả — nên cài cả bộ **không gây nhiễu**. Dev web cài cả bộ thì skill mobile vẫn nằm im, chỉ chạy khi bạn thực sự làm mobile.

### Mobile — React Native (app-host, app-guest)

| Skill | Dùng khi |
|---|---|
| `new-screen` | Build một màn hình RN mới từ đầu — align, test logic-layer, đúng convention |
| `feature-flow` | Điều phối feature nhiều màn hình (playbook 5 bước, xuất artifact HTML) |
| `clone-screen` | Port một màn hình từ codebase này sang codebase khác |
| `mobile-ux` | Nguyên tắc UX mobile — touch target, bottom sheet, flow từng bước… |

### Chung — mọi mảng (web / mobile / backend)

| Skill | Dùng khi |
|---|---|
| `tdd` | Làm feature / fix bug theo red-green-refactor |
| `diagnose` | Bug khó, regression hiệu năng — reproduce → minimise → fix → regression test |
| `grill-with-docs` | Stress-test kế hoạch với domain model, cập nhật CONTEXT.md / ADR |
| `zoom-out` | Cần bức tranh lớn / context cao hơn cho một vùng code lạ |

> **Web (Next.js)** và **Backend** chưa có skill riêng — thêm dần vào `skills/` khi có (xem bên dưới).

## Thêm skill mới

1. Tạo thư mục `skills/<tên-skill>/` với file `SKILL.md`:

   ```markdown
   ---
   name: ten-skill
   description: Mô tả rõ KHI NÀO nên dùng — đây là thứ Claude đọc để tự kích hoạt.
   ---

   Nội dung hướng dẫn skill…
   ```

   > ⚠️ Skill phải nằm **phẳng** ngay dưới `skills/<tên>/` — **không** lồng thư mục con kiểu `skills/web/<tên>/` (plugin sẽ không quét thấy). Phân nhóm mảng thì ghi vào README này thôi.

2. Có file phụ (scripts, tài liệu tham chiếu) thì để cùng thư mục đó.
3. Commit + push. Team chạy `/plugin update vietle-claude-code` là có.

## Cấu trúc

```
vietle-claude-code/
├── .claude-plugin/
│   ├── plugin.json         # khai báo plugin (trỏ ./skills)
│   └── marketplace.json    # để /plugin marketplace add
├── skills/
│   ├── new-screen/         SKILL.md
│   ├── feature-flow/       SKILL.md
│   ├── clone-screen/       SKILL.md
│   ├── mobile-ux/          SKILL.md
│   ├── tdd/                SKILL.md + tài liệu phụ
│   ├── diagnose/           SKILL.md + scripts/
│   ├── grill-with-docs/    SKILL.md + ADR/CONTEXT format
│   └── zoom-out/           SKILL.md
└── README.md
```

Sau này muốn thêm **agents** hoặc **commands** dùng chung, thêm thư mục `agents/` / `commands/` ở root và khai báo trong `plugin.json`.
