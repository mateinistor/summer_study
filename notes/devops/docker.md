# Docker Basics - Concepte și Comenzi Fundamentale

## 1. Ce este Docker?
Docker este o platformă care rezolvă problema clasică: *"La mine pe calculator codul mergea"*. Funcționează prin împachetarea aplicației împreună cu **strictul necesar** (librării, un mini-sistem de operare) pentru a rula identic pe absolut orice server.

## 2. Imagine vs. Container
Acestea sunt cele mai importante două concepte de înțeles:

* **Imaginea (Rețeta):** 
  * Este pachetul de bază, minimul necesar pentru ca programul să ruleze (ex: un mini-Linux gol + Python + codul tău).
  * Este **Read-Only** (nu poate fi modificată deloc după ce este creată).
* **Containerul (Aplicația activă):** 
  * Este imaginea „adusă la viață” și pusă în execuție. 
  * Adaugă un strat temporar de scriere (unde aplicația poate salva fișiere temporare sau log-uri).
  * Este 100% izolat de restul calculatorului (are propria rețea, memorie fixă alocată și arbore de procese ascuns de sistemul principal).

## 3. Comenzi pentru Imagini
* `docker pull <nume_imagine>` — Descarcă o imagine de pe Docker Hub (magazinul oficial).
* `docker images` — Listează toate imaginile descărcate local.
* `docker rmi <id_imagine>` — Șterge o imagine de pe disc.

## 4. Comenzi pentru Containere
* `docker run <nume_imagine>` — Creează și pornește un container dintr-o imagine.
  * `-d` — (Detached) Rulează containerul în fundal, eliberând terminalul.
  * `-p 8080:80` — (Port mapping) Leagă portul 8080 al sistemului tău de portul 80 al containerului.
  * `--name <nume>` — Dă un nume ușor de reținut containerului.
* `docker ps` — Arată containerele care rulează activ în acest moment.
* `docker ps -a` — Arată TOATE containerele din sistem (inclusiv cele oprite).
* `docker stop <id_container>` — Oprește (pune pe pauză) un container activ.
* `docker start <id_container>` — Repornește un container oprit.
* `docker rm <id_container>` — Șterge definitiv un container (trebuie să fie oprit înainte). Pentru a-l șterge forțat se folosește flag-ul `-f`.

## 5. Exemplu Practic: Dockerfile
Acesta este fișierul de configurare folosit pentru a-ți construi propria Imagine de la zero. Exemplu pentru un script Python (`app.py`):

```dockerfile
# 1. Baza: Descărcăm un mini-sistem Linux care are doar Python instalat
FROM python:3.9-slim

# 2. Setăm folderul de lucru în interiorul containerului
WORKDIR /app

# 3. Copiem fișierele noastre în container
COPY . .

# 4. Instalăm librăriile externe necesare codului
RUN pip install -r requirements.txt

# 5. Comanda executată automat în momentul în care pornește containerul
CMD ["python", "app.py"]
