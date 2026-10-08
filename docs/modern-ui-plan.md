# Proxmox VE modern interface contribution plan

Modernize the existing Proxmox VE desktop interface using shadcn-inspired design while retaining its ExtJS components, management logic, APIs, and familiar resource hierarchy. This is the recommended first approach: a modern appearance does not require rebuilding the interface in React or migrating it to Yew.

The work belongs in the existing `pve-manager` project, delivered through small reviewable contributions. Start with an optional appearance setting and a representative screen. Apply shared design rules across the interface, then improve individual layouts and accessibility where styling alone is insufficient.

## What can be reused from shadcn

The [shadcn theming guide](https://ui.shadcn.com/docs/theming) defines semantic tokens for colors, surfaces, borders, focus, and corner radii. That design approach can be translated into Proxmox CSS and Sass without installing React or Tailwind. Use consistent typography, spacing, neutral surfaces, clear primary actions, restrained shadows, and deliberate light/dark states.

The supplied React components cannot be used directly as ExtJS or Yew components. Recreating their appearance in ExtJS preserves existing widgets and event handlers. Porting their behavior to Yew is possible, but would be a separate component-development effort, with its own keyboard, focus, validation, and accessibility testing.

Shadcn supplies design conventions, not a formal accessibility certification. Adopting the look does not automatically supply its component behavior or prove the refreshed interface is accessible.

## Framework options

| Approach | What changes | Implication |
| --- | --- | --- |
| Existing ExtJS with shadcn-inspired design | Theme, selected layout settings, focused interaction improvements | Recommended first contribution; retains the existing behavior and build system |
| Vite + React + TypeScript + shadcn/ui | New application, components, authentication client, routing, and build integration | Feasible inside the same repository, but a substantial migration requiring upstream agreement |
| Rust/Yew with Proxmox components | New screens using upstream's newer UI toolkit | Aligned with upstream's direction; shadcn appearance can be adapted, but React components need reimplementation |

Yew is a modern component framework compiled to WebAssembly, and a more specialist choice than mainstream JavaScript frontend frameworks. Proxmox's [toolkit README and whitepaper](https://github.com/proxmox/proxmox-yew-widget-toolkit) explain its preference for sharing Rust code and reducing maintained languages. These considerations do not prevent useful improvements to the current ExtJS interface.

Vite is primarily a development/build tool. It is unnecessary for the proposed theme work. If a later framework replacement is agreed, use the existing API and `pveproxy`, package static browser assets, preserve classic access throughout migration, and establish a reproducible offline dependency pipeline before integrating the new build.

## Source baseline

The canonical source was inspected on 8 October 2026. `pve-manager` is pinned at `ac0c87dd37115e3046cbfa5756f2a02fc659e8cd`, dated 6 October. Its desktop directory contains **376 JavaScript files and 74,559 lines**, including comments and blank lines. These numbers describe source size, not a feature count or an effort estimate.

The [source inventory](modern-ui-source-inventory.json) records exact desktop paths and navigation anchors. The [feature matrix](modern-ui-feature-matrix.md) identifies the management areas that need regression coverage. Neither is a completed behavior audit.

The relevant reference repositories were also downloaded: `proxmox-widget-toolkit`, `proxmox-yew-widget-toolkit`, `pve-yew-mobile-gui`, and `proxmox-e2e-tests`. Their revisions are recorded in the inventory.

## Existing theme support

The current code already provides the integration points needed for this approach:

| Source | Existing role | Proposed use |
| --- | --- | --- |
| `www/index.html.tpl` | Loads ExtJS Crisp, PVE CSS, shared toolkit CSS, language, and selected theme | Conditionally add the modern stylesheet and an appearance class |
| `PVE/Service/pveproxy.pm` | Reads `PVEThemeCookie`, serves assets, chooses desktop/mobile/console templates | Validate a separate appearance preference and pass it to the desktop template |
| `www/css/ext6-pve.css` | Existing PVE adjustments to buttons, trees, icons, grids, and panels | Identify current selectors and conflicting assumptions |
| `www/css/Makefile` | Installs PVE stylesheets | Install the new stylesheet using the existing package path |
| `www/manager6/window/Settings.js` | PVE-specific appearance and interface preferences | Add the optional appearance control without exposing it to other products |
| `www/manager6/Workspace.js`, `panel/ConfigPanel.js` | Main workspace, toolbars, resource navigation | Apply selective native component configuration changes |
| Shared `src/proxmox-dark/scss/` | Existing dark-theme variables and component-specific overrides | Reference proven coverage of fields, grids, menus, windows, charts, and PVE details |
| Shared `src/proxmox-dark/Makefile` | Builds the dark stylesheet with `sassc` | Reuse the supported toolchain if modular Sass is needed |
| Shared `src/Utils.js`, `form/ThemeSelector.js`, `window/ThemeEdit.js` | Shared color theme choices and persistence | Retain existing color behavior; coordinate any future shared design changes |

Proxmox already styles the same ExtJS components centrally. This is direct evidence that a design refresh can be integrated without rebuilding the management application. ExtJS also documents [theme variables and component theme APIs](https://docs.sencha.com/extjs/7.0.0/guides/core_concepts/theming.html). Prefer supported component configuration and existing project conventions over fragile assumptions about generated HTML.

## Proposed appearance integration

Keep color scheme and visual appearance as separate preferences: users can retain light, dark, or automatic color selection while trying the modern design. Provisionally use a PVE-specific appearance cookie with an allowlist of `classic` and `modern`. The exact name and integration remain review decisions.

For the desktop template, add a `pve-modern` class to the document body and load a dedicated stylesheet after the current styles when selected. Body scoping covers menus and dialogs rendered outside individual panels. Keep the existing appearance as the default during the pilot. Preserve current browser state and selected resource when reloading to change the preference.

The mobile Yew GUI and noVNC/xterm console templates are separate. Retain their current selection precedence and do not inject desktop styles into them. Switching appearance should require no database change and should leave management operations available through the current interface.

Suggested source organization:

```text
www/css/modern-pve.css
www/css/modern/           optional Sass sources using the existing toolchain
  tokens.scss
  buttons.scss
  forms.scss
  tables.scss
  navigation.scss
  dialogs.scss
  summaries.scss
www/index.html.tpl       conditional stylesheet and appearance class
www/manager6/window/Settings.js
PVE/Service/pveproxy.pm   validated appearance bootstrap
```

A first pilot can use plain CSS, avoiding any new build dependency. If it expands to Sass, add explicit build dependencies and install rules following the existing `sassc` approach. Do not add Sencha Cmd, npm, Tailwind, or Vite merely to reproduce a design.

## Design rules

Establish a small PVE design specification before writing broad overrides. Keep familiar labels, resource ordering, advanced options, contextual help, and action placement.

| Design area | Proposed rule | Compatibility check |
| --- | --- | --- |
| Surfaces and colors | Semantic variables for background, text, panels, selected states, borders, and status | Light/dark/system mode, contrast, warning and error meanings |
| Typography | Consistent locally available font stack and text hierarchy | Long translated labels, numeric alignment, zoom, field sizing |
| Spacing and density | Predictable spacing with compact management tables | ExtJS layout measurements, virtualization, tree alignment |
| Buttons | Consistent primary, secondary, quiet, and destructive appearances | Disabled/focus/pressed states, split buttons, icon and overflow menus |
| Fields and dialogs | Clear labels, borders, help, errors, restrained corners and shadows | Combo triggers, date pickers, validation, tabs, modal sizing and focus |
| Navigation | Refined tree and context navigation, clear selection and separators | Search, expand/collapse, context menus, nested pools, tags, keyboard behavior |
| Summaries | Cleaner panels, readable metrics and status | Existing charts, notes, data updates, no metric or action removal |
| Feedback | Readable task progress, errors, loading, and completion | Permission errors, failures, long logs, interrupted connections |

Map these rules onto existing ExtJS classes and PVE component classes. Do not import Tailwind's global resets or assume shadcn class names will style ExtJS-generated markup. Avoid broad rules for all buttons, inputs, table cells, or SVGs.

## Where styling needs code changes

Colors, borders, shadows, and many text treatments are straightforward theme work. Spacing, font size, row height, toolbar height, and dialog dimensions can interact with ExtJS's measured layouts. Make deliberate component configuration changes where required and test resizing, popup alignment, scrolling, and virtualized rows.

Charts drawn by the charting engine may need configuration changes because CSS cannot recolor everything it renders. Preserve chart data, update behavior, legends, and accessibility while adjusting their presentation.

Keyboard interaction, accessible names, focus restoration, error announcement, and confirmation flows are behavior changes. Address them in focused fixes using native ExtJS mechanisms. A stylesheet cannot deliver them automatically. Keep resource permissions, API calls, task handling, authentication, forms, and advanced options intact unless a specific improvement is separately designed and tested.

A full responsive/mobile transformation or a dramatically different navigation model would require broader layout work. The first deliverable is a modern desktop appearance with supported resizing, not a claim that every mobile workflow has been redesigned.

## Phases and contribution boundaries

| Phase | Deliverable | Acceptance requirement |
| --- | --- | --- |
| 0 Upstream discussion | Design specification, source evidence, example scope, opt-in integration proposal | Maintainers agree on scope and whether presentation belongs in PVE or the shared toolkit |
| 1 Appearance pilot | Optional modern style, buttons/fields/dialogs/tree, node and VM summaries | Existing workflows still operate, themes switch reliably, layout and keyboard checks pass |
| 2 Shared presentation | Grids, menus, wizards, task viewer, status treatments, charts | All component states and dense tables are covered without layout regressions |
| 3 Screen refinement | Targeted improvements to VM/CT, storage, node, Datacenter, HA, Ceph, SDN | Full settings, advanced controls, permissions, help and actions remain available |
| 4 Accessibility and consistency | Focused behavior fixes and completed regression review | Browser, keyboard, zoom, translation, and assistive-technology evidence |
| 5 Broader rollout | Proposal for default appearance or shared reusable theme | Maintainers accept the maintenance cost, upgrade behavior and rollback route |

Deliver each step through several small PRs and patch series where appropriate. Measure actual effort during the pilot before committing to a completion date. This approach removes most of the framework-rewrite work, but broad visual changes still need careful regression testing.

If common styles move into `proxmox-widget-toolkit`, test Proxmox Backup Server and Mail Gateway consumers and use a separate worktree/branch for that repository. A PVE-only style must not silently alter those products.

## Functionality and validation

Use the feature matrix as a regression scope rather than a rewrite backlog. Preserve the Datacenter → Node → VM/Container hierarchy, storage/pool/SDN views, all advanced editors and wizards, backup/restore, migration, snapshots, replication, firewall, HA, Ceph, notifications, access control, and console behavior.

For each affected component, record representative workflows, required privileges, native widget states, edge cases, and test evidence. Review permission-based visibility, templates/locks/pending settings, disabled states, offline resources, long names, large result sets, and validation errors.

Validation should include:

- Before/after browser comparisons using the real packaged UI, with light/dark/system modes and opt-in disabled.
- Interaction scenarios for creation/editing, advanced fields, destructive confirmation, server validation, task logs, and keyboard focus. Verify actual management outcomes in disposable lab instances.
- Chromium, Firefox, and WebKit checks, supported desktop sizes, high-DPI displays, 200 percent zoom, and translated text. Test phone/tablet routing so the separate mobile UI remains unaffected.
- Large trees and tables, column resize/sorting, buffered scrolling, row selection, menus, combo boxes, wizard steps, chart rendering, and long task logs.
- Existing `make check` plus Debian package build/install/upgrade/rollback on the matching Proxmox/Debian environment. No frontend framework or service replacement is needed.
- Cross-product coverage whenever a shared toolkit contribution is introduced.

The current [proxmox-e2e-tests](https://github.com/proxmox/proxmox-e2e-tests) source supplies Perl integration scenarios and uses `proxmox-test-tools` for provisioning/execution. Browser interaction checks must be added alongside suitable lab scenarios; backend test success alone does not establish UI correctness.

Build test packages from the exact commit on the current feature branch and identify them by commit. Include installation/rollback instructions, tested screens, and known limitations. A static mockup is design evidence, not proof that management operations work.

## Upstream submission and immediate next step

Proxmox's [developer guide](https://pve.proxmox.com/wiki/Developer_Documentation) requests coordination before development and uses mailing-list patches rather than GitHub PRs for official contribution. Its [GitHub repositories](https://github.com/proxmox) are read-only mirrors. Use a fork of the original project for our ready-for-review PRs, keep the canonical source as `upstream`, commit and push each completed update, and export agreed work as versioned patches. Link any relevant Proxmox Bugzilla issue in both review and submission material.

The applicable contributor agreement and authorship/sign-off requirements must be resolved by the contributor before official submission. The companion [upstream proposal](modern-ui-upstream-proposal.md) is prepared for review and has not been sent.

**Recommended next contribution:** agree on a shadcn-inspired design specification and implement an optional PVE appearance pilot covering the resource tree, node/VM summary, one advanced edit dialog, and a task viewer. This demonstrates the look on existing functionality before expanding across the product.

This branch contains research and planning only. Source paths and inventory counts were checked locally. No appearance implementation, management tests, frontend build, or Debian package build has been performed for this planning change.
