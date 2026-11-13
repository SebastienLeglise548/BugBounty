# Suivi des rapports —  HexCode_Lich

Projet tutoré M2 — Bug Bounty MediaMarktSaturn (YesWeHack, programme public).

| # | Date | Cible | Vulnérabilité | Sévérité | Plateforme | Statut | Prime | Fichier |
|---|---|---|---|---|---|---|---|---|
| — | — | www.mediamarkt.de | aucune soumise à ce jour | — | YesWeHack | — | — |

**Statuts** : brouillon → soumis → triage → accepté / dupliqué / informatif / rejeté → corrigé

## Bilan
- Rapports soumis : 0 à ce jour.
- Observations mineures notées (non qualifiantes pour un rapport) : `/de/checkout` → 500 ; clé i18n brute `basket.summary.vat` affichée. Documentées dans `targets/www.mediamarkt.de/notes/cartographie-surface.md`.
- Leçon principale : sur une surface e-commerce durcie (Cloudflare Bot Management + Forter, périmètre réduit aux pages d'accueil), la valeur du travail est la **méthodologie documentée** et la cartographie, pas le nombre de findings.
