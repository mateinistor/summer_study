# HTTP Headers

Antetele (Headers) reprezintă metadate suplimentare pe care clientul și serverul le transmit în interiorul cererilor și răspunsurilor HTTP. Deși din punct de vedere tehnic protocolul nu cere obligatoriu antete pentru fiecare cerere simplă, randarea și funcționarea corectă a unui site modern depind în totalitate de ele.

---

## 1. Request Headers (Antete de Cerere)

Aceste antete sunt trimise de către client (browser, instrumente CLI, scripturi) către server pentru a oferi context despre mediul de rulare și datele transmise.

| Header | Rol & Descriere |
| :--- | :--- |
| **`Host`** | Specifică domeniul/site-ul solicitat. Este esențial pe serverele care găzduiesc mai multe site-uri pe același IP (*Virtual Hosting*); fără el, serverul servește doar site-ul implicit. |
| **`User-Agent`** | Identifică aplicația client, sistemul de operare și versiunea de browser. Permite serverului să adapteze conținutul sau layout-ul HTML/CSS/JS în funcție de capabilitățile browserului. |
| **`Content-Length`** | Specifică dimensiunea în octeți (*bytes*) a corpului cererii (`Body`). Este folosit când se trimit formulare sau fișiere către server, asigurând că pachetul a fost recepționat complet. |
| **`Accept-Encoding`** | Indică algoritmii de compresie suportați de client (ex: `gzip`, `br`, `deflate`), permițând serverului să comprime datele pentru transfer mai rapid. |
| **`Cookie`** | Trimite înapoi către server date de stare, identificatori de sesiune sau preferințe stocate anterior în browser. |

---

## 2. Response Headers (Antete de Răspuns)

Aceste antete sunt returnate de către serverul web către client alături de codul de stare și resursa cerută.

| Header | Rol & Descriere |
| :--- | :--- |
| **`Set-Cookie`** | Transmite browserului un identificator sau o bucată de date pe care acesta trebuie să o salveze local și să o trimită înapoi la următoarele cereri în antetul `Cookie`. |
| **`Cache-Control`** | Dictează regulile de salvare în cache (ex: cât timp are voie browserul să păstreze resursa local înainte de a cere o copie nouă de pe server). |
| **`Content-Type`** | Specifică tipul MIME al fișierului returnat (`text/html`, `application/json`, `image/png`). Browserul folosește acest antet pentru a ști cum să interpreteze și să randeze conținutul. |
| **`Content-Encoding`** | Specifică algoritmul de compresie utilizat efectiv de server pentru a împacheta datele trimise pe rețea (ex: `gzip`). |
