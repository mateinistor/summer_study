# Linux Networking - Concepte de Rețea, Porturi și Firewall

## 1. Concepte Fundamentale de Rețea
* **Adresa IP:** Coordonata unică a unui server în rețea. 
  * `127.0.0.1` (sau `localhost`): Este adresa internă de „loopback”. Orice cerere trimisă aici nu iese din server, ci se întoarce direct la el.
* **Porturile:** Dacă IP-ul este adresa clădirii, porturile sunt „ușile” specifice de acces (ex: 80 pentru trafic web HTTP, 443 pentru HTTPS, 22 pentru conexiuni securizate SSH).
* **Protocoale (TCP vs. UDP):**
  * **TCP:** Se asigură că pachetele de date ajung întregi și în ordinea corectă (folosit pentru site-uri web, transfer de fișiere).
  * **UDP:** Trimite datele rapid, fără să verifice dacă au ajuns (folosit pentru streaming video sau jocuri).

## 2. Investigarea Rețelei (Diagnosticare)
* `curl <adresa_ip>:<port>` — Face o cerere web direct de pe linia de comandă. (Ex: `curl localhost:80` testează dacă un server intern răspunde la portul 80).
* `ss -tuln` — Listează toate porturile deschise pe server și arată ce aplicații ascultă pe ele. Este esențial pentru a verifica unde rulează serviciile tale.
* `ping <adresa_ip>` — Verifică dacă un alt server este online și măsoară timpul de răspuns.

## 3. Redirecționarea Porturilor (Port Forwarding) cu iptables
`iptables` este firewall-ul nativ din Linux. Pe lângă blocarea sau permiterea traficului, are un tabel special (NAT) care poate intercepta și redirecționa pachetele de date din zbor.

### Scenariul A: Redirecționarea traficului LOCAL (pentru teste interne)
Acest scenariu se aplică atunci când o aplicație sau un test accesează un port local (ex: 80), dar serviciul real este ascuns pe alt port (ex: 20280). Pentru că traficul este generat din interiorul serverului, interceptăm pachetele folosind lanțul `OUTPUT` pe interfața de loopback.

```bash
sudo iptables -t nat -A OUTPUT -p tcp -o lo --dport 80 -j REDIRECT --to-port 20280
```

### Scenariul B: Redirecționarea traficului EXTERN (din internet)
Acest scenariu se aplică atunci când vizitatorii din afara serverului accesează un port public, iar tu vrei să trimiți acel trafic către o aplicație internă. Aici interceptăm pachetele direct la intrarea în sistem, înainte de a fi procesate, folosind lanțul PREROUTING.

```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 5000
```



## Depanare SSH și Inspectare Log-uri

### Comenzi Rapide de Diagnosticare SSH
* `ssh -v user@ip` — Modul verbose: arată pas cu pas unde se blochează conexiunea.
* `sudo service ssh status` (sau `systemctl status ssh`) — Verifică dacă serverul SSH rulează.
* `ss -tuln | grep 22` — Verifică dacă portul 22 este deschis și ascultă conexiuni.

### Monitorizarea Log-urilor de Autentificare
* `/var/log/auth.log` — Fișierul principal unde Linux înregistrează toate încercările de login.
* `sudo tail -f /var/log/auth.log` — Monitorizează în timp real încercările de conectare SSH.
* `sudo grep "Failed password" /var/log/auth.log` — Listează parolele greșite / atacurile.

---

### Erori Frecvente în Log-uri

#### 1. Permisiuni greșite pe fișierele SSH (Permission denied)
SSH refuză conexiunea dacă folderul sau cheile au permisiuni prea permisive.
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/authorized_keys
```


#### 2. Eroare la crearea sesiunii
	Eroare: "Varlink call io.systemd.Login.CreateSession failed / Transport endpoint is not connected"

Solutie rapida:
```bash
sudo systemctl restart systemd-logind
# Dacă ești în WSL (Windows Subsystem for Linux), rulează în PowerShell:
wsl --shutdown
```



