# Setup RSVP Google Sheets + Email

Ce projet envoie deja les RSVP par email via FormSubmit.

Ce guide ajoute une copie automatique des reponses dans Google Sheets.

## 1) Creer la feuille Google Sheets

1. Cree une feuille Google Sheets, par exemple: RSVP Cremaillere Lucile.
2. Renomme le premier onglet en RSVP.

## 2) Creer le script Apps Script

1. Dans la feuille: Extensions -> Apps Script.
2. Remplace le contenu par ce script:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('RSVP');
    if (!sheet) {
      sheet = SpreadsheetApp.getActiveSpreadsheet().insertSheet('RSVP');
    }

    var raw = (e && e.postData && e.postData.contents) ? e.postData.contents : '{}';
    var data = JSON.parse(raw);

    var headers = [
      'sent_at',
      'name',
      'attending',
      'guests',
      'dietary',
      'message',
      'source'
    ];

    if (sheet.getLastRow() === 0) {
      sheet.appendRow(headers);
    }

    var row = [
      data.sent_at || new Date().toISOString(),
      data.name || '',
      data.attending || '',
      data.guests || '',
      data.dietary || '',
      data.message || '',
      data.source || ''
    ];

    sheet.appendRow(row);

    return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

## 3) Deployer en Web App

1. Deploy -> New deployment.
2. Type: Web app.
3. Execute as: Me.
4. Who has access: Anyone.
5. Deploy, puis copie l'URL Web App.

## 4) Brancher l'URL dans la page

Dans invitation-trattoria.html, trouve le formulaire RSVP:

```html
<form ... data-sheets-webhook="">
```

Remplace par:

```html
<form ... data-sheets-webhook="TON_URL_WEB_APP_ICI">
```

## 5) Test

1. Ouvre la page invitation.
2. Envoie une reponse test.
3. Verifie:
   - email recu (FormSubmit)
   - ligne ajoutee dans Google Sheets

## Notes

- Le webhook Google Sheets est optionnel: si data-sheets-webhook est vide, seul l'email est envoye.
- La page est statique, donc ce n'est pas une securite backend forte.
