# **Atelier 07 : Sécurité, Utilisateurs et Rôles**

Cet atelier se concentre sur l'implémentation des mécanismes de sécurité fondamentaux de Symfony. L'objectif est de sécuriser l'accès à certaines parties de l'application (comme les pages CRUD) et d'apprendre à gérer les utilisateurs et leurs permissions via des **Rôles**.

## **Objectifs de l'Atelier**

  * Maîtriser le composant **Symfony Security**.
  * Générer l'Entité **User** et le système d'authentification.
  * Implémenter le mécanisme de **connexion (Login)** et de **déconnexion (Logout)**.
  * Restreindre l'accès aux pages CRUD en fonction des **Rôles** (ex: `ROLE_ADMIN`).

-----

## **Partie 1 : Mise en Place du Système d'Utilisateur**

Nous allons utiliser la console Symfony pour générer rapidement les éléments de base nécessaires à la gestion des comptes utilisateurs.

#### **Tâches à réaliser :**

1.  **Installez le composant Security** :

    ```bash
    composer require symfony/security-bundle
    ```

2.  **Générez l'Entité `User`** :
    Cette Entité est spéciale car elle implémente l'interface de sécurité de Symfony (`PasswordAuthenticatedUserInterface`).

    ```bash
    php bin/console make:user
    ```

      * **Nom de la classe** : `User` (laissez par défaut).
      * **Nom du champ d'identification** : `email` (recommandé pour les applications modernes).
      * **Voulez-vous stocker les utilisateurs dans la BDD avec Doctrine ?** : `yes`.
      * Laissez l'assistant générer le champ `password`.

3.  **Générez et Exécutez la Migration** :

    ```bash
    php bin/console make:migration
    php bin/console doctrine:migrations:migrate
    ```

      * **Vérifiez** : Votre base de données contient maintenant une table `user` avec les colonnes `id`, `email`, `roles` et `password`.

-----

## **Partie 2 : Authentification (Connexion/Déconnexion)**

L'authentification consiste à permettre à un utilisateur de prouver son identité (connexion) et de mettre fin à sa session (déconnexion).

#### **Tâches à réaliser :**

1.  **Générez le Formulaire de Connexion (Login)** :
    L'assistant `make:authenticator` crée un contrôleur, un template et configure le firewall (le mécanisme de garde-barrière de Symfony).

    ```bash
    php bin/console make:security:form-login
    ```

      * **Voulez-vous créer une classe de *Login Form Authenticator* ?** : `yes`.
      * **Nom de la classe** : `LoginFormAuthenticator` (laissez par défaut).
      * **Nom du Contrôleur** : `SecurityController` (laissez par défaut).

2.  **Analysez le code généré** :

      * **`config/packages/security.yaml`** : Examinez la section `firewalls` qui définit la zone sécurisée, l'URL de connexion (`/login`) et l'URL de déconnexion (`/logout`).
      * **`src/Controller/SecurityController.php`** : Contient les méthodes `login()` et `logout()`. La méthode `login()` est particulièrement simple car Symfony gère la logique lourde du formulaire et de la vérification du mot de passe en arrière-plan.

3.  **Créez un utilisateur Administrateur** :
    Puisque nous n'avons pas encore de page d'inscription, insérez manuellement un utilisateur dans la base de données. Vous devez utiliser le service de **hachage de mot de passe** de Symfony.

      * **Installez le composant de Hachage** :
        ```bash
        composer require symfony/password-hasher
        ```
      * **Créez l'utilisateur via la console (méthode non standard, mais pratique)** :
        Créez un contrôleur ou un service temporaire pour hacher un mot de passe (par exemple `admin123`) et insérez le résultat du hachage dans la colonne `password` de la table `user`. Alternativement, vous pouvez utiliser un **Data Fixture** dans un atelier futur.

4.  **Testez la Connexion** :
    Démarrez le serveur et accédez à `/login`. Connectez-vous avec l'utilisateur que vous venez de créer.

5.  **Testez la Déconnexion** :
    Ajoutez un lien de déconnexion dans votre template de base (`templates/base.html.twig`) pour permettre aux utilisateurs de se déconnecter.

    ```twig
    {# templates/base.html.twig #}
    {% if app.user %}
        <li class="nav-item">
            <a class="nav-link" href="{{ path('app_logout') }}">Déconnexion ({{ app.user.email }})</a>
        </li>
    {% else %}
        <li class="nav-item">
            <a class="nav-link" href="{{ path('app_login') }}">Connexion</a>
        </li>
    {% endif %}
    ```

-----

## **Partie 3 : Restriction d'Accès avec les Rôles**

Le système de sécurité de Symfony est basé sur les **Rôles**. Un Rôle est une chaîne de caractères (ex: `ROLE_USER`, `ROLE_ADMIN`) associée à un utilisateur et utilisée pour définir ses permissions. Par défaut, tout utilisateur connecté a le rôle `ROLE_USER`.

#### **C'est quoi un Rôle ?**

Les Rôles sont des étiquettes (labels) qui définissent ce qu'un utilisateur est autorisé à faire. Ils sont stockés dans la colonne `roles` de la table `user` (format JSON Array).

#### **Tâches à réaliser :**

1.  **Restreindre les pages CRUD aux Administrateurs** :
    Nous allons utiliser les attributs de sécurité directement sur les méthodes du `ProductController` pour exiger le rôle `ROLE_ADMIN`.

      * **Mettez à jour le rôle de votre utilisateur de test** : Dans la table `user`, changez le champ `roles` de votre utilisateur administrateur à `["ROLE_ADMIN"]`.

2.  **Appliquez la restriction au Contrôleur** :
    Ouvrez `src/Controller/ProductController.php`. Ajoutez l'attribut `#[IsGranted]` aux méthodes qui manipulent les données (`new`, `edit`, `delete`).

    ```php
    // src/Controller/ProductController.php

    use Symfony\Component\Security\Http\Attribute\IsGranted; // N'oubliez pas l'import

    // Le contrôle peut se faire au niveau de la classe pour toutes les méthodes :
    // #[IsGranted('ROLE_ADMIN')]
    // class ProductController extends AbstractController
    // { ... }

    // Ou individuellement sur les méthodes :

    #[Route('/product/new', name: 'app_product_new')]
    #[IsGranted('ROLE_ADMIN')] // Seuls les utilisateurs avec ROLE_ADMIN peuvent y accéder
    public function new(Request $request, EntityManagerInterface $entityManager): Response
    { /* ... */ }

    #[Route('/product/edit/{id}', name: 'app_product_edit')]
    #[IsGranted('ROLE_ADMIN')]
    public function edit(Request $request, Product $product, EntityManagerInterface $entityManager): Response
    { /* ... */ }

    #[Route('/product/delete/{id}', name: 'app_product_delete', methods: ['POST'])]
    #[IsGranted('ROLE_ADMIN')]
    public function delete(Request $request, Product $product, EntityManagerInterface $entityManager): Response
    { /* ... */ }

    // Les méthodes index() et show() restent publiques (pas de restriction)
    ```

3.  **Testez la restriction d'accès** :

      * **Connectez-vous** avec votre compte `ROLE_ADMIN`. Vous devriez pouvoir accéder à `/product/new`.
      * **Déconnectez-vous** et essayez d'accéder à `/product/new`. Vous devriez être redirigé vers la page de connexion, puis recevoir une erreur **403 Forbidden** (Accès refusé) si l'accès est bloqué après la connexion.

-----

## **Partie 4 : Restriction d'Affichage dans la Vue**

Il est crucial de cacher les liens ou boutons d'actions aux utilisateurs qui n'ont pas les droits, même si le contrôleur bloque déjà l'accès.

#### **Tâches à réaliser :**

1.  **Utilisez la fonction `is_granted()` de Twig** :
    Cette fonction est l'équivalent de l'attribut `#[IsGranted]` dans les templates. Elle vérifie si l'utilisateur actuellement connecté possède un certain rôle.

2.  **Modifiez les templates** :
    Dans `templates/product/show.html.twig` (et potentiellement `index.html.twig`), masquez les boutons "Modifier" et "Supprimer" si l'utilisateur n'est pas Administrateur.

    ```twig
    {# templates/product/show.html.twig #}

    {% if is_granted('ROLE_ADMIN') %}
        <a href="{{ path('app_product_edit', {id: product_item.id}) }}" class="btn btn-warning mt-3">Modifier le produit</a>
        
        <form method="post" action="{{ path('app_product_delete', {'id': product_item.id}) }}" style="display:inline-block">
            <input type="hidden" name="_token" value="{{ csrf_token('delete' ~ product_item.id) }}">
            <button type="submit" class="btn btn-danger" onclick="return confirm('Êtes-vous sûr ?');">Supprimer</button>
        </form>
    {% endif %}
    ```

3.  **Testez l'affichage** :

      * Connectez-vous en tant qu'administrateur : les boutons apparaissent.
      * Déconnectez-vous (ou créez un utilisateur simple `ROLE_USER`) : les boutons doivent disparaître.

-----

### **Synthèse et Explications**

Le composant Sécurité de Symfony opère en deux phases principales :

1.  **Authentification** : Prouver qui vous êtes (via le formulaire de connexion et le `LoginFormAuthenticator`). Si vous réussissez, Symfony crée un *Token* de sécurité qui dure le temps de votre session.
2.  **Autorisation** : Vérifier ce que vous avez le droit de faire (`#[IsGranted]`). Cette vérification est faite après l'authentification et se base sur vos Rôles.

| Composant | Rôle dans la Sécurité | Documentation Officielle |
| :--- | :--- | :--- |
| **`make:user`** | Crée l'Entité de base pour l'utilisateur. | [https://symfony.com/doc/current/security.html\#creating-the-user-class](https://www.google.com/search?q=https://symfony.com/doc/current/security.html%23creating-the-user-class) |
| **`make:auth`** | Crée le contrôleur de connexion et configure le `firewall`. | [https://symfony.com/doc/current/security/form\_login\_setup.html](https://symfony.com/doc/current/security/form_login_setup.html) |
| **`firewall`** | Définit la zone de l'application à protéger (dans `security.yaml`). | [https://symfony.com/doc/current/security.html\#configuring-access-control](https://www.google.com/search?q=https://symfony.com/doc/current/security.html%23configuring-access-control) |
| **`#[IsGranted('ROLE')]`** | Vérifie les permissions dans les contrôleurs (Autorisation). | [https://symfony.com/doc/current/security/granting\_access.html](https://www.google.com/search?q=https://symfony.com/doc/current/security/granting_access.html) |
| **`is_granted('ROLE')`** | Vérifie les permissions dans les templates Twig. | [https://symfony.com/doc/current/security.html\#checking-if-a-user-is-logged-in-the-template](https://www.google.com/search?q=https://symfony.com/doc/current/security.html%23checking-if-a-user-is-logged-in-the-template) |
