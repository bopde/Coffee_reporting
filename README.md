# LegaSea Coffee Cart

A mobile-first operations app for the LegaSea coffee cart. Designed to run on a phone, tablet, or laptop at events — no installation required. Open the page, enter your Web App URL once, and you're ready to go.

The app covers the full event workflow: point of sale, sales reporting, stock tracking, reorder logging, and an event dashboard. All data is stored in a connected Google Sheet via Google Apps Script.

---

## What's in this repo

| File | Purpose |
|---|---|
| `index.html` | The app — a single HTML file, open it in any browser |
| `apps-script-backend.js` | Google Apps Script backend — paste this into your Apps Script project |

---

## Setup

### 1. Create the Google Sheet

Create a new Google Sheet and add three tabs named exactly:

- `SalesReports`
- `StockLevels`
- `ReorderLog`

Headers are created automatically on first use.

### 2. Deploy the Apps Script

1. In your Google Sheet, go to **Extensions → Apps Script**
2. Delete the default `myFunction` stub
3. Paste the contents of `apps-script-backend.js`
4. In Script Properties (**Project Settings → Script Properties**), add a property:
   - Key: `SHEET_ID`
   - Value: your Sheet ID (the long string in your Sheet's URL between `/d/` and `/edit`)
5. Click **Deploy → New Deployment**
   - Type: **Web App**
   - Execute as: **Me**
   - Who has access: **Anyone**
6. Copy the Web App URL

### 3. Open the app

Open `index.html` in a browser. On first load you'll be prompted for the Web App URL — paste it in and save. That's it.

To share with the team, host the HTML file on GitHub Pages or any static file host and send the link. The Web App URL is stored locally in each device's browser — it never appears in the source code.

---

## Menu and pricing

| Item | Rg | Lg |
|---|---|---|
| White Coffee | $6 | $7 |
| Black Coffee | $5 | $5 |
| Hot Choc | $6 | $7 |
| Tea | $5 | $5 |
| Iced Coffee | $5 | — |
| Coke / Sprite / Coke Zero / L&P | $3 | — |
| Coffee - Other | Custom | — |
| Donation | Custom | — |
| Pies | $9 | — |
| Slice | $5.50 | — |
| Cookie | $3.50 | — |
| Pie + Coffee Combo | $13 | — |

Oat milk is available on any milk-based drink for +$1, including the Pie + Coffee Combo.

The POS groups items into four tabs: **Hot**, **Cold**, **Food**, **Other**.

---

## How the app works

### Sales Report (event workflow)

The main screen for an event. Fill in event details, then use the POS to record orders as you go. At the end of the event, review and submit the report to the Sheet.

**Three sections are independently editable:**
- **Sales** — auto-calculated from POS orders; can be manually overridden
- **Payment Totals** — auto-split from POS (cash vs eftpos); can be manually overridden
- **Stock Used** — auto-calculated from sales; can be manually overridden

Overriding one section never affects the others. Each has a Reset option to return to auto-calculated values.

Data is cached in the browser throughout the event. If you close the tab or go offline, nothing is lost — just reopen and continue.

### Stock

View current stock levels from the Sheet, update them manually, and log reorders (supplier, qty, cost per unit). Reorder costs feed into the dashboard P&L.

### Dashboard

Aggregated view of all submitted events, filterable by date range. Shows revenue, staff cost, stock cost, estimated profit, and top-selling products. Click any event for a full breakdown.

---

## Stock items

The app tracks these stock items by name. Names are used as keys between the frontend and the Sheet — they must match exactly if you edit either:

```
Cup - Lg
Cup - Rg
Milk - Cow 2L
Milk - Oat 1L
Beans - 1kg
Chocolate Powder - Bag
Tea - Box
Iced Coffee - Can
Coke
Sprite
Coke Zero
L&P
Sugar - Bag
Sweetener - Sachet
Takeaway Trays
Pie
Slice
Cookie
```

---

## Stock auto-calculation rules

When stock is auto-calculated from sales, the app uses these rules:

- **Beans:** 19g per espresso-based drink (white coffee, black coffee, coffee-other)
- **Milk (cow):** 300ml per milk drink; oat milk is allocated proportionally based on the Alt Milk count
- **Milk (oat):** 300ml per alt milk drink, replacing cow milk
- **Cups:** Rg or Lg per drink type
- **Cans/bottles:** 1:1 per unit sold
- **Food:** 1:1 per unit sold — Pies draw `Pie`, Slice draws `Slice`, Cookie draws `Cookie`
- **Pie + Coffee Combo:** 1 `Pie`, plus one regular white coffee's worth of stock (1 Rg cup, 300ml milk, 19g beans)

Chocolate powder, sugar, sweetener and takeaway trays are not auto-calculated — enter them by hand in the Stock Used section if you track them.

---

## Dashboard P&L

The estimated profit calculation uses:

- **Revenue** — from submitted sales totals
- **Staff cost** — total hours worked × $30/hr
- **Stock cost** — qty used × most recent reorder cost per unit for each item

All three are estimates. Revenue is exact (from submitted totals), staff cost is exact (from logged hours), stock cost depends on reorder data being logged in the app.

---

## Offline behaviour

The app works fully offline. Data is saved to `localStorage` continuously — nothing is lost if you lose connection mid-event. Submission requires a connection; if offline, a banner is shown and you can resubmit once connectivity returns.

---

## Security

The Web App URL is the only credential protecting access to your Sheet. It is stored in each user's browser `localStorage` and is never embedded in the source code. Keep the URL private — do not share it publicly or commit it to this repo.

To revoke access, redeploy the Apps Script as a new deployment and distribute the new URL.

---

## Making changes

**To update the menu or prices:** edit the `MENU` array in `index.html`. Each item's `price`, `label`, and `size` fields control what appears in the POS.

**To update the staff list:** edit the `STAFF` array near the top of `index.html`.

**To add a stock item:** add it to `STOCK_ITEMS` in `index.html`. If it's a can, bottle, or food item sold through the POS, also add a `sk` property on the relevant menu item pointing to the new stock item name.

**To add a product:** four places in `index.html` have to agree, or the product will be invisible or won't report:

1. `MENU` — the product itself.
    a. Give it an `id`, `label`, `price`, and a `cat` that exists in `CAT_ORDER`.
    b. Set `custom:true` *only* if staff type the price at the till; an item with both a `price` and `custom:true` ignores the price.
    c. Set `cup`, `milkMl`, `beansG`, and `sk` to whatever stock the item consumes.
2. `CAT_ORDER` and `CAT_LABEL` — only needed for a brand new category. A `cat` that is missing here has no POS tab, so its items cannot be reached at all.
3. `M2R` — maps the menu `id` to the report product name. Without an entry the item sells and is counted in the cash/eftpos totals, but its quantity never reaches the Sales section, the submitted report, or the dashboard's Top Sellers.
4. `STOCK_ITEMS` — only needed if the product draws a new stock item via `sk`.

**To add a category:**

1. Add the category key to `CAT_ORDER`, in the order the tabs should appear.
2. Add a matching display name to `CAT_LABEL`.
3. Set `cat` on each `MENU` item that belongs to it.

After any change, redeploy the Apps Script if the backend was affected, and re-host the HTML file. The service worker caches the page, so bump `V` in the service worker block at the top of `index.html` (`cc-v7` → `cc-v8`) whenever you want returning devices to pick the change up on their next load.

---

## Built with

- React 18 (via CDN, no build step)
- Tailwind CSS (via CDN)
- Google Apps Script + Google Sheets
- Service worker for offline caching

*Part of the LegaSea / Kai Ika project ecosystem.*
