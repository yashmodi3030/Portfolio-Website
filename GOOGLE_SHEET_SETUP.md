# Google Sheets Newsletter Setup Guide

This guide explains how to connect your portfolio website's newsletter form to a Google Sheet in under 2 minutes.

---

### Step 1: Create a Google Sheet
1. Go to [sheets.google.com](https://sheets.google.com) and create a new blank spreadsheet.
2. Name it something like **"Newsletter Subscribers"**.
3. In row 1, set up the headers:
   - Column A: `Timestamp`
   - Column B: `Email`
   - Column C: `Status`

---

### Step 2: Add the Google Apps Script
1. In your Google Sheet, click **Extensions** > **Apps Script**.
2. Delete any code in the editor (`Code.gs`) and paste this snippet:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = {};
    
    if (e.postData && e.postData.contents) {
      try {
        data = JSON.parse(e.postData.contents);
      } catch (err) {
        data = {};
      }
    }
    
    var email = data.email || (e.parameter ? e.parameter.email : "");
    var timestamp = new Date();
    
    if (email) {
      sheet.appendRow([timestamp, email, "Subscribed"]);
      return ContentService.createTextOutput(JSON.stringify({ result: "success", email: email }))
        .setMimeType(ContentService.MimeType.JSON);
    } else {
      return ContentService.createTextOutput(JSON.stringify({ result: "error", message: "No email provided" }))
        .setMimeType(ContentService.MimeType.JSON);
    }
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ result: "error", error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Click the disk icon (💾 **Save**) or press `Ctrl + S`.

---

### Step 3: Deploy as Web App
1. At the top right of the Apps Script editor, click **Deploy** > **New deployment**.
2. Click the gear icon (⚙️) next to *Select type* and choose **Web app**.
3. Fill in the fields:
   - **Description**: `Newsletter Webhook`
   - **Execute as**: `Me (your email)`
   - **Who has access**: **`Anyone`** *(Important: Must be set to Anyone so visitors on your website can submit without logging into your Google account).*
4. Click **Deploy**.
5. Grant permissions if prompted (click *Advanced* > *Go to Untitled project (unsafe)* > *Allow*).
6. Copy the **Web App URL** (it looks like `https://script.google.com/macros/s/AKfycb.../exec`).

---

### Step 4: Paste into your Website
1. Open `index.html`.
2. Near line 748, locate:
   ```javascript
   const GOOGLE_SHEET_WEBAPP_URL = "";
   ```
3. Paste your Web App URL between the quotes:
   ```javascript
   const GOOGLE_SHEET_WEBAPP_URL = "https://script.google.com/macros/s/AKfycb.../exec";
   ```
4. Save the file. Done!

---

> [!NOTE]
> Even before or without setting up the Google Sheet URL, your website automatically saves all subscriber emails with timestamps in the browser's `localStorage` under the key `'yash_newsletter_subscribers'`. You can view them anytime in the browser developer console by typing:
> ```javascript
> console.table(JSON.parse(localStorage.getItem('yash_newsletter_subscribers')))
> ```
