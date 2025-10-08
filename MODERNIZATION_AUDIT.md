# SS LINE Website Modernization Audit

**Project:** SS LINE (エスエスライン) Corporate Website Redesign
**Repository:** https://github.com/newtonsuu/ssline-redesign
**Live URL:** https://newtonsuu.github.io/ssline-redesign/
**Audit Date:** 2026-07-12
**Auditor:** Development Team

---

## 1. Project Overview

SS LINE is a Japanese logistics company (関東全域対応の運送会社) operating a fleet of 35+ trucks across the Kanto region. The corporate website is a single-file static site deployed on GitHub Pages. The project is in active redesign, with several new sections recently added on top of an existing base structure.

---

## 2. Technology Stack

| Layer | Technology | Notes |
|---|---|---|
| Markup | HTML5 | Single file — `index.html` (2,196 lines) |
| Styling | CSS3 | Inline `<style>` block inside `index.html` |
| Scripting | Vanilla JavaScript | Inline `<script>` block inside `index.html` |
| Fonts | Google Fonts | Noto Sans JP 400 / 500 / 700 / 900 |
| Icons | Tabler Icons Webfont | Loaded via unpkg CDN |
| Images | Local PNG / JPG | 11 files in `/images/` — 4 hero slides + 7 truck photos |
| Build Tools | None | No package.json, no bundler, no preprocessor |
| Hosting | GitHub Pages | Served via Fastly CDN |
| Version Control | Git / GitHub | `main` branch; no CI/CD pipeline |

**Assessment:** The single-file, no-build approach is appropriate for this project's scale and the client's likely maintenance capacity. It keeps the deployment surface minimal and avoids dependency rot. The primary risk of this approach is file size growth — at 2,196 lines the file is approaching the threshold where it becomes difficult to navigate without tooling.

---

## 3. Page Structure and Routing

The site is a single-page application using in-page anchor navigation. All routes are fragment identifiers within `index.html`.

| Section ID | Label (JP) | Label (EN) | Status | Notes |
|---|---|---|---|---|
| `#hero` | ヒーロー | Hero | Stable | 4-slide Ken Burns carousel |
| `#intro` | 企業紹介ストリップ | Intro Strip | Stable | Tagline bar between hero and stats |
| `#stats` | 実績 | Stats | Stable | 4 animated counters |
| `#about` | 会社概要 | About | Recently Added | Company table + 4 value cards |
| `#message` | 社長メッセージ | President's Message | Recently Added | CEO message, sticky avatar |
| `#information` | お知らせ | News / Information | Stable | 5 news items |
| `#services` | サービス | Services | Stable | 4 service cards |
| `#fleet` | 車両紹介 | Fleet | Recently Added | 7 truck cards with lightbox |
| `#ecology` | エコへの取り組み | Ecology | Recently Added | 3 environmental initiative cards |
| `#safety` | 安全への取り組み | Safety | Recently Added | 4 safety items |
| `#recruit` | 採用情報 | Recruitment | Recently Added | 2 job listings |
| `#cta` | お問い合わせ | Contact CTA | Stable | Phone + email button only |
| `#quick` | クイックナビ | Quick Nav | Stable | 5 navigation shortcut cards |
| `#footer` | フッター | Footer | Stable | 4-column link grid |

---

## 4. Feature Inventory

### 4.1 Internationalization
- JP/EN bilingual toggle; all user-visible strings are duplicated inside `[data-jp]` / `[data-en]` patterns or toggled via JS class switching
- Preference persisted to `localStorage` key `ssl-lang`
- Default language: Japanese

### 4.2 Theme System
- Dark/Light mode toggle
- Preference persisted to `localStorage` key `ssl-theme`
- FOUC prevention implemented: an inline `<script>` block at the top of `<head>` reads `localStorage` before any CSS renders, applying the correct theme class to `<html>` before the first paint
- CSS custom properties drive all theme-sensitive colors

### 4.3 Hero Carousel
- 4 slides with Ken Burns pan-and-zoom effect (CSS `@keyframes`)
- Auto-advances every 5 seconds
- Pauses on hover (`:hover` + JS `mouseenter`/`mouseleave`)
- Dot navigation indicators
- Previous / Next arrow controls
- `prefers-reduced-motion` detection disables Ken Burns and auto-advance

### 4.4 Scroll Animations
- `IntersectionObserver` triggers `.visible` class on all major section elements
- Default animation: fade + translate-Y (fade-up)
- `prefers-reduced-motion` disables all scroll animations

### 4.5 Stats Counter
- `requestAnimationFrame` loop with ease-out cubic easing
- Counts from 0 to target value over ~1.8 seconds
- Triggered once on first intersection

### 4.6 Fleet Lightbox
- Click any `.fleet-card` to open `#lb` modal overlay
- Modal shows: truck photo, full specifications table, feature list
- Previous / Next navigation arrows inside lightbox
- Close via button, Escape key, or clicking the overlay
- Bilingual content within lightbox

### 4.7 Navigation
- Fixed/sticky navbar with glassmorphism backdrop (`backdrop-filter: blur`) on scroll
- Desktop: horizontal nav with dropdown sub-menus on hover
- Mobile: hamburger icon opens full-screen overlay nav (`#mnav`)
- Mobile sub-menus are collapsible accordions
- Back-to-top button (`#totop`) appears after 600px scroll

### 4.8 Accessibility Features
- Skip link (`#main-content`) at top of page
- `prefers-reduced-motion` media query respected throughout
- Semantic HTML5 landmark elements (`<nav>`, `<main>`, `<section>`, `<footer>`, `<article>`)
- `alt` attributes on all images

---

## 5. Component Inventory

| Component | Selector(s) | Description |
|---|---|---|
| Navbar | `#nav` | Fixed top bar; desktop dropdowns; lang + theme toggles |
| Mobile Nav | `#mnav` | Full-screen overlay; collapsible sub-menus |
| Hero | `.carousel`, `.slide` | 4-slide Ken Burns carousel |
| Intro Strip | `#intro` | Company tagline between hero and stats |
| Stats Bar | `.stats-grid`, `.stat-item` | 4 animated count-up stats |
| About Grid | `.about-grid` | 2-column layout: company profile table + values |
| Message Section | `.msg-grid` | 2-column: sticky CEO avatar + long-form message |
| News List | `.news-list`, `.ni` | Vertical list of dated news items |
| Services | `.svc-grid`, `.svc-card` | 4-column service offering cards |
| Fleet Gallery | `.fleet-grid`, `.fleet-card` | Truck photo cards with hover overlay |
| Fleet Lightbox | `#lb` | Fixed modal dialog for truck specs |
| Ecology | `.eco-grid`, `.eco-card` | 3-column environmental initiative cards |
| Safety | `.saf-grid`, `.saf-item` | Safety items with left-border accent |
| Recruitment | `.job-grid`, `.job-card` | Job listing cards with detail table |
| Contact CTA | `#cta` | Phone number + email action strip |
| Quick Nav | `#quick`, `.qm-card` | 5-card shortcut navigation row |
| Footer | `footer` | 2-column: address block + 4-column link grid |
| Back-to-Top | `#totop` | Floating button, scroll-triggered visibility |

---

## 6. Data and External Dependencies

### 6.1 localStorage
| Key | Values | Purpose |
|---|---|---|
| `ssl-theme` | `"dark"` / `"light"` | Persist user theme preference |
| `ssl-lang` | `"jp"` / `"en"` | Persist user language preference |

### 6.2 Contact Information
| Type | Value |
|---|---|
| Phone | `tel:043-460-2607` |
| Email | `mailto:info@ssline.co.jp` |

### 6.3 External CDN Dependencies
| Resource | Provider | URL Pattern | Risk |
|---|---|---|---|
| Noto Sans JP | Google Fonts | `fonts.googleapis.com` | Low — very stable CDN |
| Tabler Icons Webfont | unpkg | `unpkg.com/@tabler/icons-webfont` | Medium — unpkg has occasional outages; no version pin visible |

### 6.4 Local Assets (`/images/`)
| File Purpose | Count | Format |
|---|---|---|
| Hero carousel slides | 4 | PNG / JPG |
| Fleet truck photos | 7 | PNG / JPG |

No server-side code, database, or API calls. The site is fully static.

---

## 7. Recently Completed Work

All of the following were added to `index.html` and committed to `main`:

1. **`#about` section** — Company profile table (established date, capital, address, fleet size, coverage area) and 4 company value cards with bilingual content
2. **`#message` section** — CEO / President message with sticky portrait avatar, long-form bilingual text, and signature block
3. **Fleet lightbox modal (`#lb`)** — Full-screen modal overlay for truck detail view: specifications table, feature list, prev/next navigation, keyboard (Escape) dismissal, hover overlay ("詳細を見る / View Details") on fleet cards
4. **`#ecology` section** — 3 environmental initiative cards (eco-drive, idle reduction, green procurement) with bilingual content
5. **`#safety` section** — 4 safety initiative items with left-border accent styling and bilingual content
6. **`#recruit` section** — 2 job listing cards (driver, office) with compensation/hours/requirement detail tables and bilingual content

---

## 8. Known Issues

The following issues were identified during audit. They are ordered by user impact.

### Issue 1 — Dead Links (href="#")
**Severity:** High
**Impact:** Multiple calls-to-action throughout the site silently scroll the user back to the top of the page instead of navigating to the intended section.

Affected locations:
- Hero slide 2 CTA button — should link to `#recruit`
- All 4 service card "learn more" / 詳しく links — no dedicated sub-page; should link to `#services`
- Footer "company info" links — should link to `#message` and `#about`
- Footer ecology / safety / recruit / other section links — should link to real section anchors (`#ecology`, `#safety`, `#recruit`)
- News "view all" button — should link to `#information`

### Issue 2 — Stale `<link rel="preconnect">` for Unsplash
**Severity:** Low
**Impact:** A `<link rel="preconnect" href="https://images.unsplash.com" />` tag remains in `<head>`. All images are now served locally from `/images/`; Unsplash is not used anywhere in the current codebase. This preconnect establishes a pointless TCP/TLS connection on every page load.

### Issue 3 — No Scroll Spy
**Severity:** Medium
**Impact:** The sticky navbar does not reflect the user's current scroll position. No nav item is ever marked active, and `aria-current` is never set on any nav link, reducing both usability and accessibility for assistive technology users.

### Issue 4 — No Touch / Swipe Support on Hero Carousel
**Severity:** Medium
**Impact:** Mobile users cannot swipe left/right to advance carousel slides. The carousel is only controllable via the dot indicators or arrow buttons, which are small touch targets on mobile screens.

### Issue 5 — Lightbox Focus Management
**Severity:** Medium (Accessibility)
**Impact:** When the fleet lightbox opens, keyboard focus remains on the card that was clicked — it does not move into the modal. There is no focus trap inside the modal, so Tab key allows focus to escape behind the overlay. When the lightbox closes, focus is not returned to the triggering card. This fails WCAG 2.1 Success Criteria 2.1.2 (No Keyboard Trap) and 2.4.3 (Focus Order).

### Issue 6 — Mobile Nav Does Not Auto-Close on Sub-Link Click
**Severity:** Low
**Impact:** Tapping a sub-menu link inside the mobile nav overlay (`#mnav`) navigates to the section but leaves the full-screen nav overlay open on top of the content. The user must manually tap the hamburger/close button to dismiss it.

### Issue 7 — No Contact Form
**Severity:** Medium (Business)
**Impact:** The CTA section (`#cta`) only provides a phone number and a `mailto:` email button. There is no structured inquiry form. Users on mobile may not have a default mail client configured, and there is no way to collect structured inquiry data (name, company, inquiry type).

### Issue 8 — CLS Risk on Fleet Images
**Severity:** Low (Performance)
**Impact:** Fleet card `<img>` elements do not have explicit `width` and `height` attributes. Browsers cannot reserve layout space before the images load, potentially causing Cumulative Layout Shift (CLS) — a Core Web Vital metric.

### Issue 9 — Inconsistent Typography (stat-n)
**Severity:** Low (Code Quality)
**Impact:** At least one `.stat-n` element uses an inline `style="font-size:..."` attribute for font sizing, while all other stats use the shared CSS class. This inconsistency makes future typography changes error-prone.

### Issue 10 — Footer Dead Links
**Severity:** Medium
**Impact:** Several footer columns contain `href="#"` links that scroll to the top. This duplicates Issue 1 but is called out separately because footer nav is a common destination for users looking for site structure.

---

## 9. Responsive Design Assessment

| Breakpoint | Target | Implementation |
|---|---|---|
| > 1100px | Full desktop | 4-column grids, sticky nav, full dropdown menus |
| 960px – 1100px | Tablet landscape | Reduced column counts, adjusted card sizing |
| 640px – 960px | Tablet portrait / large mobile | Hamburger nav activates, 2-column layouts |
| 400px – 640px | Mobile | Single-column layouts, adjusted typography |
| < 400px | Small mobile | Compact spacing, font-size reductions |

The responsive implementation is functional. No major layout breaks were identified. The primary mobile UX issue is the carousel swipe gap (Issue 4) and the mobile nav auto-close gap (Issue 6).

---

## 10. Performance Notes

- **No image optimization pipeline:** Images are served as-is from `/images/`. No WebP conversion, no `srcset`, no lazy loading attributes on below-fold images.
- **Render-blocking fonts:** Google Fonts are loaded with `<link>` in `<head>`. The `font-display: swap` approach should be confirmed in the `@import` or `<link>` parameters.
- **Single large HTML file:** At 2,196 lines with inline CSS and JS, the file is moderately large for a single resource. There is no minification. This is acceptable for the current traffic expectations but worth noting.
- **CDN dependency:** Tabler Icons is loaded from unpkg without a version lock in the URL. A breaking update to the package could silently break all icons.

---

## 11. Summary Scorecard

| Category | Score | Notes |
|---|---|---|
| Feature Completeness | 8/10 | All major sections present; contact form missing |
| Accessibility | 5/10 | Skip link + reduced-motion present; lightbox focus, scroll spy, aria-current missing |
| Code Quality | 7/10 | Clean, consistent patterns; minor inline style inconsistency |
| Performance | 6/10 | No image optimization, no lazy loading, no version-pinned CDN |
| Mobile UX | 7/10 | Good responsive layout; swipe and nav-close gaps |
| Link Integrity | 4/10 | Significant number of dead href="#" links in production |

---

*This audit covers the state of the codebase as of 2026-07-12. All items identified in Section 8 are tracked in `MODERNIZATION_PLAN.md` with phased resolution plans.*
