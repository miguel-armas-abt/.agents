---
name: ui-ux-penpot
description: "Design and validate usable, accessible UI flows, reusable components, and design tokens in Penpot through its MCP tools."
---

[← Back](../SKILL.md) <br>

---

# Annexes

- Penpot MCP tools: `high_level_overview`, `execute_code`, `penpot_api_info`, and `export_shape`.

---

# Table of content

- [1. Design workflow](#1-design-workflow)
- [2. Penpot project structure](#2-penpot-project-structure)
- [3. Design system](#3-design-system)
- [4. Usability and accessibility](#4-usability-and-accessibility)
- [5. Handoff and validation](#5-handoff-and-validation)

---

# 1. Design workflow

1. Establish the user, goal, entry point, constraints, and successful outcome before drawing. Map the primary path and relevant alternate, empty, loading, validation, permission, and failure states.
2. Inspect the connected Penpot file and existing UI patterns before changing it. Reuse its language, components, and tokens where appropriate. Ask the user to select elements only when the target is ambiguous and cannot be determined from the project.
3. Design around real content, backend capabilities, and product rules. Record assumptions and unresolved contract questions; do not invent backend behavior or silently omit required states.
4. Create screens and components with clear hierarchy, consistent spacing, readable text, and responsive behavior. Prefer reusable components and flex/grid layouts for repeated rows, columns, and content groups.
5. Validate the full flow visually and against the criteria in this skill before handing it to implementation.

# 2. Penpot project structure

1. Before using Penpot MCP, read the `Penpot High-Level Overview` once for the current task/context. Use `penpot_api_info` for API members whose behavior or type is unclear.
2. Confirm the intended project, page, and target shapes before editing. Changes can only be made on the active page: activate another page with `penpot.openPage(page)` before modifying it. Creating a page does not activate it.
3. Use `penpotUtils.getPages`, `getPageByName`, `shapeStructure`, `findShape(s)`, and `analyzeDescendants` to inspect and navigate. Copy `penpot.selection` into persistent tool storage immediately if the selected shapes are needed across calls.
4. Give pages, boards, components, variants, and layers semantic names. Keep the shape tree understandable and avoid text layers that merely repeat their shape names.
5. Preserve intended containment and layout. Width/height are read-only; use `resize`. Use `penpotUtils.setParentXY` for parent-relative positions. When adding flex layout to existing content, use `penpotUtils.addFlexLayout` to preserve visual order.
6. Avoid destructive edits unless requested by the design task. Inspect component instances before detaching or changing their main component; understand that removing a component and its instances has persistent effects.

# 3. Design system

1. Inspect local and connected libraries before introducing new values. Reuse existing components, colors, typography, and tokens when they fit the task.
2. Define tokens for repeated semantic values: color roles, spacing, sizing, typography, radii, borders, and shadows. Prefer semantic names such as `color.text.primary` and `space.control.gap` over names tied to a single screen or raw value.
3. Keep a small, coherent spacing and type scale. Name color roles by purpose and state; verify text/background contrast for each use. Avoid unexplained one-off values.
4. Build reusable components for repeated controls and patterns. Use component variants for meaningful axes such as size, emphasis, and state; avoid variant combinations that do not correspond to actual product behavior.
5. Make design tokens and component states usable by the target platform. State values, dimensions, typography, and behavior explicitly enough to prevent guesswork at handoff.

# 4. Usability and accessibility

1. Make the primary action and current screen purpose easy to identify. Use consistent navigation, familiar control behavior, and clear labels.
2. Design feedback for loading, success, validation, empty data, network/backend failure, and destructive actions where those states apply. Preserve user input after recoverable errors.
3. Do not communicate meaning through color alone. Include text, icon, shape, or other redundant cues for status and validation.
4. Use readable text sizing, adequate contrast, clear focus/selection states, and controls with practical touch targets. Keep layouts usable with text scaling and narrow screens; do not encode essential information only in imagery.
5. Keep forms concise. Associate labels and errors with fields, explain constraints before submission where useful, and distinguish field-level errors from form-level failures.

# 5. Handoff and validation

1. Export the relevant screen or board with `export_shape` and inspect the rendered result. Check alignment, clipping, overflow, content order, contrast, type hierarchy, and component state coverage.
2. Review the flow at representative narrow and wide sizes and with long/localized content. Validate responsive rules rather than relying on a single static canvas size.
3. Provide implementation teams with the Penpot page/board names, flow and state descriptions, token/component references, interaction behavior, and any unresolved assumptions. Link each implementation task to its designed screens and states.
4. Keep design and implementation aligned. When a backend contract or platform constraint requires a change, update the design or explicitly document the approved deviation; do not let implementation silently diverge.
5. Report what was designed and validated, and identify any remaining product decisions or unrepresented states.
