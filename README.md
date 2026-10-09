# secondbrain-albaji

Second cerveau des cours de **أكاديمية الإمام الباجي للعلوم الشرعية** : un vault Obsidian
où les sources brutes (transcriptions de cours YouTube, livres convertis en Markdown) sont
compilées par Claude Code en un wiki arabe relié par `[[wiki-links]]`, selon le pattern
« LLM Wiki ». Les règles complètes sont dans [CLAUDE.md](CLAUDE.md).

## Principe

```
raw/   (vos sources, jamais modifiées)  ──/ingest──►  wiki/  (pages compilées par Claude)
```

- `raw/youtube/<cours>/` : `meta.md`, `transcript.md` (horodaté, vocalisé), `texte.md`
  (continu) et, une fois relu et corrigé par vous, `texte-ok.md`.
- `raw/pdfs/<ouvrage>/` : original, conversion Markdown et `<slug>-ok.md` validé.
- `wiki/` : `index.md`, `log.md`, puis `lessons/` (دروس), `series/` (مواد), `concepts/`
  (مفاهيم ومصطلحات), `people/` (أعلام), `books/` (كتب).

**Seuls les fichiers `-ok.md` sont ingérés.** Tout le reste est en attente de validation.

## Commandes Claude Code

Lancer `claude` depuis ce dossier.

| Commande | Effet |
|---|---|
| `/ingest [n]` | Compile `n` sources validées (défaut 1) en pages wiki, met à jour `index.md` et `log.md`. |
| `/query <question>` | Répond à partir du wiki avec citations de pages et horodatages. |
| `/lint` | Rapport de santé : liens cassés, pages orphelines, frontmatter, contradictions. |
| `/log <note>` | Ajoute une note horodatée au journal. |

## Conventions

- Pages en arabe fosha, sans tashkeel sauf citations (Coran, hadith, savants, vers), avec un
  court résumé en français. Métadonnées et noms de dossiers en latin.
- Chaque affirmation cite sa source : page wiki, et horodatage `[mm:ss]` pour un cours.
- 20 matières de départ, une page `series/` par matière ; des milliers de cours à terme.

## Obsidian

Ouvrir ce dossier comme vault. Activer l'affichage de droite à gauche pour l'arabe
(plugin RTL ou option par note). Le dossier `.claude/` est caché par Obsidian, c'est normal.

## Pipeline amont

Les fichiers de `raw/` sont produits par le dépôt
[ar-audio-to-md](https://github.com/talab3ilm/ar-audio-to-md) (téléchargement, transcription
Whisper large-v3, tashkeel CATT). Aucun fichier audio n'est stocké ici.
