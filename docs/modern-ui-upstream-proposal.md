# Proxmox VE desktop appearance modernization proposal

Discussion draft for `pve-devel`, prepared for review before sending.

I would like to contribute a visual and interaction modernization of the existing Proxmox VE desktop interface while preserving ExtJS, the Datacenter/Node/Guest structure, management APIs, advanced options, permissions, tasks, and consoles.

The proposal adapts shadcn-inspired design conventions: semantic colors, consistent typography and spacing, refined buttons and fields, clearer selection and focus states, and restrained borders and shadows. It would implement these through the existing Proxmox styles and native ExtJS components, without adding React, Tailwind, Vite, or a new runtime.

The existing desktop template and PVE CSS provide integration points, while the shared toolkit's dark Sass theme demonstrates coverage of ExtJS widgets. I propose an optional PVE-specific appearance setting separate from the existing light/dark/system color preference. The original appearance would remain available during the pilot. Mobile Yew and standalone console templates would retain their existing presentation and routing.

The first contribution would cover the resource tree, node and VM summaries, a representative advanced edit dialog, and task viewer. Layout and font changes would be paired with interaction, resizing, keyboard, translation, and dense-table tests. Further improvements would be delivered in small independently reviewable patches, with existing functionality as the regression scope.

Before implementation, I would appreciate guidance on:

1. Whether this scoped improvement to the current ExtJS UI fits the project's plans alongside its ongoing Rust/Yew work.
2. Whether the initial styling should live in PVE or be implemented as a reusable optional style in `proxmox-widget-toolkit`.
3. The preferred appearance setting, design conventions, supported browsers, and test requirements.
4. Which representative screens and shared component states should establish acceptance for the pilot.

I recognize that copying an appearance does not copy accessible interaction behavior, and that changes to shared toolkit styles need coverage for other consuming products. The goal is to preserve existing workflows and address presentation and interaction improvements deliberately.

The detailed plan, initial feature matrix, and pinned source inventory are available for review. Subsequent contributions would follow the applicable contributor agreement, authorship/sign-off, and versioned patch requirements.
