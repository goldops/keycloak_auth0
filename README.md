🎯 Objectif du projet
Ce Proof of Concept (POC) a pour but de démontrer comment intégrer Keycloak en tant que Service Provider (SP) avec Auth0 en tant qu’Identity Provider (IdP), en utilisant SAML2 comme protocole d’authentification.
L’environnement est déployé sur une instance EC2 (2 vCPU / 4 Go) avec HTTPS activé et une interface d’administration Keycloak accessible.

Ce POC simule l’authentification d’un employé fictif d’une compagnie fictive souhaitant accéder à son espace professionnel.
L’objectif est de valider la chaîne complète :

Redirection vers l’IdP Auth0
Authentification via SAML2
Retour vers Keycloak et émission d’un token pour l’application métier

🏗 Architecture technique
Keycloak : Service Provider, interface d’admin et gestion des mappages de rôles.
Auth0 : Identity Provider externe, authentifie l’utilisateur via SAML2.
Caddy : Reverse proxy, termination HTTPS, routage interne.
PostgreSQL : Base de données persistante pour Keycloak.

👤 Itinéraire de l’utilisateur fictif
Contexte : Notre utilisateur fictif s’appelle Jean Dupuis, employé du département marketing, qui souhaite accéder à l’intranet professionnel de l’entreprise.

1. Accès à l’application métier
Jean ouvre son navigateur et visite l’URL de l’application professionnelle :
https://app.example.com

L’application n’ayant pas de session valide, elle déclenche une redirection OIDC vers le serveur d’authentification :
https://kc.example.com/realms/societe/protocol/openid-connect/auth

2. Passage par Keycloak (Service Provider)
Keycloak reçoit la requête d’authentification et détecte que l’IdP configuré pour ce realm est Auth0 via SAML2.
Il redirectionne Jean vers le point d’entrée SAML d’Auth0, en embarquant les métadonnées nécessaires.

3. Authentification sur Auth0 (Identity Provider)
Auth0 affiche la page de connexion :
Jean saisit ses identifiants (jean.dupuis@societe.com + mot de passe).
Auth0 vérifie ces informations et crée une assertion SAML signée contenant :
NameID = jean.dupuis@societe.com
Attributs : prénom, nom, service, rôle.

4. Retour vers Keycloak
Auth0 renvoie l’assertion SAML à Keycloak via un POST sécurisé vers l’endpoint ACS (Assertion Consumer Service).
Keycloak :

Vérifie la signature de l’assertion.
Mappe les attributs reçus vers ses propres rôles (role_marketing, role_employee).
Crée une session utilisateur.

5. Émission du Token OIDC pour l’application
Keycloak redirige Jean vers l’application métier avec un ID Token et un Access Token OIDC.
Ces tokens contiennent les informations nécessaires (nom, rôle, email) pour que l’application :

Affiche un accueil personnalisé : “Bonjour Jean !”
Active les modules réservés aux employés marketing.

6. Session active et accès sécurisé
Jean peut maintenant naviguer librement dans son espace professionnel.
Si sa session expire, le même processus sera rejoué, mais tant que la session Auth0 est active, la reconnexion sera quasi instantanée.
