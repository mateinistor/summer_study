# HTTP Basics

## 1. Ce este HTTP?
**HTTP** (*HyperText Transfer Protocol*) este protocolul fundamental de comunicare din World Wide Web, funcționând la **Nivelul Aplicație** (Layer 7 în modelul OSI) peste o conexiune de transport de tip **TCP**.

* **Model Client-Server:** Clientul (browser, `curl`) trimite o cerere (**Request**), iar serverul returnează un răspuns (**Response**).
* **Stateless:** Fiecare cerere este tratată independent. Serverul nu reține starea anterioară a conexiunilor; identificarea utilizatorilor se realizează prin mecanisme suplimentare (**Cookies**, **Sesiuni**, **Token-uri JWT**).
* **Text clar (Plain Text):** În versiunile HTTP/1.0 și HTTP/1.1, mesajele sunt transmise în format text lizibil direct.
* **Port standard:** Rulează implicit pe portul **`80 TCP`**.

---

## 2. HTTP vs. HTTPS
* **HTTP:** Transmisie necriptată. Oricine interceptează traficul în rețea (ex. pe Wi-Fi public prin ARP Spoofing / Wireshark) poate citi datele în clar, inclusiv parole sau cookie-uri de sesiune.
* **HTTPS (*HTTP Secure*):** Același protocol HTTP, însă încapsulat într-un tunel criptat prin **TLS/SSL**.
  * Rulează pe portul standard **`443 TCP`**.
  * Asigură **Confidențialitate** (date indescifrabile în tranzit), **Integritate** (datele nu pot fi modificate pe drum) și **Autentificare** (dovedește identitatea serverului prin certificate digitale).

---

## 3. Structura unui URL
Un URL (*Uniform Resource Locator*) conține componentele necesare localizării și interogării unei resurse:

```text
[https://user:password@exemplu.ro:443/cale/pagina?categorie=it&pret=100#detalii]
└─┬─┘   └─────┬─────┘ └────┬────┘ └─┬─┘ └─────┬────┘ └──────────┬───────┘ └───┬───┘
Scheme    Userinfo       Host     Port      Path           Query         Fragment

```
## 4. Protocoale comune la Nivel Aplicație (Layer 7)

| Protocol | Port standard | Destinație |
| :--- | :--- | :--- |
| **HTTP** | `80` (TCP) | Navigare web necriptată |
| **HTTPS** | `443` (TCP) | Navigare web securizată prin TLS/SSL |
| **SSH** | `22` (TCP) | Administrare de la distanță prin consolă securizată |
| **FTP / SFTP** | `21` / `22` (TCP) | Transfer de fișiere |
| **DNS** | `53` (UDP/TCP) | Translatarea numelor de domenii în adrese IP |
| **DHCP** | `67`, `68` (UDP) | Alocarea dinamică a adreselor IP în rețeaua locală |
| **SMTP** | `25`, `587` (TCP) | Trimiterea de mesaje e-mail între servere |
| **IMAP / POP3** | `993` / `995` (TCP) | Preluarea și citirea mesajelor e-mail |

---

## 5. Structura Mesajelor HTTP

### Formatul unei Cereri (Request)

```http
GET / HTTP/1.1

Host: tryhackme.com
User-Agent: Mozilla/5.0 Firefox/87.0
Referer: https://tryhackme.com/

```
### Formatul unui Raspuns (Response)
```http
HTTP/1.1 200 OK

Server: nginx/1.15.8
Date: Fri, 09 Apr 2021 13:34:03 GMT
Content-Type: text/html
Content-Length: 98


<html>
<head>
    <title>TryHackMe</title>
</head>
<body>
    Welcome To TryHackMe.com
</body>
</html>


```

## 6. Metode HTTP Comune (TryHackMe Summary)

Metodele HTTP sunt modul prin care clientul indică acțiunea pe care intenționează să o execute atunci când trimite o cerere către server. Deși există mai multe metode, cel mai des utilizate în practică sunt `GET` și `POST`.

---

### Cele mai folosite metode

| Metodă | Descriere | Cazuri de utilizare |
| :--- | :--- | :--- |
| **`GET`** | Preluarea și citirea de informații de pe un server web. | Încărcarea unei pagini, descărcarea unei imagini, căutări simple. |
| **`POST`** | Trimiterea de date către server și, potențial, crearea de înregistrări noi. | Înregistrare/autentificare (login), adăugarea unui comentariu, formulare web. |
| **`PUT`** | Trimiterea de date către un server web pentru a actualiza informații existente. | Modificarea setărilor de profil, actualizarea stocului unui produs. |
| **`DELETE`** | Ștergerea de informații sau înregistrări de pe un server web. | Ștergerea unui cont, eliminarea unui articol sau comentariu. |
