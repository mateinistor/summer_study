#Linux Troubleshooting

## Depanare Cron Jobs și Lock Files
# 🛠️ Ghid de Diagnosticare și Depanare sub Linux

Acest ghid conține scenarii practice, comenzi utile și metodologii pentru izolarea și rezolvarea problemelor de sistem, procese și automatizări.

---

## 🔍 1. Monitorizarea Apelurilor de Sistem cu `strace`

`strace` este un instrument de depanare care interceptează și înregistrează apelurile de sistem (system calls) efectuate de un proces. Este ideal pentru situațiile în care un executabil se comportă anormal și nu ai acces la codul sursă.

### ⚙️ Argumente frecvent utilizate

| Opțiune | Descriere |
| :--- | :--- |
| `-p <PID>` | Atașează `strace` la un proces activ, aflat deja în execuție. |
| `-e <apel>` | Filtrează doar anumite apeluri de sistem (ex: `-e trace=open,connect`). |
| `-c` | Contorizează timpul, apelurile și erorile și afișează un rezumat la final. |
| `-f` | Urmărește și procesele copil (child processes) create de procesul principal. |

### 🛠️ Scenarii practice cu `strace`

* **Depistarea fișierelor de configurare sau a dependențelor lipsă (`ENOENT`):**
  ```bash
  strace ./executabil 2>&1 | grep -iE "open|access|no such file"
  ```

* **Depistarea problemelor de permisiuni ascunse (`EACCES`):**
  ```bash
  strace ./executabil 2>&1 | grep -i "EACCES"
  ```

* **Investigarea unui proces blocat (Hang):**
  Atașează instrumentul pentru a vedea la ce apel de sistem s-a oprit (ex: un `read` pe un socket sau un `futex` de sincronizare):
  ```bash
  strace -p <PID>
  ```

---

## ⏰ 2. Depanare Task-uri Cron (Cron Job Troubleshooting)

Problemele cu `cron` apar de obicei deoarece mediul de execuție din cron este extrem de limitat în comparație cu cel al unui terminal interactiv (nu are aceleași variabile de mediu sau alias-uri).

### ⚙️ Comenzi rapide de verificare

* **Verificarea statusului serviciului:**
  ```bash
  systemctl status cron   # Pe sisteme bazate auf Debian/Ubuntu
  systemctl status crond  # Pe sisteme bazate pe RHEL/CentOS/Fedora
  ```

* **Vizualizarea listei de cron job-uri pentru utilizatorul curent:**
  ```bash
  crontab -l
  ```

### 🛠️ Scenarii practice de depanare Cron

* **Vizualizarea logurilor în timp real (Urmărirea execuției):**
  Verifică dacă daemonul Cron încearcă măcar să ruleze scriptul tău la ora programată:
  ```bash
  tail -f /var/log/syslog | grep -i cron
  # SAU pe sisteme cu journald:
  journalctl -u cron -n 50 -f
  ```

* **Problema căilor relative (Cea mai frecventă eroare):**
  Cron nu știe unde se află executabilele tale dacă nu folosești căi absolute. 
  * *Greșit:* `30 2 * * * python3 script.py`
  * *Corect:* `30 2 * * * /usr/bin/python3 /home/user/scripts/script.py`

* **Redirecționarea output-ului pentru analiză (Debug):**
  Deoarece cron nu afișează erorile pe ecran, redirecționează atât output-ul normal (`stdout`), cât și erorile (`stderr`) într-un fișier de log dedicat:
  ```bash
  * * * * * /cale/catre/script.sh >> /var/log/my_cron_debug.log 2>&1
  ```

