# Checklist API — www.mediamarkt.de (/api/v1/graphql)

> Dupliquée de docs/03-checklist-api.md. OWASP API Security Top 10.
> Légende : [x] testé · [~] partiel / candidat non confirmé · [ ] non testé.

## Découverte
- [~] Doc exposée (swagger/openapi/api-docs) — cherchés, non trouvés
- [~] GraphQL + introspection — endpoint /api/v1/graphql confirmé (persisted queries) ; introspection non testée (constat à faire, non qualifiante seule)
- [ ] Versions parallèles (/v1, /v2, /internal, /beta) — /v1 confirmé ; autres non testées
- [~] Endpoints extraits du JS / du trafic — GetUser, GetBasket, GetPointsBalance, GetPointsTransactions, GetPromotionCoupons, AddGiftcard, mutation code promo, GetConsentCategories, GetCheckoutConfig… ; lecture complète des bundles JS = à faire
- [ ] Différences API web vs mobile — non testé (apps mobiles in scope, non étudiées)

## API1 — BOLA / IDOR — RETENU
- [~] Remplacer un ID par celui d'un autre compte — 2 candidats (partyId, basketId) ; besoin du compte B
- [x] IDs devinables — partyId séquentiel (énumérable) ; basketId UUID (non devinable)
- [ ] Objets imbriqués (deux niveaux vérifiés ?) — non testé
- [x] Prouver l'accès, ne pas exfiltrer de données clients — respecté (lecture seule, PII non collectées)

## API2 — Authentification — RETENU (partiel)
- [ ] Endpoints accessibles sans token — non testé
- [ ] Token expiré / révoqué accepté — non testé (exp 15 min observé)
- [ ] Clé API dans l'URL — N/A (partyId = paramètre, pas secret)
- [~] JWT alg:none / confusion HS-RS / signature non vérifiée — à faire (T1)

## API3 — Autorisation au niveau propriété
- [ ] Mass assignment (role:admin, isVerified:true, balance:9999) — non testé (candidat : mutation update profil/adresse)
- [ ] Sur-exposition (réponse > UI : PII, hash, tokens) — non testé (candidat : réponse GetUser vs UI)

## API4 — Consommation de ressources
- [~] Pagination sans limite — GetPromotionCoupons accepte limit:1000 (client) ; non abusé (pas de DoS)
- [ ] GraphQL profond / aliasing / batching — non testé

## API5 — BFLA
- [ ] Fonctions admin avec token user — non testé
- [ ] Méthodes non documentées (PUT/DELETE/PATCH) — non testé

## API6 — Flux métier sensibles
- [~] Automatiser un flux manuel — checkout cartographié ; ajout code promo sans captcha (noté) ; non automatisé (interdit)

## API7 — SSRF
- [ ] Champs acceptant une URL — non repéré / non testé

## API8 — Mauvaise configuration
- [~] CORS / verbes / en-têtes de debug — access-control-allow-origin: * sur GetBasket noté
- [ ] Erreurs exposant stack / SQL — non observé

## API9 — Inventaire
- [x] Hôtes d'API en sous-domaine = HORS SCOPE — repérés, non testés

## API10 — Consommation d'API tierces
- [ ] Données d'un service tiers reprises sans validation — non testé

## GraphQL spécifique
- [~] Introspection en prod — à tester (constat)
- [~] Mutations sensibles sans contrôle d'accès — AddGiftcard + coupon prennent basketId côté client (candidat BOLA) ; GetPromotionCoupons dérive l'utilisateur de la session (bonne pratique)
- [ ] Contournement de rate limit par batching — non testé
- [ ] Suggestions de champs (fuite de schéma) — non testé

## Avant de rapporter
- [ ] Faille qualifiante, impact démontré, PoC minimal — N/A (candidats en attente du compte B)
