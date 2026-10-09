# raw/ — sources brutes

Propriété de l'utilisateur. Claude lit, ne modifie jamais.

- `youtube/<yyyy-mm-dd-hh-mm titre [id]>/` : `transcript.md` (horodaté, avec tashkeel), `texte.md` (continu), `meta.md` (métadonnées), et `texte-ok.md` une fois le texte relu et corrigé par l'utilisateur.
- `pdfs/<slug>/` : le PDF ou le texte d'origine, sa conversion `.md`, et `<slug>-ok.md` une fois validée.

Seuls les fichiers `-ok.md` sont ingérés par `/ingest`.
