## Folderele principale ale sistemului 
/etc - Toate fisierele de configurare ale sistemului

/var - Fisierele ale caror dimensiuni si continut se schimba constant in timp ce sistemul ruleaza
	/var/log - Unde sistemul scrie toate jurnalele(Primul loc in care sa te uiti cand investighezi un eveniment
	/var/www - Locul standard unde stau fisierele unui site web
	/var/cache - Date temporare salvate de aplicatii pentru a se incarca mai repede data viitoare
	/var/spool - Cozi de asteptare(e-mailuri, sarcini trimise catre imprimata etc)

/bin - Fisierele binare(executabile)


## Fisierul de Configurare '.bashrc'

	Locatie: '~/.bashrc' (fisier ascuns in 'home')
	Rol: Se executa automat la deschiderea fiecarui terminal nou
	Ce contine:
		Alisi: 'alias ll='ls -lah''
		Variabile de mediu: 'export PATH="$HOME/bin:$PATH"'
```bash
	Aplicare modificari pe loc:
	source ~/.bashrc
```
	
