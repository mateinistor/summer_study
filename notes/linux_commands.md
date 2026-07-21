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
	'grep "<text>" <fisier>' - Cauta un sir de caractere specific intr-un fisier
	'wc <fisier>' - Numara liniile, cuvintele si octetii dintr-un fisier
		'-l' - numara doar liniile

## 4. Reanalizare si Scurtaturi in Terminal

	'which <comanda>' - Arata calea absoluta catre executabilul unei comenzi
	'nano <fisier>' - Deschide editorul de text simplu in terminal
		'Ctrl + O' + 'Enter' - Salveaza fisierul
		'Ctrl + X' - Iesi din nano
	'clear' sau 'Ctrl + L' - Curata ecranul terminalului

## 5. Legarea Comenzilor (Pipes si Redirectionari)
	'comanda1 | comanda2' - Trimite iesirea primei comenzi ca intrare pentru a doua
		*Exemplu*: 'cat /etc/passwd | wc -l' - contorizeaza numarul de utilizatori/randuri
	'comanda > fisier' - Salveaza iesirea intr-un fisier (suprascrie continutul existent)
	'comanda >> fisier' - Salveaza iesirea la finalul unui fisier (adauga fara sa stearga)

