# Checklist web — www.mediamarkt.de

> Dupliquée de docs/02-checklist-web.md. Cochée au fur et à mesure.
> Légende : [x] testé · [~] partiel / candidat non confirmé · [ ] non testé.
> Cas B (aucune faille prouvée). Classes retenues : contrôle d'accès (IDOR/BOLA), auth (JWT/OAuth), logique métier paiement.

## Reconnaissance applicative
- [x] Technologies : Cloudflare (CDN/WAF/Bot Management + Turnstile), Forter, Sentry, API GraphQL à requêtes persistées, client Apollo (pwa-client-pqm), DNS Akamai, mail Microsoft 365
- [x] Fichiers JS lus : main.js + persisted-query-lists.js (sur www.mediamarkt.de) → AUCUN secret/clé/token en dur. Manifeste des opérations GraphQL chargé dynamiquement (loadManifest), non embarqué
- [x] robots.txt / sitemap / .well-known / graphql — robots.txt lu, sitemaps sous /sitemaps/, /api/v1/graphql confirmé ; swagger/openapi non trouvés
- [x] Sous-domaines / environnements annexes — ~48 à 164 recensés (CT) ; TOUS hors scope, documentés, non testés
- [x] Répertoires exposés — négatif : /.git/HEAD, /.env, /.DS_Store, /server-status, /backup → tous 404/erreur, rien d'exposé
- [~] Messages d'erreur verbeux — Sentry actif ; erreurs GraphQL structurées ; pas de stack trace exposée

## Authentification
- [ ] Énumération d'utilisateurs — non testé (non qualifiante)
- [ ] Politique MDP / verrouillage / rate limit login — non testé
- [ ] Reset de mot de passe — non testé
- [ ] MFA — non testé
- [x] OAuth / SSO — observé : flux Google = response_type=code, state + nonce présents, scope minimal, PKCE absent (client confidentiel). redirect_uri = auth.mediamarkt.de (SOUS-DOMAINE HORS SCOPE) → manip redirect_uri non testable. Posture correcte
- [x] JWT — 3 attaques négatives : (1) signature vérifiée (payload trafiqué → Unauthorized) ; (2) alg:none rejeté ; (3) confusion RS256→HS256 rejetée (clé reconstruite, token HS256 forgé → Unauthorized + redirection login). Validé de bout en bout

## Session
- [x] Attributs cookies — cookie session : HttpOnly + Secure + SameSite=Strict
- [ ] Session non invalidée à la déconnexion / changement MDP — non testé
- [ ] Session fixation — non testé
- [ ] Sessions concurrentes non révoquées — non testé

## Contrôle d'accès — CLASSE RETENUE
- [~] IDOR — 2 candidats : partyId (séquentiel, dans le JWT) et basketId (UUID). NON confirmés → besoin du compte B
- [ ] Élévation verticale — non testé
- [ ] Élévation horizontale entre pays — non testé
- [x] Contrôle uniquement côté client — le backend applique l'autorisation (Unauthorized sans session). Pas de contrôle client-only
- [ ] Changement de méthode HTTP — non testé
- [ ] Contournement de chemin — non testé

## Injections / XSS — non retenu (hors classes sélectionnées)
- [ ] SQL/NoSQL, OS, SSTI, LDAP/XPath/CRLF, désérialisation — non testés
- [ ] XSS réfléchie/stockée/DOM, CSP, upload HTML/SVG — non testés

## Logique métier — CLASSE RETENUE
- [x] Manipulation prix/quantité/devise/remise — montants calculés côté serveur (aucun prix envoyé par le client) → non qualifiante
- [~] Étapes du tunnel sautées — accès direct au paiement charge l'étape mais adresse+livraison déjà complétées (suivi serveur) → pas un saut
- [ ] Valeurs négatives/grandes/décimales — non testé (pas de champ montant client)
- [ ] Race conditions (double usage coupon) — non testé
- [~] Absence de rate limit — carte cadeau protégée par Turnstile, code promo sans Turnstile (asymétrie notée) ; énumération interdite

## Côté serveur — non retenu
- [ ] SSRF / LFI / XXE / upload / smuggling — non testés

## Configuration
- [x] CORS — non exploitable : ACAO:* (wildcard sans credentials) + cookie SameSite=Strict → lecture cross-origin authentifiée impossible
- [x] En-têtes de sécurité — HSTS (max-age 31536000), x-frame-options: deny
- [ ] Subdomain takeover — non testé (sous-domaines hors scope)
- [ ] CVE manuelles — non testé

## Avant de rapporter
- [ ] Reproduit 2× — N/A (aucun finding confirmé)
- [x] Preuves caviardées + horodatées — tokens/PII jamais recopiés
- [x] Vérifié hors liste exclue — manip de prix, énumération, rate-limiting, open redirect simple = écartés
