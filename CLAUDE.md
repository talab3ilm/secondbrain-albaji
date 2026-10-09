# CLAUDE.md — Second cerveau des cours arabes

Vault Obsidian maintenu par Claude Code selon le pattern « LLM Wiki » (Karpathy) :
les sources brutes vivent dans `raw/`, Claude compile et entretient `wiki/`.
Les sources sont des cours en arabe (fosha, avec des passages en darija marocaine)
transcrits depuis YouTube, et des livres/PDF convertis en Markdown.

## 1. Structure du projet

Deux dépôts git distincts (compte GitHub `talab3ilm`) :

- **Dépôt de code** `ar_audioToMd/` (le dossier parent) : scripts du pipeline, `audio/`
  (ignoré par git), `docs/nextwork/` (les deux guides d'origine, référence seulement).
- **Dépôt du vault** `ar_audioToMd/secondBrain/` (ce dossier) : ouvert dans Obsidian, ignoré
  par le dépôt de code.

```
secondBrain/
├── CLAUDE.md               # ce fichier : la constitution du vault
├── raw/                    # SOURCES, propriété de l'utilisateur. Claude ne modifie JAMAIS ce dossier
│   ├── youtube/            # un dossier par vidéo : "yyyy-mm-dd-hh-mm <titre> [<id>]/"
│   │   └── .../
│   │       ├── meta.md         # frontmatter : url, id, titre, chaîne, intervenant, date, durée, modèle, validated
│   │       ├── transcript.md   # transcription horodatée (sortie de transcribe.py), avec tashkeel
│   │       ├── texte.md        # texte continu sans horodatage (sortie de md_to_text.py)
│   │       └── texte-ok.md     # copie de texte.md relue et corrigée par l'utilisateur = feu vert pour /ingest
│   └── pdfs/               # un dossier par ouvrage : "<slug>/" avec l'original, sa conversion .md et "<slug>-ok.md"
├── wiki/                   # DOMAINE DE CLAUDE : pages compilées, reliées par [[wiki-links]]
│   ├── index.md            # catalogue maître de toutes les pages, par catégorie
│   ├── log.md              # journal append-only (ingest, lint, requêtes, notes)
│   ├── lessons/            # دروس : une page par cours (résumé structuré + citations horodatées)
│   ├── series/             # مواد : une page par matière (liste ordonnée des leçons, progression)
│   ├── concepts/           # مفاهيم ومصطلحات : une page par notion, terme, règle, question (مسألة)
│   ├── people/             # أعلام : enseignants, auteurs, savants cités
│   └── books/              # كتب : ouvrages étudiés ou cités (متون, شروح)
└── .claude/commands/       # slash commands : /ingest, /query, /lint, /log (+ /pull-sources plus tard)
```

Règles de propriété :

- `raw/` est la mémoire de référence. Claude le **lit** seulement. Aucune correction, aucun
  renommage, aucune suppression, même pour une faute de transcription évidente : la noter
  dans la page wiki concernée à la place.
- **Validation manuelle obligatoire** : seuls les fichiers dont le nom finit par `-ok.md`
  sont ingérés. L'utilisateur relit `texte.md`, le corrige dans une copie `texte-ok.md`
  (ou `<slug>-ok.md` pour un PDF) ; tant que ce fichier n'existe pas, la source est « en
  attente » et `/ingest` l'ignore. `transcript.md` n'est jamais corrigé : il sert aux
  horodatages, et un écart entre lui et `texte-ok.md` signifie que l'utilisateur a corrigé.
- `wiki/` appartient à Claude. L'utilisateur y lit ; s'il édite une page à la main, Claude
  respecte l'édition lors des passages suivants.
- `index.md` liste chaque page du wiki avec une ligne de description. Toute page créée ou
  renommée doit y apparaître dans la même opération.
- `log.md` est append-only : une entrée horodatée par action (`## 2026-10-09 14:30 — ingest`),
  jamais de réécriture de l'historique.
- Les fichiers audio (`.flac`, `.wav`, `.mp3`) ne sont jamais copiés dans le vault ni commités.

## 2. Conventions des pages

Chaque page de `wiki/` commence par un frontmatter YAML :

```yaml
---
title: ""            # titre en arabe, sans tashkeel
type: lesson         # lesson | series | concept | person | book | question | comparison
lang: ar             # langue principale de la page
sources: []          # chemins relatifs dans raw/ (ex. raw/youtube/2026-10-07-19-56 .../transcript.md)
related: []          # [[wiki-links]] vers les pages liées
speaker: ""          # pour type: lesson — [[wiki-link]] vers la page people/
series: ""           # pour type: lesson — [[wiki-link]] vers la page series/
date: ""             # pour type: lesson — date de diffusion (yyyy-mm-dd)
created: ""          # yyyy-mm-dd
last-updated: ""     # yyyy-mm-dd
---
```

- **Une idée par page** (atomique). Une leçon = une page `lessons/`, chaque notion importante
  qui en sort = sa propre page `concepts/`, reliée dans les deux sens.
- **Noms de fichiers** : en arabe sans tashkeel, sans ponctuation, espaces autorisés
  (ex. `wiki/concepts/أحكام الطهارة.md`). Même chaîne pour le `title` et le `[[wiki-link]]`.
- **Tashkeel** (décision de l'utilisateur) : les pages du wiki sont écrites **sans** tashkeel
  pour que la recherche Obsidian fonctionne (« السلام » ne trouve pas « السَّلَامُ »).
  Exceptions : citations du Coran, hadiths, paroles des savants, vers de poésie et متون,
  recopiés avec leur tashkeel depuis la source. La version vocalisée complète reste dans
  `raw/` (`transcript.md`).
- **Structure d'une page `lesson`** : `## الملخص` (5 à 10 lignes), `## المحاور` (plan du cours
  avec horodatage de début de chaque partie), `## المفاهيم` (liens vers concepts/),
  `## الفوائد` (points à retenir), `## إشكالات النسخ` (passages douteux, darija, erreurs
  probables de transcription), `## المصادر`.
- **Structure d'une page `concept`** : `## التعريف`, `## التفصيل`, `## الأدلة والأقوال` (avec
  attribution : quel enseignant, quel livre), `## الخلاف` si les sources divergent,
  `## المصادر`.
- **Citations** : toute affirmation renvoie à sa source. Pour une vidéo : nom du cours +
  horodatage `[mm:ss]` lisible dans `transcript.md`. Pour un livre : titre + page ou
  section. Jamais de paraphrase présentée comme citation.
- **Liens** : `[[Nom de page]]` uniquement vers des pages existantes ou créées dans la même
  opération. Pas de lien vers une page qu'on n'a pas l'intention d'écrire.
- Titres de sections en arabe, `##` et `###` seulement. Pas de `#` de niveau 1 dans le corps
  (le titre vient du frontmatter).

## 3. Guide de style

- Contenu des pages en **arabe fosha**. Chaque page de leçon et de livre se termine par une
  section `## Résumé (français)` de 5 lignes au plus. Les clés de frontmatter, les noms de
  dossiers, les commentaires de scripts et ce fichier restent en latin/français.
- Prose claire, phrases courtes, listes à puces plutôt que paragraphes longs.
- Synthétiser, ne pas recopier la transcription. Une page de leçon fait 300 à 800 mots,
  pas 5 000.
- Attribuer chaque position à son auteur : « قال الشيخ فلان » plutôt qu'un énoncé anonyme.
- Quand deux cours ou deux livres se contredisent, écrire une section `## الخلاف` qui expose
  les deux positions sans trancher.
- Distinguer explicitement : ce qui est dit dans la source (certain), ce que Claude déduit
  (marqué « استنتاج ») et ce qui est douteux dans la transcription (marqué « نص غير مؤكد »).
- Ne pas inventer de référence (numéro de hadith, page, nom d'ouvrage) qui n'est pas dans
  la source. Si l'enseignant cite de mémoire sans référence, le dire tel quel.
- Darija : les passages en dialecte sont résumés en fosha dans la page, avec mention
  « (بالدارجة) » et l'horodatage, sans tentative de transcription littérale.

## 4. Contexte du domaine

- **Source principale** : أكاديمية الإمام الباجي للعلوم الشرعية (مركز إرشاد), programme de
  sciences islamiques en arabe sur 4 ans, enseignants marocains (ذ. نبيل ڭزناي,
  ذ. ياسين العمري, د. البشير عصام المراكشي, directeur général, et d'autres). Cours diffusés en
  direct sur la chaîne YouTube de l'académie ; programme sur <https://albajiacademy.com/program/>.
  Tradition malikite marocaine (ouvrage déjà rencontré : شرح ميارة على المختصر).
- **Volume** : une vingtaine de matières au départ, des milliers de cours à terme. Le wiki doit
  rester navigable à cette échelle : une page `series/` par matière est le point d'entrée,
  `index.md` est organisé par matière, et les pages `concepts/` sont partagées entre matières.
- **Les 20 matières de départ** (nom exact à utiliser pour les pages `series/`) :
  السيرة النبوية · تجويد القرآن · التوحيد والعقيدة · النحو · التزكية والأخلاق · الفقه ·
  آداب طلب العلم · مصطلح الحديث · أصول الفقه · شرح الحديث · قراءة نافع · الصرف · البلاغة ·
  القواعد الفقهية · علوم القرآن · مقاصد الشريعة · تاريخ التشريع الإسلامي · الإلحاد المعاصر ·
  حفظ القرآن · زكاة العلم.
- **Autres sources** : livres en PDF (texte sélectionnable ou scanné) et versions texte de
  bibliothèques en ligne (turath.io, aljam3.com), convertis en Markdown avec leur hiérarchie
  de chapitres et leur tashkeel quand l'original en a.
- **Objectif de l'utilisateur** : retrouver ce qui a été enseigné, par notion et par cours,
  avec la possibilité de remonter à la seconde près dans la vidéo ; relier les cours entre eux
  et aux livres étudiés ; suivre la progression par matière.
- **Qualité des sources** : transcriptions Whisper large-v3 sans ponctuation, tashkeel
  automatique (CATT) parfois faux, passages en darija peu fiables. L'utilisateur relit et
  corrige chaque texte avant ingestion (`-ok.md`), ce qui ne dispense pas de citer les
  horodatages.
- **Modèles** : l'utilisateur dispose d'un abonnement Claude Max et d'un GPU local (RTX 5090)
  avec Ollama. Répartition prévue : brouillons de pages par un modèle local (qwen3.5:27b
  validé le 2026-10-09 : page de leçon correcte en 97 s pour un cours de 74 min), relecture,
  liens croisés et requêtes par Claude. Voir section 5.

## 5. Pipeline d'alimentation (raw/)

Ces outils vivent dans le dépôt de code (`../`), tournent sous WSL avec le GPU, hors de
Claude Code, et écrivent dans `raw/`. Claude ne modifie pas leur sortie.

```bash
../grab_audio.sh <id|url>            # YouTube -> ../audio/<yyyy-mm-dd-hh-mm titre [id]>.flac (date de diffusion)
../transcribe.py "<fichier.flac>"    # faster-whisper large-v3 + tashkeel CATT -> transcript.md
../md_to_text.py "<transcript.md>"   # -> texte.md sans horodatage, paragraphes de ~100 mots
```

- Un cours = un dossier dans `raw/youtube/` nommé comme le FLAC sans extension. L'audio reste
  dans `../audio/`, ignoré par git.
- `meta.md` est créé par le pipeline, pas par Claude. Le champ `validated` passe à `true`
  quand l'utilisateur dépose `texte-ok.md`.
- Les PDF et textes de bibliothèques en ligne arrivent dans `raw/pdfs/<slug>/` avec le
  fichier d'origine et sa conversion `.md` (outil : marker-pdf avec OCR Surya pour les
  scans ; pour turath.io et aljam3.com, récupération du texte directement). Convention
  `-ok.md` identique.
- À venir : un fichier `../sources.txt` listant les URL YouTube (vidéos, playlists) et les
  URL de livres à traiter ; un script lit ce fichier, traite ce qui est nouveau et dépose le
  résultat dans `raw/`. Pas de routine cloud : la transcription exige le GPU local.
- Modèle local (Ollama, `http://localhost:11434`, `qwen3.5:27b`, `num_ctx` ≥ 32768) : utilisable
  pour produire un **brouillon** de page de leçon à partir de `texte-ok.md`. Le brouillon est
  ensuite relu par Claude lors de `/ingest`, qui garde la responsabilité des liens, de
  l'index et du log. Un brouillon ne va jamais directement dans `wiki/` sans relecture.

## 6. Slash commands (comportement attendu)

- `/ingest [n]` : traiter les sources de `raw/` non encore référencées dans une page du wiki
  (vérifier le champ `sources` des pages existantes et `log.md`). Par défaut **1 à 2 cours
  par exécution** : un cours de 90 minutes représente 5 000 à 6 000 mots d'arabe, soit un
  contexte lourd. Pour chaque source : page `lessons/`, pages `concepts/` et `people/`
  créées ou mises à jour, liens croisés, `index.md`, entrée dans `log.md`.
- `/query <question>` : répondre à partir du wiki d'abord, puis de `raw/` si nécessaire, en
  citant chaque page et chaque horodatage utilisé. Signaler les désaccords entre sources.
  Proposer une mise à jour du wiki si la réponse révèle un lien manquant, sans l'écrire
  sans accord.
- `/lint` : liens cassés, pages orphelines (non listées dans `index.md` ou sans lien entrant),
  frontmatter incomplet, sources de `raw/` jamais ingérées, pages de plus de 30 jours jamais
  revues, contradictions entre pages. Rapport seulement, correction après accord.
- `/log <note>` : ajouter une entrée horodatée à `log.md` ; mettre à jour la page concernée si
  la note cite un concept, une personne ou un cours existant. Ne pas créer de page.

## 7. Pièges connus

- Texte arabe : forcer `dir="rtl"` dans tout rendu HTML ; dans Obsidian, activer le plugin
  RTL ou l'option « Right-to-left » par note.
- Les horodatages de `transcript.md` sont en secondes depuis le début de l'audio, pas de la
  vidéo YouTube : identiques en pratique, sauf si l'audio a été coupé.
- Les segments Whisper font 2 à 3 mots sans ponctuation : ne jamais citer un segment isolé
  comme s'il était une phrase complète ; lire le contexte dans `texte.md`.
- Le tashkeel automatique se trompe sur les terminaisons casuelles et les noms propres. Dans
  le doute, citer sans tashkeel.
- Claude Code ne recharge pas `.claude/commands/` à chaud : redémarrer la session après
  avoir ajouté ou modifié une commande.
