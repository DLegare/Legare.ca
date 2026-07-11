# 🔧 Guide de Dépannage - Note de Sécurité

## 🔍 Diagnostic Rapide

### Mon site montre toujours une note F

#### Vérification 1: Meta tags présents?
```bash
curl https://legare.ca/ | grep "Content-Security-Policy"
```

✅ **Si vous voyez du texte**: Meta tags présents  
❌ **Si vide**: Meta tags manquants

**Solution si manquants**:
```bash
# Vérifier que vos changements sont sur GitHub
git status
git add *.html
git commit -m "Add security meta tags"
git push origin main
```

---

#### Vérification 2: Cloudflare actif?
```bash
curl -I https://legare.ca/ | grep -i "cf-ray"
```

✅ **Si vous voyez "cf-ray"**: Cloudflare actif  
❌ **Si vide**: Cloudflare pas encore actif

**Solution**:
- Attendre propagation DNS (jusqu'à 24h)
- Vérifier nameservers: https://dnschecker.org/

---

#### Vérification 3: Worker déployé?
```bash
curl -I https://legare.ca/ | grep -i "strict-transport-security"
```

✅ **Si vous voyez le header**: Worker actif  
❌ **Si vide**: Worker pas déployé ou route manquante

**Solution**:
1. Cloudflare Dashboard → Workers & Pages
2. Vérifier que le Worker existe
3. Settings → Triggers → Routes
4. Ajouter: `https://legare.ca/*`

---

## 🚨 Problèmes Courants

### Problème 1: Site inaccessible après Cloudflare

**Symptômes**:
- Erreur 521, 522 ou 523
- "Web server is down"
- "Connection timed out"

**Causes possibles**:
1. SSL/TLS mal configuré
2. GitHub Pages pas configuré pour le domaine
3. DNS non propagé

**Solutions**:

#### Solution A: Vérifier SSL/TLS
1. Cloudflare → SSL/TLS → Overview
2. Changer de "Flexible" à **"Full"**
3. Attendre 5 minutes
4. Tester à nouveau

#### Solution B: Vérifier GitHub Pages
1. GitHub → Settings → Pages
2. Custom domain: `legare.ca`
3. ✅ Enforce HTTPS
4. Save

#### Solution C: Vérifier DNS
```bash
# Vérifier si DNS est propagé
nslookup legare.ca

# Devrait montrer des IPs Cloudflare (commence par 104. ou 172.)
```

---

### Problème 2: Note toujours F malgré Cloudflare

**Symptômes**:
- Cloudflare actif (cf-ray présent)
- Mais note reste F sur securityheaders.com

**Causes possibles**:
1. Worker pas déployé
2. Route du Worker manquante
3. Cache du site de test

**Solutions**:

#### Solution A: Vérifier Worker
```bash
# Test manuel des headers
curl -I https://legare.ca/ | grep -i "strict-transport"
```

Si vide:
1. Cloudflare → Workers & Pages
2. Cliquez sur votre Worker
3. **Quick Edit** → Vérifier le code
4. **Save and Deploy**

#### Solution B: Vérifier Route
1. Worker → Settings → Triggers
2. Routes doit contenir: `https://legare.ca/*`
3. Zone: `legare.ca`

#### Solution C: Vider le cache
1. Sur securityheaders.com:
   - Ajoutez `?nocache` à l'URL
   - Test: `https://legare.ca/?nocache`

2. Dans Cloudflare:
   - Caching → Configuration
   - **Purge Everything**

---

### Problème 3: DNS ne se propage pas

**Symptômes**:
- Nameservers changés depuis 24h+
- Site toujours inaccessible
- DNS checker montre anciens DNS

**Solutions**:

#### Solution A: Vérifier chez le registraire
1. Connexion à votre registraire (GoDaddy, Namecheap, etc.)
2. Gestion DNS ou Nameservers
3. Vérifier que les nameservers Cloudflare sont EXACTEMENT:
   - `mark.ns.cloudflare.com` (exemple)
   - `nina.ns.cloudflare.com` (exemple)

⚠️ **Attention**: Prenez les nameservers de VOTRE compte Cloudflare, pas cet exemple!

#### Solution B: Forcer la propagation
```bash
# Windows
ipconfig /flushdns

# Mac/Linux
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder

# Tester
nslookup legare.ca 8.8.8.8
```

#### Solution C: Patienter
- DNS peut prendre jusqu'à 48h (rare)
- Utiliser https://dnschecker.org/ pour suivre la propagation
- Si après 48h rien ne change → contacter le registraire

---

### Problème 4: Worker ne s'exécute pas

**Symptômes**:
- Worker déployé
- Route configurée
- Mais pas de headers ajoutés

**Diagnostic**:
```bash
# Vérifier que Cloudflare proxy la requête
curl -I https://legare.ca/ | grep -i "cf-"
```

Si aucun header `cf-*`:
- ⚠️ Le traffic ne passe PAS par Cloudflare
- ☁️ Vérifier que le DNS est en mode "Proxied" (orange)

**Solutions**:

#### Solution A: Activer le Proxy
1. Cloudflare → DNS → Records
2. Trouvez votre enregistrement `A` pour `legare.ca`
3. Cliquez sur l'icône cloud:
   - 🟠 Orange (Proxied) = BON ✅
   - ⚪ Gris (DNS only) = MAUVAIS ❌
4. Cliquez pour activer le proxy (orange)

#### Solution B: Vérifier le code du Worker
```javascript
// Le Worker DOIT contenir ceci:
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const response = await fetch(request)
  const newResponse = new Response(response.body, response)

  // Headers...
  newResponse.headers.set('Strict-Transport-Security', '...')

  return newResponse
}
```

#### Solution C: Logs du Worker
1. Worker → Logs (tab)
2. Vérifier les erreurs
3. Si erreur JavaScript → corriger le code

---

### Problème 5: CSP bloque des ressources

**Symptômes**:
- Site fonctionne
- Note A+
- Mais images/scripts ne se chargent pas
- Console browser: "Content Security Policy blocked..."

**Cause**:
Le CSP est trop restrictif pour votre site.

**Solution**:

#### Identifier la ressource bloquée
1. Ouvrir console browser (F12)
2. Chercher erreurs CSP
3. Exemple: `Refused to load script from 'https://example.com/script.js'`

#### Modifier le Worker
Éditer `cloudflare-worker.js`:

```javascript
// Avant (restrictif)
script-src 'self' 'unsafe-inline' https://maxcdn.bootstrapcdn.com

// Après (ajouter le domaine bloqué)
script-src 'self' 'unsafe-inline' https://maxcdn.bootstrapcdn.com https://example.com
```

#### Redéployer
1. Cloudflare → Workers → Votre Worker
2. Quick Edit
3. Modifier la ligne CSP
4. Save and Deploy
5. Purger le cache

---

### Problème 6: HSTS bloque l'accès HTTP

**Symptômes**:
- Impossible d'accéder à http://legare.ca/
- Browser force https://
- Message: "HSTS policy active"

**Explication**:
- C'est NORMAL et DÉSIRÉ ✅
- HSTS force HTTPS pour la sécurité
- Durée: 1 an (31536000 secondes)

**Si vous voulez désactiver (non recommandé)**:

#### Solution temporaire (développement uniquement)
1. Worker → Modifier HSTS:
```javascript
// Avant
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload'

// Après (30 jours)
'Strict-Transport-Security': 'max-age=2592000; includeSubDomains'
```

2. Vider le cache HSTS du browser:
   - Chrome: `chrome://net-internals/#hsts`
   - Chercher `legare.ca`
   - Delete domain

⚠️ **Attention**: Cela RÉDUIRA votre note de sécurité!

---

## 🧪 Tests de Validation

### Test Complet

Copiez ce script dans un terminal:

```bash
#!/bin/bash

echo "🔍 Diagnostic de sécurité pour legare.ca"
echo "========================================="
echo ""

# Test 1: DNS
echo "📡 Test 1: Résolution DNS"
nslookup legare.ca | grep "Address" | tail -2
echo ""

# Test 2: Cloudflare actif
echo "☁️  Test 2: Cloudflare actif?"
curl -sI https://legare.ca/ | grep -i "cf-ray" && echo "✅ Cloudflare actif" || echo "❌ Cloudflare inactif"
echo ""

# Test 3: HTTPS
echo "🔒 Test 3: HTTPS fonctionnel?"
curl -sI https://legare.ca/ | head -1
echo ""

# Test 4: HSTS
echo "🛡️  Test 4: HSTS présent?"
curl -sI https://legare.ca/ | grep -i "strict-transport" && echo "✅ HSTS actif" || echo "❌ HSTS manquant"
echo ""

# Test 5: CSP
echo "🔐 Test 5: CSP présent?"
curl -sI https://legare.ca/ | grep -i "content-security-policy" && echo "✅ CSP actif" || echo "❌ CSP manquant"
echo ""

# Test 6: X-Frame-Options
echo "🖼️  Test 6: X-Frame-Options présent?"
curl -sI https://legare.ca/ | grep -i "x-frame-options" && echo "✅ X-Frame-Options actif" || echo "❌ X-Frame-Options manquant"
echo ""

# Test 7: X-Content-Type-Options
echo "📄 Test 7: X-Content-Type-Options présent?"
curl -sI https://legare.ca/ | grep -i "x-content-type-options" && echo "✅ X-Content-Type-Options actif" || echo "❌ X-Content-Type-Options manquant"
echo ""

# Résumé
echo "========================================="
echo "🎯 Testez votre note sur:"
echo "   https://securityheaders.com/?q=https://legare.ca/"
echo ""
echo "Objectif: A+ ✅"
```

---

### Test Windows (PowerShell)

```powershell
Write-Host "🔍 Diagnostic de sécurité pour legare.ca" -ForegroundColor Cyan
Write-Host "=========================================" -ForegroundColor Cyan
Write-Host ""

# Test DNS
Write-Host "📡 Test DNS" -ForegroundColor Yellow
nslookup legare.ca

# Test Cloudflare
Write-Host "`n☁️  Test Cloudflare" -ForegroundColor Yellow
$response = Invoke-WebRequest -Uri "https://legare.ca/" -Method Head -UseBasicParsing
if ($response.Headers["cf-ray"]) {
	Write-Host "✅ Cloudflare actif" -ForegroundColor Green
} else {
	Write-Host "❌ Cloudflare inactif" -ForegroundColor Red
}

# Test HSTS
Write-Host "`n🛡️  Test HSTS" -ForegroundColor Yellow
if ($response.Headers["Strict-Transport-Security"]) {
	Write-Host "✅ HSTS actif: $($response.Headers['Strict-Transport-Security'])" -ForegroundColor Green
} else {
	Write-Host "❌ HSTS manquant" -ForegroundColor Red
}

# Test CSP
Write-Host "`n🔐 Test CSP" -ForegroundColor Yellow
if ($response.Headers["Content-Security-Policy"]) {
	Write-Host "✅ CSP actif" -ForegroundColor Green
} else {
	Write-Host "❌ CSP manquant" -ForegroundColor Red
}

Write-Host "`n=========================================" -ForegroundColor Cyan
Write-Host "🎯 Testez votre note sur:" -ForegroundColor Cyan
Write-Host "   https://securityheaders.com/?q=https://legare.ca/" -ForegroundColor White
```

---

## 📊 Checklist de Dépannage

Cochez au fur et à mesure:

### Configuration Cloudflare
- [ ] Compte Cloudflare créé
- [ ] Domaine ajouté à Cloudflare
- [ ] Nameservers notés (ex: mark.ns.cloudflare.com)
- [ ] Nameservers changés chez le registraire
- [ ] DNS propagé (vérifier sur dnschecker.org)
- [ ] SSL/TLS configuré en "Full"
- [ ] "Always Use HTTPS" activé
- [ ] HSTS activé (max-age: 31536000)
- [ ] DNS en mode "Proxied" (☁️ orange, PAS gris)

### Configuration Worker
- [ ] Worker créé
- [ ] Code de `cloudflare-worker.js` copié
- [ ] Worker déployé (Save and Deploy)
- [ ] Route ajoutée: `https://legare.ca/*`
- [ ] Zone sélectionnée: `legare.ca`
- [ ] Pas d'erreur dans les logs

### Configuration GitHub Pages
- [ ] Repository contient les fichiers HTML
- [ ] Custom domain: `legare.ca`
- [ ] "Enforce HTTPS" activé
- [ ] CNAME file présent (automatique)
- [ ] Meta tags de sécurité dans tous les HTML

### Tests
- [ ] `curl -I https://legare.ca/` fonctionne
- [ ] Header `cf-ray` présent (Cloudflare actif)
- [ ] Header `strict-transport-security` présent (HSTS)
- [ ] Header `content-security-policy` présent (CSP)
- [ ] Test sur securityheaders.com: **A+**

---

## 🆘 Support et Aide

### Si rien ne fonctionne après 48h

1. **Pause Cloudflare temporairement**:
   - Cloudflare Dashboard
   - Quick Actions → Pause Cloudflare
   - Tester si le site fonctionne
   - Reprendre Cloudflare

2. **Vérifier GitHub Pages**:
   - Repository → Settings → Pages
   - Voir si le site se déploie
   - Vérifier les erreurs de build

3. **Rollback DNS**:
   - Remettre les anciens nameservers
   - Site fonctionnera comme avant
   - Retry Cloudflare plus tard

### Communautés d'aide

- **Cloudflare Community**: https://community.cloudflare.com/
- **GitHub Discussions**: https://github.com/orgs/community/discussions
- **Stack Overflow**: Tag `cloudflare` ou `github-pages`

### Logs à fournir pour support

Si vous demandez de l'aide, fournissez:

```bash
# 1. DNS actuel
nslookup legare.ca

# 2. Headers HTTP
curl -I https://legare.ca/

# 3. Test Cloudflare
curl -I https://legare.ca/ | grep -i "cf-"

# 4. Test HSTS
curl -I https://legare.ca/ | grep -i "strict-transport"

# 5. Screenshot du Worker dans Cloudflare
# 6. Screenshot des DNS records dans Cloudflare
# 7. Screenshot de securityheaders.com
```

---

## ✅ Validation Finale

Une fois tout configuré, vous devriez voir:

```bash
$ curl -I https://legare.ca/

HTTP/2 200
server: cloudflare
strict-transport-security: max-age=31536000; includeSubDomains; preload
content-security-policy: default-src 'self'; script-src ...
x-frame-options: SAMEORIGIN
x-content-type-options: nosniff
referrer-policy: strict-origin-when-cross-origin
permissions-policy: geolocation=(), microphone=(), camera=(), payment=()
cf-ray: 7d3f2a1b8c9d0e1f-YUL
cf-cache-status: HIT
```

**Test final**: https://securityheaders.com/?q=https://legare.ca/

**Résultat attendu**: 
```
┌─────────────────────────┐
│   Security Headers      │
│   Report Summary        │
│                         │
│   Grade: A+             │
│                         │
│   ✅ All headers present│
└─────────────────────────┘
```

---

**Si vous avez ce résultat: FÉLICITATIONS! 🎉**

Vous avez réussi à sécuriser votre site et obtenir la meilleure note possible!

---

**Besoin d'aide?** Consultez:
- `GUIDE-RAPIDE.md` - Instructions pas à pas
- `COMPARAISON-SOLUTIONS.md` - Comprendre les options
- `ARCHITECTURE.md` - Schémas et explications détaillées
