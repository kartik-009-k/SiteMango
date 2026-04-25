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
   - `Project Type`
   - `Project Details`
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

    sheet.appendRow([
      new Date(),
      data.fullName || '',
      data.companyEmail || '',
      data.projectType || '',
      data.projectDetails || ''
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, message: String(error) }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

### Important note

If you redeploy Apps Script or create a new deployment version, update the `sheetEndpoint` URL in `index.html`.
