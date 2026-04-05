# Analyse Statique d'une Application Android (APK) avec MobSF

## 1. Présentation du laboratoire

Ce laboratoire a pour but d'effectuer une **analyse statique de sécurité** sur une application Android au format APK, en utilisant **MobSF (Mobile Security Framework)** comme outil principal.

L'analyse statique permet d'inspecter en profondeur une application — sa structure interne, ses fichiers de configuration, ses permissions déclarées et son code source décompilé — **sans avoir besoin de l'exécuter**. Cette méthode est particulièrement efficace pour détecter des failles de sécurité dès la phase de développement.

Les apports de cette démarche sont multiples :

- Comprendre comment est structurée une application Android
- Repérer des erreurs de configuration qui exposent des risques de sécurité
- Cartographier les vecteurs d'attaque possibles
- Rédiger un **rapport d'audit de sécurité mobile synthétique**

---

## 2. Objectifs pédagogiques

À l'issue de ce laboratoire, l'étudiant sera capable de :

- Maîtriser le déroulement d'une analyse statique sur un APK
- Identifier les composants critiques d'une application Android
- Lire et interpréter les résultats générés par MobSF
- Reconnaître les types de vulnérabilités fréquemment rencontrées dans les applications mobiles Android
- Mapper les failles découvertes aux exigences du standard **OWASP MASVS**
- Formuler des recommandations de sécurité concrètes et exploitables
- Produire un rapport d'audit structuré à l'image d'un professionnel

---

## 2.1 Résumé exécutif

L'analyse de l'application **PizzaRecipes** révèle un niveau de risque global **moyen**.

Les principaux problèmes détectés sont les suivants :

- Mode debug laissé actif dans le build
- Signature de l'application avec un certificat de debug
- Sauvegarde automatique des données autorisée
- Présence de composants Android exportés accessibles depuis l'extérieur

Aucune faille critique liée au code applicatif ou aux communications réseau n'a été mise en évidence.

---

## 3. Périmètre et règles d'engagement

Ce laboratoire s'inscrit exclusivement dans un cadre **légal et pédagogique**.

**Périmètre autorisé :**

- L'APK fourni dans le cadre du cours
- Un APK produit par l'étudiant dans un projet de formation

**Périmètre interdit :**

- Toute application issue du Google Play Store
- Des applications à usage commercial ou propriétaire
- Des applications système Android

L'objectif de l'analyse se limite à :

- L'identification des risques et mauvaises configurations
- La formulation de recommandations correctives

Aucune exploitation active des vulnérabilités identifiées n'est réalisée dans le cadre de ce laboratoire.

---

## 4. Environnement technique

| Élément | Détail |
|---|---|
| Système d'exploitation | Kali Linux |
| Outil principal | MobSF |
| Version MobSF | 4.4.5 |
| Type d'analyse | Statique |
| Fichier APK | app-debug.apk |
| Application analysée | PizzaRecipes |
| Package Android | com.example.pizzarecipes |
| Empreinte SHA256 | 8fd353dea4e2821743ad976ba1a4bc2209a94d5298aa44cb2cc7ffb1c7d1a23 |

---

## 5. Tâche 1 — Mise en place de l'environnement d'analyse

### Création du répertoire de travail

Un dossier dédié est créé pour regrouper tous les artefacts de l'analyse et garantir une bonne traçabilité.

```bash
mkdir -p ~/apk_analysis/$(date +%Y-%m-%d)
cd ~/apk_analysis/$(date +%Y-%m-%d)
```

> 📸 **[Insérer ici une capture d'écran du terminal montrant la création du dossier de travail]**

<img width="632" height="134" alt="Screenshot 2026-04-05 181718" src="https://github.com/user-attachments/assets/9a847f12-1943-4471-bda2-f008b7642e3a" />

---

### Placement de l'APK dans le répertoire d'analyse

L'APK est copié dans le dossier de travail fraîchement créé pour centraliser les fichiers de l'audit.

> 📸 **[Insérer ici une capture montrant la présence de l'APK dans le répertoire]**

<img width="727" height="151" alt="Screenshot 2026-04-05 181725" src="https://github.com/user-attachments/assets/08f07b3b-bf51-489b-aab4-dec89befbaf3" />


---

### Vérification de l'intégrité du fichier

Le hash SHA256 de l'APK est calculé et enregistré. Cette étape permet de vérifier que le fichier n'a pas été altéré entre son origine et l'analyse.

```bash
sha256sum app-debug.apk > apk_hash.txt
cat apk_hash.txt
```

> 📸 **[Insérer ici une capture montrant la sortie de la commande sha256sum]**

<img width="724" height="128" alt="Screenshot 2026-04-05 181731" src="https://github.com/user-attachments/assets/542e878a-a2e5-4537-9943-7fef2732dd4d" />


---

### Vérification de la taille du fichier

```bash
ls -lh app-debug.apk
```



<img width="489" height="63" alt="Screenshot 2026-04-05 181737" src="https://github.com/user-attachments/assets/5624300a-b2c0-4781-9054-9969d99e1758" />


### Génération du fichier de traçabilité

Un fichier texte récapitulatif est créé automatiquement. Il centralise les informations clés de la session d'analyse pour documenter l'audit.

```bash
echo "Date d'analyse : $(date)" > analyse_info.txt
echo "Analyste : Wassim Gharbaoui" >> analyse_info.txt
echo "VM : Kali Linux" >> analyse_info.txt
echo "APK analysé : app-debug.apk" >> analyse_info.txt
cat apk_hash.txt >> analyse_info.txt
```

<img width="760" height="261" alt="Screenshot 2026-04-05 181742" src="https://github.com/user-attachments/assets/cd25fc9f-0acd-4ca7-a6cb-c66e2696c1d3" />


## 6. Tâche 2 — Démarrage de MobSF

MobSF est lancé depuis le terminal Kali. Il démarre un serveur web local accessible via un navigateur.

```bash
cd ~/Mobile-Security-Framework-MobSF
source .venv/bin/activate
./run.sh
```

Une fois démarré, MobSF est accessible à l'adresse suivante :

```
http://127.0.0.1:8000
```

<img width="797" height="286" alt="Screenshot 2026-04-05 181749" src="https://github.com/user-attachments/assets/38e0f333-f6dc-48b1-9ae5-d99b3fc36728" />


---

### Interface web MobSF

L'interface graphique de MobSF permet d'importer un APK et de déclencher l'analyse depuis un navigateur.


<img width="1009" height="439" alt="Screenshot 2026-04-05 181758" src="https://github.com/user-attachments/assets/84a781b5-0b43-4dc9-b9e9-8e5a82fadad9" />

---

## 7. Tâche 3 — Chargement et analyse de l'APK

L'APK est importé dans MobSF pour lancer le processus d'analyse automatique.

**Procédure :**

1. Cliquer sur **Upload & Analyze**
2. Sélectionner le fichier `app-debug.apk`
3. Attendre la fin du traitement (environ 2 minutes)

Durant cette phase, MobSF effectue automatiquement :

- La décompilation de l'APK
- L'analyse du fichier AndroidManifest.xml
- L'extraction et la classification des permissions
- La détection des vulnérabilités connues

---

### Résultats synthétiques de l'analyse

| Élément | Valeur |
|---|---|
| Score de sécurité | 36 / 100 |
| Application | PizzaRecipes |
| Package | com.example.pizzarecipes |
| Version | 1.0 |
| SDK minimum | 24 |
| SDK cible | 36 |

<img width="1078" height="133" alt="Screenshot 2026-04-05 181806" src="https://github.com/user-attachments/assets/171cde6b-1809-4224-ab03-a0134a7db64a" />

---

## 8. Tâche 4 — Analyse du manifeste Android

Le fichier `AndroidManifest.xml` est la pièce centrale de toute application Android. Il déclare les composants, les permissions et les configurations de sécurité de l'application.

Lors de cette analyse, deux paramètres problématiques ont été identifiés :

```xml
android:debuggable="true"
android:allowBackup="true"
```

Ces deux attributs indiquent que l'APK analysé est un **build de développement** non sécurisé pour une mise en production.

---

### Composants exportés — Activities

| Activity | Exportée |
|---|---|
| SplashActivity | Oui |
| ListPizzaActivity | Non |
| PizzaDetailActivity | Non |

`SplashActivity` est exportée car elle déclare les intent-filters `MAIN` et `LAUNCHER`, ce qui en fait le point d'entrée officiel de l'application. Cela est normal, mais doit être documenté dans la surface d'attaque.

<img width="1871" height="216" alt="Screenshot 2026-04-05 181836" src="https://github.com/user-attachments/assets/50d054e4-3c53-405f-b599-3532aaa8d5ee" />


---

### Composants exportés — Broadcast Receivers

| Receiver | Exporté |
|---|---|
| ProfileInstallReceiver | Oui |

Le composant `androidx.profileinstaller.ProfileInstallReceiver` est exporté. Bien qu'il soit d'origine bibliothèque, il contribue à élargir la surface d'attaque exposée.

<img width="1169" height="179" alt="Screenshot 2026-04-05 181849" src="https://github.com/user-attachments/assets/6fcec539-bbb7-4648-b911-8487dd396455" />


### Providers

| Provider | Exporté |
|---|---|
| InitializationProvider | Non |

---

### Vue globale du manifeste

<img width="1897" height="555" alt="Screenshot 2026-04-05 181912" src="https://github.com/user-attachments/assets/0378661d-2edd-4e36-a9cb-3134674d17c4" />

---

## 9. Tâche 5 — Analyse des permissions

MobSF confirme que l'application ne demande **aucune permission Android dangereuse** (telles que la localisation, les contacts ou la caméra).

Permission déclarée :

```
com.example.pizzarecipes.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION
```

Cette permission personnalisée est définie avec un niveau de protection `signature`, ce qui signifie que seules les applications signées avec la même clé développeur peuvent l'utiliser. Ce mécanisme est approprié.

<img width="1493" height="194" alt="Screenshot 2026-04-05 181925" src="https://github.com/user-attachments/assets/b5b0afd7-5409-48b9-a968-647ab6faf753" />


---

## 9.1 Configuration réseau

<img width="1486" height="190" alt="Screenshot 2026-04-05 181932" src="https://github.com/user-attachments/assets/1ed745bb-7655-47af-bd0e-c51b79d24147" />

| Paramètre | Valeur observée |
|---|---|
| android:usesCleartextTraffic | Non défini |
| networkSecurityConfig | Absent |

L'absence de `usesCleartextTraffic` signifie que l'application ne force pas le trafic HTTP en clair. Sur Android 9 (API 28) et supérieur, le comportement par défaut impose uniquement le trafic **HTTPS**, ce qui est une bonne pratique.

---

### Analyse des URLs et endpoints

<img width="1472" height="106" alt="Screenshot 2026-04-05 181943" src="https://github.com/user-attachments/assets/d9b2790c-4533-4b10-af06-b8ff7cbe2547" />

MobSF n'a détecté aucun élément réseau sensible :

- Aucune URL codée en dur
- Aucun endpoint distant
- Aucune adresse email exposée

**Conclusion réseau :** l'application ne présente pas de risque lié aux communications réseau.

---

## 10. Tâche 6 — Analyse du code source et des ressources

L'examen du code décompilé et des fichiers de ressources n'a révélé aucune donnée sensible exposée :

- Aucune clé API ou token secret hardcodé
- Aucune URL interne exposée
- Aucun endpoint critique détectable
- Aucune bibliothèque native vulnérable identifiée

Les sections MobSF suivantes sont revenues sans alerte :

- Code Analysis
- Shared Library Analysis
- File Analysis
- Firebase Analysis

<img width="1369" height="175" alt="Screenshot 2026-04-05 181953" src="https://github.com/user-attachments/assets/dd016785-fa6c-46bc-8277-4c3be2abadeb" />


---

## 10.1 Surface d'attaque identifiée

Les points d'entrée exploitables potentiels recensés sont :

- `SplashActivity` accessible depuis d'autres applications (composant exporté)
- `ProfileInstallReceiver` exposé sans restriction applicative forte
- Mode debug activé, permettant l'attachement d'un débogueur externe
- Sauvegarde ADB activée, permettant l'extraction des données stockées

---

## 11. Tâche 7 — Correspondance avec OWASP MASVS

<img width="1475" height="403" alt="Screenshot 2026-04-05 182002" src="https://github.com/user-attachments/assets/81c5be47-1ef0-45af-bb43-6ef2090dc6ee" />


Les vulnérabilités détectées ont été mises en correspondance avec le standard **OWASP Mobile Application Security Verification Standard (MASVS)** :

### Mode debug activé

Catégorie MASVS : **MASVS-RESILIENCE**

Les builds destinés à la production ne doivent en aucun cas être compilés avec le mode debug actif. Cela expose l'application à des attaques par débogueur.

---

### Certificat de debug

Catégorie MASVS : **MASVS-CODE**

Une application distribuée doit impérativement être signée avec un certificat de production délivré par le développeur. L'usage d'un certificat de debug facilite la modification non autorisée du binaire.

### Tests MASTG complémentaires recommandés

- **MASTG-TEST-0011** : Vérification de l'état du mode debug
- **MASTG-TEST-0024** : Audit des composants exportés

---

## 12. Tâche 8 — Export du rapport MobSF

MobSF permet de générer et d'exporter un rapport PDF complet directement depuis son interface.

**Depuis l'interface web :**

```
Generate PDF Report
```

Le rapport exporté comprend :

- Le score de sécurité global
- La liste des vulnérabilités avec niveau de sévérité
- L'analyse détaillée du manifeste
- L'analyse du certificat de signature
- Des recommandations de remédiation automatiques

---

## 12.1 Analyse des faux positifs

Aucun faux positif n'a été identifié dans les résultats retournés par MobSF. L'ensemble des alertes générées est cohérent avec les paramètres effectivement présents dans le manifeste de l'application.

---

## 13. Vulnérabilités identifiées

### 1. Application signée avec un certificat de debug

**Sévérité :** HIGH

L'application utilise le certificat Android Debug Key Store par défaut au lieu d'un certificat de production.

**Risques associés :**

- Modification du binaire par un attaquant sans invalidation de la signature
- Reverse engineering facilité
- Signature triviale à reproduire

**Remédiation :** Signer l'application avec un certificat de production issu d'un keystore sécurisé avant toute distribution.

---

### 2. Mode debug activé

**Sévérité :** HIGH

**Preuve dans le manifeste :**

```xml
android:debuggable="true"
```

**Risque :** Un attaquant peut attacher un débogueur (via `adb`) à l'application en cours d'exécution et inspecter son état mémoire ou manipuler son comportement.

**Remédiation :**

```xml
android:debuggable="false"
```

---

### 3. Sauvegarde des données activée

**Sévérité :** WARNING

**Preuve dans le manifeste :**

```xml
android:allowBackup="true"
```

**Risque :** Les données privées de l'application peuvent être extraites via `adb backup` sans root sur des appareils Android, ce qui compromet la confidentialité des données utilisateur.

**Remédiation :**

```xml
android:allowBackup="false"
```

---

### 4. Broadcast Receiver exporté

**Sévérité :** WARNING

Le composant `ProfileInstallReceiver` est exposé à d'autres applications installées sur l'appareil. Bien qu'il soit protégé par une permission système, sa présence dans la surface d'attaque doit être documentée.

**Risque :** Surface d'attaque exposée bien que l'accès reste limité aux applications autorisées.

**Remédiation :** Restreindre l'export du receiver ou renforcer les contrôles d'accès via des permissions plus strictes.

---

## 14. Points positifs de l'application

Malgré les failles de configuration, plusieurs bonnes pratiques ont été observées :

- Aucune permission Android dangereuse demandée
- Absence totale de secrets ou de clés API hardcodés
- Aucun endpoint réseau exposé dans le code
- Aucune donnée sensible stockée de manière non sécurisée
- Aucune vulnérabilité dans les bibliothèques natives

---

## 15. Évaluation du risque global

| Indicateur | Valeur |
|---|---|
| Score MobSF | 36 / 100 |
| Niveau de risque estimé | **MEDIUM** |

**Justification :** Les failles identifiées sont principalement des erreurs de configuration typiques d'un environnement de développement. Elles ne constituent pas de vulnérabilités critiques dans le code fonctionnel de l'application, mais doivent absolument être corrigées avant toute mise en production.

---

## 16. Recommandations de remédiation

1. **Désactiver le mode debug** en définissant `android:debuggable="false"` dans le manifeste de production
2. **Utiliser un certificat de production** signé avec un keystore sécurisé et stocké de manière confidentielle
3. **Désactiver la sauvegarde ADB** via `android:allowBackup="false"` pour protéger les données utilisateur
4. **Restreindre les composants exportés** en ajoutant des permissions ou en supprimant l'attribut `exported` lorsque cela est possible
5. **Augmenter le SDK minimum supporté** pour bénéficier des protections de sécurité intégrées aux versions Android récentes

---

## 17. Conclusion

L'analyse statique de l'application **PizzaRecipes** met en évidence plusieurs faiblesses de configuration caractéristiques d'un build de développement. Ces problèmes, bien que non critiques sur le plan fonctionnel, représentent des risques réels si l'application venait à être distribuée en l'état.

La correction des points identifiés permettrait de réduire significativement la surface d'attaque et d'amener le score de sécurité MobSF à un niveau acceptable pour une mise en production.

---

## 18. Outils utilisés

- **MobSF** — Mobile Security Framework (analyse statique automatisée)
- **Kali Linux** — Environnement d'analyse de sécurité
- **Android SDK** — Outils de manipulation d'APK
- **OWASP MASVS** — Standard de référence pour la sécurité des applications mobiles

---

## 19. Annexes

### Permissions déclarées

- `com.example.pizzarecipes.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`

### Composants exportés

- `SplashActivity`
- `ProfileInstallReceiver`

### Endpoints détectés

Aucun endpoint, URL ou adresse email détecté dans l'application.

---

## Fichiers de traçabilité générés

Les fichiers suivants ont été produits durant l'analyse pour documenter chaque étape :

| Fichier | Contenu |
|---|---|
| `analyse_info.txt` | Métadonnées de la session d'analyse |
| `permissions.txt` | Liste des permissions déclarées |
| `composants_exportes.txt` | Composants Android exposés |
| `config_reseau.txt` | Paramètres de configuration réseau |
| `vulnerabilites.txt` | Vulnérabilités identifiées |
| `ressources_sensibles.txt` | Ressources potentiellement sensibles |
| `correlation_masvs.txt` | Correspondance avec OWASP MASVS |
| `top_vulnerabilites.txt` | Classement des vulnérabilités par criticité |
| `Static Analysis.pdf` | Rapport PDF exporté depuis MobSF |

---

*Rapport rédigé par **Wassim Gharbaoui** — Analyse statique de sécurité mobile*
