# Passation — Grenaille (ex-« Chasse »)

Document de reprise pour une nouvelle conversation Claude Code. Réécrit le 2026-07-26.
Voir aussi la mémoire persistante (`MEMORY.md` chargée à chaque session).

## Le site
- **Nom : Grenaille.** Slogan/sous-titre : « Quand est-ce qu'on chasse ? »
- **En ligne : https://grenaille.github.io** — Astro statique, déployé par GitHub Actions à chaque push sur `main`.
- **Dépôt : `Grenaille/Grenaille.github.io`** (migré dans une organisation GitHub le 2026-07-25 ; l'ancien `huguesdeboisse-pixel.github.io` est mort). Le remote git local pointe déjà dessus.
- **« commit » = commit ET push** (l'utilisateur veut le déploiement dans la foulée). Direct sur `main`, pas de branche/PR.
- Dossier local : `/home/alexandrine/Documents/W SITE CHASSEUR`. `npx astro check` doit rester à 0 erreur.

## Objectif & règle n°1
Recenser, département par département (101), la réglementation de la chasse. **Aucune donnée publiée sans source officielle vérifiable.** En cas de doute → le dire, jamais inventer. Un sous-agent a déjà fabriqué un statut « signé » par le passé — d'où les garde-fous.

## Logo
Lockup unique **fauconnier + GRENAILLE + sous-titre**, en deux SVG :
- `public/logo-grenaille-clair.svg` (hero, fond vert sombre — texte blanc, fauconnier sage).
- `public/logo-grenaille-sombre.svg` (header, fond crème — texte encre, fauconnier sombre).
Wordmark GRENAILLE vectorisé ; sous-titre = élément `<text>` avec `textLength` = largeur de GRENAILLE (donc même longueur exacte), police web-safe. **L'utilisateur compte refaire le logo proprement « à terme »** (idéalement il fournira un nouveau `logo_complet.svg` ; source actuelle dans `Silhouettes animales/logo_complet.svg`). Un `<h1 class="sr-only">Grenaille</h1>` conserve le titre pour le SEO.

## Espèces & sélecteur
- **89 espèces** dans `src/data/especes.json` (= liste légale complète), catégorisées ; slugs dans `src/lib/especes.ts` (alignés sur les libs). Catégories : grand gibier · petit gibier sédentaire · gibier de montagne · oiseaux de passage · gibier d'eau · ESOD · gibier d'outre-mer.
- **Sélecteur** (fiche département, dans `[code].astro` + CSS `.sel-*` dans `Base.astro`) : 5 silhouettes emblématiques (accès rapide, vertes → aplat vert sombre à la sélection, sans encart) + recherche permissive (fautes/accents) avec **menu déroulant groupé par catégorie** et sélection multiple (chips). Responsive.

## Données réglementaires (dates par espèce)
- **Vagues 1 / 2a / 2b = NATIONAL, collecté une fois** : oiseaux de passage (`src/lib/migrateurs.ts`) et gibier d'eau (`src/lib/gibier-eau.ts`). Dérogations Sud (grives/ramier), distinction côtier/intérieur, cohabitation avec les faits collectés, gestion des moratoires. **Fait.**
- **Vague 3 (gibier localisé) collectée** : daim, cerf sika, marmotte, lièvre variable, gélinotte, + montagne (chamois/isard, mouflon, tétras, lagopède, bartavelle) ajoutés aux arrêtés départementaux (~25 départements montagnards). La Savoie est réparée.
- **Chassabilité des espèces localisées = arrêté-présence** : elles ne s'affichent que là où l'arrêté du département les réglemente (`ESPECES_LOCALISEES` dans `[code].astro`). Auto-sourcé, pas de liste géographique à valider à la main. Voir mémoire `project_site_chasse_chassabilite`.

## ESOD (vague 4) — BLOQUÉ par un gap réglementaire
Les 16 espèces ESOD sont dans le modèle (catégorie ESOD) mais affichent un **statut « classement groupe 2 en cours de renouvellement 2026-2029 »** : l'arrêté ministériel 2023-2026 a expiré le 01/07/2026 et le 2026-2029 **n'est pas encore publié**. Donc **rien à collecter tant qu'il n'est pas paru** (règle n°1). À reprendre dès parution : c'est un seul document national avec les listes par département.

## Collecte — état
- **~89 départements** ont des données. **Manquent (arrêté 2026-2027 pas encore publié) : 14, 22, 50, 62, 65, 2A, 2B** — à retenter plus tard, pas un oubli. (Ne pas les ressasser : l'utilisateur le sait.)
- **DROM** (Guadeloupe, Martinique, Guyane, Mayotte) : régimes différents, plus tard. Réunion (974) a ses 3 espèces propres.
- Structurels sans fichier : Paris 75, petite couronne 92/93/94.

## Vérification des données
- `scripts/audit_donnees.py` : niveaux 1-2 (entrées suspectes + manques de complétude par filet de plausibilité). `--liens` teste les URL.
- `scripts/verifier_sources.py` : niveau 3 (télécharge le PDF de l'arrêté et vérifie que les dates y figurent). Mode `--hasard N` pour une pêche aléatoire.
- ~85% des arrêtés sont des **PDF scannés** (pas d'OCR local) → relecture par sous-agents à lecture visuelle (Read gère les PDF scannés). Vérif d'un échantillon de 24 départements = 24/24 fidèles → on ne re-vérifie que les nouveaux + pêche aléatoire.

## Méthode sous-agents (collecte/vérif)
Lancer des `general-purpose` en arrière-plan (`run_in_background`), par lots régionaux de ~5-6 départements. Règles strictes : aucune invention, un projet/consultation n'est PAS un arrêté signé, ne jamais supprimer de donnée existante (enrichissement additif). Toujours re-vérifier le diff (ajouts seulement) + `astro check` avant de committer. **La session peut couper au quota horaire** — c'est normal, on reprend au reset.

## Pistes ouvertes
1. **ESOD groupe 2** : collecter dès parution de l'arrêté 2026-2029.
2. **7 départements** manquants : retenter quand leur arrêté sort.
3. **DROM** : collecte dédiée (régimes propres).
4. **Espèces sous moratoire** (grand tétras, tourterelle des bois, courlis cendré, barge à queue noire) : encodées mais affichage « suspendu » à finaliser.
5. **Mentions légales** : obligatoires (site monétisé), reporté par l'utilisateur — nécessite nom/adresse réels.
6. **Front-end mobile** : le sélecteur a un repli mobile de base, à peaufiner éventuellement.
7. **Logo** : l'utilisateur veut le refaire proprement.

## Conventions / profil utilisateur
- Débutant git/Claude Code — expliciter en clair. Réponses directes pour les petits changements, rigueur maximale dès qu'une donnée factuelle est en jeu.
- **Toujours REGARDER le rendu** (capture/inspection) avant d'affirmer qu'un point visuel est bon — l'utilisateur a repéré deux fois des affirmations basées sur du code/tokens plutôt que sur l'image réelle.
- Le préview local a des ratés (CSSOM, screenshots qui timeout sur la page carte) — vérifier via DOM/inspection ou sur le vrai site déployé.
