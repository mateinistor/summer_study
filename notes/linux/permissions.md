# Linux Permissions: Standard & SUID

## Reprezentare & Sistem Octal
Permisiunile standard se împart în 3 categorii: **User (proprietar)**, **Group (grup)**, **Others (toți ceilalți)**.

* `4` = **Read (`r`)**
* `2` = **Write (`w`)**
* `1` = **Execute (`x`)**

Valoarea maximă a unui triplet este `7` (`rwx`).

---

## Bitul Special: SUID (Set owner User ID)
În mod normal, un proces rulează cu permisiunile utilizatorului apelant. Când bitul **SUID** este activat, binarul va rula temporar cu permisiunile **proprietarului fișierului** (owner).

* **Caz clasic:** `/usr/bin/passwd` este deținut de `root` și are SUID setat (`-rwsr-xr-x`). Permite unui utilizator standard să scrie parola nouă în fișierul protejat `/etc/shadow`.

### Reprezentare vizuală
Litera `s` înlocuiește bitul `x` la nivelul proprietarului:
* `rwx r-x r-x` $\rightarrow$ Normal (755)
* `rws r-x r-x` $\rightarrow$ SUID activat cu drept de execuție (4755)
* `rwS r-x r-x` $\rightarrow$ SUID activat, dar **fără** permisiune de execuție (`x`)

Exemple:
* `4755` = SUID + `rwxr-xr-x`
* `4700` = SUID + `rwx------`

---

## Comenzi esențiale

### Setare și eliminare SUID
```bash
# Simbolic
chmod u+s <fisier>     # Activare
chmod u-s <fisier>     # Dezactivare

# Octal
chmod 4755 <fisier>

```
### Tabel Biți Speciali (Prima Cifră Octală: 0 - 7)

Cifra din fața celor trei triade standard (`chmod [0-7]rwxrwxrwx`) controlează **SUID**, **SGID** și **Sticky Bit**:

* `4` = **SUID** (*Set owner User ID*)
* `2` = **SGID** (*Set Group ID*)
* `1` = **Sticky Bit**

| Valoare | Biți activi | Descriere & Rol practic |
| :---: | :--- | :--- |
| **`0`** | Niciunul | Comportament standard (implicit când folosești 3 cifre, ex. `755` = `0755`). |
| **`1`** | **Sticky Bit** | Doar proprietarul poate șterge/redenumi fișierele din acel director comun (ex. `/tmp` are `1777` $\rightarrow$ `drwxrwxrwt`). |
| **`2`** | **SGID** | Pe fișiere: rulează cu drepturile grupului. Pe directoare: fișierele nou create moștenesc automat grupul folderului părinte. |
| **`3`** | **SGID + Sticky** | Combinație `2 + 1`. |
| **`4`** | **SUID** | Binarul rulează temporar cu privilegiile proprietarului (*owner*). |
| **`5`** | **SUID + Sticky** | Combinație `4 + 1`. |
| **`6`** | **SUID + SGID** | Combinație `4 + 2` (capătă identitatea de execuție atât a user-ului, cât și a grupului). |
| **`7`** | **Toate 3** | SUID + SGID + Sticky Bit (`4 + 2 + 1`). |



