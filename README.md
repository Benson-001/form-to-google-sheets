## Google Apps Script for Spreadsheet Integration
This script is used to handle form submissions and store the data in a Google Spreadsheet. The script performs the following tasks:

1. **Initial Setup**: Set up the script properties with the active spreadsheet ID.
2. **Handle POST Requests**: When a POST request is received, the script locks the sheet, inserts a new row with the form data, and returns a success message.

### Code

```javascript
var sheetName = 'Sheet1';
var scriptProp = PropertiesService.getScriptProperties();

function intialSetup() {
  var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  scriptProp.setProperty('key', activeSpreadsheet.getId());
}

function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    var doc = SpreadsheetApp.openById(scriptProp.getProperty('key'));
    var sheet = doc.getSheetByName(sheetName);

    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow() + 1;

    var newRow = headers.map(function(header) {
      return header === 'timestamp' ? new Date() : e.parameter[header];
    });

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow]);

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (e) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}

## Input your web app URL
## Web App URL Integration

To set up your form to submit data to Google Sheets, follow these steps:

1. **Input your web app URL**: Open the file named `index.html`.
2. **Replace the script URL**: On line 12, replace `<SCRIPT URL>` with your script URL:

   ```html
   <form name="submit-to-google-sheet">
     <input name="email" type="email" placeholder="Email" required>
     <button type="submit">Send</button>
   </form>

   <script>
     const scriptURL = '<SCRIPT URL>'
     const form = document.forms['submit-to-google-sheet']

     form.addEventListener('submit', e => {
       e.preventDefault()
       fetch(scriptURL, { method: 'POST', body: new FormData(form)})
         .then(response => console.log('Success!', response))
         .catch(error => console.error('Error!', error.message))
     })
   </script>


