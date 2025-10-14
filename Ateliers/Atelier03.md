# **Atelier Symfony : Les Bases de Données avec Doctrine ORM**

Bienvenue dans cet atelier crucial \! Jusqu'à présent, nous avons affiché des données statiques (des tableaux PHP). Maintenant, nous allons connecter notre application à une **base de données MySQL** et utiliser **Doctrine**, l'ORM par défaut de Symfony, pour gérer, stocker et récupérer nos produits de manière persistante.

## **Objectifs de l'Atelier**

  * Configurer la connexion à la base de données.
  * Comprendre le rôle de l'**ORM Doctrine**.
  * Créer une **Entité** (le **Modèle** de la MVC).
  * Générer et exécuter des **Migrations** (synchronisation BDD).
  * Récupérer et afficher les données réelles.

-----

## **Partie 1 : Configuration et Connexion à la Base de Données**

La première étape est d'établir la communication entre Symfony et votre serveur MySQL.

#### **Tâches à réaliser :**

1.  **Création de la base de données** :
    Si ce n'est pas déjà fait, créez une base de données vide nommée `ma_boutique_db` sur votre serveur MySQL.

    ```sql
    CREATE DATABASE ma_boutique_db;
    ```

2.  **Configuration du fichier `.env`** :
    Ouvrez le fichier **`.env`** à la racine de votre projet. Ce fichier contient les variables d'environnement.

      * **Localisez** et **modifiez** la ligne `DATABASE_URL` avec vos identifiants MySQL locaux (utilisateur et mot de passe).

    <!-- end list -->

    ```dotenv
    # .env
    DATABASE_URL="mysql://root:mot_de_passe_mysql@127.0.0.1:3306/ma_boutique_db?serverVersion=8.0&charset=utf8mb4"
    ```

    *(**Note** : Remplacez `mot_de_passe_mysql` par le mot de passe de votre utilisateur `root` ou d'un utilisateur dédié.)*

3.  **Vérification de la connexion** :
    Nous utilisons l'outil Doctrine pour vérifier que la configuration fonctionne.

    ```bash
    php bin/console doctrine:database:create
    # Si la base de données est déjà là, l'outil vous le confirmera.
    ```

-----

## **Partie 2 : Création de l'Entité (Le Modèle MVC)**

**Doctrine ORM** (Object-Relational Mapping) fait le pont entre vos classes PHP (les objets) et les tables SQL (le relationnel). Une **Entité** est une classe PHP qui représente une table de la base de données.

#### **Tâches à réaliser :**

1.  **Générez l'Entité `Product`** :
    Utilisez la commande `make:entity` pour créer la classe et le *Repository* associés à notre table de produits.

    ```bash
    php bin/console make:entity Product
    ```

2.  **Définissez les champs (Propriétés)** :
    L'assistant vous demandera de définir les propriétés de l'Entité. C'est à ce moment que vous définissez le **schéma** de votre future table SQL.

    | Nom du Champ | Type (choix) | Longueur | Nullable ? |
    | :--- | :--- | :--- | :--- |
    | `name` | `string` | `255` | `no` |
    | `price` | `float` | | `no` |
    | `description` | `text` | | `yes` |
    | `image` | `string` | `255` | `yes` |

3.  **Analyse du code généré** :
    Ouvrez `src/Entity/Product.php`.

      * **Attributs PHP** : Remarquez les lignes commençant par `#[]` (ex : `#[\ORM\Column(...)]`). Ces annotations de Doctrine (appelées **métadonnées**) décrivent la table SQL correspondante.

      * **Getters/Setters** : Les méthodes publiques générées (`getName()`, `setName()`) sont les seuls moyens d'accéder aux propriétés privées de l'objet, respectant les principes de la POO.

      * **Documentation Doctrine Entités** : [https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html)

-----

## **Partie 3 : Synchronisation (Migrations)**

Les **Migrations** sont des fichiers de code qui permettent à Doctrine de générer les requêtes SQL nécessaires pour créer ou modifier les tables de votre base de données, en fonction des changements apportés à vos Entités. C'est le moyen le plus sûr de maintenir votre base de données à jour.

#### **Tâches à réaliser :**

1.  **Générez le fichier de Migration** :
    Cette commande compare l'état des Entités (PHP) avec l'état de la base de données (SQL) et génère le script de modification.

    ```bash
    php bin/console make:migration
    ```

      * **Observation** : Un fichier est créé dans le dossier `migrations/`. Vous pouvez l'ouvrir pour voir le SQL qui sera exécuté (`CREATE TABLE product...`).

2.  **Exécutez la Migration** :
    Cette commande applique le script SQL généré à votre base de données.

    ```bash
    php bin/console doctrine:migrations:migrate
    ```

      * **Vérifiez** : Confirmez la création de la table `product` en vous connectant à votre outil de base de données (PhpMyAdmin, etc.).

3.  **Insérez des données de test** :
    Insérez quelques produits directement dans la table `product` via votre outil de base de données ou en ligne de commande SQL. C'est le contenu réel que nous allons afficher.

    ```sql
    INSERT INTO product (name, price, description, image) VALUES
    ('Clavier Mécanique', 120.00, 'Un clavier idéal pour coder.', 'images/keyboard.jpg'),
    ('Écran 4K', 450.50, 'Écran haute résolution pour le graphisme.', 'images/monitor.jpg'),
    ('Souris Gamer', 75.99, 'Haute précision et ergonomie.', 'images/mouse.jpg');
    ```

      * **Documentation Doctrine Migrations** : [https://www.doctrine-project.org/projects/doctrine-migrations/en/latest/index.html](https://www.google.com/search?q=https://www.doctrine-project.org/projects/doctrine-migrations/en/latest/index.html)

-----

## **Partie 4 : Affichage des Données Persistantes**

Nous allons modifier le contrôleur pour qu'il utilise le **Repository** de Doctrine, l'outil dédié à la récupération des Entités.

#### **Tâches à réaliser :**

1.  **Injectez le Repository dans le Contrôleur** :
    Ouvrez `src/Controller/ProductController.php`. Modifiez la signature de la méthode `index()` pour y ajouter le `ProductRepository` comme argument. Symfony, grâce à son système d'**Injection de Dépendances (DI)**, le fournit automatiquement.

2.  **Récupérez les données avec Doctrine** :
    Remplacez la récupération du tableau statique par l'appel à la méthode `findAll()` du Repository.

    ```php
    // src/Controller/ProductController.php

    use App\Repository\ProductRepository; // N'oubliez pas l'import
    use Symfony\Component\HttpFoundation\Response;
    use Symfony\Component\Routing\Annotation\Route;

    #[Route('/products', name: 'app_products_index')]
    public function index(ProductRepository $productRepository): Response
    {
        // findAll() est une méthode standard de Doctrine qui retourne TOUTES les Entités Product
        $products = $productRepository->findAll();

        return $this->render('product/index.html.twig', [
            'products_list' => $products, // $products est maintenant une collection d'objets Product
        ]);
    }
    ```

3.  **Vérifiez la Vue Twig** :
    Ouvrez `templates/product/index.html.twig`.

      * **Observation importante** : La syntaxe Twig reste inchangée \!
        `{{ product.name }}` continue de fonctionner car Twig appelle automatiquement la méthode **`getName()`** de l'objet `Product` (`$product->getName()`). C'est l'un des avantages de l'ORM.

4.  **Testez l'application** :
    Démarrez votre serveur (`symfony serve`) et accédez à `/products`. La liste affichée doit correspondre exactement aux données que vous avez insérées dans votre table MySQL.

-----

### **Exercice de Consolidation : Affichage du Détail**

Adaptez la méthode `show()` de votre contrôleur pour qu'elle récupère un seul produit depuis la base de données, garantissant ainsi que toute l'application fonctionne avec les données persistantes.

1.  **Mettez à jour la méthode `show()`** :

    ```php
    // src/Controller/ProductController.php

    #[Route('/product/{id<\d+>}', name: 'app_product_show')]
    public function show(int $id, ProductRepository $productRepository): Response
    {
        // find() retourne l'objet Product correspondant à l'ID ou null si non trouvé
        $product = $productRepository->find($id);

        if (!$product) {
            // Méthode simple de Symfony pour lever une erreur 404
            throw $this->createNotFoundException('Le produit demandé n\'existe pas !');
        }

        return $this->render('product/show.html.twig', [
            'product_item' => $product,
        ]);
    }
    ```

2.  **Testez l'application** :
    Naviguez de la liste au détail. L'ensemble de la chaîne (Contrôleur -\> Repository/Doctrine -\> Base de Données -\> Twig) est maintenant fonctionnel.
