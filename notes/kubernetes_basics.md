# Kubernetes (K8s) Basics - Arhitectură și Comenzi

## 1. Ce este Kubernetes?
Dacă Docker se ocupă cu crearea și izolarea containerelor, Kubernetes este „dirijorul” (orchestratorul) care decide unde rulează aceste containere, cum comunică între ele și cum se scalează automat atunci când aplicația primește trafic intens.

## 2. Arhitectura Fizică și Logică

* **Cluster:** Întregul sistem Kubernetes, format dintr-un nod „Master” (creierul) și mai multe noduri „Worker” (brațele de muncă).
* **Nod (Node):** Un server fizic sau o mașină virtuală care face parte din cluster. Aici rulează efectiv aplicațiile.
* **Pod:** Cea mai mică unitate din K8s. Este „ambalajul” în care rulează containerele. 
  * *Regula de aur:* Un Pod = Un Container (de obicei). 
  * Pod-urile sunt temporare (efemere); dacă un Pod moare din cauza unei erori, K8s creează automat altul nou.

## 3. Resurse Cheie în K8s

* **Deployment:** Este o rețetă logică prin care îi spui lui Kubernetes câte clone (replici) ale unui Pod vrei să ruleze constant. Dacă un Nod fizic pică și pierzi 2 Pod-uri, Deployment-ul le recreează instant pe un alt Nod sănătos.
* **Service (Serviciu):** Deoarece Pod-urile se nasc și mor având IP-uri diferite, un *Service* oferă un IP fix și stabil. Astfel, utilizatorii (sau alte aplicații) pot accesa Pod-urile fără întrerupere. Funcționează și ca Load Balancer (împarte traficul în mod egal între mai multe Pod-uri).

## 4. Comenzi Fundamentale (`kubectl`)
`kubectl` este unealta principală prin care interacționezi cu clusterul K8s.

### Investigare și Navigare
* `kubectl get pods` — Listează toate Pod-urile din proiectul curent și starea lor (ex: Running, Pending).
* `kubectl get nodes` — Listează serverele din cluster.
* `kubectl describe pod <nume-pod>` — Afișează detalii complete, limitări de memorie și erori (Events) despre un Pod specific. Este **comanda supremă pentru depanare**.
* `kubectl logs <nume-pod>` — Arată log-urile generate de containerul din interiorul Pod-ului.

### Modificare și Ștergere
* `kubectl apply -f manifest.yml` — Aplică configurațiile dintr-un fișier YAML (creează resurse noi sau le actualizează pe cele existente).
* `kubectl delete pod <nume-pod>` — Șterge manual un Pod. (Dacă face parte dintr-un Deployment, K8s va genera imediat altul nou în loc).
* `kubectl delete -f manifest.yml` — Șterge curat tot ce a fost creat prin acel fișier.

## 5. Exemplu de Manifest (`deployment.yml`)
În Kubernetes, rar dăm comenzi manuale pentru a crea lucruri. În schimb, declarăm starea dorită în fișiere YAML, iar K8s se ocupă să o transforme în realitate.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: site-ul-meu-nginx
spec:
  replicas: 3 # Vrem 3 Pod-uri identice care să ruleze simultan
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: server-nginx
        image: nginx:latest # Imaginea Docker pe care o va descărca
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi" # Cere ca Nodul să aibă minimum 128MB RAM liberi
