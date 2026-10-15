# tabstop-ui: audit of the a11y-ui draft and the copied config

Date: 2026-10-15. Scope: dependency compatibility (Part 1), a read-only source audit of `~/Git Projects/a11y-ui` and the config files copied into this repo (Part 2), and reproduction of the bugs listed in `docs/PLAN.md` (Part 3).

## How to read this document

Every finding has a severity and an evidence tag.

- **Severity.** `high` blocks a locked decision in `docs/PLAN.md` or breaks a user-facing behaviour. `med` is a real defect with a workaround or a limited blast radius. `low` is cleanup.
- **Evidence.**
  - `VERIFIED` means I ran something and observed the result.
  - `READ` means the finding comes from reading the source or config, without running it.
  - `UNVERIFIED` means it comes from general knowledge of the tool and I did not confirm it here. Check these before relying on them.

Environment: Node v22.23.1, npm 10.9.8. The a11y-ui `node_modules` was already installed, with vite 5.4.21, eslint 8.57.1, typescript 5.9.3, storybook 8.6.18, tailwindcss 3.4.19 and @playwright/test 1.62.1.

The config files copied into tabstop-ui are byte-identical to the a11y-ui originals: `.eslintrc.cjs`, `index.html`, `package.json`, `postcss.config.js`, `tailwind.config.js`, `tsconfig*.json`, `vite.config.ts` and `.storybook/*` (checked with `diff`, VERIFIED). Every config finding below therefore applies to both repos.

### Side effects on the draft

- **Lint and tsc were run in place.** Running `tsc -p tsconfig.node.json --noEmit` created `a11y-ui/tsconfig.node.tsbuildinfo` because `composite: true` is set. That file did not exist before the run, so I deleted it. Nothing else in a11y-ui changed. I checked this with `find -newer` and with sha256 of `dist/*` taken before and after.
- **The build did not use `npm run build`.** That script writes into `a11y-ui/dist`, which would have broken the "do not modify" rule. Instead I ran `tsc -p tsconfig.json` (with `noEmit`) and then `vite build --outDir <scratchpad>`.
- **Part 3 ran in a scratch directory.** All reproduction work happened in `/tmp/tabstop-audit-scratch`, on copies of the draft files. The sha256 of each copy matched its original.

---

## Part 1: dependency compatibility

Data comes from `npm view <pkg> version peerDependencies peerDependenciesMeta engines` on 2026-10-15 (VERIFIED). "(optional)" means `peerDependenciesMeta` marks that peer `optional: true`. "No peer declared" means the package states nothing about that host. It does not mean the package was tested against it.

| Package | Latest | Relevant peerDependencies | engines.node | vite 8 | eslint 10 | storybook 10.6 |
|---|---|---|---|---|---|---|
| storybook | 10.6.2 | `@types/react` 16–19, prettier, vite-plus (all optional) | none declared | n/a | n/a | yes |
| vite | 8.3.4 | `@types/node ^20.19.0 \|\| >=22.12.0`, esbuild `^0.27 \|\| ^0.28`, plus tsx, jiti, less, sass, sass-embedded, stylus, sugarss, terser, yaml, @vitejs/devtools (all optional) | `^20.19.0 \|\| >=22.12.0` | yes | n/a | n/a |
| tailwindcss | 4.3.3 | none | none | n/a | n/a | n/a |
| eslint | 10.12.0 | jiti (optional) | `^20.19.0 \|\| ^22.13.0 \|\| >=24` | n/a | yes | n/a |
| @storybook/react-vite | 10.6.2 | `vite ^5 \|\| ^6 \|\| ^7 \|\| ^8`, `storybook ^10.6.2`, `typescript >= 4.9.x`, react 16.8–19 | none | **yes** | n/a | **yes** |
| @vitejs/plugin-react | 6.1.2 | `vite ^8.0.0` (required); `oxc-transform-react ^0.152.0`, `@rolldown/plugin-babel ^0.1.7 \|\| ^0.2.0`, `babel-plugin-react-compiler ^1.0.0` (optional) | `^20.19.0 \|\| >=22.12.0` | **yes (8 only)** | n/a | n/a |
| @tailwindcss/vite | 4.3.3 | `vite ^5.2.0 \|\| ^6 \|\| ^7 \|\| ^8` | none | **yes** | n/a | n/a |
| eslint-plugin-jsx-a11y | 6.10.2 (published 2024-10-26) | `eslint ^3 … ^9` | `>=4.0` | n/a | **no, capped at ^9** | n/a |
| eslint-plugin-react-hooks | 7.1.1 | `eslint ^3 … ^9 \|\| ^10` | `>=18` | n/a | **yes** | n/a |
| @typescript-eslint/parser, /eslint-plugin, /utils, typescript-eslint | 8.71.2 | `eslint ^8.57.0 \|\| ^9 \|\| ^10`, `typescript >=4.8.4 <6.1.0` | `^18.18.0 \|\| ^20.9.0 \|\| >=21.1.0` | n/a | **yes** | n/a |
| vitest | 5.0.4 | `vite ^6.4.0 \|\| ^7 \|\| ^8` (required; vite is a peer only, not a dependency); `@types/node ^22 \|\| >=24`, jsdom, happy-dom, @vitest/browser-playwright and others (optional) | `^22.12.0 \|\| ^24 \|\| >=26` | **yes** | n/a | n/a |
| @storybook/addon-a11y | 10.6.2 | `storybook ^10.6.2` | none | n/a | n/a | **yes** |
| @storybook/addon-vitest | 10.6.2 | `storybook ^10.6.2` (required); `vitest ^3 \|\| ^4 \|\| ^5`, `@vitest/runner ^3–^5`, `@vitest/browser ^3–^5`, `@vitest/browser-playwright ^4 \|\| ^5` (all marked optional in metadata) | none | via vitest | n/a | **yes** |

Supporting packages I also checked:

| Package | Latest | Notes |
|---|---|---|
| typescript | 7.0.2 | engines `>=16.20.0` |
| jsdom | 30.1.3 | engines `^22.22.2 \|\| ^24.15.0 \|\| >=26` |
| @testing-library/react | 16.3.3 | peers react 18/19, `@testing-library/dom ^10` |
| @testing-library/dom | 10.4.2 | |
| @testing-library/user-event | 14.6.8 | |
| @testing-library/jest-dom | 7.0.1 | peer `vitest >=0.32`; engines `node >=22` |
| @vitest/browser, @vitest/browser-playwright | 5.0.4 | peer `vitest 5.0.4` exactly |
| eslint-plugin-react-refresh | 0.5.7 | peer `eslint ^9 \|\| ^10` |
| eslint-plugin-storybook | 10.6.2 | peer `eslint >=8` |
| class-variance-authority | 0.7.1 | depends on `clsx ^2.1.1` |
| @storybook/addon-essentials, @storybook/addon-interactions | latest 8.6.14 | No 9.x or 10.x stable release; both are referenced in `.storybook/main.ts` |

### Newest mutually compatible set

The latest release works everywhere except in two places.

1. **eslint must step back to 9.39.5.** Severity: med. Evidence: VERIFIED. eslint-plugin-jsx-a11y 6.10.2 is the newest release and declares `eslint ^9` at most. jsx-a11y is central to this project, so 9.39.5 is the newest strictly compatible eslint. Its engines are `^18.18.0 || ^20.9.0 || >=21.1.0` (VERIFIED). The alternative is eslint 10.12.0 plus an npm `overrides` entry that forces jsx-a11y's peer. That would be an unsupported combination, and I have not tested whether jsx-a11y runs correctly under eslint 10 (UNVERIFIED).
2. **typescript must step back to 6.0.3.** Severity: low. Evidence: VERIFIED. typescript-eslint 8.71.2 declares `typescript <6.1.0`, which rules out 7.0.2.

| Package | Version | Note |
|---|---|---|
| node | 22.23.1 (local) | Meets every engines range above, including jsdom's `^22.22.2` |
| storybook, @storybook/react-vite, @storybook/addon-a11y, @storybook/addon-vitest, eslint-plugin-storybook | 10.6.2 | |
| vite | 8.3.4 | |
| @vitejs/plugin-react | 6.1.2 | Requires vite 8. Use 5.2.0 if you ever need vite 7 |
| tailwindcss, @tailwindcss/vite | 4.3.3 | |
| vitest, @vitest/browser, @vitest/browser-playwright | 5.0.4 | addon-vitest marks all of these as optional peers. In practice the addon needs vitest to do anything, and it runs stories in browser mode, so plan on all three (UNVERIFIED) |
| jsdom | 30.1.3 | |
| @testing-library/react / dom / user-event / jest-dom | 16.3.3 / 10.4.2 / 14.6.8 / 7.0.1 | |
| **eslint** | **9.39.5** | Step back from 10.12.0 because of jsx-a11y |
| eslint-plugin-jsx-a11y | 6.10.2 | |
| eslint-plugin-react-hooks | 7.1.1 | |
| typescript-eslint (+ parser and eslint-plugin) | 8.71.2 | |
| eslint-plugin-react-refresh | 0.5.7 | Accepts eslint 9 |
| **typescript** | **6.0.3** | Step back from 7.0.2 because of typescript-eslint |
| @types/node | 22.x (22.20.5) | Satisfies both the vite and the vitest peer ranges |
| class-variance-authority, clsx | 0.7.1, 2.1.1 | |
| react, react-dom | 18.3.1 or 19.3.1 | Every peer above accepts both, so this is a free choice |

Packages in the copied `package.json` that should be removed rather than bumped:

- `@storybook/addon-essentials` and `@storybook/addon-interactions` have no 10.x release. Remove them from `package.json` and `.storybook/main.ts`. Severity: med. Evidence: VERIFIED, from dist-tags.
- `autoprefixer` and `postcss` are probably redundant with `@tailwindcss/vite`. Severity: low. Evidence: UNVERIFIED.

The copied `package.json` pins storybook ^8, vite ^5, tailwind ^3 and eslint ^8. Every one of those is a major-version jump to reach the set above. Severity: med. Evidence: READ.

### Migration consequences outside npm metadata (UNVERIFIED)

`npm view` cannot show these. Each one needs confirming against the upstream migration guide. All are evidence UNVERIFIED.

- **eslint 9 drops eslintrc by default.** Severity: med. `.eslintrc.cjs` needs converting to `eslint.config.js` (flat config), and eslint 10 drops eslintrc completely.
- **Tailwind 4 does not auto-load `tailwind.config.js`.** Severity: high, because the tokens would silently disappear. Tokens either move to `@theme` in CSS or get loaded with `@config`. The `postcss.config.js` `tailwindcss` plugin entry no longer works, so use `@tailwindcss/vite` instead.
- **Storybook 9 and 10 fold essentials and interactions into core and are ESM-only.** Severity: med. `docs.autodocs: 'tag'` and the `@storybook/react` type import in `preview.ts` should move to `@storybook/react-vite`.
- **Vite 8 bundles with Rolldown.** Severity: low. `build.rollupOptions` in `vite.config.ts` may need to become `build.rolldownOptions`.

---

## Part 3: reproduction of the claimed bugs

Setup: `/tmp/tabstop-audit-scratch`. I installed react@18.3.1, react-dom@18.3.1, vitest@5.0.4, vite@8.3.4, jsdom@30.1.3, @testing-library/react@16.3.3, @testing-library/dom@10.4.2 and @testing-library/user-event@14.6.8 there. user-event was added because `fireEvent.click` does not move focus. Copies of `useFocusTrap.ts`, `useFocusReturn.ts`, `cn.ts`, `Modal.tsx`, `Modal.types.ts`, `Input.tsx` and `Input.types.ts` were used unmodified. The one exception is a control copy, `ModalSwapped.tsx`, which differs only by the order of the two hook calls.

### Lint, tsc and build in a11y-ui

| Step | Result | Evidence |
|---|---|---|
| `npm run lint` | **Fails, exit 2.** `ESLint couldn't find the plugin "eslint-plugin-react-refresh".` The plugin is listed in `.eslintrc.cjs` `plugins` but is not in `package.json` or `node_modules`. | VERIFIED |
| Lint with a scratch copy of the config (`react-refresh` dropped from `plugins`, parser path made absolute, run with `--no-eslintrc -c <copy> --resolve-plugins-relative-to .`) | **Fails, exit 1, 2 errors.** `Modal.stories.tsx:25:33` and `:58:33`: `React Hook "useState" is called in function "render" that is neither a React function component nor a custom React Hook function` (`react-hooks/rules-of-hooks`). | VERIFIED |
| `tsc -p tsconfig.json` (the first half of `npm run build`) | Passes, exit 0. | VERIFIED |
| `tsc -p tsconfig.node.json --noEmit` | Passes, exit 0. | VERIFIED |
| `vite build --outDir <scratch>` (the second half of `npm run build`, redirected) | Passes. Produces `a11y-ui.es.js` (91.48 kB) and `a11y-ui.umd.js` (41.32 kB). The output contains the bundled `react-jsx-runtime` code (see S-12). | VERIFIED |
| Tailwind CLI on `src/index.css` | **Fails.** `CssSyntaxError: … index.css:5:1: '@layer base' is used but no matching '@tailwind base' directive is present.` The file uses `@tailwindcss base;` where it should use `@tailwind base;`. | VERIFIED |

### (a) Does `offsetParent` return null in jsdom, so that useFocusTrap filters out every element?

**Yes.** VERIFIED. Severity: high for the test plan. In real browsers this code path works, so end users are not affected.

- A visible, attached `<button>` reports `offsetParent = null` in jsdom 30.1.3.
- With the original Modal, focus lands on the dialog `DIV[role=dialog]` after opening, not on the first focusable element. That happens because `getFocusableElements()` returns `[]` and the `container.focus()` fallback runs.
- Tab and Shift+Tab both leave focus on `DIV[dialog]`. With an empty list, the keydown handler calls `preventDefault()` and returns.
- Control: with `HTMLElement.prototype.offsetParent` stubbed to return `parentNode`, the trap behaves correctly. The sequence is `Close dialog -> field -> Cancel -> Close dialog`, which wraps.

Consequence: none of the keyboard tests that decision 2 requires can pass under jsdom unless the visibility check changes or the tests run in a real browser (Storybook play functions or vitest browser mode). One option is `el.getClientRects().length > 0` combined with a jsdom fallback. Another is to filter on `hidden`, `inert` and `display:none` ancestors only.

### (b) With Modal's hook order, does focus return to the trigger or to body?

**To body.** VERIFIED. Severity: high. This is user-facing and fails decision 2's focus-return requirement.

The trigger was focused and clicked with user-event, and the modal was then closed with Escape.

| Hook order | offsetParent | Focus after open | Focus after close |
|---|---|---|---|
| original (trap, then return) | jsdom (null) | `DIV[dialog]` | **`BODY`** |
| swapped (return, then trap) | jsdom (null) | `DIV[dialog]` | `BUTTON[Trigger]` |
| original (trap, then return) | stubbed (browser-like) | `BUTTON[Close dialog]` | **`BODY`** |
| swapped (return, then trap) | stubbed (browser-like) | `BUTTON[Close dialog]` | `BUTTON[Trigger]` |

Cause, confirmed by the control:

1. `useFocusTrap` (Modal.tsx:22) is called before `useFocusReturn` (Modal.tsx:23), and effects run in declaration order.
2. The trap's effect therefore moves focus into the dialog first.
3. `useFocusReturn` then records `document.activeElement`, which is now an element inside the dialog.
4. On close, it calls `.focus()` on that detached node, which does nothing, so focus falls back to `body`.

Swapping the two calls fixes it under both conditions. A sturdier fix is to capture the previously focused element during render, or in a layout effect that runs before the trap moves focus, and to restore it in the effect's cleanup.

### (c) Is the close-icon SVG path invalid?

**Yes.** VERIFIED in headless Chromium 151.0.7922.34, using the Playwright 1.62.1 install from a11y-ui. Severity: med.

The close icon uses `d="6 18L18 6M6 6l12 12"` (Modal.tsx:108).

- **Original path.** Chromium logs `Error: <path> attribute d: Expected moveto path command ('M' or 'm'), "6 18L18 6M6 6l12…".` `getTotalLength()` returns `0` and `getBBox()` returns `[0,0,0,0]`. The element screenshot is a blank 113-byte PNG.
- **Control with a leading `M`** (`M6 18L18 6M6 6l12 12`). `getTotalLength()` returns `33.94`, `getBBox()` returns `[6,6,12,12]`, and the screenshot is 338 bytes with the X drawn.

The button keeps `aria-label="Close dialog"`, so screen-reader users still get a name. Sighted users see an empty 32px button in the header. Escape and backdrop click still close the dialog.

### (d) WCAG contrast of the tokens in `tailwind.config.js`

Computed with the WCAG 2.x relative-luminance formula (VERIFIED). neutral-50 is `#f8fafc`.

| Token | Hex | vs white | vs neutral-50 | Comment claims | Difference |
|---|---|---|---|---|---|
| brand-50 | `#eff6ff` | 1.09 | 1.04 | - | - |
| brand-100 | `#dbeafe` | 1.22 | 1.17 | - | - |
| brand-200 | `#bfdbfe` | 1.42 | 1.36 | - | - |
| brand-600 | `#1d4ed8` | **6.70** | 6.41 | 7.1 ("AA & AAA Pass") | -0.40 |
| brand-700 | `#1e40af` | 8.72 | 8.34 | 9.8 | -1.08 |
| brand-800 | `#1e3a8a` | 10.36 | 9.90 | 12.6 | -2.24 |
| brand-900 | `#172554` | 14.69 | 14.04 | - | - |
| neutral-50 | `#f8fafc` | 1.05 | 1.00 | - | - |
| neutral-100 | `#f1f5f9` | 1.10 | 1.05 | - | - |
| neutral-200 | `#e2e8f0` | 1.23 | 1.18 | - | - |
| neutral-300 | `#cbd5e1` | **1.48** | 1.42 | - | - |
| neutral-400 | `#94a3b8` | 2.56 | 2.45 | - | - |
| neutral-600 | `#475569` | 7.58 | 7.24 | 7.0 | +0.58 |
| neutral-700 | `#334155` | 10.35 | 9.90 | 9.7 | +0.65 |
| neutral-800 | `#1e293b` | 14.63 | 13.98 | 14.7 | -0.07 |
| neutral-900 | `#0f172a` | 17.85 | 17.06 | 19.4 | -1.55 |
| danger-50 | `#fef2f2` | 1.09 | 1.05 | - | - |
| danger-100 | `#fee2e2` | 1.22 | 1.17 | - | - |
| danger-600 | `#dc2626` | 4.83 | 4.62 | 4.6 | +0.23 |
| danger-700 | `#b91c1c` | **6.47** | 6.18 | 7.0 | -0.53 |
| danger-800 | `#991b1b` | 8.31 | 7.94 | 9.4 | -1.09 |
| success-50 | `#f0fdf4` | 1.05 | 1.00 | - | - |
| success-700 | `#15803d` | 5.02 | 4.79 | 5.2 | -0.18 |
| success-800 | `#166534` | 7.13 | 6.81 | 7.3 | -0.17 |

What this means, based on where each token is used:

- **The AAA claim for brand-600 is false** (see S-11; med, VERIFIED). brand-600 is 6.70:1 on white and 6.41:1 on neutral-50, so it passes AA (4.5:1) and fails AAA (7:1). The comment's "AAA Pass" is wrong. The same 6.70:1 applies to white text on the primary button.
- **danger-700 is overstated** (S-11; med, VERIFIED). It is 6.47:1, not 7.0:1, so it is also below AAA. It is the danger button background and the error border.
- **9 of the 12 annotated ratios are overstated** (S-11; med, VERIFIED), by up to 2.24 for brand-800. The other 3 (neutral-600, neutral-700, danger-600) are understated. The comments are not reliable and should be regenerated from a script.
- **Every text token at shade 600 and above passes AA** (no finding; VERIFIED) on both white and neutral-50. danger-600 is the closest, at 4.62:1 on neutral-50.
- **neutral-300 fails the non-text contrast requirement** (S-6; med, VERIFIED). It is the default Input border (Input.tsx:70) at 1.48:1, and WCAG 1.4.11 asks for 3:1 for the boundary that identifies a form control. See S-6.
- **neutral-400 (2.56:1)** would fail as text but is unused (S-10; low, VERIFIED). The 50–200 shades are background tints and are not meant for text.

---

## Part 2: source audit (Button, Input, Modal, hooks, config, CI)

### Per-component checklist against the plan's decisions

| | Button | Input | Modal |
|---|---|---|---|
| Story exists | Yes, 7 stories (READ) | Yes, 5 stories (READ) | Yes, 2 stories. Both fail lint (VERIFIED) and both render closed (READ) |
| Keyboard test | **No.** There are no unit or interaction tests anywhere; `tests/` holds only the Playwright screenshot spec (READ) | **No** (READ) | **No** (READ) |
| ARIA test | **No** (READ) | **No** (READ) | **No** (READ) |
| Zero a11y-addon violations | Not checkable: Storybook was not run (UNVERIFIED) | Not checkable (UNVERIFIED) | **Cannot be checked as configured.** See S-3 |
| String-literal union variants | Yes: `ButtonVariant`, `ButtonSize` (READ) | n/a | Partly. `role` and `size` are inline unions, not exported named types (READ) |
| `onClose` required | n/a | n/a | Yes, `onClose: () => void` (READ) |
| CVA for variants | **No.** Uses `Record<…, string>` maps with `cn()`; CVA is not a dependency (READ) | **No** (READ) | **No** (READ) |
| Prop types exported | From `src/index.ts`, yes (READ). No `.d.ts` reaches consumers (S-12) | Same | Same |
| `index.ts` per component | **No** (READ) | **No** (READ) | **No** (READ) |

### Findings

**S-1. No component tests and no test runner.** Severity: high. Evidence: READ.
There is no vitest or RTL config, no `*.test.tsx`, and no play functions in the stories. Decisions 1 and 2 are therefore entirely unmet for all three Tier 1 components that exist. Dropdown and Tabs do not exist yet.

**S-2. Modal focus-return and focus-trap bugs.** Severity: high. Evidence: VERIFIED.
See Part 3 (a) and (b).

**S-3. The Storybook a11y addon never sees Modal content.** Severity: high. Evidence: READ.
`.storybook/preview.ts:14` sets `a11y.element: '#storybook-root'`. Modal portals to `document.body` (Modal.tsx:126), and both Modal stories render with `isOpen` false until a button is clicked. axe is never run against the dialog, so decision 1's "zero a11y addon violations" would pass without testing anything.

**S-4. `src/index.css` does not compile.** Severity: high. Evidence: VERIFIED.
It uses `@tailwindcss base/components/utilities` where it should use `@tailwind …`. `.storybook/preview.ts` imports this file, so Storybook would load no Tailwind styles or fail on the CSS. Under Tailwind 4 the whole file becomes `@import "tailwindcss";` anyway.

**S-5. Lint is broken.** Severity: high. Evidence: VERIFIED.
- `eslint-plugin-react-refresh` is listed in `plugins` but not installed, and no rule from it is configured.
- Even after working around that, `Modal.stories.tsx` calls `useState` inside `render`. The fix is to extract a named component, such as `function ModalDemo(args) {…}`, and use `render: (args) => <ModalDemo {...args} />`.

**S-6. The Input border does not meet non-text contrast.** Severity: med. Evidence: VERIFIED (ratio), READ (usage).
`border-neutral-300` is 1.48:1 against the white input background (Input.tsx:66, :70), and WCAG 1.4.11 needs 3:1. `shadow-sm` does not add a reliable boundary. The disabled variant uses the same border.

**S-7. Input has a dangling `aria-describedby` reference.** Severity: med. Evidence: VERIFIED.
With both `helperText` and `errorMessage` set, `aria-describedby` is `:r0:-helper :r0:-error`, but only the `-error` element is rendered. The helper `<p>` is hidden when `hasError` (Input.tsx:31 vs :86). Push the helper id only when the helper is actually rendered.

**S-8. Input error messages are announced twice.** Severity: low. Evidence: READ.
The error `<p>` has `role="alert"` and is also referenced by `aria-describedby`. Screen readers announce it twice, and also announce it on first render in the `ErrorState` story. Pick one mechanism; `aria-describedby` alone, or a live region that only fires on change, is usual. `aria-required` is redundant with native `required` (Input.tsx:61).

**S-9. Tailwind classes that reference undefined tokens or missing plugins.** Severity: med. Evidence: VERIFIED by compiling the classes with a corrected `@tailwind` entry file in scratch.
- `active:bg-danger-900` (Button.tsx:34). `danger-900` is not in the palette, so no CSS is generated and the danger button gets no active-state colour.
- `animate-in fade-in zoom-in-95` (Modal.tsx:78). These come from the `tailwindcss-animate` plugin, which is not installed, so no CSS is generated.

**S-10. Hardcoded colours outside the token set.** Severity: low. Evidence: READ.
- `active:bg-black` (Button.tsx:31).
- `text-white` and `bg-white` (Button.tsx:30, 31, 34; Input.tsx:66; Modal.tsx:78) come from Tailwind's default palette, not the custom tokens.
- `tailwind.config.js:43-44` boxShadow repeats hex literals (`#ffffff`, `#1d4ed8`, `#b91c1c`) instead of referencing tokens. `shadow-focus` and `shadow-focus-danger` are never used.
- Defined but unused tokens: brand-50/100/200/900, neutral-400, danger-50/100/600, and all success tokens.

**S-11. Contrast comments in `tailwind.config.js` are inaccurate.** Severity: med. Evidence: VERIFIED.
See Part 3 (d). The "AAA Pass" claim on brand-600 is false.

**S-12. The library build does not produce a usable package.** Severity: med. Evidence: VERIFIED for the bundling, READ for the rest.
- `vite.config.ts` externalises only `react` and `react-dom`, so `react/jsx-runtime` is bundled into `a11y-ui.es.js` (VERIFIED in the build output).
- `react` and `react-dom` are in `dependencies` instead of `peerDependencies`.
- `package.json` has no `main`, `module`, `exports` or `types`.
- `tsc` runs with `noEmit`, so no `.d.ts` is produced. The "prop types exported from index.ts" part of decision 5 therefore never reaches consumers.
- `src/index.ts` does not import `index.css`, so no stylesheet is shipped.

**S-13. Button ARIA details.** Severity: low. Evidence: READ.
- `aria-disabled` duplicates native `disabled` (Button.tsx:47-48).
- `{...props}` is spread after the ARIA attributes (Button.tsx:59), so consumers can silently override `aria-busy` or `aria-pressed`.
- In the loading state, the spinner is `aria-hidden` and the label text is unchanged, so `aria-busy` is the only signal. Support for `aria-busy` on buttons is inconsistent across screen readers (UNVERIFIED). Consider visually hidden "Loading" text.

**S-14. Modal does not isolate the background.** Severity: med. Evidence: READ.
The dialog relies only on `aria-modal="true"`. Siblings of the portal are not given `inert` or `aria-hidden`, and some screen reader and browser pairs let virtual-cursor users read the background. The Escape handler sits on `document`, so its `stopPropagation()` does nothing, and stacked modals would all close on one Escape press.

**S-15. useFocusTrap selector gaps.** Severity: low. Evidence: READ.
- `[contenteditable]` matches `contenteditable="false"`.
- `iframe`, `object` and `embed` are included without a tabindex check.
- `summary`, `audio[controls]` and `video[controls]` are missing.
- `!el.hasAttribute('disabled')` is redundant with the selector.

**S-16. Story issues.** Severity: low. Evidence: READ.
- The Input `ErrorState` and `Disabled` stories pass `value` without `onChange`, which triggers React's read-only-field warning. Use `defaultValue`.
- The Modal stories spread `args` and then hardcode `title` and `description`, so those controls do nothing.

**S-17. CI does not implement decision 4 and cannot run the target stack.** Severity: high. Evidence: READ for the workflow, VERIFIED for the engine ranges.
- `.github/workflows/storybook.yml` deploys on every push to `main`, with no lint, typecheck, test or a11y job and no `workflow_run` gate.
- It uses `node-version: 20`. vitest 5.0.4 needs `^22.12.0`, jsdom 30.1.3 needs `^22.22.2`, and jest-dom 7.0.1 needs `>=22`.
- tabstop-ui has no `package-lock.json`, and the workflow runs `npm ci`, which needs one.

**S-18. The visual regression baseline does not exist.** Severity: low. Evidence: READ.
`tests/visual-regression.spec.ts` points at `tests/snapshots`, which does not exist, so the first run only creates baselines. Combined with the known local-vs-CI rendering drift, baselines should be generated on the CI image. The Modal test clicks the first `button` on the page, which works only because the trigger renders first.

**S-19. Copied config still names the old project.** Severity: low. Evidence: READ.
`package.json` `"name": "a11y-ui"`, vite `lib.name: 'A11yUI'` and `fileName: a11y-ui.*`, and the `index.html` title all still refer to the draft. `index.html` loads `/src/index.ts`, which renders nothing, so `npm run dev` shows a blank page.

**S-20. tsconfig details.** Severity: low. Evidence: VERIFIED for the tsbuildinfo, READ for the rest.
- `tsconfig.node.json` has `composite: true`, so even `--noEmit` runs write `tsconfig.node.tsbuildinfo`. That file is not in `.gitignore` (`*.tsbuildinfo`).
- Its `include` lists `.js` config files, but `allowJs` is not set, so they are not type-checked.
- `tsconfig.json` `include` covers `tests`, which mixes Playwright types into the app program.

**S-21. eslintrc format.** Severity: med. Evidence: UNVERIFIED.
`.eslintrc.cjs` is eslintrc format, which needs converting to flat config for eslint 9 or 10 (see Part 1). `plugin:storybook/recommended` is not configured.

### DPDP check

The components and stories collect no real data. There is no form submission, analytics, auth, storage or third-party call. The Input stories use placeholder emails only, so no DPDP items are triggered at this stage.

---

## Summary of claims in PLAN.md

| Claim in PLAN.md "Known work" | Result |
|---|---|
| Modal: focus return captures the wrong element | **Confirmed** (VERIFIED). Focus returns to `body`; hook order is the cause, shown with a control |
| Modal: close icon SVG path invalid | **Confirmed** (VERIFIED in Chromium). The path renders nothing |
| Modal: focus-trap visibility check breaks under jsdom | **Confirmed** (VERIFIED). `offsetParent` is null, so focus sticks on the container and Tab does nothing |
| No Vitest/RTL setup; CI is a single storybook.yml | **Confirmed** (READ). Also, CI uses Node 20, which the target test stack does not support |
| Playwright screenshots may differ on CI | Not tested. No baselines exist yet (READ) |

New issues not in PLAN.md, ranked:

1. `index.css` does not compile (S-4, high).
2. Lint is broken (S-5, high).
3. The a11y addon cannot see the Modal (S-3, high).
4. The Input border fails 1.4.11 (S-6, med).
5. Input has a dangling describedby reference (S-7, med).
6. Classes reference undefined tokens or missing plugins (S-9, med).
7. Contrast comments are wrong, including the false AAA claim (S-11, med).
8. The package build ships no types or CSS and bundles jsx-runtime (S-12, med).
9. Two required version step-backs: eslint 9.39.5 and typescript 6.0.3 (Part 1).
