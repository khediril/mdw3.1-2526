# **Atelier 01 Préparatoire : Mise en Place de l'Environnement Symfony**

Bienvenue dans cet atelier initial \! 🚀 Avant de plonger dans le code et de construire notre première application, il est essentiel de préparer notre machine. Cet atelier vous guidera pas à pas pour installer et configurer tous les outils nécessaires au développement avec le framework Symfony.

## **Qu'est-ce qu'un Framework et pourquoi en utiliser un ?**

Imaginez que vous construisiez une maison. Au lieu de fabriquer chaque brique, chaque clou et chaque outil à partir de zéro, vous utilisez un ensemble d'outils préfabriqués et un plan de construction. [cite\_start]Un **framework** est exactement cela pour le développement web : une "boîte à outils" et une structure de base créées par des développeurs pour d'autres développeurs[cite: 542, 543].

Utiliser un framework comme Symfony nous permet de :

  * [cite\_start]**Gagner du temps** : On se concentre sur les fonctionnalités uniques de notre application plutôt que sur des tâches techniques répétitives[cite: 547].
  * [cite\_start]**Réutiliser du code** : Il fournit des bibliothèques de code prêtes à l'emploi[cite: 548].
  * [cite\_start]**Coder proprement** : Il impose des normes et une organisation, ce qui rend le code plus facile à maintenir[cite: 549].

-----

## **Étape 1 : Préparation du Système (Ubuntu)**

[cite\_start]Pour cet atelier, nous supposerons que vous travaillez sur un système **Ubuntu 22.04** (ou une version similaire)[cite: 555]. La première étape est de s'assurer que votre système est à jour.

1.  **Ouvrez le terminal** :
    [cite\_start]C'est l'outil qui nous permet de communiquer avec le système via des commandes textuelles[cite: 557]. [cite\_start]Vous pouvez l'ouvrir en cherchant "terminal" dans vos applications ou avec le raccourci `Ctrl+Alt+T`[cite: 557, 558].

2.  **Mettez à jour votre système** :
    Exécutez les deux commandes suivantes. La première télécharge la liste des paquets disponibles, la seconde installe les mises à jour.

    ```bash
    sudo apt update
    ```

    ```bash
    sudo apt upgrade
    ```

-----

## **Étape 2 : Installation des Outils Essentiels**

Nous allons maintenant installer les briques logicielles de notre environnement.

### **PHP : Le Moteur de l'Application** ⚙️

**C'est quoi ?** PHP est le langage de programmation sur lequel Symfony est construit. C'est le "moteur" qui va exécuter tout notre code côté serveur pour générer les pages web.

1.  **Installez PHP** :
    [cite\_start]Symfony requiert PHP 8.1 ou une version supérieure[cite: 571]. La commande suivante installe la version disponible dans les dépôts d'Ubuntu.

    ```bash
    sudo apt install php
    ```

2.  **Installez les extensions nécessaires** :
    [cite\_start]Symfony a besoin de quelques modules PHP supplémentaires pour fonctionner correctement, notamment pour communiquer avec la base de données (`php-mysql`) et pour gérer les fichiers XML (`php-xml`)[cite: 575].

    ```bash
    sudo apt install php-xml php-mysql
    ```

3.  **Vérifiez l'installation** :
    Assurez-vous que PHP est bien installé en vérifiant sa version.

    ```bash
    php --version
    ```

### **Composer : Le Gestionnaire de Dépendances** 📦

**C'est quoi ?** Une application moderne ne réinvente pas la roue ; elle utilise des bibliothèques (ou "packages") externes. [cite\_start]**Composer** est l'outil indispensable en PHP qui gère ces dépendances pour nous : il les télécharge, les installe et les met à jour[cite: 587, 588].

1.  **Installez Composer** :
    Exécutez les commandes suivantes l'une après l'autre. [cite\_start]Elles vont télécharger l'installateur, l'exécuter, puis le nettoyer, et enfin rendre la commande `composer` accessible globalement[cite: 579, 580, 581, 582].

    ```bash
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"
    sudo mv composer.phar /usr/local/bin/composer
    ```

2.  **Vérifiez l'installation** :

    ```bash
    composer --version
    ```

### **Git : Le Gardien de votre Code** Git

**C'est quoi ?** Git est un système de contrôle de version. Il enregistre l'historique de toutes les modifications apportées à votre code. C'est un outil fondamental pour suivre vos changements, revenir en arrière si nécessaire et collaborer à plusieurs sur un même projet.

1.  **Installez Git** :

    ```bash
    sudo apt install git
    ```

2.  **Configurez Git** :
    Vous devez dire à Git qui vous êtes. [cite\_start]Ces informations seront associées à chaque modification que vous enregistrerez[cite: 598].

    ```bash
    git config --global user.name "Votre Nom"
    git config --global user.email "votre.email@example.com"
    ```

3.  **Vérifiez l'installation** :

    ```bash
    git --version
    ```

### **Visual Studio Code : Votre Espace de Travail** 📝

[cite\_start]**C'est quoi ?** C'est un éditeur de code, ou plus précisément un Environnement de Développement Intégré (IDE)[cite: 602]. [cite\_start]Il offre des fonctionnalités avancées (coloration syntaxique, auto-complétion, débogage) qui rendent le codage beaucoup plus facile et efficace[cite: 602].

1.  **Installez VS Code** :
    [cite\_start]Téléchargez et installez le paquet `.deb` pour Ubuntu depuis le site officiel : **[https://code.visualstudio.com/](https://code.visualstudio.com/)**[cite: 611].

2.  **Ajoutez des extensions utiles** :
    Les extensions ajoutent des fonctionnalités à VS Code. [cite\_start]Ouvrez l'onglet "Extensions" (l'icône des 4 carrés) [cite: 619] et installez les extensions suivantes pour une meilleure expérience avec Symfony :

      * [cite\_start]**PHP Intelephense** [cite: 620]
      * [cite\_start]**Symfony for VSCode** [cite: 623]
      * [cite\_start]**Twig** [cite: 625]
      * [cite\_start]**Twig Language 2** [cite: 626]

### **MySQL : La Base de Données** 🗃️

[cite\_start]**C'est quoi ?** C'est le Système de Gestion de Base de Données (SGBD) que nous utiliserons[cite: 629]. C'est là que notre application stockera toutes ses données persistantes, comme les utilisateurs, les produits, les articles, etc.

1.  **Installez le serveur MySQL** :
    ```bash
    sudo apt install mysql-server
    ```

### **Symfony CLI : L'Assistant Symfony** ✨

[cite\_start]**C'est quoi ?** C'est un outil en ligne de commande créé spécialement pour Symfony[cite: 633]. [cite\_start]Il simplifie de nombreuses tâches comme la création d'un nouveau projet, le lancement d'un serveur web local optimisé pour le développement, et la vérification de la configuration de votre machine[cite: 633, 635, 637].

1.  **Installez la Symfony CLI** :
    [cite\_start]Exécutez les commandes suivantes pour ajouter le dépôt de Symfony et installer l'outil[cite: 641, 642].

    ```bash
    curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | sudo -E bash
    ```

    ```bash
    sudo apt install symfony-cli
    ```

2.  **Vérifiez votre configuration** :
    [cite\_start]La CLI inclut un outil très pratique pour vérifier que votre machine répond à toutes les exigences de Symfony[cite: 643]. Exécutez-le :

    ```bash
    symfony check:requirements
    ```

-----

Félicitations \! 🎉 Votre environnement de développement est maintenant entièrement configuré et prêt pour commencer à créer des applications avec Symfony. Dans le prochain atelier, nous utiliserons ces outils pour créer notre premier projet.
