Mise en place d'une environnement automatisé ansible 
---------------------------------------------------

## 1/ Préparation des environnements de travail 

Afin de réaliser un teste complet, il est nécessaire de mettre en place un mini Labo avec (au moins) 2 machine. 
Dans notre cas, nous allons mettre en place un labo avec 3 machine :
- Un contrôleur Ansible "Ansible-control" 
- Un serveur Applicatif "Ansible-app-svr"
- Un serveur de base des données "Ansible-db-svr"

	### 1.1/ Installation des outils de virtualisation 
		Pour avoir un Labo près à fonctionner, nous allons utiliser l'outils Vagrant, qui sert à faire la gestion des Machines virtuelle, avec Virtual Box
	
	Télécharger Vagrant ici : https://www.vagrantup.com/downloads.html
	Télécharger Virtual Box ici : https://www.virtualbox.org/wiki/Downloads
			
	### 1.2/ Préparation des machine Linux Centos-7
	Pour récupérer les images Vagrant, on peut :
	- soit éditer la configuration du Vagrantfile dès le début, avec un `vagrant ini`,  puis dans le fichier Vagrantfile généré rajouter le nom de l'image à récupérer plus tout la configuration pour la machine à créer
	- soit rechercher une image sur le Atlas (https://app.vagrantup.com/boxes/search), puis initialiser la configuration du Vagrantfile avec le nom d'une image connu. Exemple : `vagrant init bento/centos-7.4`

			Note : on peut créer plusieurs machine à partir de la même configuration Vagrantfile. 
			
	### 1.3/ Scripter la configuration des machines avec Vagrant
	Exemple d'un script de configuration pour les machines :
	config.vm.define "ansible-db-svr" do |ansibdb| 
	    ansibdb.vm.box = "bento/centos-7.4"
	    ansibdb.vm.box_check_update = false
	    ansibdb.vm.network "private_network", ip: "192.168.33.50"
	    ansibdb.vm.provider "virtualbox" do |vb|

        # Customize the amount of memory on the VM:
        vb.name = "ansible-db-svr"
        vb.memory = "450"
    end
  end
	### 1.4/ Démarrage des machines
Pour démarer les machine avec Vagrant : 
`vagrant up <nom_de_la_machine>`
- pour faire appliquer des modification sur le fichier Vagrantfile, on rajoute le flag `--provision`

- une fois la machine démarré, on peut se connecter sur la machine en faisant : `vagrant ssh`. 

- pour gérer les machine dans un outils comme putty et supperPutty, on récupéré la clé privé crée par Vagrant et qui se situe à la racine du dossier Vagrant dans le répertoire `.vagrant\machines\<nom_de_la_machine>\virtualbox\private_key`, puis faire la configuration dans putty pour se connecter au machine. 
Tout fois, les machines seront uniquement accessible via putty (ou Vagrant) seulement après qu'ils soient démarré avec la commande `vagrant up`

## 2/ Configuration des machines 
Afin de pouvoir exécuter Ansible dans notre labo, une configuration minimal s'impose, tel que le partage des clés, s'assurer que les Users ont des privilèges suffisants pour exécuter Ansible, etc... 

### 2.1/ Préparation des clés ssh (avec gestion des autorisations)
Création des clés ssh depuis la machine en `Ansible-control`:
`ssh-keygen -t <type_crypte> -b <taille_clé>` 
ex: `ssh-keygen -t rsa -b 4096` 

Envoi la clé vers les machine cible : `Ansible-app-svr` et `Ansible-db-svr`
`ssh-copy-id -i /path/to/ssh-priv-key <user>@<host_name | host_ip>`
assurez-vous d'avoir un mot de passe pour l'utilisateur choisi 

ex: `ssh-copy-id -i ~/.ssh/id_rsa.pub centos@Ansible-app-svr`

### 2.2/ Préparation des Users pour Ansible
entant que `root`, on crée un User `Ansible` avec comme mot de passe `Ansible` 
`$root> useradd -d /home/ansible -m ansible`
`$root> passwd ansible` puis fournir le mot de passe.

### 2.3/ Donner les droit root au User Ansible
entant que `root` faire cette opperation sur tout les machines `ansible-control`, `ansible-ap-svr`, `ansible-db-svr`
`$root> visudo` puis, ajouter à la fin de la page la ligne suivante :
`ansible ALL = PASSWD: ALL`

## 3/ Installation d'Ansible 
### 3.1/ Installation d'Ansible sur le contrôleur
pour l'installation d'Ansible, on utilise le repository officielle Centos avec yum
`sudo yum -y install ansible`

### 3.2/ Installation des outils optionnel dans les nœuds 
parfois, on a besoin d'avoir des outils dans les machines cible, comme `git` pour pourvoir utiliser certaine module ansible, comme `ansible_git`
Cependant, on doit installer les outils nécessaire.  

## 4/ Configuration des script Ansible
### 4.1/ Création du fichier d'inventaire 
Pour contrôler la liste des hosts qui doivent fonctionner avec Ansible, on peut faire son propre fichier inventaire afin d'éviter qu'Ansible utiliser l'inventaire par défaut. 

### 4.2/ Configuration des rôles ansible
pour créer les roles, on utilise la commande `Ansible-galaxy init <nom_du_role>` 
ex: `ansible-galaxy init ngixn-install`

### 4.3/ Configuration des playbooks
Pour une meilleurs façon de gérer les playbook ansible, on crée un playbook de départ, que l'on peut appelé : `startup.yml`, dans laquelle on peut importer les autres playbools et les appeler par ordre de priorité. 

## 5/ Exécution d'Ansiblea
Pour lancer les playbook, on utilise la commende : 
`ansible-playbook /path/to/playbook`
ex: `ansible-playbook ./startup.yml`

--------------------------------------
