## Depanare Cron Jobs și Lock Files

### 1. Verificare Cron
* `sudo systemctl status cron` — Verifică dacă serviciul cron este activ.
* `sudo crontab -l` — Listează sarcinile programate pentru utilizatorul curent.
* `sudo crontab -e` — Editează sarcinile programate.

### 2. Conceptul de Stale Lock File
* Multe scripturi de fundal creează un fișier `.lock` la pornire pentru a preveni rulări paralele.
* Dacă scriptul crapă violent, fișierul `.lock` rămâne pe disc și blochează viitoarele executări.
* **Soluție:** Se identifică și se șterge fișierul `.lock` rămas (`rm /cale/catre/fișier.lock`).
