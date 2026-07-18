1. Navigare și Fișiere (The Basics)
    pwd (Print Working Directory) — Îți arată calea completă a folderului în care te afli acum.
    ls -la — Listează toate fișierele și folderele din directorul curent, inclusiv cele ascunse
    cd [cale] (Change Directory) — Te mută în folderul specificat. (cd .. te duce înapoi cu un nivel).
    cat [fișier] — Afișează tot conținutul unui fișier direct în terminal.

2. Filtrare și Manipulare Text
    grep "[text]" [fișier] — Caută și afișează doar liniile care conțin textul respectiv.

    grep -B 1 "[text]" [fișier] — Afișează linia găsită + 1 linie de dinainte (Before) pentru context.

    grep -A 1 "[text]" [fișier] — Afișează linia găsită + 1 linie de după (After).

    grep -c "[text]" [fișier] — Contorizează și afișează numărul de linii în care apare textul.

    sort -u [fișier] — Sortează liniile alfabetic și elimină toate duplicatele (Unique).

    comm -12 [fișier1.sorted] [fișier2.sorted] — Compară două fișiere deja sortate și afișează doar liniile comune lor.

3. Monitorizare Resurse și Procese (The Admin Tools)
    ps aux — Face o „fotografie” statică tuturor proceselor care rulează în sistem în acea milisecundă.
    
    htop — Deschide un manager de sarcini interactiv și live (video). Arată consumul de CPU/RAM și permite oprirea proceselor (cu F9).

4. Legături între comenzi (Pipes)
    | (Pipe) — Ia rezultatul (output-ul) comenzii din stânga și îl trimite ca intrare (input) către comanda din dreapta.
