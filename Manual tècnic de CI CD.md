# Manual tècnic de CI/CD amb GitHub (nivell 1r GS Informàtica)

Pensat perquè l’alumnat **entengui i posi en marxa** un flux bàsic de **CI/CD** amb **GitHub** i **GitHub Actions**, sobre un projecte web senzill. Inclou un exemple **pas a pas** i bones pràctiques mínimes.

---

## 0) Què és CI/CD (en 2 minuts)

* **CI (Integració Contínua):** comprovo automàticament que el codi compila, passa *linters* i tests cada vegada que faig *push* o obro un *pull request*.
* **CD (Desplegament Contínu):** si tot va bé, el codi es desplega automàticament a un entorn (p. ex. **GitHub Pages**) sense passos manuals.

```
Dev → Push/PR → (CI) Lint+Tests+Build → (CD) Deploy → Web en línia
```

---

## 1) Requisits previs

* Compte a **GitHub**.
* Git instal·lat a l’ordinador.
* Editor (VS Code, etc.).
* Coneixements bàsics de Git (commit, branch, push) i HTML/CSS/JS.

---

## 2) Projecte d’exemple (web estàtica + proves)

Crearem un repositori amb:

```
ci-cd-web/
├─ src/
│  ├─ index.html
│  ├─ styles.css
│  └─ app.js
├─ tests/
│  └─ app.test.js
├─ package.json
└─ .github/
   └─ workflows/
      └─ ci-cd.yml   (workflow de GitHub Actions)
```

### 2.1 Inicialitza el projecte

```bash
mkdir ci-cd-web && cd ci-cd-web
git init
npm init -y
npm install --save-dev jest
```

### 2.2 Arxius mínims

**`src/index.html`**

```html
<!doctype html>
<html lang="ca">
  <head>
    <meta charset="utf-8" />
    <title>CI/CD Demo</title>
    <link rel="stylesheet" href="./styles.css" />
  </head>
  <body>
    <h1 id="title">Hola CI/CD 👋</h1>
    <script src="./app.js"></script>
  </body>
</html>
```

**`src/styles.css`**

```css
body { font-family: system-ui, sans-serif; max-width: 60ch; margin: 4rem auto; }
```

**`src/app.js`**

```js
export function greet(name) {
  return `Hola ${name}`;
}
console.log(greet("ASIX"));
```

**`tests/app.test.js`**

```js
import { greet } from "../src/app.js";

test("greet retorna el nom", () => {
  expect(greet("Món")).toBe("Hola Món");
});
```

**`package.json`** (afegiu scripts)

```json
{
  "name": "ci-cd-web",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "test": "jest --runInBand",
    "build": "cp -r src dist"
  },
  "devDependencies": {
    "jest": "^29.7.0"
  }
}
```

* `npm test` executa els tests.
* `npm run build` copia `src/` a `dist/` (llest per desplegar a Pages).

**Fes el primer commit i crea el repositori remot**

```bash
git add .
git commit -m "Inicial: web + tests"
gh repo create ci-cd-web --public --source=. --remote=origin --push
# (o crea'l manualment a GitHub i fes: git remote add origin ...; git push -u origin main)
```

---

## 3) CI: Workflow per validar (lint/tests/build)

Crea la carpeta de *workflows* i el fitxer:

**`.github/workflows/ci-cd.yml`**

```yaml
name: CI & CD

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

permissions:
  contents: read
  pages: write       # necessari per desplegar a GitHub Pages
  id-token: write    # necessari per a la publicació OIDC

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout codi
        uses: actions/checkout@v4

      - name: Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Instal·lar dependències
        run: npm ci

      - name: Executar tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Carregar artefacte per a Pages
        uses: actions/upload-pages-artifact@v3
        with:
          path: "dist"

  deploy:
    needs: ci
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: ${{ steps.deploy.outputs.page_url }}
    steps:
      - name: Configurar Pages
        uses: actions/configure-pages@v5

      - name: Desplegar a Pages
        id: deploy
        uses: actions/deploy-pages@v4
```

Què fa:

* **CI** (job `ci`): *checkout* → Node 20 → `npm ci` → `npm test` → `npm run build` → puja `dist/` com a artefacte.
* **CD** (job `deploy`): si és un *push* a `main`, configura **Pages** i **desplega** l’artefacte.

> **Primer cop**: a Settings → **Pages**:
>
> * **Build and deployment** = “**GitHub Actions**”.

Fes *push* a `main` i obre la pestanya **Actions** per veure el *pipeline*. Quan acabi, vés a **Settings → Pages** i copia l’URL del lloc.

---

## 4) Flux de treball recomanat (branques i PRs)

1. Crea una branca per cada tasca:

   ```bash
   git checkout -b feat/canvia-titol
   ```
2. Fes commits petits i clars.
3. Obre un **Pull Request** cap a `main`.
4. Deixa que **CI** s’executi al PR. Si falla, arregla-ho i torna a fer *push*.
5. Quan tot passa, demana **revisió** i **merge**.
6. En fer *merge* a `main`, el job `deploy` publicarà el nou *build* a **Pages**.

> **Opcional (recomanat):** a **Settings → Branches → Add rule**:
>
> * *Require a pull request before merging*
> * *Require status checks to pass before merging* i marca el workflow de CI.
> * Protegir `main` evita trencar producció.

---

## 5) Afegir qualitat bàsica (lint) — opcional però útil

```bash
npm install --save-dev eslint
npx eslint --init
```

Afegeix script i pas al workflow:

**`package.json`**

```json
"scripts": {
  "lint": "eslint .",
  "test": "jest --runInBand",
  "build": "cp -r src dist"
}
```

**workflow (`ci-cd.yml`)**

```yaml
  - name: Lint
    run: npm run lint
```

---

## 6) Secrets i variables (quan calguin)

* Si el desplegament necessita claus (API keys, etc.), desa-les a **Settings → Secrets and variables → Actions → New repository secret**.
* Al workflow, accedeix-hi amb `${{ secrets.NOM_DEL_SECRET }}`.
* **Mai** posis claus en el codi o *commits*.

(Connexió amb seguretat i dades: bones pràctiques de protecció i configuració de credencials per a serveis web i scripts d’automatització. )

---

## 7) Verificació ràpida & troubleshooting

**Checks bàsics**

* *Actions* mostra ✅ a `ci` i `deploy`.
* *Settings → Pages* mostra l’URL i l’últim desplegament.

**Errors freqüents**

* *Permisos insuficients per Pages*: assegura `pages: write` i `id-token: write` al bloc `permissions`.
* *Cap fitxer per desplegar*: comprova que `npm run build` crea `dist/` i que `upload-pages-artifact` apunta a `dist`.
* *Tests fallen*: revisa `tests/*.test.js` i la sortida de Jest.
* *Node no trobat*: confirma `actions/setup-node@v4` i una versió existent (“20” està bé).

---

## 8) Extensions útils (quan vulguis pujar de nivell)

* **Matriu de versions** (prova amb Node 18 i 20):

  ```yaml
  strategy:
    matrix:
      node: [18, 20]
  ...
  with:
    node-version: ${{ matrix.node }}
  ```
* **Anàlisi de codi** (CodeQL), **cobertura de tests** (p. ex. amb `jest --coverage` i publicació de *report*), **previews** en PR (deploy d’entorn temporal).

---

## 9) Checklist per a l’alumnat

* [ ] Repo creat amb estructura `src/`, `tests/`, `dist/`.
* [ ] Workflow `ci-cd.yml` amb **CI (lint/tests/build)** i **CD (deploy)**.
* [ ] **PRs** obligats a `main` i **status checks** requerits.
* [ ] Pàgina publicada a **GitHub Pages**.
* [ ] *README* amb passos de desenvolupament, com executar tests i enllaç a la web.
* [ ] (Opcional) *badge* d’estat del workflow:

  ```md
  ![CI](https://github.com/USUARI/ci-cd-web/actions/workflows/ci-cd.yml/badge.svg)
  ```

---

## 10) Rúbrica curta d’avaluació (4 nivells)

| Ítem            | 1 – Inicial      | 2 – Bàsic         | 3 – Bo            | 4 – Excel·lent             |
| --------------- | ---------------- | ----------------- | ----------------- | -------------------------- |
| Estructura repo | Falta / desorden | Bàsica            | Completa          | + bones pràctiques         |
| CI (tests/lint) | Inexistent       | S’executa parcial | Passa estable     | Matrius, cobertura         |
| CD (deploy)     | No desplega      | Manual            | Automàtic a Pages | Entorns/Regles             |
| Flux Git        | Directe a `main` | Branques sense PR | PRs i checks      | Regles i revisió           |
| Documentació    | Escassa          | Útil bàsica       | Bona i clara      | Completa + troubleshooting |

---

## 11) Connexions amb resultats d’aprenentatge del cicle

* **Automatització i scripts** (planificació de tasques, documentació): coherent amb *0374ra3* i *0374ra7*, sobre automatització i ús d’scripts en sistemes operatius. 
* **Implantació web i publicació**: s’alinea amb *0376ra1* (preparar entorns), *0376ra5* (documents web amb guions de servidor — aquí en versió client, inici) i *0376ra3* (administrar i publicar). 
* **Seguretat bàsica i bones pràctiques** (secrets, control d’accés, revisions): s’emmarca en *0378* (seguretat i alta disponibilitat). 

---

## 12) Resum final

* Has creat un **pipeline CI/CD** amb **GitHub Actions** que:

  * Valida el codi (tests, lint).
  * Genera un *build*.
  * **Desplega** automàticament a **GitHub Pages** en *merge* a `main`.
* Has configurat un **flux amb branques i PRs** perquè *producció* sigui fiable.

Quan ho tinguis llest, comparteix **URL de Pages** i **enllaç al PR** final per a la revisió.

