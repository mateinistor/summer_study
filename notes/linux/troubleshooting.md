#Linux Troubleshooting

## Depanare Cron Jobs și Lock Files

### 1. Verificare Cron
* `sudo systemctl status cron` — Verifică dacă serviciul cron este activ.
* `sudo crontab -l` — Listează sarcinile programate pentru utilizatorul curent.
* `sudo crontab -e` — Editează sarcinile programate.

### 2. Conceptul de Stale Lock File
* Multe scripturi de fundal creează un fișier `.lock` la pornire pentru a preveni rulări paralele.
* Dacă scriptul crapă violent, fișierul `.lock` rămâne pe disc și blochează viitoarele executări.
* **Soluție:** Se identifică și se șterge fișierul `.lock` rămas (`rm /cale/catre/fișier.lock`).


---

## Strace

Utilitar de diagnosticare și depanare la nivel de nucleu (kernel) care interceptează și înregistrează apelurile de sistem (*syscalls*) efectuate de un proces, precum și semnalele recepționate de acesta.

---

### Utilizare de Bază

* **Rularea unui binar sub monitorizare:**
  ```bash
  strace ./executabil
```

### Scenarii Practice de Depanare (Troubleshooting)

* **Depistarea fișierelor de configurare sau a dependențelor lipsă:**

Identifică fișierele pe care programul încearcă să le deschidă, dar eșuează cu eroarea ENOENT:

   ```bash
   strace ./executabil 2>&1 | grep -iE "open|access|no such file"
   ```

* **Depistarea problemelor de permisiuni ascunse:**

Identifică apelurile refuzate cu EACCES (Permission denied):

   ```bash
   strace ./executabil 2>&1 | grep -i "EACCES"
   ```

* **Investigarea unui proces blocat (Hang):**

Atașează strace direct pe procesul care consumă resurse sau nu răspunde pentru a vedea la ce apel de sistem s-a oprit (de ex. un read pe un socket sau un futex de sincronizare):

   ```bash
   strace -p <PID>
   ```











