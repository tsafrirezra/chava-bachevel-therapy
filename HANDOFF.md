# Project Handoff — Therapeutic Riding Registration Minisite
# החווה בחבל — שיעורי רכיבה טיפוליים

## Live URLs
- **Registration form:** https://tsafrirezra.github.io/chava-bachevel-therapy/
- **Admin panel:** https://tsafrirezra.github.io/chava-bachevel-therapy/admin.html
- **GitHub repo:** https://github.com/tsafrirezra/chava-bachevel-therapy
- **PayBox payment:** https://links.payboxapp.com/vGTbYsFAQ2b

## Architecture
```
[User submits form] → [Image ping GET] → [Google Apps Script] → [Google Sheet]
                                               ↓
                                     [Mark slot as "תפוס" in slots tab]
                                               ↓
                              [Success screen + PayBox payment link]

[Page loads] → [JSONP script tag] → [Apps Script getSlots] → [Populates dropdown]
                                    (returns only "פנוי" slots)
```

- **Frontend:** `index.html` + `admin.html` (HTML + CSS + JS, zero dependencies)
- **Backend:** Google Apps Script (doGet) → Google Sheet with 2 tabs
- **Hosting:** GitHub Pages (main branch)
- **Payment:** External PayBox link (separate step after registration)

## Google Sheet Structure
**Sheet 1 (main tab) — Registrations:**
| שם ילד/ה | גיל | שם הורה | טלפון | שעה מועדפת | יום | תאריך הרשמה |

**Sheet 2 ("slots" tab) — Time Slots:**
| שעה | סטטוס |
|------|--------|
| 13:00 | פנוי |
| 13:30 | תפוס |
| ... | ... |

## Google Apps Script
- **URL:** `https://script.google.com/macros/s/AKfycbwRKJn-Tycnl1P-SNiOboS1STcqlfUEpUkSNBxJHUIXBBUUOy5Gqahp2TRx43UaTkjJ/exec`
- **Actions supported:**
  - No action → registers a student, marks slot as "תפוס"
  - `?action=getSlots` → returns available slots (JSONP supported via `&callback=fnName`)
  - `?action=getAllSlots` → returns all slots with status (for admin panel)
  - `?action=addSlot&slot=HH:MM` → adds a new time slot
  - `?action=removeSlot&slot=HH:MM` → removes a slot entirely
  - `?action=restoreSlot&slot=HH:MM` → reopens a taken slot (sets status back to "פנוי")

## Admin Panel
- **URL:** https://tsafrirezra.github.io/chava-bachevel-therapy/admin.html
- **Password:** `chavah2025` (change in `admin.html` line: `const ADMIN_PASSWORD = 'chavah2025'`)
- **Features:** view all slots with status, add slots, remove slots, reopen taken slots

## How Slot Removal Works
1. User registers → form submits via image ping (GET request)
2. Apps Script appends registration row to Sheet 1
3. Apps Script finds the slot in the "slots" tab and sets status → "תפוס"
4. Next visitor loads the page → JSONP call fetches only "פנוי" slots → taken slot not shown
5. Admin can reopen a slot via admin panel ("פתח שוב" button)

## Common Tasks

### Change PayBox link
Search for `payboxapp.com` in `index.html` — appears in 2 places (button + text link).

### Add/remove time slots
Use the admin panel at `/admin.html` — no code changes needed.

### Change activity hours display text
Edit line containing `שעות פעילות` in `index.html`.

### Change admin password
In `admin.html`: `const ADMIN_PASSWORD = 'chavah2025'`

### Update Google Apps Script URL
If you redeploy the Apps Script, update `SCRIPT_URL` in both `index.html` and `admin.html`.

### Add a new form field
1. Add HTML input in the form section of `index.html`
2. Add validation in the JS submit handler
3. Add field to `sendToSheet({...})`
4. Update Apps Script `doGet` to read `e.parameter.newfield` and append it
5. Add column header in Google Sheet row 1

### Redeploy Apps Script changes
After any code change in Apps Script:
1. Deploy → Manage deployments → ✏️ pencil
2. Change version to **"New version"**
3. Click Deploy — URL stays the same

## Deployment Flow
```bash
# Edit index.html or admin.html locally
git add . && git commit -m "description" && git push
# GitHub Pages auto-deploys within ~30 seconds
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Slots not loading in dropdown | Check Apps Script is deployed as "Anyone" (not "Anyone with Google account") |
| Slot not disappearing after registration | Check "slots" tab exists in Google Sheet with שעה/סטטוס columns |
| Admin panel shows error | Redeploy Apps Script as new version, then reload |
| Site shows old version | Hard refresh (Ctrl+Shift+R) or open incognito |
| Site 404 | Check GitHub Pages: repo Settings → Pages → Source: main, / |
| Script URL broken | Redeploy Apps Script → update `SCRIPT_URL` in both HTML files |
| Empty slots tab | Delete the "slots" tab → call `?action=getSlots` URL → it recreates with defaults |

## Contacts
- **נתנאל:** 052-595-0524
- **צפריר:** 052-529-9868
