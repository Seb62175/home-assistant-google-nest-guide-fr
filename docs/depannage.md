# Dépannage Google Nest / Home Assistant

Retour d'expérience du 28 octobre 2026. Les messages sont reproduits pour faciliter les recherches.

## 1. `redirect_uri_mismatch` (Google OAuth, erreur 400)

**Symptôme :** l'écran Google refuse la connexion.

**Vérification :** dans le client OAuth de type **Application Web**, comparer les **URI de redirection autorisés** à l'URI mentionnée dans l'erreur. La correspondance doit être exacte (schéma HTTPS, domaine, chemin, barre oblique).

L'URI habituelle de Home Assistant est `https://my.home-assistant.io/redirect/oauth`. Une URL de rappel propre à l'instance peut être utilisée selon le parcours d'authentification.

**Ne pas** créer de nouveaux projets à chaque tentative.

## 2. `Error listing eligible Pub/Sub topics: Error while listing existing cloud console topics`

**Symptôme :** Home Assistant ne parvient pas à énumérer les sujets.

**Vérifications :**

- API **Cloud Pub/Sub** activée dans le bon projet Google Cloud ;
- ID technique du projet correct ;
- compte Google autorisé à accéder au projet Cloud ;
- sujet créé dans le projet attendu ;
- journal détaillé : chercher notamment `403`, `404`, `PERMISSION_DENIED`, `SERVICE_DISABLED`.

Cette erreur a été rencontrée pendant notre configuration initiale. Sa présence seule ne permet pas de conclure à une cause unique.

## 3. « Le thème Pub/Sub est introuvable »

**Cause constatée :** le sujet Device Access pointait vers **un autre projet Google Cloud** que celui contenant le sujet créé.

**Correction :** utiliser le nom complet du sujet :

```text
projects/ID_DU_PROJET_CLOUD/topics/NOM_DU_SUJET
```

Vérifier le statut **Enabled** dans Device Access et le rôle **Pub/Sub Publisher** pour `sdm-publisher@googlegroups.com`.

## 4. « Activer les événements »

Home Assistant demande de configurer le sujet Pub/Sub dans Device Access.

**Correction :** vérifier que le projet Device Access référence le bon sujet et que les événements sont activés. Si le statut est déjà **Enabled**, ne pas créer de sujet supplémentaire : poursuivre le diagnostic.

## 5. « Authentification réussie », mais « Échec de la configuration »

Une authentification OAuth réussie ne signifie pas que l'API SDM est disponible.

Dans notre cas, après la connexion réussie, Home Assistant affichait :

> Échec de la configuration, nouvel essai : Erreur de communication avec l'API Device Access, veuillez consulter les journaux pour plus de détails.

**Cause confirmée sur notre installation :** **Smart Device Management API désactivée** dans le projet Google Cloud.

**Solution qui a fonctionné :**

1. Ouvrir [Smart Device Management API](https://console.cloud.google.com/apis/library/smartdevicemanagement.googleapis.com) en sélectionnant le projet Cloud utilisé.
2. Cliquer sur **Activer**.
3. Retourner dans Home Assistant et recharger l'intégration Nest.
4. Vérifier la découverte du thermostat.

**Résultat constaté :** apparition du thermostat et de ses 3 entités, disparition de l'erreur.

## 6. Les journaux Home Assistant ne montrent rien avec `nest`

La recherche dans les journaux peut être peu utile ou remonter des résultats non liés à Nest. Vérifier le message d'état de l'intégration dans **Paramètres → Appareils et services → Google Nest**.

Si besoin, activer temporairement la journalisation de débogage, reproduire l'erreur, puis exporter les journaux **après anonymisation**.

## Fausses pistes rencontrées

- Recréer un compte de service Google Cloud : **non nécessaire par défaut** pour le flux OAuth documenté.
- Migrer immédiatement l'application Nest vers Google Home : **pas démontré nécessaire** pour résoudre notre erreur API.
- Attribuer l'erreur Nest aux avertissements MQTT, Roomba ou RFPlayer : **aucune preuve** de lien dans notre cas.
- Recréer sans cesse le sujet Pub/Sub alors qu'il est déjà **Enabled**.

## Checklist rapide

- [ ] Bon projet Google Cloud sélectionné
- [ ] Cloud Pub/Sub API activée
- [ ] **Smart Device Management API activée**
- [ ] Client OAuth Web et redirections exactes
- [ ] Projet Device Access et OAuth Client ID cohérents
- [ ] Sujet Pub/Sub présent, nom complet exact
- [ ] `sdm-publisher@googlegroups.com` possède Pub/Sub Publisher sur le sujet
- [ ] Device Access affiche **Enabled**
- [ ] Même compte Google pour l'accès aux appareils et l'autorisation OAuth
- [ ] Thermostat réellement détecté dans Home Assistant
