##  Utilitarul nmap (Network Mapper & Port Scanner)

`nmap` este unealta standard de scanare a rețelei și recunoaștere activă. Este folosit pentru a descoperi gazde active într-o rețea, a identifica porturile deschise (TCP/UDP), a detecta versiunile exacte ale serviciilor care rulează și sistemul de operare al țintei.

### Sintaxă de Bază

```bash
nmap [OPȚIUNI] <IP_SAU_DOMENIU>
```
---

#### Principii de Funcționare și Categorii de Scanări

La nivel de transport, Nmap folosește trei mecanisme distincte pentru a determina starea unui port:

| Categorie Scanare | Flag-uri Nmap | Cum funcționează la nivel de pachete | Răspuns: Port DESCHIS | Răspuns: Port ÎNCHIS | Răspuns: Port FILTRAT | Când se folosește |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1. TCP Standard** (bazate pe `SYN`) | `-sT` (Connect)<br>`-sS` (SYN Stealth) | Inițiază conexiunea trimițând un pachet `SYN`: <br>• `-sT` parcurge complet handshake-ul în 3 pași (`SYN` $\rightarrow$ `SYN/ACK` $\rightarrow$ `ACK`). Nu cere root, dar apare în log-uri.<br>• `-sS` răspunde cu `RST` după primirea `SYN/ACK` (half-open). Rapid, discret, necesită `sudo`. | Primește **`SYN/ACK`** | Primește **`RST`** | **Niciun răspuns** (Drop) sau eroare ICMP | • **`-sS`**: Standardul recomandat când ai drepturi de `root`/`sudo`; optim pentru viteză, zgomot redus în log-urile aplicațiilor.<br>• **`-sT`**: Când **nu ai drepturi de root** pe mașina de pe care scanezi sau când ești forțat să treci traficul printr-un proxy/tunel. |
| **2. UDP Stateless** (fără sesiune) | `-sU` (UDP Scan) | Trimite de regulă pachete UDP goale sau payload-uri specifice pe porturile comune (ex. DNS pe 53). Protocolul nu are confirmări (`ACK`). | **Niciun răspuns** (marcat **`open\|filtered`**; devine cert cu `-sV`) | Primește **`ICMP Type 3`** (Port Unreachable) | Niciun răspuns sau eroare ICMP de tip administrativ | Când vrei să descoperi servicii specifice care folosesc UDP (DNS 53, DHCP 67/68, SNMP 161, NTP 123). Din cauza rate-limiting-ului ICMP, se folosește țintit pe porturi cheie sau `--top-ports`. |
| **3. Evasion TCP** (RFC 793 Compliant) | `-sN` (Null)<br>`-sF` (FIN)<br>`-sX` (Xmas) | Trimite pachete anormale **fără flag-ul `SYN`** (Null = niciun flag, FIN = doar `FIN`, Xmas = `FIN`+`PSH`+`URG`) pentru a ocoli firewall-urile stateless: <br>• Dacă portul e închis $\rightarrow$ primește `RST`.<br>• Dacă e deschis $\rightarrow$ ținta ignoră pachetul. | **Niciun răspuns** (marcat **`open\|filtered`**) | Primește **`RST`** | Eroare ICMP (Unreachable) | Când vrei să ocolești firewall-uri stateless simple sau filtre IDS vechi care verifică doar pachetele cu flag `SYN`. **Funcționează doar pe sisteme Unix/Linux**; sistemele Windows răspund mereu cu `RST` indiferent de starea portului. |

> **Atenție la scanările de evaziune pe Windows:** Sistemele de operare Microsoft Windows și unele echipamente Cisco nu respectă standardul RFC 793 și răspund cu `RST` la orice pachet malformat, făcând ca toate porturile să pară incorect închise la scanările `-sN`, `-sF` și `-sX`.

---

### Ghid Rapid de Flag-uri și Opțiuni Nmap

---

#### 1. Tipuri de Scanare de Bază (Probe Types)
* `-sS` — **TCP SYN Scan (Stealth):** Trimite SYN, primește SYN/ACK, trimite RST. Rapid, discret la nivel de aplicație; necesită `sudo`. (Default cu root).
* `-sT` — **TCP Connect Scan:** Finalizează complet handshake-ul în 3 pași prin apel de sistem. Zgomotos în log-uri; nu necesită root. (Default fără root).
* `-sU` — **UDP Scan:** Scanează porturi UDP (fără conexiune/stateless). Lent din cauza rate-limiting-ului ICMP.
* `-sN` — **TCP Null Scan:** Nu setează niciun flag (toate pe 0).
* `-sF` — **TCP FIN Scan:** Setează doar flag-ul FIN.
* `-sX` — **TCP Xmas Scan:** Setează flag-urile FIN, PSH și URG („aprins ca un pom de Crăciun”).

---

#### 2. Scanarea Țintei și Selecția Porturilor (Target & Port Selection)
* `-p <porturi>` — Specifică porturile de scanat:
  * `-p 22,80,443` — Doar porturile enumerate.
  * `-p 1-1000` — Interval de porturi (de la 1 la 1000).
  * `-p-` — Toate cele 65.535 de porturi posibile.
  * `-p U:53,T:22` — Separat pe protocoale (UDP 53, TCP 22).
* `--top-ports <număr>` — Scanează top $N$ cele mai frecvente porturi din baza de date Nmap (ex: `--top-ports 100`).
* `-F` — **Fast mode:** Scanează primele 100 cele mai utilizate porturi (echivalent cu `--top-ports 100`).
* `-r` — Scanează porturile secvențial (de la cel mai mic la cel mai mare), fără a le alege aleatoriu.

---

#### 3. Host Discovery (Descoperirea Gazdelor)
* `-Pn` — **No Ping:** Sare peste verificarea ICMP; tratează toate țintele ca fiind pornite (esențial când firewall-ul blochează ping-ul).
* `-sn` — **Ping Scan (No Port Scan):** Determină doar dacă mașina este activă (Host Discovery), fără a scana porturile.
* `-PR` — **ARP Ping:** Folosește cadre ARP pentru host discovery (comportament implicit dacă ești în aceeași rețea locală).

---

#### 4. Detecție de Servicii și Sistem de Operare
* `-sV` — **Service Version Detection:** Interoghează porturile deschise pentru a determina versiunea exactă a serviciului / aplicației care rulează.
  * `--version-intensity <0-9>` — Nivelul de adâncime al probelor de versiune (implicit 7; 9 este cel mai agresiv).
* `-O` — **OS Detection:** Analizează răspunsurile stivei TCP/IP pentru a identifica sistemul de operare.
* `-A` — **Aggressive Scan:** Activează simultan detecția OS (`-O`), versiunea serviciilor (`-sV`), scripturile de bază (`-sC`) și traceroute (`--traceroute`).

---

#### 5. Ajustarea Vitezei și a Resurselor (Timing & Performance)
* `-T<0-5>` — Profile de viteză și agresivitate pentru timing:
  * `-T0` (Paranoid) / `-T1` (Sneaky) — Scanări extrem de lente, folosite pentru evaziune IDS.
  * `-T2` (Polite) — Încetinește traficul pentru a consuma mai puțină lățime de bandă.
  * `-T3` (Normal) — Profilul implicit.
  * `-T4` (Aggressive) — Scanare rapidă, potrivită pentru rețele stabile/CTF.
  * `-T5` (Insane) — Foarte rapid, dar poate rata porturi pe rețele instabile.
* `--min-rate <număr>` — Forțează Nmap să nu trimită mai puțin de un număr specificat de pachete pe secundă (ex: `--min-rate 1000`).

---

#### 6. Filtrarea și Formatarea Rezultatelor (Output & Filtering)
* `-v` / `-vv` — **Verbosity:** Nivel de detaliere crescut; afișează porturile deschise în consolă pe măsură ce sunt găsite și arată timpul estimat (ETA).
* `--open` — Afișează exclusiv porturile care sunt garantat **deschise** (ignoră stările `closed` și `filtered`).
* `--reason` — Afișează motivul tehnic exact pentru care un port se află într-o stare anume (ex: `syn-ack`, `conn-refused`, `no-response`).
* `--packet-trace` — Afișează fiecare pachet trimis și primit de Nmap la nivel de cablu (util pentru debugging de rețea).
* `-oN <fișier>` — Salvează rezultatul în format standard text (Normal output).
* `-oG <fișier>` — Salvează rezultatul în format **Grepable** (ușor de parsat cu `grep`, `awk`, `cut`).
* `-oX <fișier>` — Salvează rezultatul în format XML (pentru import în alte utilitare).
* `-oA <nume_bază>` — Salvează simultan scanarea în toate cele trei formate de bază (`.nmap`, `.gnmap`, `.xml`).

---

### Comportament Implicit (Default Behavior)

Dacă se rulează comanda simplă fără flag-uri explicite (`nmap <IP>`):

* **Fără privilegii (`nmap <IP>`):** Folosește automat **TCP Connect (`-sT`)**, deoarece un utilizator obișnuit nu are permisiuni pentru socket-uri brute (raw sockets) și este forțat să folosească apelul de sistem `connect()`.
* **Cu privilegii (`sudo nmap <IP>`):** Folosește automat **SYN Stealth (`-sS`)**, fabricând manual pachete TCP pe care le întrerupe înainte de stabilirea completă a conexiunii.
* **Porturi scanate:** În mod implicit verifică doar **top 1.000 cele mai comune porturi**, nu toate cele 65.535.

---

### Tipuri de Scanare TCP / UDP

* **TCP Connect Scan ( -sT ):** Finalizează handshake-ul complet în 3 pași (`SYN` -> `SYN/ACK` -> `ACK`). Este zgomotoasă și lasă urme clare în log-urile aplicațiilor de pe țintă:

```bash
nmap -sT 10.10.10.10
```

* **SYN Half-Open / Stealth Scan ( -sS ):** Trimite `SYN`, primește `SYN/ACK`, dar răspunde cu `RST` (reset) pentru a rupe conexiunea înainte de a fi logată de aplicație. Rapidă și discretă:

```bash
sudo nmap -sS 10.10.10.10
```

* **UDP Scan ( -sU ):** Scanează servicii fără conexiune (ex. DNS 53, SNMP 161, DHCP 67/68). Mult mai lentă deoarece serviciile UDP nu returnează întotdeauna răspunsuri:

```bash
sudo nmap -sU 10.10.10.10
```


### Interpretarea Stării Porturilor în Funcție de Scanare

---

#### 1. Scanările Standard TCP (`-sT` și `-sS`)

Inițiază conexiunea trimițând un pachet `SYN` (Synchronize) și interpretează răspunsul conform mecanismului clasic de 3-way handshake:

| Răspuns primit de la țintă | Stare raportată | Explicație tehnică |
| :--- | :--- | :--- |
| **`SYN/ACK`** | `open` | Serviciul este activ și a acceptat sincronizarea. La `-sT` se trimite `ACK` pentru a finaliza conexiunea; la `-sS` se trimite direct `RST` pentru a tăia conexiunea înainte de logare. |
| **`RST`** (Reset) | `closed` | Portul este închis; sistemul de operare țintă refuză conexiunea conform standardului TCP. |
| **Niciun răspuns** (Drop) / Eroare ICMP | `filtered` | Pachetele au fost blocate sau aruncate de un firewall înainte de a ajunge la serviciu. |

---

#### 2. Scanarea fără Conexiune (`-sU` UDP)

UDP este un protocol stateless (fără handshake, fără confirmări `ACK`). Nmap trimite pachete UDP brute (de regulă goale, sau payload-uri specifice pentru porturi comune precum DNS 53):

| Răspuns primit de la țintă | Stare raportată | Explicație tehnică |
| :--- | :--- | :--- |
| **Niciun răspuns** | `open|filtered` | Serviciul este deschis și ignoră pachetul gol, **sau** un firewall a dat drop pachetului. Necesită `-sV` pentru clarificare prin payload-uri de aplicație. |
| **Răspuns UDP** (Rar) | `open` | Serviciul a recunoscut datele și a trimis un răspuns valid de nivel aplicație. |
| **`ICMP Type 3`** (*Port Unreachable*) | `closed` | Sistemul țintă confirmă prin ICMP că nu ascultă niciun serviciu pe acel port. |
| **Eroare ICMP administrativă** | `filtered` | Firewall-ul a respins explicit pachetul (ex. ICMP Type 3 Codes 1, 2, 9, 10 sau 13). |

---

#### 3. Scanările de Evaziune Firewall (`-sN`, `-sF`, `-sX`)

Nu trimit niciodată pachete cu `SYN`. Trimit pachete TCP anormale pentru a trece neobservate de firewall-urile stateless (care filtrează doar tentativele de inițiere cu `SYN`):

* **`-sN` (Null Scan):** Pachet TCP fără niciun flag setat (toți biții sunt 0).
* **`-sF` (FIN Scan):** Pachet trimis doar cu flag-ul `FIN` aprins.
* **`-sX` (Xmas Scan):** Pachet malformat cu flag-urile `FIN`, `PSH` și `URG` aprinse simultan (arată ca un pom de Crăciun în Wireshark).

**Regula RFC 793:**

| Răspuns primit de la țintă | Stare raportată | Explicație tehnică |
| :--- | :--- | :--- |
| **Niciun răspuns** | `open|filtered` | Conform standardului, un port deschis este obligat să ignore orice pachet nesincronizat primit fără `SYN`/`ACK`. |
| **`RST`** (Reset) | `closed` | Conform standardului, dacă portul este închis, sistemul țintă trebuie să trimită `RST`. |
| **`ICMP Type 3`** (*Unreachable*) | `filtered` | Pachetul a fost blocat de o regulă de firewall pe traseu. |

> **Particularitate Microsoft Windows:** Sistemele Windows și unele echipamente Cisco nu respectă standardul RFC 793; ele răspund cu `RST` la orice pachet malformat, indiferent dacă portul este deschis sau nu. Astfel, scanările `-sN`, `-sF` și `-sX` vor raporta eronat că **toate porturile sunt închise** pe mașinile Windows.


---

---

### Descoperirea Gazdelor / Host Discovery (`-sn`)

Înainte de a scana porturile unei ținte, se verifică ce mașini sunt pornite în rețea (*host discovery* / *ping sweep*).

* **Flag principal:** `-sn` (*No port scan* — oprește scanarea porturilor și raportează doar dacă IP-urile răspund).
* **Mecanism intern de verificare:**
  * **ICMP Echo Request:** Ping clasic de nivel rețea.
  * **TCP SYN (Port 443):** Probă pe HTTPS; răspunsul cu `SYN/ACK` sau `RST` confirmă că gazda este activă.
  * **TCP ACK (Port 80):** Probă pe HTTP; orice mașină activă va răspunde cu `RST` la un `ACK` neașteptat.
  * **ARP Requests:** Folosit prioritar și automat pe rețeaua locală LAN (dacă ești pe același subnet și rulezi cu `sudo`). Este o metodă la nivel Layer 2, imposibil de blocat de firewall-ul sistemului de operare.

#### Sintaxa de Scanare și Notația CIDR

Pentru a viza mai multe mașini simultan, se specifică intervale sau notația de subnet CIDR (`IP/Prefix`), unde prefixul blochează biții de rețea ($32 - \text{Prefix}$ biți liberi pentru gazde):

| Tip Țintă | Exemplu Comandă | Număr Adrese Scanate |
| :--- | :--- | :--- |
| **Subnet `/24` (Cea mai folosită)** | `nmap -sn 192.168.1.0/24` | **256 IP-uri** (`.1` la `.254` utile pentru calculatoare) |
| **Rețea extinsă `/16`** | `nmap -sn 10.0.0.0/16` | **65.536 IP-uri** (pentru corporații sau infrastructuri mari) |
| **Interval cu cratimă** | `nmap -sn 192.168.1.1-50` | **50 IP-uri** (de la `.1` până la `.50` consecutiv) |
| **Gazdă unică (Single Host)** | `nmap -sn 192.168.1.10` sau `/32` | **1 IP** |

> **Calcul rapid adrese:** $\text{IP-uri totale} = 2^{(32 - \text{Prefix})}$. Cu cât numărul prefixului este mai mare, cu atât rețeaua este mai restrânsă.


---


---

### Motorul de Scriptare Nmap / NSE (Nmap Scripting Engine)

NSE extinde funcționalitatea de bază a Nmap dintr-un simplu scanner de porturi într-un instrument activ de audit, recunoaștere avansată și detectare de vulnerabilități. Scripturile sunt scrise în limbajul **Lua** și se găsesc local în `/usr/share/nmap/scripts/`.

#### Sintaxă de Utilizare

* `-sC` sau `--script=default` — Rulează setul standard de scripturi considerate utile, rapide și fără risc (*safe*).
* `--script=<categorie>` — Rulează toate scripturile dintr-o anumită categorie (ex: `--script=vuln`).
* `--script="<cat1> or <cat2>"` — Combină categorii logice (ex: `--script="safe or discovery"`).
* `--script=<nume-script>` — Rulează un script specific (ex: `--script=http-title`).

#### Categorii Principale de Scripturi

| Categorie | Nivel de Risc | Descriere și Utilizare Practică |
| :--- | :--- | :--- |
| **`safe`** | Minim | Nu afectează stabilitatea țintei și nu consumă resurse majore (ex. extragere titlu pagină HTTP, certificat SSL). |
| **`discovery`** | Redus / Mediu | Interoghează activ serviciile pentru a mapa resurse suplimentare din rețea (ex. enumerare utilizatori/rute prin SNMP). |
| **`vuln`** | Mediu | Scanează serviciile detectate pentru a identifica vulnerabilități cunoscute (CVE-uri specifice). |
| **`auth`** | Mediu | Încearcă ocolirea autentificării sau verificarea credențialelor anonime/implicite (ex. login anonim pe FTP). |
| **`brute`** | Zgomotos | Execută atacuri de tip forță brută folosind dicționare pe formulare de autentificare (SSH, baze de date, FTP). |
| **`intrusive`** | Ridicat | Poate bloca sau prăbuși serviciul investigat (*crash/DoS*); consumă bandă mare și declanșează alerte pe firewall/IDS. |
| **`exploit`** | Maxim | Încearcă exploatarea activă a unei vulnerabilități pentru a livra un payload sau a obține acces neautorizat. |

---

#### Utilizarea Practică a Scripturilor NSE

Scripturile pot fi rulate individual, grupate prin virgulă sau configurate cu parametri suplimentari în funcție de cerințele serviciului testat.

* **Rularea unui singur script:**
  ```bash
  nmap --script=<nume-script> <IP>
  # Exemplu:
  nmap --script=http-fileupload-exploiter 10.10.10.5
  ```

* **Rularea mai multor scripturi simultan:** Se specifică separate prin virgulă (fără spații între ele):
  ```bash
  nmap --script=smb-enum-users,smb-enum-shares <IP>
  ```

* **Transmiterea argumentelor (`--script-args`):** Unele scripturi necesită date suplimentare (căi pe server, fișiere locale, credențiale). Sintaxa este `<nume-script>.<parametru>=valoare`:
  ```bash
  nmap -p 80 --script http-put --script-args http-put.url='/dav/shell.php',http-put.file='./shell.php' <IP>
  ```

* **Manual integrat / Documentație (`--script-help`):** Afișează din terminal descrierea completă a scriptului, categoria din care face parte și argumentele acceptate:
  ```bash
  nmap --script-help <nume-script>
  # Exemplu:
  nmap --script-help http-put
  ```
---

#### Localizarea și Gestionarea Scripturilor NSE

Pe sistemele Linux, scripturile NSE sunt stocate local în directorul `/usr/share/nmap/scripts/`.

* **Fișierul `script.db`:** Un fișier text ASCII aflat în directorul de scripturi care servește drept catalog; mapează fiecare fișier `.nse` la categoriile din care face parte (`default`, `safe`, `vuln`, etc.).

* **Căutarea scripturilor local:**
  * **Filtrare cu `grep` în `script.db` (recomandat):**
    ```bash
    # Căutare după serviciu / protocol (ex: ftp):
    grep "ftp" /usr/share/nmap/scripts/script.db

    # Căutare după categorie specifică:
    grep '"safe"' /usr/share/nmap/scripts/script.db
    ```
  * **Listare directă cu `ls`:**
    ```bash
    ls -l /usr/share/nmap/scripts/*ftp*
    ```

* **Instalarea și actualizarea manuală a scripturilor:**
  Dacă descarci un script extern sau creezi unul propriu (scris în Lua), acesta trebuie salvat în directorul Nmap, urmat de reconstruirea fișierului catalog:
  ```bash
  # 1. Descărcarea scriptului:
  sudo wget -O /usr/share/nmap/scripts/<nume-script>.nse [https://svn.nmap.org/nmap/scripts/](https://svn.nmap.org/nmap/scripts/)<nume-script>.nse

  # 2. Actualizarea bazei de date de scripturi:
  sudo nmap --script-updatedb
  ```

---


### Firewall Evasion & IDS Bypassing

Dispozitivele de filtrare a traficului (Firewalls, IDS/IPS) pot bloca sau detecta scanările Nmap standard. Următoarele opțiuni permit ocolirea sau identificarea acestora.

* **Scanare fără ping (`-Pn`):**
  * Ignoră faza de *Host Discovery* (ping ICMP) și presupune că ținta este activă.
  * **Caz de utilizare:** Esențial împotriva sistemelor Windows sau serverelor care filtrează cererile ICMP prin firewall implicit (evită marcarea țintei ca fiind *dead/offline*).
  ```bash
  nmap -Pn <IP>
  ```

* **Fragmentarea pachetelor (`-f`):**
  * Împarte anteturile și datele TCP în mici fragmente de 8 octeți (după antetul IP).
  * **Caz de utilizare:** Împiedică firewall-urile vechi sau IDS-urile bazate pe semnături simple să analizeze pachetul complet dintr-o singură privire.
  ```bash
  nmap -f <IP>
  ```

* **Setarea manuală a MTU (`--mtu`):**
  * Alternativă la `-f` pentru a specifica o dimensiune fixă pentru Maximum Transmission Unit.
  * **Regulă:** Valoarea trebuie să fie obligatoriu un multiplu de 8.
  ```bash
  nmap --mtu 16 <IP>
  ```

* **Întârzierea pachetelor (`--scan-delay`):**
  * Introduce o pauză între pachetele trimise pentru a preveni declanșarea regulilor de *rate-limiting* sau detecția anomaliilor de volum în rețele instabile.
  ```bash
  nmap --scan-delay 200ms <IP>
  ```

* **Detectarea firewall-ului prin Checksum invalid (`--badsum`):**
  * Trimite deliberat pachete cu checksum TCP/UDP/IP eronat.
  * O stivă reală a unui OS ignoră complet pachetul (*drop*). Dacă primești un răspuns (ex: `RST`), cel mai probabil răspunsul vine de la un firewall sau IDS inline neatent la validitatea checksum-ului.
  ```bash
  nmap --badsum <IP>
  ```

* **Limitarea retransmisiilor ( `--max-retries` ):**
  * Limitează de câte ori retrimite Nmap un pachet de probă către un port care nu răspunde.
  * **Caz de utilizare:** Crește considerabil viteza scanării pe mașini protejate de firewall-uri care dau *drop* la pachete și reduce volumul de trafic suspect în rețea.

```bash
nmap --max-retries 1 <IP>
```

* **Abandonarea automată a gazdelor lente ( `--host-timeout` ):**
  * Specifică timpul maxim alocat scanării unui singur IP înainte de a renunța la el.
  * **Caz de utilizare:** Previne blocarea procesului de scanare pe ținte extrem de lente, protejate de filtre stricte sau cu pierderi masive de pachete.

```bash
nmap --host-timeout 15m <IP>
```
