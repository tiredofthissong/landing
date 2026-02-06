# Test Coverage Analysis

## Current State

This project has **zero test coverage**. There is no test framework, no `package.json`, no test configuration, and no test files. All code lives in a single `index.html` (502 lines) containing inline CSS and JavaScript, deployed as a static site via GitHub Pages.

---

## Testable Areas

Despite being a simple static landing page, there are several meaningful areas that warrant testing. Below is a breakdown by category, ordered by priority.

### 1. Contact Form Submission Logic (High Priority)

**What it does:** The inline JavaScript (`index.html:442-487`) handles form submission via the Formspree API, manages loading/success/error UI states, and resets the form on success.

**What should be tested:**

- **Successful submission:** When the Formspree API returns `response.ok`, the form should reset, the status element should display a success message with class `success`, and the button should return to its default state.
- **Failed submission (non-ok response):** When the API returns a non-ok response, the status element should display an error message with class `error`, and the button should return to its default state.
- **Network error:** When `fetch` throws (e.g., network failure), the error state should be shown identically to a non-ok response.
- **Loading state:** While the request is in flight, the submit button should have the `loading` class and display "Sending..." text.
- **Button reset in all paths:** The `finally` block should always restore the button text to "Send Message" and remove the `loading` class, regardless of success or failure.
- **FormData construction:** The form data sent to Formspree should include `name`, `email`, and `message` fields with correct values.
- **Correct headers:** The fetch request should include `Accept: application/json`.

**Recommended approach:** Extract the form logic into a testable module. Use Jest or Vitest with jsdom to unit test the form handler by mocking `fetch`.

### 2. Smooth Scroll Navigation (Medium Priority)

**What it does:** The code at `index.html:489-498` attaches click handlers to all anchor links with `href` starting with `#`, calling `scrollIntoView` on the target element.

**What should be tested:**

- **`preventDefault` is called** on the click event to suppress default anchor behavior.
- **`scrollIntoView`** is called with `{ behavior: 'smooth', block: 'start' }` on the correct target element.
- **Missing target handling:** If `document.querySelector` returns `null` for the href value, `scrollIntoView` should not be called (no runtime error).

**Recommended approach:** Unit test with jsdom, mocking `scrollIntoView` on target elements.

### 3. HTML Structure & Accessibility (High Priority)

**What should be tested:**

- **Semantic structure:** Page has exactly one `<h1>`, form inputs have associated `<label>` elements with matching `for`/`id` attributes.
- **Required fields:** All three form inputs (`name`, `email`, `message`) have the `required` attribute.
- **Email input type:** The email field uses `type="email"` for built-in validation.
- **Alt text:** All images (`headshot.jpg` and 5 company logos) have non-empty `alt` attributes.
- **Language attribute:** The `<html>` element has `lang="en"`.
- **Viewport meta tag:** Present and correctly configured for mobile.
- **External links:** The LinkedIn link has `target="_blank"` (and should ideally have `rel="noopener noreferrer"` for security).

**Recommended approach:** Use an HTML parser (e.g., cheerio or jsdom) to validate the DOM structure in unit tests, or use an accessibility testing tool like axe-core or pa11y.

### 4. Responsive Design (Medium Priority)

**What should be tested:**

- **Mobile breakpoint at 600px:** The companies grid switches from 5 columns to 2 columns.
- **Fifth logo centering:** On mobile, the 5th company logo spans the full grid width and is centered at 50% max-width.
- **Container padding reduction:** Padding changes from `80px 30px` to `50px 20px` on mobile.
- **Profile photo size reduction:** 180px to 150px on mobile.
- **Heading font size reduction:** 2.5rem to 2rem on mobile.

**Recommended approach:** Use Playwright or Cypress to test at different viewport sizes and assert computed styles or visual snapshots.

### 5. Visual Regression (Low-Medium Priority)

**What should be tested:**

- **Full-page screenshots** at desktop (1280px) and mobile (375px) widths to catch unintended style regressions.
- **Company logo grayscale filter:** Logos should render in grayscale by default and full color on hover.
- **Button hover states:** LinkedIn button and submit button should show correct hover styling.

**Recommended approach:** Playwright visual comparison tests with stored baseline screenshots.

### 6. External Integrations (Low Priority)

**What should be tested:**

- **Google Analytics script tag:** The gtag.js script is present with the correct tracking ID (`G-BFHZ89ES9Z`).
- **Formspree endpoint:** The form action URL (`https://formspree.io/f/mreazrvv`) is correct.
- **LinkedIn URL:** The profile link points to the correct LinkedIn profile.
- **CNAME file:** Contains the expected domain (`breaux.is`).

**Recommended approach:** Simple unit tests that parse the HTML and assert on attribute values, or a smoke test that checks the external URLs return 200 status codes.

### 7. Cross-Browser Compatibility (Low Priority)

**What should be tested:**

- Page renders correctly in Chrome, Firefox, Safari, and Edge.
- CSS features used (grid, border-radius, box-shadow, filter, object-fit) are supported across target browsers.
- `fetch` API availability (relevant for very old browsers, though not a concern for modern ones).

**Recommended approach:** Playwright multi-browser test matrix.

---

## Recommended Test Infrastructure Setup

To go from zero to a working test suite, the following steps are needed:

### Step 1: Initialize the project

```bash
npm init -y
```

### Step 2: Install test dependencies

```bash
# Unit testing
npm install --save-dev vitest jsdom @testing-library/dom @testing-library/jest-dom

# E2E / visual testing
npm install --save-dev playwright @playwright/test

# Accessibility testing
npm install --save-dev axe-core

# HTML parsing (for structural tests)
npm install --save-dev cheerio
```

### Step 3: Refactor for testability

Extract the inline JavaScript from `index.html` into a separate file (e.g., `main.js`) so that functions can be imported and tested independently.

### Step 4: Create test directory structure

```
tests/
├── unit/
│   ├── form-submission.test.js
│   └── smooth-scroll.test.js
├── integration/
│   ├── accessibility.test.js
│   └── html-structure.test.js
└── e2e/
    ├── responsive.spec.js
    └── visual-regression.spec.js
```

### Step 5: Add test scripts to package.json

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:e2e": "playwright test",
    "test:coverage": "vitest run --coverage"
  }
}
```

---

## Priority Summary

| Area | Priority | Effort | Impact |
|------|----------|--------|--------|
| Contact form logic | High | Low | Prevents broken form submissions |
| HTML structure & accessibility | High | Low | Ensures usability and SEO |
| Smooth scroll navigation | Medium | Low | Prevents broken navigation |
| Responsive design | Medium | Medium | Prevents mobile layout issues |
| Visual regression | Low-Medium | Medium | Catches style drift |
| External integrations | Low | Low | Catches broken links/configs |
| Cross-browser compatibility | Low | High | Broad coverage, diminishing returns |

The highest-value starting point is **unit tests for the contact form logic** and **structural/accessibility checks on the HTML**. These are low-effort, high-impact tests that cover the most critical user-facing functionality.
