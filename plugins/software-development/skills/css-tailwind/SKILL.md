---
name: css-tailwind
description: "Tailwind CSS conventions, Radix UI primitives, and shadcn/ui component patterns for modern React frontends. Loaded by frontend-software-developer-agent when styling components, implementing responsive design, or using shadcn/ui."
user-invocable: false
disable-model-invocation: true
---

# CSS / Tailwind / shadcn Cheat Sheet

## Tailwind Responsive Breakpoints

Mobile-first — unprefixed utilities apply to all sizes, prefixed apply at that breakpoint and up.

| Prefix | Min-width | Typical device |
|--------|-----------|----------------|
| (none) | 0px | Mobile |
| `sm` | 640px | Large phone |
| `md` | 768px | Tablet |
| `lg` | 1024px | Laptop |
| `xl` | 1280px | Desktop |
| `2xl` | 1536px | Large desktop |

## Spacing Scale (selection)

`0` = 0px, `1` = 4px, `2` = 8px, `3` = 12px, `4` = 16px, `6` = 24px, `8` = 32px, `12` = 48px, `16` = 64px, `20` = 80px, `24` = 96px. Use `p-`, `m-`, `gap-`, `space-x-`, `space-y-` prefixes.

## Color System

Format: `{property}-{color}-{shade}`. Shades: 50 (lightest) to 950 (darkest). Examples: `bg-slate-100`, `text-blue-600`, `border-red-500`. Use `dark:` prefix for dark mode variants: `bg-white dark:bg-slate-900`.

## Dark Mode

Enable in `tailwind.config.ts`: `darkMode: "class"`. Toggle by adding `dark` class to `<html>`. All dark styles use `dark:` prefix.

## Common Layout Patterns

### Flexbox

```html
<!-- Centered content -->
<div class="flex items-center justify-center min-h-screen">

<!-- Horizontal nav with spacing -->
<nav class="flex items-center gap-4">

<!-- Sticky footer layout -->
<div class="flex flex-col min-h-screen">
  <main class="flex-1">...</main>
  <footer>...</footer>
</div>
```

### Grid

```html
<!-- Responsive grid: 1 col mobile, 2 tablet, 3 desktop -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

<!-- Sidebar layout -->
<div class="grid grid-cols-[250px_1fr]">
```

## Typography Utilities

`text-xs` (12px), `text-sm` (14px), `text-base` (16px), `text-lg` (18px), `text-xl` (20px), `text-2xl` (24px). Weight: `font-normal`, `font-medium`, `font-semibold`, `font-bold`. Line: `leading-tight`, `leading-normal`, `leading-relaxed`. Truncation: `truncate`, `line-clamp-2`.

## Animations

Built-in: `animate-spin`, `animate-ping`, `animate-pulse`, `animate-bounce`. Transitions: `transition-all duration-200 ease-in-out`. Hover: `hover:scale-105 transition-transform`.

## Radix UI Primitives

Radix provides unstyled, accessible primitives. Style them with Tailwind.

```tsx
import * as Dialog from "@radix-ui/react-dialog";

<Dialog.Root>
  <Dialog.Trigger className="rounded-md bg-blue-600 px-4 py-2 text-white hover:bg-blue-700">
    Open
  </Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay className="fixed inset-0 bg-black/50 data-[state=open]:animate-in data-[state=closed]:animate-out" />
    <Dialog.Content className="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 rounded-lg bg-white p-6 shadow-xl dark:bg-slate-900">
      <Dialog.Title className="text-lg font-semibold">Title</Dialog.Title>
      <Dialog.Description className="mt-2 text-sm text-slate-500">Description</Dialog.Description>
      <Dialog.Close className="absolute right-4 top-4">X</Dialog.Close>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

## shadcn/ui

**Install components:** `npx shadcn@latest add button card dialog`

**cn() utility** — merges Tailwind classes with conflict resolution (uses `clsx` + `tailwind-merge`):

```tsx
import { cn } from "@/lib/utils";

function Card({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn("rounded-lg border bg-card p-6 shadow-sm", className)} {...props} />;
}
```

**Extending components** — wrap and override:

```tsx
import { Button, type ButtonProps } from "@/components/ui/button";

function LoadingButton({ loading, children, ...props }: ButtonProps & { loading?: boolean }) {
  return <Button disabled={loading} {...props}>{loading ? <Spinner className="mr-2 h-4 w-4 animate-spin" /> : null}{children}</Button>;
}
```

## Full Example: Responsive Card

```tsx
import { cn } from "@/lib/utils";

interface ProductCardProps { title: string; price: number; image: string; className?: string }

function ProductCard({ title, price, image, className }: ProductCardProps) {
  return (
    <div className={cn("group rounded-lg border bg-white shadow-sm transition-shadow hover:shadow-md dark:bg-slate-900", className)}>
      <img src={image} alt={title} className="aspect-video w-full rounded-t-lg object-cover" />
      <div className="p-4 sm:p-6">
        <h3 className="text-base font-semibold text-slate-900 dark:text-white sm:text-lg">{title}</h3>
        <p className="mt-1 text-lg font-bold text-blue-600">${price.toFixed(2)}</p>
      </div>
    </div>
  );
}
```

## Key Rules

- Mobile-first: design for small screens, enhance with `sm:`, `md:`, `lg:`.
- Use `cn()` for all className merging — never manual string concatenation.
- Prefer shadcn/ui components over building from scratch.
- Use CSS variables (`hsl(var(--primary))`) for theming, not hardcoded colors in shared components.
