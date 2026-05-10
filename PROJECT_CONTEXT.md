# Project Context — החווה בחבל Therapeutic Riding Registration

## Overview
Registration minisite for individual therapeutic horse riding sessions at "החווה בחבל". Single-page HTML app with an admin panel, hosted on GitHub Pages. Google Sheets is the backend for registrations and slot management.

## Live URLs
- **Site:** https://tsafrirezra.github.io/chava-bachevel-therapy/
- **Admin:** https://tsafrirezra.github.io/chava-bachevel-therapy/admin.html
- **Repo:** https://github.com/tsafrirezra/chava-bachevel-therapy
- **Payment (PayBox):** https://links.payboxapp.com/vGTbYsFAQ2b

## Architecture
```
[Registration Form]
       │
       ├─ On load: JSONP script tag → Apps Script ?action=getSlots
       │           ← returns only available ("פנוי") slots → populates dropdown
       │
       └─ On submit: Image ping (GET) → Apps Script (no action)
                     → appends row to Sheet 1 (registrations)
                     → marks slot as "תפוס" in Sheet 2 (slots tab)
                     → success screen + PayBox link

[Admin Panel]
       │
       ├─ On load: JSONP → Apps Script ?action=getAllSlots
       │           ← returns all slots with taken/available status
       │
       ├─ Add slot: Image ping → ?action=addSlot&slot=HH:MM
       ├─ Remove slot: Image ping → ?action=removeSlot&slot=HH:MM
       └─ Restore slot: Image ping → ?action=restoreSlot&slot=HH:MM
                        → sets status back to "פנוי" in slots tab
```

- **Frontend:** `index.html` + `admin.html` (RTL Hebrew, no external dependencies)
- **Backend:** Google Apps Script (doGet) → Google Sheets (2 tabs)
- **Hosting:** GitHub Pages (legacy mode, main branch)
- **Payment:** External PayBox link, shown after successful registration

## Google Apps Script
- **URL:** `https://script.google.com/macros/s/AKfycbwRKJn-Tycnl1P-SNiOboS1STcqlfUEpUkSNBxJHUIXBBUUOy5Gqahp2TRx43UaTkjJ/exec`
- **Method:** `doGet(e)` — reads `e.parameter.*`
- **Access:** Anyone (no login required)

### Apps Script Actions
| Action | Parameters | Returns |
|--------|-----------|---------|
| (none) | childName, age, parentName, phone, time, day | `ok` |
| getSlots | callback (optional) | JSON array of available slots |
| getAllSlots | callback (optional) | JSON array of `{slot, taken}` objects |
| addSlot | slot | `ok` |
| removeSlot | slot | `ok` |
| restoreSlot | slot | `ok` |

### Apps Script Code Pattern
```javascript
function doGet(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var regSheet = ss.getSheets()[0];          // registrations (first tab)
  var slotsSheet = ss.getSheetByName('slots'); // slot config (second tab)
  // reads via getDisplayValues() to avoid Date object conversion issues
  // writes slot status via getRange(row, 2).setValue('תפוס'/'פנוי')
}
```

## Google Sheet Structure
**Tab 1 — Registrations (first/main sheet):**
| A | B | C | D | E | F | G |
|---|---|---|---|---|---|---|
| שם ילד/ה | גיל | שם הורה | טלפון | שעה מועדפת | יום | תאריך הרשמה |

**Tab 2 — "slots":**
| A | B |
|---|---|
| שעה | סטטוס |
| 13:00 | פנוי |
| 13:30 | תפוס |

## Form Fields
| Field | Type | Notes |
|-------|------|-------|
| שם הילד/ה | text | required |
| גיל | dropdown | 4–18 |
| שם הורה/אפוטרופוס | text | required |
| טלפון | tel | min 9 digits |
| שעה מועדפת | dropdown | loaded dynamically from sheet + "זמן/יום אחר" |

## Admin Panel
- **Password:** `chavah2025` — stored in `admin.html` as `const ADMIN_PASSWORD`
- **Note:** Password is client-side only (visible in source). Adequate for basic protection, not cryptographically secure.
- **Features:** view all slots + status, add slot, remove slot, restore taken slot

## Key Technical Decisions

### JSONP instead of fetch() for reading slots
`fetch()` failed silently due to CORS/redirect issues with Google Apps Script URLs (which redirect from script.google.com to googleusercontent.com). JSONP via `<script>` tag injection bypasses CORS entirely. Apps Script returns `callbackName(data)` when `?callback=callbackName` is passed.

### Image ping (GET) for form submission
Same as the original horse course site. `POST`/`fetch` with `no-cors` failed on mobile. Image ping (`new Image().src = url`) works universally across all browsers and mobile devices without CORS issues.

### getDisplayValues() instead of getValues()
Google Sheets auto-converts time strings like "13:00" into Date objects when stored in cells. `getValues()` returns these as JavaScript Date objects causing serialization issues. `getDisplayValues()` returns the cell's displayed text ("13:00") regardless of underlying type.

### ss.getSheets()[0] for registrations
Uses `getSheets()[0]` (first tab by position) instead of `getActiveSheet()` to avoid the script accidentally writing registrations to the "slots" tab if it becomes the active sheet.

### Slot tracking via status column
Registrations are tracked by marking the "slots" tab column B as "תפוס" when someone registers. This is more reliable than comparing registration records to slot list (which had date format comparison issues). Restoring a slot sets it back to "פנוי" without deleting the registration record.

## Program Details
- **Name:** שיעורי רכיבה טיפוליים — החווה בחבל
- **Day:** יום רביעי
- **Hours:** 13:00 – 20:00 (slots every 30 minutes)
- **Type:** פרטני (individual sessions)
- **Price:** 150 ש״ח לשיעור
- **Contacts:** נתנאל 052-595-0524 | צפריר 052-529-9868

## Design
- RTL Hebrew layout
- Earthy green/brown color palette (matches farm theme)
- Mobile responsive, no external CSS/JS dependencies
- Red "מספר המקומות מוגבל" banner at top

## Deployment Flow
```bash
# Edit index.html or admin.html
git add . && git commit -m "message" && git push
# GitHub Pages auto-deploys within ~30 seconds
```

## Related Project
The original horse course minisite lives at:
- **Site:** https://tsafrirezra.github.io/havab-horse-course/
- **Repo:** https://github.com/tsafrirezra/havab-horse-course
- Same GitHub account, same design language, different Google Sheet & Apps Script
