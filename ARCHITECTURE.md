# 🏗️ Architecture et Flux de Sécurité

## Architecture Actuelle (Note F)

```
┌─────────────┐
│  Visiteur   │
│  (Browser)  │
└──────┬──────┘
	   │ HTTP/HTTPS
	   │ ❌ Pas de headers sécurisés
	   ↓
┌──────────────────────────┐
│   GitHub Pages           │
│   185.199.109.153        │
│                          │
│   ❌ Pas de .htaccess    │
│   ❌ Pas de PHP          │
│   ❌ Pas de config       │
└──────────────────────────┘
	   │
	   ↓
┌──────────────────────────┐
│   legare.ca              │
│   (HTML statique)        │
│                          │
│   ✅ Meta tags (partiel) │
│   ❌ HSTS manquant       │
└──────────────────────────┘

Résultat: Note F ❌
```

---

## Architecture Recommandée (Note A+)

```
┌─────────────┐
│  Visiteur   │
│  (Browser)  │
└──────┬──────┘
	   │
	   │ HTTPS forcé
	   ↓
┌──────────────────────────────────────────────┐
│              CLOUDFLARE                      │
│  ┌────────────────────────────────────────┐ │
│  │  Cloudflare Worker                     │ │
│  │  (cloudflare-worker.js)                │ │
│  │                                        │ │
│  │  ✅ Ajoute HSTS                        │ │
│  │  ✅ Ajoute CSP                         │ │
│  │  ✅ Ajoute X-Frame-Options             │ │
│  │  ✅ Ajoute X-Content-Type-Options      │ │
│  │  ✅ Ajoute Referrer-Policy             │ │
│  │  ✅ Ajoute Permissions-Policy          │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  + CDN (Cache, rapidité)                    │
│  + Protection DDoS                           │
│  + SSL/TLS                                   │
│  + Analytics                                 │
└──────────────┬───────────────────────────────┘
			   │
			   │ Requête proxifiée
			   ↓
┌──────────────────────────────────────────────┐
│           GitHub Pages                       │
│           185.199.109.153                    │
│                                              │
│   Sert le contenu HTML statique              │
└──────────────┬───────────────────────────────┘
			   │
			   ↓
┌──────────────────────────────────────────────┐
│           legare.ca                          │
│           (HTML + Meta tags)                 │
│                                              │
│   ✅ Meta tags (double protection)          │
│   ✅ HSTS (via Cloudflare)                  │
└──────────────────────────────────────────────┘

Résultat: Note A+ ✅
```

---

## Flux de Requête Détaillé

### 1️⃣ Sans Cloudflare (Situation actuelle - Note F)

```
Utilisateur tape: https://legare.ca/
		 ↓
	Résolution DNS
		 ↓
	185.199.109.153 (GitHub Pages)
		 ↓
	Serveur GitHub retourne HTML
		 ↓
	❌ Pas de HSTS
	❌ Pas de CSP (header)
	❌ Pas de Permissions-Policy
		 ↓
	Page affichée avec protection minimale
		 ↓
	🔴 Note: F
```

---

### 2️⃣ Avec Cloudflare (Solution recommandée - Note A+)

```
Utilisateur tape: http://legare.ca/
		 ↓
	Résolution DNS
		 ↓
	Nameservers Cloudflare
		 ↓
	Cloudflare force HTTPS (redirect 301)
		 ↓
	https://legare.ca/
		 ↓
┌────────────────────────────────────┐
│   Cloudflare Worker s'exécute      │
│                                    │
│   1. Fetch de GitHub Pages         │
│   2. Ajoute tous les headers       │
│   3. Retourne réponse sécurisée    │
└────────────────────────────────────┘
		 ↓
	Requête envoyée à GitHub Pages
	(185.199.109.153)
		 ↓
	GitHub retourne HTML
		 ↓
	Cloudflare ajoute les headers:
	✅ Strict-Transport-Security
	✅ Content-Security-Policy
	✅ X-Frame-Options
	✅ X-Content-Type-Options
	✅ Referrer-Policy
	✅ Permissions-Policy
		 ↓
	Mise en cache CDN (optionnel)
		 ↓
	Page affichée avec protection maximale
		 ↓
	🟢 Note: A+
```

---

## Comparaison Headers HTTP

### Avant (GitHub Pages seul)

```http
HTTP/2 200 OK
server: GitHub.com
content-type: text/html; charset=utf-8
last-modified: Thu, 10 Jul 2026 10:00:00 GMT
etag: "abc123"
cache-control: max-age=600
```

❌ **Aucun header de sécurité!**

---

### Après (GitHub Pages + Cloudflare)

```http
HTTP/2 200 OK
server: cloudflare
content-type: text/html; charset=utf-8
strict-transport-security: max-age=31536000; includeSubDomains; preload
content-security-policy: default-src 'self'; script-src 'self' 'unsafe-inline' ...
x-frame-options: SAMEORIGIN
x-content-type-options: nosniff
referrer-policy: strict-origin-when-cross-origin
permissions-policy: geolocation=(), microphone=(), camera=(), payment=()
x-xss-protection: 1; mode=block
cf-ray: 7d3f2a1b8c9d0e1f-YUL
cf-cache-status: HIT
```

✅ **Tous les headers de sécurité présents!**

---

## Configuration DNS

### Avant (DNS direct vers GitHub)

```
Type    Name        Value                   TTL
----------------------------------------------
A       legare.ca   185.199.108.153         3600
A       legare.ca   185.199.109.153         3600
A       legare.ca   185.199.110.153         3600
A       legare.ca   185.199.111.153         3600
CNAME   www         username.github.io      3600
```

---

### Après (DNS via Cloudflare)

```
Chez votre registraire:
Type    Name            Value
--------------------------------------------------
NS      legare.ca       mark.ns.cloudflare.com
NS      legare.ca       nina.ns.cloudflare.com

Dans Cloudflare:
Type    Name        Value                   Proxy Status
---------------------------------------------------------
A       @           185.199.108.153         ✅ Proxied
A       @           185.199.109.153         ✅ Proxied
A       @           185.199.110.153         ✅ Proxied
A       @           185.199.111.153         ✅ Proxied
CNAME   www         legare.ca               ✅ Proxied
```

⚠️ **Important**: Le statut "Proxied" (☁️ orange) est essentiel!

---

## Cloudflare Worker - Comment ça fonctionne

```javascript
// 1. Intercepte TOUTES les requêtes vers legare.ca
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

// 2. Pour chaque requête:
async function handleRequest(request) {
  // A. Récupère la page de GitHub Pages
  const response = await fetch(request)

  // B. Clone la réponse
  const newResponse = new Response(response.body, response)

  // C. Ajoute les headers de sécurité
  newResponse.headers.set('Strict-Transport-Security', '...')
  newResponse.headers.set('Content-Security-Policy', '...')
  // ... tous les autres headers

  // D. Retourne au visiteur
  return newResponse
}
```

### Résultat:
```
Visiteur → Cloudflare Worker → GitHub Pages
		   ↓ (ajoute headers)
		   ← Réponse sécurisée
```

---

## Timeline de Déploiement

```
Jour 1 - Heure 0:00
├─ Créer compte Cloudflare (5 min)
├─ Ajouter domaine (2 min)
├─ Noter les nameservers (1 min)
│
├─ Heure 0:10
├─ Se connecter au registraire (2 min)
├─ Changer les nameservers (5 min)
│
├─ Heure 0:20
├─ Configurer SSL/TLS (3 min)
├─ Activer HSTS (5 min)
├─ Créer Worker (5 min)
├─ Ajouter route Worker (2 min)
│
└─ Heure 0:35 ✅ Configuration terminée!

Propagation DNS: 30 min à 24h
├─ Après 30 min: 50% des utilisateurs
├─ Après 2h: 90% des utilisateurs
└─ Après 24h: 100% des utilisateurs

Test Note A+: Dès que DNS propagé ✅
```

---

## Sécurité - Avant/Après

### Attaques Bloquées AVANT Cloudflare

| Attaque | Protection |
|---------|------------|
| Man-in-the-Middle | ❌ Faible |
| Clickjacking | ❌ Aucune |
| XSS | ❌ Aucune |
| MIME Sniffing | ❌ Aucune |
| Injection Code | ❌ Aucune |
| DDoS | ❌ Aucune |

**Score total: 0/6** 🔴

---

### Attaques Bloquées APRÈS Cloudflare

| Attaque | Protection | Header Responsable |
|---------|------------|-------------------|
| Man-in-the-Middle | ✅ Forte | HSTS |
| Clickjacking | ✅ Forte | X-Frame-Options |
| XSS | ✅ Forte | CSP |
| MIME Sniffing | ✅ Forte | X-Content-Type-Options |
| Injection Code | ✅ Forte | CSP |
| DDoS | ✅ Forte | Cloudflare |

**Score total: 6/6** 🟢

---

## Monitoring et Vérification

### Tests à effectuer après déploiement:

```bash
# 1. Vérifier la résolution DNS
nslookup legare.ca

# 2. Vérifier les headers HTTP
curl -I https://legare.ca/

# 3. Tester HSTS
curl -I https://legare.ca/ | grep -i strict-transport

# 4. Tester CSP
curl -I https://legare.ca/ | grep -i content-security-policy

# 5. Vérifier Cloudflare
curl -I https://legare.ca/ | grep -i cf-ray
```

### Outils de test en ligne:

1. **Security Headers**
   - URL: https://securityheaders.com/
   - Test: `https://legare.ca/`
   - Objectif: **A+** ✅

2. **Mozilla Observatory**
   - URL: https://observatory.mozilla.org/
   - Test: `legare.ca`
   - Objectif: **A+** ✅

3. **SSL Labs**
   - URL: https://www.ssllabs.com/ssltest/
   - Test: `legare.ca`
   - Objectif: **A** ou **A+** ✅

---

## Rollback Plan (En cas de problème)

Si quelque chose ne fonctionne pas:

```
┌─────────────────────────────────────┐
│ PROBLÈME: Site inaccessible        │
└─────────────────────────────────────┘
		 ↓
┌─────────────────────────────────────┐
│ Solution 1: Pause Cloudflare        │
│ - Cloudflare Dashboard              │
│ - Quick Actions                     │
│ - Pause Cloudflare                  │
└─────────────────────────────────────┘
		 ↓
┌─────────────────────────────────────┐
│ Solution 2: Désactiver Worker       │
│ - Workers & Pages                   │
│ - Votre Worker                      │
│ - Delete Route                      │
└─────────────────────────────────────┘
		 ↓
┌─────────────────────────────────────┐
│ Solution 3: Rollback DNS            │
│ - Retour aux anciens nameservers    │
│ - Attendre propagation (24h)        │
└─────────────────────────────────────┘
```

---

## Résumé Visuel

```
┌─────────────────────────────────────────────────────────┐
│                   VOTRE PARCOURS                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🔴 F (Maintenant)                                      │
│  │                                                      │
│  ├─► Meta Tags ajoutés ✅                              │
│  │                                                      │
│  🟡 C/D (Après push GitHub)                            │
│  │                                                      │
│  ├─► Cloudflare configuré ✅                           │
│  │                                                      │
│  🟢 A+ (Dans 24h max)                                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🎯 Prochaine Étape

Suivez **GUIDE-RAPIDE.md** pour passer à A+ maintenant! 🚀
