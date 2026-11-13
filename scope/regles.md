# Règles du programme — MediaMarktSaturn Bug Bounty Program

- **Plateforme** : YesWeHack (programme public)
- **URL de la policy** : https://yeswehack.com/programs/mediamarktsaturn-bug-bounty-program
- **Dernière MAJ programme** : 2026-11-04 (relevé le 2026-11-13)
- **Safe harbor** : non explicite sur la page. Hors-scope → VDP : https://vdp.mediamarktsaturn.com
- **Débit maximum** : non chiffré — « limit yourself about requests per second », « refrain from using automated tools »
- **En-tête d'identification exigé** : **oui** — ajouter au User-Agent le suffixe `-MMS-BugBounty`
- **Comptes de test** : alias `@yeswehack.ninja` obligatoire
- **Scanners automatisés** : **interdits** ; ne pas scanner l'infra de l'hébergeur PlusServer GmbH
- **Langue des rapports** : anglais · **1re réponse annoncée** : < 1 jour

## Techniques interdites

- [x] DoS / stress test — « any type of denial of service attacks » + interférence réseau/infra
- [x] Social engineering / phishing — accès aux systèmes ou contact des employés
- [x] Spam (SMS, e-mail) — « bulk messaging » interdit
- [x] Tests sur des comptes tiers — ne pas accéder/modifier/supprimer les comptes d'autres clients
- [x] Scanners automatisés — interdits
- [ ] Brute force — non nommé explicitement (mais « absence de rate limiting / brute force » = non récompensé ; débit à limiter)
- [x] Autre : violer les lois (DE/EU) ; créer plus de comptes que nécessaire ; copier/divulguer des fichiers ; toute divulgation même partielle ; scanner l'infra PlusServer

### Règles de preuve de concept (PoC)
- Ne modifier **que ses propres données**.
- SQLi : se limiter à identifier la **version du SGBD**.
- XSS : `alert()` minimal, **sans script externe**.
- Formulaire de contact : **un seul** message de test.
- **Aucune PII** dans le rapport (caviarder).
- Compte compromis : ne pas le modifier ni l'utiliser pour chercher des failles post-auth.

## Vulnérabilités hors récompense (non-qualifying) — à relire AVANT chaque soumission

- DoS ; clickjacking ; flags de cookies manquants ; en-têtes de sécurité manquants sans exploitation.
- CSRF faible sévérité (logout, login, panier) ; SPF/DKIM/DMARC ; gestion de session.
- **Énumération d'utilisateurs** ; absence de rate limiting / brute force / captcha ; mots de passe faibles.
- Subdomain takeover ; **SSRF aveugle sans PoC** ; clés API publiques (Google Maps, Firebase).
- **Introspection GraphQL** ; self-XSS et XSS sans impact tiers ; XSS via Host header ; CVE < 90 jours.
- SSL/TLS ; upload de fichiers malveillants ; CSV injection ; mixed content.
- Info disclosure (stack traces, versions, IP, EXIF, Origin IP) ; rapports de scanners non validés.
- Protections client mobile (pinning, obfuscation, root/jailbreak).
- Injection de message non authentifiée sur `/api/v1/msg` ou `/api/v1/mms-sst`.
- **Manipulation de prix côté client rejetée par le backend**.

## Récompenses (CVSS v3.1, en €)

| Sévérité | Score CVSS | Montant |
|---|---|---|
| Faible (Low) | 0.1–3.9 | 100 – 200 € |
| Moyen (Medium) | 4.0–6.9 | 200 – 600 € |
| Élevé (High) | 7.0–8.9 | 600 – 2 500 € |
| Critique (Critical) | 9.0–10.0 | 2 500 – 4 000 € |

- Montant **interpolé** dans la bande selon le score, arrondi à 10 € (ex. 5.5 → 410 €, 8.0 → 1 600 €, 9.8 → 3 700 €).
- **Bonus d'impact** : jusqu'à +50 %, à la discrétion de l'équipe.
- **Systemic issues** (même faille, N endpoints) : 1er/2e 100 %, 3e 75 %, 4e 50 %, 5e 25 %, 6e+ 10 %.
- **OneFixOneReward** : un seul correctif couvrant plusieurs endpoints = une seule prime, les autres fermés Informative.

## Contact

- Sécurité : pas d'e-mail affiché — soumission via la plateforme YesWeHack (compte requis).
- Hors-scope : VDP https://vdp.mediamarktsaturn.com
- Délai 1re réponse : < 1 jour. Triage/résolution : non annoncés.
- Contestation : un commentaire consolidé unique ; décision de réévaluation finale.
