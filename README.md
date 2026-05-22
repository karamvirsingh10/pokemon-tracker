# Complete Pokémon Tracker

A streamlined workflow for tracking Pokémon card purchases at shows.
Snap a photo (or describe a card by voice), confirm it, and Claude handles the rest — Collectr and your Supabase database, all in one shot.

---

## How It Works

### At the Card Show (Phone)

1. Open this project in Claude on your phone
2. Send a **photo** of a card, or **describe it by voice** (e.g. "Radiant Greninja from Astral Radiance, card 46 out of 189")
3. Tell Claude the price (e.g. "$2") — either in the same message or when prompted
4. Claude looks up the card via the PokémonTCG API, shows you a confirmation with the card image, rarity, and condition
5. Reply **yes** to queue it, or give corrections (condition, price, set, etc.)
6. Repeat for each card
7. When finished, say **"done"**

### After Saying "Done"

1. Claude shows a **session summary** — all cards queued with prices
2. A `show-YYYY-MM-DD.csv` is saved to the `Shows/` folder as a local record
3. Claude asks you to confirm with **"go"**
4. After **"go"**, Claude automatically:
   - Opens Chrome and navigates to **app.getcollectr.com**
   - Adds each card to the correct portfolio (price paid, qty, condition, date)
   - **Singles only:** Archives to the **`completed_cards`** table in Supabase
   - **Singles only:** Adds to the **`collection`** table in Supabase (Pokémon Set Checklist)
   - **Sealed products:** Collectr only — no Supabase archiving

No manual steps required — just confirm and it runs.

---

## Useful Commands (During a Session)

| Command | What it does |
|---------|--------------|
| **yes** | Queue the current card |
| **no** / **skip** | Discard the current card |
| **undo** / **remove last** | Remove the last confirmed card |
| **list** | Show all cards queued this session |
| **done** | End the session and trigger submission |
| "condition is LP" | Correct the condition before queuing |
| "I paid $35" | Set the price before queuing |
| "skip price" | Leave price blank |
| "quantity 2" | Set quantity to 2 |
| "note: small scratch" | Add a note to the card |

---

## Project Files

| File/Folder | Purpose |
|-------------|---------|
| `Shows/` | All per-session purchase CSVs live here |
| `Shows/show-YYYY-MM-DD.csv` | Per-session purchase record (saved after "done") |
| `completed.csv` | Legacy purchase history |
| `README.md` | This file |

---

## Supabase Tables

### `completed_cards`
Every submitted card is archived here with `status = "submitted"`.

| Column | What gets written |
|--------|-------------------|
| `card_name` | Pokémon card name (e.g. "Radiant Greninja") |
| `set_name` | Full set name (e.g. "Astral Radiance") |
| `card_number` | Collector number (e.g. "46/189") |
| `rarity` | Rarity (e.g. "Radiant Rare") |
| `condition` | Near Mint / Lightly Played / etc. |
| `cost_paid` | Price paid in CAD |
| `quantity` | Number of copies |
| `status` | Always "submitted" |
| `confidence` | Claude's ID confidence (0–1) |
| `source` | "photo" or "manual entry" |
| `tcg_card_id` | PokémonTCG API ID (e.g. "swsh10-46") |
| `submitted_at` | Auto-filled by database |

### `collection` (Pokémon Set Checklist)
Each card is also inserted here to keep your set checklist up to date.

| Column | What gets written |
|--------|-------------------|
| `product_name` | Card name |
| `set_name` | Set name |
| `card_number` | Zero-padded collector number (e.g. "046/189") |
| `rarity` | Rarity |
| `card_condition` | Condition |
| `quantity` | Quantity |
| `notes` | Any notes |

---

## Collectr

Claude opens Chrome, navigates to `app.getcollectr.com`, and adds each card to your **Sets** portfolio. For each card it:

1. Searches by card name + number
2. Clicks through to the product page
3. Opens **⋮ → Enter Product Details → + Add an Entry**
4. Fills in price paid, quantity (1), condition (Near Mint), and today's date
5. Saves and verifies the entry

> **Note:** If the product page shows "Login to add" despite being logged in, this is a known Collectr rendering glitch — Claude will go back and reload the page automatically.

---

## Condition Reference

| Full Name | Abbreviation |
|-----------|-------------|
| Near Mint | NM (default) |
| Lightly Played | LP |
| Moderately Played | MP |
| Heavily Played | HP |
| Damaged | DMG |
