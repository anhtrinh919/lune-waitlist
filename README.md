# Lune — Waitlist Landing Page

Static landing page for the Lune waitlist (5 free pilot builds). Single file: `index.html`.
Hosted on GitHub Pages. Design implemented from the Claude Design handoff (Nocturne / Moonlight).

## How signups are captured (current)

The form posts to **FormSubmit** (`https://formsubmit.co/ajax/…`), a free service that emails
every signup to **tuananhtrinh919@gmail.com**. No account needed.

**One-time activation:** the very first signup triggers an activation email from FormSubmit —
click the link in it once, and every signup after that lands in the inbox automatically.

Each signup is also backed up to the visitor's browser `localStorage` (key `lune_signups`) as a
session-level safety net.

## Upgrading to a Google Sheet + auto-email (no third party)

When you want signups written to a Google Sheet **and** an email — with no third-party service —
swap the backend to a Google Apps Script. ~2 minutes, one time:

1. Create a Google Sheet. Top row headers: `Time | Name | Email | What to build`.
2. In the Sheet: **Extensions → Apps Script**. Paste the code below. Save.
3. **Deploy → New deployment → Web app**. Execute as **Me**, access **Anyone**. Copy the Web-app URL.
4. In `index.html`, set `FORM_ENDPOINT` to that URL and change the `fetch` body to send
   `name`, `email`, `idea` as form fields (or keep JSON and read `e.postData` in the script).

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = JSON.parse(e.postData.contents);
  sheet.appendRow([new Date(), data.Name || data.name, data.Email || data.email, data["What to build"] || data.idea || '']);
  MailApp.sendEmail({
    to: 'tuananhtrinh919@gmail.com',
    subject: '🌙 New Lune waitlist signup',
    body: 'Name: ' + (data.Name || data.name) + '\nEmail: ' + (data.Email || data.email) +
          '\nWhat to build: ' + (data["What to build"] || data.idea || '(none)')
  });
  return ContentService.createTextOutput(JSON.stringify({ok: true})).setMimeType(ContentService.MimeType.JSON);
}
```

Note: Apps Script web apps don't return CORS headers, so post with
`fetch(url, { method:'POST', body: JSON.stringify(payload) })` (no custom headers) and treat any
resolved response as success.
