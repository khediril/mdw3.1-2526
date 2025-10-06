# Atelier Guidé : Premiers Pas avec Symfony 7.3 (3 Heures)

## Introduction : Le Flux de Travail de Symfony

Bienvenue dans cet atelier d'introduction au framework **Symfony 7.3**. L'objectif est de comprendre le cycle de vie d'une requête (le flux **Request $\rightarrow$ Route $\rightarrow$ Controller $\rightarrow$ Template $\rightarrow$ Response**) en créant une application web simple.

### ⚠️ Prérequis Techniques (À vérifier AVANT de commencer)

Assurez-vous que les éléments suivants sont installés et configurés :

1.  **PHP** (version $\ge 8.2$).
2.  **Composer** (gestionnaire de dépendances).
3.  **Symfony CLI** (fortement recommandé).
4.  Un éditeur de code (VS Code, PhpStorm, etc.).

-----

## Phase 1 : Démarrage et Architecture

Cette phase permet de mettre en place l'environnement et de découvrir la structure du projet.

### Q 1.0 : Installation et Lancement

**Tâche :** Installez un nouveau projet Symfony et lancez le serveur de développement.

1.  Ouvrez votre terminal et déplacez-vous dans le répertoire de travail souhaité.
2.  Créez un nouveau projet Symfony avec le *squelette webapp* (qui inclut déjà Twig et les annotations/attributs) :
    ```bash
    symfony new mon_projet_symfony --webapp
    cd mon_projet_symfony
    ```
3.  Démarrez le serveur local de développement :
    ```bash
    symfony server:start
    ```
4.  Ouvrez l'URL affichée (généralement `http://127.0.0.1:8000/`) dans votre navigateur.
    *Vous devriez voir la page de bienvenue de Symfony et la **Web Debug Toolbar**.*

### Q 1.1 : Structure du Projet

**Tâche :** Répondez aux questions suivantes dans vos notes :

1.  Quel fichier est le **point d'entrée** unique de toutes les requêtes HTTP dans une application Symfony ? (Indice : regardez dans le dossier `public/`).
2.  Dans quel dossier allons-nous écrire la majorité de notre code PHP (les classes et les contrôleurs) ?
3.  À quoi sert le dossier `templates/` ?

-----

## Phase 2 : Le Contrôleur et le Routage

Nous allons créer notre propre page d'accueil pour remplacer celle par défaut.

### Q 2.0 : Création du Contrôleur

**Tâche :** Générez le squelette d'un contrôleur.

1.  Arrêtez le serveur (`Ctrl+C`) et utilisez l'outil **Maker Bundle** pour créer une nouvelle classe de contrôleur nommée `HomeController` :
    ```bash
    php bin/console make:controller HomeController
    ```
2.  **Ouvrez le fichier** créé dans `src/Controller/HomeController.php`. Observez la méthode `index()` et l'attribut PHP **`#[Route]`** associé.

### Q 2.1 : Route Statique et Réponse HTTP

**Tâche :** Modifiez le Contrôleur pour qu'il réponde à l'URL racine (`/`) avec un simple message.

1.  Assurez-vous que l'attribut **`#[Route]`** pour la méthode `index()` est bien configuré pour l'URL racine :
    ```php
    #[Route('/', name: 'app_home')]
    ```
2.  Modifiez la méthode `index()` pour qu'elle **retourne directement un objet `Response`** contenant une simple chaîne de texte :
    ```php
    // Dans src/Controller/HomeController.php
    use Symfony\Component\HttpFoundation\Response;

    // ... dans la méthode index()
    return new Response('<h1>Bienvenue sur mon Symfony 7.3 !</h1>');
    ```
3.  Relancez le serveur et vérifiez le résultat dans votre navigateur.

### Q 2.2 : Route Dynamique avec Paramètres

**Tâche :** Créez une nouvelle page qui prend un nom en paramètre dans l'URL.

1.  Dans la classe `HomeController`, ajoutez une nouvelle méthode `greet()` qui accepte un argument de type `string` :

    ```php
    // Dans src/Controller/HomeController.php
    #[Route('/hello/{name}', name: 'app_greet')]
    public function greet(string $name = 'Monde'): Response
    {
        return new Response('Bonjour, ' . $name . ' !');
    }
    ```

    *Le `= 'Monde'` permet d'avoir une valeur par défaut.*

2.  Testez les URL suivantes :

      * `http://127.0.0.1:8000/hello/Toto`
      * `http://127.0.0.1:8000/hello/Alice`

-----

## Phase 3 : La Vue avec Twig et les Assets

Nous allons utiliser **Twig** pour un affichage propre et gérer les ressources statiques.

### Q 3.0 : Rendre un Template

**Tâche :** Modifiez la méthode `greet()` pour qu'elle utilise un template Twig pour le rendu.

1.  Dans la méthode `greet()`, remplacez l'objet `Response` par un appel à la méthode **`$this->render()`**.
2.  La méthode `render()` prend le chemin du template (`hello/greet.html.twig`) et un **tableau de variables** à transmettre à la vue :
    ```php
    // Dans src/Controller/HomeController.php, méthode greet()
    return $this->render('home/greet.html.twig', [
        'user_name' => $name, // On passe la variable 'name' sous le nom 'user_name'
    ]);
    ```

### Q 3.1 : Création du Template Twig

**Tâche :** Créez le fichier de la vue et utilisez la variable transmise.

1.  Créez le fichier **`templates/home/greet.html.twig`**.
2.  Structurez le template en utilisant l'**héritage de Twig** (étendez `base.html.twig`) et affichez la variable `user_name` :
    ```twig
    {# templates/home/greet.html.twig #}
    {% extends 'base.html.twig' %}

    {% block title %}Bonjour {{ user_name }}{% endblock %}

    {% block body %}
        <h1>Salut {{ user_name }} !</h1>

        <p>Ceci est la première page générée avec Twig.</p>
        
        {# Q 3.2 : Lien Inter-Pages #}
        <p>
            <a href="{{ path('app_home') }}">Retour à la page d'accueil (Route: app_home)</a>
        </p>

    {% endblock %}
    ```
3.  Vérifiez que la page fonctionne en accédant à l'URL dynamique.

### Q 3.3 : Intégrer une Image (Gestion des Assets)

**Tâche :** Ajoutez une image (asset statique) à votre template en utilisant la bonne méthode Symfony.

1.  Créez un dossier pour les images dans votre répertoire public :
    ```bash
    mkdir public/images
    ```
2.  Ajoutez une image de votre choix (ex: un logo ou une image de test) dans ce dossier et nommez-la `logo.png` (ou un autre nom).
3.  Dans votre template `greet.html.twig`, utilisez la fonction Twig **`asset()`** pour générer le chemin correct vers cette image :
    ```twig
    {# Ajoutez cette ligne sous le titre dans templates/home/greet.html.twig #}
    <img src="{{ asset('images/logo.png') }}" alt="Logo Symfony" style="width: 100px;">
    ```
4.  Actualisez la page. L'image est-elle affichée ? Si oui, pourquoi l'utilisation de `asset()` est-elle préférable à un simple chemin relatif comme `/images/logo.png` ?

-----

## Phase 4 : Conclusion

### Q 4.0 : Bilan du Cycle de Vie

**Tâche :** Décrivez le chemin parcouru par une requête dans Symfony, en mentionnant les quatre composants principaux : **Route**, **Controller**, **Template** et **Response**.

### Q 4.1 : Prochaines Étapes

Notre application est statique. Quels sont les trois prochains grands sujets à maîtriser dans Symfony pour créer une application complète avec gestion d'utilisateurs et de données ?

1.  **Doctrine ORM :** Pour la gestion de la base de données.
2.  **Formulaires :** Pour la collecte et la validation des données utilisateur.
3.  **Sécurité :** Pour l'authentification et les autorisations.

-----

**FIN DE L'ATELIER**