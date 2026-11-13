# Sélection des classes de vulnérabilités — www.mediamarkt.de

> Base : `notes/cartographie-surface.md` + grille non-qualifiante de la policy (`scope/regles.md`).
> Principe : on ne retient qu'une classe (a) cohérente avec l'architecture observée,
> (b) RÉCOMPENSÉE (hors liste non-qualifying), (c) testable manuellement (Cloudflare bloque l'auto).

## Contexte qui oriente le choix
- API **GraphQL** à requêtes persistées (hash SHA-256) portant la logique produit/compte.
- **JWT RS256** (cookie `a=`) dès la nav anonyme ; login email-first + OAuth Google/Apple + MediaMarkt-ID + invité.
- **IDs d'objets numériques** : catégories séquentielles (202, 203), produits = Art.-Nr (7 chiffres) ; commandes/factures à relever connecté.
- **Flux d'argent** : panier + add-ons payants, financement, cartes cadeaux, points fidélité, marketplace.
- Défenses : Cloudflare Bot Management/Turnstile + Forter (anti-fraude) → tout manuel, débit bas.

## Classes RETENUES

### 1. Contrôle d'accès / IDOR-BOLA — CWE-639 / CWE-284  [priorité 1]
- **Pourquoi** : IDs numériques exposés + API GraphQL portant la logique compte/commande. Classe récompensée.
- **Où** : espace compte connecté (commandes, factures, adresses), resolvers GraphQL avec 2 comptes A/B.
- **Impact** : accès aux données d'un autre client → High/Critical (2 500–4 000 €).

### 2. Authentification — JWT & OAuth — CWE-287 / CWE-347  [priorité 2]
- **Pourquoi** : JWT RS256 maison + SSO multiples. Un contournement d'auth est Critical.
- **Où** : robustesse JWT (alg confusion RS256→HS256, `kid`, claims/rôle post-login) ; flux OAuth (redirect_uri, state, liaison) ; MediaMarkt-ID.
- **Impact** : prise de compte → Critical.
- **NB** : l'énumération de comptes (email-first) est **non-qualifiante** → observée, jamais soumise.

### 3. Logique métier / paiement — CWE-840  [priorité 3, si le checkout se débloque]
- **Pourquoi** : add-ons payants, financement, cartes cadeaux, points fidélité = flux d'argent.
- **Où** : intégrité prix/remises **validée côté serveur**, abus de coupons/points, réutilisation de cartes cadeaux.
- **Impact** : Medium/High selon l'abus.
- **Blocage** : `/de/checkout` → 500 ; à relever connecté d'abord. (Manip de prix rejetée côté backend = non-qualifiante.)

## Classes ÉCARTÉES
- **Non récompensées** : énumération d'utilisateurs, introspection GraphQL, absence de rate limiting/brute force, SPF/DMARC, info disclosure (500, stack traces, clé i18n), manip de prix rejetée par le backend, self-XSS, SSL/TLS, clickjacking, en-têtes manquants sans exploitation.
- **Hors scope** : sous-domaines, `*.mediamarkt.pt`, `www.mediamarkt.lu` (zone grise), infra PlusServer.

## Plan de test (ordre)
1. Créer comptes A/B (`@yeswehack.ninja`), se connecter via navigateur Burp (UA `-MMS-BugBounty`).
2. Espace compte connecté : format des IDs commandes/factures → **IDOR** (classe 1).
3. Rejouer les requêtes GraphQL à 200 entre A et B → **BOLA** resolvers (classe 1).
4. JWT post-login (claims, rôle) + OAuth → **auth** (classe 2).
5. Si checkout accessible connecté → **logique paiement** (classe 3).
