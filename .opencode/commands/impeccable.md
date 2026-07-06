---
description: 'Design, critique, audit, polish, and refine UI using Impeccable (23 commands). Usage: /impeccable <command> [target]. E.g.: /impeccable critique the pricing page'
---

# Impeccable Command

Load the `impeccable` skill, parse the subcommand and target from user input, and execute.

## How to Use

```
/impeccable <command> [target]

Examples:
/impeccable critique the pricing page
/impeccable craft a hero section for enterprise customers
/impeccable polish the checkout form
/impeccable live
```

## Protocol

1. LOAD skill `impeccable`
2. EXTRACT `<command>` and `[target]` from user input after `/impeccable`
3. If no command given, run setup flow (`init` if no context, or show menu)
4. ROUTE to the matching subcommand (see table below)
5. EXECUTE using the impeccable skill flows and reference files

## All Commands

| Category     | Command              | Purpose                         |
| ------------ | -------------------- | ------------------------------- |
| **Build**    | `craft [feature]`    | End-to-end design then build    |
|              | `shape [feature]`    | Plan UX before writing code     |
|              | `init`               | Setup PRODUCT.md, DESIGN.md     |
|              | `document`           | Generate DESIGN.md from code    |
|              | `extract [target]`   | Pull tokens into design system  |
| **Evaluate** | `critique [target]`  | Full design review with scoring |
|              | `audit [target]`     | a11y, perf, responsive checks   |
| **Refine**   | `polish [target]`    | Final quality pass              |
|              | `bolder [target]`    | Amplify safe/bland designs      |
|              | `quieter [target]`   | Tone down aggressive designs    |
|              | `distill [target]`   | Strip to essence                |
|              | `harden [target]`    | Production edge cases, i18n     |
|              | `onboard [target]`   | First-run flows, empty states   |
| **Enhance**  | `animate [target]`   | Purposeful motion               |
|              | `colorize [target]`  | Strategic color                 |
|              | `typeset [target]`   | Typography hierarchy            |
|              | `layout [target]`    | Spacing, rhythm, alignment      |
|              | `delight [target]`   | Personality and polish          |
|              | `overdrive [target]` | Push past limits                |
| **Fix**      | `clarify [target]`   | UX copy, labels, errors         |
|              | `adapt [target]`     | Responsive behavior             |
|              | `optimize [target]`  | UI performance                  |
| **Iterate**  | `live`               | Browser picker, 3 variants      |
| **Manage**   | `hooks`              | Design detector hook management |
|              | `pin <command>`      | Create standalone shortcut      |
|              | `unpin <command>`    | Remove standalone shortcut      |

## Pinned Shortcuts

The following commands are also available as standalone shortcuts:
`/critique`, `/audit`, `/polish`, `/craft`, `/shape`, `/init`, `/live`, `/document`, `/extract`, `/harden`, `/layout`, `/typeset`, `/colorize`, `/animate`, `/distill`, `/clarify`, `/adapt`, `/bolder`, `/quieter`, `/delight`, `/overdrive`, `/optimize`, `/onboard`, `/hooks`
