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


