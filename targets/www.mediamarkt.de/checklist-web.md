# Checklist web — www.mediamarkt.de

> Dupliquée de docs/02-checklist-web.md. Cochée au fur et à mesure.
> Légende : [x] testé · [~] partiel / candidat non confirmé · [ ] non testé.
> Cas B (aucune faille prouvée). Classes retenues : contrôle d'accès (IDOR/BOLA), auth (JWT/OAuth), logique métier paiement.

## Reconnaissance applicative
- [x] Technologies : Cloudflare (CDN/WAF/Bot Management + Turnstile), Forter (anti-fraude), Sentry, API GraphQL à requêtes persistées, client Apollo (pwa-client-pqm), DNS Akamai, mail Microsoft 365
- [ ] Fichiers JS téléchargés et lus — à faire (recon active) ; opérations GraphQL partiellement extraites par navigation
- [x] robots.txt / sitemap / .well-known / graphql — robots.txt lu (endpoints API révélés), sitemaps sous /sitemaps/, /api/v1/graphql confirmé ; swagger/openapi non trouvés
- [x] Sous-domaines / environnements annexes — ~48 à 164 recensés (CT) ; TOUS hors scope → documentés, non testés
- [ ] Répertoires exposés (.git/.env/backup/dumps) — non testé
- [~] Messages d'erreur verbeux — Sentry actif ; erreurs GraphQL structurées (error_code) ; pas de stack trace exposée observée

## Authentification
- [ ] Énumération d'utilisateurs — non testé (login email-first repéré ; énumération = non qualifiante)
- [ ] Politique MDP / verrouillage / rate limit login — non testé
- [ ] Reset de mot de passe — non testé
- [ ] MFA — non testé
- [~] OAuth / SSO — options repérées (Google, Apple, MediaMarkt-ID) ; observation du flux authorize = à faire (T3)
- [~] JWT — structure analysée (RS256, kid o2sign, iss mmse, claims sub/user_id/r=partyId, exp 15 min) ; alg:none / confusion HS-RS = à faire (T1)

## Session
- [x] Attributs cookies — cookie session `a` : HttpOnly + Secure + SameSite=Strict
- [ ] Session non invalidée à la déconnexion / au changement de MDP — non testé
- [ ] Session fixation — non testé
- [ ] Sessions concurrentes non révoquées — non testé

## Contrôle d'accès — CLASSE RETENUE
- [~] IDOR — 2 candidats : partyId (GetPointsBalance/Transactions, séquentiel, présent dans le JWT) et basketId (AddGiftcard + mutation coupon, UUID). NON confirmés → besoin du compte B
- [ ] Élévation verticale (endpoint admin avec compte user) — non testé
- [ ] Élévation horizontale entre tenants / pays — non testé
- [ ] Contrôle uniquement côté client — non testé
- [ ] Changement de méthode HTTP (GET vers POST/PUT/PATCH/DELETE) — non testé
- [ ] Contournement de chemin — non testé

## Injections — non retenu (hors classes sélectionnées)
- [ ] SQL / NoSQL — non testé
- [ ] Commandes OS — non testé
- [ ] SSTI — non testé
- [ ] LDAP / XPath / CRLF — non testé
- [ ] Désérialisation — non testé

## XSS — non retenu
- [ ] Réfléchie / stockée / DOM — non testé
- [ ] CSP — non analysée en détail
- [ ] Upload rendu HTML/SVG — non testé

## Logique métier — CLASSE RETENUE
- [x] Manipulation prix/quantité/devise/remise — testé (cartographie) : montants calculés cote serveur (GetBasket, AddGiftcard, coupon n'envoient aucun prix) ; manip de prix sans objet, non qualifiante
- [ ] Étapes du tunnel sautées / rejouées — non testé
- [ ] Valeurs négatives / grandes / décimales / unicode — non testé (pas de champ montant côté client)
- [ ] Race conditions (double usage coupon / carte cadeau) — non testé
- [~] Absence de rate limit sur action coûteuse — ajout carte cadeau protégé par Turnstile, ajout code promo sans Turnstile (asymétrie) ; non testé (énumération interdite), noté

## Côté serveur — non retenu
- [ ] SSRF / LFI / XXE / upload / request smuggling — non testés

## Configuration
- [~] CORS permissif — GetBasket renvoie access-control-allow-origin: * sur endpoint authentifié ; wildcard + cookies bloqué en cross-origin crédentialé, a priori non exploitable (à confirmer)
- [x] En-têtes de sécurité — HSTS (max-age 31536000), x-frame-options: deny ; CSP non détaillée
- [ ] Subdomain takeover — non testé (sous-domaines hors scope)
- [ ] Identifiants par défaut — N/A
- [ ] CVE confirmées manuellement — non testé

## Avant de rapporter
- [ ] Reproduit 2x depuis une session propre — N/A (aucun finding confirmé)
- [~] Impact métier en une phrase — rédigé pour les candidats dans le rapport projet
- [x] Preuves caviardées + horodatées — captures internes ; tokens/PII jamais recopiés
- [x] Vérifié hors liste exclue — manip de prix, énumération, rate-limiting, open redirect simple = écartés
