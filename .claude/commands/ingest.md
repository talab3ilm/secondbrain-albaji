---
description: Ingérer les sources validées (-ok) de raw/ dans wiki/
argument-hint: [nombre de sources, défaut 1]
---

Tu compiles des cours arabes dans le wiki. Relis d'abord `CLAUDE.md` (conventions, style, domaine).

## Sélection des sources

1. Liste les dossiers de `raw/youtube/` et `raw/pdfs/`.
2. Une source est **ingérable** uniquement si elle contient un fichier Markdown dont le nom se termine par `-ok.md` (`texte-ok.md` pour une vidéo, `<slug>-ok.md` pour un PDF). C'est la marque de validation manuelle de l'utilisateur. Ignore toute source sans `-ok.md`, sans exception.
3. Une source est **déjà ingérée** si son chemin apparaît dans le champ `sources` d'une page de `wiki/lessons/` ou `wiki/books/`, ou dans une entrée `ingest` de `wiki/log.md`. Ignore-la.
4. Traite au plus `$ARGUMENTS` sources (défaut : 1). Un cours de 90 minutes est long : ne charge pas plusieurs cours en même temps.
5. S'il n'y a rien d'ingérable, dis-le, liste les sources en attente de validation, et arrête-toi.

## Pour chaque source

1. Lis `meta.md` (métadonnées), puis le fichier `-ok.md` en entier. Pour les citations horodatées, cherche les passages dans `transcript.md` (segments `**[début -> fin]**`).
2. Écris ou mets à jour la page `wiki/lessons/<titre>.md` (vidéo) ou `wiki/books/<titre>.md` (PDF) selon la structure imposée par `CLAUDE.md`, avec le frontmatter complet et le champ `sources` pointant vers les fichiers de `raw/` utilisés.
3. Pour chaque notion, terme ou مسألة important : crée `wiki/concepts/<nom>.md` ou enrichis la page existante (ajoute l'attribution et la source ; si la nouvelle source contredit la page, ajoute une section `## الخلاف`). Ne crée pas de page pour une notion seulement mentionnée en passant.
4. Enseignant et savants cités : page dans `wiki/people/`. Ouvrages cités : page dans `wiki/books/` (une courte fiche suffit si l'ouvrage n'est pas la source elle-même).
5. Matière : rattache la leçon à sa page `wiki/series/<matière>.md` (crée-la si besoin à partir de la liste des matières de `CLAUDE.md`) et ajoute la leçon à la liste ordonnée des leçons de cette matière.
6. Relie toutes ces pages entre elles avec des `[[wiki-links]]` dans les deux sens. Aucun lien vers une page qui n'existe pas.
7. Mets à jour `wiki/index.md` (toute page nouvelle ou renommée, avec une ligne de description).
8. Ajoute une entrée à `wiki/log.md` : `## <yyyy-mm-dd hh:mm> — ingest` suivie de la source traitée et des pages créées ou modifiées.

## Interdits

- Ne modifie, ne renomme et ne supprime jamais rien dans `raw/`.
- N'invente aucune référence (hadith, page, auteur) absente de la source.
- N'écris pas de tashkeel dans le wiki, sauf dans les citations (Coran, hadith, paroles des savants, vers).

Termine par un résumé : sources traitées, pages créées, pages modifiées, points douteux signalés.
