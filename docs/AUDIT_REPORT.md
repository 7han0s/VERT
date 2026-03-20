# VERT Codebase Audit & Triage Report

**Authors**: [7han0s](https://github.com/7han0s) x [Jules](https://github.com/jules-ai)

This report provides a high-level overview of the VERT codebase, highlighting potential issues, security concerns, code quality flags, and actionable optimizations. Additionally, we have triaged the top open issues from the upstream repository.

---

## 1. Upstream Issue Triage

The following are the top relevant open issues from [VERT-sh/VERT/issues](https://github.com/VERT-sh/VERT/issues).

### High Priority / Critical Functionality
*   **#241**: [Bug/Feature Request] Split %date% template into separate Date and Datetime tags *(Planned for immediate fix)*
*   **#221**: latest image fails to convert .webm → .mp4 on Windows 11, but pinned SHA works
*   **#216**: Video file fails to dowload

### Feature Requests
*   **#238**: [Feature Request] add scaling options when converting from SVG
*   **#230**: [Feature Request] PWA Support
*   **#217**: [Feature Request]: PDN Support
*   **#236**: Convert ACSM
*   **#235**: M3u/M3u8 to json converter???

### Optimization & Chore
*   **#237**: migrate to tailwindcss v4
*   **#240**: Question: Is sending user data/IP to ipinfo still needed?
*   **#214**: Takes 50MB of data by just accessing the page
*   **#219**: SVG image cropped when converted

---

## 2. Codebase Audit

### A. Security Issues & Vulnerabilities
The `npm audit` command revealed several moderate and high-severity issues in dependencies:
*   **High (kysely)**: SQL Injection via unsanitized JSON path keys when ignoring/silencing compilation errors or using `Kysely<any>`. Affects `@inlang/sdk` -> `@lix-js/sdk`.
*   **Moderate (esbuild)**: Enables any website to send any requests to the development server and read the response. Affects `vite`.
*   **Low (cookie)**: Accepts cookie name, path, and domain with out of bounds characters. Affects `@sveltejs/kit`.
*   **XSS Vectors in Svelte Files**: The linter flagged multiple uses of `{@html}` tags across various components (e.g., `VertdErrorDetails.svelte`, `Privacy.svelte`, `Credits.svelte`, `About.svelte`). These are potential Cross-Site Scripting (XSS) vulnerabilities if the injected HTML contains unsanitized user input. It's recommended to strictly use the `sanitize-html` library before rendering raw HTML strings.

### B. Logic & Code Quality Flags
*   **Svelte 5 Updates Needed**:
    *   In `Dialog.svelte`, `VertdError.svelte`, and `Toast.svelte`, `$props()` is used without destructuring, leading to Svelte compilation warnings about `customElement.props`.
    *   Variables derived from `$props()` are referenced locally outside of closures (e.g., in `Dialog.svelte`), only capturing initial values instead of remaining reactive.
*   **Unused Variables / Dead Code**:
    *   Various unused variables across components such as `category` in `FormatDropdown.svelte`, `isUp` in `Dropdown.svelte`, `e` in `Uploader.svelte`, and `sanitize` in `Privacy.svelte`.
*   **Typescript `any` Usage**:
    *   Multiple instances of `any` types (e.g., `FormatDropdown.svelte`, `VertdError.svelte`, `Toast.svelte`). Explicit typing is recommended to retain TypeScript's safety features.
*   **Global/Undefined Variables**:
    *   `__COMMIT_HASH__` is used in `Footer.svelte` but flagged as undefined by the linter, suggesting a potential Vite/Rollup replacement issue or missing global type definition.
    *   `NodeJS` is referenced in `Tooltip.svelte` and `Sponsors.svelte` but is undefined in the browser context.

### C. Clean-up & Optimizations
*   **Dependency Management**: There was an `ERESOLVE` conflict with `@stripe/stripe-js` (`^8.5.2` vs `^3 || ^4` required by `svelte-stripe`). A legacy peer deps install was required. The `svelte-stripe` dependency should be updated or replaced to match current Stripe versions.
*   **Prettier Formatting**: Running Prettier reported formatting issues in 36 files. Integrating Prettier and ESLint more strictly into a pre-commit hook (like Husky) would maintain a consistent code style.
*   **Data Usage**: As noted in issue #214, the initial page load consumes significant data (~50MB), likely due to aggressive loading or pre-caching of WASM binaries for converters (ffmpeg, imagemagick). Lazy-loading these WASM dependencies only when their specific conversion type is selected would drastically reduce initial load time.

---

*End of Report.*
