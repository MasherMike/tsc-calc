# Legal Pages — Design Spec

**Goal:** Add Privacy Policy, Terms of Service, and Disclaimer as accordion panels inside the existing About tab.

**Architecture:** Three collapsible panels appended below the "About This App" card in `#view-about`. Accordion behavior — opening one panel closes the others. All panels start collapsed. Implemented entirely within `index.html` (inline CSS + JS), no new files or dependencies.

**Tech Stack:** Vanilla HTML/CSS/JS, CSS variables matching existing app theme.

---

## UI Structure

The About tab (`#view-about`) currently contains one card ("About This App"). Three new legal panels are added below it, each structured as:

```
┌─────────────────────────────────┐
│ [icon] Panel Title         ▶/▼  │  ← .legal-header (tappable)
├─────────────────────────────────┤
│  Legal text content...          │  ← .legal-body (hidden/shown)
│  Effective: March 14, 2026      │
└─────────────────────────────────┘
```

Panel order:
1. 🔒 Privacy Policy
2. 📋 Terms of Service
3. ⚠️ Disclaimer

---

## Accordion Behavior

- All panels start collapsed (`display: none` on `.legal-body`)
- `toggleLegal(id)` function: closes all panels, then opens the targeted one (unless it was already open — in that case collapses it)
- Chevron icon rotates 90° on open via CSS `transform: rotate(90deg)` transition
- Matches existing dark mode automatically via CSS variables (`--light`, `--mid`, `--dark`, `--white`, `--teal`)

---

## CSS

New classes added to the `<style>` block:

| Class | Purpose |
|---|---|
| `.legal-panel` | Wrapper card — matches existing `.card` style |
| `.legal-header` | Tappable row with title + chevron, cursor pointer |
| `.legal-body` | Collapsible content area, `display:none` by default |
| `.legal-body.open` | `display:block` when expanded |
| `.legal-chevron` | `▶` span that rotates on open |

---

## JavaScript

One function added to the script block:

```js
function toggleLegal(id) {
  const allBodies = document.querySelectorAll('.legal-body');
  const allChevrons = document.querySelectorAll('.legal-chevron');
  const target = document.getElementById(id);
  const isOpen = target.classList.contains('open');
  allBodies.forEach(b => b.classList.remove('open'));
  allChevrons.forEach(c => c.classList.remove('open'));
  if (!isOpen) {
    target.classList.add('open');
    target.previousElementSibling.querySelector('.legal-chevron').classList.add('open');
  }
}
```

---

## Content

### Privacy Policy
**Effective: March 14, 2026**

- No personal data is collected or transmitted to external servers during normal use.
- All user data (saved orders, favorites, goals) is stored exclusively in `localStorage` on the user's device.
- No cookies, no third-party analytics, no tracking.
- The AI analysis feature routes requests through a secure proxy server. No user data is logged or retained by the proxy.
- Data practices may change as features expand. This policy will be updated accordingly.
- ~~Contact: legal@obsidiancorvus.com~~ _(deferred — contact email not yet active; will be added once legal@obsidiancorvus.com is set up)_

### Terms of Service
**Effective: March 14, 2026**

- TSC Nutrition Calculator ("the App") is a product of Obsidian Corvus Strategy LLC.
- The App is provided "as is" without warranties of any kind, express or implied.
- Nutritional data is sourced from restaurant-published materials and is provided for informational purposes only. Values may vary based on preparation and serving size.
- The App is not intended for use in making medical, clinical, or dietary treatment decisions.
- All intellectual property, including the App's design, code, and methodology, is owned by Obsidian Corvus Strategy LLC. Unauthorized reproduction or distribution is prohibited.
- Continued use of the App constitutes acceptance of these terms.
- Terms may be updated at any time. Check this page for the most current version.

### Disclaimer
**Effective: March 14, 2026**

- Nutritional information displayed in this App is for general informational purposes only.
- Values are based on standard serving sizes as published by the restaurant and may differ from actual prepared items.
- This App is not a substitute for professional nutrition, dietary, or medical advice.
- Always consult the in-store nutrition guide and a qualified professional for decisions affecting your health.
- Obsidian Corvus Strategy LLC makes no representations as to the accuracy or completeness of any information on this App.

---

## Files Modified

| File | Change |
|---|---|
| `index.html` | Add CSS classes to `<style>` block; add 3 legal panel HTML blocks to `#view-about`; add `toggleLegal()` to script block |

No new files created. Single-file constraint preserved.

---

## Testing Checklist

- [ ] All three panels start collapsed on page load
- [ ] Tapping a header expands its panel
- [ ] Tapping an open header collapses it
- [ ] Opening one panel collapses any other open panel
- [ ] Chevron rotates correctly on open/close
- [ ] Panels render correctly in light mode
- [ ] Panels render correctly in dark mode
- [ ] Content is readable on mobile (iPhone-sized viewport)
- [ ] No JS errors in console
