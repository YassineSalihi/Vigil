# Rapport d'analyse de sécurité mobile

## A. Informations générales
- **Date**: 2026-05-11
- **Analyste**: SALIHI Yassine
- **Cible**: InsecureBankv2 (com.android.insecurebankv2)
- **Hash SHA256**: `1da8bf57d266109f9a07c01bf7111a1975ce01f190b9d914bcd3ae3dbef96f21`
- **Outils utilisés**: BeVigil (web), MobSF (local — remplace Yaazhini, incompatible Linux)
- **Environnement**: Arch Linux

---

## B. Résumé exécutif

L'analyse de l'application InsecureBankv2 a révélé 6 vulnérabilités, toutes classées
"low" par BeVigil, mais dont la combinaison représente un risque réel et exploitable.

Les points les plus critiques sont l'utilisation d'un client HTTP non sécurisé dans les
flux d'authentification et de transfert, combinée à des Activities Android exportées sans
contrôle d'accès. La classe CryptoClass.java implémente AES en mode CBC avec PKCS5Padding,
exposant l'application à une attaque Padding Oracle. Aucune vulnérabilité critique isolée,
mais la chaîne d'attaque HTTP clair + activité exportée + crypto faible constitue un
vecteur d'exploitation sérieux.

**Niveau de risque global : MOYEN**

---

## C. Top 5 constats

### 1. Client HTTP non sécurisé — FIND-001
- **Sévérité**: Medium (réévalué depuis Low)
- **Preuve**: `new DefaultHttpClient()` dans DoLogin.java, DoTransfer.java, ChangePassword.java
- **Impact**: Credentials et données bancaires transmis en clair, interceptables par MITM
- **Remédiation**: Remplacer par `HttpsURLConnection` ou OkHttp avec TLS 1.2+
- **Référence OWASP**: MASVS-NETWORK-1 / CWE-757

### 2. Activities exportées sans contrôle — FIND-002
- **Sévérité**: Medium (réévalué depuis Low)
- **Preuve**: PostLogin, DoTransfer, ViewStatement, ChangePassword dans AndroidManifest.xml
- **Impact**: Une app tierce peut invoquer directement ces écrans, contournant l'authentification
- **Remédiation**: Ajouter `android:exported="false"` ou restreindre avec des permissions custom
- **Référence OWASP**: MASVS-PLATFORM-1 / CWE-926

### 3. Padding Oracle (AES/CBC) — FIND-003
- **Sévérité**: Medium (réévalué depuis Low)
- **Preuve**: `Cipher.getInstance("AES/CBC/PKCS5Padding")` dans CryptoClass.java (x2)
- **Impact**: Déchiffrement possible des données chiffrées sans connaissance de la clé
- **Remédiation**: Migrer vers AES/GCM/NoPadding (chiffrement authentifié)
- **Référence OWASP**: MASVS-CRYPTO-1

### 4. Algorithme MD5 — FIND-004
- **Sévérité**: Low
- **Preuve**: `getInstance("MD5")` dans plusieurs fichiers Google libs (zzbl, zzak, zzhl, zza)
- **Impact**: Collision possible, intégrité des données non garantie
- **Remédiation**: Remplacer par SHA-256 minimum (libs tierces à mettre à jour)
- **Référence OWASP**: MASVS-CRYPTO-1 / CWE-327

### 5. Requête SQL non paramétrée — FIND-005
- **Sévérité**: Low
- **Preuve**: `rawQuery("SELECT * FROM " + str + " LIMIT 0", null)` dans zzj.java
- **Impact**: Injection SQL possible si `str` est contrôlable par l'utilisateur
- **Remédiation**: Utiliser des requêtes paramétrées / PreparedStatement
- **Référence OWASP**: MASVS-CODE-1 / CWE-89

---

## D. Faux positifs notables

- Vulnérabilités MD5, SQL et désérialisation dans les libs Google (gms) : code tiers,
  non contrôlé par le développeur de l'app. Risque réel limité dans ce contexte.
- `ObjectInputStream` dans zzw.java (Tag Manager) : usage interne Google SDK,
  les données désérialisées ne sont pas directement contrôlables par un attaquant externe.

---

## E. Recommandations prioritaires

1. **Immédiat** — Remplacer tous les `DefaultHttpClient()` par une implémentation TLS
2. **Immédiat** — Ajouter `android:exported="false"` sur toutes les Activities non publiques
3. **Court terme** — Migrer CryptoClass.java de AES/CBC vers AES/GCM
4. **Court terme** — Mettre à jour les dépendances Google Play Services
5. **Long terme** — Audit complet du stockage local (SharedPreferences, SQLite)

---

## F. Annexes
- [Notes BeVigil](../01-bevigil/bevigil_notes.md)
- [Export manifest issues](../01-bevigil/bevigil_com.android.insecurebankv2_manifest_issues_1778518883800.csv)
- [Export vulnérabilités](../01-bevigil/bevigil_com.android.insecurebankv2_vuln_1778518876959.csv)
- [Triage complet](../03-triage/triage.csv)
