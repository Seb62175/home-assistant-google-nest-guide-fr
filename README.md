# Google Nest + Home Assistant — Guide français (2026)

Guide communautaire de configuration de l'intégration officielle **Google Nest** dans **Home Assistant**, basé sur une installation réelle finalisée le **28 octobre 2026**.

> **Le piège qui nous a bloqués :** activer **Cloud Pub/Sub API** ne suffit pas. Il faut **également activer Smart Device Management API** dans le **même projet Google Cloud**. Sans cette dernière, l'authentification OAuth peut réussir, tandis que Home Assistant affiche « Erreur de communication avec l'API Device Access ».

## Résultat obtenu

- Authentification Google réussie
- Thermostat Nest découvert dans Home Assistant
- 1 appareil et 3 entités affichés dans l'installation testée
- Configuration finale fonctionnelle

## Prérequis

- Home Assistant fonctionnel et accessible via une URL HTTPS pour le parcours d'authentification
- Thermostat Nest déjà configuré et visible dans l'application Nest / Google Home
- Compte Google ayant accès au thermostat
- Accès à Google Cloud Console et à [Google Device Access](https://console.nest.google.com/device-access)
- Projet Device Access éligible (des frais d'inscription peuvent s'appliquer selon les conditions de Google)

La configuration exacte des écrans peut évoluer. Consulter en priorité la [documentation officielle Home Assistant](https://www.home-assistant.io/integrations/nest/).

## 1. Créer un projet Google Cloud

Dans [Google Cloud Console](https://console.cloud.google.com/), créer un projet, par exemple `HA Nest`. **Noter son ID technique** (exemple fictif : `ha-nest-123456`) : il ne faut pas confondre le nom affiché et l'ID.

## 2. Activer les DEUX API indispensables

Dans **API et services → Bibliothèque**, en ayant sélectionné le projet :

1. [Cloud Pub/Sub API](https://console.cloud.google.com/apis/library/pubsub.googleapis.com) — cliquer sur **Activer**.
2. [Smart Device Management API](https://console.cloud.google.com/apis/library/smartdevicemanagement.googleapis.com) — cliquer sur **Activer**.

Pour chacune, vérifier que la console affiche **API activée** ou **Gérer**.

**Important :** c'est l'oubli de la seconde API qui a provoqué l'échec final dans notre cas.

## 3. Configurer OAuth

Dans **Google Auth Platform**, configurer l'écran de consentement et un **client OAuth de type Application Web** conformément à la [procédure officielle](https://www.home-assistant.io/integrations/nest/).

Pour le flux Home Assistant, l'URI de redirection généralement utilisée est :

```text
https://my.home-assistant.io/redirect/oauth
```

Selon la méthode de connexion et la version de Home Assistant, une URL de rappel externe peut aussi être requise : vérifier **l'URI exacte affichée dans le message `redirect_uri_mismatch`**, puis l'ajouter telle quelle à la liste autorisée dans Google Cloud.

Conserver le **Client ID** et le **Client Secret** dans un gestionnaire de secrets. **Ne jamais les publier sur GitHub.**

Si l'application OAuth est en mode **Test**, ajouter le compte Google concerné aux utilisateurs de test si nécessaire.

## 4. Créer le projet Google Device Access

Dans [Device Access Console](https://console.nest.google.com/device-access) :

1. Créer le projet Device Access.
2. Renseigner l'**OAuth Client ID** créé à l'étape précédente.
3. Conserver l'**ID de projet Device Access**, différent de l'ID Google Cloud.
4. Vérifier les autorisations demandées, notamment `https://www.googleapis.com/auth/sdm.service`.

## 5. Créer le sujet Cloud Pub/Sub

Dans Google Cloud → **Pub/Sub → Sujets**, créer par exemple `home-assistant-nest` dans le projet choisi.

Format du sujet :

```text
projects/ha-nest-123456/topics/home-assistant-nest
```

Dans les **autorisations IAM du sujet**, accorder au principal Google :

```text
sdm-publisher@googlegroups.com
```

le rôle **Pub/Sub Publisher** (Éditeur Pub/Sub), afin que Google Device Access puisse publier les événements.

Dans Device Access, renseigner **exactement** le nom complet du sujet et activer les événements. Vérifier le statut **Enabled**.

Home Assistant peut également créer ou gérer un abonnement Pub/Sub lors de sa configuration : suivre l'assistant et la documentation de la version installée. Éviter de multiplier les sujets et abonnements pendant le dépannage.

## 6. Ajouter Google Nest dans Home Assistant

Dans **Paramètres → Appareils et services → Ajouter une intégration → Google Nest** :

1. Saisir l'ID de projet **Device Access** lorsque demandé (pas l'ID Google Cloud).
2. Sélectionner ou renseigner les identifiants OAuth du bon projet.
3. Se connecter au compte Google ayant accès au thermostat et approuver les autorisations.
4. Configurer le sujet Pub/Sub proposé par l'assistant.
5. Terminer, puis attendre la découverte des appareils.

Une **authentification réussie** ne garantit pas à elle seule que l'intégration démarre : vérifier que l'appareil apparaît réellement dans Home Assistant.

## 7. Vérifications finales

- Google Nest n'est plus dans « Attention requise »
- Le thermostat apparaît dans les appareils de l'intégration
- Les entités sont disponibles et leurs valeurs se mettent à jour
- La consigne peut être modifiée depuis Home Assistant (si prise en charge par le modèle)
- La journalisation de débogage peut être désactivée après validation

## Dépannage

Voir [docs/depannage.md](docs/depannage.md) pour les messages d'erreur rencontrés et les vérifications ciblées.

## Sécurité

Voir [docs/securite.md](docs/securite.md) avant de publier des captures, journaux ou fichiers de configuration.

## Sources officielles

- [Intégration Nest — Home Assistant](https://www.home-assistant.io/integrations/nest/)
- [Google Device Access](https://developers.google.com/nest/device-access)
- [Google Cloud Pub/Sub](https://cloud.google.com/pubsub/docs)

---

**Statut :** retour d'expérience vérifié sur une installation fonctionnelle le 28 octobre 2026. Ce dépôt est un guide communautaire non officiel, sans affiliation avec Google ou Home Assistant.
