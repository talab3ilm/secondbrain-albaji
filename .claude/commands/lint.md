---
description: Contrôle de santé du wiki (rapport seulement)
---

Analyse `wiki/` et produis un rapport structuré. Ne corrige rien avant accord explicite.

Vérifie :

1. **Liens cassés** : `[[Nom]]` sans fichier correspondant dans `wiki/`.
2. **Pages orphelines** : absentes de `wiki/index.md`, ou sans aucun lien entrant.
3. **Frontmatter** : champs manquants ou vides par rapport à `CLAUDE.md` (title, type, lang, sources, related, created, last-updated ; speaker, series, date pour une leçon). `title` ou nom de fichier contenant du tashkeel.
4. **Sources** : chemins du champ `sources` qui n'existent plus dans `raw/` ; sources `-ok.md` de `raw/` jamais ingérées.
5. **Tashkeel hors citation** dans le corps des pages.
6. **Pages obsolètes** : `last-updated` de plus de 30 jours alors qu'une source plus récente les concerne.
7. **Contradictions** entre pages sur un même concept, sans section `## الخلاف`.
8. **Matières** : leçons sans page `series/`, pages `series/` dont la liste des leçons est incomplète.

Présente le rapport par catégorie avec les chemins exacts, puis propose un plan de correction et attends la validation. Ajoute une entrée `## <yyyy-mm-dd hh:mm> — lint` à `wiki/log.md` avec le nombre de problèmes par catégorie.
