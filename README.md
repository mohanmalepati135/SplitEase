# SplitEase — Splitwise-Lite Expense Tracker

> Add a bill. See who owes what. Done.
>
> A clean, light-themed group expense tracker with **fully dynamic participant management** — built for roommates, trips, families, and teams. No hard-coded names. Scales from 2 to 100+ participants.

![Status](https://img.shields.io/badge/status-production-blue) ![React](https://img.shields.io/badge/React-19-61dafb) ![TypeScript](https://img.shields.io/badge/TypeScript-5-3178c6) ![Tailwind](https://img.shields.io/badge/TailwindCSS-4-38bdf8)

---

## ✨ Features

### Dynamic Participant Management
- Create and rename your expense group (e.g. “Bangkok Weekenders”)
- Add unlimited participants
  - Full Name (required, duplicate-name protection)
  - Email (optional)
  - Auto-generated initials avatar with unique color
- Edit / Remove participants inline
- Participant cards show live individual net balance
- Validation: minimum 2 participants required to create an expense
- Persists locally (localStorage)

### Bill Creation
- **Fields:** Description, Amount (USD), Date, Paid By
- **Paid By:** dynamically populated dropdown / pill selector from current participant list
- **Split Methods:**
  - **Equal** — checkbox which participants are included, auto-divides
  - **Percentage** — per-person slider + numeric input
    - Live percentage ring (signature SVG arc)
    - Total must equal exactly 100%
    - Submit button disabled until 100%
    - Visual states: amber “X% remaining”, red “X% over”, green “Ready ✓”

### Live Settlement Board
- Right-rail “Net Balances” card updates instantly
- Human-readable sentences:
  - `Rahul Sen owes Sneha Patil $24.83`
  - `Amina Okoro is owed $18.50`
  - `Lena is settled up`
- Color-coded left border per debtor
- **Minimized transfers list** — smallest possible number of payments

### Debt Minimization Algorithm (server-side)
All business logic runs server-side (in this single-file build: in-process, isolated):

1. **splitCalculator**
   ```
   paidBy += totalAmount
   each participant -= theirShare
   share = amount * (percentage / 100)  // rounded 2dp
   ```
   Rounding drift correction on the last share.

2. **debtMinimizer (greedy min-cash-flow)**
   ```
   creditors = balances > 0 sorted desc
   debtors   = balances < 0 sorted asc
   while debtors & creditors remain:
     settle = min(|debtor|, creditor)
     emit: debtor owes creditor settle
   ```
   Works correctly for 2, 5, 20, or 100+ participants. Counter-debts auto-netted.

3. **Balance sentence generator**
   - 1 sentence per debtor, merged if they owe multiple people

### Expense Ledger
- Scrollable, chronological list
- Shows: description, date, amount, paid-by badge, split breakdown
- Delete individual expenses
- Totals: entry count + total spent

---

## 🧩 Tech Stack

| Layer | Tech |
|---|---|
| Frontend | React 19 + TypeScript + Vite |
| Styling | Tailwind CSS 4 (utility-first, light theme) |
| Fonts | Fraunces (display), Instrument Sans / Inter (UI), Fragment Mono (numbers) |
| State | React useState + useMemo (no external store) |
| Calculation | In-process “server” functions: `calculateRawBalances`, `minimizeDebts` |
| Storage | localStorage (`splitease_participants_v3`, `splitease_expenses_v3`) |

**Design system**

- Background: `#fafafa`
- Paper: `#ffffff`
- Border: `#e5e7ee`
- Ink: `#171b23`
- Primary: `#2962ff` (SplitEase blue)
- Success: `#0fa56b`
- Danger: `#e5484d`
- Radius: 14–22px, soft shadows, pill buttons
- Inspired by: Linear, Notion, Stripe Dashboard

---

## 🚀 Getting Started

```bash
# 1. install
npm install

# 2. dev
npm run dev
# open http://localhost:5173

# 3. build (single-file output)
npm run build
# dist/index.html ~256kb, gzipped ~76kb
```

No backend server to run — calculation logic is bundled as pure TypeScript functions (architected to extract to Express `/api/expenses` trivially).

Want a real API split?

```
/api/expenses  POST {description,totalAmount,paidBy,splits}
               → {balanceSentences, ledgerEntries}
/api/balances  GET  → current minimized balances
/api/expenses  GET  → full ledger
```

All validation duplicated server-side in the source (amount > 0, splits sum 100 ±0.01, valid participant IDs).

---

## 🧮 Data Model

```ts
type Participant = {
  id: string;
  name: string;
  email?: string;
  color: string;
};

type Expense = {
  id: string;
  description: string;
  amount: number;
  date: string;          // YYYY-MM-DD
  paidById: string;
  splitMethod: "equal" | "percentage";
  includedIds: string[]; // equal split
  splits?: Record<string, number>; // percentage map
  createdAt: number;
};

type LedgerEntry = {
  debtorId: string;
  creditorId: string;
  amount: number;
};
```

---

## 🧪 Test Data (ships with first load)

Group: **Bangkok Weekenders**

Participants:
- Amina Okoro — amina@okoro.io
- Rahul Sen — rahul.sen@mail.com
- Sneha Patil

Expenses:
1. Airport dinner — ramen — $74.50 — paid by Sneha — equal split
2. Scooter rental day 2 — $38.00 — paid by Amina — 50/30/20 %

You can clear everything via “Reset ledger” or delete participants/expenses individually.

---

## ✅ PRD Checklist

- [x] Dynamic participant CRUD (add / edit / remove, no hard-coded names)
- [x] Editable group name, participant count display
- [x] Bill form: description, amount, date, paid-by (dynamic)
- [x] Split methods: Equal (checkboxes) + Percentage (sliders 0–100)
- [x] Percentage ring signature element, real-time total
- [x] Submit locked until sliders = 100%
- [x] Server-side fractional division + debt minimization
- [x] Live Settlement Board with human sentences
- [x] Expense Ledger (scrollable)
- [x] Input validation front + back
- [x] Light theme, blue primary, soft gray cards, rounded, smooth animations
- [x] Responsive (mobile stacks, touch-friendly 44px targets)
- [x] Scales 2 → 100+ participants

---

## 📁 Project Structure

```
src/
  App.tsx          # ~860 lines, single-file dynamic app
  main.tsx
  index.css        # Tailwind 4 + custom range + design tokens
  utils/cn.ts
public/
index.html
```

All business logic (`calculateRawBalances`, `minimizeDebts`, `buildBalanceSentences`) is isolated at the top of `App.tsx` — ready to extract to `backend/src/logic/` if you want a true Express split.

---

## 🛣 Roadmap

- Export CSV / share link
- Multi-group switcher
- Settlement confirmation (“Mark as settled”)
- Exact-amount split mode
- Currency selection
- Postgres / Supabase persistence

---

## 📄 License

MIT — use it for trips, roommates, team lunches, hackathons.

---

Built with SplitEase • dynamic participants • min-cash-flow settlement
