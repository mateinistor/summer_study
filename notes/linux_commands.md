# Linux Commands - Quick Reference Sheet


## 1. Navigare

	'pwd' - Afiseaza directoul curent
	'ls -lah' - Listeaza fisierele si directoarele in format detaliat:
		'-l' - format lung(permisiuni, dimensiune, proprietar)
		'-a' - afiseaza si fisierele ascunse
		'-h' - dimensiuni citibile (human-readable)
	'cd <dir>' - Schimba directorul curent
	'cd ~' sau 'cd' - Schimba directorul in 'home'
	'cd ..' - Se intoarce cu un nivel in structura de directoare

## 2. Manipulare fisiere si directoare

	'mkdir -p <path/folder>' - Creeaza un director
		'-p' - pentru  a crea si path-ul, in caz ca nu exista
	'touch <fisier>' - Creeaza un fisier gol
	'cp <sursa> <destinatie>' - Copiaza un fisier
	'cp -r <folder_sursa> <destinatie>' - Copiaza un director recursiv
	'mv <sursa> <destinatie>' - Muta sau redenumeste un fisier/director
	'rm <fisier>' - Sterge un fisier
	'rm -rf <folder' - Sterge un director si tot continutul sau
		'-r' - recursiv
		'-f' - fortat

## 3. Vizualizare si Procesare Text

	'cat <fisier>' - Afiseaza tot continutul fisierului in terminal
	'head -n <X> <fisier>' - Afiseaza primele 'X' linii din fisier
	'tail -n <X> <fisier>' - Afiseaza ultimele 'X' linii din fisier
	'less <fisier>' - Deschide un fisier mare pentru citire pagina cu pagina('q' - iesire, '/' - cautare, 'g'/'G' - inceput/sfarsit
	'grep "<text>" <fisier>' - Cauta un sir de caractere specific intr-un fisier
	'wc <fisier>' - Numara liniile, cuvintele si octetii dintr-un fisier
		'-l' - numara doar liniile
	'sed 's/vechi/nou/g' <fisier>' - Inlocuieste un text cu altul in tot fisierul
		'-i' : Salveaza modificarile direct in fisier
## 4. Reanalizare si Scurtaturi in Terminal

	'nano <fisier>' - Deschide editorul de text simplu in terminal
		'Ctrl + O' + 'Enter' - Salveaza fisierul
		'Ctrl + X' - Iesi din nano
	'clear' sau 'Ctrl + L' - Curata ecranul terminalului

## 5. Legarea Comenzilor (Pipes si Redirectionari)

	'comanda1 | comanda2' - Trimite iesirea primei comenzi ca intrare pentru a doua
		*Exemplu*: 'cat /etc/passwd | wc -l' - contorizeaza numarul de utilizatori/randuri
	'comanda > fisier' - Salveaza iesirea intr-un fisier (suprascrie continutul existent)
	'comanda >> fisier' - Salveaza iesirea la finalul unui fisier (adauga fara sa stearga)

## 6. Cautare Fisiere si Executabile

        'which <comanda>' - Arata calea absoluta catre executabilul unei comenzi
        'whereis <comanda>' - Cauta mai complex: arata calea executabilului, fisierele sursa si paginile de manual
	'locate <nume_fisier>' - Cauta fisiere in tot sistemul folosind o baza de date indexata
	'sudo updatedb' - Actualizeaza manual baza de date pentru 'locate'
	'find <path> -name "<nume>" - Cauta fisiere/directoare in timp real pe disc dupa nume
		'-iname' - Cauta fara sa tina cont de majuscule/minuscule
		'type f' - Cauta doar fisiere

## 7. Permisiuni Fisiere si Directoare

	### Vizualizare si Intelegere Permisiuni

		'ls -l' - Afiseaza permisiunile fisierelor
		Structura: '[tip][proprietar][[grup][altii]'
		'r' = 4 | 'w' = 2 | 'x' = 1
	
	### Schimbare Permisiuni

		'chmod +x <fisier>' - Face fisierul executabil pentru toata lumea
		'chmod -x <fisier>' - Scoate dreptul de executare
		'chmod 755 <fisier>' - Permisiuni standard
		'chmod 644 <fisier>' - Permisiuni standard pentru fisiere text
		'chmod -R 755 <folder>' - Aplica permisiunile recursiv pentru tot folderul si continutul sau

	### Schimbare Proprietar / Grup

		'chown <user> <fisier> - Schimba proprietarul fisierului
		'chown <user>:<grup> <fisier>' - Schimba si proprietarul si grupul
		'sudo chown -R <user>:<grup> <folder> - Schimba proprietarul recursiv pe tot folderul

## 8. Managment Procese si Sistem

	### Identificare Procese

		'ps aux' - Listeaza toate procesele active din sistem si ID-ul lor
		'top' / 'htop' - Afiseaza toate procesele in tipm real
		'pgrep <nume_proces>' - Cauta si afiseaza direct ID-ul unui proces dupa nume
	
	### Oprire Procese

		'kill <PID>' - Trimite semnalul standard de oprire
		'kill -9 <PID>' - Inchidere fortata. Se foloseste cand procesul s-a blocat si nu raspunde
		'killall <nume_proces>' - Opreste toate procesele care au un anumit nume


## Utilitarul `awk` - Procesare și Filtrare de Text

`awk` este folosit pentru a analiza, filtra și manipula fișiere structurate pe linii și coloane (ex: log-uri, fișiere CSV, output-uri de la alte comenzi).

### 1. Structura de Bază
Comanda funcționează pe principiul: **`awk 'tipar { acțiune }' fișier`**
* **Tipar (Pattern):** O condiție sau filtru (ex: `NR==1`, `NF > 3`, `/eroare/`). Dacă lipsește, acțiunea se aplică pe *toate* liniile.
* **Acțiune:** Ce să facă cu linia care se potrivește (ex: `{print $1}`). Dacă lipsește, acțiunea implicită este printarea liniei.

---

### 2. Variabile Interne Esențiale
`awk` creează automat câteva variabile utile la fiecare linie citită:

* **`$0`** : Întreaga linie curentă.
* **`$1`, `$2`, `$3`...** : Coloana 1, coloana 2, coloana 3 etc.
* **`NF` (Number of Fields)** : Numărul total de coloane din linia curentă.
  * `$NF` înseamnă **ultima coloană** din linie.
* **`NR` (Number of Records)** : Numărul total al liniei curente (însumat din toate fișierele).
* **`FNR` (File Number of Records)** : Numărul liniei curente raportat doar la fișierul activ.

---

### 3. Exemple Practice Frecvente

#### A. Extragerea anumitor coloane
Afișează doar prima și a treia coloană dintr-un output (implicit, separatorul de coloane este spațiul):
```bash
ps aux | awk '{print $1, $3}'


