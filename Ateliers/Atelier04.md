# **Atelier Symfony : CRUD et Formulaires Avancés**

Cet atelier fait suite à la mise en place de l'environnement et à la création de votre Entité `Product`. L'objectif est de fournir une interface complète pour **Créer, Modifier et Supprimer (CRUD)** des produits de manière sécurisée et rapide grâce aux **formulaires Symfony**.

## **Objectifs de l'Atelier**

  * Maîtriser le composant **Symfony Forms**.
  * Générer des formulaires liés à une Entité.
  * Implémenter la logique de persistance (Création et Modification) dans le Contrôleur.
  * Sécuriser les actions de suppression avec des jetons **CSRF**.

-----

## **Partie 1 : Création du Formulaire Produit**

Au lieu de gérer le HTML et la validation des champs manuellement, Symfony fournit un composant pour générer des formulaires à partir d'une Entité.

#### **Tâches à réaliser :**

1.  **Générez la classe de Formulaire** :
    Utilisez la console Symfony pour créer le fichier `ProductType.php`. Cette classe définira la structure de notre formulaire.

    ```bash
    php bin/console make:form ProductType
    ```

      * Le fichier `src/Form/ProductType.php` est créé.

2.  **Configurez les champs du Formulaire** :
    Ouvrez `src/Form/ProductType.php`. L'assistant a déjà inclus la plupart des champs (`name`, `price`, `description`, `image`).

      * **Question** : Pourquoi l'ID n'est-il pas inclus ? (*Réponse : L'ID est généré automatiquement par la base de données.*)
      * Pour le champ `description`, modifiez le type de `TextType::class` à `TextareaType::class` pour une meilleure ergonomie. N'oubliez pas l'import `use Symfony\Component\Form\Extension\Core\Type\TextareaType;`.

    <!-- end list -->

    ```php
    // src/Form/ProductType.php

    // ... imports ...
    use Symfony\Component\Form\Extension\Core\Type\TextareaType; 
    use Symfony\Component\Form\Extension\Core\Type\MoneyType; // Pour le prix
    // ...

    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name') // Par défaut, c'est TextType::class
            ->add('price', MoneyType::class, [
                'currency' => 'EUR', // Affichage de la devise
            ])
            ->add('description', TextareaType::class) // Utilisation d'un champ texte multilignes
            ->add('image') // On simule l'URL de l'image pour l'instant
        ;
    }
    // ...
    ```

-----

## **Partie 2 : L'Action de Création (Create)**

Nous allons créer une nouvelle méthode dans notre `ProductController` qui gère l'affichage et le traitement du formulaire.

#### **Tâches à réaliser :**

1.  **Créez la méthode `new()` dans le Contrôleur** :
    Cette méthode est responsable de :
    a. Créer un nouvel objet `Product` vide.
    b. Créer le formulaire en le liant à cet objet.
    c. Gérer la soumission de la requête (`Request`).
    d. Persister l'objet en base de données via **Doctrine**.

    ```php
    // src/Controller/ProductController.php

    use App\Form\ProductType;
    use App\Entity\Product;
    use Doctrine\ORM\EntityManagerInterface; // N'oubliez pas l'import
    use Symfony\Component\HttpFoundation\Request;

    #[Route('/product/new', name: 'app_product_new')]
    public function new(Request $request, EntityManagerInterface $entityManager): Response
    {
        $product = new Product();
        
        // 1. Création du formulaire lié à l'objet $product
        $form = $this->createForm(ProductType::class, $product);

        // 2. Traitement de la requête
        $form->handleRequest($request);

        // 3. Vérification de la soumission et de la validité
        if ($form->isSubmitted() && $form->isValid()) {
            // $product est automatiquement hydraté avec les données du formulaire

            // Persistance (via Doctrine)
            $entityManager->persist($product); // Prépare l'objet
            $entityManager->flush();           // Exécute la requête SQL (INSERT)

            // Redirection vers la liste des produits après succès
            return $this->redirectToRoute('app_products_index');
        }

        // 4. Affichage du formulaire
        return $this->render('product/new.html.twig', [
            'productForm' => $form->createView(),
        ]);
    }
    ```

      * **Injection de Dépendances** : Notez que nous injectons `Request` pour gérer la soumission et `EntityManagerInterface` pour Doctrine (l'outil qui gère les entités et la BDD).

2.  **Créez la vue `new.html.twig`** :
    Ce template affichera le formulaire. La fonction `form_start()`, `form_row()` et `form_end()` de Twig sont des outils très pratiques pour un rendu rapide.

    ```twig
    {# templates/product/new.html.twig #}

    {% extends 'base.html.twig' %}

    {% block title %}Ajouter un nouveau produit{% endblock %}

    {% block body %}
    <div class="container my-5">
        <h1>Créer un nouveau produit</h1>

        {# Début du formulaire, génère la balise <form> avec la méthode POST et l'action #}
        {{ form_start(productForm) }}

        {# Affichage de chaque champ (label, input, erreur) sur une ligne #}
        {{ form_row(productForm.name) }}
        {{ form_row(productForm.price) }}
        {{ form_row(productForm.description) }}
        {{ form_row(productForm.image) }}

        <button type="submit" class="btn btn-success mt-3">Ajouter le produit</button>

        {# Fin du formulaire, génère les champs cachés nécessaires (comme le token CSRF) #}
        {{ form_end(productForm) }}
    </div>
    {% endblock %}
    ```

3.  **Testez la création** :
    Accédez à `/product/new` et ajoutez un produit. Vérifiez qu'il apparaît dans la liste (`/products`).

-----

## **Partie 3 : Mise à Jour (Update)**

La mise à jour utilise presque la même logique que la création, mais nous travaillons sur un objet `Product` existant, récupéré depuis la base de données.

#### **Tâches à réaliser :**

1.  **Créez la méthode `edit()`** :
    Utilisez la fonctionnalité **ParamConverter** de Symfony. En demandant un objet `Product` directement dans la signature de la méthode, Symfony va chercher l'objet en BDD en utilisant la valeur de l'ID passé dans l'URL.

    ```php
    // src/Controller/ProductController.php

    #[Route('/product/edit/{id}', name: 'app_product_edit')]
    public function edit(Request $request, Product $product, EntityManagerInterface $entityManager): Response
    {
        // $product est déjà chargé par Doctrine (grâce à ParamConverter)
        
        // 1. Création du formulaire, lié à l'objet $product existant
        $form = $this->createForm(ProductType::class, $product);

        // 2. Traitement de la requête (Le formulaire est pré-rempli)
        $form->handleRequest($request);

        // 3. Vérification de la soumission et de la validité
        if ($form->isSubmitted() && $form->isValid()) {
            // Doctrine sait que $product a déjà été géré (attaché)
            // L'appel à flush() exécute une requête SQL de type UPDATE
            $entityManager->flush();

            return $this->redirectToRoute('app_products_index');
        }

        // 4. Affichage du formulaire pré-rempli
        return $this->render('product/edit.html.twig', [
            'productForm' => $form->createView(),
            'product_item' => $product,
        ]);
    }
    ```

2.  **Créez la vue `edit.html.twig`** :
    Vous pouvez copier la vue `new.html.twig` et ajuster le titre.

3.  **Liez l'action d'édition dans la vue de détail** :
    Ajoutez un lien vers la page de modification dans `templates/product/show.html.twig`.

    ```twig
    {# templates/product/show.html.twig #}

    {# ... contenu ... #}
    <a href="{{ path('app_product_edit', {id: product_item.id}) }}" class="btn btn-warning mt-3">Modifier le produit</a>
    ```

4.  **Testez la modification** :
    Accédez à la page de détail d'un produit, cliquez sur "Modifier", changez le prix et validez. Vérifiez que la BDD est mise à jour.

-----

## **Partie 4 : Suppression (Delete) et Sécurité CSRF**

La suppression est simple en termes de code Doctrine, mais elle doit être sécurisée pour empêcher les attaques **CSRF (Cross-Site Request Forgery)**.

#### **C'est quoi un jeton CSRF ?** 🛡️

C'est un jeton de sécurité unique et temporaire que Symfony utilise pour s'assurer qu'une requête (comme la suppression) provient bien d'un formulaire valide de votre propre site et non d'un site malveillant.

#### **Tâches à réaliser :**

1.  **Créez la méthode `delete()`** :
    Cette méthode est exécutée lorsqu'un formulaire de suppression (souvent masqué) est soumis.

    ```php
    // src/Controller/ProductController.php

    #[Route('/product/delete/{id}', name: 'app_product_delete', methods: ['POST'])]
    public function delete(Request $request, Product $product, EntityManagerInterface $entityManager): Response
    {
        // 1. Récupération du jeton CSRF soumis par le formulaire
        $submittedToken = $request->request->get('_token');

        // 2. Vérification du jeton (ID du jeton et valeur soumise)
        if ($this->isCsrfTokenValid('delete' . $product->getId(), $submittedToken)) {
            
            // 3. Suppression (via Doctrine)
            $entityManager->remove($product); // Prépare la suppression
            $entityManager->flush();           // Exécute la requête SQL (DELETE)
        }

        // Redirection vers la liste
        return $this->redirectToRoute('app_products_index');
    }
    ```

2.  **Ajoutez le formulaire de suppression à la vue de détail** :
    Dans `templates/product/show.html.twig`, ajoutez un petit formulaire dédié à la suppression (avec la méthode `POST`) pour inclure le jeton CSRF nécessaire.

    ```twig
    {# templates/product/show.html.twig #}

    {# ... après le bouton de modification ... #}

    <form method="post" action="{{ path('app_product_delete', {'id': product_item.id}) }}" style="display:inline-block">
        {# Génère le champ caché _token avec le jeton de sécurité #}
        <input type="hidden" name="_token" value="{{ csrf_token('delete' ~ product_item.id) }}">
        <button type="submit" class="btn btn-danger" onclick="return confirm('Êtes-vous sûr de vouloir supprimer ce produit ?');">Supprimer</button>
    </form>
    ```

3.  **Testez la suppression** :
    Accédez à la page de détail, cliquez sur "Supprimer", confirmez l'action. Le produit doit disparaître de la base de données.

-----

### **Synthèse des Tâches**

| Opération | Contrôleur (Méthode) | Doctrine | Sécurité | Formulaire |
| :--- | :--- | :--- | :--- | :--- |
| **Create** (Ajout) | `new()` | `persist()` et `flush()` | Automatique par le composant Form | `ProductType` lié à `new Product()` |
| **Read** (Lecture) | `index()`, `show()` | `findAll()`, `find()` | N/A | N/A |
| **Update** (Modif.) | `edit()` | `flush()` (sur Entité attachée) | Automatique par le composant Form | `ProductType` lié à `$product` existant |
| **Delete** (Suppr.) | `delete()` | `remove()` et `flush()` | **Manuelle** (`isCsrfTokenValid`) | Formulaire simple avec champ `_token` |

  * **Documentation Symfony Forms** : [https://symfony.com/doc/current/forms.html](https://symfony.com/doc/current/forms.html)
  * **Documentation Doctrine CRUD** : [https://symfony.com/doc/current/doctrine.html\#persisting-objects-to-the-database](https://www.google.com/search?q=https://symfony.com/doc/current/doctrine.html%23persisting-objects-to-the-database)
