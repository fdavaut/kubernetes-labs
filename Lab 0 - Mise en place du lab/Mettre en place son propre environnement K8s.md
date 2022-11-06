# Mettre en place son propre environnement K8s

Pour ce cours sur Kubernetes, nous allons lancer 3 VMs (un master node K8s et 2 worker nodes K8s) qui composent notre cluster.

**Step 1 :** Assurez-vous que ces 3 logiciels sont installés :
* [Vagrant](https://www.vagrantup.com/downloads) : Lien direct vers la version (2.3.0) que j'utilise dans ma démonstration --> [Lien](https://releases.hashicorp.com/vagrant/2.3.0/)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads) : en fonction de votre système d'exploitation, installez celui qui correspond. J'utilise VirtualBox pour MacOs Intel --> [Lien](https://download.virtualbox.org/virtualbox/6.1.36/VirtualBox-6.1.36-152435-OSX.dmg)
* [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html): Ansible ne peut pas être installé sur Windows sans WSL2. Veuillez vous assurer qu'Ansible est correctement installé. 

**Step 2 :** Télécharger les fichiers du Lab et déployer votre cluster Kubernetes

* Télécharger le contenu du répertoire `Fichiers installation cluster K8s`
* Placez-vous dans le répertoire où se trouve le contenu
* Ensuite, démarrez l'exécution avec la commande vagrant `$ vagran up`

## Autres Outils utilisés

Parce que je sais qu'on me pose souvent la question, comment ai-je personalisé mon environnement de travail. Sachez que ce n'est pas un outil mais plusieurs outils avec différentes options à la clé

* Commençons par mon éditeur de texte préféré : [VSCODE](https://code.visualstudio.com/) avec les extension suivantes
  * GitLens
  * GitHub Theme (J'ai choisi Dark Default)
  * Docker
  * YAML
  * MARKDOWN All in One
  * vscode-icons
  * Ci dessous les configurations de mon terminal
  ```json
  {
   (...)
    "workbench.colorTheme": "GitHub Dark Default",
    "workbench.colorCustomizations": {
        "terminal.background":"#000000",
        "terminal.foreground":"#6bb8e6",
        "terminal.ansiBlack":"#000000",
        "terminal.ansiBrightBlack":"#414141",
        "terminal.ansiBlue": "#bceafa",
        "terminal.ansiBrightBlue":"#77dcff",
        "terminal.ansiCyan":"#a3e9d4",
        "terminal.ansiBrightCyan":"#a3e9d5",
        "terminal.ansiGreen":"#68f400",
        "terminal.ansiBrightGreen":"#dcf152",
        "terminal.ansiMagenta":"#f58a8d",
        "terminal.ansiBrightMagenta":"#ffb4b8",
        "terminal.ansiRed":"#ff4c41",
        "terminal.ansiBrightRed":"#ff7c77",
        "terminal.ansiWhite":"#ffffff",
        "terminal.ansiBrightWhite":"#ffffff",
        "terminal.ansiYellow":"#ffe100",
        "terminal.ansiBrightYellow":"#fff68b",
        }
  }

  ```
* Personalisation du Terminal, je vous recommande ces tutoriels
  * Utilisateurs MacOs : https://jgrandchavin.medium.com/how-to-get-beautiful-and-efficient-terminal-on-macos-22c40c516bec
  * Utilisateurs Windows : https://www.youtube.com/watch?v=AK2JE2YsKto&ab_channel=TheDigitalLife
  * Utilisateurs Linux : https://betterprogramming.pub/5-steps-to-a-beautiful-terminal-that-youll-love-using-9e94ecb4191b 
* Auto completion des commande Kubernetes
* Surveillance automatique des changements avec la commande `watch`
* JQ pour le formattage des output au format JSON