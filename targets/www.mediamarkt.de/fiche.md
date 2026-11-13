# Fiche cible — www.mediamarkt.de

- **Programme** : MediaMarktSaturn — https://yeswehack.com/programs/mediamarktsaturn-bug-bounty-program
- **Dans le périmètre** : oui (vérifié le 2026-11-13) — 11 hôtes `www.*`, tous les sous-domaines exclus
- **Débit autorisé** : <à confirmer dans la policy YWH> — travail mené à débit bas (5 req/s en recon)
- **En-tête d'identification** : User-Agent suffixé `-MMS-BugBounty` (règle Burp Match & Replace, vérifiée le 2026-11-10)

## Technologies

| Couche | Détail |
|---|---|
| CDN / WAF | **Cloudflare** + **Bot Management / Turnstile** (challenge, 403 probabilistes) |
| Anti-fraude | **Forter** (cookie `forterToken`) |
| Observabilité | **Sentry** (télémétrie d'erreurs), **ContentSquare** (session replay) |
| DNS / mail | NS **Akamai** ; messagerie **Microsoft 365** ; SPF `-all`, DMARC `p=reject` (strict) |
| API | **GraphQL** `/api/v1/graphql?operationName=...` — Apollo **Persisted Queries** (hash SHA-256) |
| Auth | **JWT RS256** (cookie `a=`, iss `mmsl`, aud `webmobile`, sub UUID), posé dès la nav anonyme |
| Framework | Plateforme e-commerce propriétaire MMS ; `www.mediamarkt.lu` sur **Shopify** |
| Transport | **HSTS** partout ; **HTTP/3** sur plusieurs hôtes |

## Recon active (httpx, 2026-11-13, 12 hôtes in-scope, 5 req/s)

| Hôte | Code | Technos |
|---|---|---|
| www.mediamarkt.de | 403 (titre « MediaMarkt ») | Cloudflare, Bot Management, HSTS |
| www.saturn.de | 403 (titre « Saturn ») | Cloudflare, Bot Management, HSTS |
| www.mediamarkt.at | 403 | Cloudflare, Bot Management, HSTS, HTTP/3 |
| www.mediamarkt.nl / .be / .hu / .ch | 403 | Cloudflare, Bot Management, HSTS |
| www.mediaworld.it / .com.tr | 403 | Cloudflare, Bot Management, HSTS |
| www.mediamarkt.es | 403 | Cloudflare, HSTS |
| www.mediamarkt.pl | 301 | Cloudflare, Bot Management, HSTS, HTTP/3 |
| www.mediamarkt.lu | 301 | Cloudflare, HSTS, HTTP/3, **Shopify** (CNAME `mm-lux.myshopify.com`) |

## Comptes de test

| Rôle | Identifiant | Notes |
|---|---|---|
| Utilisateur A (attaquant) | <à créer> | alias `@yeswehack.ninja` |
| Utilisateur B (victime) | <à créer> | alias `@yeswehack.ninja` |
| Admin | | non fourni |

## Surface d'attaque

| Fonctionnalité | Intérêt | Testée | Notes |
|---|---|---|---|
| Login email-first | énumération de comptes | ☐ | champ e-mail → « Weiter » (réponse connu/inconnu ?) |
| OAuth Google / Apple | liaison de compte, redirect_uri/state | ☐ | à cartographier |
| MediaMarkt-ID | fédération carte fidélité | ☐ | |
| Commande invité (Gastbestellung) | logique métier sans compte | ☐ | périmètre à part |
| API GraphQL | IDOR/BOLA au niveau resolvers, introspection | ☐ | persisted queries, 200/403 |
| Panier / add-ons payants | logique métier, prix | ☐ | modal upsell (assurances, services) |
| Checkout / paiement | tunnel d'achat | ☐ | **500 cette séance**, à retester connecté |
| Espace compte (commandes/factures) | IDOR | ☐ | format des IDs à relever connecté |

## Identifiants d'objets observés

- **Catégories** : ID numérique court et **séquentiel** (`tv-audio-202`, `fernseher-203`) → piste énumération (à noter, pas à bruteforcer).
- **Produits** : ID à ~7 chiffres = numéro d'article (ex. `3047277`).
- **JWT `sub`** : format **UUID**.
- **GraphQL** : opérations via `operationName` + hash **SHA-256** (persisted queries). Opérations vues : `GetUser` (observée), `GetPaidBundles`/`GetFreeBundles` (robots.txt), `GetProductCollectionItems`/`CategoryV4` (code public tiers).
- **Commandes / factures** : <à relever en session connectée — déterminant pour l'IDOR>

## Pistes à creuser

- [ ] Relever le **checkout connecté** et les moyens de paiement réels (500 à contourner).
- [ ] Lire 1-2 requêtes **GraphQL à 200 JSON** (operationName + variables) ; vérifier GET vs POST et l'introspection.
- [ ] Tester l'**énumération de comptes** sur le login email-first (réponse e-mail connu vs inconnu).
- [ ] Vérifier la **robustesse du JWT** (alg confusion RS256→HS256 ?) et la portée des claims après connexion.
- [ ] **IDOR** sur les identifiants numériques (commandes/factures une fois connecté).
- [ ] Cartographier la surface **OAuth** (redirect_uri, state) et la pile **Shopify** du .lu.

## Écarté (et pourquoi)

- **~48 sous-domaines** (crt.sh) + **163** (subfinder du 13/11) : **hors périmètre** → notés comme paysage du SI, jamais testés.
- **Tiers** (Cloudflare challenge, Sentry, Forter, GA, Meta, TikTok, ContentSquare, `mdc.mediamarkt.de`...) : appelés par la page, **hors scope** → jamais testés.
- **nmap / scan de ports** : inutile derrière Cloudflare (origine masquée) et proche du scanner bruyant interdit → écarté volontairement.
- **Valeurs de jetons/cookies** : volontairement non recopiées (secrets).
