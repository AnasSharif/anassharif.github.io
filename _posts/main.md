---
title: Main
date: 2022-10-19 12:05:22 +/-TTTT
categories: [google-sheet, localizaions]
tags: [google,google-sheet]     # TAG names should always be lowercase
---
# Google Apps Script for Android & IOS Translation

Normally we get `strings.xml` and `Localizable.strings` files from our translation team, but today was one of those days, where (due to time constraints) we got the source Google Sheet file. We had to generate strings.xml files out of these. This gave me the perfect opportunity to explore [Google Apps Script](https://developers.google.com/apps-script) which you could run on your Google Sheet.

# Google Sheet format for Translations
![Translation Format sample](/assets/translation_sheet_format.png)


The above is the general format of the sheet. First row represents a language codes, and first column represent keys. So in the above case, for language en, the Android `strings.xml` file and for iOS `Localizable.strings` folder should be as follows:

### Android Strings
```XML
<?xml version="1.0" encoding="UTF-8"?>
<resources>
    <string name="tutorial">Tutorial</string>
    <string name="share">Share</string>
</resources>
```
### IOS String

```Strings
"tutorial" = "Tutorial";
"share" = "Share";
```
# Adding Google Apps Script

To add the script, in your Google Sheet, click on `Tools` -> `Script editor`. You can then add your JS code in the Editor. In our case, we will add a menu “Translations” to the menu bar, and then a sub-menu under it called “Android” and “IOS”. Clicking on thess submenus will generate a “Translations-1” `(1 mean Android 2 for iOS) `folder and sub folders like “values-en”, “values-bs” For iso “en.lproj”, “de.lproj”, etc… each containing the corresponding `strings.xml` or `Localizable.strings` file. We can then download the folder “Translations” and use it in our Android or IOS project.

## The Code

For the full working script take a look at this [Gist](https://gist.github.com/AnasSharif/0241dc4199941b828c612b3301eb9fd2)

The `onOpen()` function adds the menu to the bar, along with the JS function to invoke when the menu is clicked.

```javascript
function onOpen() {
  SpreadsheetApp.getUi()
      .createMenu('Export Translations')
      .addItem('Android', 'exportAndroidTranslations')
      .addItem('IOS', 'exportIosTranslations')
      .addToUi();
}
```
The `exportAndroidTranslations()` or  `exportIosTranslations()` functions call main function `exportTranslations` with platfrom `Int` param then this uses other functions to get the data from the Sheet, parses the data, constructs the `strings.xml` or `Localizable.strings` file’s content, and then uploads the file (along with the values or en.lproj folder) to Google Drive.

```javascript
function exportTranslations(platform){
   const ss = SpreadsheetApp.getActiveSpreadsheet();
   const sheet = ss.getActiveSheet();
   //Get first header/codes
   const headers = getHeaders(sheet);
   values = sheet.getDataRange().getValues()
   //Get All keys from column default is 0/First
   const keys = getColumn(0);

   const folder = DriveApp.createFolder(`Translations-${platform}`);

   for (let i = 1; i < headers.length; i++) {
      const langCode = headers[i];
      // 1 is Android now load translations
      if (platform == 1){
        const subFolder = folder.createFolder(`values-${langCode.toLowerCase()}`);
         subFolder.createFile("strings.xml", constructAndroidTranslation(keys, getColumn(i)));
      } else if (platform == 2){
        const subFolder = folder.createFolder(`${langCode.toLowerCase()}.lproj`);
         subFolder.createFile("Localizable.strings", constructIOSTranslation(keys, getColumn(i)));
      }

   }
}
```
The `DriveApp` is used to create folders and files in `Google Drive`.

The remaining functions are just to parse the Cells in the sheet and get the data in a proper format. You can take a look at the [Gist](https://gist.github.com/AnasSharif/0241dc4199941b828c612b3301eb9fd2) to know more.


## Conclusion
This is nothing huge. All we did was read a Google Sheet, and processed the data, and uploaded it to Google Drive. But we did it all from Google Sheets, and we can ship the script along with the Sheet itself. That’s neat. I hope this helps you the next time you have some data in a Google Sheet and you have to do something with it. Go crazy!
