# Tailwind CSS Syntax Reference

A single-page lookup for Tailwind CSS v4 in a React + TypeScript codebase: the CSS-first configuration, custom utilities and variants, and the typed class-composition patterns that keep component APIs honest.

This page assumes the [TypeScript](./typescript.md) and [React](./react.md) references. It covers the parts of Tailwind that change how you write components, not the full utility catalogue — for "what is the class for `justify-content: space-between`", the Tailwind docs search is faster than any page like this.

**How to use this page.** One long file, so browser find (`Ctrl+F` / `Cmd+F`) is the search tool. Search for the directive or helper — `@theme`, `@utility`, `cva`, `tailwind-merge` — rather than a description.

| Mark | Meaning |
|---|---|
| `/* → */` | the CSS a utility generates |
| `// →` | the value the expression produces at runtime |
| `// ^?` | the type TypeScript infers |
| `// ✗` | an error, followed by the message |
| `/* v3 */` | how this was written in Tailwind v3 |

Checked against **Tailwind CSS 4.3.3**, with `class-variance-authority` 0.7.1, `tailwind-merge` 3.7, `clsx` 2.1.1 and `tailwind-variants` 3.3. Every claim about which utilities exist was verified by compiling them.

---

## Contents

[Setup](#setup) · [The CSS-first config](#the-css-first-config) · [@theme](#theme) · [Theme namespaces](#theme-namespaces) · [Using theme values](#using-theme-values) · [@source](#source) · [@utility](#utility) · [@custom-variant](#custom-variant) · [@apply and @reference](#apply-and-reference) · [@layer](#layer) · [@plugin and @config](#plugin-and-config) · [Dark mode](#dark-mode) · [Arbitrary values](#arbitrary-values) · [The className problem](#the-classname-problem) · [clsx and tailwind-merge](#clsx-and-tailwind-merge) · [cva](#cva) · [tailwind-variants](#tailwind-variants) · [Typed component APIs](#typed-component-apis) · [Composition](#composition) · [Responsive design](#responsive-design) · [Container queries](#container-queries) · [State variants](#state-variants) · [Data and ARIA variants](#data-and-aria-variants) · [Animation](#animation) · [Forms and typography](#forms-and-typography) · [Idioms](#general-idioms) · [Gotchas](#gotchas) · [v3 → v4](#v3--v4) · [Versions](#version-notes)

---

## Setup

Pick the integration that matches your bundler. All of them read the same CSS.

```bash
# Vite — the fastest, and the default for a React SPA
npm i tailwindcss @tailwindcss/vite

# PostCSS — for anything PostCSS-based
npm i tailwindcss @tailwindcss/postcss

# webpack — a dedicated loader, added in v4.2, ~2x faster than the PostCSS route
npm i tailwindcss @tailwindcss/webpack

# CLI — no bundler
npm i tailwindcss @tailwindcss/cli
```

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

```js
// postcss.config.mjs
export default {
  plugins: { "@tailwindcss/postcss": {} },
};
// note: autoprefixer and postcss-import are no longer needed — v4 does both.
```

```css
/* src/app.css — the whole setup, in one line */
@import "tailwindcss";
```

```tsx
// src/main.tsx
import "./app.css";
```

```bash
# CLI
npx @tailwindcss/cli -i src/app.css -o dist/out.css --watch
```

### Next.js

```ts
// Next 16 uses Turbopack by default; the PostCSS plugin is the supported path
// postcss.config.mjs
export default { plugins: { "@tailwindcss/postcss": {} } };
```

```tsx
// app/layout.tsx
import "./globals.css";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return <html lang="en"><body>{children}</body></html>;
}
```

### Editor support

```jsonc
// .vscode/settings.json — v4 needs the IntelliSense extension told where the
// entry CSS is, since there is no config file to discover
{
  "tailwindCSS.experimental.configFile": "src/app.css"
}
```

Install `bradlc.vscode-tailwindcss`. Without it you get no completion, no hover-to-see-CSS, and no warning on a misspelled class — and Tailwind gives you no other feedback, because an unknown class silently generates nothing.

---

## The CSS-first config

**v4 has no `tailwind.config.js`.** Configuration lives in CSS, which means the design tokens and the stylesheet are the same artifact.

```css
/* v3 */
/* tailwind.config.js */
/*   module.exports = {                          */
/*     content: ["./src/**\/*.{ts,tsx}"],        */
/*     theme: {                                  */
/*       extend: {                               */
/*         colors: { brand: { 500: "#3b82f6" } },*/
/*         fontFamily: { display: ["Inter"] },   */
/*       },                                      */
/*     },                                        */
/*   }                                           */
/* app.css */
/*   @tailwind base;                             */
/*   @tailwind components;                       */
/*   @tailwind utilities;                        */
```

```css
/* v4 — all of the above, in the stylesheet */
@import "tailwindcss";

@theme {
  --color-brand-500: oklch(0.62 0.19 260);
  --font-display: "Inter", sans-serif;
}
```

| v3 | v4 |
|---|---|
| `tailwind.config.js` | `@theme { }` in CSS |
| `@tailwind base/components/utilities` | `@import "tailwindcss"` |
| `content: [...]` | automatic detection, plus `@source` |
| `theme.extend.colors` | `--color-*` custom properties |
| `plugins: [...]` | `@plugin "..."` |
| `addUtilities()` | `@utility` |
| `addVariant()` | `@custom-variant` |
| `darkMode: "class"` | `@custom-variant dark (...)` |
| `postcss-import`, `autoprefixer` | built in |

A JS config still works if you need it — see [@config](#plugin-and-config) — but it is an escape hatch, not the default.

---

## @theme

`@theme` declares design tokens. Each one becomes **both** a CSS custom property and a set of utility classes.

```css
@import "tailwindcss";

@theme {
  --color-brand-50: oklch(0.97 0.02 260);
  --color-brand-500: oklch(0.62 0.19 260);
  --color-brand-900: oklch(0.38 0.13 260);

  --font-display: "Inter Variable", ui-sans-serif, system-ui, sans-serif;

  --spacing-gutter: 1.5rem;

  --radius-card: 0.75rem;

  --breakpoint-3xl: 120rem;

  --shadow-glow: 0 0 20px oklch(0.62 0.19 260 / 0.5);

  --ease-snappy: cubic-bezier(0.2, 0, 0, 1);
}
```

Every one of those generates utilities. Verified by compiling:

```html
<div class="bg-brand-500 text-brand-50 border-brand-900"></div>
<div class="font-display"></div>
<div class="p-gutter m-gutter gap-gutter"></div>
<div class="rounded-card"></div>
<div class="3xl:flex"></div>
<div class="shadow-glow"></div>
<div class="ease-snappy"></div>
```

```css
/* --color-brand-500 gives you the whole colour family at once: */
/* bg-brand-500  text-brand-500  border-brand-500  ring-brand-500     */
/* fill-brand-500  stroke-brand-500  decoration-brand-500  from-brand-500 */
/* accent-brand-500  caret-brand-500  outline-brand-500  shadow-brand-500 */
/* plus every opacity modifier: bg-brand-500/50 */
```

### Overriding and resetting

```css
@theme {
  /* replace one namespace entirely */
  --color-*: initial;
  --color-white: #fff;
  --color-ink: oklch(0.2 0 0);

  /* wipe every default token */
  --*: initial;
}
```

Use `--color-*: initial` when you want a closed palette — it removes `red-500`, `slate-200` and the rest, so a stray default colour becomes a class that generates nothing rather than a silent inconsistency.

### `@theme inline`

```css
/* the default: the utility references the variable, so it stays live */
@theme {
  --color-accent: var(--brand);     /* bg-accent → background: var(--color-accent) */
}

/* inline: the VALUE is substituted at build time */
@theme inline {
  --color-accent: var(--brand);     /* bg-accent → background: var(--brand) */
}
```

Reach for `inline` when a token is defined in terms of another custom property that changes at runtime — most commonly a theming setup where `--brand` is swapped on a parent element. Without `inline`, the indirection is frozen at the `@theme` layer and runtime changes do not propagate.

---

## Theme namespaces

The prefix of the variable decides which utilities are generated.

| Namespace | Generates | Example |
|---|---|---|
| `--color-*` | every colour utility | `bg-brand-500` |
| `--font-*` | `font-*` | `font-display` |
| `--text-*` | `text-*` (size) | `text-hero` |
| `--font-weight-*` | `font-*` | `font-heavy` |
| `--tracking-*` | `tracking-*` | `tracking-tight` |
| `--leading-*` | `leading-*` | `leading-snug` |
| `--spacing-*` | `p-*`, `m-*`, `gap-*`, `w-*`, `h-*`, `inset-*` | `p-gutter` |
| `--breakpoint-*` | responsive variants | `3xl:flex` |
| `--container-*` | `max-w-*` and container-query variants | `max-w-prose` |
| `--radius-*` | `rounded-*` | `rounded-card` |
| `--shadow-*` | `shadow-*` | `shadow-glow` |
| `--inset-shadow-*` | `inset-shadow-*` | |
| `--drop-shadow-*` | `drop-shadow-*` | |
| `--text-shadow-*` | `text-shadow-*` (4.1+) | |
| `--blur-*` | `blur-*` | |
| `--perspective-*` | `perspective-*` | |
| `--aspect-*` | `aspect-*` | |
| `--ease-*` | `ease-*` | `ease-snappy` |
| `--animate-*` | `animate-*` | `animate-wiggle` |

```css
/* a single --spacing value drives the whole dynamic scale */
@theme {
  --spacing: 0.25rem;      /* p-1 = 0.25rem, p-7 = 1.75rem, p-93 = 23.25rem */
}
/* v4 generates spacing utilities on demand from this multiplier, so p-93
   works without being predefined. v3 required every step in the config. */
```

---

## Using theme values

Tokens are real CSS variables, available everywhere.

```css
.custom {
  background: var(--color-brand-500);
  font-family: var(--font-display);
  padding: var(--spacing-gutter);
}
```

```tsx
// in inline styles, which is how you bridge to a JS-computed value
function Bar({ pct }: { pct: number }) {
  return (
    <div
      className="h-2 rounded-full"
      style={{
        width: `${pct}%`,
        backgroundColor: "var(--color-brand-500)",
      }}
    />
  );
}

// custom properties in a style object need a cast — React's CSSProperties
// does not allow arbitrary keys
const style = { ["--row-height" as string]: "2rem" } as React.CSSProperties;
<div style={style} className="h-(--row-height)" />;
```

```tsx
// reading a token at runtime
const brand = getComputedStyle(document.documentElement)
  .getPropertyValue("--color-brand-500")
  .trim();

// this is the supported way to share design tokens with a charting library
// or a canvas — there is no JS config object to import in v4.
```

```css
/* the arbitrary-value syntax for a variable is parentheses, not brackets */
/* v4 */
.x { /* class="bg-(--my-color) w-(--my-width)" */ }
/* v3 used class="bg-[var(--my-color)]" — still valid, just longer */
```

---

## @source

v4 detects your source files automatically by scanning the project, respecting `.gitignore` and skipping binaries. You only intervene at the edges.

```css
@import "tailwindcss";

/* add a path the scanner misses — typically a linked package */
@source "../packages/ui/src/**/*.{ts,tsx}";

/* scan a dependency that ships class names in its dist output */
@source "../node_modules/@acme/components/dist";

/* exclude something noisy */
@source not "../src/legacy";

/* register class names that never appear literally in source */
@source inline("bg-red-500 bg-green-500 bg-blue-500");
```

`@source inline(...)` is the v4 replacement for the v3 `safelist`.

```tsx
// the reason safelisting exists: Tailwind scans for COMPLETE class strings.
// It does not evaluate your code.

// ✗ never generates anything
const bad = `bg-${color}-500`;
declare const color: string;

// ✓ the full class names appear literally, so the scanner finds them
const TONE = {
  red: "bg-red-500",
  green: "bg-green-500",
} as const;

function Dot({ tone }: { tone: keyof typeof TONE }) {
  return <span className={TONE[tone]} />;
}
```

This is the single most common Tailwind bug, and the map-of-literals pattern above is the fix. It also gets you a typed `tone` prop for free.

---

## @utility

Defines a custom utility. Replaces v3's `addUtilities` plugin and the `@layer utilities` pattern.

```css
@utility content-auto {
  content-visibility: auto;
}

/* a functional utility takes a value */
@utility tab-* {
  tab-size: --value(integer);
}

/* pull the value from a theme namespace */
@utility indent-* {
  text-indent: --value(--spacing-*);
}

/* accept several value kinds */
@utility scrollbar-* {
  scrollbar-color: --value(--color-*, [color]);
}
```

```html
<div class="content-auto"></div>
<div class="indent-4"></div>
```

| Helper | Accepts |
|---|---|
| `--value(integer)` | `tab-2` |
| `--value(number)` | `opacity-50` |
| `--value(percentage)` | `w-50%` |
| `--value(ratio)` | `aspect-16/9` |
| `--value(--color-*)` | a theme token |
| `--value([length])` | an arbitrary value in brackets |
| `--modifier(...)` | the part after `/`, as in `bg-black/50` |

```css
/* utilities defined this way participate in variants automatically */
/* hover:content-auto, md:indent-4 and lg:tab-4 all work with no extra setup */
```

Custom utilities are for genuinely new CSS properties. For "a button that looks like this", use a component with `cva` — a `.btn` utility class reintroduces exactly the naming problem Tailwind exists to avoid.

---

## @custom-variant

Defines a new variant. Replaces `addVariant`.

```css
/* simple */
@custom-variant pointer-fine (@media (pointer: fine));

/* selector-based — & is the element */
@custom-variant hocus (&:hover, &:focus-visible);

@custom-variant theme-midnight (&:where([data-theme="midnight"] *));

/* nested, for a compound condition */
@custom-variant supports-grid {
  @supports (display: grid) {
    @slot;
  }
}
```

```html
<button class="hocus:bg-brand-500 pointer-fine:shadow-lg theme-midnight:text-white"></button>
```

```css
/* v4.3 added stacked and compound forms to @variant, for applying a variant
   inside plain CSS rather than declaring one: */
.card {
  @variant hover:focus {
    outline: 2px solid;
  }
  @variant hover, focus {
    color: var(--color-brand-500);
  }
}
```

---

## @apply and @reference

```css
/* @apply still exists, and is still usually the wrong tool */
.prose-link {
  @apply text-brand-500 underline underline-offset-2 hover:text-brand-900;
}
```

The v4 trap: `@apply` in a **separate** CSS file — a CSS module, a Vue/Svelte `<style>` block — has no access to your theme unless you import it.

```css
/* Button.module.css */
@reference "../app.css";      /* pulls in theme + utilities, emits nothing */

.button {
  @apply bg-brand-500 px-gutter rounded-card;
}
```

```css
/* without @reference: */
/* ✗ Cannot apply unknown utility class `bg-brand-500`. Are you using CSS
     modules or similar and missing `@reference`? */

/* if you only need theme VALUES, skip @apply and use the variables —
   it is faster, because @reference re-processes the entry CSS per file */
.button {
  background: var(--color-brand-500);
  border-radius: var(--radius-card);
}
```

In React, prefer composing class strings in TSX over `@apply`. `@apply` moves styling back into CSS files, splits a component's appearance across two places, and loses the type safety of a variant API.

---

## @layer

```css
@layer base {
  h1 { font-size: var(--text-3xl); font-weight: 700; }
  body { @apply bg-white text-slate-900 antialiased; }
}

@layer components {
  .card { @apply rounded-card border p-gutter shadow-sm; }
}
```

v4 uses **native CSS cascade layers**, not Tailwind's own implementation. The order is `theme, base, components, utilities`, so a utility always beats a component class regardless of specificity — which is why `<div className="card bg-red-50">` overrides the card's background as you would expect.

---

## @plugin and @config

```css
/* JS plugins still work */
@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms";

/* with options */
@plugin "@tailwindcss/forms" {
  strategy: base;
}

/* load a v3-style JS config, for incremental migration */
@config "../tailwind.config.js";
```

`@config` is the bridge for a codebase mid-migration, or one whose tokens are generated by a script. Note that a JS config is **not** loaded automatically in v4 — if you upgraded and your theme vanished, this line is what is missing.

---

## Dark mode

v4 defaults to the OS preference. Class- or attribute-based switching is opt-in.

```css
@import "tailwindcss";

/* class-based: <html class="dark"> */
@custom-variant dark (&:where(.dark, .dark *));

/* attribute-based: <html data-theme="dark"> */
@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));
```

```css
/* v3 */
/*   module.exports = { darkMode: "class" } */
```

```tsx
<div className="bg-white text-slate-900 dark:bg-slate-900 dark:text-white" />;
```

```tsx
// a theme toggle that respects the OS default and persists a choice
import { useEffect, useState } from "react";

type Theme = "light" | "dark" | "system";

function useTheme() {
  const [theme, setTheme] = useState<Theme>(
    () => (localStorage.getItem("theme") as Theme | null) ?? "system",
  );

  useEffect(() => {
    const root = document.documentElement;
    const dark =
      theme === "dark" ||
      (theme === "system" &&
        window.matchMedia("(prefers-color-scheme: dark)").matches);

    root.classList.toggle("dark", dark);
    localStorage.setItem("theme", theme);
  }, [theme]);

  return [theme, setTheme] as const;
  //     ^? readonly [Theme, React.Dispatch<React.SetStateAction<Theme>>]
}
```

```html
<!-- run before first paint, or the page flashes light then goes dark -->
<script>
  document.documentElement.classList.toggle(
    "dark",
    localStorage.theme === "dark" ||
      (!("theme" in localStorage) &&
        matchMedia("(prefers-color-scheme: dark)").matches),
  );
</script>
```

### Semantic tokens beat `dark:` everywhere

```css
/* scattering dark: across every component doubles the class count and
   makes a third theme impossible. Define the tokens once instead: */
@theme {
  --color-surface: oklch(1 0 0);
  --color-content: oklch(0.2 0 0);
}

@layer base {
  .dark {
    --color-surface: oklch(0.2 0 0);
    --color-content: oklch(0.98 0 0);
  }
}
```

```tsx
// one class, both themes
<div className="bg-surface text-content" />;
```

---

## Arbitrary values

```tsx
<div className="top-[117px] bg-[#1da1f2] text-[22px] grid-cols-[1fr_500px_2fr]" />;

// a CSS variable uses parentheses in v4
<div className="bg-(--brand) w-(--sidebar-width)" />;
// v3: bg-[var(--brand)]

// an arbitrary PROPERTY
<div className="[mask-type:luminance]" />;

// an arbitrary VARIANT
<div className="[&>*]:mx-2 [&_p]:leading-7 [&:nth-child(3)]:underline" />;

// spaces must be underscores
<div className="grid-cols-[1fr_500px_2fr] shadow-[0_0_0_1px_black]" />;

// an underscore you actually want must be escaped
<div className="content-['hello\_world']" />;

// ambiguity is resolved with a type hint
<div className="text-(length:--my-size) text-(color:--my-color)" />;
```

Arbitrary values are an escape hatch. More than a couple in one component usually means the value belongs in `@theme`.

---

## The className problem

A React component that accepts `className` has to merge it with its own classes. Naive concatenation loses, because **Tailwind class order in the string does not decide the winner — source order in the generated CSS does.**

```tsx
// ✗ the caller cannot override the padding
function Bad({ className }: { className?: string }) {
  return <div className={`p-4 bg-white ${className ?? ""}`} />;
}
<Bad className="p-8" />;
// → class="p-4 bg-white p-8"
// both rules exist; the one later in the stylesheet wins, which is p-8 here
// only by luck of alphabetical generation order. With p-4 vs px-8 you get a
// genuinely unpredictable result.
```

The fix is `tailwind-merge`, which understands the utility groups and drops the earlier conflicting class from the string entirely.

---

## clsx and tailwind-merge

```bash
npm i clsx tailwind-merge
```

```ts
// src/lib/cn.ts — put this in every project
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

```ts
import { cn } from "./lib/cn";

// clsx handles conditionals
cn("p-4", true && "bg-white", false && "hidden", null, undefined);
// → 'p-4 bg-white'

cn("p-4", { "bg-white": true, "opacity-50": false });
// → 'p-4 bg-white'

cn(["p-4", "m-2"]);
// → 'p-4 m-2'

// twMerge resolves conflicts, last one wins
cn("p-4", "p-8");                       // → 'p-8'
cn("px-4 py-2", "p-8");                 // → 'p-8'
cn("p-8", "px-4");                      // → 'p-8 px-4'
cn("bg-red-500", "bg-blue-500");        // → 'bg-blue-500'
cn("text-sm", "text-lg");               // → 'text-lg'

// it is variant-aware
cn("hover:bg-red-500", "hover:bg-blue-500");   // → 'hover:bg-blue-500'
cn("bg-red-500", "hover:bg-blue-500");         // → 'bg-red-500 hover:bg-blue-500'

// and it leaves non-Tailwind classes alone
cn("my-custom-class", "p-4");           // → 'my-custom-class p-4'
```

```tsx
// the corrected component
function Good({ className }: { className?: string }) {
  return <div className={cn("p-4 bg-white", className)} />;
}
<Good className="p-8" />;
// → class="bg-white p-8"
```

`ClassValue` is exported by clsx and is the right type for a prop that forwards to `cn`.

```ts
import type { ClassValue } from "clsx";
type Props = { className?: ClassValue };
```

### Teaching twMerge about your theme

```ts
// custom utilities and non-standard scales need registering, or twMerge
// cannot know that two of your classes conflict
import { extendTailwindMerge } from "tailwind-merge";

export const twMergeCustom = extendTailwindMerge({
  extend: {
    classGroups: {
      "font-size": [{ text: ["hero", "display"] }],
    },
  },
});
```

---

## cva

`class-variance-authority` turns a set of variants into a typed function. This is the standard way to give a component a real API.

```bash
npm i class-variance-authority
```

```ts
// src/ui/button-variants.ts
import { cva, type VariantProps } from "class-variance-authority";

export const buttonVariants = cva(
  // base — always applied
  "inline-flex items-center justify-center rounded-md font-medium transition-colors " +
    "focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-brand-500 " +
    "disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        primary: "bg-brand-500 text-white hover:bg-brand-900",
        secondary: "bg-slate-100 text-slate-900 hover:bg-slate-200",
        ghost: "hover:bg-slate-100",
        destructive: "bg-red-500 text-white hover:bg-red-600",
      },
      size: {
        sm: "h-8 px-3 text-sm",
        md: "h-10 px-4",
        lg: "h-12 px-6 text-lg",
      },
      fullWidth: {
        true: "w-full",
      },
    },
    compoundVariants: [
      { variant: "ghost", size: "sm", class: "px-2" },
    ],
    defaultVariants: {
      variant: "primary",
      size: "md",
    },
  },
);

export type ButtonVariants = VariantProps<typeof buttonVariants>;
//          ^? { variant?: "primary" | "secondary" | "ghost" | "destructive" | null | undefined;
//               size?: "sm" | "md" | "lg" | null | undefined;
//               fullWidth?: boolean | null | undefined }
```

```ts
buttonVariants();
// → the base classes + primary + md, from defaultVariants

buttonVariants({ variant: "ghost", size: "sm" });
// → base + ghost + sm + the compound 'px-2'

buttonVariants({ fullWidth: true });
// → base + primary + md + 'w-full'

// ✗ buttonVariants({ variant: "danger" })
//   Type '"danger"' is not assignable to type
//   '"primary" | "secondary" | "ghost" | "destructive" | null | undefined'.
```

```tsx
// the component
import { cn } from "../lib/cn";
import { buttonVariants, type ButtonVariants } from "./button-variants";
import type { ComponentProps } from "react";

type ButtonProps = ComponentProps<"button"> & ButtonVariants;

export function Button({ className, variant, size, fullWidth, ...rest }: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size, fullWidth }), className)}
      {...rest}
    />
  );
}
```

```tsx
<Button>Save</Button>;
<Button variant="destructive" size="lg">Delete</Button>;
<Button variant="ghost" fullWidth disabled type="submit">Cancel</Button>;

// every native button prop still works, because of ComponentProps<"button">
<Button onClick={() => {}} aria-label="Save" form="f" />;

// ✗ <Button variant="danger" />
//   Type '"danger"' is not assignable to type ...
// ✗ <Button size="xl" />
//   Type '"xl"' is not assignable to type '"sm" | "md" | "lg" | null | undefined'.
```

That is the whole pattern: `cva` owns the class logic and the variant types, `cn` merges the caller's overrides, `ComponentProps<"button">` supplies everything native. Note the variant props are destructured out of `rest` — otherwise `variant="ghost"` ends up as a DOM attribute and React warns.

### The `| null` in VariantProps

```tsx
// cva's generated types include null, which is awkward when you pass a
// variant through from another typed source
type Tone = NonNullable<ButtonVariants["variant"]>;
//   ^? "primary" | "secondary" | "ghost" | "destructive"

function Toolbar({ tone }: { tone: Tone }) {
  return <Button variant={tone}>Go</Button>;
}
```

---

## tailwind-variants

`tailwind-variants` is cva plus built-in `tailwind-merge` and multi-part ("slot") components.

```bash
npm i tailwind-variants
```

```ts
import { tv, type VariantProps } from "tailwind-variants";

export const button = tv({
  base: "inline-flex items-center rounded-md font-medium transition-colors",
  variants: {
    variant: {
      primary: "bg-brand-500 text-white hover:bg-brand-900",
      ghost: "hover:bg-slate-100",
    },
    size: {
      sm: "h-8 px-3 text-sm",
      md: "h-10 px-4",
    },
  },
  defaultVariants: { variant: "primary", size: "md" },
});

button({ variant: "ghost" });
// → merged classes, with conflicts already resolved — no cn() needed

// the className override goes in as a prop
button({ size: "sm", class: "px-8" });
// → the base px-3 is replaced by px-8, not appended
```

```ts
// slots: one definition for a multi-element component
const card = tv({
  slots: {
    root: "rounded-card border bg-white",
    header: "border-b p-gutter font-medium",
    body: "p-gutter",
  },
  variants: {
    padded: {
      false: { header: "p-0", body: "p-0" },
    },
  },
});

const { root, header, body } = card({ padded: false });
```

```tsx
function Card({ children }: { children: React.ReactNode }) {
  const { root, header, body } = card();
  return (
    <div className={root()}>
      <div className={header()}>Title</div>
      <div className={body()}>{children}</div>
    </div>
  );
}
```

| | `cva` | `tailwind-variants` |
|---|---|---|
| size | tiny | larger |
| conflict resolution | bring your own `cn` | built in |
| slots | no | yes |
| compound variants | yes | yes |
| responsive variants | no | yes (opt-in) |
| ecosystem | shadcn/ui and most templates | growing |

Pick one and use it everywhere. `cva` + `cn` is the more common pairing and the one shadcn/ui generates; `tv` earns its extra weight when components have several parts.

---

## Typed component APIs

```tsx
// the full shape of a well-typed Tailwind component
import { cva, type VariantProps } from "class-variance-authority";
import type { ComponentProps } from "react";
import { cn } from "../lib/cn";

const alertVariants = cva("rounded-card border p-gutter", {
  variants: {
    tone: {
      info: "border-blue-200 bg-blue-50 text-blue-900",
      warn: "border-amber-200 bg-amber-50 text-amber-900",
      error: "border-red-200 bg-red-50 text-red-900",
    },
  },
  defaultVariants: { tone: "info" },
});

type AlertProps = ComponentProps<"div"> &
  VariantProps<typeof alertVariants> & {
    title?: string;
  };

export function Alert({ tone, title, className, children, ...rest }: AlertProps) {
  return (
    <div role="alert" className={cn(alertVariants({ tone }), className)} {...rest}>
      {title && <p className="font-medium">{title}</p>}
      {children}
    </div>
  );
}
```

### Polymorphic, with `asChild`

```tsx
// the shadcn/ui pattern: render as a different element without wrapping
// npm i @radix-ui/react-slot
import { Slot } from "@radix-ui/react-slot";

type BtnProps = ComponentProps<"button"> &
  VariantProps<typeof alertVariants> & { asChild?: boolean };

export function Btn({ asChild, className, ...rest }: BtnProps) {
  const Comp = asChild ? Slot : "button";
  return <Comp className={cn("inline-flex px-4", className)} {...rest} />;
}

<Btn asChild><a href="/x">A link that looks like a button</a></Btn>;
```

### Constraining a className prop

```tsx
// you cannot type-check arbitrary Tailwind strings, but you CAN restrict a
// prop to a closed set, which is usually what you actually want
const GAPS = {
  none: "gap-0",
  sm: "gap-2",
  md: "gap-4",
  lg: "gap-8",
} as const;

type Gap = keyof typeof GAPS;

function Stack({ gap = "md", children }: { gap?: Gap; children: React.ReactNode }) {
  return <div className={cn("flex flex-col", GAPS[gap])}>{children}</div>;
}

<Stack gap="lg">…</Stack>;
// ✗ <Stack gap="xl">…</Stack>
//   Type '"xl"' is not assignable to type '"none" | "sm" | "md" | "lg" | undefined'.
```

This also solves the scanner problem from [@source](#source): the literals are in the source, so the classes are generated.

---

## Composition

```tsx
import type { ComponentProps } from "react";
import { cn } from "../lib/cn";

// forward className, spread the rest, put your classes FIRST so the caller wins
type BoxProps = ComponentProps<"div">;

function Box({ className, ...rest }: BoxProps) {
  return <div className={cn("rounded-card border p-gutter", className)} {...rest} />;
}
```

```tsx
// composing two components that both style
function Panel({ className, ...rest }: BoxProps) {
  return <Box className={cn("bg-slate-50 shadow-sm", className)} {...rest} />;
}
// cn flattens through the chain: Box's base → Panel's → the caller's
<Panel className="p-0" />;
// → 'rounded-card border bg-slate-50 shadow-sm p-0'
```

```tsx
// ✗ the order mistake
function Wrong({ className }: { className?: string }) {
  return <div className={cn(className, "p-4")} />;   // component always wins
}
// the caller's className must come LAST for overrides to work.
```

---

## Responsive design

Mobile-first: an unprefixed utility applies everywhere, a prefixed one applies at that breakpoint **and up**.

| Prefix | Min width |
|---|---|
| `sm:` | 40rem (640px) |
| `md:` | 48rem (768px) |
| `lg:` | 64rem (1024px) |
| `xl:` | 80rem (1280px) |
| `2xl:` | 96rem (1536px) |

```tsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4" />;
<p className="text-sm md:text-base lg:text-lg" />;
<div className="hidden md:block" />;
<div className="block md:hidden" />;

// max-* for a range
<div className="md:max-lg:bg-red-500" />;      /* 768px–1023px only */
<div className="max-md:hidden" />;             /* below 768px */

// arbitrary breakpoints
<div className="min-[900px]:flex max-[400px]:hidden" />;

// your own, from @theme
<div className="3xl:flex" />;
```

```css
@theme {
  --breakpoint-3xl: 120rem;
  --breakpoint-sm: 30rem;      /* override a default */
  --breakpoint-*: initial;     /* start from nothing */
}
```

---

## Container queries

Built into core in v4 — no plugin.

```tsx
// mark the container, then size children against IT rather than the viewport
<div className="@container">
  <div className="flex flex-col @md:flex-row @lg:gap-8">
    <aside className="@md:w-1/3" />
    <main className="@md:w-2/3" />
  </div>
</div>;

// named containers, for nesting
<div className="@container/card">
  <div className="@md/card:flex" />
</div>;

// arbitrary sizes
<div className="@container"><div className="@[400px]:grid" /></div>;

// @container-size (v4.3) queries the block axis too
<div className="@container-size h-[300px]">
  <p className="@[block-size>200px]:text-brand-500" />
</div>;

// max-width container queries
<div className="@container"><div className="@max-md:hidden" /></div>;
```

Container queries are the right tool for a reusable component: a card that goes two-column when *it* is wide works in a sidebar and in a full-width grid, which a `md:` breakpoint cannot.

---

## State variants

```tsx
<button className="bg-brand-500 hover:bg-brand-900 active:scale-95 focus-visible:ring-2 disabled:opacity-50" />;

// group — style a child based on an ancestor's state
<a href="#" className="group block rounded-card p-4 hover:bg-slate-50">
  <h3 className="text-slate-900 group-hover:text-brand-500">Title</h3>
  <span className="opacity-0 group-hover:opacity-100">→</span>
</a>;

// named groups, when nesting
<div className="group/item">
  <div className="group/button">
    <span className="group-hover/item:underline group-hover/button:text-red-500" />
  </div>
</div>;

// peer — style a sibling based on a PREVIOUS sibling's state
<div>
  <input type="checkbox" className="peer sr-only" />
  <div className="peer-checked:bg-brand-500 peer-focus-visible:ring-2" />
</div>;

// peer only reaches LATER siblings — CSS has no previous-sibling combinator.
// For an element before the trigger, use has: on the parent.

// has — style a parent based on its contents
<label className="rounded border p-4 has-[:checked]:border-brand-500 has-[:focus-visible]:ring-2">
  <input type="checkbox" />
  Option
</label>;

// not — invert any variant
<div className="not-hover:opacity-70 not-first:border-t" />;

// structural
<li className="first:pt-0 last:pb-0 odd:bg-slate-50 even:bg-white only:hidden empty:hidden" />;

// children and descendants
<ul className="[&>li]:py-2 *:border-b **:text-sm" />;
```

| Variant | Styles |
|---|---|
| `hover:` `focus:` `active:` | the element's own state |
| `focus-visible:` `focus-within:` | keyboard focus, focus inside |
| `disabled:` `checked:` `required:` `invalid:` | form state |
| `group-*:` | a descendant, from an ancestor's state |
| `peer-*:` | a later sibling, from an earlier one's state |
| `has-[...]:` | a parent, from its contents |
| `not-*:` | the inverse of any variant |
| `*:` / `**:` | direct children / all descendants |
| `first:` `last:` `odd:` `even:` `only:` `empty:` | structural position |
| `motion-safe:` `motion-reduce:` | animation preference |
| `print:` | print media |
| `rtl:` `ltr:` | text direction |
| `starting:` | `@starting-style`, for enter transitions |

---

## Data and ARIA variants

The cleanest bridge between React state and Tailwind: put state on the DOM as an attribute, style it in CSS.

```tsx
import { cn } from "../lib/cn";

function Tab({ active, children }: { active: boolean; children: React.ReactNode }) {
  return (
    <button
      data-active={active || undefined}
      className={cn(
        "px-4 py-2 text-slate-500 border-b-2 border-transparent",
        "data-active:text-brand-500 data-active:border-brand-500",
      )}
    >
      {children}
    </button>
  );
}

// `active || undefined` matters: data-active={false} still renders the
// attribute as the string "false", which data-active: would then match.
```

```tsx
// arbitrary data variants
<div className="data-[state=open]:rotate-180 data-[size=large]:p-8" />;

// ARIA variants
<button aria-expanded className="aria-expanded:rotate-180" />;
<th aria-sort="ascending" className="aria-[sort=ascending]:text-brand-500" />;

// this is how headless libraries (Radix, Headless UI) expect to be styled —
// they set data-state, data-disabled and so on for you
<div className="data-[state=open]:opacity-100 data-[state=closed]:opacity-0" />;

// note: `animate-in` / `animate-out` are NOT core Tailwind. They come from a
// plugin — `tw-animate-css` for v4 (`tailwindcss-animate` was the v3 one), which
// shadcn/ui pulls in. Without it those classes generate nothing, silently:
//   @plugin "tw-animate-css";
//   <div className="data-[state=open]:animate-in data-[state=closed]:animate-out" />
```

Prefer this to `className={isOpen ? "rotate-180" : ""}`: the class string stays static and greppable, the state lives in one attribute, and conditional styling composes with variants (`md:data-active:p-8`).

---

## Animation

```tsx
<div className="transition-colors duration-200 ease-snappy hover:bg-brand-500" />;
<div className="transition-all duration-300 delay-150" />;
<div className="transition-transform hover:scale-105 active:scale-95" />;

// built-in animations
<div className="animate-spin" />;
<div className="animate-pulse" />;
<div className="animate-bounce" />;
<div className="animate-ping" />;
```

```css
/* a custom animation — keyframes go INSIDE @theme */
@theme {
  --animate-wiggle: wiggle 1s ease-in-out infinite;

  @keyframes wiggle {
    0%, 100% { transform: rotate(-3deg); }
    50% { transform: rotate(3deg); }
  }
}
```

```tsx
<div className="animate-wiggle" />;

// respect the user's preference
<div className="motion-safe:animate-wiggle motion-reduce:animate-none" />;
```

```tsx
// enter transitions without JS, via @starting-style
<div className="opacity-100 transition-opacity starting:opacity-0" />;
```

---

## Forms and typography

```css
@plugin "@tailwindcss/forms";
@plugin "@tailwindcss/typography";
```

```tsx
// typography: style a block of HTML you do not control
<article className="prose prose-slate dark:prose-invert max-w-none">
  <div dangerouslySetInnerHTML={{ __html: html }} />
</article>;
declare const html: string;

// override individual elements
<article className="prose prose-headings:font-display prose-a:text-brand-500 prose-img:rounded-card" />;
```

```tsx
// forms: sane resets so utilities actually apply to inputs
<input
  type="text"
  className="w-full rounded-card border-slate-300 focus:border-brand-500 focus:ring-brand-500"
/>;

// without the plugin, browser defaults on select and checkbox fight your
// utilities and you end up reaching for appearance-none by hand.
```

---

## General idioms

### Project structure

```
src/
  app.css              ← @import, @theme, @custom-variant, @plugin
  lib/cn.ts            ← the cn() helper
  ui/
    button.tsx         ← component + its cva definition
    alert.tsx
```

Keep the `cva` call in the same file as the component unless it is shared. One file, one component, one variant table.

### Ordering classes

```bash
npm i -D prettier-plugin-tailwindcss
```

```json
{ "plugins": ["prettier-plugin-tailwindcss"] }
```

Automatic, canonical class order. It ends every argument about ordering, and it makes diffs readable because the same classes always land in the same place. It also sorts inside `cva` and `cn` calls if you tell it where to look:

```json
{
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindFunctions": ["cn", "cva", "tv", "clsx"]
}
```

### Extracting is usually wrong

```tsx
import type { ComponentProps } from "react";
import { cn } from "../lib/cn";

// ✗ a "utility" that is really a component
/* @utility btn { @apply rounded px-4 py-2 font-medium; } */

// ✓ a component with a typed API
export function Btn2(props: ComponentProps<"button">) {
  return <button className={cn("rounded px-4 py-2 font-medium", props.className)} {...props} />;
}
```

The rule: repeat utilities freely inside one component; extract a **component** when the markup repeats. Never extract a class name — that is the abstraction Tailwind was built to remove.

### Long class strings

```tsx
import { cn } from "../lib/cn";

// group them by concern across lines; the formatter keeps them tidy
<button
  className={cn(
    // layout
    "inline-flex items-center justify-center gap-2",
    // box
    "h-10 rounded-card px-4",
    // type
    "text-sm font-medium",
    // colour + state
    "bg-brand-500 text-white hover:bg-brand-900",
    "focus-visible:ring-2 focus-visible:ring-brand-500",
    "disabled:pointer-events-none disabled:opacity-50",
    className,
  )}
/>;
declare const className: string | undefined;
```

### Sharing tokens with JS

```ts
// there is no config object to import in v4, so read the CSS variables
export function token(name: string): string {
  return getComputedStyle(document.documentElement).getPropertyValue(name).trim();
}

token("--color-brand-500");     // → 'oklch(0.62 0.19 260)'

// for a chart library, a canvas, or a three.js material
```

---

## Gotchas

| Trap | Reality |
|---|---|
| `` `bg-${color}-500` `` | never generated — the scanner reads literal strings only |
| `tailwind.config.js` after upgrading | not loaded in v4; add `@config` or port to `@theme` |
| `` `${base} ${className}` `` | conflicting utilities both survive; use `cn` |
| `cn(className, "p-4")` | component wins; the caller's class must come last |
| `@apply` in a CSS module | needs `@reference "../app.css"` |
| `data-active={false}` | renders `data-active="false"`, which `data-active:` matches |
| variant props spread onto the DOM | destructure `variant`/`size` out before `{...rest}` |
| `peer-*` on an earlier sibling | only reaches *later* siblings; use `has:` on the parent |
| a misspelled class | silently generates nothing — install the IntelliSense extension |
| `@tailwind base/components/utilities` | replaced by `@import "tailwindcss"` |
| `bg-[var(--x)]` | still works, but v4 prefers `bg-(--x)` |
| dark mode class strategy | opt in with `@custom-variant dark`; not automatic |
| `space-x-*` with flex-wrap | prefer `gap-*`, which wraps correctly |
| very long `dark:` chains | define semantic tokens instead |

```tsx
// the dynamic-class bug, and the three fixes
declare const color: "red" | "green";

// ✗ nothing is generated
<div className={`bg-${color}-500`} />;

// ✓ 1. a literal map (also gives you a typed prop)
const BG = { red: "bg-red-500", green: "bg-green-500" } as const;
<div className={BG[color]} />;

// ✓ 2. cva, for anything with more than one axis
// ✓ 3. an inline style or a CSS variable, for a genuinely runtime value
<div style={{ backgroundColor: `var(--color-${color}-500)` }} />;
```

```tsx
import { cva, type VariantProps } from "class-variance-authority";
import type { ComponentProps } from "react";
import { cn } from "../lib/cn";

// the DOM-attribute leak
const badVariants = cva("px-4", { variants: { tone: { a: "bg-red-500" } } });
type P = ComponentProps<"button"> & VariantProps<typeof badVariants>;

// ✗ tone ends up on the <button>
function Leaky(props: P) {
  return <button className={badVariants({ tone: props.tone })} {...props} />;
}
// this TYPE-CHECKS — a JSX spread of extra props is structurally fine. The
// failure is at runtime only:
// → React warning: React does not recognize the `tone` prop on a DOM element.

// ✓ destructure it out
function Clean({ tone, className, ...rest }: P) {
  return <button className={cn(badVariants({ tone }), className)} {...rest} />;
}
```

---

## v3 → v4

| Area | v3 | v4 |
|---|---|---|
| entry | `@tailwind base; @tailwind components; @tailwind utilities;` | `@import "tailwindcss";` |
| config | `tailwind.config.js` | `@theme { }` in CSS |
| content paths | `content: [...]` | automatic, plus `@source` |
| safelist | `safelist: [...]` | `@source inline(...)` |
| theme values | `theme.extend.*` | `--color-*`, `--spacing-*`, … |
| custom utilities | `addUtilities()` / `@layer utilities` | `@utility` |
| custom variants | `addVariant()` | `@custom-variant` |
| dark mode | `darkMode: "class"` | `@custom-variant dark (...)` |
| plugins | `plugins: [...]` | `@plugin "..."` |
| CSS variables | `bg-[var(--x)]` | `bg-(--x)` |
| container queries | `@tailwindcss/container-queries` | core |
| PostCSS setup | `tailwindcss`, `autoprefixer`, `postcss-import` | `@tailwindcss/postcss` alone |
| layers | Tailwind's own | native CSS cascade layers |
| colours | rgb/hsl | oklch, wider gamut |
| `shadow-sm` | small shadow | renamed — `shadow-xs` is the old `shadow-sm` |
| `outline-none` | removed outline | renamed `outline-hidden` |
| `ring` | 3px | 1px; use `ring-3` for the old default |
| browser support | back to IE-era | Safari 16.4+, Chrome 111+, Firefox 128+ |

```bash
# the automated upgrade, which handles most of the above
npx @tailwindcss/upgrade
```

The renames are the ones that bite silently, because the old names still exist and just mean something slightly different. `shadow-sm`, `ring` and `outline-none` are worth grepping for by hand after the codemod runs.

```css
/* the minimum viable v4 migration: keep the JS config, switch the entry */
@import "tailwindcss";
@config "../tailwind.config.js";
/* then port the theme to @theme incrementally and delete the @config line */
```

---

## Version notes

| Feature | Requires |
|---|---|
| JIT by default | v3.0 |
| arbitrary variants `[&>*]:` | v3.1 |
| container queries (plugin) | v3.2 |
| CSS-first config, `@theme`, `@import "tailwindcss"` | **v4.0** |
| automatic content detection, `@source` | v4.0 |
| `@utility`, `@custom-variant` | v4.0 |
| container queries in core | v4.0 |
| `not-*`, `starting:`, 3D transforms | v4.0 |
| `bg-(--var)` shorthand | v4.0 |
| `text-shadow-*` | v4.1 |
| `mask-*` utilities | v4.1 |
| improved older-Safari fallbacks | v4.1 |
| `@tailwindcss/webpack` loader | v4.2 |
| `mauve`, `olive`, `mist`, `taupe` palettes | v4.2 |
| logical-property utilities, `font-features-*` | v4.2 |
| `scrollbar-*` utilities | v4.3 |
| `scrollbar-gutter-*` | v4.3 |
| `zoom-*`, `tab-*` | v4.3 |
| `@container-size` | v4.3 |
| stacked / compound `@variant` | v4.3 |

`start-*` and `end-*` are deprecated in favour of `inline-s-*` and `inline-e-*` as of v4.2.

Browser floor for v4: Safari 16.4+, Chrome 111+, Firefox 128+. v4 depends on `@property` and `color-mix()`, so there is no polyfill path to older browsers — stay on v3 if you must support them.

Check with `npx tailwindcss --help` (it prints the version) or `npm ls tailwindcss`.
