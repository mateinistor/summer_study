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
