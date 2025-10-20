## Tuto : Installation de MySQL sur Ubuntu 24.04 et Gestion des Utilisateurs

Voici les étapes pour installer MySQL Server sur votre système Ubuntu 24.04, configurer un utilisateur administrateur, et gérer les connexions.

-----

## 1\. Installation de MySQL Server

Ouvrez un terminal et suivez ces étapes :

1.  **Mise à jour des paquets :**

    ```bash
    sudo apt update
    ```

2.  **Installation de MySQL Server :**

    ```bash
    sudo apt install mysql-server -y
    ```

3.  **Vérification de l'état (Optionnel) :**

    ```bash
    sudo systemctl status mysql
    # Le service devrait être "active (running)"
    ```

-----

## 2\. Connexion en tant que Super-utilisateur (Root)

Par défaut sur Ubuntu, l'utilisateur **root** de MySQL est authentifié via le plugin `auth_socket`. Cela signifie que l'utilisateur système qui utilise `sudo` peut se connecter sans mot de passe.

  * **Connexion Super-utilisateur (root) :**
    ```bash
    sudo mysql
    ```
    Vous êtes maintenant dans l'invite MySQL (`mysql>`).

-----

## 3\. Création de l'Utilisateur Admin avec Mot de Passe

Pour permettre la connexion à MySQL avec un mot de passe (nécessaire pour la plupart des applications externes), vous devez créer un nouvel utilisateur ou modifier le compte `root`. Il est **recommandé** de créer un nouvel utilisateur administrateur.

1.  **Créez l'utilisateur et définissez un mot de passe fort :**
    (Remplacez `mon_admin` et `mot_de_passe_fort`)

    ```sql
    CREATE USER 'mon_admin'@'localhost' IDENTIFIED WITH mysql_native_password BY 'mot_de_passe_fort';
    ```

2.  **Accordez tous les privilèges à cet utilisateur :**
    L'option `*.*` signifie "toutes les bases de données et toutes les tables". `WITH GRANT OPTION` permet à cet utilisateur de créer d'autres utilisateurs et d'accorder des privilèges.

    ```sql
    GRANT ALL PRIVILEGES ON *.* TO 'mon_admin'@'localhost' WITH GRANT OPTION;
    ```

3.  **Actualisez les privilèges :**

    ```sql
    FLUSH PRIVILEGES;
    ```

4.  **Quittez la console MySQL :**

    ```sql
    EXIT;
    ```

-----

## 4\. Connexion en tant qu'Utilisateur Normal (Admin créé)

Vous vous connectez maintenant en utilisant le nom d'utilisateur et le mot de passe que vous venez de définir.

  * **Connexion en tant qu'utilisateur `mon_admin` :**
    ```bash
    mysql -u mon_admin -p
    ```
    Le système vous demandera d'entrer le `mot_de_passe_fort`.

-----

## 5\. Exemple : Création d'un Utilisateur à Droits Limités

Pour une base de données de production, vous ne devriez jamais utiliser l'utilisateur admin. Voici comment créer un utilisateur spécifique pour une seule base de données :

1.  **Connectez-vous avec votre utilisateur admin (`mon_admin`) :**

    ```bash
    mysql -u mon_admin -p
    ```

2.  **Créez la base de données et l'utilisateur limité :**

    ```sql
    CREATE DATABASE ma_base_projet;
    CREATE USER 'user_projet'@'localhost' IDENTIFIED BY 'mdp_projet';
    ```

3.  **Accordez les droits limités sur la base de données spécifique :**
    Ici, on donne les droits de lecture, écriture, modification et suppression sur la base `ma_base_projet` uniquement.

    ```sql
    GRANT SELECT, INSERT, UPDATE, DELETE ON ma_base_projet.* TO 'user_projet'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;
    ```

<!-- end list -->

  * **Connexion en tant qu'utilisateur limité :**
    ```bash
    mysql -u user_projet -p
    ```
    Cet utilisateur pourra travailler uniquement dans la base de données `ma_base_projet`.
