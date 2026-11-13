# Cartographie de surface — www.mediamarkt.de (observé)

> Source : navigation manuelle via proxy Burp, HTTP history (relevé 2026-11-11).
> 100 % observé à l'écran — aucun scan, aucune énumération. Ce qui n'a pas été vu est marqué « non observé ».
> Valeurs de jetons et de cookies volontairement non recopiées (secrets).

## 1. Hôte in-scope observé
- `www.mediamarkt.de` → IP 172.65.208.234 (bloc Cloudflare). TLS actif (HTTPS).

## 2. Endpoints observés
| Méthode | Chemin | Code | Type | Remarque |
|---|---|---|---|---|
| GET | `/` | 403 puis 200 | HTML | 403 tant que le challenge Cloudflare n'est pas passé |
| GET | `/cdn-cgi/challenge-platform/...` | 200 | script | Orchestration anti-bot Cloudflare |
| POST | `/cdn-cgi/challenge-platform/h/b/fo...` | 200 | text | Soumission du challenge → délivre `cf_clearance` |
| GET | `/public/setCookie/ts_id/<uuid>` | 200 | text | Pose un cookie `ts_id` (uuid) |
| GET | `/setCookie/t_fpd/true` | 200 | JSON | Pose un cookie de fingerprint `t_fpd` |
| GET | `/de/category/tv-audio-202.html` | 200 | HTML | Page catégorie « TV & Audio » |
| GET | `/de/category/fernseher-203.html` | 200 | HTML | Sous-catégorie « Fernseher » |
| GET | `/de/product/_<slug>-3047277.html` | 200 | HTML | Fiche produit — ID 3047277 = Art.-Nr. |
| GET | `/de/translations/<feature>.json` | 200 | JSON | Fichiers i18n publics (révèlent des fonctionnalités) |

### Motif d'URL (observé)
- Catégorie : `/de/category/<slug>-<id>.html` — IDs courts et **séquentiels** (202, 203) → semblent énumérables (à noter, pas à bruteforcer).
- Produit : `/de/product/_<slug>-<id>.html` — ID à ~7 chiffres (= Art.-Nr., ex. 3047277).
- Préfixe de langue `/de/`.

## 3. API GraphQL (pièce clé)
- `GET /api/v1/graphql?operationName=<Nom>` sur www.mediamarkt.de — nombreux appels.
- **Apollo Persisted Queries** : operationName + `extensions.persistedQuery.sha256Hash`. L'API n'accepte sans doute que des requêtes enregistrées ; introspection/arbitraire probablement restreints (à vérifier en test).
- Statuts **mixtes** : certaines requêtes → 403 (bloc Cloudflare, `Server: cloudflare`, `Cf-Ray`) ; une autre → **200 JSON (6350 o)**. L'API est atteignable, le blocage Cloudflare est **probabiliste par requête**, pas total.
- Opérations identifiées : `GetUser` (observée), `GetPaidBundles`/`GetFreeBundles` et `MultiChannelMARepairStatusResult` (robots.txt), `CategoryV4`/`GetProductCollectionItems` (code public de tiers).
- Autres API déclarées (robots.txt, non sollicitées) : `/api/v1/msg`, `/api/v1/mms-sst`, `/api/v1/partners/`, `/rde_server/`, `*/myaccount/auth*`.

## 4. Authentification (observé)
- **JWT RS256** dans le cookie `a=` : `kid:o2sign`, `iss:mmsl`, `aud:webmobile`, `sub:<UUID>`, `iat`/`exp`. Signé serveur. Présent **dès la navigation anonyme** (jeton invité) → à re-relever **après connexion** (rôle, claims).
- Cookies d'identité/anti-fraude : `optid` (UUID), `forterToken`, `ts_id`, `pwaconsent`, `msbps` (~34 cookies au total).
- Points d'entrée : login **email-first** (champ e-mail → « Weiter », pas de mot de passe écran 1 → piste **énumération de comptes**), OAuth **Google/Apple**, **MediaMarkt-ID** (carte fidélité), **commande invité**.
- Piste test : alg confusion JWT (RS256→HS256 ?), portée des claims.

## 5. Parcours d'achat (flux d'argent)
- Ajout au panier → modal d'upsell : assurances PlusGarantie (+79,99 € / +2,99 €/mois / +129,99 €), services d'installation (+99,99 € / +29,99 €).
- Financement affiché (eff. 10,90 %). Anomalie UI : clé de traduction brute `basket.summary.vat` visible à l'écran (fuite de clé i18n interne — mineur).
- `GET /de/checkout` → **Error 500** (accès direct ET parcours normal). Tunnel inaccessible cette séance → à retester **connecté**. Cause probable : checkout nécessitant une connexion et/ou appels API bloqués par Cloudflare.
- Moyens de paiement réels : non atteints (bloqués par le 500). Rappel OSINT à confirmer : PayPal, facture (Santander/Klarna), financement, CB, cartes cadeaux, bons.

## 6. Défenses confirmées (observé)
- **Cloudflare** frontal : CDN + Bot Management / Turnstile (`/cdn-cgi/challenge-platform/`, cookies `__cf_bm`, `cf_clearance`). 403 tant que le challenge n'est pas résolu + blocage probabiliste de certains appels API.
- **Forter** : anti-fraude (`forterToken`).
- En-têtes sécurité (réponse API) : HSTS `max-age=31536000`, `X-Frame-Options: SAMEORIGIN`, `Referrer-Policy: same-origin`, `Cache-Control: no-store`. Cookie `__cf_bm` en `Secure; HttpOnly; SameSite=None; Partitioned` (CHIPS).
- Conséquence méthodo : **automatisation bloquée → navigation manuelle, débit bas**.

## 7. Tiers hors scope (notés, jamais testés)
- `challenges.cloudflare.com` (widget Turnstile), `*.sentry.io` (télémétrie Sentry), `*.contentsquare.net` (session replay), traceurs pub (GA, Bing, DoubleClick, Snapchat, TikTok, Meta, Pinterest, Trade Desk, OpenAI « KI-Suche »).
- `mdc.mediamarkt.de` = **sous-domaine → hors scope**. Noté, jamais testé.

## 8. Synthèse — surface retenue (base pour la sélection)
1. **API GraphQL** `/api/v1/graphql` (persisted queries) — surface technique principale.
2. **Authentification** : JWT RS256 + OAuth + MediaMarkt-ID + invité → contrôle d'accès.
3. **Logique métier / argent** : panier, add-ons payants, checkout, financement, cartes cadeaux, fidélité.
4. **IDOR potentiels** : IDs produits/catégories numériques ; IDs commandes/factures à relever connecté.
5. **Contraintes** : Cloudflare/Turnstile (probabiliste), Forter → tout manuel.
