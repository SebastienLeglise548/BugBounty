# Checklist API — www.mediamarkt.de (/api/v1/graphql)

> OWASP API Security Top 10. [x] testé · [~] partiel · [ ] non testé.
> API GraphQL à requêtes persistées (hash par opération), client Apollo pwa-client-pqm.

## Découverte
- [x] Doc exposée — négatif : /swagger.json, /openapi.json, /api/v1/swagger.json, /api-docs → 404/Not Found
- [x] GraphQL + introspection — GET brut rejeté, POST bloqué au edge (403). GET persistés uniquement → introspection NON accessible
- [x] Versions parallèles — /api/v2, /api/internal, /api/beta, /graphql → tous 404. API cloisonnée sur /api/v1
- [x] Endpoints extraits du JS / trafic — opérations relevées par navigation ; bundles JS lus (manifeste chargé dynamiquement, non énumérable statiquement)
- [ ] Différences API web vs mobile — non testé (apps mobiles in scope, non étudiées)

## API1 — BOLA / IDOR — RETENU
- [~] Remplacer un ID par celui d'un autre compte — 2 candidats (partyId, basketId) ; besoin du compte B
- [x] IDs devinables — partyId séquentiel (énumérable) ; basketId UUID (non devinable)
- [ ] Objets imbriqués — non testé
- [x] Prouver l'accès, ne pas exfiltrer de données — respecté (lecture seule, PII non collectées)

## API2 — Authentification — RETENU
- [x] Endpoints sans token — Unauthorized sans session valide (auth serveur-side)
- [x] Token expiré / révoqué — validation JWT complète prouvée ; exp présent + refresh → expiration appliquée
- [ ] Clé API dans l'URL — N/A (partyId = paramètre, pas secret)
- [x] JWT signature + alg:none + confusion RS256→HS256 — tous rejetés. JWT validé de bout en bout

## API3 — Autorisation niveau propriété
- [x] Mass assignment — non vulnérable : rejeu d'UpdatePersonalAccountProfile avec champs injectés (type:B2B, role:admin, isVerified) → 200 mais champs ignorés (type reste B2C). Non qualifiant
- [x] Sur-exposition — négatif : getProfile renvoie la fiche de l'utilisateur lui-même, pas de données tierces/hash/token

## API4 — Consommation de ressources
- [~] Pagination sans limite — GetPromotionCoupons accepte limit:1000 ; non abusé (pas de DoS)
- [x] GraphQL profond / aliasing / batching — non faisable : POST arbitraire bloqué 403

## API5 — BFLA
- [~] Fonctions admin avec token user — non testable (hashes des opérations admin inconnus, manifeste dynamique ; POST bloqué)
- [x] Méthodes non documentées — PUT/DELETE/PATCH/OPTIONS → tous 403

## API6 — Flux métier sensibles
- [~] Automatiser un flux manuel — checkout cartographié ; non automatisé (interdit)

## API7 — SSRF
- [x] Champs acceptant une URL — aucun champ URL dans l'espace compte (tout structuré) → SSRF non applicable

## API8 — Mauvaise configuration
- [x] CORS / verbes / debug headers — ACAO:* non exploitable ; PUT/DELETE/PATCH/OPTIONS → 403
- [x] Erreurs exposant stack / SQL — requête invalide → 403 HTML générique, aucune fuite

## API9 — Inventaire
- [x] Hôtes d'API en sous-domaine = HORS SCOPE — repérés, non testés

## API10 — Consommation d'API tierces
- [ ] Données d'un service tiers reprises sans validation — non testé

## GraphQL spécifique
- [x] Introspection en prod — NON accessible (GET brut rejeté + POST 403)
- [~] Mutations sensibles sans contrôle d'accès — AddGiftcard + coupon prennent basketId côté client (candidat BOLA) ; GetPromotionCoupons = identité par session
- [x] Batching rate-limit bypass / suggestions de champs — non faisables (POST arbitraire bloqué 403)

## Avant de rapporter
- [ ] Faille qualifiante, impact démontré, PoC minimal — N/A (candidats en attente du compte B)
