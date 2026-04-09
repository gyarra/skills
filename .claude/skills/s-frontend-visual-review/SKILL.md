---
name: s-frontend-visual-review
description: Use when reviewing frontend work visually — after implementing UI changes, before pushing a PR, or when asked to check how a page looks and behaves
---

# Frontend Review via Chrome DevTools

Review frontend work by viewing and testing it in a real browser using Chrome DevTools MCP tools. This goes beyond lint and type checks — it verifies what the user actually sees and experiences.

## When to Use

- After implementing UI changes (new pages, components, layout modifications)
- Before pushing a PR that touches frontend code
- When asked to "check how it looks" or "test the page"
- When debugging visual or interaction bugs

## Prerequisites

The frontend dev server must be running (`cd frontend && npm run dev`). Wait for it to be ready before proceeding.

## Step 1: Navigate to the Page

Open the page under review in Chrome DevTools.

```
mcp__chrome-devtools__navigate_page → url: "http://localhost:3000/<route>"
```

If reviewing multiple pages, use `new_page` for additional tabs. Use `list_pages` and `select_page` to switch between them.

**Determine which routes to check** from the git diff — look at which files under `frontend/src/app/` changed and map them to URLs.

## Step 2: Visual Inspection

Use both tools — they check different things. The snapshot is a **text-based accessibility tree** (structure, roles, labels, element UIDs for interaction). The screenshot is a **visual image** (layout, colors, spacing).

### 2a: Accessibility Tree Snapshot

```
mcp__chrome-devtools__take_snapshot
```

Check the snapshot for:
- **Structure**: Does the page have the expected headings, sections, and interactive elements?
- **Content**: Is real data rendering (not "undefined", "null", or empty)?
- **Accessibility**: Do interactive elements have proper roles? Are images missing alt text? Are form inputs labeled?
- **Language**: User-facing text should be in Spanish (admin dashboard text in English)

### 2b: Visual Screenshot

```
mcp__chrome-devtools__take_screenshot
```

Check the screenshot for:
- **Layout**: Correct spacing, alignment, no overlapping elements
- **Colors**: Consistent with the site theme
- **Typography**: Heading hierarchy makes sense, text is readable
- **Empty states**: If no data, does it show a meaningful empty state?
- **Broken images**: Any missing or broken media (movie posters, theater logos)

### 2c: Responsive Check

Resize to mobile viewport and re-check:

```
mcp__chrome-devtools__resize_page → width: 375, height: 812
mcp__chrome-devtools__take_screenshot
```

Then reset to desktop:

```
mcp__chrome-devtools__resize_page → width: 1280, height: 800
```

Check: Does the layout stack properly on mobile? No horizontal overflow? Touch targets large enough?

## Step 3: Console Errors

```
mcp__chrome-devtools__list_console_messages → types: ["error", "warn"]
```

For each error, use `get_console_message` for the full stack trace. Common issues:
- React hydration mismatches (Server Component vs Client Component rendering differences)
- Failed Supabase queries
- Missing environment variables
- Component rendering errors

**Any console error is a finding.** Don't ignore warnings about deprecated APIs or missing keys.

## Step 4: Network Requests

```
mcp__chrome-devtools__list_network_requests → resourceTypes: ["fetch", "xhr"]
```

Check:
- **Failed requests** (4xx, 5xx status): Get details with `get_network_request`
- **Slow requests**: Any Supabase query > 2s is worth flagging
- **Unexpected requests**: Duplicate calls, requests to wrong endpoints
- **Missing requests**: Expected Supabase calls that never fire

For failed requests, inspect the response body:
```
mcp__chrome-devtools__get_network_request → reqid: <id>
```

## Step 5: Interactive Testing

Test user interactions relevant to the changed code.

### Forms
Use `take_snapshot` to get element UIDs, then:
```
mcp__chrome-devtools__fill → uid: "<input-uid>", value: "test input"
mcp__chrome-devtools__click → uid: "<submit-button-uid>"
```

After submitting, check:
- Does the form validate correctly? (try empty/invalid inputs)
- Does success feedback appear?
- Do error messages show for invalid input?
- Does the page state update correctly?

### Navigation
```
mcp__chrome-devtools__click → uid: "<link-uid>", includeSnapshot: true
```

Check: Does clicking links/buttons navigate to the right page? Does the back button work?

### Dynamic UI
For dropdowns, modals, tooltips — click to trigger them, then take a snapshot to verify they render:
```
mcp__chrome-devtools__click → uid: "<trigger-uid>"
mcp__chrome-devtools__take_snapshot
```

### Admin Pages
For admin pages, you must be logged in first. Navigate to `/auth/login`, authenticate, then proceed to `/admin/*` routes. Verify the admin layout gate blocks unauthenticated access.

## Step 6: Lighthouse Audit (Optional)

Run when reviewing a full page or before a release:

```
mcp__chrome-devtools__lighthouse_audit → device: "desktop", mode: "navigation"
```

Focus on:
- **Accessibility score**: Should be 90+. Flag any issues under 80.
- **SEO**: Missing meta tags, incorrect heading hierarchy
- **Best Practices**: Mixed content, deprecated APIs, console errors

For performance-specific audits, use the performance trace tools:
```
mcp__chrome-devtools__performance_start_trace → reload: true
mcp__chrome-devtools__performance_stop_trace
```

## Review Report Format

After completing the review, summarize findings:

```
## Frontend Review: [Page/Feature Name]

**URL**: http://localhost:3000/...
**Branch**: feature/...

### Visual
- [ ] Layout renders correctly (desktop + mobile)
- [ ] Colors consistent with theme
- [ ] Typography follows hierarchy
- [ ] Empty/loading/error states handled
- [ ] User-facing text in Spanish

### Console
- [ ] No errors
- [ ] No unhandled warnings

### Network
- [ ] All Supabase queries succeed
- [ ] No unexpected or duplicate requests

### Interactions
- [ ] Forms validate and submit correctly
- [ ] Navigation works as expected
- [ ] Dynamic UI elements open/close properly

### Issues Found
1. **[severity]** Description — evidence (screenshot/console output) — suggested fix

### Passed
- What was confirmed working
```

## Common Findings

| Symptom | Likely Cause |
|---------|-------------|
| Blank page, no errors | Missing `"use client"` on component using hooks |
| Hydration mismatch warning | Server/client render different content (often dates or auth state) |
| Supabase query returns empty | Wrong table name, RLS blocking, missing `.select()` columns |
| Layout broken on mobile only | Missing responsive classes, fixed widths instead of flex/grid |
| Flash of unstyled content | Font loading issue, missing `next/font` setup |
| Console error about missing key | List rendering without `key` prop |
| Admin page shows unauthorized | Not logged in, or user not in `admin_users` table |
| Images not loading | Supabase Storage URL expired, or TMDB poster path wrong |

## Rules

- **Always take a snapshot before interacting.** You need UIDs from the snapshot to click/fill elements.
- **Always check console errors.** Even if the page looks fine visually, console errors indicate real problems.
- **Test the happy path first, then edge cases.** Verify basic functionality works before testing error states.
- **Don't skip mobile.** Resize and check every time. Layout bugs on mobile are the most common miss.
- **Report with evidence.** Include screenshot references and console output, not just "looks wrong."
