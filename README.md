------------------------------------------------------------------------------------------------------
ATELIER FROM IMAGE TO CLUSTER
------------------------------------------------------------------------------------------------------
L’idée en 30 secondes : Cet atelier consiste à **industrialiser le cycle de vie d’une application** simple en construisant une **image applicative Nginx** personnalisée avec **Packer**, puis en déployant automatiquement cette application sur un **cluster Kubernetes** léger (K3d) à l’aide d’**Ansible**, le tout dans un environnement reproductible via **GitHub Codespaces**.
L’objectif est de comprendre comment des outils d’Infrastructure as Code permettent de passer d’un artefact applicatif maîtrisé à un déploiement cohérent et automatisé sur une plateforme d’exécution.
  
-------------------------------------------------------------------------------------------------------
Séquence 1 : Codespace de Github
-------------------------------------------------------------------------------------------------------
Objectif : Création d'un Codespace Github  
Difficulté : Très facile (~5 minutes)
-------------------------------------------------------------------------------------------------------
**Faites un Fork de ce projet**. Si besion, voici une vidéo d'accompagnement pour vous aider dans les "Forks" : [Forker ce projet](https://youtu.be/p33-7XQ29zQ) 
  
Ensuite depuis l'onglet [CODE] de votre nouveau Repository, **ouvrez un Codespace Github**.
  
---------------------------------------------------
Séquence 2 : Création du cluster Kubernetes K3d
---------------------------------------------------
Objectif : Créer votre cluster Kubernetes K3d  
Difficulté : Simple (~5 minutes)
---------------------------------------------------
Vous allez dans cette séquence mettre en place un cluster Kubernetes K3d contenant un master et 2 workers.  
Dans le terminal du Codespace copier/coller les codes ci-dessous etape par étape :  

**Création du cluster K3d**  
```
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```
```
k3d cluster create lab \
  --servers 1 \
  --agents 2
```
**vérification du cluster**  
```
kubectl get nodes
```
**Déploiement d'une application (Docker Mario)**  
```
kubectl create deployment mario --image=sevenajay/mario
kubectl expose deployment mario --type=NodePort --port=80
kubectl get svc
```
**Forward du port 80**  
```
kubectl port-forward svc/mario 8080:80 >/tmp/mario.log 2>&1 &
```
**Réccupération de l'URL de l'application Mario** 
Votre application Mario est déployée sur le cluster K3d. Pour obtenir votre URL cliquez sur l'onglet **[PORTS]** dans votre Codespace et rendez public votre port **8080** (Visibilité du port).
Ouvrez l'URL dans votre navigateur et jouer !

---------------------------------------------------
Séquence 3 : Exercice
---------------------------------------------------
Objectif : Customisez un image Docker avec Packer et déploiement sur K3d via Ansible
Difficulté : Moyen/Difficile (~2h)
---------------------------------------------------  
Votre mission (si vous l'acceptez) : Créez une **image applicative customisée à l'aide de Packer** (Image de base Nginx embarquant le fichier index.html présent à la racine de ce Repository), puis déployer cette image customisée sur votre **cluster K3d** via **Ansible**, le tout toujours dans **GitHub Codespace**.  

**Architecture cible :** Ci-dessous, l'architecture cible souhaitée.   
  
![Screenshot Actions](Architecture_cible.png)   
  
---------------------------------------------------  
## Processus de travail (résumé)

1. Installation du cluster Kubernetes K3d (Séquence 1)
2. Installation de Packer et Ansible
3. Build de l'image customisée (Nginx + index.html)
4. Import de l'image dans K3d
5. Déploiement du service dans K3d via Ansible
6. Ouverture des ports et vérification du fonctionnement

---------------------------------------------------
Séquence 4 : Documentation  
Difficulté : Facile (~30 minutes)
---------------------------------------------------
**Complétez et documentez ce fichier README.md** pour nous expliquer comment utiliser votre solution.  
Faites preuve de pédagogie et soyez clair dans vos expliquations et processus de travail.  

Image to Cluster – Déploiement Kubernetes avec Packer et Ansible

Objectif

L’objectif de ce projet est de mettre en place une chaîne DevOps complète permettant de construire, déployer et exposer une application web sur un cluster Kubernetes. Pour cela, nous utilisons Packer afin de créer une image Docker personnalisée, puis Ansible pour automatiser son déploiement sur un cluster k3d.

---

Architecture

Le fonctionnement global repose sur les étapes suivantes :

- Un fichier `index.html` sert de contenu applicatif
- Packer permet de créer une image Docker Nginx personnalisée
- Cette image est importée dans un cluster Kubernetes k3d
- Ansible automatise le déploiement de l’application
- Kubernetes expose le service pour le rendre accessible

---

Étapes de réalisation

Création du cluster Kubernetes

Un cluster Kubernetes a été créé à l’aide de k3d avec un nœud principal et deux nœuds workers, permettant de simuler un environnement distribué.

```bash
k3d cluster create lab --servers 1 --agents 2
kubectl get nodes
```
2. Création de l’image avec Packer

Une image Docker personnalisée a été construite à partir de l’image officielle Nginx. Le fichier index.html présent dans le projet est injecté dans l’image afin de remplacer la page par défaut.
```
packer {
  required_plugins {
    docker = {
      version = ">= 1.0.0"
      source  = "github.com/hashicorp/docker"
    }
  }
}

source "docker" "nginx" {
  image  = "nginx:latest"
  commit = true
}

build {
  sources = ["source.docker.nginx"]

  provisioner "file" {
    source      = "index.html"
    destination = "/usr/share/nginx/html/index.html"
  }
}
```
L’image est ensuite construite avec les commandes suivantes :
```
packer init .
packer build nginx.pkr.hcl
```
Cette étape permet de créer une image contenant le contenu personnalisé.   

3. Import de l’image dans k3d

L’image Docker étant locale, elle doit être importée dans le cluster k3d afin d’être utilisable par Kubernetes.
```
k3d image import nginx:latest -c lab
```

4. Déploiement avec Ansible

Le déploiement de l’application est automatisé à l’aide d’un playbook Ansible qui exécute des commandes Kubernetes.
```
- name: Deploy nginx
  hosts: localhost
  connection: local

  tasks:
    - name: Create deployment
      command: kubectl create deployment nginx-custom --image=nginx:latest
```
Exécution du playbook :
```
ansible-playbook deploy.yml
```
Cette étape permet de créer un déploiement Kubernetes basé sur l’image personnalisée.

5. Exposition du service

Afin de rendre l’application accessible, le déploiement est exposé via un service Kubernetes.
```
kubectl expose deployment nginx-custom --type=NodePort --port=80
kubectl port-forward svc/nginx-custom 8080:80
```

Le port 8080 est ensuite utilisé pour accéder à l’application depuis l’environnement Codespaces.

Vérification

Une fois le déploiement terminé, l’application est accessible via le port 8080. Le bon fonctionnement est validé lorsque la page affichée correspond au contenu du fichier index.html personnalisé et non à la page par défaut de Nginx.

---------------------------------------------------
Evaluation
---------------------------------------------------
Cet atelier, **noté sur 20 points**, est évalué sur la base du barème suivant :  
- Repository exécutable sans erreur majeure (4 points)
- Fonctionnement conforme au scénario annoncé (4 points)
- Degré d'automatisation du projet (utilisation de Makefile ? script ? ...) (4 points)
- Qualité du Readme (lisibilité, erreur, ...) (4 points)
- Processus travail (quantité de commits, cohérence globale, interventions externes, ...) (4 points) 


