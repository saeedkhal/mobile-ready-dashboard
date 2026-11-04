# Patterns

Copy-paste implementations of the five rules. Adapt names, i18n and tokens to the project.

## Global CSS (Rules 3, 4, 5) — one block per app

Tailwind v4 (`index.css`):

```css
@import "tailwindcss";

/* touch: variant → touch:min-h-11 */
@custom-variant touch (@media (pointer: coarse));

@layer base {
  /* Rule 3: no iOS focus zoom on phones */
  @media (max-width: 639px) {
    input, textarea, select,
    .ant-input, .ant-input-number-input, .ant-select-selection-search-input,
    .ant-select-selection-item, .ant-picker-input > input {
      font-size: 16px !important;
    }
  }

  /* Rule 4: 44px targets on touch only */
  @media (pointer: coarse) {
    button, [role="button"], a.ant-btn, .ant-btn,
    .ant-pagination-item, .ant-pagination-prev, .ant-pagination-next,
    .ant-tabs-tab, .ant-menu-item, .ant-dropdown-menu-item {
      min-height: 44px;
      min-width: 44px;
    }
    .ant-checkbox-wrapper, .ant-radio-wrapper { min-height: 44px; align-items: center; }
  }
}
```

antd alternative for Rule 3/4 via theme tokens (when the app already uses `ConfigProvider`):

```tsx
const isTouch = window.matchMedia('(pointer: coarse)').matches
const isPhone = window.matchMedia('(max-width: 639px)').matches

<ConfigProvider
  theme={{
    token: { ...(isPhone && { fontSize: 16 }) },
    components: isTouch
      ? { Button: { controlHeight: 44 }, Input: { controlHeight: 44 }, Select: { controlHeight: 44 } }
      : undefined,
  }}
>
```

(Subscribe to `matchMedia(...).addEventListener('change', ...)` if the value must react to rotation/resizing.)

## Rule 1 — Table → cards

antd:

```tsx
import { Card, Grid, List, Table, Tag } from 'antd'

function InvoicesTable({ data, loading, pagination }: Props) {
  const screens = Grid.useBreakpoint()
  const isPhone = !screens.sm // sm = 576 in antd; use a 640 media query if you need Tailwind's sm exactly

  if (isPhone) {
    return (
      <List
        loading={loading}
        dataSource={data}
        pagination={pagination}
        rowKey="id"
        renderItem={(row) => (
          <Card size="small" className="mb-3">
            <div className="flex items-start justify-between gap-2">
              <span className="min-w-0 truncate font-medium">{row.studentName}</span>
              <Tag color={statusColor(row.status)}>{t(`status.${row.status}`)}</Tag>
            </div>
            <dl className="mt-2 grid grid-cols-2 gap-x-3 gap-y-1 text-sm">
              <dt className="text-gray-500">{t('invoice.dueDate')}</dt>
              <dd className="whitespace-nowrap">{formatDate(row.dueDate)}</dd>
              <dt className="text-gray-500">{t('invoice.amount')}</dt>
              <dd className="whitespace-nowrap tabular-nums"><bdi>{formatMoney(row.amount)}</bdi></dd>
            </dl>
            <div className="mt-3 flex justify-end gap-2">{renderActions(row)}</div>
          </Card>
        )}
      />
    )
  }

  return <Table columns={columns} dataSource={data} loading={loading} pagination={pagination} rowKey="id" />
}
```

Exact 640px breakpoint hook (if the project has none):

```ts
import { useSyncExternalStore } from 'react'

export function useMediaQuery(query: string) {
  return useSyncExternalStore(
    (cb) => {
      const mql = window.matchMedia(query)
      mql.addEventListener('change', cb)
      return () => mql.removeEventListener('change', cb)
    },
    () => window.matchMedia(query).matches,
    () => false,
  )
}

// const isPhone = useMediaQuery('(max-width: 639px)')
```

Pure CSS (no JS), for a plain `<table>`:

```css
@media (max-width: 639px) {
  .responsive-table thead { display: none; }
  .responsive-table tr { display: block; margin-bottom: 12px; border: 1px solid var(--border); border-radius: 8px; padding: 8px 12px; }
  .responsive-table td { display: flex; justify-content: space-between; gap: 12px; padding: 4px 0; border: 0; }
  .responsive-table td::before { content: attr(data-label); color: var(--muted); }
}
```

(`<td data-label="Status">`; the JS version above is better when the card needs a different order or emphasis.)

## Rule 2 — Figures

```tsx
<div className="min-w-0">
  <p className="truncate text-sm text-gray-500">{t('dashboard.revenue')}</p>
  <p className="whitespace-nowrap text-2xl font-semibold tabular-nums sm:text-3xl">
    <bdi>+{formatMoney(value)}</bdi>
  </p>
</div>
```

antd `Statistic`: pass `valueStyle={{ whiteSpace: 'nowrap', fontSize: 'clamp(1.25rem, 5vw, 1.875rem)' }}`.

## Rule 5 — Fixed bottom bar

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
```

```tsx
<main className="pb-[calc(64px+env(safe-area-inset-bottom))] sm:pb-0">{children}</main>

<div className="fixed inset-x-0 bottom-0 border-t bg-white px-4 pt-3 pb-[calc(12px+env(safe-area-inset-bottom))] sm:hidden">
  <Button block type="primary">{t('common.save')}</Button>
</div>
```

The content padding (`64px`) must match the bar's height without its safe-area part.

## Verifying in a browser

```js
// no horizontal scroll
document.documentElement.scrollWidth <= window.innerWidth

// inputs under 16px on the current page
[...document.querySelectorAll('input, textarea, select')]
  .filter((el) => parseFloat(getComputedStyle(el).fontSize) < 16)

// targets under 44px (run with touch emulation on)
[...document.querySelectorAll('button, [role=button], a')]
  .filter((el) => { const r = el.getBoundingClientRect(); return r.width && (r.height < 44 || r.width < 44) })
```
