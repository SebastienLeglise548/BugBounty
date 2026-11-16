# Preuves — www.mediamarkt.de

Captures d'écran des tests clés (compte A, novembre 2026). Seules des captures **sans donnée
sensible** sont publiées : les jetons, cookies de session, identifiants de compte et e-mails ont été
écartés de la sélection (les captures les contenant ne sont pas versées au dépôt).

| Fichier | Ce que la capture montre |
|---|---|
| `jwt-alg-confusion-redirection-login.png` | Jeton forgé par **confusion d'algorithme RS256→HS256** rejeté → redirection vers la page de connexion. L'authentification n'est pas contournée. |
| `xss-recherche-script-erreur-500.png` | Charge `<script>alert(17)</script>` dans la recherche → **erreur 500** (filtrage WAF Cloudflare). Pas de XSS réfléchie. |
| `sqli-recherche-texte-litteral.png` | `test' AND '1'='1` traité comme **texte littéral** (454 résultats) → recherche paramétrée, non injectable. |
| `chat-live-oracle-tiers-hors-scope.png` | Le live chat appelle des domaines **tiers** (`custhelp.com`, `rightnowtech.com`, `rnengage.com` = Oracle Service Cloud) → hors périmètre, non testé. |
| `api-methodes-http-403.png` | Méthodes `PUT` / `DELETE` / `PATCH` / `OPTIONS` sur `/api/v1/graphql` → toutes **403**. API verrouillée aux requêtes persistées. |

Aucune vulnérabilité confirmée (Cas B) : ces captures illustrent la démarche et les défenses observées.
