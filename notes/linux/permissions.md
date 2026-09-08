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

Valoarea octală folosește 4 cifre:
* **`4` = SUID**
* **`2` = SGID**
* **`1` = Sticky Bit**

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
