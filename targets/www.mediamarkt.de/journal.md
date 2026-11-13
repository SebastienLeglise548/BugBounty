# Journal de bord — www.mediamarkt.de

Une ligne par session. Preuve de bonne foi en cas de litige, et mémoire de travail.
Cible unique testée : www.mediamarkt.de. Les 11 autres hôtes du scope ne servent qu'à la recon.

## 2026-11-10 — Conformité et préparation du proxy

**Fait**
- Compte chasseur YesWeHack actif, alias de messagerie `@yeswehack.ninja` prêt pour les comptes de test.
- Dans Burp, création d'une règle Match & Replace qui ajoute un suffixe d'identification au User-Agent
  (`-MMS-BugBounty`), pour que l'équipe défense puisse reconnaître mon trafic.
- Vérification de la règle sur `https://httpbin.org/user-agent` : le User-Agent renvoyé se termine bien
  par `-MMS-BugBounty`. La règle fonctionne de bout en bout.
- Restriction du scope de Burp aux 12 hôtes autorisés (include ancré par regex), exclusion de
  `*.mediamarkt.pt` et des sous-domaines. Historique limité aux éléments in-scope.

**Observations**
- L'historique Burp affiche la requête avant modification, mais le User-Agent réellement envoyé porte
  bien le suffixe (confirmé côté réponse httpbin).

**Prochaine étape**
- Démarrer la cartographie de l'entreprise, puis de la surface de www.mediamarkt.de.

## 2026-11-11 — Cartographie entreprise, surface et recon passive

**Fait**
- Cartographie de l'entreprise en sources ouvertes (100 % passif) : identité du groupe, modèle
  e-commerce, zones à enjeu, antécédent de sécurité.
- Cartographie de surface de www.mediamarkt.de par navigation manuelle à débit bas dans le navigateur
  Burp : endpoints, API GraphQL, authentification, parcours d'achat, défenses.
- Recon passive : WHOIS, DNS (A, MX, NS, TXT), politique e-mail (SPF, DMARC), robots.txt et sitemaps,
  certificats publics (crt.sh).

**Observations**
- Groupe MediaMarktSaturn Retail Group GmbH (Ingolstadt), filiale de Ceconomy AG ; rachat par JD.com
  en cours (à confirmer). Marques MediaMarkt, Saturn, MediaWorld. ~50 000 salariés, 1 000+ magasins,
  11 pays. Antécédent : rançongiciel Hive en novembre 2021 (NL/DE).
- Front Cloudflare avec challenge Turnstile (403 probabilistes), anti-fraude Forter, suivi d'erreurs
  Sentry, session replay ContentSquare. Le challenge ne se rend pas toujours dans la VM.
- API applicative en GraphQL (`/api/v1/graphql?operationName=...`) avec requêtes persistées (hash
  SHA-256). Statuts mixtes 200/403 selon la requête : l'API est atteignable, le blocage Cloudflare est
  probabiliste, pas total.
- Authentification par jeton JWT RS256 posé dès la navigation anonyme (jeton invité). Connexion en
  email-first, plus OAuth Google/Apple, MediaMarkt-ID, et commande invité.
- URLs catégorie `/de/category/<slug>-<id>.html` avec identifiants courts et séquentiels (202, 203) ;
  produit `/de/product/_<slug>-<id>.html` avec identifiant à 7 chiffres (= numéro d'article).
- `/de/checkout` renvoie une erreur 500 en accès direct comme en parcours normal : tunnel de paiement
  inaccessible cette séance. Anomalie d'affichage : clé de traduction brute `basket.summary.vat` visible.
- Politique e-mail stricte (SPF `-all`, DMARC `p=reject`). Certificats publics : ~48 sous-domaines,
  tous hors scope, notés comme paysage du SI, jamais testés.

**Impasses** (ne pas refaire)
- Automatisation bloquée par Cloudflare/Turnstile : les tests passeront par le navigateur, en manuel.
- Archives web publiques indisponibles ce jour (à relancer plus tard).

**Prochaine étape**
- Mettre le travail dans le dépôt, puis compléter : checkout en session connectée, lecture des
  requêtes GraphQL à 200, format des identifiants de commandes et factures (pistes IDOR).

## 2026-11-13 — Mise en place du dépôt et recon active

**Fait**
- Fork et clone du dépôt modèle, mise en place du périmètre dans `scope/`.
- Recon passive scriptée sur `mediamarkt.de` (crt.sh + subfinder) : 164 sous-domaines, 1 seul dans le
  périmètre (`www.mediamarkt.de`).
- Recon active légère : `httpx` sur les 12 hôtes in-scope (5 req/s, User-Agent d'identification) et
  résolution DNS des 12 hôtes. Résultats dans `recon/`.

**Observations**
- Les 12 cibles sont derrière Cloudflare, la plupart avec Cloudflare Bot Management (challenge
  « Just a moment... », 403 en CLI). HSTS partout, HTTP/3 sur plusieurs hôtes.
- La résolution DNS confirme Cloudflare sur 11 hôtes. `www.mediamarkt.lu` est un CNAME vers
  `mm-lux.myshopify.com` : ce site tourne sur Shopify, architecture distincte des autres.

**Prochaine étape**
- Configurer Burp sur la cible, créer deux comptes de test, relever le checkout connecté et le format
  des identifiants (IDOR), lire une requête GraphQL à 200.

## 2026-11-13 (soir) — Session de test (compte A)

Objectif : relever le tunnel de paiement en session connectée et cartographier les surfaces à enjeu
(contrôle d'accès, logique métier).

- Checkout connecté relevé : panier → adresse → livraison → paiement → récapitulatif. L'erreur 500
  observée le 11/11 sur /de/checkout ne se produit qu'en accès direct anonyme ; connecté, la page
  s'affiche normalement (le 500 n'est donc pas un défaut exploitable).
- Moyens de paiement : Apple Pay, Klarna, financement (Consors Finanz), Amazon Pay, carte (Click to
  Pay), Vorkasse, virement, PayPal.
- Trois leviers de réduction au paiement : carte cadeau, code promo, coupons de fidélité.
- Contrôle d'accès : plusieurs opérations GraphQL transmettent des identifiants d'objets côté client —
  `partyId` (solde et historique de points, identifiant séquentiel) et `basketId` (mutations de panier,
  UUID). Candidats IDOR/BOLA à confirmer avec un second compte.
- Bonne pratique à l'inverse : `GetPromotionCoupons` dérive l'utilisateur de la session, sans identifiant
  client.
- Logique métier : les montants sont calculés côté serveur (aucun prix transmis par le client) ; l'ajout
  d'une carte cadeau est protégé par un challenge anti-robot, l'ajout d'un code promo ne l'est pas
  (asymétrie notée, non testée — l'énumération est hors périmètre).
- Conformité : correction d'un réglage navigateur qui empêchait le suffixe d'identification du User-Agent
  de s'appliquer à tout le trafic.

Aucune vulnérabilité prouvée à ce stade (Cas B). Surface plutôt durcie.
