# Sécurité et confidentialité

Ce dépôt est public. Ne jamais y publier :

- **OAuth Client Secret**, jetons d'accès ou de rafraîchissement ;
- jeton **DuckDNS**, clés API ou identifiants Home Assistant ;
- fichiers `.storage/`, sauvegardes Home Assistant ou exports contenant des secrets ;
- adresse postale, URL personnelle, e-mail, noms de famille, identifiants d'appareils ou numéros de série ;
- captures Google Cloud montrant des identifiants privés ou données de compte non masquées.

## Captures d'écran

Avant d'ajouter une capture : masquer les comptes, les domaines personnels, les adresses IP, les secrets et tout identifiant sensible. Un ID de projet Cloud ou Device Access n'est pas forcément un secret, mais l'anonymiser facilite la réutilisation du guide.

## Si un secret a déjà été exposé

Le supprimer du dépôt **ne suffit pas** s'il reste dans l'historique Git. Révoquer/faire tourner le secret concerné et vérifier l'historique. Éviter de copier-coller des journaux complets sans relecture.

## Permissions

Préférer les droits minimaux nécessaires. Ne pas attribuer le rôle **Owner** à des comptes tiers uniquement pour résoudre une erreur. Le rôle **Pub/Sub Publisher** est destiné au principal de publication Google sur le sujet concerné.

## Documentation

Utiliser des identifiants fictifs tels que `ha-nest-123456` et `example.com` dans les exemples.
