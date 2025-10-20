# **Atelier Symfony : Intégration d'un Template Bootstrap**

Cet atelier se concentre sur l'amélioration de l'interface utilisateur de votre application en intégrant un thème HTML/CSS/JS externe (un template Bootstrap gratuit). Vous apprendrez à manipuler le moteur de template Twig pour gérer les *assets* (CSS, JS, images) de manière structurée et réutilisable.

## **Objectifs de l'Atelier**

  * Maîtriser le concept d'**Héritage de Templates** dans Twig.
  * Utiliser les fonctions Twig `asset()` et `block` pour l'intégration.
  * Intégrer les fichiers CSS, JavaScript et les Images d'un template externe.
  * Appliquer le style du template aux pages existantes (Liste, Détail, Formulaires).

-----

## **Partie 1 : Préparation et Choix du Template**

Avant de commencer l'intégration, nous devons disposer d'un template et comprendre sa structure.

#### **Tâches à réaliser :**

1.  **Choisissez un Template** :
    Rendez-vous sur un site comme [**ThemeWagon**](https://themewagon.com/theme-tag/ecommerce-template/) ou **Start Bootstrap** et téléchargez un template *gratuit* (souvent avec un fichier `index.html`).
    Télécharger le template suivant :[**Electro**] (https://themewagon.com/themes/electro-bootstrap/)

3.  **Analysez la Structure du Template** :
    Ouvrez les fichiers téléchargés. Un template typique contient :

      * Un ou plusieurs fichiers **HTML** (ex: `index.html`, `about.html`).
      * Un dossier **CSS** (contenant souvent `style.css` ou `bootstrap.min.css`).
      * Un dossier **JS** (contenant les scripts personnalisés ou les librairies).
      * Un dossier **IMG** ou **ASSETS** (contenant les images, le logo, les polices).

4.  **Nettoyez le dossier `public`** :
    Dans votre projet Symfony, le dossier **`public/`** est la racine web. Tout ce qui est accessible publiquement par le navigateur doit y être placé.

      * Dans `MaBoutique/public/`, créez un dossier **`theme/`** pour isoler les fichiers du template.
      * **Copiez** tous les dossiers CSS, JS, IMAGES du template téléchargé à l'intérieur de `public/theme/`.

-----

## **Partie 2 : Création du Template de Base (Base Layout)**

Dans Symfony, tous les templates héritent d'un fichier de base (`base.html.twig`). Nous allons modifier ce fichier pour y insérer la structure HTML complète (entête, pied de page) et tous les liens vers les assets du template.

#### **Tâches à réaliser :**

1.  **Analysez `base.html.twig`** :
    Ouvrez `templates/base.html.twig`. Vous y voyez des `blocks` (`title`, `stylesheets`, `body`, `javascripts`).

2.  **Transférez la structure HTML** :
    Ouvrez le fichier `index.html` de votre template téléchargé.

      * **Copiez** tout le code HTML (incluant `<!DOCTYPE...>`, `<head>`, et `<body>`).
      * **Collez** ce code dans `templates/base.html.twig`, en respectant la structure suivante :

3.  **Intégrez les Assets avec la fonction `asset()`** :
    La fonction Twig **`asset('chemin')`** génère un chemin public vers un fichier, pointant vers le dossier `public/`.

      * **Dans la balise `<head>`**, remplacez les liens CSS par `asset()` :

        ```html
        {# AVANT (dans le template téléchargé) : #}
        <link href="assets/css/styles.css" rel="stylesheet" />

        {# APRÈS (dans base.html.twig, en pointant vers public/theme/) : #}
        <link href="{{ asset('theme/css/styles.css') }}" rel="stylesheet" />
        ```

      * **Dans la balise `<body>`** (avant la fermeture `</body>`), remplacez les liens JavaScript :

        ```html
        {# APRÈS : #}
        <script src="{{ asset('theme/js/scripts.js') }}"></script>
        ```

4.  **Identifiez et placez les Blocs Twig** :
    Le but est de laisser les zones dynamiques ouvertes pour les templates enfants :

      * Remplacez la balise `<title>` :
        ```html
        <title>{% block title %}Bienvenue sur MaBoutique{% endblock %}</title>
        ```
      * Localisez la zone du template qui doit contenir le contenu de la page (souvent dans un `div.container` ou `main`). **Supprimez le contenu statique** de cette zone et insérez le bloc `body` :
        ```html
        <div class="container-fluid py-5">
            {# TOUT le contenu de la page enfant ira ici #}
            {% block body %}{% endblock %} 
        </div>
        ```

-----

## **Partie 3 : Application du Thème aux Vues Existant**

Maintenant que le template de base est prêt, nous allons l'appliquer à nos pages.

#### **Tâches à réaliser :**

1.  **Réorganisez la Vue Liste (`index.html.twig`)** :
    Ouvrez `templates/product/index.html.twig`. Vous n'avez plus besoin des balises `<html>`, `<head>`, etc. Le contenu doit uniquement se trouver dans le bloc `body`.

      * **Appliquez les classes Bootstrap** du template à la liste de produits. Par exemple, au lieu d'une simple `div.row`, utilisez la structure d'un *Grid System* ou des cartes mises en forme par le template.

2.  **Réorganisez la Vue Détail (`show.html.twig`)** :
    Adaptez la vue détail en utilisant la structure de présentation d'un produit fournie par le template (souvent une mise en page à deux colonnes pour l'image et la description).

3.  **Intégrez les Images du Template (Logo, Favicon)** :
    Si votre template utilise un logo ou un favicon (dans l'entête ou dans la barre de navigation), assurez-vous que leur chemin utilise `asset()` et pointe vers le dossier `public/theme/images/` (ou équivalent).

    ```html
    {# Exemple d'intégration de logo dans le header de base.html.twig #}
    <img src="{{ asset('theme/assets/logo.png') }}" alt="Logo du site" height="30" />
    ```

4.  **Testez la navigation** :
    Démarrez le serveur (`symfony serve`). Naviguez entre la page d'accueil, la liste des produits et la page de détail. L'ensemble des pages devrait maintenant avoir l'apparence du template Bootstrap intégré.

-----

## **Concepts Clés : L'Héritage Twig**

| Directive Twig | Rôle | Explication |
| :--- | :--- | :--- |
| `{% extends 'base.html.twig' %}` | **Héritage** | **Doit être la première ligne** du template. Il indique que ce fichier va hériter de la structure et des liens définis dans le template parent. |
| `{% block body %}` | **Définition de Bloc** | Définit une zone de contenu qui pourra être remplacée ou étendue par les templates enfants. |
| `{{ asset('path') }}` | **Gestion d'Assets** | Génère un chemin absolu vers un fichier dans le dossier `public/`. Essentiel pour que les CSS, JS et images fonctionnent quel que soit le niveau de l'URL. |

  * **Documentation Twig Héritage** : [https://twig.symfony.com/doc/3.x/templates.html\#template-inheritance](https://www.google.com/search?q=https://twig.symfony.com/doc/3.x/templates.html%23template-inheritance)
  * **Documentation Symfony Assets** : [https://symfony.com/doc/current/assetic/asset\_management.html](https://symfony.com/doc/current/assetic/asset_management.html)
  * **Documentation Bootstrap 5.3** : [https://getbootstrap.com/docs/5.3/getting-started/introduction/](https://getbootstrap.com/docs/5.3/getting-started/introduction/)
