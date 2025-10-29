# Comprendre què vol dir *rebasar* una branca en Git

## 🧩 Com interpretar els esquemes de commits en Git

Quan es mostra un esquema com aquest:

```
A---B---C
```

o aquest altre:

```
D---E
```

cada lletra representa un **commit** (un punt de control amb canvis al projecte), i els guions (`---`) indiquen la **seqüència temporal i la relació** entre commits.

Per exemple:
- `A` pot ser el primer commit (creació del projecte)
- `B` pot representar un segon commit (afegir un fitxer o secció nova)
- `C` seria el següent commit amb més canvis

Quan hi ha branques, l'estructura pot veure’s així:

```
main:     A---B---C
                \
feature:         D---E
```

Això significa que:
- La branca `main` té els commits `A`, `B` i `C`.
- A partir del commit `B` algú va crear una branca `feature` i hi ha fet els commits `D` i `E`.
- Les dues branques tenen històries que **han divergit**.

---

## 🔍 Què vol dir *rebasar* (rebase)

**Rebasar** una branca significa **recol·locar els teus commits al damunt dels commits més recents del remot**, com si haguessis començat a treballar després que aquests canvis s’haguessin fet.  
En lloc de fer una fusió (*merge*) que crea un commit addicional, el rebase reescriu l’historial per fer-lo lineal i més net.

### Exemple visual

**Abans del rebase:**
```
main:   A---B---C
              \
feature:       D---E
```

**Després del rebase (`git rebase main`):**
```
main:   A---B---C---D'---E'
```

Els commits `D` i `E` es reescriuen com `D'` i `E'` sobre la base actual de `main`.  
El codi final és el mateix, però l’historial queda lineal.

---

## 🔍 Diferències pràctiques: què passa amb els canvis en cada cas

Imagina que:
- Has fet canvis **locals** a `main` (per exemple, has modificat `index.html` i fet `git commit`).
- Mentrestant, el teu company ha fet *push* d’altres canvis al remot (també a `main`).
- Ara el teu repositori local i el remot **han divergint**.

Quan fas `git pull`, Git ha d’unificar ambdós conjunts de canvis.  
Les opcions que mostra el missatge d’ajuda determinen **com s’unifiquen** aquests canvis i **quin historial queda**.

---

### 🟢 1. `git config pull.rebase false` → *fusionar* (*merge*)

**Què fa:**
- Baixa els commits remots.
- Combina els teus commits locals amb els remots **creant un commit nou de fusió**.
- **Cap canvi es perd.**

**Resultat:**
```
A---B---C---D   ← local
     \
      E---F     ← remot
       \
        G       ← commit de fusió (merge)
```

➡️ Tots els canvis (locals i remots) queden **conservats**.  
Només s’afegeix un **nou commit de fusió (G)** per connectar les dues històries.

**Avantatge:** segur, res es perd.  
**Inconvenient:** l’historial pot quedar més “brut” (amb commits de fusió addicionals).

---

### 🟡 2. `git config pull.rebase true` → *rebasar* (*rebase*)

**Què fa:**
- Descarrega els commits remots.
- Mou temporalment els teus commits locals fora de la branca.
- Aplica els commits remots.
- Torna a aplicar els teus commits al damunt (com si els haguessis fet després).

**Resultat:**
```
A---B---C---E---F---D'
```

➡️ El contingut del projecte és el mateix, **tots els canvis es conserven**,  
però els teus commits (`D`) es **reescriuen com a nous commits (`D'`)**.  
És a dir, es canvia l’ordre per tenir un historial lineal.

**Avantatge:** historial molt net i seqüencial.  
**Inconvenient:** si hi ha conflictes, s’han de resoldre manualment commit per commit.  
⚠️ Els *hash* dels teus commits canvien, així que **no s’ha d’usar en branques compartides** sense coordinació.

---

### 🔵 3. `git config pull.ff only` → *només avanç ràpid* (*fast-forward only*)

**Què fa:**
- Només actualitza si el teu historial és exactament una extensió del remot.
- Si hi ha divergència, Git **no fa res** i mostra un error.

**Resultat (quan és possible):**
```
A---B---C---D---E   ← s’avança sense fusió
```

**Resultat (quan hi ha divergència):**
❌ Git cancel·la el *pull* i no toca res fins que es resolgui manualment.

**Avantatge:** manté un historial 100% lineal i segur.  
**Inconvenient:** no resol divergències; només funciona si no hi ha commits locals nous.

---

### 🧭 En resum visual

| Opció | Canvis locals | Canvis remots | Canvis perduts | Historial resultant |
|--------|----------------|----------------|----------------|----------------------|
| **Merge** (`rebase false`) | ✅ Conservar | ✅ Conservar | ❌ Cap | Historial amb commit de fusió |
| **Rebase** (`rebase true`) | ✅ Conservar (reescrits) | ✅ Conservar | ❌ Cap | Historial lineal i net |
| **Fast-forward** (`ff only`) | ❌ No hi ha locals o s’aturen | ✅ Conservar | ❌ Cap | Historial net (només si no hi ha divergència) |

## ✅ Quan sí que s’utilitza *rebase*

El *rebase* és molt útil quan vols mantenir un historial **net i lineal** abans de fusionar una branca amb `main`.

### Escenari recomanat

Suposem que tens aquesta situació:

```
main:     A---B---C
                \
feature:         D---E
```

Has treballat a la branca `feature` mentre altres companys han continuat afegint canvis a `main`.  
Ara vols integrar els teus canvis, però primer vols assegurar-te que la teva branca està actualitzada amb la darrera versió de `main`.

### Passos per fer-ho correctament

1. Assegura’t que estàs a la teva branca:
   ```bash
   git checkout feature
   ```

2. Actualitza el teu repositori local amb els canvis més recents de `main`:
   ```bash
   git fetch origin
   ```

3. Rebaseja la teva branca sobre `main`:
   ```bash
   git rebase origin/main
   ```

**Resultat:**

```
main:     A---B---C
feature:              D'---E'
```

Ara la branca `feature` s’ha **alineat amb la versió més recent de `main`**, com si haguessis començat a treballar després dels commits `A`, `B` i `C`.  
Quan facis el *merge* a `main`, no es crearà cap commit addicional de fusió i l’historial quedarà lineal i clar.

---

### 💡 Avantatge principal
Fer un *rebase* **abans de fer el merge** és molt útil per:
- Mantenir un historial net i fàcil de llegir.
- Evitar commits de fusió innecessaris.
- Integrar-te millor amb canvis recents d’altres companys.

### ⚠️ Precaució
No facis *rebase* sobre una branca que **ja hagis compartit** o que altres persones també estiguin editant.  
Només fes-ho amb **branques personals de treball local** o abans de crear una *Pull Request*.

---
---

> 💬 **En resum:**  
> Totes les opcions conserven els canvis, però:
> - *Merge* afegeix un **commit extra** de fusió.  
> - *Rebase* **reordena i reescriu** els commits locals.  
> - *Fast-forward* només **avança** si no hi ha conflicte (no toca res més).

---
## 💡 Exemple pràctic amb error de divergència

Suposem que:
- Tu estàs treballant a la branca `feature-header`.
- El teu company ha fet un *push* a `origin/feature-header` abans que tu.
- Quan proves de fer `git push`, reps el missatge d’ajuda següent:

```
ayuda: Las ramas se han divergido y hay que especificar cómo reconciliarlas.
ayuda: Se puede hacerlo ejecutando uno de los comandos siguiente antes del
ayuda: próximo pull:

ayuda:   git config pull.rebase false  # fusionar
ayuda:   git config pull.rebase true   # rebasar
ayuda:   git config pull.ff only       # solo avance rápido
```

Això indica que la teva branca local i la remota han evolucionat de manera diferent.

Per arreglar-ho, cal actualitzar primer la teva branca amb el remot:

---

### 🟢 Opció 1: Fer una fusió (*merge*) automàtica
Aquesta opció manté l’historial complet amb tots els commits tal com són, però pot generar un commit extra de fusió.

```bash
git pull --no-rebase origin feature-header
```
(o equivalent a `git config pull.rebase false`)

**Resultat:**  
Historial amb un *merge commit*. Més segur, però una mica més "brut".

---

### 🟡 Opció 2: Rebasar (*rebase*) la branca
Aquesta opció torna a aplicar els teus commits damunt dels canvis remots, com si els haguessis fet després.

```bash
git pull --rebase origin feature-header
```
(o equivalent a `git config pull.rebase true`)

**Resultat:**  
Historial més lineal i ordenat, però pot ser més complex si hi ha conflictes.

---

### 🔵 Opció 3: Només permetre "fast-forward"
Aquesta opció només actualitza si la teva branca es pot avançar directament sense conflictes.  
Si hi ha divergència, Git cancel·la el *pull*.

```bash
git pull --ff-only origin feature-header
```
(o equivalent a `git config pull.ff only`)

**Resultat:**  
Evita merges o rebase automàtics. Segur, però només serveix si la teva branca pot avançar directament.

---

## 🧭 En resum

| Opció                         | Ordre de configuració                    | Resultat principal | Avantatge | Risc o inconvenient |
|-------------------------------|------------------------------------------|--------------------|------------|----------------------|
| **Fusionar** (*merge*)        | `git config pull.rebase false`           | Crea commit de fusió | Fàcil i segur | Historial menys net |
| **Rebasar** (*rebase*)        | `git config pull.rebase true`            | Reescriu commits | Historial lineal | Pot generar conflictes complexos |
| **Només avanç ràpid** (*ff*)  | `git config pull.ff only`                | Només avança si no hi ha divergència | Evita errors | No resol divergències |

---

> 📘 **Consell:**  
> En projectes d’equip petits o educatius, és recomanable usar *merge* per mantenir la traçabilitat i evitar complicacions amb *rebase*.  
> El *rebase* és molt útil per mantenir un historial net, però cal comprendre bé què fa abans d’usar-lo.

---

## 📎 Recursos recomanats
- [Rebase vs Merge (GitHub Docs)](https://docs.github.com/en/get-started/using-git/about-git-rebase)
- [Git Book - Rewriting History](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)
