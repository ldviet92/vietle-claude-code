# Project rules (dayladau backend)

## WORKFLOW — follow this order for every feature / code change

Do these steps in order, every time, without being asked:

1. **Discuss & confirm first.** Restate the goal, show the plan / folder structure, and
   wait for explicit approval. Do **not** write or edit code until the user says go.
   Use the `brainstorming` skill to explore, then `grill-with-docs` to pressure-test the plan.
2. **TDD.** Use the `tdd` skill — write failing tests first, then the minimum code to pass,
   then refactor (red → green → refactor). Tests before implementation.
3. **Implement to the `coding-standards` skill** — typed structs (not `gjson` string-path
   digging), named constants (no magic strings/numbers), small single-purpose functions,
   type safety, AAA-pattern tests including error paths.
4. **Performance check — only for code that touches the database or a hot path.**
   For every new or changed repository method:
   - run `EXPLAIN` and write the three answers as a comment on the method:
     *which index / max rows scanned / how many queries per request*;
   - if it replaces an existing function, run old vs new with
     `go test -bench=<Name> -count=6` and `benchstat`, and put the result in the commit
     message. Slower than before with no stated reason → do not proceed to review.
   Benchmarks that need MySQL read `BENCH_DSN` and `b.Skip()` when it is unset.
   Skip this step for handler/validation-only changes.
5. **Code-review.** Run the `code-review` skill after the feature works; address findings.
6. **Ask before `git commit`.**

## GIT — branching

- **Commit thẳng vào nhánh `dev`.** Mặc định mọi thay đổi được commit trực tiếp lên `dev`.
  KHÔNG tự tạo nhánh mới, KHÔNG tự merge — việc tạo nhánh làm rối history.
- **Chỉ tạo nhánh mới khi developer yêu cầu rõ ràng** (ví dụ: "tạo branch cho việc này").
  Nếu đang không ở `dev`, hỏi lại trước khi commit thay vì tự chuyển/tạo nhánh.
- **TUYỆT ĐỐI KHÔNG merge hay push vào `master`.** `master` là nhánh production, chỉ
  trưởng phòng được merge để phát hành cho khách hàng. Developer ở đây KHÔNG có quyền
  merge vào master. Mọi thay đổi dừng ở `dev` — mặc định `git push origin HEAD:dev`.
  Không `git merge` khi đang đứng trên `master`, không `git push origin ...:master`,
  không mở PR nhắm vào `master`. Việc nào có vẻ sẽ chạm tới master thì hỏi lại trước.
  Lưu ý `master` và `dev` có lúc trùng commit nên rất dễ nhầm — kiểm tra bằng
  `git rev-parse origin/dev` chứ đừng đoán.

## ALWAYS

- **CRM host** — làm màn / API CRM host thì đọc `docs/crm-ban-chat.md` trước (đích, ba đội, luật
  đã chốt kèm lý do); chốt luật mới thì cập nhật vào đó.
- **API conventions** — every new or modified HTTP endpoint follows `docs/api-conventions.md`
  (list response shape, paging, error status codes, identity from token). Read it before
  writing a handler; do not restate its rules here.
- **Layered structure** — organize each feature into its own folder with
  `handler` → `service` → `repository` layers (e.g. `search/handler.go`,
  `search/service.go`, `search/repository.go`).
- **One owner per table.** Every table has exactly one package that may write SQL
  (or gorm) against it — `order`, `order_item`, `bill` belong to `order/`; other code
  calls that package's repository methods instead of querying the table. Inside the
  owner package, split files by the *question asked* (`repository_get.go`,
  `repository_search.go`, `repository_payout.go`), never by table. Reports that
  aggregate across tables read a pre-built read model (e.g. `listing_metric_daily`)
  rather than joining owned tables. Methods must not expose `*gorm.DB` or raw filter
  structs outside the package.
- **Comments in Vietnamese**; identifiers and commit messages in English.
