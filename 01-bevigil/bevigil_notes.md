# Notes d'analyse BeVigil — InsecureBankv2

**Cible:** com.android.insecurebankv2  
**Date analyse:** 2026-05-11  
**Analyste:** SALIHI Yassine  

---

## Ce qui est certain

- 4 Activities exportées et accessibles par d'autres apps (CWE-926)
- Algorithme MD5 utilisé dans plusieurs fichiers Google libs (CWE-327)
- Requête SQL non paramétrée détectée (CWE-89)
- Client HTTP non sécurisé `DefaultHttpClient()` utilisé dans 3 fichiers métier (CWE-757)
- Mode AES/CBC/PKCS5Padding utilisé → vulnérable au Padding Oracle (ChangePassword, DoTransfer)
- Désérialisation d'objet non sécurisée détectée (CWE-502)

## Ce qui est hypothèse

- Les Activities exportées (PostLogin, DoTransfer, etc.) sont probablement invocables sans authentification préalable
- Les communications HTTP vers le serveur bancaire transitent en clair (pas de TLS)
- La clé AES utilisée dans CryptoClass.java est possiblement hardcodée (à vérifier manuellement)

## Points d'intérêt

- `CryptoClass.java` : AES/CBC/PKCS5Padding utilisé deux fois → cible prioritaire pour analyse statique
- `DoTransfer.java` + `ChangePassword.java` : HTTP non sécurisé + activités exportées = double exposition
- `DoLogin.java` : DefaultHttpClient → credentials potentiellement transmis en clair

---

## Domaines et sous-domaines

- Non détectés dans les exports BeVigil (pas de section domains dans les CSV)

## Endpoints et APIs

- Non listés explicitement, mais impliqués par :
  - `DoLogin.java` → endpoint de login
  - `DoTransfer.java` → endpoint de transfert bancaire
  - `ChangePassword.java` → endpoint changement de mot de passe

## URLs HTTP/HTTPS

- Protocole HTTP non sécurisé confirmé via `DefaultHttpClient()` dans :
  - `sources/com/android/insecurebankv2/ChangePassword.java`
  - `sources/com/android/insecurebankv2/DoTransfer.java`
  - `sources/com/android/insecurebankv2/DoLogin.java`

## Emails et identifiants

- Aucun email détecté dans les exports

## Technologies détectées

- Google Mobile Ads SDK (com.google.android.gms.ads)
- Google Analytics SDK (com.google.android.gms.analytics)
- Google Tag Manager (com.google.android.gms.tagmanager)
- Cryptographie maison : `CryptoClass.java` avec AES/CBC

---

## Vulnérabilités — Tableau récapitulatif

| # | Sévérité | Règle                        | CWE     | Fichier(s) concerné(s)                          |
|---|----------|------------------------------|---------|-------------------------------------------------|
| 1 | Low      | Exported Activity            | CWE-926 | PostLogin, DoTransfer, ViewStatement, ChangePassword (Manifest) |
| 2 | Low      | Weak Crypto (MD5)            | CWE-327 | Google libs (zzbl, zzak, zzhl, zza)             |
| 3 | Low      | SQL non paramétrisée         | CWE-89  | zzj.java (Google Analytics)                     |
| 4 | Low      | Insecure HTTP Client         | CWE-757 | ChangePassword, DoTransfer, DoLogin             |
| 5 | Low      | CBC Padding Oracle           | N/A     | CryptoClass.java, zzar.java                     |
| 6 | Low      | Object Deserialization       | CWE-502 | zzw.java (Tag Manager)                          |

