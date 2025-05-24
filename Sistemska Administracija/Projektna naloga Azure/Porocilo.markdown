# Poročilo za projektno nalogo: Azure

## Podatki o skupini

| **Ime skupine** | **Člani** |
|-----------------|-----------|
| TriWave         | Anđela Radaković, David Novak, Aleksandra Mickoska |

## Uvod

Naša spletna aplikacija, razvita za predmet Spletno programiranje, omogoča upravljanje in predvajanje radijskih postaj. Aplikacija vključuje funkcionalnosti, kot so registracija in prijava uporabnikov, dodajanje in prikaz radijskih postaj ter predvajanje radijskih tokov. To poročilo opisuje postopek namestitve aplikacije v Docker (1. del), ustvarjanje brezplačnega Azure računa (2. del), vzpostavitev Linux navidezne naprave (3. del), raziskavo Azure portala (4. del) in vzpostavitev Docker aplikacije na navidezni napravi (5. del).

---

## 1. Namestitev aplikacije v Docker okolju

### Opis postopka

Aplikacija je razdeljena na dva dela:
- **Frontend**: React aplikacija, ki teče na portu 3000.
- **Backend**: Node.js/Express API, ki teče na portu 5000 in se povezuje z MongoDB Atlas.
Uporabili smo MongoDB Atlas kot oblačno podatkovno bazo, kar izpolnjuje zahtevo po nelokalni bazi. Za razvoj smo uporabljali Git, vse konfiguracijske datoteke pa so shranjene v repozitoriju.

Cilj je namestiti našo spletno aplikacijo v Docker okolju z uporabo `Dockerfile` in `docker-compose.yml`. 

**Zakaj potrebujemo `Dockerfile` in `docker-compose.yml`?**

Docker omogoča, da aplikacijo zapakiramo v izolirano okolje – **vsebnike** (containers), ki vsebujejo vse potrebne odvisnosti, knjižnice in konfiguracije. To omogoča enostavno prenosljivost, doslednost med okolji (lokalno, testno, produkcijsko) in lažje odpravljanje težav.
- **Dockerfile**:
  - Opisuje **kako** zgraditi sliko za posamezen del aplikacije (npr. frontend ali backend). Vsebuje ukaze za nastavitev okolja, namestitev odvisnosti in zagon aplikacije.
  - Vsaka komponenta ima svoj Dockerfile, saj imajo različne zahteve in procese gradnje (npr. React mora zgraditi statične datoteke, Express pa zažene strežnik).
- **docker-compose.yml**:
  - Omogoča upravljanje **več vsebnikov hkrati** (multi-container aplikacija).
  - V našem primeru poveže frontend, backend in zunanjo podatkovno bazo (MongoDB Atlas).
  - Poenostavi zagon celotne aplikacije z enim ukazom, določa okoljske spremenljivke, mrežo in vrstni red zagona.

### Koraki namestitve

1. **Ustvarjanje Docker slik**:

   - Frontend `Dockerfile` (`WebApp/frontend/Dockerfile`):
     ```dockerfile
     FROM node:18
     WORKDIR /usr/src/app
     COPY package*.json ./
     RUN npm install --silent
     RUN npm install -g serve --silent
     COPY . .
     RUN npm run build
     CMD ["serve", "-s", "build", "-l", "3000"]
     ```
     - **Pojasnilo**:
       - `FROM node:18`: Uporabi Node.js 18 kot osnovno sliko.
       - `WORKDIR /usr/src/app`: Nastavi delovni imenik znotraj vsebnika.
       - `COPY package*.json ./`: Kopira `package.json` in `package-lock.json` za namestitev odvisnosti.
       - `RUN npm install --silent`: Namesti odvisnosti, zastavica `--silent` zmanjša izhodne dnevnike.
       - `RUN npm install -g serve --silent`: Globalno namesti `serve` za strežbo React aplikacije.
       - `COPY . .`: Kopira celotno kodo aplikacije.
       - `RUN npm run build`: Zgradi optimizirano različico React aplikacije.
       - `CMD ["serve", "-s", "build", "-l", "3000"]`: Zažene strežnik `serve`, zastavica `-s` omogoča enostransko aplikacijo, `-l 3000` določa vrata.
   - Backend `Dockerfile` (`WebApp/backend/Dockerfile`):
     ```dockerfile
     FROM node:18
     WORKDIR /usr/src/app
     COPY package*.json ./
     RUN npm install
     COPY . .
     CMD ["node", "index.js"]
     ```
     - **Pojasnilo**:
       - Enaki ukazi kot pri frontendu, razen `RUN npm run build` in `serve`, saj backend zažene Node.js aplikacijo z `node index.js`.
   - `docker-compose.yml`:
     ```yaml
     services:
       frontend:
         build:
           context: ./frontend
           dockerfile: Dockerfile
         ports:
           - "3000:3000"
         environment:
           - REACT_APP_API_URL=http://backend:5000
         depends_on:
           - backend
       backend:
         build:
           context: ./backend
           dockerfile: Dockerfile
         ports:
           - "5000:5000"
         environment:
           - MONGODB_URI=mongodb+srv://<user>:<password>@cluster0.mongodb.net/triwave
           - JWT_SECRET=moj_jwt_skrivnost
           - PORT=5000
     ```
     - **Pojasnilo**:
       - `services`: Definira frontend in backend storitvi.
       - `build.context`: Določa imenik z `Dockerfile` ( `./frontend` ali `./backend`).
       - `build.dockerfile`: Ime `Dockerfile` datoteke.
       - `ports`: Preslika vrata gostitelja na vrata vsebnika (npr. `3000:3000`).
       - `environment`: Nastavi okoljske spremenljivke znotraj vsebnika.
       - `depends_on`: Zagotovi, da se backend zažene pred frontendom.

2. **Zagon aplikacije**:
   - Zgradili in zagnali smo aplikacijo:
     ```powershell
     docker-compose up --build -d
     ```
     - **Pojasnilo**:
       - `docker-compose up`: Zažene storitve iz `docker-compose.yml`.
       - `--build`: Prisili gradnjo novih slik, tudi če že obstajajo.
       - `-d`: Zažene vsebnike v ozadju (detached mode).
    - **Dokazilo**: Uspešna gradnja in zagon sta prikazana na zaslonskih posnetkih.    
     ![Zdrabda aplikacije](Build.png)
     ![Zdrabda aplikacije1](Build1.png)


3. **Testiranje aplikacije**:
   - Odprli smo `http://localhost:3000` v brskalniku.
   - Testirali smo funkcionalnosti:
     - Registracija uporabnika (`/register`).
     - Prijava (`/login`) in dostop do zaščitenih strani (`/profile`, `/stations`).
     - Dodajanje postaje (`/stations/new`).
     - Prikaz podrobnosti postaje (`/stations/:id`) in predvajanje.
     - Predvajalnik (`/player`) z vnosom URL-ja toka.
     - Odjava (`/profile`).

### Težave in rešitve

Med namestitvijo smo naleteli na več izzivov, ki so zahtevali poglobljeno razumevanje Dockerja in aplikacijske arhitekture:

1. **Manjkajoči WSL (Windows Subsystem for Linux)**:
   - **Težava**: Pri prvem poskusu zagona `docker-compose up` smo dobili napako, da Docker Desktop ne more delovati, ker WSL ni bil nameščen na Windows sistemu. Docker Desktop na Windows zahteva WSL za zagon Linux vsebnikov.
   - **Rešitev**: Namestili smo WSL in posodobili sistem:
     ```powershell
     wsl --install
     wsl --update
     ```
     - **Pojasnilo**:
       - `wsl --install`: Namesti WSL in privzeto Ubuntu distribucijo.
       - `wsl --update`: Posodobi WSL na najnovejšo različico.
     - Po namestitvi smo ponovno zagnali Docker Desktop in poskusili zgraditi aplikacijo.

2. **Napačno klicanje API-ja (frontend namesto backend)**:
   - **Težava**: Frontend je pošiljal HTTP zahteve na lasten naslov (`http://localhost:3000/api/...`) namesto na backend (`http://backend:5000/api/...`). To je povzročilo napake, kot je `404 Not Found`, saj frontend ne gosti API-ja.
   - **Rešitev**: Popravili smo konfiguracijo v frontend kodi (npr. v `WebApp/frontend/src/api.js` ali podobni datoteki):
     ```javascript
     // Prej
     const API_URL = 'http://localhost:3000/api';
     // Potem
     const API_URL = process.env.REACT_APP_API_URL + '/api';
     ```
     - Preverili smo, da je `REACT_APP_API_URL=http://backend:5000` pravilno nastavljen v `WebApp/frontend/.env`.
     - Ponovno smo zgradili in zagnali aplikacijo:
       ```powershell
       docker-compose up --build -d
       ```
     - **Pojasnilo**: Napačna konfiguracija je izhajala iz nesporazuma o Docker omrežju, kjer storitve komunicirajo preko imen (npr. `backend`) znotraj `docker-compose` omrežja.

3. **Napaka pri Docker snapshotu**:
   - **Težava**: Pri gradnji frontend slike smo naleteli na napako:
     ```
     failed to prepare extraction snapshot ... parent snapshot ... does not exist
     ```
     Napaka je bila posledica poškodovanega Docker gradbenega predpomnilnika, verjetno zaradi prekinjene gradnje ali omejenega prostora na disku.
   - **Rešitev**: Počistili smo gradbeni predpomnilnik in ponovno zgradili brez uporabe predpomnilnika:
     ```powershell
     docker-compose down
     docker builder prune -a -f
     docker-compose up --build --no-cache -d
     ```
     - **Pojasnilo**:
       - `docker-compose down`: Ustavil in odstranil vse vsebnike ter omrežja.
       - `docker builder prune -a -f`: Odstranil ves gradbeni predpomnilnik, zastavica `-a` vključuje vse projekte, `-f` prepreči potrditveni poziv.
       - `docker-compose up --build --no-cache`: Zgradi slike brez uporabe predpomnilnika (`--no-cache`), kar zagotovi svežo gradnjo.
     - Po tem je gradnja uspela, kar je omogočilo zagon aplikacije.

- **Vsebniki (containers)** v Dockerju:
![Containers](Containers.png)

---

## 2. Ustvarjanje brezplačnega Azure računa

*dopolni*

---

## 3. Vzpostavitev Linux navidezne naprave

*dopolni*

---

## 4. Raziskava Azure portala

V okviru projekta smo raziskali Azure portal in odgovorili na naslednja vprašanja, povezana z uporabo Azure for Students naročnine. Vsi odgovori so podprti z zaslonskimi posnetki.

### 4.1 Kje in kako omogočite "port forwarding"?

Port forwarding v Azure portalu omogočimo z dodajanjem vhodnega varnostnega pravila v **Network Security Group (NSG)**, povezanega z navidezno napravo (VM). Za TriWave aplikacijo smo odprli port 3000 za frontend.

**Koraki**:
1. Prijava v Azure portal na `https://portal.azure.com`.
2. Poiskali smo **Virtual machines** in izbrali VM (npr. `TriWave-VM`).
3. V meniju smo izbrali **Networking** > **Network Security Group**.
4. V NSG smo izbrali **Inbound security rules** in kliknili **+ Add**.
5. Nastavili smo pravilo:
   - **Source**: Any
   - **Destination port ranges**: 3000
   - **Protocol**: TCP
   - **Action**: Allow
   - **Priority**: 100
   - **Name**: `Allow-Frontend-3000`
6. Kliknili smo **Add**.

**Rezultat**: Port 3000 je odprt za dostop do frontend aplikacije.

![Vhodno pravilo za port 3000](screenshots/port_forwarding.png)

### 4.2 Kakšen tip diska je bil dodan vaši navidezni napravi in kakšna je njegova kapaciteta?

Tip diska in kapaciteta VM-ja se preverita v Azure portalu pod nastavitvami diska.

**Koraki**:
1. Poiskali smo **Virtual machines** in izbrali VM.
2. Izbrali smo **Disks** pod **Settings**.
3. Preverili smo **OS disk** za vrsto in kapaciteto.

**Rezultat**: VM (`TriWave-VM`) uporablja **Standard SSD** disk za operacijski sistem s kapaciteto 128 GiB. Ni dodatnih podatkovnih diskov.

![Tip in kapaciteta diska](screenshots/vm_disk.png)

### 4.3 Kje preverimo stanje trenutne porabe virov v naši naročnini ("Azure for Students")?

Stanje porabe virov v naročnini Azure for Students preverimo pod **Subscriptions** v Azure portalu. Poraba je vidna 24 ur po vzpostavitvi.

**Koraki**:
1. V iskalno vrstico smo vnesli **Subscriptions** in izbrali **Azure for Students**.
2. Izbrali smo **Usage + quotas** pod **Settings**.
3. Izbrali smo vire (npr. Compute) in regijo (npr. West Europe) za prikaz tabele porabe.

**Rezultat**: Tabela prikazuje trenutno porabo (npr. 1 VM, 128 GiB diska) in omejitve naročnine.

![Poraba virov](screenshots/resource_usage.png)

---

## 5. Vzpostavitev Docker aplikacije na navidezni napravi

*To bodo dopolnili člani skupine.*

---

## Zaključek

Vzpostavitev aplikacije v Docker okolju je bila zahtevna, predvsem zaradi težav s konfiguracijo okolja (WSL, Docker cache, napačni API URL-ji), ki sta zahtevali poglobljeno razumevanje React strukture in Docker gradnje. S pomočjo sistematičnega odpravljanja težav (popravki, čiščenje predpomnilnika) smo uspešno namestili aplikacijo, ki deluje z MongoDB Atlas. Delo z Dockerjem nas je naučilo, kako strukturirati aplikacijo v ločene storitve in kako jih povezati s pomočjo docker-compose. Raziskava Azure portala pa je razširila naše znanje o upravljanju virov v oblačnem okolju, kar bo koristno za prihodnje projekte in implementacijo v produkcijo.