---
name: mobile-ux
description: Mobile UX principles for React Native apps — touch targets, progressive disclosure, step-by-step flows, bottom sheets, and Airbnb-style patterns.
origin: custom
---

# Mobile UX Principles

Practical UX rules for React Native apps, based on Airbnb DLS and iOS/Android Human Interface Guidelines.

## When to Activate

- Designing or reviewing any screen or component
- Building multi-step flows (booking, search, onboarding, checkout)
- Designing bottom sheets, modals, or filter panels
- Reviewing form UX (validation, error states, empty states)
- Deciding how much information to show on a screen

---

## Core Rules

### 1. Touch Targets — Minimum 44×44px

Every tappable element must be at least 44×44px. Use `hitSlop` if the visual is smaller.

```tsx
// PASS: GOOD
<TouchableOpacity hitSlop={{ top: 8, bottom: 8, left: 8, right: 8 }}>
  <Icon name="close" size={20} />
</TouchableOpacity>

// FAIL: BAD — icon-only button with no padding
<TouchableOpacity>
  <Icon name="close" size={16} />
</TouchableOpacity>
```

**Checklist:**
- Icon buttons: `width: 44, height: 44` minimum, `rounded-full`
- Chips/tags: `paddingVertical: 10` minimum
- List rows: `paddingVertical: 14` minimum
- Close/back buttons: always use `hitSlop`

---

### 2. One Primary Action Per Screen

Each screen or step should present exactly one decision to the user. If you have 3+ inputs on one screen, split them.

**Signs you need to split:**
- More than 2 required fields visible at once
- User must scroll to see the CTA button
- Screen title can't be expressed as a single question

```
// PASS: GOOD — one question per step
Step 1: "Bạn muốn đến đâu?"  → location input only
Step 2: "Khi nào bạn đến?"   → date picker only
Step 3: "Bao nhiêu khách?"   → guest counter only

// FAIL: BAD — everything on one screen
[Location input]
[Check-in date]
[Check-out date]
[Guest count]
[Promo code]
[Submit]
```

---

### 3. Step Flow Design (Wizard Pattern)

Use this anatomy for every step screen:

```
┌─────────────────────────────┐
│  ← Back         2 / 3       │  progress + back (always)
├─────────────────────────────┤
│                             │
│  Tiêu đề rõ ràng            │  heading: what to do now
│  Mô tả ngắn (tùy chọn)      │  subtext: context/why
│                             │
│  [Single input or choice]   │  one decision only
│                             │
│                             │  generous whitespace
├─────────────────────────────┤
│  [CTA — full width, sticky] │  always visible, never scrolls away
└─────────────────────────────┘
```

**Step flow rules:**
- Maximum 5 steps — redesign the flow if you need more
- Back always undoes only the current step, never restarts
- Progress indicator: dots for ≤4 steps, `"2 / 5"` text for more
- Never show step N+1 content until step N is complete
- Allow skipping only truly optional steps — label clearly "Bỏ qua"

---

### 4. CTA Buttons — Descriptive, Not Generic

The CTA label should tell the user exactly what happens next.

```
// PASS: GOOD
"Chọn ngày"
"Xác nhận · 500,000đ"
"Đặt phòng ngay"
"Tiếp theo: Thanh toán"

// FAIL: BAD
"Tiếp theo"
"OK"
"Xác nhận"
"Submit"
```

**CTA design rules:**
- Full width, sticky at bottom (`position: absolute` or inside safe area)
- Primary: `bg-primary rounded-full py-3.5`
- Disabled state: `bg-gray-300` — never hide, always show disabled
- Loading state: replace label with `<ActivityIndicator color="#fff" />`
- Show price in button when it's a payment action

---

### 5. Bottom Sheets

Use bottom sheets for: filters, date pickers, option menus, detail previews.
Never use full-screen modals for these — they feel heavy on mobile.

```
// PASS: GOOD bottom sheet spec
maxHeight: '65%'         // not 92% — leaves context visible
borderTopLeftRadius: 20
borderTopRightRadius: 20
handle: width 40, height 4, bg #e5e7eb, centered at top
```

**Bottom sheet rules:**
- Always show drag handle at top
- Backdrop tap = dismiss
- `maxHeight` between 50%–70% — never more than 80%
- Sticky footer with CTA always visible
- "Xoá tất cả" / clear action: only show when filters are active
- Sections inside sheet: use `ScrollView`, never nested FlatList

---

### 6. Progressive Disclosure

Show only what the user needs right now. Reveal more on demand.

**Patterns:**
- Card → Detail: show location + price + rating on card, full info on tap
- Collapsed sections: long content behind "Xem thêm" / "Show more"
- Filters: show top 3 most-used, hide advanced behind "Thêm bộ lọc"
- Errors: show inline after field blur, not all-at-once on submit

---

### 7. Inline Validation — Never Submit-Time Errors

```tsx
// PASS: GOOD — validate on blur
<TextInput
  onBlur={() => {
    if (!value) setError('Vui lòng nhập số điện thoại');
  }}
/>
{error && <Text className="text-red-500 text-xs mt-1">{error}</Text>}

// FAIL: BAD — validate only on submit, show all errors at once
function handleSubmit() {
  const errors = validateAll(form); // user sees 5 errors at once
}
```

---

### 8. Empty States & Loading States — Always Handle Both

Every screen that fetches data must show:
- **Loading**: `ActivityIndicator` centered, primary color
- **Empty**: illustration or icon + short message + optional CTA
- **Error**: message + "Thử lại" retry button

```tsx
if (loading) return <ActivityIndicator color="#2f9792" />;
if (error) return <ErrorState onRetry={refetch} />;
if (data.length === 0) return <EmptyState message="Không có kết quả" />;
return <DataList data={data} />;
```

---

### 9. Haptic Feedback on Key Interactions

Use light haptics on: toggle (heart/favorite), chip select, success, error.

```tsx
import { Vibration } from 'react-native';

// Light tap feedback
Vibration.vibrate(10);

// Or with react-native-haptic-feedback (preferred)
HapticFeedback.trigger('impactLight');
```

Apply to: favorite toggle, filter chip select, booking confirm success.

---

### 10. Airbnb-Specific Patterns (DLS)

| Pattern | Rule |
|---------|------|
| Search bar | Full pill `rounded-full`, white bg, `border-gray-200`, subtle shadow |
| Listing card | 4:3 image, no shadow on card, circular favorite button |
| Category bar | Active = 2px primary underline, no filled container |
| Map view | Price pins on map, listing cards in bottom sheet below |
| Filters | Bottom sheet 65% height, pill apply button, badge count |
| Booking flow | 2 steps max (form → confirm), price in CTA button |
| Photos | Full-bleed, 4:3 ratio, carousel with dot pagination |
| Trust signals | Star + review count inline on card, not just detail screen |

---

## Quick Checklist Before Shipping Any Screen

- [ ] All touch targets ≥ 44×44px
- [ ] CTA button is full-width, sticky, descriptive label
- [ ] Loading + empty + error states all handled
- [ ] No more than 2 primary inputs visible at once
- [ ] Bottom sheets ≤ 70% screen height
- [ ] Inline validation on blur (not submit)
- [ ] Back navigation works correctly
- [ ] Step indicator shown if multi-step flow
