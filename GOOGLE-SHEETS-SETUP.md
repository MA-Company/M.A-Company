# Enquiry Forms → Google Sheets (from scratch)

This guide sets up the **two** Google Sheets that receive enquiries from the website
contact form, with automatic email alerts. No coding experience needed — you copy,
paste, and click.

You will build:

| Sheet | Receives | Emails sent to |
|-------|----------|----------------|
| **Main** | United States + International (and Canada *until* the Canada sheet is live) | `info@macompany.site` |
| **Canada (Global Reach)** | Canada only | `GlobalReachAFJ@gmail.com` **and** `info@macompany.site` |

The website form (`contact.html`) already knows how to send Canada enquiries to one
URL and everything else to another. You just create the two Sheets, copy their two
web-app URLs, and paste them into the form. That's it.

> **Good to know:** until you paste the live Canada URL, Canada enquiries automatically
> fall back to the Main sheet, so **no enquiry is ever lost** while you set this up.

---

## Part A — Create the MAIN sheet (US + International)

> The Main script URL is already live in the site. Follow these steps only if you want
> to rebuild it from scratch, or to understand how it works. If you just want Canada
> working, skip to **Part B**.

### Step 1 — Create the spreadsheet
1. Go to <https://sheets.google.com> and sign in with the account that should **own**
   this data (e.g. the M.A. & Company Google account).
2. Click **Blank spreadsheet**.
3. Rename it (top-left) to something like **`MA Company — Enquiries (Main)`**.

### Step 2 — Open the script editor
1. In the sheet, click **Extensions ▸ Apps Script**.
2. A new tab opens with a file called `Code.gs` containing an empty `myFunction()`.
3. Delete everything in that file.

### Step 3 — Paste the script
1. Copy the **entire** script from the [The Script](#the-script) section at the bottom
   of this file.
2. Paste it into `Code.gs`.
3. At the top, leave the recipients line as the Main address:
   ```js
   var RECIPIENTS = 'info@macompany.site';
   ```
4. Click the **💾 Save** icon (or `Ctrl/Cmd + S`).

### Step 4 — Deploy as a Web app
1. Click **Deploy ▸ New deployment** (top-right).
2. Click the **⚙️ gear** next to *Select type* → choose **Web app**.
3. Fill in:
   - **Description:** `Enquiry endpoint v1`
   - **Execute as:** **Me** (your email)
   - **Who has access:** **Anyone**  ← important, the website is an anonymous visitor
4. Click **Deploy**.

### Step 5 — Authorize it (first time only)
1. Google asks you to authorize. Click **Authorize access**.
2. Choose your account. You'll see **"Google hasn't verified this app"** — this is
   normal for your own script. Click **Advanced ▸ Go to … (unsafe)**, then **Allow**.
   *(It's "unverified" only because it's your private script, not a public product.)*

### Step 6 — Copy the Web app URL
1. After deploying you'll see a **Web app URL** ending in **`/exec`**. Copy it.
   It looks like:
   `https://script.google.com/macros/s/AKfycb..................../exec`
2. Keep it handy for **Part C**.

---

## Part B — Create the CANADA sheet (Global Reach)

Repeat the **exact same Steps 1–6** in a **separate** spreadsheet, with two differences:

- **Name** the spreadsheet **`Global Reach — Canada Enquiries`**.
- In **Step 3**, change the recipients line to **both** addresses (comma-separated):
  ```js
  var RECIPIENTS = 'GlobalReachAFJ@gmail.com, info@macompany.site';
  ```

Everything else is identical. At the end you'll have a **second** `/exec` URL — this is
your **Canada URL**.

> **Whose account owns it?** Either works. You can create this sheet under the M.A. &
> Company account (so you keep all data in one place) or under the Global Reach Google
> account. The email alerts go to **both** addresses regardless of who owns the file —
> ownership only decides who can open the spreadsheet itself.

---

## Part C — Connect the two URLs to the website

Open **`contact.html`** and find this block near the bottom (inside `<script>`):

```js
const MAIN_SCRIPT_URL   = 'https://script.google.com/macros/s/AKfycb..../exec';
const CANADA_SCRIPT_URL = 'CANADA_SCRIPT_URL_PLACEHOLDER';
```

1. Replace `MAIN_SCRIPT_URL` with your **Part A** `/exec` URL *(already done if the main
   script was already live)*.
2. Replace `CANADA_SCRIPT_URL_PLACEHOLDER` with your **Part B** `/exec` URL.

Save the file and re-upload it to your host. Done — Canada enquiries now go to the
Global Reach sheet and email both addresses; everything else goes to the Main sheet.

> Keep the quotes. Each URL must be wrapped in `'...'` and end in `/exec`
> (not `/dev`).

---

## Part D — Test it

1. Open the live site's **Request an Enquiry** page.
2. Submit one test enquiry with **Market = United States** → check the **Main** sheet
   gets a new row and `info@macompany.site` gets an email.
3. Submit another with **Market = Canada** → check the **Canada** sheet gets the row and
   **both** `GlobalReachAFJ@gmail.com` and `info@macompany.site` get an email.

The first submission to each sheet automatically creates a tab called **`Enquiries`**
with a bold header row. You don't need to set up columns by hand.

---

## Part E — Changing the script later (important!)

If you ever edit the Apps Script code, you must **publish a new version** or the website
keeps running the old code:

1. **Deploy ▸ Manage deployments**.
2. Click the **✏️ pencil** (Edit) on your existing deployment.
3. Set **Version** to **New version**, then **Deploy**.

Doing it this way keeps the **same URL**, so you don't need to touch `contact.html` again.
*(Only "New deployment" creates a brand-new URL — avoid that unless you want to.)*

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| Form says "Something went wrong" | The `/exec` URL is wrong, or **Who has access** isn't **Anyone**. Re-check Step 4. |
| Row appears but no email | Open Apps Script ▸ **Executions** to see the error. Gmail sends ~100 emails/day max on free accounts; check the recipient address spelling. |
| Nothing happens at all | Make sure the URL ends in `/exec`, not `/dev`. The `/dev` URL only works while you're logged in. |
| Changed the code but behaviour is the same | You didn't publish a new version — see **Part E**. |
| Columns are out of order / blank | The header names come from `FIELDS` in the script. Don't rename the sheet columns by hand; if you change fields, change them in both the script and the website form. |

---

## Field reference (sheet column ↔ form field)

The website sends these field names; the script maps them to columns in this order:

| Column | Form field name |
|--------|-----------------|
| Timestamp | *(added automatically)* |
| Full Name | `fullName` |
| Company | `company` |
| Email | `email` |
| Phone | `phone` |
| Market | `market` |
| Products | `finalProductList` |
| Quantity | `quantity` |
| Packing | `packingType` |
| Message | `message` |
| Heard About | `heardAbout` |

---

## The Script

Paste this whole block into `Code.gs`. The **only** line you change between the Main and
Canada sheets is `RECIPIENTS` (see Parts A & B).

```js
/**
 * M.A. & Company — Enquiry form → Google Sheet + email alert.
 * Paste into Extensions ▸ Apps Script for the sheet that should receive these enquiries.
 *
 * ── CONFIGURE PER SHEET ───────────────────────────────────────────────────
 *   MAIN sheet (US + International):  'info@macompany.site'
 *   CANADA sheet (Global Reach):      'GlobalReachAFJ@gmail.com, info@macompany.site'
 */
var RECIPIENTS  = 'info@macompany.site';        // who gets the alert (comma-separated)
var SHEET_NAME  = 'Enquiries';                  // tab name created inside the spreadsheet
var EMAIL_LABEL = 'M.A. & Company Website';     // appears in the email subject

// Columns, in order. Keep the second value in sync with the website form field names.
var FIELDS = [
  ['Full Name',   'fullName'],
  ['Company',     'company'],
  ['Email',       'email'],
  ['Phone',       'phone'],
  ['Market',      'market'],
  ['Products',    'finalProductList'],
  ['Quantity',    'quantity'],
  ['Packing',     'packingType'],
  ['Message',     'message'],
  ['Heard About', 'heardAbout']
];

function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.waitLock(30000); // stop two submissions writing to the same row at once
  try {
    var ss    = SpreadsheetApp.getActiveSpreadsheet();
    var sheet = ss.getSheetByName(SHEET_NAME) || ss.insertSheet(SHEET_NAME);

    // First run: create the bold, frozen header row.
    if (sheet.getLastRow() === 0) {
      var header = ['Timestamp'].concat(FIELDS.map(function (f) { return f[0]; }));
      sheet.appendRow(header);
      sheet.getRange(1, 1, 1, header.length).setFontWeight('bold');
      sheet.setFrozenRows(1);
    }

    var p   = (e && e.parameter) ? e.parameter : {};
    var now = new Date();
    var row = [now].concat(FIELDS.map(function (f) { return p[f[1]] || ''; }));
    sheet.appendRow(row);

    sendNotification(p, now);

    return ContentService
      .createTextOutput(JSON.stringify({ result: 'success' }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'error', message: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}

function sendNotification(p, now) {
  var who     = p.company || p.fullName || 'Website visitor';
  var subject = 'New ' + (p.market || '') + ' Enquiry — ' + who + ' [' + EMAIL_LABEL + ']';
  var lines   = FIELDS.map(function (f) { return f[0] + ': ' + (p[f[1]] || '—'); });
  lines.unshift('Received: ' + now);
  MailApp.sendEmail({
    to: RECIPIENTS,
    subject: subject,
    replyTo: p.email || RECIPIENTS,   // press Reply to answer the customer directly
    body: lines.join('\n')
  });
}

// Lets you open the /exec URL in a browser to confirm it's deployed.
function doGet() {
  return ContentService.createTextOutput('M.A. & Company enquiry endpoint is live.');
}
```
