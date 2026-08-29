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

### Interpretarea Răspunsurilor HTTP

| Cod Status | Semnificație | Relevanță în Securitate |
| :--- | :--- | :--- |
| **200 OK** | Succes | Resursa există și este accesibilă direct. |
| **301 / 302** | Redirecționare | Directorul există și redirecționează către un alt traseu. |
| **401 Unauthorized** | Autentificare cerută | Pagină sau panou protejat prin parolă (țintă de interes). |
| **403 Forbidden** | Acces restricționat | Resursa există fizic pe server, dar accesul direct este blocat. |
| **404 Not Found** | Inexistent | Traseul nu există (ignorat automat de utilitar). |
