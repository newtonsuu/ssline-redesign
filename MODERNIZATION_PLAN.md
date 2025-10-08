# SS LINE Website Modernization Plan

**Project:** SS LINE (エスエスライン) Corporate Website Redesign
**Repository:** https://github.com/newtonsuu/ssline-redesign
**Live URL:** https://newtonsuu.github.io/ssline-redesign/
**Plan Date:** 2026-07-12
**Based on:** `MODERNIZATION_AUDIT.md` — Issues 1–10

---

## Overview

This plan organizes all identified improvements into six phased deliverables. Phases are ordered by impact and risk — Phase 1 fixes visible production defects, later phases add enhancements. Each phase can be implemented and deployed independently. All changes are confined to `index.html` unless otherwise noted.

---

## Phase 1 — Bug Fixes: Dead Links and Stale Resources

**Priority:** Critical
**Effort:** Low (< 1 hour)
**Addresses:** Audit Issues 1, 2, 10

### Rationale

Multiple calls-to-action and footer links silently scroll the page to the top because they target `href="#"`. This is a production defect — it actively misleads users and wastes the navigation work done in the new sections. This is the highest-priority fix.

### Changes

#### 1.1 Hero Slide 2 CTA Button

**File:** `index.html`
**Current behavior:** CTA button on hero slide 2 scrolls to the top of the page.
**New behavior:** Navigates to the `#recruit` section.

```html
<!-- Before -->
<a href="#" class="btn btn-primary">採用情報を見る / View Jobs</a>

<!-- After -->
<a href="#recruit" class="btn btn-primary">採用情報を見る / View Jobs</a>
```

#### 1.2 Service Card "Learn More" Links (×4)

**File:** `index.html`
**Current behavior:** Each service card "詳しく / Learn More" link scrolls to the top.
**New behavior:** Links to `#services` (the services section itself, since no dedicated sub-pages exist). This is a holding state — when/if dedicated service pages are built, these links will be updated to point to them.

```html
<!-- Before (all 4 cards) -->
<a href="#" class="svc-link">詳しく / Learn More</a>

<!-- After -->
<a href="#services" class="svc-link">詳しく / Learn More</a>
```

#### 1.3 Footer Company Links

**File:** `index.html`
**Current behavior:** Footer links labeled "会社概要" and "社長メッセージ" (and English equivalents) target `href="#"`.
**New behavior:** Wire to real section anchors.

| Footer Link Label | Target Anchor |
|---|---|
| 会社概要 / About Us | `#about` |
| 社長メッセージ / President's Message | `#message` |

#### 1.4 Footer Section Links

**File:** `index.html`
**Current behavior:** Footer links for ecology, safety, recruitment, and other recently-added sections target `href="#"`.
**New behavior:** Wire to real section anchors.

| Footer Link Label | Target Anchor |
|---|---|
| エコへの取り組み / Ecology | `#ecology` |
| 安全への取り組み / Safety | `#safety` |
| 採用情報 / Recruitment | `#recruit` |
| 車両紹介 / Fleet | `#fleet` |

#### 1.5 News "View All" Button

**File:** `index.html`
**Current behavior:** "すべて見る / View All" button in the information section targets `href="#"`.
**New behavior:** Points to `#information` to scroll back to the top of the news section. (No separate news archive page exists; this is a holding state.)

```html
<!-- Before -->
<a href="#" class="btn btn-outline">すべて見る / View All</a>

<!-- After -->
<a href="#information" class="btn btn-outline">すべて見る / View All</a>
```

#### 1.6 Remove Stale Unsplash Preconnect

**File:** `index.html`
**Current behavior:** `<link rel="preconnect" href="https://images.unsplash.com" />` in `<head>` causes a pointless TCP/TLS handshake on every page load. Images are now served locally.
**New behavior:** Tag removed entirely.

```html
<!-- Remove this line from <head> -->
<link rel="preconnect" href="https://images.unsplash.com" />
```

### Testing Required
- Click every CTA button and "learn more" link and confirm they navigate to the correct section
- Click every footer link and confirm correct section navigation
- Verify in DevTools Network tab that no connection to `images.unsplash.com` is initiated on load

### Risk Assessment
None. Pure anchor href value changes. No JavaScript, no CSS changes. Cannot break existing functionality.

---

## Phase 2 — JavaScript Improvements

**Priority:** High
**Effort:** Medium (2–4 hours)
**Addresses:** Audit Issues 3, 4, 5, 6

### Rationale

These four issues collectively degrade the experience for the two largest user segments: mobile users (Issues 4, 6) and keyboard/assistive technology users (Issues 3, 5). They are all JavaScript-only changes.

### Changes

#### 2.1 Scroll Spy — Active Nav Highlighting

**File:** `index.html` (JS block)
**Current behavior:** No nav item is ever marked active; `aria-current` is never set.
**New behavior:** As the user scrolls, the nav link corresponding to the section closest to the viewport top gains the `.active` class and `aria-current="page"`. All other nav links have these removed.

**Implementation approach:**
- Create an `IntersectionObserver` instance that observes all `section[id]` elements
- Use a `rootMargin` of `-40% 0px -55% 0px` to fire when a section occupies the middle band of the viewport
- On intersection change, find the corresponding `<a href="#sectionId">` in `#nav` and toggle `.active` + `aria-current`
- Use a `Map` to track currently-intersecting sections; pick the topmost when multiple sections intersect simultaneously

**CSS required (add to existing style block):**
```css
.nav-item > a.active {
  color: var(--mint);
  border-bottom: 2px solid var(--mint);
}
```

#### 2.2 Touch / Swipe Support for Hero Carousel

**File:** `index.html` (JS block)
**Current behavior:** Hero carousel does not respond to touch gestures on mobile.
**New behavior:** Swiping left advances to the next slide; swiping right goes to the previous slide. A 50px horizontal displacement threshold prevents accidental swipes during vertical scrolling.

**Implementation approach:**
```javascript
// Add to carousel initialization
const carousel = document.querySelector('.carousel');
let touchStartX = 0;
let touchStartY = 0;

carousel.addEventListener('touchstart', (e) => {
  touchStartX = e.touches[0].clientX;
  touchStartY = e.touches[0].clientY;
}, { passive: true });

carousel.addEventListener('touchend', (e) => {
  const dx = e.changedTouches[0].clientX - touchStartX;
  const dy = e.changedTouches[0].clientY - touchStartY;
  // Only treat as horizontal swipe if horizontal movement dominates
  if (Math.abs(dx) > 50 && Math.abs(dx) > Math.abs(dy)) {
    dx < 0 ? goToNextSlide() : goToPrevSlide();
  }
});
```

The `{ passive: true }` flag on `touchstart` ensures the browser does not wait for the handler before scrolling, preserving scroll performance.

#### 2.3 Lightbox Focus Management

**File:** `index.html` (JS block)
**Current behavior:** Focus does not move into the lightbox on open; Tab escapes the modal overlay; focus is not restored on close.
**New behavior:** On open — focus moves to the lightbox close button. Inside the modal — Tab cycles only through focusable elements within `#lb`. On close — focus returns to the fleet card that triggered the open.

**Implementation approach:**
```javascript
// Track the element that triggered the lightbox
let lbTriggerEl = null;

function openLightbox(cardEl) {
  lbTriggerEl = cardEl;
  // ... existing open logic ...
  const lb = document.getElementById('lb');
  lb.removeAttribute('hidden');
  // Move focus to close button after transition
  const closeBtn = lb.querySelector('.lb-close');
  closeBtn.focus();
}

function closeLightbox() {
  const lb = document.getElementById('lb');
  lb.setAttribute('hidden', '');
  // Return focus to the triggering card
  if (lbTriggerEl) {
    lbTriggerEl.focus();
    lbTriggerEl = null;
  }
}

// Focus trap: intercept Tab inside #lb
document.getElementById('lb').addEventListener('keydown', (e) => {
  if (e.key !== 'Tab') return;
  const focusable = [...lb.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  )];
  const first = focusable[0];
  const last = focusable[focusable.length - 1];
  if (e.shiftKey && document.activeElement === first) {
    e.preventDefault();
    last.focus();
  } else if (!e.shiftKey && document.activeElement === last) {
    e.preventDefault();
    first.focus();
  }
});
```

Also add `role="dialog"`, `aria-modal="true"`, and `aria-labelledby` pointing to the lightbox heading to `#lb`.

#### 2.4 Mobile Nav Auto-Close on Sub-Link Click

**File:** `index.html` (JS block)
**Current behavior:** Tapping a link inside `#mnav` navigates to the section but leaves the overlay open.
**New behavior:** Any `<a>` click inside `#mnav` triggers the existing mobile nav close sequence (same as tapping the hamburger/close button).

**Implementation approach:**
```javascript
// Add after mobile nav initialization
document.querySelectorAll('#mnav a[href]').forEach(link => {
  link.addEventListener('click', () => {
    closeMobileNav(); // call the existing close function
  });
});
```

This piggybacks on the existing close function — no new state is introduced.

### Testing Required
- Scroll through the full page and verify the correct nav item is highlighted at each section
- Test carousel swipe on a real mobile device or Chrome DevTools touch simulation
- Open the fleet lightbox with keyboard only (Enter on a fleet card); verify focus moves to close button; Tab through; verify trap; Escape to close; verify focus returns
- On mobile, tap a mobile nav sub-link; verify the overlay closes and the correct section is visible

### Risk Assessment
Low. Changes are additive JavaScript. Existing carousel auto-advance, keyboard arrow controls, and dot navigation remain unchanged. Scroll spy is read-only — it never modifies section content, only nav link classes.

---

## Phase 3 — Contact Form Section

**Priority:** Medium
**Effort:** Medium (3–5 hours)
**Addresses:** Audit Issue 7

### Rationale

The current CTA section provides only a phone number and a `mailto:` link. Users without a configured mail client have no way to send an inquiry. A structured form also makes it easier for the company to receive well-formatted inquiries. Since the site is fully static (no server), the form will use a `mailto:` fallback with client-side validation and a success state.

### Changes

#### 3.1 New `#contact` Section

**File:** `index.html`
**Placement:** After `#cta`, before `#quick`

The section contains a 6-field inquiry form:

| Field | JP Label | EN Label | Type | Required |
|---|---|---|---|---|
| 氏名 | 氏名 | Name | `text` | Yes |
| 会社名 | 会社名 | Company Name | `text` | No |
| 電話番号 | 電話番号 | Phone Number | `tel` | No |
| メールアドレス | メールアドレス | Email Address | `email` | Yes |
| お問い合わせ種別 | お問い合わせ種別 | Inquiry Type | `select` | Yes |
| お問い合わせ内容 | お問い合わせ内容 | Message | `textarea` | Yes |

Inquiry Type select options (bilingual):
- 一般貨物 / General Freight
- 企業間物流 / B2B Logistics
- 定期便 / Scheduled Delivery
- チャーター / Charter
- 採用 / Recruitment
- その他 / Other

#### 3.2 Client-Side Validation

On form submit:
1. Check all required fields are non-empty
2. Check email field matches a valid email pattern
3. If validation fails: add `.field-error` class to the offending fields and show bilingual inline error messages below each field
4. If validation passes: open `mailto:info@ssline.co.jp` with pre-filled subject and body constructed from form values, then display a success message replacing the form

**Error message examples:**
- Required field: `このフィールドは必須です。 / This field is required.`
- Invalid email: `有効なメールアドレスを入力してください。 / Please enter a valid email address.`

#### 3.3 Success State

After successful submission, replace the form with a confirmation panel:
```
お問い合わせありがとうございます。
Thank you for your inquiry.
担当者よりご連絡いたします。
Our team will be in touch shortly.
```

Include a "別のお問い合わせ / Send Another" link that restores the form.

#### 3.4 CSS

Add form-specific styles to the existing inline style block:
- `.form-group`, `.form-label`, `.form-input`, `.form-select`, `.form-textarea` for layout
- `.field-error` for red border + error icon on invalid fields
- `.error-msg` for inline error text below each field
- `.form-success` for the confirmation panel
- Styles must respect both dark and light theme CSS custom properties
- Bilingual label visibility controlled by the existing lang toggle mechanism

### Testing Required
- Submit form with all fields empty — all required fields should show error states
- Submit with invalid email format — email field should show error
- Submit with all required fields valid — `mailto:` should open with pre-filled content and success message should appear
- Test in both JP and EN language modes
- Test in both dark and light themes
- Verify form is accessible: all inputs have associated `<label>` elements; error messages are associated via `aria-describedby`

### Risk Assessment
Low. This is an additive new section. The `mailto:` fallback is the same mechanism already used by the existing email button in `#cta`. No new external dependencies are introduced.

---

## Phase 4 — Hero Progress Bar

**Priority:** Low–Medium
**Effort:** Low (< 1 hour)
**Addresses:** Audit Issue — UX enhancement (not in original issue list)

### Rationale

The hero carousel advances automatically every 5 seconds with no visible indication to the user of when the next slide transition will occur. A progress bar provides temporal context, reduces perceived unpredictability, and visually reinforces the 5-second interval.

### Changes

#### 4.1 HTML

**File:** `index.html`
**Placement:** Immediately after the `.carousel` element, inside the hero section.

```html
<div class="hero-progress" aria-hidden="true">
  <div class="hero-progress-bar" id="heroProgressBar"></div>
</div>
```

`aria-hidden="true"` is appropriate — the progress bar is purely decorative. Screen reader users get no value from being notified about a countdown timer on a decorative slideshow.

#### 4.2 CSS

```css
.hero-progress {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 3px;
  background: rgba(255,255,255,0.2);
  z-index: 10;
}

.hero-progress-bar {
  height: 100%;
  background: var(--mint);
  width: 0%;
  animation: progressFill 5s linear forwards;
}

@keyframes progressFill {
  from { width: 0%; }
  to   { width: 100%; }
}

/* Respect reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  .hero-progress-bar { animation: none; width: 100%; }
}
```

#### 4.3 JavaScript

On each slide advance (auto or manual), reset the progress bar animation:
```javascript
function resetProgressBar() {
  const bar = document.getElementById('heroProgressBar');
  bar.style.animation = 'none';
  // Force reflow to restart animation
  bar.offsetHeight;
  bar.style.animation = 'progressFill 5s linear forwards';
}
```

Call `resetProgressBar()` inside the existing `goToSlide()` function (or equivalent) after updating the active slide index.

When the carousel is paused on hover, the progress bar animation should also pause:
```css
.carousel:hover .hero-progress-bar {
  animation-play-state: paused;
}
```

### Testing Required
- Verify progress bar fills from left to right over exactly 5 seconds
- Verify bar resets instantly on manual slide change (dot click or arrow click)
- Verify bar pauses on carousel hover
- Verify bar is not visible or non-animating when `prefers-reduced-motion` is enabled

### Risk Assessment
None. CSS animation isolated to new elements. No changes to existing carousel JS logic beyond a single function call addition.

---

## Phase 5 — Active Nav Styling

**Priority:** Medium
**Effort:** Low (< 1 hour)
**Depends on:** Phase 2.1 (scroll spy must be implemented first)

### Rationale

Once the scroll spy from Phase 2.1 is in place, the `.active` class is already being toggled on nav links. This phase adds the visual CSS treatment for those active states and ensures dropdown parent links also reflect active sub-section state.

### Changes

#### 5.1 Active Link CSS

**File:** `index.html` (style block)

```css
/* Top-level nav link active state */
.nav-item > a.active {
  color: var(--mint);
}

/* Accent underline on active link */
.nav-item > a.active::after {
  content: '';
  display: block;
  width: 100%;
  height: 2px;
  background: var(--mint);
  margin-top: 2px;
  border-radius: 1px;
}

/* Parent nav item active when a child section is in view */
.nav-item.has-active > a {
  color: var(--mint);
}
```

#### 5.2 Dropdown Parent Active State

When a section that lives under a dropdown group (e.g., `#about`, `#message` under the "会社情報" parent) is active, the scroll spy should also add `has-active` to the parent `.nav-item`:

```javascript
// In scroll spy observer callback, after setting active on direct link:
const parentNavItem = activeLink?.closest('.nav-item.has-dropdown');
if (parentNavItem) {
  // Clear has-active from all items first
  document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('has-active'));
  parentNavItem.classList.add('has-active');
}
```

#### 5.3 aria-current

The scroll spy (Phase 2.1) sets `aria-current="page"` on the active link. Confirm the CSS does not rely solely on `aria-current` for visual styling (to maintain separation between ARIA semantics and visual state) — `.active` class drives the visual; `aria-current` drives the accessibility tree.

### Testing Required
- Scroll through all sections and confirm the correct nav item becomes highlighted at each section entry point
- For sections under a dropdown parent, confirm the parent nav label also gets the active highlight
- Inspect with a screen reader or axe DevTools: confirm `aria-current="page"` is present on exactly one nav link at a time

### Risk Assessment
None. CSS additions only. The `.active` class is already in use by Phase 2.1; this phase only adds visual declarations.

---

## Phase 6 — Minor Accessibility and Polish

**Priority:** Low
**Effort:** Low–Medium (2–3 hours)
**Addresses:** Audit Issues 8, 9, and remaining minor accessibility gaps

### Changes

#### 6.1 Fleet Image Dimensions (CLS Prevention)

**File:** `index.html`
**Current behavior:** Fleet card `<img>` tags have no `width` / `height` attributes.
**New behavior:** Add explicit `width` and `height` matching each image's natural pixel dimensions.

For each fleet card image, look up the image file's actual dimensions (e.g., using browser DevTools or `identify` from ImageMagick) and add:
```html
<img src="images/truck1.jpg" alt="..." width="800" height="534" loading="lazy">
```

Also add `loading="lazy"` to all below-fold fleet images. The first 1–2 fleet images (if visible in the initial viewport on desktop) should use `loading="eager"` or omit the attribute.

#### 6.2 Fix Inline Font Size on stat-n (Typography Consistency)

**File:** `index.html`
**Current behavior:** One `.stat-n` element uses `style="font-size:..."` inline.
**New behavior:** Remove inline style; if this stat requires a different size, add a modifier class (e.g., `.stat-n--lg`) and declare the font size in the CSS block.

This prevents the inconsistency from being accidentally retained in a future refactor of the `.stat-n` base class.

#### 6.3 aria-live on Stats Section

**File:** `index.html`
**Current behavior:** The count-up animation updates number text content with no ARIA notification.
**New behavior:** Add `aria-live="polite"` and `aria-atomic="true"` to each `.stat-n` element so that screen readers announce the final value after the animation completes.

```html
<span class="stat-n" aria-live="polite" aria-atomic="true">0</span>
```

Note: `aria-live="polite"` means the announcement waits for the user to be idle, which is appropriate here — we do not want to interrupt ongoing speech.

#### 6.4 Scope scroll-margin-top to Section Elements

**File:** `index.html` (style block)
**Current behavior:** The `[id]` CSS attribute selector applies `scroll-margin-top` to every element with an ID, including the lightbox modal, the mobile nav, and other non-anchor elements.
**New behavior:** Scope the rule to section elements only.

```css
/* Before */
[id] { scroll-margin-top: 72px; }

/* After */
section[id], div[id].section-anchor { scroll-margin-top: 72px; }
```

#### 6.5 Focus-Visible Styles for Dark Mode

**File:** `index.html` (style block)
**Current behavior:** Default browser focus rings may be invisible against dark backgrounds.
**New behavior:** Add an explicit `:focus-visible` style for dark mode that uses a high-contrast outline.

```css
@media (prefers-color-scheme: dark) {
  :focus-visible {
    outline: 2px solid var(--mint);
    outline-offset: 3px;
  }
}

/* Also scoped to the .dark class for theme-toggle users */
.dark :focus-visible {
  outline: 2px solid var(--mint);
  outline-offset: 3px;
}
```

#### 6.6 Print Media Query

**File:** `index.html` (style block)
**Current behavior:** No print styles; printing the page includes the navbar, carousel controls, lightbox markup, all animations, and dark backgrounds — producing unusable output.
**New behavior:** Add a `@media print` block that:
- Hides: `#nav`, `#mnav`, `.carousel-controls`, `.carousel-dots`, `#lb`, `#totop`, `#quick`, `.hero-progress`
- Shows all sections at full width with no sidebars
- Forces white backgrounds and dark text
- Removes box shadows and `backdrop-filter` (not printable)
- Expands collapsed sections (if any)

```css
@media print {
  #nav, #mnav, .carousel-controls, .carousel-dots,
  #lb, #totop, #quick, .hero-progress { display: none !important; }

  body, section, .about-grid, .msg-grid {
    background: #fff !important;
    color: #000 !important;
    box-shadow: none !important;
  }

  section { page-break-inside: avoid; }

  a[href]::after {
    content: " (" attr(href) ")";
    font-size: 0.8em;
    color: #555;
  }
}
```

### Testing Required
- Open Chrome DevTools, throttle network to Slow 3G, and measure CLS before and after fleet image dimension fix (should reduce to near-zero)
- Use a screen reader (NVDA + Firefox or VoiceOver + Safari) and navigate to the stats section to verify the final counter values are announced
- Tab through the page in dark mode and confirm all interactive elements have a visible focus ring
- Use browser File → Print (or Ctrl+P) and verify the print preview shows clean content without nav/controls

### Risk Assessment
Low. All changes are additive or narrowing (scoping existing rules more specifically). The `scroll-margin-top` scoping change could affect scroll position for any non-section elements with IDs if they are used as anchor targets — check the `[id]` inventory before deploying.

---

## Implementation Summary

| Phase | Focus | Effort | Priority | Audit Issues Resolved |
|---|---|---|---|---|
| 1 | Dead links + stale preconnect | < 1 hr | Critical | 1, 2, 10 |
| 2 | JS: scroll spy, swipe, lightbox focus, mobile nav | 2–4 hrs | High | 3, 4, 5, 6 |
| 3 | Contact form section | 3–5 hrs | Medium | 7 |
| 4 | Hero progress bar | < 1 hr | Low–Medium | — |
| 5 | Active nav styling | < 1 hr | Medium | 3 (visual) |
| 6 | Accessibility + polish | 2–3 hrs | Low | 8, 9 |

**Total estimated effort:** 9–15 hours across all phases

---

## Files Affected

All changes are confined to:
- `index.html` — HTML structure, inline CSS block, inline JS block

No new files, no new dependencies, no build tooling changes required.

---

## Deployment Notes

GitHub Pages serves `index.html` directly from the `main` branch. Deployment is automatic on push to `main` — no additional steps required. Each phase can be committed and pushed independently. Cache invalidation is handled by GitHub Pages / Fastly automatically on each push.

---

*Phases are designed to be independent. If bandwidth is limited, prioritize Phase 1 (critical production defect) and Phase 2.3 (accessibility compliance) above all others.*
