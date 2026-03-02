---
description: Generate Playwright test specs from a test plan or feature description
---

# Create Tests

You are tasked with generating Playwright end-to-end test specs (`.spec.js` files) for the Slate application. Tests should be practical, runnable, and match the actual UI.

## Initial Setup

When invoked:
1. **Determine what to test**:
   - If a plan path is provided as argument, read it and extract testable phases/criteria
   - If a feature description is provided, use that as the basis
   - If no argument, ask the user what they want tests for

2. **Understand current state**:
   - Read `playwright.config.js` to understand the test configuration
   - Read existing tests in `tests/` to avoid duplicates and follow established patterns
   - Read relevant templates in `templates/` and `templates/partials/` to understand actual DOM structure (selectors, IDs, classes)
   - Read `server.py` to understand routes, form fields, and HTMX interactions

## Research Phase

Before writing any tests, you MUST understand the actual UI:

1. **Read the templates** that correspond to the features being tested:
   - `templates/base.html` — main page layout, sidebar, panel structure
   - `templates/respond.html` — Respond page layout
   - `templates/partials/` — all HTMX response fragments
   - `static/app.js` — JS interactions (slider sync, hamburger toggle)

2. **Read `server.py`** to understand:
   - Route handlers and their form field names
   - HTMX target IDs and swap strategies
   - Authentication setup
   - Response patterns

3. **Read existing test files** to follow conventions:
   - Import style, describe/test structure
   - Selector patterns already in use
   - Timeout values and wait strategies
   - How auth is handled

## Writing Tests

### File Naming
- Place tests in `tests/` directory
- Name files descriptively: `tests/<feature>.spec.js`
- If adding to an existing test file, edit it rather than creating a new one

### Test Structure
```javascript
// @ts-check
const { test, expect } = require('@playwright/test');

test.describe('Feature Name', () => {
  test('descriptive test name', async ({ page }) => {
    // Arrange
    await page.goto('/route');

    // Act
    await page.locator('#actual-selector').fill('value');
    await page.locator('#actual-button').click();

    // Assert
    await expect(page.locator('#actual-result')).toBeVisible({ timeout: 10_000 });
  });
});
```

### Key Patterns

1. **Use real selectors** — always verify selectors against actual templates. Never guess IDs or class names.

2. **Auth is handled by config** — `playwright.config.js` has `httpCredentials` set. Don't add auth to individual tests unless testing auth failure (use `browser.newContext` with wrong creds for that).

3. **HTMX waits** — HTMX swaps content asynchronously. Use `await expect(locator).toBeVisible({ timeout: N })` rather than `waitForTimeout`.

4. **Handle empty corpus** — many features depend on having data. Either:
   - Save test data first within the test
   - Use conditional checks: `if (await locator.count() > 0) { ... }`
   - Mark data-dependent tests clearly

5. **Timeouts** — LLM-dependent operations (extract, assemble, generate) need long timeouts (30-60s). Pure UI operations need short ones (5-10s).

6. **Mobile tests** — use `browser.newContext({ viewport: { width: 375, height: 812 }, httpCredentials: {...} })`.

7. **Don't use `waitForTimeout` for assertions** — prefer `expect().toBeVisible()` with timeout. Only use `waitForTimeout` for debounce delays that have no observable DOM change.

### Selector Reference

Always verify against templates, but common selectors include:
- Write page: `#note-area`, `#save-btn`, `#panel`, `#manual-search`, `#min-sim`
- Respond page: `#input-text`, `#url-input`, `#url-form`, `#assemble-form`, `#context-panel`, `#generate-btn`, `#respond-result`
- Sidebar: `#sidebar`, `#hamburger-open`, `.stat-box`, `.stat-label`
- Partials: `.save-confirm`, `.save-summary`, `.browse-detail`, `.card`, `.empty-hint`

## Output

1. Write the test file(s) to `tests/`
2. Present a summary of what was generated:
   - Number of test suites (describe blocks)
   - Number of individual tests
   - Which features/phases are covered
   - Any features that couldn't be automated (and why)
3. Run `npx playwright test --list` to verify the tests are parseable
4. Suggest running with `npx playwright test` (requires server to be running)

## Important Guidelines

- **Never guess selectors** — read the templates first
- **Keep tests independent** — each test should work in isolation (except where data setup is sequential by design)
- **Prefer fewer, comprehensive tests** over many tiny ones — each test has startup overhead
- **Test the user's flow**, not implementation details
- **Skip visual-only tests** — Playwright headless can't reliably test CSS layout; focus on functional correctness
