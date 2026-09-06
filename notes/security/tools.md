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

# Scanare fără Ping (presupune că mașina este activă; util când firewall-ul blochează ICMP)
nmap -Pn <IP>

# Detectarea versiunilor exacte de servicii (ex. versiune Apache, FileZilla, OpenSSH)
nmap -sV <IP>

# Rularea scripturilor implicite de enumerare și vulnerabilități comune (NSE)
nmap -sC <IP>

# Scanare pe porturi specifice
nmap -p 21,80,443,3389 <IP>

# Scanarea tuturor celor 65.535 de porturi (full scan)
nmap -p- <IP>

# Scanare SYN Stealth (rapidă, discretă, nu finalizează 3-way handshake; cere drepturi de root)
sudo nmap -sS <IP>

# Scanare TCP Connect completă (utilizabilă fără drepturi de administrator)
nmap -sT <IP>

# Scanare pentru porturi UDP
sudo nmap -sU <IP>

# Ajustarea vitezei de scanare (0-5, unde T4 este optim pentru conexiuni stabile de laborator)
nmap -T4 <IP>

# Mod verbos (afișează porturile deschise în timp real, pe măsură ce le găsește)
nmap -v <IP>

# Rularea unui script specific NSE (ex. testare login anonim pe serverul FTP)
nmap -p 21 --script=ftp-anon <IP>

# Salvarea rezultatelor în toate cele 3 formate de bază (normal, XML, grepable)
nmap -oA scan_results <IP>

# Comandă combinată standard pentru enumerarea completă a unei ținte
sudo nmap -Pn -sS -sV -sC -p- -T4 -v <IP>
