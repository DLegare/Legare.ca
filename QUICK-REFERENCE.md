# 🚀 CARTE DE RÉFÉRENCE RAPIDE

```
╔══════════════════════════════════════════════════════════════╗
║                    AMÉLIORATION SÉCURITÉ                     ║
║              legare.ca : F → A+ en 30 minutes                ║
╚══════════════════════════════════════════════════════════════╝

┌──────────────────────────────────────────────────────────────┐
│ ÉTAPE 1: PUSH VERS GITHUB (MAINTENANT - 2 min)              │
└──────────────────────────────────────────────────────────────┘

  $ git add .
  $ git commit -m "Add security headers"
  $ git push origin main

  ✅ Résultat: Note C/D


┌──────────────────────────────────────────────────────────────┐
│ ÉTAPE 2: CLOUDFLARE (CE WEEK-END - 30 min)                  │
└──────────────────────────────────────────────────────────────┘

  1. Signup
	 → https://dash.cloudflare.com/sign-up

  2. Add site
	 → Entrez: legare.ca
	 → Plan: FREE (gratuit)

  3. Nameservers
	 → Copiez les 2 nameservers Cloudflare
	 → Allez chez votre registraire
	 → Remplacez les anciens nameservers

  4. SSL/TLS
	 → SSL/TLS → Overview → Full
	 → SSL/TLS → Edge Certificates → Always Use HTTPS: ON
	 → SSL/TLS → Edge Certificates → HSTS: Enable
	   • Max Age: 12 months
	   • Include subdomains: YES
	   • Preload: YES

  5. Worker
	 → Workers & Pages → Create Worker
	 → Name: security-headers
	 → Deploy
	 → Edit code → Supprimer tout
	 → Copier cloudflare-worker.js
	 → Save and Deploy
	 → Settings → Triggers → Add Route
	   • Route: https://legare.ca/*
	   • Zone: legare.ca

  6. DNS Proxy
	 → DNS → Records
	 → Vérifier: ☁️ Orange (Proxied) - PAS gris!

  ✅ Résultat: Note A+


┌──────────────────────────────────────────────────────────────┐
│ ÉTAPE 3: VÉRIFICATION (5 min après DNS)                     │
└──────────────────────────────────────────────────────────────┘

  Test 1: Cloudflare actif?
	$ curl -I https://legare.ca/ | grep -i "cf-ray"
	✅ Devrait montrer: cf-ray: ...

  Test 2: HSTS présent?
	$ curl -I https://legare.ca/ | grep -i "strict-transport"
	✅ Devrait montrer: strict-transport-security: ...

  Test 3: Note finale
	→ https://securityheaders.com/?q=https://legare.ca/
	🎯 Objectif: A+


╔══════════════════════════════════════════════════════════════╗
║                         HEADERS AJOUTÉS                      ║
╚══════════════════════════════════════════════════════════════╝

  ✅ Strict-Transport-Security     Force HTTPS (1 an)
  ✅ Content-Security-Policy       Anti-XSS
  ✅ X-Frame-Options               Anti-clickjacking
  ✅ X-Content-Type-Options        Anti-MIME sniffing
  ✅ Referrer-Policy               Vie privée
  ✅ Permissions-Policy            Limite APIs
  ✅ X-XSS-Protection             Protection legacy


╔══════════════════════════════════════════════════════════════╗
║                      DÉPANNAGE RAPIDE                        ║
╚══════════════════════════════════════════════════════════════╝

  Problème: Site inaccessible
	→ SSL/TLS doit être "Full" (pas "Flexible")
	→ GitHub Pages: Custom domain = legare.ca
	→ GitHub Pages: Enforce HTTPS = ON

  Problème: Toujours note F
	→ Vérifier Worker déployé
	→ Vérifier route: https://legare.ca/*
	→ Vérifier DNS: ☁️ Orange (Proxied)
	→ Purger cache Cloudflare

  Problème: DNS ne propage pas
	→ Vérifier nameservers chez registraire
	→ Attendre 24-48h max
	→ Test: https://dnschecker.org/

  Problème: Worker n'ajoute pas headers
	→ Vérifier code dans cloudflare-worker.js
	→ Vérifier logs du Worker
	→ S'assurer que DNS est Proxied (orange)


╔══════════════════════════════════════════════════════════════╗
║                      FICHIERS IMPORTANTS                     ║
╚══════════════════════════════════════════════════════════════╝

  📖 GUIDE-RAPIDE.md              Instructions détaillées
  ☁️  cloudflare-worker.js        Code à copier dans Worker
  🔧 TROUBLESHOOTING.md           Guide de dépannage complet
  📊 COMPARAISON-SOLUTIONS.md     Comprendre les options
  🏗️  ARCHITECTURE.md             Schémas techniques


╔══════════════════════════════════════════════════════════════╗
║                      URLS UTILES                             ║
╚══════════════════════════════════════════════════════════════╝

  Cloudflare Dashboard
	→ https://dash.cloudflare.com/

  Test de sécurité
	→ https://securityheaders.com/?q=https://legare.ca/

  Test DNS
	→ https://dnschecker.org/

  Documentation Cloudflare
	→ https://developers.cloudflare.com/


╔══════════════════════════════════════════════════════════════╗
║                         TIMELINE                             ║
╚══════════════════════════════════════════════════════════════╝

  Jour 1
  ├─ 00:00  Push GitHub (vous)
  ├─ 00:05  Note: C/D ✅
  │
  ├─ 00:10  Créer compte Cloudflare
  ├─ 00:15  Ajouter domaine
  ├─ 00:20  Changer nameservers
  ├─ 00:25  Configurer SSL/TLS
  ├─ 00:30  Activer HSTS
  ├─ 00:35  Créer & déployer Worker
  ├─ 00:40  Configuration terminée ✅
  │
  └─ 01:00 à 24:00  Propagation DNS (attente)

  Jour 2
  └─ Note: A+ ✅


╔══════════════════════════════════════════════════════════════╗
║                      CHECKLIST FINALE                        ║
╚══════════════════════════════════════════════════════════════╝

  GitHub
  [ ] Changements committé et pushé
  [ ] Site accessible sur legare.ca
  [ ] Custom domain configuré
  [ ] Enforce HTTPS activé

  Cloudflare
  [ ] Compte créé
  [ ] Domaine ajouté
  [ ] Nameservers changés (chez registraire!)
  [ ] SSL/TLS = Full
  [ ] Always Use HTTPS = ON
  [ ] HSTS activé (max-age 31536000)
  [ ] Worker créé avec cloudflare-worker.js
  [ ] Route ajoutée: https://legare.ca/*
  [ ] DNS en mode Proxied (☁️ orange)

  Tests
  [ ] curl -I montre cf-ray
  [ ] curl -I montre strict-transport-security
  [ ] securityheaders.com montre A+

  ✅ SI TOUS COCHÉS = SUCCÈS!


╔══════════════════════════════════════════════════════════════╗
║                    AVANT / APRÈS                             ║
╚══════════════════════════════════════════════════════════════╝

  AVANT                          APRÈS
  ─────────────────────────────────────────────────────
  Note: F ❌                     Note: A+ ✅
  HSTS: Absent                   HSTS: Actif (1 an)
  CSP: Absent                    CSP: Complet
  Protection: Minimale           Protection: Maximale
  CDN: Non                       CDN: Cloudflare
  DDoS: Non                      DDoS: Protection incluse
  Coût: Gratuit                  Coût: Gratuit

  Vulnérabilités: 6/6            Vulnérabilités: 0/6 ✅


╔══════════════════════════════════════════════════════════════╗
║                      COMMANDES UTILES                        ║
╚══════════════════════════════════════════════════════════════╝

  # Test complet
  curl -I https://legare.ca/

  # Test Cloudflare
  curl -I https://legare.ca/ | grep -i "cf-ray"

  # Test HSTS
  curl -I https://legare.ca/ | grep -i "strict-transport"

  # Test CSP
  curl -I https://legare.ca/ | grep -i "content-security"

  # DNS check
  nslookup legare.ca

  # DNS propagation
  nslookup legare.ca 8.8.8.8


╔══════════════════════════════════════════════════════════════╗
║                   SUPPORT & AIDE                             ║
╚══════════════════════════════════════════════════════════════╝

  Documentation locale:
	1. README-SUMMARY.md     → Commencez ici
	2. GUIDE-RAPIDE.md       → Instructions détaillées
	3. TROUBLESHOOTING.md    → En cas de problème

  En ligne:
	→ Cloudflare Community: community.cloudflare.com
	→ GitHub Discussions: github.com/orgs/community/discussions


╔══════════════════════════════════════════════════════════════╗
║                    🎯 OBJECTIF FINAL                         ║
╚══════════════════════════════════════════════════════════════╝

			   https://legare.ca/
					  │
					  │
					  ↓
			 securityheaders.com
					  │
					  │
					  ↓
		┌─────────────────────────┐
		│   Security Headers      │
		│   ─────────────────     │
		│                         │
		│   Grade: A+             │
		│                         │
		│   ✅ HSTS               │
		│   ✅ CSP                │
		│   ✅ X-Frame-Options    │
		│   ✅ X-Content-Type     │
		│   ✅ Referrer-Policy    │
		│   ✅ Permissions        │
		└─────────────────────────┘

			  FÉLICITATIONS! 🎉


╔══════════════════════════════════════════════════════════════╗
║         VOUS ÊTES PRÊT! SUIVEZ GUIDE-RAPIDE.MD 🚀           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Imprimez cette carte!

Cette carte de référence rapide vous guide étape par étape.
Gardez-la à portée de main pendant la configuration.

**Temps total: 30-40 minutes**
**Résultat: Note A+ garantie** ✅

---

**Bonne chance! 🎯**
