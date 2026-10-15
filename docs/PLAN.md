# tabstop-ui: plan

Accessible React + TypeScript component library. Tailwind, Storybook, deployed to GitHub Pages.

## Scope
- Tier 1 (full depth): Button, Input, Modal, Dropdown, Tabs
- Tier 2 (lighter coverage, stated in README): Toast, Tooltip

## Decisions
1. Tier 1 components each need: story, keyboard test, ARIA test, zero a11y addon violations.
2. A11y verification: Storybook a11y addon (axe-core) plus RTL interaction tests asserting Tab/Shift+Tab/Enter/Esc and focus return. No unverifiable manual-testing claims.
3. Visual regression: Storybook test runner and Playwright. Chromatic is a later, additive option.
4. CI gating: deploy runs only after CI succeeds (workflow_run). Same pattern as unified-api-ingester.
5. TypeScript: string-literal unions for variants, required onClose on Modal, CVA for variants, prop types exported from index.ts.

## Out of scope
Theming beyond Tailwind config, RTL/i18n, server-side rendering.

## Known work from audit of earlier draft
- Modal: focus return captures the wrong element; close icon SVG path invalid; focus trap visibility check breaks under jsdom.
- No Vitest/RTL setup yet; CI is a single storybook.yml.
- Playwright screenshots made locally may differ on the CI runner.
