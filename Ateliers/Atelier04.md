# **Atelier 04 : Relations, Requêtes Avancées (DQL) et Services**

Cet atelier va vous permettre de modéliser des données complexes (relations un à plusieurs), d'exécuter des requêtes personnalisées via Doctrine Query Language (**DQL**) et de comprendre le fonctionnement des **Services** et de l'**Injection de Dépendances (DI)**, qui sont au cœur de Symfony.

## **Objectifs de l'Atelier**

  * Maîtriser les relations **One-To-Many / Many-To-One** avec Doctrine.
  * Créer des **méthodes personnalisées** dans le Repository (méthodes de consultation).
  * Rédiger des requêtes avancées en **DQL** (Doctrine Query Language).
  * Comprendre et appliquer le concept de **Service** et d'**Injection de Dépendances**.

-----

## **Partie 1 : Modélisation des Relations (Catégories)**

Nous allons enrichir notre application en créant une Entité `Category` et en la liant à notre Entité `Product` (relation **Un à Plusieurs** : une catégorie a plusieurs produits, un produit appartient à une seule catégorie).

#### **Tâches à réaliser :**

1.  **Générez l'Entité `Category`** :

    ```bash
    php bin/console make:entity Category
    ```

      * Ajoutez le champ `name` de type `string` (longueur 255).
      * Laissez l'assistant générer l'ID.

2.  **Créez la relation One-To-Many / Many-To-One** :
    Utilisez à nouveau la commande `make:entity` pour modifier l'Entité `Product` et ajouter la relation.

    ```bash
    php bin/console make:entity Product
    ```

      * Répondez aux questions :
          * Nouveau champ : `category`
          * Type de champ : choisissez `relation`
          * À quelle Entité est-elle liée ? `Category`
          * Type de relation : choisissez `ManyToOne` (Plusieurs `Product` vers Une `Category`)
          * **Nom du champ sur `Category`** : `products` (Doctrine crée une collection d'objets `Product` dans l'Entité `Category`).

3.  **Analysez les Entités** :

      * Ouvrez `src/Entity/Category.php` : Il doit contenir une propriété `$products` de type `Collection` avec des méthodes `addProduct()` et `removeProduct()`.
      * Ouvrez `src/Entity/Product.php` : Il doit contenir une propriété `$category` de type `Category` avec les getters/setters habituels.

4.  **Générez et Exécutez la Migration** :

    ```bash
    php bin/console make:migration
    # Une nouvelle migration est générée pour ajouter la colonne 'category_id' à la table 'product'.
    php bin/console doctrine:migrations:migrate
    ```

5.  **Mettez à jour les données** :
    Allez dans votre base de données, ajoutez quelques catégories dans la table `category` et mettez à jour la colonne `category_id` de quelques produits dans la table `product` pour créer les liens.

-----

## **Partie 2 : Requêtes Personnalisées (DQL)**

Dans le cadre du M de MVC, le **Repository** est l'endroit idéal pour centraliser toutes les requêtes de lecture de données complexes. Nous allons écrire notre première requête personnalisée en **DQL** (Doctrine Query Language).

**C'est quoi le DQL ?** C'est un langage de requête orienté objet. Au lieu d'écrire du SQL avec les noms de tables et de colonnes (`SELECT * FROM product p`), vous utilisez les noms d'Entités et de propriétés (`SELECT p FROM App\Entity\Product p`).

#### **Tâches à réaliser :**

1.  **Créez une méthode dans le Repository** :
    Ouvrez `src/Repository/ProductRepository.php`. Ajoutez une méthode pour récupérer les 3 derniers produits, triés par ID décroissant, et joindre la catégorie pour éviter les requêtes multiples (problème **N+1**).

    ```php
    // src/Repository/ProductRepository.php

    /**
     * @return Product[] Returns an array of the 3 latest Product objects
     */
    public function findLatestProductsWithCategory(int $limit = 3): array
    {
        return $this->createQueryBuilder('p') // Alias 'p' pour l'Entité Product
            ->leftJoin('p.category', 'c')    // Jointure sur la propriété 'category', alias 'c'
            ->addSelect('c')                 // Ajoute l'objet Category aux résultats
            ->orderBy('p.id', 'DESC')
            ->setMaxResults($limit)
            ->getQuery()
            ->getResult()
        ;
    }
    ```

2.  **Appelez la méthode dans le Contrôleur** :
    Créez une méthode simple dans `ProductController` pour tester votre nouvelle requête.

    ```php
    // src/Controller/ProductController.php

    // ...

    #[Route('/', name: 'app_homepage')]
    public function homepage(ProductRepository $productRepository): Response
    {
        $latestProducts = $productRepository->findLatestProductsWithCategory(3);

        return $this->render('product/homepage.html.twig', [
            'latestProducts' => $latestProducts,
        ]);
    }
    ```

3.  **Affichez les données avec la relation** :
    Dans une nouvelle vue `templates/product/homepage.html.twig`, affichez les produits. L'accès à la catégorie est direct car elle a été jointe à l'objet.

    ```twig
    {# templates/product/homepage.html.twig #}

    {% extends 'base.html.twig' %}

    {% block title %}Accueil - Derniers produits{% endblock %}

    {% block body %}
    <div class="container">
        <h1>Nos 3 dernières nouveautés</h1>
        {% for product in latestProducts %}
            <div class="card my-2">
                <div class="card-body">
                    <h5 class="card-title">{{ product.name }}</h5>
                    <p class="card-subtitle mb-2 text-muted">
                        Catégorie : {{ product.category.name }} {# Accès direct à l'objet Category lié #}
                    </p>
                    <p>{{ product.description }}</p>
                </div>
            </div>
        {% endfor %}
    </div>
    {% endblock %}
    ```

-----

## **Partie 3 : Services et Injection de Dépendances**

L'un des concepts les plus importants de Symfony est l'**Injection de Dépendances (DI)**, gérée par le **Conteneur de Services**.

**C'est quoi un Service ?** 🛠️
Dans Symfony, un **Service** est simplement un objet PHP qui effectue une **tâche spécifique et réutilisable** (ex: gérer l'envoi d'emails, calculer des taxes, formater des données). Tous les objets qui ne sont pas des Entités (Modèle) ou des Contrôleurs sont généralement des Services.

**C'est quoi l'Injection de Dépendances (DI) ?** 💉
C'est le principe selon lequel un objet (par exemple, un Contrôleur) ne crée pas les objets dont il a besoin (ses **dépendances**), mais les reçoit de l'extérieur (du **Conteneur de Services**). C'est pourquoi vous pouvez simplement demander `$productRepository` ou `$entityManager` en paramètre de vos méthodes.

#### **Tâches à réaliser :**

1.  **Générez un Service de Calcul** :
    Créez un service simple pour calculer le prix avec TVA, une logique métier qui ne devrait pas être dans le contrôleur.

          * il suffit de créer le fichier `src/Service/PriceCalculator.php` dans symfony la classe PriceCalculator sera considérée automatiquement comme un service.

2.  **Implémentez la logique dans le Service** :
    Dans `PriceCalculator.php`, définissez une constante pour la TVA et créez une méthode de calcul.

    ```php
    // src/Service/PriceCalculator.php

    namespace App\Service;

    class PriceCalculator
    {
        private const TVA = 0.20; // Taux de TVA à 20%

        public function calculatePriceWithTVA(float $priceHT): float
        {
            return $priceHT * (1 + self::TVA);
        }

        public function getTvaRate(): float
        {
            return self::TVA * 100;
        }
    }
    ```

3.  **Injectez et utilisez le Service dans le Contrôleur** :
    Modifiez la méthode `show()` du `ProductController` pour injecter et utiliser le service de calcul de prix.

    ```php
    // src/Controller/ProductController.php (méthode show)

    use App\Service\PriceCalculator; // N'oubliez pas l'import

    #[Route('/product/{id<\d+>}', name: 'app_product_show')]
    public function show(
        int $id, 
        ProductRepository $productRepository,
        PriceCalculator $calculator // Injection du Service !
    ): Response
    {
        $product = $productRepository->find($id);

        if (!$product) {
            throw $this->createNotFoundException('Le produit demandé n\'existe pas !');
        }
        
        // Utilisation du Service
        $priceTTC = $calculator->calculatePriceWithTVA($product->getPrice());
        $tvaRate = $calculator->getTvaRate();

        return $this->render('product/show.html.twig', [
            'product_item' => $product,
            'price_ttc' => $priceTTC,
            'tva_rate' => $tvaRate,
        ]);
    }
    ```

4.  **Affichez les résultats dans la Vue** :
    Mettez à jour `templates/product/show.html.twig` pour afficher les informations calculées par le Service.

    ```twig
    {# templates/product/show.html.twig #}

    {# ... au niveau de l'affichage du prix ... #}
    <p>Prix Hors Taxes : <strong>{{ product_item.price|number_format(2, ',', ' ') }} €</strong></p>
    <p>Taux de TVA : {{ tva_rate }}%</p>
    <h3>Prix Toutes Taxes Comprises : <strong>{{ price_ttc|number_format(2, ',', ' ') }} €</strong></h3>
    ```

5.  **Conclusion** : Le Contrôleur a délégué la logique métier de calcul au Service, ce qui le rend plus léger et permet de réutiliser cette logique partout ailleurs dans l'application.

-----

### **Ressources et Liens Officiels**

| Concept | Description | Documentation Officielle |
| :--- | :--- | :--- |
| **Relations Doctrine** | Modéliser les liens entre les Entités (OneToOne, OneToMany, etc.). | [https://symfony.com/doc/current/doctrine.html\#establishing-relationships](https://www.google.com/search?q=https://symfony.com/doc/current/doctrine.html%23establishing-relationships) |
| **Doctrine Query Language (DQL)** | Langage de requête orienté objet pour les requêtes avancées. | [https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html](https://www.google.com/search?q=https://www.doctrine-project.com/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html) |
| **Injection de Dépendances (DI)** | Principe d'injecter des Services dans les classes qui en ont besoin. | [https://symfony.com/doc/current/service\_container.html](https://symfony.com/doc/current/service_container.html) |
| **Services** | Classes qui effectuent des tâches spécifiques et réutilisables. | [https://symfony.com/doc/current/service\_container.html\#creating-a-service](https://www.google.com/search?q=https://symfony.com/doc/current/service_container.html%23creating-a-service) |
