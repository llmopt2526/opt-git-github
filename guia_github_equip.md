# Guia de referència: Ús de GitHub en treball en equip

## 1. Introducció a l'ús de github com a eina per a desenvolupar en equip

Com hem vist, el control de versions és una tècnica essencial en el desenvolupament de programari que permet registrar l'historial de canvis d'un projecte, facilitar la col·laboració entre diversos desenvolupadors i mantenir la traçabilitat dels canvis. Git és el sistema de control de versions més utilitzat actualment, i GitHub és una plataforma que permet allotjar repositoris Git de manera remota, afegint funcionalitats per a la col·laboració, revisió i gestió de projectes.

GitHub és especialment útil per a:
- Treballar en equip de manera asincrònica.
- Gestionar branques i integració de canvis.
- Fer seguiment de tasques i incidències.
- Revisar codi mitjançant Pull Requests.

> 📎 [Documentació oficial de GitHub](https://docs.github.com/)

---

## 🧩 Conceptes clau: Branca i Fork

### Què és una branca?
Una **branca** (branch) és una línia de desenvolupament paral·lela dins del mateix repositori.  
Permet treballar en una funcionalitat o correcció concreta sense afectar el codi principal (`main` o `master`).  
Quan la branca està llesta i revisada, els canvis es poden fusionar (merge) amb la branca principal.

Exemple:
```bash
git checkout -b feature-header
```
👉 Això crea una nova branca anomenada `feature-header` per treballar-hi sense modificar `main`.

### Què és un fork?
Un **fork** és una còpia d’un repositori complet a un altre compte d’usuari de GitHub.  
S’utilitza per col·laborar en projectes als quals no tens accés directe d’escriptura (com repositoris públics).  
Després de fer canvis al fork, es pot proposar una integració al projecte original mitjançant una **Pull Request**.

> 💡 En resum:  
> - *Branca* → per treballar dins del mateix repositori.  
> - *Fork* → per crear la teva pròpia còpia d’un projecte aliè.

---

## 2. Configuració inicial d’un repositori compartit

### Creació del repositori
- Un membre de l'equip (normalment qui fa de coordinador) crea un repositori a GitHub amb nom representatiu, per exemple: `projecte-web-equipe1`.
- És recomanable afegir un fitxer `README.md` i, si es vol, un `.gitignore` adaptat a HTML/JS.

### Afegir col·laboradors
- A l'apartat **Settings > Collaborators**, s'afegeixen els membres de l'equip amb els seus usuaris de GitHub.

### Estructura inicial
```
projecte-web-equipe1/
├── index.html
├── css/
│   └── style.css
└── js/
    └── script.js
```

---

## 3. Clonació del repositori i entorn local de treball

Cada membre ha de clonar el repositori al seu ordinador:
```bash
git clone https://github.com/usuari/projecte-web-equipe1.git
```

Després, accedeix a la carpeta i crea una branca per treballar:
```bash
cd projecte-web-equipe1
git checkout -b nom-branca
```

---

## 4. Branching: treballar amb branques

Cada tasca o funcionalitat hauria de fer-se en una branca independent:
- `main` o `master`: branca principal i estable.
- `feature-header`, `fix-botons`, `estils-footer`, etc.: branques per funcionalitats específiques.

Crear una branca:
```bash
git checkout -b feature-nom
```

Canviar de branca:
```bash
git checkout main
```

Llistar branques:
```bash
git branch
```

---

## 5. Commits i sincronització

### Afegir canvis a l'índex
Els commits **sempre s’han de fer des de la branca de treball (feature-nom)**, no directament a `main`.

Exemple:
```bash
git add fitxer.html
git commit -m "Afegit l'encapçalament amb logo i menú"
```
Així els canvis queden registrats a la teva branca (`feature-nom`) i més endavant es podran integrar a `main` mitjançant una *Pull Request*.

### Sincronitzar amb el repositori remot
```bash
git pull origin main  # Abans de pujar els canvis
git push origin nom-branca
```

---

## 6. Resolució de conflictes

Els conflictes es produeixen quan dos usuaris han modificat la mateixa línia d’un fitxer. Git avisarà i marcarà les línies en conflicte així:
```html
<<<<<<< HEAD
Contingut de la branca local
=======
Contingut de la branca remota
>>>>>>> nom-branca
```

El desenvolupador ha de decidir quina versió mantenir, esborrar els marcadors i fer un nou commit.

---

## 7. Pull Requests i revisió de codi

Un cop acabada una tasca:

1. Assegura’t que estàs a **la teva branca de treball (feature-nom)** i que el repositori local està actualitzat amb el remot:
```bash
git checkout feature-nom
git fetch origin
git merge origin/main
```

2. Fes `push` de la teva branca (un cop sincronitzada):
```bash
git push origin feature-nom
```

3. A GitHub, crea una **Pull Request (PR)** cap a `main` perquè els companys puguin revisar els canvis.

4. Un altre membre revisa els canvis, comenta si cal i aprova.

> 💡 **Consell:** fer `fetch` i `merge` abans de pujar canvis ajuda a evitar conflictes i assegura que treballes amb la versió més recent del projecte.


---

## 8. Integració de canvis: merge de branques

Un cop aprovada la PR:
- Es fa el **merge** des de GitHub cap a `main`.
- Opcionalment, es pot esborrar la branca.

També es pot fer merge manualment:
```bash
git checkout main
git pull origin main
git merge nom-branca
git push origin main
```

---

## 9. Exemple pràctic: desenvolupament col·laboratiu d’un lloc web

### Context
Un equip de 3 persones desenvolupa una web informativa sobre un festival local.

### Tasques dividides
- **Persona A**: estructura HTML general, secció de contacte.
- **Persona B**: estils CSS i layout responsive.
- **Persona C**: funcionalitats JavaScript com un slider d’imatges i validació de formulari.

### Flux de treball
1. Tots clonen el repositori i creen branques (`html-estructura`, `css-estils`, `js-funcions`).
2. Cada membre treballa localment, fa commits i puja canvis.
3. Quan acaben, creen una Pull Request.
4. Els altres membres revisen i aproven.
5. Es fa merge a `main` i es comprova la integració.

---

## 10. Bones pràctiques en treball en equip amb GitHub

- Fer commits freqüents i amb missatges descriptius.
- Treballar sempre en branques, no directament a `main`.
- Fer `pull` abans de començar a treballar per evitar conflictes.
- Revisar les PR amb atenció i fer comentaris constructius.
- Mantenir una estructura clara al repositori.

---

## 11. Resum i recursos addicionals

Aquesta guia ha presentat un flux bàsic però complet per treballar en equip amb GitHub sense automatitzacions. És aplicable a qualsevol projecte web senzill i serveix com a base per incorporar metodologies més avançades com CI/CD.

### Recursos útils
- 📘 [Git Handbook (GitHub)](https://guides.github.com/introduction/git-handbook/)
- 📗 [Pro Git Book (en línia i gratuït)](https://git-scm.com/book/en/v2)
- 🧾 [Cheat sheet de Git](https://education.github.com/git-cheat-sheet-education.pdf)

> ℹ️ Aquesta guia no inclou GitHub Actions ni altres eines d'integració contínua, ja que se centra exclusivament en la gestió col·laborativa i la validació manual del codi.
