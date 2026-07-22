---
name: host-hub-design
description: Build Dayladau host pages in the Airbnb host-hub style — big greeting, horizontal stat strip, pill-tab reservation buckets, request-row attention list, sub-titled sections. Brand-teal as the action color, neutral black/gray everywhere else. Use when creating or restyling any host-facing dashboard, list, or detail page in `host/page/*.js`.
---

# Host-hub design system

This skill codifies the visual language used on the `/today` page (`page/today.js` + `page/today.css`). Apply it whenever you build or restyle a host-facing page.

## When to use

- New host-facing page or screen (dashboard, reservation list, request inbox, performance summary)
- Restyling an existing host page to match the new Today look
- Adding a new section to `today.js` or any sibling page that should feel consistent

Do **not** apply this to guest-facing pages, marketing pages, or printable reports — those have their own conventions.

## Reference files

- `design/dayladau-design-system.css` — base CSS variables (brand colors, spacing, radii)
- `design/airbnb-design-guidelines.md` — high-level principles (clarity, data-driven, mobile-first)
- `page/today.js` + `page/today.css` — canonical implementation of this skill
- `design/home-redesign-option-b.html` — static mockup of the look-and-feel

## Design philosophy

1. **Action-first, not report-first.** Top of the page should show what the host can *do* today. Charts and analytics belong on `/reports`, not the entry page.
2. **Brand teal `rgb(0, 132, 137)` is the action color.** Primary buttons, active tabs, the user avatar, links, the CTA stat tile — all teal. Neutral black/gray drives the rest of the UI. Never use gradients.
3. **Specific over generic.** Card titles should say what the row *is* (e.g. "Gia hạn thời gian · Lê Hoàng"), not what category it belongs to. Subtitles explain the section's purpose in one short Vietnamese sentence.
4. **No emoji in product UI.** Use inline stroke SVG icons (1.8px stroke). Emoji is only acceptable in light-weight greeting/onboarding copy where the tone allows.
5. **Whitespace is structural.** Pages use 80px horizontal page padding on desktop, 24px on mobile, 32–48px vertical gaps between major sections.

## Layout skeleton

Every host-hub page follows this top-to-bottom rhythm. Skip sections that don't apply but keep the order:

```
[Header]                       // shared <Header />
[Hero]                         // 32px headline + 16px gray sub-line
[Stat strip]                   // 4 horizontal tiles, last one is teal CTA
[Pill tabs]                    // status buckets, scrollable
[Card grid]                    // 2-col grid of cards, full-width on mobile
[Section: action queue]        // full-width, one row per actionable item
[Footer]                       // shared <Footer />
```

The body wrapper is `<div className='{page}-root'>` so CSS variables scope to the page only.

## Tokens (define inside the root)

Every page namespaces its own CSS variables. Copy this block into `page/{name}.css` and rename the prefix:

```css
.{name}-root {
  --{name}-primary: rgb(0, 132, 137);
  --{name}-primary-dark: rgb(0, 112, 117);
  --{name}-primary-soft: rgba(0, 132, 137, 0.08);
  --{name}-black: #222222;
  --{name}-dark: #484848;
  --{name}-medium: #717171;
  --{name}-light: #B0B0B0;
  --{name}-line: #DDDDDD;
  --{name}-line-soft: #EBEBEB;
  --{name}-bg: #FFFFFF;
  --{name}-bg-soft: #F7F7F7;
  --{name}-danger: #C13515;
  --{name}-success: #008A05;
  --{name}-warning: #E07700;
}
```

Why namespace? CSS files in `page/*.css` are globally scoped at runtime. A bare `.hero` or `.btn` will collide with another page's class. Always prefix with the page name (e.g. `today-*`, `inbox-*`, `calendar-*`).

## Component primitives

### Hero

```jsx
<section className='{name}-hero'>
  <h1>{greeting}, {hostName}</h1>
  <div className='{name}-hero-sub'>{date} — {one concrete fact about today}</div>
</section>
```

```css
.{name}-hero h1 { font-size: 32px; font-weight: 600; letter-spacing: -0.6px; }
.{name}-hero-sub { font-size: 16px; color: var(--{name}-medium); }
```

The sub-line must contain at least one *fact* derived from today's data, not a generic tagline. Examples: `bạn có 5 khách đang lưu trú`, `3 khách sắp đến hôm nay`. If there's no relevant fact, show only the date.

### Stat strip

4 tiles in a horizontal grid. Last tile is a teal CTA tile linking to the related deep-dive (reports, full list).

```jsx
<div className='{name}-stats'>
  <div className='{name}-stat'>
    <div className='{name}-stat-label'>Đặt phòng tháng này</div>
    <div className='{name}-stat-value'>{count}</div>
  </div>
  {/* …more tiles… */}
  <div className='{name}-stat {name}-stat--cta' onClick={() => route('/reports')}>
    <div className='{name}-stat-label'>Báo cáo chi tiết</div>
    <div className='{name}-stat-value'>Xem báo cáo →</div>
  </div>
</div>
```

```css
.{name}-stats { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; margin-bottom: 40px; }
.{name}-stat { background: var(--{name}-bg); border: 1px solid var(--{name}-line-soft);
  border-radius: 16px; padding: 18px 20px; min-height: 100px;
  display: flex; flex-direction: column; gap: 4px; justify-content: center; }
.{name}-stat-label { font-size: 13px; color: var(--{name}-medium); font-weight: 500; }
.{name}-stat-value { font-size: 24px; font-weight: 600; letter-spacing: -0.4px; }
.{name}-stat--cta { background: var(--{name}-primary); border-color: var(--{name}-primary);
  color: white; cursor: pointer; transition: background 0.15s, transform 0.15s; }
.{name}-stat--cta:hover { background: var(--{name}-primary-dark); transform: translateY(-1px); }
.{name}-stat--cta .{name}-stat-label { color: rgba(255,255,255,0.85); }
.{name}-stat--cta .{name}-stat-value { color: white; font-size: 16px; }
@media (max-width: 900px) { .{name}-stats { grid-template-columns: repeat(2, 1fr); } }
```

**3 tiles also OK** if there's no natural CTA — just drop the CTA tile and adjust to `repeat(3, 1fr)`.

### Pill tab row

Pills are status filters. Active = solid teal. Show count after the label.

```jsx
<div className='{name}-tabs'>
  {TABS.map((t) => (
    <button key={t.key}
      className={`{name}-tab ${activeTab === t.key ? 'active' : ''}`}
      onClick={() => setActiveTab(t.key)}>
      {t.label}<span className='count'> · {counts[t.key]}</span>
    </button>
  ))}
</div>
```

```css
.{name}-tabs { display: flex; gap: 8px; margin-bottom: 24px; overflow-x: auto; padding-bottom: 4px; }
.{name}-tab { background: var(--{name}-bg); border: 1px solid var(--{name}-line);
  border-radius: 28px; padding: 10px 18px; font-size: 14px; font-weight: 500;
  color: var(--{name}-black); white-space: nowrap; cursor: pointer; transition: all 0.15s; }
.{name}-tab:hover { border-color: var(--{name}-primary); }
.{name}-tab.active { background: var(--{name}-primary); color: white; border-color: var(--{name}-primary); }
.{name}-tab .count { margin-left: 4px; opacity: 0.7; font-weight: 400; }
.{name}-tab.active .count { opacity: 1; }
```

### Data card (reservation / listing / generic record)

```jsx
<article className='{name}-card' onClick={openDetail}>
  <div className='{name}-card-avatar'>{initials(name)}</div>
  <div className='{name}-card-body'>
    <div className={`{name}-card-status ${tone}`}><span className='dot'></span>{statusLabel}</div>
    <div className='{name}-card-title'>{primary}</div>
    <div className='{name}-card-sub'>{secondary}</div>
    <div className='{name}-card-meta'>
      <span>{detail1}</span><span className='sep'>·</span><span>{detail2}</span>
    </div>
    <div className='{name}-card-actions' onClick={(e) => e.stopPropagation()}>
      <button className='{name}-btn {name}-btn--primary'>Primary</button>
      <button className='{name}-btn {name}-btn--ghost'>Ghost</button>
    </div>
  </div>
</article>
```

Status tones: `in` (success green), `out` (danger red), `stay` (brand teal), `pending` (warning orange). The dot is a `width: 6px; height: 6px; border-radius: 50%;` colored by the tone.

```css
.{name}-card { background: var(--{name}-bg); border: 1px solid var(--{name}-line-soft);
  border-radius: 16px; padding: 20px; display: flex; gap: 16px;
  transition: box-shadow 0.18s, transform 0.18s; cursor: pointer; }
.{name}-card:hover { box-shadow: 0 6px 20px rgba(0,0,0,0.08); transform: translateY(-1px); }
.{name}-card-avatar { width: 56px; height: 56px; border-radius: 50%;
  background: var(--{name}-bg-soft); color: var(--{name}-dark);
  display: flex; align-items: center; justify-content: center;
  font-weight: 600; font-size: 20px; flex-shrink: 0; }
.{name}-card-status { display: inline-flex; align-items: center; gap: 6px;
  font-size: 12px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.5px;
  color: var(--{name}-medium); margin-bottom: 6px; }
.{name}-card-status .dot { width: 6px; height: 6px; border-radius: 50%; }
.{name}-card-status.in .dot { background: var(--{name}-success); }
.{name}-card-status.out .dot { background: var(--{name}-danger); }
.{name}-card-status.stay .dot { background: var(--{name}-primary); }
.{name}-card-status.pending .dot { background: var(--{name}-warning); }
```

### Attention card (action queue row)

For pending requests, modifications, alerts. Single row, icon-left, body-middle, action-buttons-right.

```jsx
<div className='{name}-attn-card' onClick={openDetail}>
  <div className='{name}-attn-icon'><IconAlert /></div>
  <div className='{name}-attn-body'>
    <div className='{name}-attn-title'>{specific title with type + entity}</div>
    <div className='{name}-attn-sub'>{listing} · {code} · {diff} · {timeAgo}</div>
  </div>
  <div className='{name}-attn-actions' onClick={(e) => e.stopPropagation()}>
    <button className='{name}-btn {name}-btn--primary' disabled={busy}>Duyệt</button>
    <button className='{name}-btn {name}-btn--ghost' disabled={busy}>Từ chối</button>
  </div>
</div>
```

The title should not be a category alone (avoid "3 yêu cầu từ khách"). It should name the entity ("Gia hạn thời gian · Lê Hoàng"). One row per actionable item, not summaries.

```css
.{name}-attn-card { background: var(--{name}-bg); border: 1px solid var(--{name}-line-soft);
  border-radius: 16px; padding: 20px 22px; margin-bottom: 12px;
  display: flex; align-items: center; gap: 18px;
  cursor: pointer; transition: box-shadow 0.18s; }
.{name}-attn-card:hover { box-shadow: 0 6px 20px rgba(0,0,0,0.08); }
.{name}-attn-icon { width: 44px; height: 44px; border-radius: 50%;
  background: var(--{name}-bg-soft); color: var(--{name}-black);
  display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.{name}-attn-icon svg { width: 22px; height: 22px; stroke-width: 1.8; }
.{name}-attn-body { flex: 1; min-width: 0; }
.{name}-attn-title { font-size: 15px; font-weight: 600; color: var(--{name}-black); margin-bottom: 2px; }
.{name}-attn-sub { font-size: 14px; color: var(--{name}-medium); }
.{name}-attn-actions { display: flex; gap: 8px; flex-shrink: 0; }
.{name}-attn-actions .{name}-btn { padding: 7px 14px; font-size: 12px; }
.{name}-attn-actions .{name}-btn:disabled { opacity: 0.55; cursor: default; }
@media (max-width: 720px) {
  .{name}-attn-card { flex-wrap: wrap; }
  .{name}-attn-actions { width: 100%; margin-top: 4px; }
}
```

### Section header with subtitle

Every major section has a title + one-line subtitle that explains what the section contains. No section may use a bare title — the subtitle removes ambiguity.

```jsx
<div className='{name}-section-head'>
  <div>
    <h2 className='{name}-section-title'>Cần xử lý</h2>
    <p className='{name}-section-sub'>Yêu cầu thay đổi từ khách đang chờ bạn duyệt</p>
  </div>
  <a className='{name}-section-link' onClick={...}>Tất cả</a>
</div>
```

```css
.{name}-section-head { display: flex; align-items: baseline; justify-content: space-between; margin-bottom: 20px; }
.{name}-section-title { font-size: 22px; font-weight: 600; letter-spacing: -0.3px; margin: 0; }
.{name}-section-sub { font-size: 14px; color: var(--{name}-medium); margin: 4px 0 0; }
.{name}-section-link { color: var(--{name}-primary); font-weight: 600; font-size: 14px;
  text-decoration: underline; cursor: pointer; }
```

### Buttons

Three variants. All bo tròn full pill (24px radius).

```css
.{name}-btn { background: var(--{name}-bg); border: 1px solid var(--{name}-primary);
  color: var(--{name}-primary); border-radius: 24px; padding: 8px 16px;
  font-size: 13px; font-weight: 600; font-family: inherit; cursor: pointer;
  transition: background 0.15s; }
.{name}-btn:hover { background: var(--{name}-primary-soft); }
.{name}-btn--primary { background: var(--{name}-primary); color: white; }
.{name}-btn--primary:hover { background: var(--{name}-primary-dark); border-color: var(--{name}-primary-dark); }
.{name}-btn--ghost { border-color: var(--{name}-line); color: var(--{name}-dark); }
.{name}-btn--ghost:hover { background: var(--{name}-bg-soft); border-color: var(--{name}-medium); }
```

### Empty / loading states

```css
.{name}-empty { background: var(--{name}-bg); border: 1px dashed var(--{name}-line);
  border-radius: 16px; padding: 40px 24px; text-align: center;
  color: var(--{name}-medium); font-size: 14px; }
.{name}-skeleton { background: var(--{name}-bg-soft); border-radius: 16px;
  height: 132px; animation: {name}-pulse 1.4s ease-in-out infinite; }
@keyframes {name}-pulse { 0%, 100% { opacity: 0.6; } 50% { opacity: 1; } }
```

Empty copy should be concrete and reassuring, not marketing. Good: "Không có yêu cầu nào — bạn đã xử lý hết". Bad: "Bạn chưa có dữ liệu".

## Icons

Inline SVG only. Stroke-based, 1.8px line-weight, 22–24px box. Reuse this set:

```jsx
const IconAlert = () => (
  <svg viewBox='0 0 24 24' fill='none' stroke='currentColor' strokeLinecap='round' strokeLinejoin='round'>
    <path d='M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z'/>
    <line x1='12' y1='9' x2='12' y2='13'/><line x1='12' y1='17' x2='12.01' y2='17'/>
  </svg>
)
const IconChat = () => (
  <svg viewBox='0 0 24 24' fill='none' stroke='currentColor' strokeLinecap='round' strokeLinejoin='round'>
    <path d='M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z'/>
  </svg>
)
const IconCalendar = () => (
  <svg viewBox='0 0 24 24' fill='none' stroke='currentColor' strokeLinecap='round' strokeLinejoin='round'>
    <rect x='3' y='4' width='18' height='18' rx='2' ry='2'/>
    <line x1='16' y1='2' x2='16' y2='6'/><line x1='8' y1='2' x2='8' y2='6'/><line x1='3' y1='10' x2='21' y2='10'/>
  </svg>
)
```

## Page wrapper

```jsx
return (
  <div className='{name}-root'>
    <Header />
    <main className='{name}-page'>
      {/* sections */}
    </main>
    <Footer />
  </div>
)
```

```css
.{name}-root { background: var(--{name}-bg); color: var(--{name}-black);
  font-size: 15px; line-height: 1.5; -webkit-font-smoothing: antialiased; min-height: 100vh; }
.{name}-page { max-width: 1280px; margin: 0 auto; padding: 48px 80px 80px; }
@media (max-width: 900px) { .{name}-page { padding: 24px; } }
```

## Vietnamese copy conventions

- Greetings: time-of-day (Chào buổi sáng / trưa / chiều / tối)
- Status labels: short, no period. "Sắp đến · Mai 14:00", "Đang lưu trú · trả phòng 16/05 11:00"
- Action verbs: imperative + specific. "Yêu cầu dọn phòng", "Gửi mã cửa", "Duyệt", "Từ chối"
- Counts read naturally: "5 khách đang lưu trú", "3 yêu cầu thay đổi từ khách"
- Subtitles describe the section in plain Vietnamese; don't use "Quản lý X" — say what's *in* the section

## Anti-patterns (do not do these)

- **No background gradients.** Solid colors only. The CTA stat tile is solid teal, not a gradient.
- **No emoji in product UI.** Stroke SVG instead. (Greetings allow one emoji like 👋 if it adds warmth.)
- **No raw enum values shown to users.** Translate `extend_time` → "Gia hạn thời gian" via a label map.
- **No summary cards where individual rows belong.** "3 yêu cầu" is a count, not an actionable item. Show 3 rows.
- **No bare section titles.** Always include the subtitle.
- **No shadows on resting cards.** Only on hover.
- **No two-column layouts with one filler column.** If the rail only has one item, promote it to a stat strip or full-width section.
- **No marketing copy in empty/recommendation states.** Be specific and concrete.

## Implementation checklist

When applying this skill to a new page:

- [ ] CSS file `page/{name}.css` exists, namespaced `.{name}-*` throughout
- [ ] Root wrapper `.{name}-root` declares the local CSS variables
- [ ] Hero section uses `<Header/>` from `page/header.js` (do not reimplement)
- [ ] Stat strip has either 3 plain tiles or 3 + 1 CTA tile
- [ ] Every section has a `{name}-section-sub` subtitle
- [ ] Status tones used: `in`, `out`, `stay`, `pending` (no other colors)
- [ ] Buttons use `{name}-btn` + variant modifier (`--primary` / `--ghost`)
- [ ] Icons are inline stroke SVG, not emoji or Material Symbols
- [ ] Empty state copy is concrete ("không có X — bạn đã xử lý hết"), not generic
- [ ] Mobile: stat strip collapses to 2 columns, card grid to 1 column, action buttons wrap full-width

## Reference implementation

`page/today.js` (the file) is the canonical reference. When in doubt about a pattern, read it. When extending the system, update both the page and this skill so they stay in sync.
