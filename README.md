# RISF test

## Demande

Création d'un cluster Kubernetes en local ( avec Docker for Windows/MAC, Kind ou k3s )

Déploiement d'un micro service nginx pour afficher une page web static:

    Contenu de la page web: "HELLO RISF"
    Création d'une image docker se basant sur l'image nginx officiel et y ajouter le .html.
    utilisation de cette image pour le déploiement.

Déploiement d'un deuxième micro service nginx pour afficher une page web static:

    Contenu de la page web: "HELLO ITSF"
    Utilisation de l'image docker nginx officiel
    Le .html devrait être monté via un PV en local.

Bonus : ces micro-services ne devront pas tourner avec le user root.

Exposition des pages web avec Ingress :

    Création d'une pki ( CA root )
    générer un certificat hello-risf.local.domain signé par le CA
    généré un certificat hello-itsf.local.domain signé par le CA
    exposer les pages web sous les noms de domaine hello-risf.local.domain et hello-itsf.local.domain avec les certificats générés
    trafic en HTTPS jusqu'à Ingress, puis en HTTP jusqu'aux micro service nginx.

Lorsque l'on visitera les deux sites web depuis le navigateur internet, le warning de sécurité ne devra pas apparaître

## Réalisation

1. Créer le déploiement du Nginx RISF
`kubectl apply -f ./risf/risf.deployment.yml`
2. Créer le service pour Nginx RISF
`kubectl apply -f ./risf/risf.service.yml`
3. On peut maintenant se rendre sur la page `http://localhost:8082`
4. Créer le volume persistant
`kubectl apply -f ./itsf/itsf.persistent-volume.yml`
5. Créer le claim
`kubectl apply -f ./itsf/itsf.persistent-volume-claim.yml`
6. Créer le déploiement du Nginx ITSF
`kubectl apply -f ./itsf/itsf.deployment.yml`
7. Créer le service pour Nginx ITSF
`kubectl apply -f ./itsf/itsf.service.yml`
8. On peut maintenant se rendre sur la page `http://localhost:8083`