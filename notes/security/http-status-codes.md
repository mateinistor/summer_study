# Cheat Sheet: Coduri de Stare HTTP

## Categorii de Coduri de Stare HTTP

| Interval | Categorie | Descriere |
| :--- | :--- | :--- |
| **100-199** | **Răspuns Informațional** (*Information Response*) | Trimise pentru a anunța clientul că prima parte a cererii a fost acceptată și că ar trebui să continue trimiterea restului cererii. Aceste coduri nu mai sunt foarte des întâlnite. |
| **200-299** | **Succes** (*Success*) | Acest interval indică faptul că cererea clientului a fost procesată cu succes. |
| **300-399** | **Redirecționare** (*Redirection*) | Folosite pentru a redirecționa cererea clientului către o altă resursă (o pagină web diferită sau un site complet separat). |
| **400-499** | **Erori Client** (*Client Errors*) | Utilizate pentru a notifica clientul că a existat o problemă/eroare la nivelul cererii trimise de el. |
| **500-599** | **Erori Server** (*Server Errors*) | Rezervate pentru erorile apărute pe partea de server; indică, de obicei, o problemă majoră a serverului în procesarea cererii. |

---

## Coduri de Stare HTTP Comune

| Cod de Stare | Nume | Descriere |
| :--- | :--- | :--- |
| `200` | **OK** | Cererea a fost finalizată cu succes. |
| `201` | **Created** (Creat) | O resursă a fost creată cu succes (de exemplu, un utilizator nou sau un articol de blog). |
| `301` | **Moved Permanently** (Mutat Permanent) | Redirecționează browserul către o pagină nouă sau anunță motoarele de căutare că resursa a fost mutată definitiv la altă adresă. |
| `302` | **Found** (Găsit / Redirecționare Temporară) | Similar cu redirectul 301, dar reprezintă o schimbare temporară ce poate reveni sau se poate modifica în viitorul apropiat. |
| `400` | **Bad Request** (Cerere Incorectă) | Notifică browserul că cererea a fost greșită sau incompletă (de exemplu, lipsește un parametru obligatoriu așteptat de server). |
| `401` | **Not Authorised** (Neautorizat) | Accesul este restricționat până când clientul se autentifică în aplicația web (cel mai des prin utilizator și parolă). |
| `403` | **Forbidden** (Interzis) | Nu ai permisiunea de a accesa această resursă, indiferent dacă ești autentificat sau nu. |
| `405` | **Method Not Allowed** (Metodă Nepermisă) | Resursa nu permite metoda HTTP trimisă (de exemplu, trimiți un request `GET` pe `/create-account` când serverul așteaptă o metodă `POST`). |
| `404` | **Page Not Found** (Pagină Negăsită) | Pagina sau resursa solicitată nu există pe server. |
| `500` | **Internal Server Error** (Eroare Internă de Server) | Serverul a întâmpinat o eroare neașteptată pe care nu știe cum să o gestioneze. |
| `503` | **Service Unavailable** (Serviciu Indisponibil) | Serverul nu poate procesa cererea deoarece este supraîncărcat sau se află în mentenanță. |
