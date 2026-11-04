# mobile-ready-dashboard

A Claude Code skill that keeps web dashboards usable on phones. Claude loads it whenever it builds or changes a dashboard screen and applies five rules:

1. **Tables become cards** under 640px, so status and deadline are visible without swiping.
2. **Amounts stay on one line**; big figures shrink on phones instead of wrapping their `+` sign.
3. **Inputs are 16px on phones** (no iOS Safari zoom), with the right keyboard per field.
4. **44px touch targets** on touch screens only; a mouse keeps the dense layout.
5. **Fixed bottom bars** pad with the safe area and leave room under the content.

Every change ends with a phone-width checklist.

## Install

```bash
git clone https://github.com/saeedkhal/mobile-ready-dashboard ~/.claude/skills/mobile-ready-dashboard
```

Or per project: clone into `<project>/.claude/skills/mobile-ready-dashboard`.

## Files

- `SKILL.md`: the rules and the checklist
- `references/patterns.md`: copy-paste code for antd, Tailwind v4 and plain CSS
