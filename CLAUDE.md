# Partogramme — suivi du travail en salle de naissance

Site statique (HTML/CSS/JS sans build). Données de TP fictives. Ce fichier dit ce qu'une IA ne peut pas deviner.

## À savoir (leçons des sessions passées)

- **config.js ne contient QUE l'URL et la clé publishable.** Le 30/09/2026, ce fichier a été
  trouvé avec plusieurs clés collées sur une même ligne (anon + service_role + sb_secret) :
  le site était en erreur 401 et une clé secrète a fuité dans le code. Corrigé, mais la clé
  secrète doit être régénérée (rotate) dans Supabase → Settings → API keys.
- **Pas de Python sur ce PC** et `npx` est bloqué par la politique d'exécution PowerShell.
  Pour tester en local : `node serve.js` depuis `Projets/` (mini-serveur déjà écrit), puis
  ouvrir http://127.0.0.1:8777/
- **Le dépôt git local n'existe plus** (PC réinitialisé). Le site en ligne est en 404 tant
  qu'on n'a pas refait `git init` + `git remote add origin` + `git fetch` + `git reset --hard
  origin/main` puis `git push` (procédure dans le CLAUDE.md parent).

## Base de données (Supabase, projet `xszmtpdudvxcoqfoasxu`)

6 tables, toutes en RLS ouvert aux rôles `anon`/`authenticated` (lecture, ajout, modification).
**Seul `examen` a une règle DELETE** ; les autres tables n'ont pas de suppression (voulu : on
ne supprime pas une patiente ou une grossesse).

- `patiente` : nom, nom_marital, prenom, gs_rh (enum groupe_sanguin : A+…AB-, "inconnu"), date_naissance
- `praticien` : nom, prenom, specialite (enum : Sage-femme, Anesthésiste, Obstétricien, Pédiatre)
- `grossesse` : id_patiente → patiente ; debut, parite, hiv, toxo (booléens), duree_travail et
  duree_expulsion (type time, saisis en h:min), 4 clés vers praticien (sage-femme,
  obstétricien, anesthésiste, pédiatre) = l'« équipe référente »
- `examen` : id_grossesse → grossesse ; date_heure (timestamptz), dilatation (0–10 cm),
  presentation_hauteur (-5 à +5), id_orientation → orientation, remarque
- `orientation` : table de référence (9 lignes, OIGA…OS, Siège) avec nom_orientation et
  fichier_pictogramme ; lecture seule
- `enfant` : id_grossesse → grossesse ; prenom, sexe (enum Garçon/Fille),
  date_heure_naissance (timestamptz), poids_naissance_kg (float), taille_cm

## Code et bibliothèques

- CDN : `@supabase/supabase-js@2`, `chart.js@4` (courbe de dilatation sur canvas), `exceljs@4`
  (export Excel du partogramme), polices `@fontsource/inter`.
- Un fichier JS par page : `app.js` (navigation par onglets + page Patientes + page Suivi),
  `grossesses.js`, `examens.js` (formulaire + courbe + KPI), `enfants.js`, `praticiens.js`,
  `export.js`. `index.html` charge tout dans cet ordre.
- Convention : tables et colonnes en français sans pluriel (`patiente`, `id_grossesse`),
  identifiants HTML en français (`liste-examens`, `message-formulaire`).
- La maquette d'origine est `maquette.png` (comparer si refonte visuelle demandée).
