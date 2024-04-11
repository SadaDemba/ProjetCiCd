# Projet : Mise en place du CICD dans un environnement Kubernetes pour une application de microservices 

## Contexte
Vous travaillez sur le déploiement d'une application composée de microservices dans un environnement Kubernetes. L'application est constituée de quatre services : le frontend React, le service de gestion des ordres, le service utilisateur interagissant avec une base de données MongoDB, et enfin, une instance MongoDB.
Lien vers l'énoncé `https://github.com/mpakoupete/cicd-project.git`.

## Étapes du Projet :

**1. Configuration en Local :**
   - Configurez et testez l'application en microservices localement.
   - Assurez-vous que les services communiquent correctement entre eux en utilisant les noms de services suivants :
     - `frontend` pour le frontend React,
     - `order-service` pour le service de gestion des ordres,
     - `user-service` pour le service utilisateur,
     - `mongo-service` pour la base de données MongoDB.

** Solution de l'étape 1 :**
   - Clonez le dépôt contenant les sources de votre application de microservices.
   - Accéder au repertoire de chaque service, installer les dependances puis lancer le serveur.
   - Tester l'application.
     

**2. Dockerization :**
   - Rédigez des fichiers Dockerfile pour chacun des trois services développés.
   - Assurez-vous que les images construites sont stockées dans un repository privé. Vous pouvez utiliser votre compte privé DockerHub pour cela.

** Solution de l'étape 2 :**
   - Dans cette partie il faut vous assurer que vous avez docker installé en local sur votre machine.
   - Creer les fichiers dockerfile pour chacun des service dans leurs répertoires respectifs.
   - Creer le fichier docker-compose à la racine du projet que nous allons utiliser pour executer nos services.
   - Se mettre dans le repertoire racine et taper la commande `docker-compose -f <nom-du-fichier> up`
   - Tester à nouveau l'application.

**3. Mise en place du Pipeline CI :**
   - Utilisez votre outil CI préféré pour mettre en place un pipeline qui construit les images de chaque service à partir des Dockerfiles.
   - Assurez-vous que les images sont correctement poussées dans le repository privé.

** Solution de l'étape 3 :**
   - Dans cette étape nous avons choisi github action comme outils d'integration continu après avoir rencontrer pas mal de problèmes avec Circle Ci et Travis Ci.
     En effet avec ces outils j'avais du mal à executer un processus après des actions spécifiques telle que la création d'une nouvelle branche à partir de main par exemple.
   - Vous retrouverez les configurations dans notre fichier yml se trouvant dans `.github/workflows`. Il s'agit pour le moment du job `build-and-push-main`.
   - Creer un repo privé dans dockerhub sur lequel nous allons pousser les images qui seront créée.
   - Generer un token à partir de dockerhub permettant d'accéder à ce repo et gardez le en lieu sûr.
   - Creer un secret sur github dans lequel vous aller mettre le token précédemment créé. Ce secret nous permettra de nous connecter sur DockerHub et d'y pousser les images.
     Dans notre cas notre       secret esp appelé `DOCKER_ACCESS_TOKEN` comme vous pouvez le voir dans le fichier yml.
   - Faites un test en poussant sur main. vous verrez de nouvelles images créées dans votre repos dockerhub.
     

**4. Versioning :**
   - Mettez en place un second pipeline qui se déclenche automatiquement lors de la création d'une branche correspondant à la version de l'application (ex : `2.0.0`) et lorsque le code est poussé dans cette branche.
   - Ce pipeline doit construire des images avec des tags correspondant à la version (ex : `dockerhubloginname/user-service:2.0.0`) et les pousser dans le repository privé.

** Solution de l'étape 4 :**
   - Ajouter le job `build-and-push-versioned`au fichier yml et tester en créant une nouvelle branche de version au format x.y.z avec x, y et z étant des chiffres.

** NB : Vu que je suis sur mac m3 j'ai fait une configuration permettant de créér, pour chaque service, une image avec la platforme Amd64 et Arm64 en utilisant la commande buildx.
        Ceci ralentit considérablement le processus de publication des image mais permet de pouvoir tester les images quelque soit notre Operating System.** 

**5. Déploiement dans Kubernetes :**
   - Effectuez une mise à jour des fichiers de déploiement Kubernetes pour refléter les nouvelles images.
   - Poussez ces fichiers de déploiement dans un repository Git séparé, nommé `microapp-deploy`.

** Solution de l'étape 5 :** Nous commençons maintenant la mise en place du déploiement continu
   Voici le lien vers le repo microapp-deploy : `https://github.com/SadaDemba/microapp-deploy.git`
   - Cloner le repository ci dessus.
   - Pour le moment tous les fichier du repertoire principal nous interresse sauf le fichier `ingress.yml` sur lequel nous reviendrons plus tard.
   - Nous allons utiliser minikube pour faire cette partie. Assurer vous de demarrer minikube en lançant la commande `minikube start`.
   - Dans le repertoire database, j'ai mis tous les objet qui sont en rapport avec la base donnée mongodb.
   - Vous y trouverez l'objet statefulset demandé comme exigence afin de garder les donner même si le pod noeud s'eteinds, ainsi que les fichiers `storageclass` et `persistancevolume` qui sont        nécessaire au statefulset. Enfin il y a le service mongodb qui permet d'exposer notre BdD.
    
** NB : Nous avons deux clés à configurer:**
    - Première clé pour la base de données comme demandé sur le point bonus:
        `kubectl create secret generic mongodb-password --from-literal=password=admin`
      Ici nous mettons comme secret `admin` comme cela a été configuré dans le service user lors de la connection à la BdD.
      
   - Seconde clé pour accéder au repo privé DockerHub afin d'y recuperer les images et les déployer sur K8s:

        `kubectl create secret docker-registry myregistrykey \
        --docker-server=https://index.docker.io/v1/ \
        --docker-username=<votre-username-dockerhub> \
        --docker-password=<votre-token-dockerhub > \
        --docker-email=<votre-adresse-email>`
     
     Nous avons nommé la clé `myregistrykey`et c'est ce nom qui se trouve  dans les fichiers de déploiement.
     Assurez vous de remplacer <votre-username-dockerhub>, <votre-token-dockerhub > et <votre-adresse-email> par vos informations.

   - Faite la commande permettant de déployer vos services. Commençons par la base de données.
   - Pour cela, placez vous dans le répertoire courant et exectez la commande `kubectl apply -f /database`
   - Ensuite deployons le reste avec `kubectl apply -f .`. Je rappelle que pour le moment ingress ne nous interesse pas.
   - Les images seront maintenant déployés sur le cluster.
   - Pour les tester, il faut ouvrir le navigateur et taper l'url `http://<adresse_ip_du_noeud>:<port_NodePort>`. pour avoir l'adresse ip du noeud faire
     `kubectl get nodes -o wide`et le node port ici c'est 30000, comme configurer dans le service frontend.
   Attention, si vous etes sur mac m3, il vous faut faire pour le moment un port forwarding pour tester à cause de l'isolation du noyeau docker sur l'OS.
     `kubectl port-forward service/nom-du-service port-local:port-service`
   Nous allons, à la fin, mettre en place ingress afin de pouvoir gerer une redirection vers les apis.
     
**6. Déploiement avec ArgoCD :**
   - Configurez une installation d'ArgoCD sur le cluster Kubernetes.
   - Faites en sorte qu'ArgoCD récupère automatiquement les fichiers de déploiement du repository Git `microapp-deploy` et les déploie sur le cluster.

** Solution de l'étape 6 :**
  - Pour la configuration de argocd, commencer par creer le namespace argocd comme suit `kubectl create namespace argocd`.
  - Puis executer la ccommande suivante afin de faire l'installation: `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`.
  - Executer cet commande pour acceder à l'interface de argocd avec localhost:8080 `kubectl port-forward -n argocd svc/argocd-server 8080:443`.
  - Pour se connecter revenir sur le terminal et taper la commande ci dessous pour avoir le mot de passe. Le mot d'utilisateur par défaut est admin.
    `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo`
  - Une fois connecté, créér un nouvelle application en y spécifiant votre repos microapp-deploy. A chaque changement sur le repo, argo sera executé, redéployant ainsi les services.

** Configuration de Ingress :**
  - Maintenant notre ingress nous sera util. Créons d'abord le controller ingress avec la commande suivante:
    `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml`
  - Ensuite, appliquons notre ingress avec la commande `kubectl apply -f ingress.yml`.
  - A cette étape, sauf si vous êtes sur mac m3:
     - Il faut faire la commande `kubectl tunnel` pour acceder au service de type LoadBalancer de ingress.
     - Modifier votre fichier de redirection locale /etc/hosts en y ajoutant une nouvelle ligne `127.0.0.1 api.microapp-deploy.com`. Cette ligne permet de rediriger tous les appels de cet url          vers le localhost qui est à comme @ ip 127.0.0.1 . Ainsi pour acceder à l api user , il faut faire la requete vers le path `api.microapp-deploy.com/user`. Ainsi pour la requete getUsers         ce sera l'url `api.microapp-deploy.com/user/user`.
  - Si vous êtes sur une puce m3, il faut faire un port forward du service LoadBalancer de ingress pour y accéder.
   `kubectl port-forward svc/ingress-nginx-controller 3005:80 -n ingress-nginx`
   J'ai choisi le port 3005. Ceci engendre comme conséquence que le path pour récuper les users contiendra le numero de port comme ceci `api.microapp-deploy.com:3005/user/user`. Dans les 
   configurations actuelles du repos j'ai choisi la première car les utilisateurs sur mac m3 sont largement minoritaires. Pour tester sur mac m3 il faudra simplement changer, sur le frontend, 
   l'url permettant daccéder aux services en y ajoutant le numero de port.

Le tour étant joué, vous pouvez maintenant tester votre application avec `localhost:30000`.
