## Folderele principale ale sistemului 

* `/etc` — Toate fișierele de configurare ale sistemului.
* `/var` — Fișiere ale căror dimensiuni și conținut se schimbă constant (jurnale, cache).
  * `/var/log` — Unde sistemul scrie toate jurnalele (primul loc în care te uiți când investighezi un eveniment).
  * `/var/www` — Locul standard unde stau fișierele unui site web.
  * `/var/cache` — Date temporare salvate de aplicații pentru încărcare rapidă.
  * `/var/spool` — Cozi de așteptare (e-mailuri, sarcini de imprimantă etc.).
* `/bin` — Fișierele binare (executabile).


## Fișierul de Configurare '.bashrc'

* **Locație:** `~/.bashrc` (fișier ascuns în `home`).
* **Rol:** Se execută automat la deschiderea fiecărui terminal nou.
* **Ce conține:**
  * **Aliași:** `alias ll='ls -lah'`
  * **Variabile de mediu:** `export PATH="$HOME/bin:$PATH"`

**Aplicare modificări pe loc:**
```bash
source ~/.bashrc
