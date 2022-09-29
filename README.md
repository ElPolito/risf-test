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

## SSL

Génération des certificats SSL d'après cette [page d'IBM](https://www.ibm.com/docs/en/runbook-automation?topic=certificate-generate-rba-server)

1. Génération de la clé privée pour le CA root (dans le dossier **ssl/rootca**)
    `openssl genrsa -out rootCAKey.pem 2048`
2. Génération du certificat pour le CA root (même dossier)
    `openssl req -x509 -sha256 -new -nodes -key rootCAKey.pem -days 3650 -out rootCACert.pem`
3. Ajout du certificat du CA root sur le navigateur
4. Génération de la clé privée pour RISF (dans le dossier **ssl/rba/risf**)
    `openssl genrsa -out rbaServerKey.pem 2048`
5. Génération du CSR pour RISF (même dossier)
    `openssl req -new -key rbaServerKey.pem -sha256 -out rbaServerCert.csr -config rbaServerCertReq.config`
6. Génération de la clé privée en base 64 (même dossier)
    `openssl base64 -in rbaServerKey.pem`
7. Ajout de la clé privée en base 64 dans le fichier **risf/risf.secret.yml** comme valeur pour la clé **tls.key** (enlever les sauts de ligne)
8. Copie du fichier **ssl/rba/risf/rbaServerCert.csr** dans le dossier **ssl/rootca/certs/risf** 
9. Génération du certificat pour RISF (dans le dossier **ssl/rootca/certs/risf**)
    `openssl x509 -req -sha256 -in rbaServerCert.csr -CA ../rootCACert.pem -CAkey ../rootCAKey.pem -CAcreateserial -out rbaServerCert.pem -days 365 -extfile v3.ext`
10. Génération du certificat en base 64 (même dossier)
    `openssl base64 -in rbaServerCert.pem`
11. Ajout du certificat en base 64 dans le fichier **risf/risf.secret.yml** comme valeur pour la clé **tls.crt** (enlever les sauts de ligne)
12. Refaire les mêmes étapes en modifiant les dossiers et les fichiers pour ITSF en commençant à l'étape 4


## Lancement

### RISF

1. Appliquer le secret
   `kubectl apply -f ./risf/risf.secret.yml`
2. Créer le déploiement
   `kubectl apply -f ./risf/risf.deployment.yml`
3. Créer le service
   `kubectl apply -f ./risf/risf.service.yml`

### ITSF

1. Appliquer le secret
    `kubectl apply -f ./itsf/itsf.secret.yml`
2. Créer le volume persistant
    `kubectl apply -f ./itsf/itsf.persistent-volume.yml`
3. Créer le claim du volume
    `kubectl apply -f ./itsf/itsf.persistent-volume-claim.yml`
4. Créer le déploiement
   `kubectl apply -f ./itsf/itsf.deployment.yml`
5. Créer le service
   `kubectl apply -f ./itsf/itsf.service.yml`

### Ingress

Pour déployer l'Ingress il faut avoir un ingress controller. Si ce n'est pas le cas, il est possible de le déployer avec cette commande :

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.1/deploy/static/provider/cloud/deploy.yaml`

1. Déployer l'Ingress
    `kubectl apply -f ./ingress.yml`