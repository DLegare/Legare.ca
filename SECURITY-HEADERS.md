# Security Headers Configuration for GitHub Pages

Votre site https://legare.ca/ a actuellement une note de sécurité **F**. Voici comment améliorer cette note à **A+**.

## ❌ Problème: GitHub Pages ne supporte pas les headers HTTP personnalisés

GitHub Pages ne permet pas:
- Les fichiers `.htaccess` (Apache seulement)
- Les fichiers PHP comme `security-headers.php`
- Les fichiers `_headers` natifs

## ✅ Solutions pour améliorer votre note de sécurité

### **Solution 1: Cloudflare (Recommandé - Gratuit)**

C'est la meilleure solution pour ajouter des headers de sécurité à votre site GitHub Pages.

#### Étapes:

1. **Créer un compte Cloudflare gratuit** sur https://dash.cloudflare.com/sign-up

2. **Ajouter votre domaine** (legare.ca) à Cloudflare

3. **Mettre à jour vos DNS** chez votre registraire de domaine avec les nameservers Cloudflare

4. **Activer HTTPS/SSL** dans Cloudflare:
   - Allez dans SSL/TLS → Overview
   - Choisissez "Full" ou "Full (strict)"

5. **Ajouter les règles de sécurité** dans Cloudflare:

   **Option A: Page Rules (Simple)**
   - Allez dans "Rules" → "Page Rules"
   - Créez une règle pour `https://legare.ca/*`
   - Activez "Security Level: High"

   **Option B: Workers (Avancé - Meilleur contrôle)**
   - Allez dans "Workers & Pages" → "Create application"
   - Choisissez "Create Worker"
   - Copiez le contenu de `cloudflare-worker.js` dans l'éditeur
   - Déployez le Worker
   - Ajoutez une route: `https://legare.ca/*` → Votre Worker

6. **Activer HSTS** dans Cloudflare:
   - Allez dans SSL/TLS → Edge Certificates
   - Activez "Always Use HTTPS"
   - Activez "HSTS" avec ces paramètres:
	 - Max Age: 12 months (31536000)
	 - Include subdomains: Oui
	 - Preload: Oui

#### Résultat attendu avec Cloudflare: **Note A+**

---

### **Solution 2: Meta Tags HTTP-Equiv (Partiel - Déjà appliqué)**

J'ai ajouté des meta tags de sécurité à toutes vos pages HTML:
- `index.html`
- `about.html`
- `services.html`
- `contact.html`
- `privacy.html`

Ces meta tags ajoutent:
- ✅ Content-Security-Policy
- ✅ X-Content-Type-Options
- ✅ X-Frame-Options
- ✅ Referrer-Policy

⚠️ **Limitation**: Les meta tags ne peuvent pas ajouter:
- ❌ Strict-Transport-Security (HSTS)
- ❌ Permissions-Policy
- ❌ Certaines protections avancées

**Résultat attendu**: Amélioration de F à **C ou D** (mais pas A+)

---

### **Solution 3: Migration vers Cloudflare Pages** (Alternative complète)

Si vous voulez un contrôle total, migrez de GitHub Pages à Cloudflare Pages:

1. Allez sur https://pages.cloudflare.com/
2. Connectez votre repository GitHub
3. Créez un fichier `_headers` à la racine (déjà créé dans votre projet)
4. Déployez

Le fichier `_headers` sera automatiquement respecté sur Cloudflare Pages.

**Résultat attendu**: **Note A+**

---

## 📋 Fichiers créés/modifiés

### Fichiers créés:
- ✅ `_headers` - Headers de sécurité pour Cloudflare Pages
- ✅ `cloudflare-worker.js` - Worker Cloudflare pour GitHub Pages
- ✅ `SECURITY-HEADERS.md` - Ce fichier d'instructions

### Fichiers modifiés:
- ✅ `index.html` - Ajout de meta tags de sécurité
- ✅ `about.html` - Ajout de meta tags de sécurité
- ✅ `services.html` - Ajout de meta tags de sécurité
- ✅ `contact.html` - Ajout de meta tags de sécurité
- ✅ `privacy.html` - Ajout de meta tags de sécurité

### Fichiers obsolètes (ne fonctionnent pas sur GitHub Pages):
- ⚠️ `.htaccess` - Fonctionne uniquement sur Apache (pas GitHub Pages)
- ⚠️ `security-headers.php` - Fonctionne uniquement sur serveurs PHP

---

## 🎯 Recommandation

**Pour obtenir un A+, utilisez la Solution 1 (Cloudflare):**

1. Créez un compte Cloudflare gratuit
2. Ajoutez votre domaine legare.ca
3. Configurez les nameservers
4. Activez HSTS et les Workers (ou utilisez les Page Rules)
5. Testez votre site sur https://securityheaders.com/

**Temps estimé**: 15-30 minutes

---

## 🧪 Tester votre configuration

Après avoir appliqué une solution, testez votre site:

- https://securityheaders.com/?q=https://legare.ca/
- https://observatory.mozilla.org/analyze/legare.ca

Objectif: **Grade A+**

---

## 📚 Headers de sécurité expliqués

| Header | Fonction | Impact |
|--------|----------|--------|
| **Strict-Transport-Security** | Force HTTPS pour 1 an | Empêche les attaques man-in-the-middle |
| **Content-Security-Policy** | Contrôle les ressources chargées | Empêche XSS et injection de code |
| **X-Frame-Options** | Empêche l'intégration en iframe | Protège contre le clickjacking |
| **X-Content-Type-Options** | Empêche le MIME sniffing | Protège contre l'exécution de code |
| **Referrer-Policy** | Contrôle les informations de référence | Protège la vie privée |
| **Permissions-Policy** | Contrôle les APIs du navigateur | Limite l'accès aux fonctionnalités sensibles |

---

## 🆘 Besoin d'aide?

Si vous rencontrez des problèmes:
1. Vérifiez que HTTPS est activé sur votre site
2. Assurez-vous que Cloudflare est correctement configuré
3. Vérifiez les DNS avec https://dnschecker.org/
4. Consultez la documentation Cloudflare: https://developers.cloudflare.com/

---

**Note**: Les meta tags HTTP ont été ajoutés à toutes vos pages HTML, ce qui améliorera légèrement votre note. Mais pour atteindre A+, **Cloudflare est obligatoire** car GitHub Pages ne supporte pas HSTS nativement.
