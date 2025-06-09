# Končna Dokumentacija Projekta: TriWave

### Ime projekta
TriWave

### Člani skupine
- Anđela Radaković
- David Novak
- Aleksandra Mickoska

### Povezave do Github repozitorije
[TriWave-SpletnoProgramiranje](https://github.com/aleksandra0611/TriWave-SpletnoProgramiranje)
[TriWave-SistemskaAdministracija](https://github.com/aleksandra0611/TriWave-SistemskaAdministracija)
[TriWave-Dokumentacija](https://github.com/aleksandra0611/TriWave-Dokumentacija)



### Ganttov diagram
Ganttov diagram prikazuje časovni potek projekta od vzpostavitve projekta do konca projekta (do tretjega letnika).

```mermaid
gantt
    title Ganttov diagram projekta TriWave
    dateFormat  YYYY-MM-DD

    section Načrtovanje
    Določitev zahtev            :done, 2025-04-21, 2025-04-25
    Načrtovanje arhitekture     :done, 2025-04-25, 2025-05-02
    
    section Razvoj
    Razvoj backend aplikacije   :active, 2025-05-11, 2025-06-09
    Integracija z MongoDB Atlas :done, 2025-05-12, 2025-05-13
    Razvoj frontend aplikacije  :ative, 2025-05-17, 2025-06-09

    section Azure VM
    Raziskava Azure portala     :done, 2025-05-20, 2025-05-23
    Ustvarjanje Azure računa    :done, 2025-05-21, 2025-05-22
    Vzpostavitev Linux VM       :done, 2025-05-21, 2025-05-22
    Namestitev na Azure VM      :done, 2025-05-21, 2025-05-22

    section Dockerizacija
    Namestitev v Docker okolju  :done, 2025-05-23, 2025-05-25

    section CI/CD
    Nastavitev Docker Hub       :done, 2025-06-02, 2025-06-04
    GitHub Actions CI/CD        :done, 2025-06-02, 2025-06-06
    Webhook implementacija      :done, 2025-06-02, 2025-06-06

    section Testiranje
    Testiranje aplikacije       :active, 2025-06-06, 2025-06-09

    section Dokumentacija
    Priprava končne dokumentacije :active, 2025-06-06, 2025-06-09
```

---

## Primeri uporabe

### Opis problema
Naša spletna aplikacija TriWave omogoča upravljanje in predvajanje radijskih postaj. Uporabniki se lahko registrirajo, prijavijo, dodajajo radijske postaje, pregledujejo sezname postaj in predvajajo radijske tokove. Problem, ki ga rešuje aplikacija, je pomanjkanje enostavne platforme za upravljanje priljubljenih radijskih postaj z intuitivnim vmesnikom in oblačno podatkovno bazo. Aplikacija ne temelji na matematični enačbi, temveč na funkcionalnih zahtevah za shranjevanje, upravljanje in predvajanje podatkov o radijskih postajah.

### Primer uporabe 1: Registracija uporabnika
Sekvenčni diagram prikazuje postopek registracije novega uporabnika.

```mermaid
sequenceDiagram
    actor Uporabnik
    participant Frontend
    participant Backend
    participant MongoDB

    Uporabnik->>Frontend: Vnos podatkov (email, geslo)
    Frontend->>Backend: POST /api/register
    Backend->>MongoDB: Shrani uporabnika
    MongoDB-->>Backend: Potrditev shranjevanja
    Backend-->>Frontend: Uspešna registracija (200 OK)
    Frontend-->>Uporabnik: Prikaz potrditve
```

### Primer uporabe 2: Dodajanje radijske postaje
Sekvenčni diagram prikazuje dodajanje nove radijske postaje s strani prijavljenega uporabnika.

```mermaid
sequenceDiagram
    actor Uporabnik
    participant Frontend
    participant Backend
    participant MongoDB

    Uporabnik->>Frontend: Vnos podatkov postaje (ime, URL)
    Frontend->>Backend: POST /api/stations/new (JWT token)
    Backend->>MongoDB: Preveri uporabnika in shrani postajo
    MongoDB-->>Backend: Potrditev shranjevanja
    Backend-->>Frontend: Uspešno dodana postaja (200 OK)
    Frontend-->>Uporabnik: Prikaz potrditve
```

### Primer uporabe 3: Predvajanje radijske postaje
Sekvenčni diagram prikazuje predvajanje radijske postaje.

```mermaid
sequenceDiagram
    actor Uporabnik
    participant Frontend
    participant Backend
    participant StreamServer

    Uporabnik->>Frontend: Klik na "Predvajaj" za postajo
    Frontend->>Backend: GET /api/stations/:id
    Backend-->>Frontend: Podatki o postaji (URL toka)
    Frontend->>StreamServer: Zahteva za pretakanje toka
    StreamServer-->>Frontend: Zvočni tok
    Frontend-->>Uporabnik: Predvajanje zvoka
```

---

## Arhitektura programske rešitve

### Diagram arhitekture
Arhitektura aplikacije je sestavljena iz frontend in backend dela, ki tečeta v ločenih Docker vsebnikih, povezanih z MongoDB Atlas podatkovno bazo.

```mermaid
graph TD
    A[Uporabnik] -->|HTTP| B[Frontend - React]
    B -->|HTTP API| C[Backend - Node.js/Express]
    C -->|MongoDB URI| D[MongoDB Atlas]
    B -->|HTTP| E[Stream Server]
    C -->|Spotify API| F[Spotify Service]
    C -->|Lyrics API| G[Lyrics.ovh]
    C -->|Radio Browser API| H[Radio Browser]
    B -->|OpenStreetMap| I[OpenStreetMap]
```

### Tehnologije
- **Programski jezik**:
  - Frontend: JavaScript (React)
  - Backend: JavaScript (Node.js/Express)
  - Webhook: Python (Flask)
- **Podatkovna baza**: MongoDB Atlas (NoSQL, oblačna)
- **Komunikacijski protokoli**:
  - HTTP/HTTPS za frontend-backend, Spotify, Lyrics.ovh, Radio Browser, OpenStreetMap
  - TCP za povezavo z MongoDB Atlas
- **Spletni strežnik**:
  - Frontend: `serve` (za strežbo statičnih React datotek)
  - Backend: Express (Node.js)
  - Webhook: Flask
- **Interpreter**: Node.js (v18), Python (3.x)
- **Drugo**:
  - Docker za vsebniško okolje
  - Docker Compose za upravljanje več vsebnikov
  - Git za nadzor verzij
  - Azure VM (Ubuntu) za gostovanje

### Knjižnice in API-ji
- **Frontend**:
  - **React**: Za izgradnjo uporabniškega vmesnika (enostranska aplikacija, hitro renderiranje)
  - **Axios**: Za HTTP zahteve
  - **React Router**: Za navigacijo med stranmi brez osveževanja
  - **Leaflet**: Za interaktivni zemljevid
  - **Chart.js, D3.js**: Za vizualizacije (tortni, stolpčni grafi, treemap)
  - **Tailwind CSS**: Za stilizacijo
  - **Framer Motion**: Za animacije
  - **OpenStreetMap**: Za ploščice zemljevida
  - **Z razlogom**: React za prilagodljivost, Axios za enostavne zahteve, Leaflet za zemljevide, Chart.js/D3.js za analitiko, Tailwind za hitro stilizacijo, Framer Motion za animacije.
- **Backend**:
  - **Express**: Za izgradnjo REST API-ja (hitra in enostavna implementacija)
  - **Mongoose**: Za interakcijo z MongoDB (poenostavi delo z NoSQL bazo)
  - **jsonwebtoken**: Za avtentikacijo z JWT
  - **Spotify Web API**: Za integracijo s Spotify storitvami
  - **Lyrics.ovh API**: Za besedila pesmi
  - **Radio Browser API**: Za uvoz postaj
  - **Z razlogom**: Express za hitro API gradnjo, Mongoose za NoSQL, JWT za varnost, zunanje API-ji za dodatne funkcionalnosti.
- **Webhook**:
  - **Flask**: Za izgradnjo webhook strežnika (lahek in preprost za implementacijo)
  - **Z razlogom**: Flask je minimalističen in primeren za majhne strežnike.

### Komunikacija
- **Protokoli**:
  - HTTP/1.1 za komunikacijo med frontendom in backendom
  - HTTPS za varno povezavo z MongoDB Atlas in Spotify API
  - Webhook POST zahteve za CI/CD
- **Odprta vrata**:
  - **3000**: Frontend (React, `serve`)
  - **5000**: Backend (Express)
  - **5001**: Webhook strežnik (Flask)
  - **22**: SSH za dostop do Azure VM

### Razredni diagram
#### Backend (Node.js/Express)
Primer za ustvarjanje postaje:
```mermaid
classDiagram
    class User {
        -String _id
        -String email
        -String password
        -Array favorites
        -Array listeningHistory
        -Object spotify
        +register()
        +login()
    }

    class Station {
        -String _id
        -String name
        -String streamUrl
        -String city
        -Array genre
        -String broadcastLanguage
        -String logo
        -Object coordinates
        -String userId
        +create()
        +getById()
        +getAll()
    }

    User "1" o-- "*" Station : creates
```

#### Frontend (React)
Frontend ne vsebuje tradicionalnih razredov, saj temelji na komponentah. Ključne komponente:
```mermaid
classDiagram
    class App {
        +render()
    }

    class Register {
        +handleSubmit()
        +render()
    }

    class Login {
        +handleSubmit()
        +render()
    }

    class StationList {
        +fetchStations()
        +render()
    }

    class RadioPlayer {
        +playStation()
        +fetchLyrics()
        +render()
    }

    class MapComponent {
        +renderMap()
        +zoomToStation()
    }

    class BottomPlayer {
        +playStream()
        +render()
    }

    App o-- Register : contains
    App o-- Login : contains
    App o-- StationList : contains
    App o-- RadioPlayer : contains
    RadioPlayer o-- MapComponent : contains
    RadioPlayer o-- BottomPlayer : contains
```

---

## DevOps (CI/CD)

### Potek dela
CI/CD pipeline avtomatizira gradnjo, objavo in posodabljanje aplikacije. Uporablja GitHub Actions za gradnjo in objavo Docker slik ter webhook za posodabljanje na Azure VM.

```mermaid
graph TD
    A[Push/Pull Request] -->|GitHub| B[GitHub Actions]
    B --> C[Build Backend Image]
    B --> D[Build Frontend Image]
    C --> E[Push to Docker Hub]
    D --> E
    E --> F[Trigger Webhook]
    F -->|HTTP POST| G[Azure VM Webhook]
    G --> H[Stop Containers]
    G --> I[Pull Images]
    G --> J[Generate docker-compose.yml]
    G --> K[Run Containers]
```

### Akcije
1. **Build Backend/Frontend**:
   - GitHub Actions (`ci-cd.yml`) zažene gradnjo slik:
     - `build-backend`: Zgradi in objavi `aleksandra06/spletno-backend:latest`.
     - `build-frontend`: Zgradi in objavi `aleksandra06/spletno-frontend:latest`.
   - Uporablja `docker/build-push-action@v4` in `secrets.DOCKERHUB_TOKEN` za prijavo.
2. **Webhook**:
   - Pošlje POST zahtevo na `http://104.40.152.85:5001/webhook`.
   - Webhook strežnik (`webhook.py`) na Azure VM:
     - Preveri `X-Webhook-Secret`.
     - Ustavi obstoječe vsebnike.
     - Povleče nove slike.
     - Ustvari `docker-compose.yml`.
     - Zažene vsebnike z `docker-compose up -d`.

### Dodatni potek
- **Testiranje backend kode**:
  - Datoteka `test-backend.yml` izvaja teste z uporabo Node.js in `npm test`.
  - Sproži se ob `push` ali `pull request` na vejah `main`/`development`.

### Izzivi in rešitve
- **Port konflikt**: Backend je zasedel port 5000, zato smo webhook premaknili na 5001.
- **NSG pravila**: Zaprt port 5001 je povzročil napako `ETIMEDOUT`. Dodali smo pravila za porte 3000, 5000, 5001.
- **Napačne poti v `ci-cd.yml`**: Popravili smo kontekste (`./WebApp/backend`).

---

## Varnost programske rešitve

### Požarni zid
- **Azure Network Security Group (NSG)**:
  - Pravila za omejitev dostopa:
    - **Port 22**: SSH, omejen na specifične IP naslove (če mogoče).
    - **Port 3000**: Frontend, odprt za vse (HTTP).
    - **Port 5000**: Backend API, odprt za vse (HTTP).
    - **Port 5001**: Webhook, odprt za GitHub Actions IP-je (če mogoče).
  - Pravila so nastavljena v Azure Portalu pod **Networking** → **Inbound security rules**.

### Omejitve uporabnikov
- **Azure VM**:
  - Uporabnik `triwave` z omejenimi pravicami (dodan v skupino `docker` za upravljanje vsebnikov).
  - Uporaba `sudo` za administrativne ukaze.
- **Docker**:
  - Vsebnik teče kot ne-root uporabnik znotraj vsebnika.
  - Omejen dostop do sistemskih virov z Docker konfiguracijo.

### Varnostne vloge uporabnikov
- **Aplikacija**:
  - JWT avtentikacija za zaščitene poti (npr. `/api/stations/new`).
  - Uporabniki imajo omejen dostop do lastnih podatkov (npr. samo lastnik lahko ureja svoje postaje).
- **GitHub**:
  - Uporaba `secrets` za shranjevanje občutljivih podatkov (npr. `DOCKERHUB_TOKEN`, `WEBHOOK_SECRET`).
- **Docker Hub**:
  - Zasebni repozitorij za backend zaradi občutljivih podatkov.

### Varovanje podatkov
- **MongoDB Atlas**:
  - Povezava prek HTTPS z uporabo `MONGODB_URI`.
  - Geslo shranjeno v `/etc/environment`, dostopno samo sistemu.
- **Spotify API**:
  - Client ID in Secret shranjena v `/etc/environment`.
  - Uporaba varnega `SPOTIFY_REDIRECT_URI`.
- **Webhook**:
  - Preverjanje `X-Webhook-Secret` za avtentikacijo zahtev.
- **Zgodovina ukazov**:
  - Čiščenje z `history -c` in `echo '' > ~/.bash_history` za preprečevanje razkritja občutljivih podatkov.

---

## Zaključek
Projekt TriWave je uspešno implementiral spletno aplikacijo za upravljanje radijskih postaj, z uporabo sodobnih tehnologij (React, Node.js, MongoDB Atlas, Docker) in DevOps praks (GitHub Actions, webhook). Kljub izzivom, kot so port konflikti, napačne konfiguracije in omejitve Azure VM, smo dosegli zanesljivo delovanje aplikacije in avtomatiziran CI/CD pipeline. Dokumentacija vključuje vse zahtevane elemente, vključno z UML diagrami, in zagotavlja celovit pregled projekta.