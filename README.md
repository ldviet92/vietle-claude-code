# vietle-claude-code

Bộ **skills Claude Code** dùng chung cho team Dayladau, tách theo mảng: **mobile · web · backend · shared**. Cài một lần, cả team xài chung các workflow đã đúc kết: build screen, TDD, diagnose bug, planning…

Đây là một **Claude Code plugin marketplace** — cài qua `/plugin`, cập nhật qua `/plugin update`.

## Cài đặt

Trong Claude Code, thêm marketplace một lần:

```
/plugin marketplace add ldviet92/vietle-claude-code
```

Rồi cài đúng plugin theo mảng bạn làm (cài được nhiều cái):

```
/plugin install vietle-mobile@vietle-claude-code    # dev mobile (React Native)
/plugin install vietle-web@vietle-claude-code       # dev web (Next.js)
/plugin install vietle-backend@vietle-claude-code   # dev backend (Go)
/plugin install vietle-shared@vietle-claude-code    # ai cũng nên cài
```

Cập nhật khi repo có thay đổi:

```
/plugin marketplace update vietle-claude-code
/plugin update vietle-mobile   # (hoặc plugin bạn đã cài)
```

> Skill **tự kích hoạt** khi task khớp mô tả của nó — không cần nhớ gọi tay. Vẫn gọi tay được bằng `/<tên-skill>`.

## Các plugin & skill

### `vietle-mobile` — React Native (app-host, app-guest)
| Skill | Dùng khi |
|---|---|
| `new-screen` | Build một màn hình RN mới từ đầu — align, test logic-layer, đúng convention |
| `feature-flow` | Điều phối feature nhiều màn hình (playbook 5 bước, xuất artifact HTML) |
| `clone-screen` | Port một màn hình từ codebase này sang codebase khác |
| `mobile-ux` | Nguyên tắc UX mobile — touch target, bottom sheet, flow từng bước… |

### `vietle-web` — Next.js / host web
| Skill | Dùng khi |
|---|---|
| `airbnb-ui-skills` | Dựng UI theo design system kiểu Airbnb (light mode, Inter, 4px grid) |
| `host-hub-design` | Style trang host theo Airbnb host-hub (greeting, stat strip, pill-tab…) |
| `frontend-patterns` | Pattern React/Next.js — state management, performance, UI |
| `frontend-design` | Dựng giao diện frontend chất lượng cao, tránh "AI aesthetic" |

### `vietle-backend` — Go backend
| Skill | Dùng khi |
|---|---|
| `build-feature` | Workflow build feature: discuss → TDD → layered handler/service/repo → review |

### `vietle-shared` — dùng chung mọi mảng
| Skill | Dùng khi |
|---|---|
| `tdd` | Làm feature / fix bug theo red-green-refactor |
| `diagnose` | Bug khó, regression hiệu năng — reproduce → minimise → fix → regression test |
| `grill-with-docs` | Stress-test kế hoạch với domain model, cập nhật CONTEXT.md / ADR |
| `zoom-out` | Cần bức tranh lớn / context cao hơn cho một vùng code lạ |

## Luật làm việc (CLAUDE.md)

Thư mục `rules/` chứa bộ luật mẫu cho Claude Code — không cài qua `/plugin`, chép tay vào đúng chỗ:

| File | Chép vào | Nội dung |
|---|---|---|
| `rules/global/CLAUDE.md` | `~/.claude/CLAUDE.md` | Luật chung: chỉ làm khi được bật đèn xanh, không đọc ra ngoài thư mục khi chưa hỏi, mở đầu bằng tóm tắt, hiểu bản chất trước khi làm |
| `rules/backend/CLAUDE.md` | `CLAUDE.md` ở gốc repo backend | Workflow backend: discuss → TDD → coding-standards → check hiệu năng → review; luật git `dev`/`master`; phân tầng, mỗi bảng một chủ |

## Thêm skill mới

1. Vào plugin đúng mảng, tạo `plugins/<mảng>/skills/<tên-skill>/SKILL.md`:

   ```markdown
   ---
   name: ten-skill
   description: Mô tả rõ KHI NÀO nên dùng — đây là thứ Claude đọc để tự kích hoạt.
   ---

   Nội dung hướng dẫn skill…
   ```

   > ⚠️ Skill phải nằm **phẳng** ngay dưới `skills/<tên>/` — **không** lồng thêm thư mục con (plugin sẽ không quét thấy). Có file phụ (scripts, tài liệu) thì để cùng thư mục đó.

2. Cần mảng mới? Tạo `plugins/<mảng>/.claude-plugin/plugin.json` + `plugins/<mảng>/skills/`, rồi khai báo thêm 1 entry trong `.claude-plugin/marketplace.json`.
3. Commit + push. Team chạy `/plugin update` là có.

## Cấu trúc

```
vietle-claude-code/
├── .claude-plugin/
│   └── marketplace.json          # khai báo 4 plugin (mobile/web/backend/shared)
├── plugins/
│   ├── mobile/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/  new-screen · feature-flow · clone-screen · mobile-ux
│   ├── web/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/  airbnb-ui-skills · host-hub-design · frontend-patterns · frontend-design
│   ├── backend/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/  build-feature
│   └── shared/
│       ├── .claude-plugin/plugin.json
│       └── skills/  tdd · diagnose · grill-with-docs · zoom-out
└── README.md
```

Về sau muốn thêm **agents** / **commands** dùng chung: thêm `agents/` / `commands/` vào plugin tương ứng — Claude Code tự nhận.
