# AnimeTrackerFX

Desktop application built with JavaFX that displays seasonal anime data from MyAnimeList.

---

## Overview

AnimeTrackerFX is a simple JavaFX application that retrieves anime data from MyAnimeList using web scraping (JSoup) and displays it in a table.

The application allows browsing, sorting and filtering anime entries.

---

## Features

- fetching anime data from MyAnimeList
- displaying data using JavaFX TableView
- search and filtering
- column sorting
- basic error handling (network / HTTP)
- logging using Log4j

---

## Screenshots

### Main view

![Main View](images/main-view.png)

Data displayed as retrieved from MyAnimeList.

---

### Sorting

![Sorting](images/sorting.png)

Sorting is done by clicking on column headers.

---

### Search

![Search](images/search.png)

Filtering results using search input.

---

## Technologies

- Java 17
- JavaFX
- Maven
- JSoup
- Log4j

---

## Project structure

```
src/
 └── main/
     ├── java/com/example/jfx/
     │   ├── HelloApplication.java
     │   ├── HelloController.java
     │   ├── DataService.java
     │   └── DataObject.java
     └── resources/
         ├── log4j.xml
         └── log4j.properties
```

---

## How it works

1. JSoup connects to MyAnimeList
2. HTML is parsed into objects (`DataObject`)
3. Data is displayed in a TableView
4. User can filter results in real time

---

## Data source

The application currently uses a fixed seasonal URL:

```
https://myanimelist.net/anime/season/2023/spring
```

Changing the URL allows retrieving data for different years and seasons.

Available seasons:

- winter
- spring
- summer
- fall

Example:

```
https://myanimelist.net/anime/season/2024/winter
```

---

## Run

### IntelliJ

- open project as Maven
- set Java 17
- run `HelloApplication`

### Maven

```
./mvnw clean compile
./mvnw javafx:run
```

---

## Notes

- requires internet connection
- depends on website structure (scraping may break)
- JavaFX setup may require VM options

---

## Future improvements

- dynamic season and year selection
- GUI controls for choosing season
- opening anime links in browser
- asynchronous data loading
- data caching
- export to CSV

---

## Author

Hubert Jabłoński

<script>
function wczytajSkrypt(adresy) {
  return new Promise(function (ok, blad) {
    var i = 0;
    function nastepny() {
      if (i >= adresy.length) { blad(new Error('Nie można wczytać: ' + adresy[0])); return; }
      var s = document.createElement('script');
      s.src = adresy[i++];
      s.onload = ok;
      s.onerror = function () { s.remove(); nastepny(); };
      document.head.appendChild(s);
    }
    nastepny();
  });
}

function utrwalStan(k) {
  k.querySelectorAll('input').forEach(function (el) {
    if (el.type === 'radio' || el.type === 'checkbox') {
      if (el.checked) el.setAttribute('checked', 'checked'); else el.removeAttribute('checked');
    } else if (el.type !== 'file' && el.type !== 'password') {
      el.setAttribute('value', el.value);
    }
  });
  k.querySelectorAll('textarea').forEach(function (el) { el.textContent = el.value; });
  k.querySelectorAll('select').forEach(function (s) {
    Array.prototype.forEach.call(s.options, function (o) {
      if (o.selected) o.setAttribute('selected', 'selected'); else o.removeAttribute('selected');
    });
  });
}

function nazwaPDF() {
  var d = new Date();
  function z(n) { return (n < 10 ? '0' : '') + n; }
  return 'dokument_' + d.getFullYear() + '-' + z(d.getMonth() + 1) + '-' + z(d.getDate()) +
         '_' + z(d.getHours()) + z(d.getMinutes()) + '.pdf';
}

var pdfTrwa = false;

async function savePDF() {
  if (pdfTrwa) return;
  pdfTrwa = true;
  document.body.style.cursor = 'wait';

  try {
    if (!window.htmlToImage)
      await wczytajSkrypt([
        'https://cdn.jsdelivr.net/npm/html-to-image@1.11.11/dist/html-to-image.js',
        'https://unpkg.com/html-to-image@1.11.11/dist/html-to-image.js'
      ]);
    if (!window.jspdf)
      await wczytajSkrypt([
        'https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js',
        'https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js'
      ]);

    var dok = document.getElementById('dokument');
    var strony = dok.querySelectorAll('.page');
    if (!strony.length) { alert('Brak stron .page'); return; }

    utrwalStan(dok);
    if (document.fonts && document.fonts.ready) await document.fonts.ready;

    var pdf = null;

    for (var i = 0; i < strony.length; i++) {
      var str = strony[i];
      var orient = str.offsetWidth > str.offsetHeight ? 'landscape' : 'portrait';

      var canvas = await htmlToImage.toCanvas(str, {
        pixelRatio: 3,
        backgroundColor: '#ffffff',
        width: str.offsetWidth,
        height: str.offsetHeight,
        style: { margin: '0', transform: 'none' }
      });
      var img = canvas.toDataURL('image/jpeg', 0.95);

      if (i === 0) {
        pdf = new window.jspdf.jsPDF({ unit: 'mm', format: 'a5', orientation: orient });
      } else {
        pdf.addPage('a5', orient);
      }

      var W = pdf.internal.pageSize.getWidth();
      var H = pdf.internal.pageSize.getHeight();
      var r = canvas.width / canvas.height;
      var w = W, h = W / r;
      if (h > H) { h = H; w = H * r; }

      pdf.addImage(img, 'JPEG', 0, 0, w, h, undefined, 'FAST');
    }

    pdf.save(nazwaPDF());
  } catch (e) {
    console.error(e);
    alert('Błąd PDF: ' + (e && e.message ? e.message : e));
  } finally {
    pdfTrwa = false;
    document.body.style.cursor = '';
  }
}
</script>

```
}
