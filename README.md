# SiteMango

This repository is configured for automatic deployment to **GitHub Pages**.

## Deploy flow

- Push to the `main` branch to trigger deployment.
- Or run the workflow manually from **Actions → Deploy static site → Run workflow**.

The workflow lives at:

- `.github/workflows/deploy.yml`

## Local preview

Open `index.html` directly in a browser, or run a simple local server:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000`.

## Connect the Flagship Project form to Google Sheets (Apps Script)

1. Create a Google Sheet and name the first tab `Leads`.
2. Add these headers in row 1:
   - `Timestamp`
   - `Full Name`
   - `Company Email`
   - `Project Details`
   - `Client IP (optional)`
3. In that spreadsheet, open **Extensions → Apps Script**.
4. Replace the default code with the script below.
5. Click **Deploy → New deployment → Web app**:
   - Execute as: **Me**
   - Who has access: **Anyone**
6. Copy the Web App URL and paste it into `index.html` as `sheetEndpoint`.

```javascript
function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Leads');
    const data = JSON.parse(e.postData.contents || '{}');

    // Honeypot spam check
    if ((data.website || '').trim() !== '') {
      return jsonResponse({ ok: false, reason: 'Spam check failed.' });
    }

    // Required field check
    if (!data.fullName || !data.companyEmail || !data.projectDetails) {
      return jsonResponse({ ok: false, reason: 'Please complete all required fields.' });
    }

    // Duplicate cooldown check (same email, last 2 minutes)
    const values = sheet.getDataRange().getValues();
    const now = Date.now();
    const cooldownMs = 2 * 60 * 1000;
    const email = String(data.companyEmail).toLowerCase().trim();

    for (let i = values.length - 1; i > 0; i--) {
      const rowTimestamp = new Date(values[i][0]).getTime();
      const rowEmail = String(values[i][2] || '').toLowerCase().trim();
      if (rowEmail === email && now - rowTimestamp < cooldownMs) {
        return jsonResponse({ ok: false, reason: 'Please wait before submitting again.' });
      }
      if (now - rowTimestamp >= cooldownMs) {
        break;
      }
    }

    const clientIp = e?.parameter?.ip || '';

    sheet.appendRow([
      new Date(),
      data.fullName || '',
      data.companyEmail || '',
      data.projectDetails || '',
      clientIp
    ]);

    return jsonResponse({ ok: true });
  } catch (error) {
    return jsonResponse({ ok: false, reason: String(error) });
  }
}

function jsonResponse(payload) {
  return ContentService
    .createTextOutput(JSON.stringify(payload))
    .setMimeType(ContentService.MimeType.JSON);
}
```

### Important notes

- If you redeploy Apps Script or create a new deployment version, update the `sheetEndpoint` URL in `index.html`.
- Update social preview metadata (`canonical`, `og:url`, `og:image`, `twitter:image`) in `index.html` before production deploy.
