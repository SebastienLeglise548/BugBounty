# Recon passive — mediamarkt.de (aucun paquet non sollicité vers la cible)

> Relevé 2026-11-11 (WHOIS/DNS/mail/robots/CT) + 2026-11-13 (crt.sh + subfinder scriptés).

## WHOIS
- Registre DENIC (.de), titulaire masqué. Changement de statut daté du 2026-03-27.
- Serveurs de noms : Akamai.

## DNS
- `mediamarkt.de` → bloc Cloudflare. `www` = CNAME Cloudflare. Pas d'enregistrement AAAA.
- Messagerie : Microsoft 365.
- Résolution des 12 hôtes in-scope : 11 vers `*.cdn.cloudflare.net` ; `www.mediamarkt.lu` → `mm-lux.myshopify.com` (Shopify). Détail dans `dns-2026-11-13.txt`.

## Politique e-mail (anti-usurpation)
- SPF `-all` (échec strict). DMARC `p=reject`, alignement strict. Posture forte.

## TXT (services SaaS visibles)
- Microsoft 365, Dynamics 365, Adobe, Atlassian, Cisco, KnowBe4, Pinterest, Validity.

## robots.txt / sitemaps
- Endpoints révélés : `/api/v1/graphql` (GetPaidBundles, GetFreeBundles), `/api/v1/msg`, `/api/v1/mms-sst`, `/api/v1/partners/`, `/rde_server/`, `*/myaccount/auth*`.
- Sitemaps sous `/sitemaps/`. `/sitemap.xml` racine → 404.

## Certificats publics (crt.sh + subfinder)
- ~48 sous-domaines (crt.sh, 11/11) ; 164 au total (crt.sh + subfinder, 13/11).
- **Un seul dans le périmètre** : `www.mediamarkt.de`. Tous les autres = sous-domaines **hors scope**, documentés comme paysage du SI (recette, cartes cadeaux, tarifs, logistique, supervision), **jamais testés**. Liste filtrée dans `subdomains-2026-11-13.txt`.

## Fingerprint technologique (passif, déduit)
- CDN/protection Cloudflare, DNS Akamai, messagerie Microsoft 365, API GraphQL, Sentry, Forter, ContentSquare.

## Code public de tiers
- Dépôts tiers qui appellent l'API publique du site. **Aucun secret/clé/jeton** de l'entreprise exposé. Usage retenu : enrichir la liste des opérations GraphQL (CategoryV4, GetProductCollectionItems). Rien d'autre à utiliser.

## Reporté / indisponible
- Archives web publiques (service indisponible ce jour), dorks sur fichiers, vérification par domaine dans une base de fuites (compte requis), lecture des bundles JavaScript (prévue en recon active).
