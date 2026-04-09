# 📦 Node.js — Guida Completa ai Comandi

> Riferimento completo per sviluppatori: comandi Node.js, npm, gestione processi/porte, curl, debugging e molto altro.

---

## 📋 Indice

1. [Node.js — Comandi Base](#1-nodejs--comandi-base)
2. [npm — Gestione Pacchetti](#2-npm--gestione-pacchetti)
3. [npx](#3-npx)
4. [Gestione Processi e Porte](#4-gestione-processi-e-porte)
5. [Variabili d'Ambiente](#5-variabili-dambiente)
6. [curl — Testare il Server](#6-curl--testare-il-server)
7. [Debugging](#7-debugging)
8. [PM2 — Process Manager](#8-pm2--process-manager)
9. [Nodemon — Sviluppo](#9-nodemon--sviluppo)
10. [Script package.json Utili](#10-script-packagejson-utili)
11. [REPL Interattivo](#11-repl-interattivo)
12. [Moduli e Import/Export](#12-moduli-e-importexport)
13. [Cheatsheet Rapido](#13-cheatsheet-rapido)

---

## 1. Node.js — Comandi Base

```bash
# Verificare la versione installata
node --version
node -v

# Eseguire un file JavaScript
node app.js
node index.js

# Eseguire codice JS inline direttamente da terminale
node -e "console.log('Hello World')"
node --eval "console.log(process.version)"

# Eseguire con un file .env (Node.js 20.6+)
node --env-file=.env app.js

# Eseguire con opzioni di memoria aumentata (utile per build pesanti)
node --max-old-space-size=4096 app.js

# Controllare la versione di V8 (engine JS di Node)
node -e "console.log(process.versions.v8)"

# Stampare tutte le versioni (node, v8, openssl, etc.)
node -e "console.log(process.versions)"

# Eseguire in modalità strict
node --use-strict app.js

# Eseguire con ES Modules abilitati (alternativa al campo "type":"module" nel package.json)
node --input-type=module < app.mjs

# Vedere tutte le flag disponibili
node --help
```

---

## 2. npm — Gestione Pacchetti

### Inizializzazione Progetto

```bash
# Creare un nuovo package.json (interattivo)
npm init

# Creare un package.json con valori di default (skip domande)
npm init -y
npm init --yes
```

### Installazione Pacchetti

```bash
# Installare tutte le dipendenze (dal package.json esistente)
npm install
npm i

# Installare un pacchetto e salvarlo nelle dipendenze
npm install express
npm i express

# Installare una versione specifica
npm install express@4.18.2

# Installare come devDependency (solo sviluppo, non produzione)
npm install --save-dev nodemon
npm i -D nodemon

# Installare come dipendenza globale (accessibile ovunque nel sistema)
npm install -g pm2
npm i -g nodemon

# Installare più pacchetti insieme
npm install express cors dotenv mongoose

# Installare solo le dipendenze di produzione (esclude devDependencies)
npm install --production
npm install --omit=dev

# Installare da un URL git
npm install https://github.com/user/repo.git

# Forzare reinstallazione anche se già presenti
npm install --force
```

### Rimozione Pacchetti

```bash
# Rimuovere un pacchetto
npm uninstall express
npm remove express
npm rm express

# Rimuovere un pacchetto globale
npm uninstall -g pm2

# Rimuovere e aggiornare package.json
npm uninstall --save express
```

### Aggiornamento Pacchetti

```bash
# Vedere i pacchetti aggiornabili
npm outdated

# Aggiornare tutti i pacchetti (rispetta i range in package.json)
npm update

# Aggiornare un pacchetto specifico
npm update express

# Aggiornare all'ultima versione (ignora i range semver)
npm install express@latest
```

### Audit e Sicurezza

```bash
# Controllare vulnerabilità nelle dipendenze
npm audit

# Correggere automaticamente le vulnerabilità
npm audit fix

# Correggere anche breaking changes
npm audit fix --force
```

### Ispezione e Info

```bash
# Listare i pacchetti installati nel progetto
npm list
npm ls

# Listare solo il primo livello (più leggibile)
npm list --depth=0

# Listare i pacchetti globali
npm list -g --depth=0

# Vedere info su un pacchetto
npm info express
npm view express

# Vedere la versione di un pacchetto installato
npm list express

# Cercare un pacchetto nel registry
npm search express

# Vedere il percorso della cartella globale
npm root -g

# Vedere la configurazione npm
npm config list

# Impostare un valore di configurazione
npm config set registry https://registry.npmjs.org/
```

### Pulizia Cache

```bash
# Pulire la cache di npm
npm cache clean --force

# Verificare l'integrità della cache
npm cache verify
```

---

## 3. npx

> `npx` esegue un pacchetto senza installarlo globalmente.

```bash
# Eseguire un pacchetto senza installarlo
npx create-react-app my-app
npx create-next-app@latest my-app

# Eseguire una versione specifica
npx cowsay@1.5.0 "Hello"

# Eseguire un file locale nella cartella node_modules/.bin
npx nodemon app.js
npx eslint .

# Eseguire senza cache (scarica sempre la versione più recente)
npx --no-install create-react-app my-app

# Forzare il download ignorando la cache
npx --ignore-existing cowsay "test"
```

---

## 4. Gestione Processi e Porte

> Sezione fondamentale: capita spesso che una porta sia già in uso e il server non parta.

### 🔍 Trovare il Processo che Occupa una Porta

```bash
# macOS / Linux — trovare chi usa la porta 3000
lsof -i :3000

# Linux — alternativa con ss
ss -tulpn | grep :3000

# Linux — con netstat
netstat -tulpn | grep :3000

# Windows — trovare chi usa la porta 3000
netstat -ano | findstr :3000
```

### 💀 Killare un Processo per Liberare una Porta

```bash
# macOS / Linux — kill tramite PID (il PID appare nella colonna dell'output di lsof)
kill -9 <PID>

# Esempio pratico: trovare e killare in un solo comando (macOS/Linux)
kill -9 $(lsof -t -i:3000)

# Versione più sicura (SIGTERM invece di SIGKILL)
kill $(lsof -t -i:3000)

# Linux — kill con fuser
fuser -k 3000/tcp

# Windows — kill tramite PID
taskkill /PID <PID> /F

# Windows — kill diretto per porta
for /f "tokens=5" %a in ('netstat -aon ^| findstr :3000') do taskkill /f /pid %a
```

### 🔍 Trovare e Gestire Processi Node.js

```bash
# Vedere tutti i processi node in esecuzione (macOS/Linux)
ps aux | grep node

# Killare tutti i processi node (ATTENZIONE: li termina tutti)
pkill -f node
killall node

# Killare un processo specifico per nome
pkill -f "node app.js"

# Windows — vedere i processi node
tasklist | findstr node

# Windows — killare tutti i processi node
taskkill /IM node.exe /F
```

### 📊 Monitorare le Connessioni di Rete

```bash
# Vedere tutte le porte in ascolto
lsof -i -P -n | grep LISTEN

# Vedere solo le porte TCP in ascolto
ss -tlnp                    # Linux
netstat -tlnp               # Linux
netstat -an | grep LISTEN   # macOS

# Vedere connessioni attive su una porta specifica
lsof -i :3000 -n -P
```

### ⚙️ Segnali di Sistema

```bash
# SIGTERM (15) — terminazione pulita (il processo può fare cleanup)
kill -15 <PID>
kill <PID>        # SIGTERM è il default

# SIGKILL (9) — terminazione immediata e forzata (non ignorabile)
kill -9 <PID>

# SIGHUP (1) — riavvio del processo (usato da alcuni server)
kill -1 <PID>

# Vedere tutti i segnali disponibili
kill -l
```

---

## 5. Variabili d'Ambiente

```bash
# Passare variabili inline (macOS/Linux)
PORT=4000 node app.js
NODE_ENV=production PORT=8080 node app.js

# Usare un file .env con dotenv
# 1. Installare dotenv
npm install dotenv

# 2. Nel codice (app.js)
# require('dotenv').config()

# 3. Creare il file .env
# PORT=3000
# DB_URL=mongodb://localhost/mydb

# Node.js 20.6+ — senza dotenv, direttamente da CLI
node --env-file=.env app.js

# Vedere le variabili d'ambiente attuali
printenv
env

# Vedere una variabile specifica
echo $PORT
echo $NODE_ENV
```

### Impostare NODE_ENV

```bash
# Sviluppo
NODE_ENV=development node app.js

# Produzione
NODE_ENV=production node app.js

# Test
NODE_ENV=test node app.js

# Windows (Command Prompt)
set NODE_ENV=production && node app.js

# Windows (PowerShell)
$env:NODE_ENV="production"; node app.js
```

---

## 6. curl — Testare il Server

> Tutti gli esempi assumono un server in ascolto su `http://localhost:3000`.

### GET

```bash
# GET semplice
curl http://localhost:3000

# GET con output formattato (se la risposta è JSON)
curl http://localhost:3000/api/users | json_pp
curl http://localhost:3000/api/users | python3 -m json.tool

# GET con headers visibili (request + response)
curl -v http://localhost:3000/api/users

# GET mostrando solo gli header di risposta
curl -I http://localhost:3000/api/users

# GET con header personalizzato
curl -H "Authorization: Bearer TOKEN123" http://localhost:3000/api/profile

# GET con più header
curl -H "Authorization: Bearer TOKEN123" \
     -H "Accept: application/json" \
     http://localhost:3000/api/users

# GET con parametri query string
curl "http://localhost:3000/api/users?page=1&limit=10"

# GET salvando la risposta in un file
curl http://localhost:3000/api/data -o risposta.json

# GET silenzioso (niente progress bar) — utile negli script
curl -s http://localhost:3000/api/users

# GET con timeout (5 secondi)
curl --max-time 5 http://localhost:3000/api/users

# GET seguendo i redirect
curl -L http://localhost:3000/redirect
```

### POST

```bash
# POST con body JSON
curl -X POST http://localhost:3000/api/users \
     -H "Content-Type: application/json" \
     -d '{"name": "Mario", "email": "mario@example.com"}'

# POST con body JSON su più righe (più leggibile)
curl -X POST http://localhost:3000/api/users \
     -H "Content-Type: application/json" \
     -d '{
       "name": "Mario Rossi",
       "email": "mario@example.com",
       "age": 30
     }'

# POST con dati da un file JSON
curl -X POST http://localhost:3000/api/users \
     -H "Content-Type: application/json" \
     -d @payload.json

# POST con form data (application/x-www-form-urlencoded)
curl -X POST http://localhost:3000/login \
     -d "username=mario&password=secret123"

# POST con multipart/form-data (upload file)
curl -X POST http://localhost:3000/upload \
     -F "file=@/path/to/file.jpg" \
     -F "description=La mia foto"

# POST con autenticazione Basic
curl -X POST http://localhost:3000/api/login \
     -u "admin:password" \
     -H "Content-Type: application/json" \
     -d '{"remember": true}'

# POST con Bearer Token
curl -X POST http://localhost:3000/api/posts \
     -H "Authorization: Bearer eyJhbGci..." \
     -H "Content-Type: application/json" \
     -d '{"title": "Nuovo Post", "body": "Contenuto..."}'
```

### PUT e PATCH

```bash
# PUT — sostituzione completa di una risorsa
curl -X PUT http://localhost:3000/api/users/1 \
     -H "Content-Type: application/json" \
     -d '{"name": "Mario Aggiornato", "email": "nuovo@example.com"}'

# PATCH — aggiornamento parziale di una risorsa
curl -X PATCH http://localhost:3000/api/users/1 \
     -H "Content-Type: application/json" \
     -d '{"email": "nuovaemail@example.com"}'
```

### DELETE

```bash
# DELETE semplice
curl -X DELETE http://localhost:3000/api/users/1

# DELETE con autenticazione
curl -X DELETE http://localhost:3000/api/users/1 \
     -H "Authorization: Bearer TOKEN123"

# DELETE con body (raro, ma supportato)
curl -X DELETE http://localhost:3000/api/items \
     -H "Content-Type: application/json" \
     -d '{"ids": [1, 2, 3]}'
```

### Autenticazione

```bash
# Basic Auth
curl -u "username:password" http://localhost:3000/api/protected

# Bearer Token (JWT)
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
     http://localhost:3000/api/me

# API Key nell'header
curl -H "X-API-Key: la-mia-api-key-segreta" http://localhost:3000/api/data

# API Key come query param
curl "http://localhost:3000/api/data?api_key=la-mia-api-key"

# Cookie
curl -b "session=abc123; token=xyz" http://localhost:3000/dashboard

# Salvare e riusare i cookie (simulare una sessione)
curl -c cookies.txt -X POST http://localhost:3000/login \
     -d "username=mario&password=secret"

curl -b cookies.txt http://localhost:3000/dashboard
```

### Opzioni curl Utili

```bash
# -v  Verbose: mostra request e response completi (header inclusi)
curl -v http://localhost:3000/api/users

# -s  Silent: niente progress bar (utile negli script)
curl -s http://localhost:3000/api/users

# -o  Output: salva la risposta in un file
curl -o output.json http://localhost:3000/api/users

# -w  Write-out: mostra statistiche dopo la richiesta
curl -s -o /dev/null -w "Status: %{http_code}\nTempo: %{time_total}s\n" \
     http://localhost:3000/api/users

# --compressed  Accettare risposta compressa (gzip)
curl --compressed http://localhost:3000/api/data

# --retry  Riprovare in caso di errore
curl --retry 3 http://localhost:3000/api/data

# -k  Ignorare errori SSL/TLS (utile in sviluppo con certificati self-signed)
curl -k https://localhost:3443/api/users

# Specificare HTTP version
curl --http1.1 http://localhost:3000/api/users
curl --http2 http://localhost:3000/api/users
```

### WebSocket (con wscat)

```bash
# Installare wscat
npm install -g wscat

# Connettersi a un WebSocket server
wscat -c ws://localhost:3000

# Con header personalizzato
wscat -c ws://localhost:3000 -H "Authorization: Bearer TOKEN"
```

---

## 7. Debugging

### Debugger Nativo di Node.js

```bash
# Avviare in modalità inspect (apre il debugger su porta 9229)
node --inspect app.js

# Avviare e mettere in pausa subito (breakpoint all'inizio)
node --inspect-brk app.js

# Debugger su una porta specifica
node --inspect=9230 app.js

# Poi aprire Chrome e andare su: chrome://inspect
```

### Logging e Profiling

```bash
# Abilitare i log di debug di Node.js internamente
NODE_DEBUG=http,net node app.js

# Profiling CPU
node --prof app.js
# Genera un file isolate-*.log — per processarlo:
node --prof-process isolate-*.log > profile.txt

# Profiling heap (memoria)
node --heap-prof app.js
```

### Eseguire Test

```bash
# Node.js 18+ ha un test runner built-in
node --test

# Eseguire un file di test specifico
node --test test/user.test.js

# Con Jest (il più comune)
npx jest
npx jest --watch           # modalità watch (riesegue al salvataggio)
npx jest --coverage        # con report di copertura

# Con Mocha
npx mocha test/**/*.test.js
```

---

## 8. PM2 — Process Manager

> PM2 è lo standard per eseguire Node.js in produzione: gestisce crash, restart automatici, log e cluster.

```bash
# Installare PM2 globalmente
npm install -g pm2

# Avviare un'app
pm2 start app.js

# Avviare con nome personalizzato
pm2 start app.js --name "my-api"

# Avviare in modalità cluster (sfrutta tutti i core della CPU)
pm2 start app.js -i max        # usa tutti i core
pm2 start app.js -i 4          # usa 4 core

# Avviare con variabili d'ambiente
pm2 start app.js --env production

# Vedere lo stato di tutte le app
pm2 list
pm2 status

# Monitoraggio in tempo reale (CPU, RAM, log)
pm2 monit

# Vedere i log
pm2 logs
pm2 logs my-api
pm2 logs --lines 100

# Riavviare un'app
pm2 restart my-api

# Ricaricare senza downtime (graceful reload)
pm2 reload my-api

# Fermare un'app
pm2 stop my-api

# Eliminare un'app dalla lista
pm2 delete my-api

# Avviare PM2 al boot del sistema
pm2 startup
pm2 save          # salva la lista delle app correnti

# Aggiornare PM2
pm2 update

# Usare un file di configurazione (ecosystem.config.js)
pm2 start ecosystem.config.js
pm2 start ecosystem.config.js --env production
```

#### Esempio ecosystem.config.js

```javascript
module.exports = {
  apps: [{
    name: 'my-api',
    script: 'app.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 8080
    },
    watch: false,
    max_memory_restart: '1G',
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss'
  }]
}
```

---

## 9. Nodemon — Sviluppo

> Nodemon riavvia automaticamente il server quando i file vengono modificati.

```bash
# Installare
npm install -g nodemon
# oppure come devDependency
npm install --save-dev nodemon

# Avviare con nodemon
nodemon app.js

# Specificare l'estensione dei file da monitorare
nodemon --ext js,json,env app.js

# Ignorare una cartella
nodemon --ignore node_modules/ --ignore tests/ app.js

# Con delay (aspetta 1 secondo prima di riavviare)
nodemon --delay 1000ms app.js

# Passare argomenti al processo Node
nodemon app.js -- --port 3000

# Eseguire un comando personalizzato al posto di node
nodemon --exec "npm test" app.js
```

#### Configurazione nodemon.json

```json
{
  "watch": ["src", "config"],
  "ext": "js,json,env",
  "ignore": ["node_modules", "*.test.js"],
  "delay": "500",
  "env": {
    "NODE_ENV": "development"
  }
}
```

---

## 10. Script package.json Utili

```json
{
  "scripts": {
    "start":       "node app.js",
    "dev":         "nodemon app.js",
    "dev:debug":   "nodemon --inspect app.js",
    "prod":        "NODE_ENV=production node app.js",
    "test":        "jest",
    "test:watch":  "jest --watch",
    "test:cov":    "jest --coverage",
    "lint":        "eslint .",
    "lint:fix":    "eslint . --fix",
    "format":      "prettier --write .",
    "build":       "tsc",
    "clean":       "rm -rf dist node_modules",
    "reinstall":   "rm -rf node_modules package-lock.json && npm install"
  }
}
```

```bash
# Eseguire gli script
npm start
npm run dev
npm test           # "test" non richiede "run"
npm run lint
npm run build

# Passare argomenti a uno script
npm run dev -- --port 4000

# Vedere tutti gli script disponibili
npm run
```

---

## 11. REPL Interattivo

> Il REPL (Read-Eval-Print Loop) permette di eseguire JS interattivamente nel terminale.

```bash
# Avviare il REPL
node

# Comandi speciali dentro il REPL
.help          # mostra tutti i comandi REPL
.exit          # uscire (anche: Ctrl+D o Ctrl+C due volte)
.clear         # resettare il contesto
.save file.js  # salvare la sessione in un file
.load file.js  # caricare ed eseguire un file

# Esempi di utilizzo nel REPL
> 2 + 2                          # 4
> const x = [1,2,3]
> x.map(n => n * 2)              # [2, 4, 6]
> require('path').join('a','b')  # 'a/b'
> process.env.PATH               # mostra la variabile PATH
> _                              # contiene il risultato precedente
```

---

## 12. Moduli e Import/Export

### CommonJS (default in Node.js)

```javascript
// Esportare
module.exports = { funzione, valore }
module.exports = classe

// Importare
const modulo = require('./modulo')
const { funzione } = require('./modulo')
const path = require('path')      // modulo built-in
const express = require('express') // pacchetto npm
```

### ES Modules (con "type": "module" in package.json o file .mjs)

```javascript
// Esportare
export const nome = 'valore'
export default function miaFunzione() {}

// Importare
import modulo from './modulo.js'
import { funzione } from './modulo.js'
import * as tutto from './modulo.js'
```

### Moduli Built-in Utili

```javascript
const path   = require('path')   // gestione percorsi file
const fs     = require('fs')     // file system
const http   = require('http')   // server HTTP
const https  = require('https')  // server HTTPS
const os     = require('os')     // info sistema operativo
const crypto = require('crypto') // crittografia
const events = require('events') // EventEmitter
const stream = require('stream') // stream
const util   = require('util')   // utilità (promisify, etc.)
const url    = require('url')    // parsing URL
const dns    = require('dns')    // DNS lookup
const child_process = require('child_process') // processi figli
```

---

## 13. Cheatsheet Rapido

### 🚀 Setup Nuovo Progetto

```bash
mkdir mio-progetto && cd mio-progetto
npm init -y
npm install express dotenv
npm install --save-dev nodemon
echo "node_modules\n.env" > .gitignore
echo "PORT=3000" > .env
touch app.js
```

### 🔥 Porta Occupata — Soluzione Rapida

```bash
# macOS / Linux
kill -9 $(lsof -t -i:3000)

# Linux alternativa
fuser -k 3000/tcp

# Windows
netstat -ano | findstr :3000
# poi con il PID trovato:
taskkill /PID <PID> /F
```

### 🧪 Test Rapido API con curl

```bash
# GET
curl -s http://localhost:3000/api/users | python3 -m json.tool

# POST JSON
curl -s -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Test"}' | python3 -m json.tool

# Mostra solo lo status code
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/health
```

### 📦 Comandi npm Essenziali

| Comando | Descrizione |
|---|---|
| `npm init -y` | Crea package.json con default |
| `npm install` | Installa le dipendenze |
| `npm install pkg` | Aggiunge una dipendenza |
| `npm install -D pkg` | Aggiunge una devDependency |
| `npm install -g pkg` | Installa globalmente |
| `npm uninstall pkg` | Rimuove una dipendenza |
| `npm update` | Aggiorna i pacchetti |
| `npm outdated` | Mostra pacchetti aggiornabili |
| `npm audit fix` | Corregge vulnerabilità |
| `npm list --depth=0` | Lista pacchetti installati |
| `npm run script` | Esegue uno script |
| `npm cache clean --force` | Pulisce la cache |

### 🌐 curl Cheatsheet

| Comando | Descrizione |
|---|---|
| `curl URL` | GET base |
| `curl -X POST URL` | POST |
| `curl -X PUT URL` | PUT |
| `curl -X PATCH URL` | PATCH |
| `curl -X DELETE URL` | DELETE |
| `-H "key: value"` | Aggiunge un header |
| `-d '{"key":"val"}'` | Body della richiesta |
| `-d @file.json` | Body da file |
| `-u user:pass` | Basic auth |
| `-v` | Verbose (mostra tutto) |
| `-s` | Silent (no progress) |
| `-I` | Solo header risposta |
| `-o file.json` | Salva output su file |
| `-L` | Segui redirect |
| `-k` | Ignora errori SSL |

---

*Generato come riferimento per sviluppatori Node.js — aggiornato ad aprile 2026*
