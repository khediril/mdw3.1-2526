# **Atelier Symfony : Création d'une API RESTful**

Cet atelier vous apprend à déconnecter votre logique métier (Contrôleurs, Services, Doctrine) de l'interface utilisateur (Twig). L'objectif est de transformer votre application en une **Interface de Programmation Applicative (API)** qui communique en utilisant le format standard du web : **JSON**.

## **Objectifs de l'Atelier**

  * Comprendre le concept et l'utilité d'une **API REST**.
  * Maîtriser le rôle du format de données **JSON**.
  * Utiliser les réponses **`JsonResponse`** pour servir des données.
  * Utiliser le composant **Serializer** de Symfony pour convertir les Entités Doctrine en JSON.
  * Créer des *endpoints* API pour la lecture de données (`GET`).

-----

## **Partie 1 : Introduction à l'API et au Format JSON**

### **1. Qu'est-ce qu'une API REST ?** 🌐

Une **API REST** (Representational State Transfer) est un ensemble de règles et de conventions pour concevoir des services web. Contrairement à une application web classique qui retourne du HTML (une page), une API REST retourne des **données brutes** (du texte) via des URLs spécifiques (*endpoints*).

| Application Web (Classique) | API RESTful |
| :--- | :--- |
| **Retourne** : HTML, CSS, JS (une interface complète). | **Retourne** : JSON, XML (des données brutes). |
| **But** : Afficher une page à l'utilisateur. | **But** : Fournir des données à une autre application. |
| **Méthodes HTTP** : Généralement `GET` et `POST`. | **Méthodes HTTP** : Toutes (`GET`, `POST`, `PUT`, `DELETE`). |

### **2. Le Rôle du JSON (JavaScript Object Notation)** 📊

**C'est quoi ?** JSON est le format de données standard du web moderne. C'est un format textuel léger, facile à lire pour les humains et facile à analyser pour les machines.

**Structure (Rappel) :** Les données sont stockées sous forme de paires **clé-valeur** (comme un tableau associatif en PHP ou un objet en JavaScript).

```json
{
  "id": 1,
  "name": "Clavier Mécanique",
  "price": 120.00,
  "category": {
    "id": 5,
    "name": "Périphériques"
  }
}
```

**Rôle :** JSON est le **langage commun** pour échanger des données entre le serveur (votre API Symfony) et le client (une application mobile ou un frontend React/Angular/Vue). Votre serveur "série" (transforme) les objets PHP en JSON, et le client "désérialise" (transforme) le JSON en objet JavaScript.

-----

## **Partie 2 : Préparation du Projet et Serialisation**

Pour créer facilement des réponses JSON à partir de vos Entités Doctrine, nous allons installer le composant dédié de Symfony.

#### **Tâches à réaliser :**

1.  **Installez le composant Serializer** :

    ```bash
    composer require symfony/serializer
    ```

2.  **Créez un Contrôleur dédié à l'API** :
    Créez un nouveau contrôleur pour organiser vos *endpoints* API, en utilisant un préfixe de route spécifique (`/api`).

    ```bash
    php bin/console make:controller ApiProductController
    ```

    ```php
    // src/Controller/ApiProductController.php

    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\Routing\Attribute\Route;

    #[Route('/api/products', name: 'app_api_product_')]
    class ApiProductController extends AbstractController
    {
        // ...
    }
    ```

-----

## **Partie 3 : Création des Endpoints de Lecture (GET)**

Nous allons créer deux *endpoints* pour lire les données : lister tous les produits et afficher un produit par ID.

### **1. Endpoint de Listing des Produits (`GET /api/products`)**

Nous allons utiliser le service **`SerializerInterface`** de Symfony pour convertir nos objets `Product` en JSON.

#### **Tâches à réaliser :**

1.  **Créez la méthode `index()` dans `ApiProductController`** :
    Injectez le `ProductRepository` et le `SerializerInterface`.

    ```php
    // src/Controller/ApiProductController.php

    use App\Repository\ProductRepository;
    use Symfony\Component\HttpFoundation\Response;
    use Symfony\Component\Serializer\SerializerInterface; // Import clé

    #[Route('', name: 'index', methods: ['GET'])]
    public function index(ProductRepository $productRepository, SerializerInterface $serializer): Response
    {
        // 1. Récupération des Entités
        $products = $productRepository->findAll();

        // 2. Serialisation : Convertit le tableau d'objets Product en chaîne JSON
        // Le deuxième argument 'json' est le format de sortie souhaité
        $jsonContent = $serializer->serialize($products, 'json', [
            // Option pour ignorer la relation circulaire (Product -> Category -> Products)
            'ignored_attributes' => ['products'], 
        ]);

        // 3. Retourne une réponse HTTP avec le contenu JSON
        // Le type de contenu est automatiquement défini sur application/json
        return new Response($jsonContent, Response::HTTP_OK, [
            'Content-Type' => 'application/json'
        ]);
        
        // Note : On pourrait simplifier en utilisant la classe JsonResponse
        // return $this->json($products); 
    }
    ```

2.  **Testez l'Endpoint** :
    Démarrez votre serveur (`symfony serve`) et accédez à `http://127.0.0.1:8000/api/products`.

      * **Résultat attendu** : Le navigateur affiche une chaîne de caractères JSON, pas de HTML.

### **2. Endpoint de Détail d'un Produit (`GET /api/products/{id}`)**

Nous allons utiliser le **ParamConverter** (comme dans l'atelier CRUD) pour récupérer directement l'objet `Product` correspondant à l'ID de l'URL.

#### **Tâches à réaliser :**

1.  **Créez la méthode `show()`** :

    ```php
    // src/Controller/ApiProductController.php

    use App\Entity\Product;

    #[Route('/{id}', name: 'show', methods: ['GET'])]
    public function show(Product $product, SerializerInterface $serializer): Response
    {
        // ParamConverter a trouvé et injecté l'objet $product

        // Serialisation de l'objet unique
        $jsonContent = $serializer->serialize($product, 'json', [
             'ignored_attributes' => ['products'],
        ]);

        return new Response($jsonContent, Response::HTTP_OK, [
            'Content-Type' => 'application/json'
        ]);
    }
    ```

2.  **Testez l'Endpoint** :
    Accédez à `http://127.0.0.1:8000/api/products/1` (si le produit 1 existe). Vous devriez voir les données de ce produit au format JSON.

-----

## **Partie 4 : Le Concept de Service dans les APIs**

Dans une API, le **Service** est crucial, car il contient la logique métier et les règles de validation, garantissant que les données servies ou reçues sont toujours cohérentes, peu importe si elles proviennent d'une requête API ou d'un formulaire HTML.

**Rappel :** Les services (comme `PriceCalculator` de l'atelier précédent) sont l'endroit idéal pour effectuer des transformations ou des calculs **avant** la sérialisation en JSON.

#### **Tâches à réaliser (Exercice de réflexion) :**

1.  **Réintégrez le calcul TTC** :
    Modifiez la méthode `index()` dans `ApiProductController`. Avant de sérialiser, vous voudriez peut-être que les données JSON contiennent le prix TTC, et non seulement le prix HT stocké en base de données.

      * **Problème** : L'Entité `Product` n'a pas de propriété `$priceTTC`.
      * **Solution** : Ajoutez une méthode à l'Entité `Product` (un **Getter Virtuel**).

2.  **Créez un Getter Virtuel dans l'Entité `Product`** :
    Dans `src/Entity/Product.php`, créez une méthode publique qui utilise votre service (`PriceCalculator`) pour calculer le prix TTC. Pour simplifier ce test, nous allons utiliser une méthode sans l'injection de service (méthode `getPriceTTC` ci-dessous), mais dans un contexte réel, la meilleure pratique serait d'utiliser le service dans l'Entité via l'injection dans le `__construct` (méthode avancée).

    ```php
    // src/Entity/Product.php

    // ...

    // Ce Getter sera inclus dans le JSON par le Serializer
    public function getPriceTTC(): float
    {
        // Pour l'exercice, utilisez une valeur simple (20% de TVA)
        return $this->price * 1.20; 
    }
    ```

3.  **Testez et Observez** :
    Rafraîchissez l'URL `http://127.0.0.1:8000/api/products`.

      * **Résultat** : La réponse JSON inclut maintenant une nouvelle clé `priceTTC` pour chaque produit, même si elle n'existe pas dans la base de données. Le Serializer inclut toutes les méthodes publiques commençant par `get` ou `is`.

-----

### **Ressources et Liens Officiels**

| Concept | Rôle dans l'API | Documentation Officielle |
| :--- | :--- | :--- |
| **JSON Response** | La classe qui permet de retourner une réponse HTTP avec un en-tête `Content-Type: application/json`. | [https://symfony.com/doc/current/controller.html\#returning-json-response](https://www.google.com/search?q=https://symfony.com/doc/current/controller.html%23returning-json-response) |
| **Serializer** | Le composant qui convertit les objets PHP (Entités) en JSON et vice-versa. | [https://symfony.com/doc/current/components/serializer.html](https://symfony.com/doc/current/components/serializer.html) |
| **Requêtes API (Méthodes)** | Utilisation des verbes HTTP (`GET`, `POST`, `PUT`, `DELETE`) pour les actions CRUD. | [https://developer.mozilla.org/fr/docs/Web/HTTP/Methods](https://developer.mozilla.org/fr/docs/Web/HTTP/Methods) |
| **Groupes de Serialisation** | (Avancé) Permet de choisir quelles données inclure ou exclure du JSON (pour éviter les boucles infinies ou masquer des données sensibles). | [https://symfony.com/doc/current/serializer.html\#using-serialization-groups](https://www.google.com/search?q=https://symfony.com/doc/current/serializer.html%23using-serialization-groups) |
