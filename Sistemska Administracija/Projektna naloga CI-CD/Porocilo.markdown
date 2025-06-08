# Poročilo za Projektno Nalogo 2: CI/CD Pipeline

## Podatki o skupini

| **Ime skupine** | **Člani** |
|-----------------|-----------|
| TriWave         | Anđela Radaković, David Novak, Aleksandra Mickoska |

## Uvod
Cilj projektne naloge je bil vzpostaviti CI/CD pipeline za spletno aplikacijo, razvito v okviru predmeta Spletno Programiranje. Pipeline uporablja Docker Hub za shranjevanje Docker slik, GitHub Actions za avtomatizacijo gradnje in objave, ter webhook strežnik na Azure virtualnem stroju (VM) za posodabljanje aplikacije ob vsakem `push` ali `merge` v repozitorij. Aplikacija je sestavljena iz dveh delov: **backend** (Node.js, MongoDB) in **frontend** (React), ki se izvajata v Docker vsebnikih. Pipeline zagotavlja, da se ob spremembah kode zgradijo nove Docker slike, naložijo na Docker Hub, in samodejno posodobijo na VM preko webhooka.

Projekt smo izvedli v treh fazah:
1. **Docker Hub**: Nastavitev računa, repozitorijev in ročna gradnja/push slik.
2. **GitHub Actions**: Implementacija delovnega toka za avtomatizacijo gradnje in webhook sprožanja.
3. **Webhook**: Vzpostavitev strežnika na Azure VM za sprejemanje webhook zahtev in posodabljanje vsebnikov.

Skupina TriWave je delovala v sprintih, uporabila je orodja za vodenje projekta (Jira/OpenProj), in dokumentirala vse korake, izzive in rešitve v tem poročilu.

---

## 1. Docker Hub nastavitev

### 1.1 Račun in repozitoriji
- **Ustvarjanje računa**:
  - Vodja skupine je ustvarila račun na [Docker Hub](https://hub.docker.com) z uporabniškim imenom `aleksandra06`.
  - Uporabili smo brezplačni načrt, ki omogoča en zasebni repozitorij in neomejeno javnih repozitorijev.
  - Dostopni žeton (Personal Access Token) smo generirali v **Settings** → **Security** → **New Access Token** za varno prijavo iz ukazne vrstice.

- **Repozitoriji**:
  - Ustvarili smo dva repozitorija:
    - `aleksandra06/spletno-backend` (zasebni, za backend aplikacijo).
    - `aleksandra06/spletno-frontend` (javni, za frontend aplikacijo).
  - Zasebni repozitorij smo uporabili za backend zaradi občutljivih podatkov (npr. MongoDB povezave, Spotify API ključi).
  - Repozitorije smo ustvarili v Docker Hub vmesniku z gumbom **Create Repository**, kjer smo določili imena in vidnost.

  ![Docker Hub repozitoriji](Repos.png)

### 1.2 Gradnja in objava Docker slik
- **CLI ukazi**:
  ```bash
  docker login -u aleksandra06 -p <dostopni-žeton>
  cd WebApp/backend
  docker build -t aleksandra06/spletno-backend:latest .
  docker push aleksandra06/spletno-backend:latest
  cd ../frontend
  docker build -t aleksandra06/spletno-frontend:latest .
  docker push aleksandra06/spletno-frontend:latest
  ```
  - **Pojasnilo**:
    - `docker login`: Prijava v Docker Hub z uporabniškim imenom in dostopnim žetonom.
    - `cd WebApp/backend`: Premik v mapo z backend kodo, kjer je `Dockerfile`.
    - `docker build`: Zgradi Docker sliko iz `Dockerfile` in označi z `latest`.
    - `docker push`: Naloži sliko na Docker Hub repozitorij.
    - Enak postopek za frontend.

  ![Backend gradnja](Build-backend.png)
  ![Frontend gradnja](Build-frontend.png)
  ![Slike na repozitorijih](Images-repo.png)

### 1.3 Izzivi in rešitve
Ni bilo težav.

---

## 2. GitHub Actions delovni tok

### 2.1 Uvod
GitHub Actions smo uporabili za avtomatizacijo CI/CD pipelinea. Delovni tok (`ci-cd.yml`) izvaja tri naloge:
1. Gradi in objavi backend sliko (`spletno-backend`).
2. Gradi in objavi frontend sliko (`spletno-frontend`).
3. Sproži webhook za posodobitev aplikacije na VM.

Delovni tok se sproži ob `push` ali `pull request` na vejah `main` ali `development`.

### 2.2 Postopek
- **Ustvarjanje datoteke**:
Ustvarili smo mapo `.github/workflows` v naš repozitorij, smo dodali datoteko `ci-cd.yml` in smo shranili in naložili datoteko v repozitorij na vejo `development`.

- **Datoteka**: `.github/workflows/ci-cd.yml`:
  ```yaml
  name: CI/CD Pipeline

  on:
    push:
      branches:
        - main
        - development
    pull_request:
      branches:
        - main
        - development

  jobs:
    build-backend:
      runs-on: ubuntu-latest
      steps:
        - name: Checkout code
          uses: actions/checkout@v3

        - name: Log in to Docker Hub
          uses: docker/login-action@v2
          with:
            username: ${{ secrets.DOCKERHUB_USERNAME }}
            password: ${{ secrets.DOCKERHUB_TOKEN }}

        - name: Build and push backend image
          uses: docker/build-push-action@v4
          with:
            context: ./WebApp/backend
            file: ./WebApp/backend/Dockerfile
            push: true
            tags: ${{ secrets.DOCKERHUB_USERNAME }}/spletno-backend:latest

    build-frontend:
      runs-on: ubuntu-latest
      steps:
        - name: Checkout code
          uses: actions/checkout@v3

        - name: Log in to Docker Hub
          uses: docker/login-action@v2
          with:
            username: ${{ secrets.DOCKERHUB_USERNAME }}
            password: ${{ secrets.DOCKERHUB_TOKEN }}

        - name: Build and push frontend image
          uses: docker/build-push-action@v4
          with:
            context: ./WebApp/frontend
            file: ./WebApp/frontend/Dockerfile
            push: true
            tags: ${{ secrets.DOCKERHUB_USERNAME }}/spletno-frontend:latest

    deploy:
      runs-on: ubuntu-latest
      needs: [build-backend, build-frontend]
      steps:
        - name: Trigger webhook
          uses: joelwmale/webhook-action@master
          with:
            url: ${{ secrets.WEBHOOK_URL }}
            headers: '{"X-Webhook-Secret": "${{ secrets.WEBHOOK_SECRET }}"}'
            body: |
              {
                "event": "new_image",
                "images": {
                  "backend": "${{ secrets.DOCKERHUB_USERNAME }}/spletno-backend:latest",
                  "frontend": "${{ secrets.DOCKERHUB_USERNAME }}/spletno-frontend:latest"
                }
              }
  ```

  - **Pojasnilo**:
    - `on`: Določa sprožilce (push ali PR na `main`/`development`).
    - `jobs`: Tri naloge:
      - `build-backend`: Prijavi v Docker Hub, zgradi in objavi backend sliko.
      - `build-frontend`: Enako za frontend.
      - `deploy`: Pošlje POST zahtevo na webhook URL.
    - `secrets`: Uporablja GitHub Secrets za varno shranjevanje občutljivih podatkov.

### 2.3 GitHub Secrets
- **Dodajanje skrivnosti**:
  - V repozitoriju smo odšli na **Settings** → **Secrets and variables** → **Actions** → **New repository secret**.
  - Dodali smo:
    - `DOCKERHUB_USERNAME`: `aleksandra06` (uporabniško ime za Docker Hub).
    - `DOCKERHUB_TOKEN`: Dostopni žeton, generiran v Docker Hub (Settings → Security).
    - `WEBHOOK_URL`: `http://104.40.152.85:5001/webhook` (sprva na portu 5000, posodobljen na 5001 zaradi konflikta).
    - `WEBHOOK_SECRET`: kodni žeton za preverjanje webhook zahtev.
  - **Lokacija**: Skrivnosti so shranjene v GitHub repozitoriju, dostopne samo v Actions delovnih tokovih.

  ![Github Secrets](Secrets.png)

### 2.4 Izzivi
- **Težava 1**: Napačne poti do `Dockerfile`.
  - **Vzrok**: V `ci-cd.yml` smo sprva določili napačne kontekste (npr. `./backend` namesto `./WebApp/backend`).
  - **Rešitev**: Posodobili smo poti v `ci-cd.yml`:
    ```yaml
    context: ./WebApp/backend
    file: ./WebApp/backend/Dockerfile
    ```
- **Težava 2**: Nepravilni `WEBHOOK_URL`.
  - **Vzrok**: Sprva smo uporabili port 5000, ki je bil zaseden z backend vsebnikom.
  - **Rešitev**: Posodobili `WEBHOOK_URL` na `http://104.40.152.85:5001/webhook` in spremenili `webhook.py` za uporabo porta 5001.
- **Težava 3**: `deploy` job neuspeh:
  ```bash
  FetchError: request to 104.40.152.85:5001 failed, reason: connect ETIMEDOUT
  ```
  - **Vzrok**: Zaprt port 5001 v Azure NSG ali neaktiven `webhook.service`.
  - **Rešitev**:
    - Dodali NSG pravila za porte 5001, 5000, 3000.
    - Preverili status storitve:
      ```bash
      sudo systemctl status webhook.service
      ```
### 2.5 Možni dodatni delovni toki
Za izboljšanje kakovosti kode in zgodnje odkrivanje napak bi lahko v naš projekt dodali dodatni GitHub Actions delovni tok za preverjanje kode z orodjem za lintanje, kot je ESLint. Namen tega delovnega toka bi bil zagotavljanje kakovosti kode, doslednosti v slogu pisanja kode ter zgodnje odkrivanje potencialnih napak ali slabih praks v kodi, še preden se izvede gradnja Docker slik.

- **Namen**:
  - **Zagotavljanje kakovosti kode**: ESLint preverja skladnost kode z vnaprej določenimi pravili (npr. uporaba pravilnih formatov, izogibanje nepotrebnim spremenljivkam, pravilna uporaba funkcij).
  - **Zgodnje odkrivanje napak**: Odkrije sintaksne napake, potencialne hrošče (npr. nedefinirane spremenljivke) ali varnostne pomanjkljivosti (npr. nepravilna obdelava vnosov).
  - **Izboljšanje vzdrževanja kode**: Enoten slog kode olajša sodelovanje v skupini in vzdrževanje projekta.

- **Mesto v CI/CD pipeline-u**:
  - Delovni tok za lintanje bi bil umeščen **pred gradnjo Docker slik** v delovnem toku `ci-cd.yml`. To zagotavlja, da se slika zgradi le, če koda ustreza standardom kakovosti. Če lintanje ne uspe, se pipeline ustavi, kar prepreči gradnjo in objavo neustrezne kode.
  - Primer zaporedja: `Checkout code` → `Install Node.js and dependencies` → `Run ESLint` → `Build Docker images` (če lintanje uspe).

- **Primer strukture delovnega toka**:
  Spodaj je primer `.yml` datoteke za lintanje backend in frontend kode z ESLint:
  ```yaml
  name: Code Linting

  on:
    push:
      branches:
        - main
        - development
    pull_request:
      branches:
        - main
        - development

  jobs:
    lint:
      runs-on: ubuntu-latest
      steps:
        - name: Checkout code
          uses: actions/checkout@v3

        - name: Set up Node.js
          uses: actions/setup-node@v3
          with:
            node-version: '18'

        - name: Install backend dependencies
          run: |
            cd WebApp/backend
            npm install
            npm install eslint --save-dev

        - name: Run ESLint on backend
          run: |
            cd WebApp/backend
            npx eslint .

        - name: Install frontend dependencies
          run: |
            cd WebApp/frontend
            npm install
            npm install eslint --save-dev

        - name: Run ESLint on frontend
          run: |
            cd WebApp/frontend
            npx eslint .
  ```
  - **Pojasnilo**:
    - `actions/setup-node@v3`: Namesti Node.js (različica 18), ki je potreben za izvajanje ESLint.
    - `npm install`: Namesti odvisnosti projekta, vključno z ESLint.
    - `npx eslint .`: Zažene ESLint za preverjanje vseh `.js` ali `.jsx` datotek v mapi.
    - Delovni tok se izvede ob vsakem `push` ali `pull request` na vejah `main` ali `development`.

- **Potrebna orodja in odvisnosti**:
  - **Node.js**: Potreben za izvajanje ESLint in drugih JavaScript odvisnosti (različica 18 ali novejša).
  - **ESLint**: Orodje za lintanje, ki ga namestimo kot razvojno odvisnost (`npm install eslint --save-dev`).
  - **Konfiguracija ESLint**: Potrebna je datoteka `.eslintrc.json` v backend in frontend mapah, ki določa pravila lintanja (npr. Airbnb slog ali lastna pravila). Primer:
    ```json
    {
      "env": {
        "node": true,
        "browser": true,
        "es2021": true
      },
      "extends": [
        "eslint:recommended",
        "plugin:react/recommended"
      ],
      "parserOptions": {
        "ecmaVersion": 12,
        "sourceType": "module"
      },
      "rules": {
        "no-unused-vars": "warn",
        "indent": ["error", 2]
      }
    }
    ```
  - **Dodatne odvisnosti**: Za frontend lahko dodamo `eslint-plugin-react` in `eslint-plugin-react-hooks` za React-specifična pravila.

---

## 3. Webhook implementacija

### 3.1 Uvod
Webhook strežnik smo implementirali na Azure VM (`TriWaveVM`) z uporabo Python Flask aplikacije (`webhook.py`). Strežnik sprejema POST zahteve od GitHub Actions, preveri `X-Webhook-Secret`, ustavi/obne obstoječe vsebnike, povleče nove slike iz Docker Hub, ustvari `docker-compose.yml`, in zažene vsebnike.

### 3.2 Nastavitev VM
- **Dostop**:
  ```bash
  ssh triwave@104.40.152.85
  ```
  - Uporabnik: `triwave`, VM IP: `104.40.152.85`.

- **Namestitev programske opreme**:
  ```bash
  sudo apt-get update
  sudo apt-get install -y docker.io docker-compose python3-pip
  sudo apt install python3-flask
  pip3 install flask
  sudo systemctl start docker
  sudo systemctl enable docker
  docker login -u aleksandra06
  ```
  - **Pojasnilo**:
    - `apt-get update`: Posodobi seznam paketov.
    - `install docker.io docker-compose python3-pip`: Namesti Docker, Docker Compose in Python.
    - `install python3-flask`: Namesti Flask za webhook strežnik.
    - `systemctl start/enable docker`: Zažene in omogoči Docker storitev.
    - `docker login`: Prijavi VM v Docker Hub za vlečenje slik.

### 3.3 Webhook strežnik
- **Datoteka**: `/home/triwave/webhook.py`:
  ```python
  from flask import Flask, request
  import subprocess
  import os

  app = Flask(__name__)

  SECRET_TOKEN = 'token'  # Must match WEBHOOK_SECRET in GitHub Secrets

  @app.route('/webhook', methods=['POST'])
  def webhook():
      # Verify webhook secret
      if request.headers.get('X-Webhook-Secret') != SECRET_TOKEN:
          return {'status': 'unauthorized'}, 403

      data = request.get_json()
      if data.get('event') == 'new_image' and 'images' in data:
          try:
              backend_image = data['images']['backend']
              frontend_image = data['images']['frontend']

              # Validate input
              if not backend_image or not frontend_image:
                  return {'status': 'invalid payload'}, 400

              # Stop and delete existing containers
              subprocess.run(['docker', 'stop', 'spletno-backend', 'spletno-frontend'], check=False)
              subprocess.run(['docker', 'rm', 'spletno-backend', 'spletno-frontend'], check=False)

              # Pull new images
              subprocess.run(['docker', 'pull', backend_image], check=True)
              subprocess.run(['docker', 'pull', frontend_image], check=True)

              # Generate docker-compose.yml
              with open('/home/triwave/docker-compose.yml', 'w') as f:
                  f.write(f"""
  version: '3'
  services:
    backend:
      image: {backend_image}
      container_name: spletno-backend
      ports:
        - "5000:5000"
      environment:
        - MONGODB_URI={os.getenv('MONGODB_URI')}
        - JWT_SECRET={os.getenv('JWT_SECRET')}
        - JWT_EXPIRY={os.getenv('JWT_EXPIRY')}
        - SESSION_SECRET={os.getenv('SESSION_SECRET')}
        - SPOTIFY_CLIENT_ID={os.getenv('SPOTIFY_CLIENT_ID')}
        - SPOTIFY_CLIENT_SECRET={os.getenv('SPOTIFY_CLIENT_SECRET')}
        - SPOTIFY_REDIRECT_URI={os.getenv('SPOTIFY_REDIRECT_URI')}
        - PORT=5000
      restart: always
      networks:
        - app-network
    frontend:
      image: {frontend_image}
      container_name: spletno-frontend
      ports:
        - "3000:3000"
      environment:
        - REACT_APP_API_URL=http://backend:5000
      depends_on:
        - backend
      restart: always
      networks:
        - app-network
  networks:
    app-network:
      driver: bridge
  """)
      # Run containers
      subprocess.run(['docker-compose', '-f', '/home/triwave/docker-compose.yml', 'up', '-d'], check=True)
      return {'status': 'success'}, 200
  except subprocess.CalledProcessError as e:
      return {'status': 'error', 'message': str(e)}, 500
  except Exception as e:
      return {'status': 'error', 'message': str(e)}, 500
  return {'status': 'ignored'}, 200

  if __name__ == '__main__':
      app.run(host='0.0.0.0', port=5001)
  ```

  - **Pojasnilo**:
    - Skripta ustvari Flask aplikacijo, ki posluša na portu 5001.
    - Preverja `X-Webhook-Secret` za avtentikacijo.
    - Ob prejemu veljavne zahteve:
      - Ustavi in odstrani obstoječe vsebnike (`spletno-backend`, `spletno-frontend`).
      - Povleče nove slike iz Docker Hub.
      - Ustvari `docker-compose.yml` z okolijskimi spremenljivkami iz `/etc/environment`.
      - Zažene vsebnike z `docker-compose up -d`.
    - Vrne JSON odgovore (npr. `{"status": "success"}`).

- **Namestitev**:
  ```bash
  nano /home/triwave/webhook.py
  pip3 install flask
  ```
  - **Pojasnilo**:
    - `nano`: Urejanje `webhook.py`.
    - `pip3 install flask`: Namesti Flask knjižnico.

### 3.4 Okoljske spremenljivke
- **Datoteka**: `/etc/environment`:
  ```bash
  sudo nano /etc/environment
  ```
  - Vsebina:
    ```
    MONGODB_URI=mongodb+srv://user:pass@cluster0.mongodb.net/spletno
    JWT_SECRET=myjwtsecret123
    JWT_EXPIRY=1h
    SESSION_SECRET=mysessionsecret456
    SPOTIFY_CLIENT_ID=abc123xyz789
    SPOTIFY_CLIENT_SECRET=def456uvw012
    SPOTIFY_REDIRECT_URI=http://104.40.152.85:5000/callback
    ```
  - **Pojasnilo**:
    - Zgornji podatki niso resnični.
    - `MONGODB_URI`: Povezava do MongoDB Atlas baze.
    - `JWT_SECRET`, `JWT_EXPIRY`: Za JWT avtentikacijo v backendu.
    - `SESSION_SECRET`: Za seje v backendu.
    - `SPOTIFY_CLIENT_ID`, `SPOTIFY_CLIENT_SECRET`, `SPOTIFY_REDIRECT_URI`: Za Spotify API integracijo.
    - Shranjene v `/etc/environment` za sistemsko dostopnost.
  - **Varnost**:
    ```bash
    history -c && echo '' > ~/.bash_history
    ```
    - Čiščenje zgodovine ukazov za preprečevanje razkritja občutljivih podatkov.

### 3.5 Systemd storitev
- **Datoteka**: `/etc/systemd/system/webhook.service`:
  ```bash
  sudo nano /etc/systemd/system/webhook.service
  ```
  - Vsebina:
    ```ini
    [Unit]
    Description=Webhook Server for CI/CD
    After=network.target

    [Service]
    User=triwave
    WorkingDirectory=/home/triwave
    ExecStart=/usr/bin/python3 /home/triwave/webhook.py
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
  - **Pojasnilo**:
    - `Description`: Opis storitve.
    - `After=network.target`: Čaka na omrežje.
    - `User`: Izvaja kot uporabnik `triwave`.
    - `ExecStart`: Zažene `webhook.py`.
    - `Restart=always`: Samodejno znova zažene ob napakah.

- **Upravljanje**:
  ```bash
  sudo systemctl daemon-reload
  sudo systemctl enable webhook.service
  sudo systemctl start webhook.service
  sudo systemctl status webhook.service
  ```
  - **Pojasnilo**:
    - `daemon-reload`: Osveži konfiguracijo `systemd`.
    - `enable`: Omogoči storitev ob zagonu sistema.
    - `start`: Zažene storitev.
    - `status`: Preveri stanje (npr. `active: running`).

  ![Webhook Status](Webhook-status.png)

### 3.6 Izzivi
1. **Težava**: Port konflikt na 5000:
   - **Log**: `Address already in use: Port 5000`
   - **Vzrok**: Backend vsebnik (`spletno-backend`) je zasedel port 5000.
   - **Rešitev**:
     - Spremenili port webhook strežnika na 5001 v `webhook.py`:
       ```python
       app.run(host='0.0.0', port=5001)
       ```
     - Posodobili `WEBHOOK_URL` na `http://104.40.152.85:5001/webhook`.

2. **Težava**: Neuspešen `deploy` job:
   - **Log**:
     ```bash
     FetchError: request to 104.40.152.85:5001 failed, reason: connect ETIMEDOUT
     ```
   - **Vzrok**: Zaprt port 5001 v NSG.
   - **Rešitev**:
     - Dodali NSG pravila za porte 5001, 5000, 3000 v Azure Portal.
     - Preverili povezljivost:
       ```bash
       curl http://104.40.152.85:5001/webhook
       ```

### 3.7 Testiranje
- **Preverjanje webhooka**:
  ```bash
  curl -X POST http://104.40.152.85:5001/webhook -H "Content-Type: application/json" -H "X-Webhook-Secret: Inotreal" -d '{
    "event": "new_image",
    "images": {
      "backend": "aleksandra06/spletno-backend:latest",
      "frontend": "aleksandra06/spletno-frontend:latest"
    }
  }'
  ```
  - **Pojasnilo**: Pošlje testno zahtevo, ki simulira GitHub Actions.

- **Preverjanje vsebnikov**:
  ```bash
  docker ps
  ```
  - **Pojasnilo**: Pokaže tekoče vsebnike (`spletno-backend`, `spletno-frontend`).

- **Preverjanje aplikacije**:
  - Backend: `curl http://104.40.152.85:5000/api/health`
  - Frontend: `http://104.40.152.85:3000` v brskalniku.

### 3.8 Varnostne luknje in predlagane izboljšave
Poleg že implementirane preverbe `X-Webhook-Secret` smo identificirali dodatne varnostne pomisleke in predlagamo naslednje izboljšave za prihodnjo implementacijo:

- **Potencialna ranljivost za injekcijo ukazov**:
  - **Težava**: Skripta `webhook.py` uporablja `subprocess.run` za izvajanje ukazov, kot so `docker stop`, `docker rm`, `docker pull` in `docker-compose up`. Če vnosi (npr. `backend_image` ali `frontend_image`) niso ustrezno očiščeni, bi napadalec lahko vstavil zlonamerne ukaze (npr. prek zlonamernega webhook payload-a).
  - **Zakaj je to problem**: Neustrezno čiščenje vnosov lahko omogoči izvajanje poljubnih ukazov na strežniku, kar lahko vodi do kompromitacije sistema (npr. izbris podatkov ali namestitev zlonamerne programske opreme).
  - **Predlagana rešitev**: V prihodnosti bi implementirali čiščenje vnosov v `webhook.py`, da preprečimo injekcijo ukazov. To bi dosegli z:
    - Preverjanjem, ali vnosi (`backend_image`, `frontend_image`) ustrezajo pričakovanemu formatu (npr. regularni izraz, ki dovoljuje samo veljavna imena Docker slik, kot je `aleksandra06/spletno-backend:latest`).
    - Uporabo knjižnice, kot je `shlex.quote`, za varno obdelavo vnosov pred posredovanjem v `subprocess.run`.
    - Primer implementacije:
      ```python
      import re
      def is_valid_image(image):
          pattern = r'^[a-zA-Z0-9_/:-]+$'
          return bool(re.match(pattern, image))
      
      if not is_valid_image(backend_image) or not is_valid_image(frontend_image):
          return {'status': 'invalid image name'}, 400
      ```
    - **Kako bi to implementirali**: Dodali bi funkcijo za preverjanje vnosov pred klicem `subprocess.run` in omejili dovoljene znake v imenih slik. To bi storili v naslednjem sprintu, saj trenutna implementacija predvideva zaupanja vredne vnose iz GitHub Actions.

- **Šibka validacija skrivnosti (WEBHOOK_SECRET)**:
  - **Težava**: Trenutna implementacija v `webhook.py` preverja `X-Webhook-Secret`, vendar poročilo ne obravnava moči skrivnosti ali tveganj za napade z grobo silo (brute-force attacks). Če je `WEBHOOK_SECRET` prekratka ali predvidljiva, bi napadalec lahko uganil vrednost in sprožil nepooblaščene webhook zahteve.
  - **Zakaj je to problem**: Šibka skrivnost omogoča nepooblaščen dostop do webhook endpointa, kar lahko vodi do nepooblaščenega posodabljanja vsebnikov ali drugih zlonamernih dejanj.
  - **Predlagana rešitev**:
    - **Uporaba kriptografsko varne skrivnosti**: V prihodnosti bi uporabili daljšo, naključno generirano skrivnost (npr. 32 znakov, generiranih z orodjem, kot je `secrets` modul v Pythonu). Primer:
      ```python
      import secrets
      webhook_secret = secrets.token_urlsafe(32)
      ```
    - **Periodično menjavanje skrivnosti**: Skrivnost bi menjali vsakih 3–6 mesecev ali ob sumu na kompromitacijo. To bi zahtevalo posodobitev `WEBHOOK_SECRET` v GitHub Secrets in `webhook.py`.
    - **Dodajanje IP omejitev v Azure NSG**: Za dodatno varnost bi omejili dostop do webhook endpointa (port 5001) samo na IP naslove GitHub Actions. Seznam IP naslovov je na voljo na [GitHub Meta API](https://api.github.com/meta). Primer pravila v Azure NSG:
      - Vir: Seznam IP naslovov GitHub Actions (npr. `192.30.252.0/22`).
      - Cilj: Port 5001 na VM (`104.40.152.85`).
      - Protokol: TCP.
      - Dejanje: Dovoli.
    - **Kako bi to implementirali**: V naslednjem sprintu bi generirali novo skrivnost, posodobili GitHub Secrets in `webhook.py`, ter dodali NSG pravilo v Azure Portal za omejitev IP naslovov. To bi zmanjšalo tveganje za nepooblaščen dostop.


---

## 4. Zaključek
CI/CD pipeline smo uspešno implementirali, kljub več izzivom (port konflikti, NSG pravila, sintaksne napake). Pipeline omogoča avtomatizirano gradnjo, objavo in posodabljanje aplikacije. 

Projekt je izpolnil vse zahteve naloge in zagotovil praktično izkušnjo z DevOps orodji.
