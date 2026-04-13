# Google Apps Script (GAS) 程式碼

請將以下程式碼複製並貼上到您的 Google Apps Script 專案中。
這個腳本會讀取試算表中的所有工作表資料，並將其轉換為 JSON 格式回傳，同時解決 CORS 問題。

```javascript
function doGet(e) {
  // 請將下方替換為您的試算表 ID
  // var spreadsheetId = 'YOUR_SPREADSHEET_ID'; 
  // var ss = SpreadsheetApp.openById(spreadsheetId);
  
  // 如果腳本是綁定在試算表上，可以直接使用：
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheets = ss.getSheets();
  var result = {};

  sheets.forEach(function(sheet) {
    var sheetName = sheet.getName();
    var data = sheet.getDataRange().getValues();
    
    if (data.length <= 1) return; // 跳過空的工作表或只有標題的工作表
    
    var headers = data[0];
    var sheetData = [];

    for (var i = 1; i < data.length; i++) {
      var row = data[i];
      var obj = {};
      for (var j = 0; j < headers.length; j++) {
        var header = headers[j];
        var value = row[j];
        
        // 處理日期格式，確保轉換為字串
        if (value instanceof Date) {
          value = Utilities.formatDate(value, Session.getScriptTimeZone(), "yyyy-MM-dd");
        }
        
        obj[header] = value;
      }
      sheetData.push(obj);
    }
    result[sheetName] = sheetData;
  });

  // 將結果轉換為 JSON 字串
  var jsonString = JSON.stringify(result);
  
  // 建立 TextOutput，並設定 MimeType 為 JSON
  var output = ContentService.createTextOutput(jsonString);
  output.setMimeType(ContentService.MimeType.JSON);
  
  // 解決 CORS 問題（GAS 預設允許跨域請求，但回傳格式必須正確）
  return output;
}
```

### 部署步驟：
1. 在 GAS 編輯器中，點擊右上角的「部署」>「新增部署」。
2. 選擇類型為「網頁應用程式」。
3. 執行身分選擇「我」。
4. 誰可以存取選擇「所有人」。
5. 點擊「部署」，並複製產生的「網頁應用程式網址 (Web App URL)」。
6. 將該網址貼到本儀表板的設定中即可讀取資料。
