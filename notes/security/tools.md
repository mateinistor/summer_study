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


---

## 2. Utilitarul nmap (Network Mapper & Port Scanner)

`nmap` este unealta standard de scanare a rețelei și recunoaștere activă. Este folosit pentru a descoperi gazde active într-o rețea, a identifica porturile deschise (TCP/UDP), a detecta versiunile exacte ale serviciilor care rulează și sistemul de operare al țintei.

### Sintaxă de Bază

```bash
nmap [OPȚIUNI] <IP_SAU_DOMENIU>
```
---

#### Principii de Funcționare și Categorii de Scanări

La nivel de transport, Nmap folosește trei mecanisme distincte pentru a determina starea unui port:

| Categorie Scanare | Flag-uri Nmap | Cum funcționează la nivel de pachete | Răspuns: Port DESCHIS | Răspuns: Port ÎNCHIS | Răspuns: Port FILTRAT |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. TCP Standard** (bazate pe `SYN`) | `-sT` (Connect)<br>`-sS` (SYN Stealth) | Inițiază conexiunea trimițând un pachet `SYN`: <br>• `-sT` parcurge complet handshake-ul în 3 pași (`SYN` $\rightarrow$ `SYN/ACK` $\rightarrow$ `ACK`). Nu cere root, dar apare în log-uri.<br>• `-sS` răspunde cu `RST` după primirea `SYN/ACK` (half-open). Rapid, discret, necesită `sudo`. | Primește **`SYN/ACK`** | Primește **`RST`** | **Niciun răspuns** (Drop) sau eroare ICMP |
| **2. UDP Stateless** (fără sesiune) | `-sU` (UDP Scan) | Trimite de regulă pachete UDP goale sau payload-uri specifice pe porturile comune (ex. DNS pe 53). Protocolul nu are confirmări (`ACK`). | **Niciun răspuns** (marcat **`open\|filtered`**; devine cert cu `-sV`) | Primește **`ICMP Type 3`** (Port Unreachable) | Niciun răspuns sau eroare ICMP de tip administrativ |
| **3. Evasion TCP** (RFC 793 Compliant) | `-sN` (Null)<br>`-sF` (FIN)<br>`-sX` (Xmas) | Trimite pachete anormale **fără flag-ul `SYN`** (Null = niciun flag, FIN = doar `FIN`, Xmas = `FIN`+`PSH`+`URG`) pentru a ocoli firewall-urile stateless: <br>• Dacă portul e închis $\rightarrow$ primește `RST`.<br>• Dacă e deschis $\rightarrow$ ținta ignoră pachetul. | **Niciun răspuns** (marcat **`open\|filtered`**) | Primește **`RST`** | Eroare ICMP (Unreachable) |

> **Atenție la scanările de evaziune pe Windows:** Sistemele de operare Microsoft Windows și unele echipamente Cisco nu respectă standardul RFC 793 și răspund cu `RST` la orice pachet malformat, făcând ca toate porturile să pară incorect închise la scanările `-sN`, `-sF` și `-sX`.



### Opțiuni și Flag-uri Frecvente

* **Scanare fără Ping ( -Pn ):** Presupune că toate gazdele sunt active și sare peste etapa de verificare prin ping (esențial pentru mașini de laborator protejate de firewall):

```bash
nmap -Pn 10.10.10.10
```

* **Detectarea versiunilor exacte de servicii ( -sV ):** Trimite interogări specifice către porturile deschise pentru a afla software-ul exact și versiunea:

```bash
nmap -sV 10.10.10.10
```

* **Rularea scripturilor implicite de enumerare ( -sC ):** Activează setul de bază de scripturi automate NSE (Nmap Scripting Engine) pentru a detecta vulnerabilități și configurări nesigure:

```bash
nmap -sC 10.10.10.10
```

* **Specificarea porturilor sau scanare completă ( -p ):** Limitează scanarea la porturi selectate sau verifică toate cele 65.535 de porturi:

```bash
nmap -p 21,80,443,3389 10.10.10.10
```

```bash
nmap -p- 10.10.10.10
```

* **Scanare rapidă și discretă SYN Stealth ( -sS ):** Trimite pachete SYN fără a finaliza conexiunea 3-way handshake (necesită drepturi de administrator):

```bash
sudo nmap -sS 10.10.10.10
```

* **Ajustarea vitezei și agresivității scanării ( -T4 ):** Optimizează timpii de așteptare pentru conexiuni stabile și laboratoare rapide:

```bash
nmap -T4 10.10.10.10
```

* **Mod verbos în timp real ( -v ):** Afișează porturile deschise direct în terminal pe măsură ce sunt descoperite, fără a aștepta finalul scanării:

```bash
nmap -v 10.10.10.10
```

* **Rularea unui script specific NSE ( --script ):** Execută un script anume de enumerare (ex. testarea accesului anonim pe FTP):

```bash
nmap -p 21 --script=ftp-anon 10.10.10.10
```

* **Salvarea rezultatelor în toate formatele ( -oA ):** Generează automat rapoarte în formatele standard, XML și grepable:

```bash
nmap -oA nmap_scan_results 10.10.10.10
```


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
