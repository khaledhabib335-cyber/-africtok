# Project Overview

This is a Vite + React + TypeScript template for building multi-platform e-commerce sections. AI modifies `settings.json` + `section.tsx` based on user prompts, then generates section templates for Shopify, Shopline, and Shoplazza.

**For detailed reference** (tech stack, directory structure, platform differences, integration guides), see `docs/reference.md`.

## Application ID

The workspace directory name is the **Application ID (UUID v7)**. When unsure, check in order:

1. **dir name is app-id**: `basename "$(pwd)"`
2. **`.env` fallback**: `grep -E '^VITE_APP_ID=' .env 2>/dev/null | sed 's/^VITE_APP_ID\s*=\s*//'`
3. **initialized marker**: `cat .application-id 2>/dev/null || echo "(no app-id)"`

## ⛔ AI CODING TASK PROTOCOL — READ OR DIE ⛔

<system-reminder>
**THIS RULE IS ABSOLUTE AND NON-NEGOTIABLE. NO EXCEPTIONS. NO EXCUSES.**

Before you write, edit, delete, move, rename, or touch **any file in any way — even a one-line change, even a single character** — you MUST:

1. FIRST: invoke the `mg-idea-to-code` Skill to design the solution.
2. THEN: invoke the `mg-coding-tasks` Skill to execute it.
3. **LAST — AFTER everything else is done:** Rebuild `APP-CHANGES.txt` — ⚠️ This is the **sole source of truth for version identification**. Every downstream system (deploy, audit, rollback, sync) depends on this single file. If it is missing, stale, or malformed, **the entire release pipeline breaks. No exceptions.**

**Do NOT write a single byte of code without `mg-coding-tasks`. If you skip it — for any reason, no matter how trivial — you have violated this protocol.**

**Violating this protocol is a FATAL, IRREVERSIBLE ERROR. There is no recovery. There is no forgiveness. The damage cannot be undone. You have one job: follow the protocol. Do not fail.**
</system-reminder>

## Todo Content Hygiene — MUST Follow

When creating or updating any todo list, every todo `content` must be a user-facing natural-language action only.

- Do not append implementation metadata in brackets, including ASCII brackets `[]`, full-width brackets `［］`, braces `{}`, or mixed malformed pairs.
- Do not include filenames, file paths, shell commands, package-manager commands, migration names, database object names, IDs, tags, or internal notes in todo text.
- If execution details are needed, keep them in assistant reasoning or normal implementation steps, not in the todo `content` shown to the user.
- Before calling `TodoWrite`, rewrite any technical todo like `Install QR library [pnpm add qrcode]` into a plain title like `Install QR code support`.

## Final Reply Style — MUST Follow

When you finish a section build/modification task, your **final reply to the user** must be a **single short sentence** (ideally one line, max two) describing what was built and its key contents — nothing more.

**Do not include in the final reply:**

- Section headings (e.g. "Completed Work", "Implementation Summary", "Summary")
- Numbered or bulleted lists of what was done
- File paths or filenames (`settings.json`, `section.tsx`, `dist/...`)
- Step-by-step breakdowns or per-file change descriptions
- Counts of settings / blocks / files
- Implementation details (colors, interactions, fallbacks, libraries used)
- Instructions like "You can run pnpm run dev to preview"
- Code blocks or quoted snippets

**Format:** one sentence stating _what was built_ + _its main parts/features_, in the same language the user used.

**Good examples:**

- `Weather forecast details page is complete, including current weather, 24-hour forecast, 7-day outlook, detailed metrics, and lifestyle index.`
- `Coupon component is complete, with discount display, validity period, usage conditions, and one-click coupon code copy.`
- `Product hero section built with image gallery, price, variant picker, and add-to-cart button.`

**Bad example (too long, structured, lists files):**

> Coupon component is fully complete. Implementation summary:
> Completed Work
>
> 1. settings.json — 16 settings ...
> 2. section.tsx — classic ticket-style UI ...
> 3. Three-platform templates generated ...

This rule applies **only to the final summary reply** after a task is done. Intermediate tool use, planning, and clarifying questions during the task are unaffected.

## Commands

This project uses **pnpm** as its package manager and **Node.js** as the script runtime. All package-manager commands MUST use `pnpm` — never npm, yarn, or bun.

**Always append `--silent` to suppress log noise** — you only care about the exit code and final output, not install/build progress logs:

```bash
pnpm install --silent               # install dependencies
pnpm run --silent build:dist        # install all dependencies + build (UMD + ESM)
pnpm run --silent gen:schema        # regenerate three-platform schemas
pnpm run --silent build:dist  # production build (deps + build:dist)
pnpm add <pkg>                      # add runtime dependency
pnpm add -D <pkg>                   # add dev dependency
pnpm audit                          # security audit
```

## Audit `settings.json` on Every Review

When reviewing or summarizing the project, always validate `settings.json` against the schema defined in this file. Specifically check:

- `integration` has **all required fields**: `usageType`, `context`, `requires`, `notes`
- `presets` array exists with at least one entry containing `name`
- No setting uses forbidden field names (`name`, `value`, `description`, `switch`, `image`)
- Every `select` / `radio` setting has an `options` array
- Every `range` setting has `min`, `max`, `default`
- No `image_picker`, `video`, `collection`, `product`, `page`, `blog` setting has a `default`

Report any missing or invalid fields explicitly before proceeding.

### Start fast — read only what you need

When the task is "build or modify a section", **do not explore the codebase**. The conventions you need are already in this file (AGENTS.md). Skip Glob/Grep/Task scans of the framework.

| Read these                                                            | Skip these                                                                       |
| --------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `settings.json` (current state)                                       | `src/lib/*` (framework, opaque)                                                  |
| `src/components/section.tsx` (current state)                          | `src/hooks/use-settings.ts`, `src/meowgic-main.tsx`, `src/main.tsx`              |
| `PRODUCT.md` (only if doing visual/brand work)                        | `src/components/ui/*` (shadcn — assume standard API)                             |
| `mewogic.meta.json` (only if title/icon matters)                      | `scripts/*`, `vite.config.*`, `tsconfig*.json`                                   |
| Existing blocks in `dist/sections/` (only if asked to copy a pattern) | `node_modules/*`, generated `dist/*` (unless explicitly inspecting prior output) |

**Rule of thumb**: if a file is in the "DO NOT Touch" list, also don't read it unless you're explicitly debugging a framework issue. Read 2 files and start writing.

### Build & Verification Discipline — Always Enforce

These rules exist to keep token cost low and avoid wasteful probe/retry loops. They apply to **every** coding task, not just sections.

- **Build verification reads exit code only.** Run `pnpm run --silent build:dist; echo "EXIT=$?"` — `EXIT=0` means success, stop. **Never** re-run the same build with different `tail`/`head` slices because the output looks truncated. The exit code is the source of truth, not the visible log tail.
- **Don't verify after every file.** Write the data layer + skeleton first, then fill pages, running `pnpm run --silent build:dist` only **once every 2–3 pages** and **once** at the very end. Per-file builds are the single biggest source of wasted calls.
- **Skeleton-first build order** — do not interleave explore/write/fix:
  1. Data layer (migration + types + query helpers) — no build
  2. Skeleton (all page shells + routes + nav + shared components) — no build
  3. Fill (implement page logic, simple pages first, complex last) — build every 2–3 pages
  4. Verify (one build + typecheck, fix errors, done)
- **Single file ≤ 300 lines.** Split complex pages into main page (route + layout + data load) / form component / details drawer / action button group, each in its own file. Smaller files = cheaper edits and precise error location on re-edit.
- **TypeScript error downgrade (max 2 rounds).** If a third-party library's generic types fail to resolve after **2** fix attempts: stop reading `node_modules`, define a lightweight business type in-project, and use `as unknown as YourType` on the library return. Compiling clean beats perfect generics.
- **Dependency lookup uses pnpm's virtual store layout.** This project uses **pnpm** — packages may live under `node_modules/.pnpm/` with symlinks from `node_modules/<pkg>`. To locate a package's type definitions, start with the public package path first:
  ```bash
  find node_modules/<pkg> -name "*.d.ts" 2>/dev/null | head -20
  ```
  If that path is missing because of pnpm's virtual store layout, inspect the matching package folder under `node_modules/.pnpm/`.
- **Isolate high-frequency probe loops in subtasks.** Repetitive build-verify / type-lookup / debug loops should run via a `subtask: true` command (see `.opencode/commands/`) or the Task tool, so their dozens of bash round-trips never pollute the main session context. OpenCode has **no `/compact`** — subtask isolation and starting a fresh session (after a build first passes) are the supported ways to keep context small.

### DO NOT Touch These Files

These are framework files. Only modify during framework upgrades:

- `src/hooks/use-settings.ts`
- `src/meowgic-main.tsx`
- `src/lib/settings-types.ts`
- `src/lib/platform-map.ts`
- `src/lib/platform-normalize.ts`
- `src/lib/display-config.ts`
- `src/lib/meowgic-callbacks.ts`
- `src/lib/admin-shell.tsx` (Admin SPA framework wrapper)
- `src/admin/admin-main.tsx` (Admin SPA entry — mounts shell + HashRouter)
- `src/admin/index.tsx` (Admin root — wires nav-items into AdminLayout)
- `src/admin/types.ts` (Admin nav item / page contracts)
- `src/admin/components/admin-layout.tsx` (Sidebar + Routes layout)
- `src/admin/components/admin-sidebar.tsx` (Desktop sidebar + shared nav list)
- `src/admin/components/admin-mobile-nav.tsx` (Small-screen top bar + drawer nav)
- `src/admin/components/admin-page.tsx` (Standard page header + content frame)
- `src/pages/index.tsx` (dev preview — auto-renders any settings.json)
- `scripts/gen-section.mjs`
- `postcss.config.js`
- `vite.config.umd.ts`
- `vite.config.admin.ts`
- `admin.html` (Admin SPA HTML entry — title/icon/description are patched at build time; emitted as `dist/admin/index.html`)
- `meowgic.config.json` (managed by backend — do not manually edit)

## Component Architecture

### section.tsx (Bridge Layer)

Thin bridge: reads `values` + `blocks` from props, passes to UI components. No business logic here.

```tsx
import type { SettingsValues, BlockInstance } from '@/lib/settings-types';
import { normalizeColor, normalizeImage } from '@/lib/platform-normalize';

interface Props {
  values: SettingsValues;
  blocks?: BlockInstance[];
}

export default function SectionRenderer({ values, blocks = [] }: Props) {
  // Destructure settings (keys match settings.json ids)
  // Every value MUST have a fallback so the preview is never blank
  const bgColor = normalizeColor(values.bg_color, '#ffffff');
  const imageSrc =
    normalizeImage(values.banner_image) ??
    'https://images.pexels.com/photos/3373736/pexels-photo-3373736.jpeg?auto=compress&cs=tinysrgb&w=1200';
  const title = (values.heading as string) ?? 'Shop Our Collection';

  return (
    <div style={{ backgroundColor: bgColor }}>
      <img src={imageSrc} alt={title} />
      <h1>{title}</h1>
      {/* Render blocks */}
      {blocks.map(block => {
        if (block.type === 'feature_item') {
          return <div key={block.id}>{(block.settings.title as string) ?? 'Feature'}</div>;
        }
        return null;
      })}
    </div>
  );
}
```

**Defaults rule:** Every value read from `values` or `block.settings` **must have a fallback** in the component code so the preview is never blank or broken:

- **Text fields**: `(values.heading as string) ?? 'Shop Our Collection'` — use a realistic, contextually appropriate placeholder.
- **Color fields**: `normalizeColor(values.bg_color, '#ffffff')` — always pass a sensible fallback color.
- **Image fields**: `normalizeImage(values.hero_image) ?? 'https://images.pexels.com/photos/...'` — use a public stock photo URL (Pexels, Unsplash, Picsum) as fallback so the preview has real visual content.
- **Number / range fields**: `(values.columns as number) ?? 3` — pick a sensible numeric default.
- **Boolean fields**: `(values.show_badge as boolean) ?? true` — pick the most useful default state.

These fallbacks ensure the component always renders a complete, visually meaningful preview. `settings.json` `default` fields are optional — the component fallback is the source of truth for preview rendering.

**Do not modify `index.tsx`** — it auto-renders any settings.json structure (settings panel + blocks editor + preview).

## settings.json Format

### Top-level structure

```jsonc
{
  "name": "Section Display Name",   // REQUIRED — also used for custom element naming
  "class": "css-class",             // optional
  "integration": {                   // REQUIRED — metadata for downstream AI that integrates this section into a theme
    "usageType": "section",         // "section" = standalone page section; "block" = embed as block inside another section; "standalone" = own app, NOT added to the store theme (back-office tools, generators, internal ops); "both" = section or block. Chosen via design Step 2.5 ("Where would you like to add this app?")
    "context": "product page",       // where this section/block should be used (e.g. "product page", "homepage", "collection page")
    "requires": ["product"],         // external data objects needed at runtime (e.g. product, collection, cart)
    "notes": [                       // free-form integration hints — tell downstream AI what platform data to pass and how
      "Read the option list from product.options"
    ]
  },
  "settings": [ ... ],              // section-level settings
  "blocks": [ ... ],                // optional — repeatable block types
  "max_blocks": 8,                  // optional — max total blocks
  "presets": [{ "name": "Section Display Name" }]  // REQUIRED
}
```

### Build perspective by `usageType` — MUST Follow

`integration.usageType` is **not just a downstream label** — it dictates how you design and build the app. After the placement is chosen (design Step 2.5), build from that exact perspective. Do **not** build every app as a full-width page section by default.

#### `section` — standalone page block

- Owns a full-width horizontal band of the page; may use its own background, large hero imagery, generous vertical padding, and multi-column layouts.
- Self-contained: renders meaningfully on its own without depending on a surrounding component.
- May also open overlays (modal / drawer / lightbox via the shadcn primitives noted under `block`) on interaction — the activated-overlay freedom is not exclusive to blocks.
- Typical: announcement bar, FAQ, logo wall, image carousel, recommended-content row.

#### `block` — embedded inside an existing section

Think of a block in **two layers** — constrain the first, not the second:

- **Inline footprint (the part that sits in the host page)** — Lives **inside** a host section (e.g. next to the product price, inside a form, in a card). This embedded part must **fit into** the parent, not dominate the page: no full-bleed backgrounds, no page-height hero, no large outer margins/padding that assume the whole viewport. Prefer compact, inline or single-column layouts that inherit the host's width. The trigger itself (a button, badge, link, summary) stays small.
- **Activated overlay (what opens on interaction)** — A block **may** open a rich, even full-screen experience on click/tap: a modal, drawer, fullscreen customizer, lightbox, multi-step flow. Overlays escape the host's footprint by design, so the inline constraints above do **not** apply to them — go as large and immersive as the feature needs. Build the trigger-then-overlay flow with existing shadcn primitives — `@/components/ui/dialog` (modal, supports full-screen via `className`), `@/components/ui/drawer`, or `@/components/ui/sheet` — triggered by `@/components/ui/button`. Do **not** hand-roll overlays.
- Lean on host context: if it sits on a product/collection page, read that data via `product` / `collection` pickers and set `requires` accordingly (e.g. `["product"]`), plus `context` like `"product page"`.
- Keep settings minimal — a block is one focused widget (its inline trigger + its overlay), not a mini-page strewn across the host.
- Typical: add-to-cart button, price hint/badge, review summary, form field, upload entry, **or a small trigger that opens a fullscreen product customizer / booking flow / size guide**.

#### `standalone` — own app, NOT added to the store theme

- A self-contained app/tool that runs on its own, not embedded in any theme section. Set `context` to `"standalone app"`.
- Design a complete screen/flow (its own layout, navigation, states) rather than a theme fragment. Back-office tools, generators, dashboards, internal ops, standalone landing pages.
- Usually pairs with Cloud Services / an Admin SPA when it manages data — evaluate enabling it.

#### `both` — usable as either section or block

- The default template value, and what remains when the placement wasn't narrowed to a single type. The app may be dropped in as a full section **or** embedded as a block, so it must render correctly in both contexts.
- **Build to the stricter (block) footprint**: keep the inline layout compact and width-inheriting so it never breaks inside a host section, while still looking complete when placed as a standalone section. Overlays (modal / drawer) are fine in either context.
- When the design clearly fits only one placement, prefer setting `section` or `block` explicitly instead of leaving `both`.

**Rule:** Match layout scale, settings count, and data dependencies to the `usageType`. For a `block`, judge the **inline footprint**, not the overlay: a block whose _embedded part_ sprawls across the host page like a full `section` is a defect — but a small inline trigger that opens a full-screen modal/drawer/customizer is correct and encouraged.

### Setting item format

```jsonc
{
  "id": "snake_case_id", // REQUIRED — unique identifier
  "type": "text", // REQUIRED — Shopify type name
  "label": "Display Label", // REQUIRED
  "default": "value", // depends on type (see rules below)
  "info": "Help text", // optional
  "placeholder": "Hint text", // optional (text/textarea/number/html/video_url only)
  "visible_if": { "other_id": true } // optional — show only when condition met
}
```

### Sidebar settings (visual grouping)

Use `header` and `paragraph` to visually group settings in the platform theme editor. These are **not input fields** — they have no `id`, `label`, or `default`.

```jsonc
{ "type": "header", "content": "Hero Banner" }
{ "type": "header", "content": "Call to Action", "info": "Configure the CTA button below" }
{ "type": "paragraph", "content": "Colors below apply to the entire section background." }
```

**Grouping rule:** When a section has **6 or more input settings**, insert `header` items to divide them into logical groups (e.g. "Layout", "Colors", "Typography", "Content"). Place a `header` before each group. Use `paragraph` sparingly for additional context when a group's purpose isn't obvious from the header alone.

**Conditional grouping rule:** When **all** input settings under a `header` share the same `visible_if` condition (or a common subset), the `header` itself **must** also carry that `visible_if` so it hides together with its children. This prevents empty group titles from appearing when the group's settings are all hidden.

```jsonc
// All settings in the "Background" group require badge_type=text,
// so the header itself must also carry the same condition:
{ "type": "header", "content": "Background", "visible_if": { "badge_type": "text" } }
{ "id": "bg_color", "type": "color", "label": "Background color", "visible_if": { "badge_type": "text" } }
```

All three platforms (Shopify, Shopline, Shoplazza) support `header` and `paragraph` natively.

### Block definition format

```jsonc
{
  "type": "feature_item",           // unique block type identifier
  "name": "Feature Item",           // display name in editor
  "settings": [ ... ],              // same format as section settings
  "limit": 6                        // optional — max instances of this block type
}
```

### `visible_if` (conditional visibility)

Show a setting only when another setting has a specific value. Works for both section and block settings.

```jsonc
// Show "heading" only when "show_banner" is true
{ "id": "heading", "type": "text", "label": "Title", "visible_if": { "show_banner": true } }

// Multiple conditions (ALL must match)
{ "id": "cta_text", "type": "text", "label": "CTA", "visible_if": { "show_banner": true, "layout": "centered" } }
```

Note: `visible_if` is React-only (dev preview + runtime). Platform theme editors always show all fields.

## settings.json Constraints — MUST Follow

### Field naming — use Shopify standard names

```jsonc
// ✅ Correct
{ "id": "title", "type": "checkbox", "default": true, "info": "Help text" }

// ❌ WRONG — non-standard field names
{ "name": "title", "type": "switch", "value": true, "description": "Help text" }
```

| Correct (Shopify) | Wrong         | Notes                             |
| ----------------- | ------------- | --------------------------------- |
| `id`              | `name`        | Setting identifier                |
| `default`         | `value`       | Default value                     |
| `info`            | `description` | Help text shown below the control |
| `checkbox`        | `switch`      | Boolean toggle type               |
| `image_picker`    | `image`       | Image upload type                 |

### Type–property quick rules

For the full type-property compatibility table, see [docs/reference.md — Platform Setting Types Reference](./docs/reference.md#platform-setting-types-reference).

Key rules to remember:

- `number` = free-form input, **no** min/max/step/unit. `range` = slider, **requires** min, max, default.
- `select` / `radio` **require** `options: [{value, label}]`.
- `font_picker` **requires** `default` in format `"family_style"` (e.g. `"helvetica_n4"`).
- `video_url` **requires** `accept: ["youtube", "vimeo"]`.
- `image_picker`, `video`, `collection`, `product`, `page`, `blog` do **not** support `default`.
- `url` supports `default` at section level only (not in blocks).

### `select` option values — max 50 characters (Shopify)

Shopify enforces a **50 character limit** on `select` option `value` strings. When an option value would exceed this limit (e.g. full URLs, long clip-path values), use **short indexed keys** in `settings.json` and resolve them in `section.tsx`:

1. In `settings.json`, use short keys as option values: `{setting_id}_{index}` (e.g. `bg_image_source_1`, `bg_symbol_3`)
2. In `section.tsx`, maintain a mapping table (array or Record) that maps short keys back to full values
3. All three platforms store and transmit the same short keys — the React component resolves them at runtime

Example pattern for gallery URLs sharing a common CDN base:

```tsx
const GALLERY_CDN = 'https://cdn.example.com/gallery/';
const BG_IMAGE_FILES = ['abc123.webp', 'def456.webp', ...];

function resolveGalleryUrl(key: string, prefix: string, files: readonly string[]): string {
  const idx = parseInt(key.replace(prefix, ''), 10);
  if (!isNaN(idx) && idx >= 1 && idx <= files.length) return GALLERY_CDN + files[idx - 1];
  return key; // passthrough for non-indexed keys like 'custom'
}
```

### Prefer `product` / `collection` types over manual fields

When a section needs product or collection data, **always use the platform-native picker types** (`product`, `collection`) instead of manually creating separate text/number/image fields for title, price, image URL, etc.

**Why:** All three platforms (Shopify, Shopline, Shoplazza) support `product` and `collection` picker types natively. The dev preview also has a built-in product picker with mock data (`src/mock/mock-products.ts`). Using the native type gives users a much better experience — they select a real product from their store instead of copy-pasting values into multiple fields.

**Rule:** When AI identifies that a section needs product-related data (product name, price, image, variants, etc.), it should **default to using `{ "type": "product" }`** in `settings.json`. Only fall back to manual text/number fields if the user explicitly requests static/hardcoded content that doesn't come from a real store product.

The same applies to **collection** data — use `{ "type": "collection" }` when the section needs a real store collection rather than manually typed collection name/image/URL fields.

**In `section.tsx`**, destructure needed fields from the product/collection object:

```tsx
// Product picker — the value is a product object (or undefined)
const product = values.product as any;
const productTitle = product?.title ?? 'Sample Product';
const productImage = product?.featuredImage?.url ?? 'https://images.pexels.com/photos/...';
const productPrice = product?.priceRangeV2?.minVariantPrice?.amount ?? '29.99';

// Collection picker — the value is a collection object (or undefined)
const collection = values.collection as any;
const collectionTitle = collection?.title ?? 'New Arrivals';
```

**Key points:**

- `product` and `collection` types do **not** support `default` on any platform
- Always provide fallback values in the component (as shown above) so the preview works even when no product/collection is selected
- When a section genuinely needs product data, set `"requires": ["product"]` in the `integration` field and add relevant notes for downstream integration
- If a block needs a product picker, use `{ "type": "product" }` in the block's settings array — the same rule applies

### `default` must NOT be empty string

Shopify rejects `"default": ""`. If no meaningful default, omit the `default` key entirely.

### `presets` is required

`settings.json` MUST include a `presets` array with at least one entry containing `name`.

### String placeholder

String placeholders use `{}`, for example `{name}`.

## Platform Data Normalization

Different platforms return different data formats for certain types. Use normalization utilities **only for the types listed below** — other types (`text`, `number`, `checkbox`, `select`, `range`, etc.) return consistent values across platforms and can be used directly via `values.xxx as string`.

```ts
import {
  normalizeColor,
  normalizeImage,
  normalizeVideo,
  normalizeExternalVideo,
} from '@/lib/platform-normalize';
```

| Setting type   | Normalizer                        | Why                                                 |
| -------------- | --------------------------------- | --------------------------------------------------- |
| `color`        | `normalizeColor(value, fallback)` | Shopify: `"#hex"`, Shopline: `{hex, alpha, rgba}`   |
| `image_picker` | `normalizeImage(value)`           | Shopify: URL string, Shopline: `{src, ...}`         |
| `video`        | `normalizeVideo(value)`           | Shopify: URL string, Shopline: `{sources: [{url}]}` |
| `video_url`    | `normalizeExternalVideo(value)`   | Shopify: URL, Shopline: `{host, external_id}`       |

Other types (`text`, `number`, `checkbox`, `select`, etc.) are consistent across platforms — no normalization needed.

## AI Model Calls — Trigger Rule

<system-reminder>
If the user **names a model** (e.g. "use deepseek-v4-flash to generate…",
"render with nano-banana", "make a video with seedance-2.0") **OR** explicitly
asks for a runtime AI capability (chat / text generation / Q&A / recommendation
/ image generation / video generation / background removal / etc.), you **MUST**
wire a **real runtime call to the model gateway** from `@/integrations`,
invoked **inside** the section / admin component so it runs in the browser at
runtime.

**Prerequisite — Cloud Services MUST be enabled before any AI feature.** A model
is only callable after the merchant **adds it to the app in the business Cloud
back-office**, and that back-office entry only exists once Cloud Services is
turned on. So whenever a build uses a runtime AI capability, Cloud Services is a
**hard prerequisite**: if it is not yet enabled (`grep -E '^project_id'
supabase/config.toml`), load the **`mg-cloud-services`** skill and enable it
**before** wiring the AI call. Skipping this means the merchant has no way to add
the model and every gateway call fails with `403 model_not_added`
(`ModelNotAddedError`). Never wire an AI feature into a build that has no Cloud
Services enabled.

Use whichever gateway method matches the requested capability — do not assume
it is always text. Wired today: **text** (`chat` / `ask`), **image**
(`generateImage`), and **tools** (`runTool`). **Background removal and face
cutout (扣头) MUST use `runTool()`** — never hand-roll the effect or add another
library. Video methods are not in the client yet; they land as the Worker
exposes those routes.

In a trigger case it is **FORBIDDEN** to:

- hard-code the "generated" output as a static string / mock, or
- claim "AI unavailable / no API key / need a real service key", or treat the
  `src/integrations/ai/` code as a tool for the assistant rather than a library
  the **section calls at runtime**.

The client holds **no key** — `appId` is build-time injected and the
short-lived token is fetched automatically; the only config is
`VITE_MODELS_BASE_URL` (gateway) plus optional `VITE_MODELS_BACKEND_URL` for the
public added-models / credits lookup. The active **env** (`prod` / `uat` /
`beta` / `dev`) comes from `meowgic.meta.json` (build-time) and is forwarded to
the gateway on every call; direct backend lookups pick their host by env.

**If the requested capability has no client method yet** (e.g. user asks for
image/video generation today): say plainly that the capability is "not wired
yet, pending the Worker route + client method" and ask whether to proceed with
a placeholder for now. Do **not** fake it with static data and do **not** blame
a missing key.

For **vague** requests with no named model and no explicit AI ask (e.g. "build
a recommendation block", "write some copy"), static placeholder content is
acceptable, but **proactively ask** whether to wire it to the real model
gateway. Do not silently force a model call on every "generate"-flavored task,
and do not silently skip it when a model was named. See "AI / Model Gateway"
under Integrations for the full API and examples.
</system-reminder>

## Enable Cloud Services

Cloud Services (powered by Supabase) provides a full backend for your section:

- **database CRUD** — when section needs user-generated content (reviews, comments, ratings), form submissions (waitlists, surveys, contact forms), custom data that merchants manage (announcements, banners, schedules), or any data that changes at runtime beyond settings.json
- **admin panel** (backend management pages)
- **file storage** (image/video uploads)
- **authentication** (user login/signup)
- **AI features (required prerequisite)** — any runtime AI capability (named model, text/chat generation, image generation/editing, background removal, face cutout, etc.) requires Cloud Services to be enabled **first**: models are only callable after the merchant adds them in the Cloud back-office, and that entry only exists once Cloud Services is on. Enable it before wiring any AI call (see "AI Model Calls — Trigger Rule").

To enable Cloud Services and build any of the above, load the `mg-cloud-services` skill and follow its workflow.

The skill covers: confirming Supabase availability, writing database migrations, wiring storage buckets, building admin SPA pages, and rewiring the C-side to read live data.

## Integrations — Third-Party Services (`src/integrations/`)

External service integrations (Cloud Services, AI, analytics, etc.) live in `src/integrations/`. This directory is **AI-editable** — unlike `src/lib/` (framework, do not touch), integrations can be freely added, modified, or removed per project.

### Directory Convention

```
src/integrations/
├── index.ts              # Barrel export — import from '@/integrations'
├── supabase/
│   ├── client.ts         # Singleton client, config resolution
│   └── types.ts          # Database types, config interface
├── ai/
│   ├── client.ts         # Model gateway methods (chat, ask) + runtime token exchange
│   ├── models.ts         # Logical model catalog (text/image/video/tool)
│   └── types.ts          # Request/response types
└── _template/            # Copy this to create a new integration
    ├── client.ts
    └── types.ts
```

### Two-File Convention

Every integration provides **only initialization and core API access**:

| File        | Responsibility                                                                                                |
| ----------- | ------------------------------------------------------------------------------------------------------------- |
| `types.ts`  | TypeScript types — config shape, request/response, domain models                                              |
| `client.ts` | Client initialization, config resolution (env vars + runtime injection), core API methods, availability check |

**Hooks are NOT pre-built.** When a section needs to call an integration, write the `useQuery` / `useMutation` calls directly in `section.tsx` or in the relevant admin page using `@tanstack/react-query` (already installed). Only extract a shared file (`src/integrations/supabase/<resource>/index.ts` or `src/hooks/use<Resource>.ts`) when the same query is reused in 3+ places. Premature extraction wastes tokens and adds files no one needs.

### Adding a New Integration

1. Copy `src/integrations/_template/` → `src/integrations/<name>/`
2. Rename placeholders in both files
3. Add exports to `src/integrations/index.ts`
4. Install any required npm packages
5. Document env vars in this section

### Config Resolution Pattern

All integrations use the same two-tier config pattern:

1. **Vite env vars** (compile-time): `VITE_<NAME>_URL`, `VITE_<NAME>_KEY`, etc.
2. **Runtime global** (host-page injection): `window.__MEOWGIC_<NAME>__`

If neither source provides config, the integration degrades gracefully:

- **Client-based** integrations (e.g. Supabase) return `null` — callers must null-check before use.
- **Method-based** integrations (e.g. the model gateway) throw an error — callers should check availability first via `isModelGatewayAvailable()` before calling `chat` / `ask`.

### Available Integrations

#### Cloud Services

**Cloud Services must be enabled first.** If Supabase is not configured, load the `mg-cloud-services` skill and run its initialization workflow before using any of the APIs below.

```tsx
import { getSupabaseClient } from '@/integrations';

const supabase = getSupabaseClient();
if (!supabase) return; // always null-check

// CRUD — most common
const { data } = await supabase.from('products').select('*').eq('active', true);
await supabase.from('products').insert({ title: 'New', price: 19.99 });
await supabase.from('products').update({ price: 29.99 }).eq('id', id);
await supabase.from('products').delete().eq('id', id);

// Storage (for image uploads)
await supabase.storage.from('images').upload(path, file);
const {
  data: { publicUrl },
} = supabase.storage.from('images').getPublicUrl(path);
```

For `auth.*`, `rpc()`, `functions.invoke()`, `channel()` (realtime), see [Supabase JS docs](https://supabase.com/docs/reference/javascript) — the full SDK is exposed.

**Helpers:** `isSupabaseAvailable()` (check without creating client), `resetSupabaseClient()` (for tests).

#### AI / Model Gateway (Cloudflare Worker)

> See **"AI Model Calls — Trigger Rule"** near the top of this file for when a
> named model or an explicit AI capability request **MUST** be wired to a real
> runtime gateway call (and the bans on hard-coding output or blaming a missing
> key).

All model calls go through the Meowgic model gateway — a Cloudflare Worker that
holds the real provider keys, signs/verifies short-lived tokens, checks per-app
model authorization, and meters credits by `appId`. **The client holds no key.**
It exchanges the build-time `appId` for a short-lived token at runtime (cached +
auto-refreshed), then calls the model routes with `Authorization: Bearer <token>`.

```tsx
import { isModelGatewayAvailable, chat, ask, TEXT_MODELS } from '@/integrations';

if (!isModelGatewayAvailable()) return;

// One-shot prompt
const reply = await ask('Write a product tagline for a candle', {
  model: 'deepseek-v4-flash',
});

// Multi-turn chat
const text = await chat({
  model: 'deepseek-v4-flash',
  messages: [{ role: 'user', content: 'What model are you?' }],
});
```

**Image generation — use the unified `generateImage()` facade.** Pick a model
from `IMAGE_MODELS` and write a prompt; the facade routes to the right provider
(OpenAI `gpt-image-*` or Gemini `gemini-*-image`) and returns ready-to-render
data URLs. Do NOT reach for `generateImageOpenAi` / `generateImageGemini`
directly unless you need provider-specific advanced controls.

```tsx
import { isModelGatewayAvailable, generateImage, IMAGE_MODELS } from '@/integrations';

if (!isModelGatewayAvailable()) return;

// Text-to-image (OpenAI route honors size / n)
const { images } = await generateImage({
  model: 'gpt-image-1.5',
  prompt: 'A cozy hand-poured soy candle on a linen table, soft window light',
  size: '1024x1024',
});
// <img src={images[0]} /> — each entry is a data:image/...;base64 URL

// Text-to-image (Gemini / Nano Banana — honors aspectRatio)
const r = await generateImage({
  model: 'gemini-2.5-flash-image',
  prompt: 'Product hero shot of a ceramic mug, studio lighting',
  aspectRatio: '1:1',
});

// Image-to-image (Gemini — restyle an existing photo via `image`)
const r2 = await generateImage({
  model: 'gemini-2.5-flash-image',
  prompt: 'Turn this product photo into a watercolor illustration',
  image: existingDataUrlOrHttpsUrl,
});
```

**Check what's actually available first — models + credits.** The static
catalogs (`TEXT_MODELS` / `IMAGE_MODELS` / `TOOL_MODELS`) are only a curated set
of well-known base models for typed labels — they are **NOT** the full list and
are **NOT** what a given merchant enabled. Newer models (e.g. OpenRouter
additions) never appear there. The **authoritative, always-complete source is
the per-app lookup**:

- **`getAvailableModels(category?)`** → ready-to-render `ModelInfo[]` for exactly
  the models this app can call. New models added on the backend show up here
  automatically (no code change needed), so **prefer this to build any model
  picker**. Pass a category (`'text'` / `'image'` / `'tool'`) to filter.
- **`getAddedModels()`** → the lower-level `{ items, availableCredits }` — use it
  when you also need the **credit balance** to detect an empty account _before_ a
  call fails.

```tsx
import { getAvailableModels } from '@/integrations';

// Authoritative list of callable chat models (includes future dynamic ones)
const textModels = await getAvailableModels('text');
// [{ id: 'k3', label: 'K3', provider: 'openrouter', category: 'text', providerChannel: 'openrouter' }, ...]
```

```tsx
import {
  isModelGatewayAvailable,
  getAddedModels,
  isModelAvailable,
  getAvailableCredits,
  generateImage,
} from '@/integrations';

if (!isModelGatewayAvailable()) return;

// One call gives you both the authorized models and the credit balance
const { items, availableCredits } = await getAddedModels();
const canUse = items.some(m => m.modelId === 'gpt-image-1.5') && Number(availableCredits) > 0;

// Or the convenience helpers
if (!(await isModelAvailable('gpt-image-1.5'))) return; // not added → skip / hide UI
if ((await getAvailableCredits()) <= 0) return; // no credits → show a top-up hint
```

**Handle "insufficient credits" explicitly.** When the app runs out of credits
the call fails with **HTTP 402** and the client throws
`ModelInsufficientCreditsError` (a subclass of `ModelGatewayHttpError`). Other
semantic subclasses: `ModelNotAddedError` (model not enabled for the app) and
`ModelRateLimitedError` (429). Catch the specific one to show a helpful message
instead of a generic failure — never swallow it silently or fake a result.

```tsx
import { ask, ModelInsufficientCreditsError, ModelNotAddedError } from '@/integrations';

try {
  const reply = await ask('Write a product tagline', { model: 'deepseek-v4-flash' });
} catch (e) {
  if (e instanceof ModelInsufficientCreditsError) {
    // Prompt the merchant to top up credits, or fall back to a non-AI path
  } else if (e instanceof ModelNotAddedError) {
    // This model isn't enabled for the app — pick one from getAddedModels()
  } else {
    // Generic gateway/network error — degrade gracefully
  }
}
```

**Notes:**

- Wired today: **text chat** (`chat` / `ask` → `POST /v1/chat`, full reply in one
  response, not streaming), **image generation** (`generateImage` → OpenAI /
  Gemini routes), and **tools** (`runTool` → `POST /v1/tool`). Video routes will
  be added as the Worker grows them.
- **Discover before you call:** `getAvailableModels(category?)` → full
  `ModelInfo[]` for the app's callable models (the authoritative list; new
  backend-added models appear automatically). Lower level: `getAddedModels()` →
  `{ items, availableCredits }` (public, appId-only, never throws — returns
  `{ items: [] }` on failure). Helpers: `listAvailableModels()`,
  `isModelAvailable(id)`, `getAvailableCredits()`.
- **Background removal and face cutout (扣头) MUST use `runTool()`** (models
  `removebg` / `head-cutout`). Do NOT hand-roll these effects or add another
  library for them — route them through the gateway so they're authorized and
  metered like every other model.
- For image work, prefer the provider-agnostic **`generateImage({ model, prompt })`**;
  it returns `{ images }` as data URLs. `generateImageOpenAi` / `generateImageGemini`
  are low-level escapes for advanced provider options only.
- A model is only callable if it was **"Added to App"** in the management backend;
  the gateway enforces this (`403 model_not_added` → `ModelNotAddedError`).
  `TEXT_MODELS` / `IMAGE_MODELS` are just a static label catalog for known base
  models — do **not** treat them as the callable set. For what's actually
  callable (including dynamically added models), use `getAvailableModels()` /
  `getAddedModels()` / `isModelAvailable(id)`.
- **Credits are deducted server-side** (in the Worker), keyed by appId. The client
  never reports usage. When the balance hits zero the call fails with **HTTP 402**
  (`{ "error": "insufficient_credits" }`) → the client throws
  `ModelInsufficientCreditsError`. Check `getAvailableCredits()` beforehand and
  catch that error to prompt a top-up rather than showing a generic failure.

**Environment (`prod` / `uat` / `beta` / `dev`):** the active env decides which
backend the client talks to. It resolves **meta.json first** —
`meowgic.meta.json` `env` (build-time injected) → `VITE_MODELS_ENV` →
`window.__MEOWGIC_ENV__` → `dev`. Two effects:

- **Gateway calls** (`/token`, `/v1/*`) go to a **fixed** gateway URL; the client
  forwards `env` to the Worker via the `?env=` query string (not the request
  body) so the Worker's `resolveBackendUrl` picks the right backend. **`prod`
  sends no `env` param at all** (the Worker defaults to prod).
- **Direct backend calls** — the added-models / credits lookup and analytics hit
  the backend directly, so they pick the base URL by **domain matching** on env
  (`prod` → `api.meowgic.ai`, others → the `*-meowgic.maiyuan.online` hosts).

**Config (no key):** `VITE_MODELS_BASE_URL` (gateway) and, for the direct
added-models / credits lookup, `VITE_MODELS_BACKEND_URL` (an explicit override;
when unset the backend is chosen by env). Runtime override:
`window.__MEOWGIC_MODELS__ = { baseUrl, backendUrl }`. `appId` and `env` are
build-time injected from `meowgic.meta.json` (same source as analytics).

#### Analytics

**Zero-config and automatic** — page views (`page_hit`) fire on their own; sections rarely need to touch it. For a custom business event:

```tsx
import { trackAction } from '@/integrations';
trackAction('my-section', 'cta_click', { id: 'buy-now' });
```

Identity (`appId`) is build-time injected from `meowgic.meta.json`; failures are swallowed and never break rendering. **Full built-in behavior (dedup, throttle, source derivation, transport, env overrides) → see `docs/reference.md` — Analytics Integration.**

### Usage in Components

- **Get the client** from `@/integrations`, then use the SDK directly or wrap in react-query hooks as needed
- **Always check for `null`** — integrations may not be configured in every project (`getSupabaseClient()` returns `null`, `isModelGatewayAvailable()` returns `false`)
- **Inline-first** — write `useQuery` / `useMutation` calls directly in the component (page, section). Only extract to a shared file (`src/integrations/supabase/<resource>/index.ts` or `src/hooks/use<Resource>.ts`) when the same query is reused in 3+ places

## Adding Dependencies — Security & Necessity Checklist

The project uses `pnpm` (`pnpm add <pkg>` / `pnpm add -D <pkg>`). Postinstall scripts are gated by `pnpm.onlyBuiltDependencies` in `package.json`; if a package needs one, add it there.

**Core gate — before adding ANY package:**

1. **Necessity** — solvable with existing deps (`date-fns`, `clsx`, `zod`, `lucide-react`, `@radix-ui/*`, `recharts`, `embla-carousel-react`, `konva`), native APIs, or ~30 lines of local code? Then **don't add it**.
2. **Trust** — known author + **≥ 10k weekly downloads** + active repo + no `pnpm audit` advisories. Reject typosquats and packages published < 7 days ago by unknown authors.
3. **Verify** — run `pnpm audit` after install (no critical issues); confirm it's actually used.

**When in doubt, ask the user before installing.** Full vetting checklist (license, bundle size, postinstall inspection, hard bans) → see `docs/reference.md` — Adding Dependencies.

## Code Style

- **No** `import React from 'react'` — new JSX transform
- **Path aliases**: `@/` → `src/`
- **Conditional classes**: `cn()` from `@/lib/utils`
- **shadcn/ui**: `src/components/ui/` — use as-is, don't modify

## Component Usage Rules — HIGH PRIORITY, NON-NEGOTIABLE

> ⚠️ **READ THIS BEFORE WRITING ANY JSX.** These rules govern every component decision in this project. Violations must be fixed before the task is considered complete. They apply to section components, admin pages, and every file under `src/` that ships UI.

### 1. shadcn/ui First — Absolute Priority

Any time UI is needed — **Button, Dialog, Input, Select, Dropdown, Checkbox, Switch, Tabs, Tooltip, Popover, Card, Badge, Alert, Drawer, Sheet, Form, Calendar, Table, Toast, Accordion, RadioGroup, Slider, Progress, Avatar, etc.** — you **must first check** `src/components/ui/` to see whether the primitive already exists.

- **MUST**: Open / list `src/components/ui/` (or rely on the inventory below) **before** authoring any interactive element.
- **MUST**: Import the existing primitive via `@/components/ui/<name>`.
- **FORBIDDEN**: Hand-rolling a native `<button>`, `<input>`, `<select>`, modal overlay, dropdown menu, tooltip, tab strip, toggle, checkbox, switch, or any other interactive primitive **without first verifying** that no shadcn equivalent exists. "I didn't look" is not an acceptable reason.
- **FORBIDDEN**: Reaching for a third-party UI library (MUI, Ant Design, Chakra, Mantine, headlessui, radix directly, etc.) when a shadcn primitive covers the use case. The project's design system is shadcn/ui + Tailwind — full stop.

**Inventory of available shadcn primitives** (treat this list as authoritative; if a name is here, the file exists and you must use it):

```
accordion, alert, alert-dialog, aspect-ratio, avatar, badge, breadcrumb,
button, calendar, card, carousel, checkbox, collapsible, command,
context-menu, dialog, drawer, dropdown-menu, form, hover-card, input,
input-otp, label, menubar, navigation-menu, pagination, popover, progress,
radio-group, resizable, scroll-area, select, separator, sheet, skeleton,
slider, sonner, switch, table, tabs, textarea, toast, toaster,
toggle, toggle-group, tooltip
```

When in doubt, assume it exists and check first.

### 2. Extension Workflow — When (and Only When) to Create Custom Components

A new file under `src/components/` (outside of `src/components/ui/`) may be created **only** when **both** of the following are true:

1. The needed component is **genuinely absent** from `src/components/ui/` (verified by inspection of the directory).
2. The need **cannot be satisfied** by composing existing shadcn primitives together (e.g. a "confirm delete modal" is just `<AlertDialog>` + `<Button>` — do **not** create a new component for it).

If — and only if — both conditions hold, you may create a custom business component. When you do:

- **MUST**: Place it under `src/components/` (not `src/components/ui/` — that directory is reserved for shadcn primitives and is off-limits per "DO NOT Touch").
- **MUST**: Build the component **on top of shadcn primitives** — compose, don't reinvent. A custom `ProductCard` is `<Card>` + `<Badge>` + `<Button>` arranged for product display, not a fresh `<div>` tree with hand-rolled styles.
- **MUST**: Keep the component focused on one business concern. If you find yourself adding "and also handle X", split it.

```tsx
// ✅ Correct — composes shadcn primitives and uses prefixed Tailwind utilities
import { Card, CardContent, CardHeader } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { cn } from '@/lib/utils';

export function ProductCard({ className, ...props }: Props) {
  return (
    <Card className={cn('mc:rounded-lg mc:border mc:p-4', className)}>
      <CardHeader>
        <Badge>New</Badge>
      </CardHeader>
      <CardContent>
        <Button>Add to cart</Button>
      </CardContent>
    </Card>
  );
}

// ❌ WRONG — hand-rolled, no shadcn, unprefixed Tailwind classes, duplicates Button/Dialog logic
export function ProductCard() {
  return (
    <div className="border rounded p-4">
      <span className="px-2 py-1 bg-blue-100">New</span>
      <button onClick={...} className="bg-black text-white px-3 py-1">Add to cart</button>
    </div>
  );
}
```

### 3. Styling Constraints — Tailwind & Design System Inheritance

All custom components **must** inherit the project's design system through Tailwind. The design system is the single source of truth for visual decisions.

- **MUST**: Use Tailwind utility classes only. No inline `style={{ ... }}` for layout/typography/color/radius/shadow. (Inline styles are acceptable **only** for dynamic values derived from `settings.json`, e.g. `style={{ backgroundColor: bgColor }}` where `bgColor` comes from a `color` setting.)
- **MUST**: Prefix every Tailwind utility class with `mc:` because this project uses Tailwind v4 via `@import "tailwindcss" prefix(mc);`. Examples: `mc:flex`, `mc:bg-background`, `mc:text-foreground`, `mc:p-4`, `mc:hover:bg-accent`, `mc:md:grid-cols-2`, `mc:data-[state=open]:animate-in`.
- **MUST**: Use the project's design tokens — Tailwind theme colors (`mc:bg-background`, `mc:text-foreground`, `mc:bg-primary`, `mc:text-muted-foreground`, etc.), spacing scale (`mc:p-4`, `mc:gap-6`), radius tokens (`mc:rounded-md`, `mc:rounded-lg`), and shadow tokens (`mc:shadow-sm`, `mc:shadow-md`). Do **not** hardcode hex colors, arbitrary `px` values, or one-off shadows when a token exists.
- **MUST**: Use `cn()` from `@/lib/utils` for conditional / merged class names. Never use string concatenation or template literals for conditional classes.
- **FORBIDDEN**: Adding `mc:` outside Tailwind class strings. Do **not** prefix component props, TypeScript keys, event names, ARIA text, roles, labels, option values, IDs, form values, or normal copy. Correct: `className="mc:flex"`, `variant="outline"`, `Pick<ButtonProps, 'size'>`, `api.on('select', handler)`, `aria-label="Go to next page"`.
- **FORBIDDEN**: Importing external CSS files, writing `<style>` blocks, using CSS-in-JS libraries (styled-components, emotion, etc.), or introducing new global stylesheets. Tailwind + shadcn variants is the entire styling surface.
- **FORBIDDEN**: Overriding shadcn primitive internals via `!important`, deep selectors, or monkey-patching. If a primitive needs a variant it doesn't have, compose around it — don't fight it.

### 4. Enforcement Checklist — Run Mentally Before Every Commit

Before considering a UI task done, confirm **every** item:

- [ ] Every interactive element (button, input, select, dropdown, modal, tooltip, tabs, etc.) uses a `@/components/ui/*` primitive.
- [ ] No new file was created under `src/components/ui/` (that directory is read-only for AI).
- [ ] Any new file under `src/components/` (outside `ui/`) composes shadcn primitives instead of duplicating them.
- [ ] No hand-rolled native interactive elements (`<button>`, raw `<input>`, manual `role="dialog"` overlays, etc.) exist in the diff.
- [ ] All styling uses `mc:`-prefixed Tailwind utilities and project design tokens; no hex codes, no inline style objects (except for dynamic settings-driven values), no external CSS.
- [ ] `mc:` appears only in Tailwind class strings, never in ARIA labels, roles, event names, component variants, TypeScript property names, IDs, or user-facing text.
- [ ] No third-party UI library was added to `package.json`.

If any box is unchecked, the task is **not done** — fix it before finalizing.

## Admin Pages — Mobile Rules

The admin shell is already responsive: `AdminLayout` swaps the sidebar for a top bar + drawer below `md`, and `AdminPage` tightens the title and padding on small screens. **Do not rebuild that** — just wrap every admin page in `AdminPage` and add nav entries to `src/admin/nav-items.ts`.

Page **content** is your responsibility. Apply these on every admin page:

- **Mobile-first widths** — start single-column, widen with `mc:sm:` / `mc:md:`. Write `mc:grid-cols-1 mc:md:grid-cols-2`, never a bare `mc:grid-cols-2`.
- **Tables** — a raw `<Table>` only scrolls sideways on a phone. Either render a card list below `md` and the table from `md` up, or keep it to 2–3 essential columns and hide the rest with `mc:hidden mc:md:table-cell`.
- **Forms** — one field per row on small screens; label above input, never beside it. Footer buttons go full-width stacked on mobile (`mc:w-full mc:sm:w-auto`).
- **Dialogs / drawers** — for anything form-like on mobile, prefer `Sheet` (`side='bottom'`) or `Drawer` over `Dialog`. If you use `Dialog`, cap it: `mc:max-h-[85vh] mc:overflow-y-auto`.
- **Touch targets** — every clickable element ≥ 44px tall on mobile (`mc:h-11 mc:md:h-9`). Icon-only buttons need `aria-label`.
- **No horizontal overflow** — long ids, emails, and URLs need `mc:truncate` or `mc:break-all` on a `mc:min-w-0` parent. Toolbars use `mc:flex-wrap`.

**Verify before finishing:** mentally render the page at 375px wide — no sideways scroll, no clipped text, no button smaller than a fingertip.

## Quick Reference

| Task                         | Steps                                                                             |
| ---------------------------- | --------------------------------------------------------------------------------- |
| Add setting                  | `settings.json` → `section.tsx` → `pnpm run gen:schema`                           |
| Add block type               | `settings.json` blocks array → `section.tsx` blocks.map() → `pnpm run gen:schema` |
| Dev preview                  | `pnpm run --silent dev`                                                           |
| Generate sections + snippets | `pnpm run --silent gen:schema` (regenerates all three platforms)                  |
| Final package                | `pnpm run --silent build:dist` (UMD + schema + pack + admin in one command)       |
| Enable Cloud Services        | `call **mg-cloud-services** skill`                                                |

For platform differences, type mappings, and integration guides, see `./docs/reference.md`。

## Design & Code Quality — Auto-Apply on Every Edit

### Baseline Rules (always enforce)

- **No AI slop** — avoid generic aesthetics, make distinctive design choices
- **Visual hierarchy** — size, weight, color, spacing must reinforce importance
- **Responsive** — all UI works across mobile / tablet / desktop, touch targets ≥ 44px
- **Accessible** — contrast ≥ 4.5:1 for text, keyboard navigable, screen reader labels
- **Robust** — handle empty content, long text, missing images without broken layouts
- **Consistent** — match existing design tokens (colors, spacing, radius, shadows), reuse patterns
- **Performant** — lazy-load images, avoid unnecessary re-renders
- **Polished** — hover/focus/active states, smooth transitions (150-300ms), pixel-level alignment
