# **Atelier 01 : Préparation de l'Environnement de Développement Symfony**

Bienvenue dans cet atelier initial \! 🚀 Avant de plonger dans le code et de construire notre première application, il est essentiel de préparer notre machine. Cet atelier vous guidera pas à pas pour installer et configurer tous les outils nécessaires au développement avec le framework **Symfony**.

## **Introduction : Le Concept de Framework**

**C'est quoi ?** Un **framework** (ou "cadre de travail") est une structure logicielle complète qui fournit un ensemble de composants, de bibliothèques et de conventions pour développer des applications. Il agit comme une "boîte à outils"  qui standardise le processus de développement.

**Pourquoi l'utiliser ?**

  * **Rapidité** : On se concentre sur la logique métier spécifique de l'application, pas sur les tâches techniques récurrentes (comme la gestion des requêtes HTTP ou l'accès aux données).
  * **Qualité et Maintenabilité** : Il impose des normes de développement et une architecture claire (comme le **MVC**), ce qui rend le code plus propre et plus facile à maintenir, même en équipe.

-----

## **Étape 1 : Le Moteur et son Gestionnaire**

### **1. PHP : Le Moteur du Framework** ⚙️

PHP est le langage de programmation côté serveur sur lequel Symfony est construit. C'est lui qui exécute le code de votre application.

**Tâches à réaliser :**

1.  **Mise à jour du système (pour Ubuntu)** :

    ```bash
    sudo apt update
    sudo apt upgrade
    ```

2.  **Installation de PHP et ses extensions** :
    Symfony 7 requiert PHP 8.1 ou supérieur. Nous installons les extensions courantes (`xml`, `pdo-mysql`) nécessaires au bon fonctionnement des composants de base.

    ```bash
    sudo apt install php php-xml php-pdo php-mysql
    ```

3.  **Vérification** :

    ```bash
    php --version
    ```

      * **Documentation Officielle PHP** : [https://www.php.net/](https://www.php.net/)

### **2. Composer : Le Gestionnaire de Dépendances** 📦

**C'est quoi ?** Composer est l'outil indispensable pour **télécharger, installer et gérer** toutes les bibliothèques et les paquets externes dont Symfony a besoin pour fonctionner. Il évite de télécharger manuellement chaque composant.

**Tâches à réaliser :**

1.  **Installation de Composer (globale)** :

    ```bash
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"
    sudo mv composer.phar /usr/local/bin/composer
    ```

2.  **Vérification** :

    ```bash
    composer --version
    ```

      * **Documentation Officielle Composer** : [https://getcomposer.org/](https://getcomposer.org/)

-----

## **Étape 2 : Les Outils de Projet et de Base de Données**

### **3. Symfony CLI : L'Assistant en Ligne de Commande** ✨

**C'est quoi ?** C'est un outil développé par l'équipe Symfony. Il est essentiel pour simplifier les tâches comme la **création de projets**, la **gestion des certificats SSL** et, surtout, le lancement d'un **serveur web local** optimisé pour le développement.

**Tâches à réaliser :**

1.  **Installation de la CLI (pour Ubuntu)** :

    ```bash
    curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | sudo -E bash
    sudo apt install symfony-cli
    ```

2.  **Vérification de l'environnement** :
    Cet outil permet de vérifier que votre machine respecte toutes les exigences logicielles de Symfony.

    ```bash
    symfony check:requirements
    ```

      * **Documentation Officielle Symfony CLI** : [https://symfony.com/download](https://symfony.com/download)

### **4. Git : Le Contrôle de Version** Git

**C'est quoi ?** Git est le système de contrôle de version le plus populaire. Il vous permet d'enregistrer l'historique de toutes les modifications de votre code, de revenir à des versions antérieures et de collaborer efficacement sur des projets.

**Tâches à réaliser :**

1.  **Installation et configuration** :
    ```bash
    sudo apt install git
    git config --global user.name "Votre Prénom Nom"
    git config --global user.email "votre.email@etu.univ.tn"
    ```
      * **Documentation Officielle Git** : [https://git-scm.com/doc](https://git-scm.com/doc)

### **5. MySQL : Le SGBD** 🗃️

**C'est quoi ?** C'est le système de gestion de base de données que nous utiliserons pour stocker les informations de notre application (produits, utilisateurs, etc.).

**Tâches à réaliser :**

1.  **Installation du serveur MySQL** :
    ```bash
    sudo apt install mysql-server
    ```
      * **Documentation Officielle MySQL** : [https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)

-----

## **Étape 3 : Création du Projet et Démarrage**

1.  **Créez votre premier projet Symfony** :
    Nous utilisons la CLI de Symfony et le modèle `webapp` qui inclut les paquets nécessaires pour une application web classique (routes, Twig, etc.).

    ```bash
    symfony new MaBoutique --webapp
    ```

2.  **Accédez au dossier du projet** :

    ```bash
    cd MaBoutique
    ```

3.  **Démarrez le serveur de développement** :
    C'est la commande la plus importante pour le développement. Elle lance un serveur web sur votre machine.

    ```bash
    symfony serve
    ```

      * **Testez** : Ouvrez votre navigateur et accédez à **`http://127.0.0.1:8000`**. Vous devriez voir la page de bienvenue de Symfony.

4.  **Arrêtez le serveur** :
    Quand vous avez fini de travailler, arrêtez le serveur avec :

    ```bash
    symfony server:stop
    ```

-----

## **Outil de travail : VS Code** 📝

Bien qu'il ne soit pas strictement nécessaire pour Symfony, l'utilisation d'un éditeur de code moderne comme **Visual Studio Code** (VS Code) est fortement recommandée.

**Pourquoi ?**

  * **Coloration Syntaxique** : Rend le code plus lisible.
  * **IntelliSense/Auto-complétion** : Propose des suggestions de code basées sur le framework.

**Recommandation d'extensions :**

  * **PHP Intelephense** : Support avancé pour PHP.
  * **Symfony for VSCode** : Aide à la complétion des routes, services et templates Twig.
  * **Twig Language 2** : Meilleur support pour le langage de template Twig.
      * **Téléchargement VS Code** : [https://code.visualstudio.com/](https://code.visualstudio.com/)

-----

### **Synthèse des Outils et de leurs Rôles**

| Outil | Rôle Principal | Lien de Documentation |
| :--- | :--- | :--- |
| **PHP** | Moteur qui exécute le code côté serveur. | [https://www.php.net/](https://www.php.net/) |
| **Composer** | Gestionnaire des dépendances et des librairies. | [https://getcomposer.org/](https://getcomposer.org/) |
| **Symfony CLI** | Assistant pour créer des projets et lancer le serveur web local. | [https://symfony.com/download](https://symfony.com/download) |
| **Git** | Système de contrôle de version. | [https://git-scm.com/doc](https://git-scm.com/doc) |
| **MySQL** | Stockage des données de l'application. | [https://dev.mysql.com/doc/](https://dev.mysql.com/doc/) |
