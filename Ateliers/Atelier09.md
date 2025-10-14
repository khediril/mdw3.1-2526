# **Atelier Symfony : Gestion des Assets avec AssetMapper et Symfony UX**

Cet atelier vous introduit aux méthodes modernes de gestion des fichiers frontend (JavaScript, CSS, images) et à l'ajout de fonctionnalités frontend avancées (comme la réactivité) directement depuis le code PHP/Twig, sans la complexité de Webpack ou Node.js.

## **Objectifs de l'Atelier**

  * Comprendre le rôle d'**AssetMapper** et sa différence avec Webpack Encore.
  * Gérer les fichiers CSS et JavaScript via des importations simples.
  * Installer et utiliser des composants **Symfony UX** (comme Stimulus) pour rendre les pages interactives.
  * Utiliser la fonction Twig `asset()` pour référencer les fichiers du *mapper*.

-----

## **Partie 1 : Introduction à AssetMapper**

Traditionnellement, les projets modernes utilisaient des outils complexes comme Webpack Encore (basé sur Node.js) pour gérer les dépendances frontend et la minification. **AssetMapper** est l'approche native de Symfony pour gérer les *assets* sans dépendre de Node.js.

### **C'est quoi AssetMapper ?** 🗺️

AssetMapper est un composant qui **lit, copie et sert** vos fichiers frontend. Il :

1.  Lit les fichiers dans le dossier **`assets/`** de votre projet.
2.  Traite les imports JS et CSS (il sait les suivre).
3.  Copie les fichiers finaux dans le dossier public **`public/assets/`**.
4.  Ajoute des empreintes de hachage (ex: `app.12345.css`) aux noms de fichiers pour la gestion du cache.

#### **Tâches à réaliser :**

1.  **Installez AssetMapper (Si ce n'est pas déjà fait)** :
    Un projet créé avec `symfony new --webapp` l'inclut par défaut. Sinon :

    ```bash
    composer require symfony/asset-mapper
    ```

2.  **Analysez les dossiers** :

      * **`assets/`** : Contient vos fichiers sources (JS, CSS, images) que vous éditez.
      * **`public/assets/`** : Contient les fichiers compilés/mappés, générés par Symfony, qui sont servis au navigateur. **Ne touchez jamais aux fichiers dans ce dossier \!**

3.  **Testez un Import simple (CSS)** :

      * Créez un fichier `assets/styles/custom.css`.
      * Ajoutez une règle simple : `body { background-color: lightblue; }`.
      * **Dans `templates/base.html.twig`**, liez votre nouveau fichier CSS à l'aide de la fonction Twig `asset()` :

    <!-- end list -->

    ```twig
    {# templates/base.html.twig #}
    <head>
        {# Le chemin 'styles/custom.css' est relatif au dossier 'assets/' #}
        <link rel="stylesheet" href="{{ asset('styles/custom.css') }}"> 
        {# ... autres assets ... #}
    </head>
    ```

4.  **Observez le rendu** :

      * Démarrez votre serveur et rafraîchissez une page. Le fond doit être bleu clair.
      * **Observez la balise `<link>` générée dans le HTML source** : Elle pointe vers `public/assets/styles/custom.HASH.css`. C'est AssetMapper qui a mappé le fichier et ajouté l'empreinte pour le cache.

-----

## **Partie 2 : Utilisation des Dépendances Frontend avec AssetMapper**

AssetMapper peut gérer des librairies externes (comme Axios, Chart.js, etc.) sans utiliser `npm`.

#### **Tâches à réaliser :**

1.  **Installez une librairie JavaScript (ex: Sortable.js)** :
    Nous utilisons la commande `importmap:require` qui télécharge la librairie et configure l'import.

    ```bash
    php bin/console importmap:require sortablejs
    ```

2.  **Analysez la configuration** :

      * Ouvrez **`config/packages/importmap.php`**. Vous y voyez que `sortablejs` a été ajouté.

3.  **Utilisez la librairie dans votre JS** :

      * Ouvrez votre fichier JavaScript principal, par exemple `assets/app.js`.
      * **Importez** la librairie en utilisant le nom défini dans `importmap.php`.

    <!-- end list -->

    ```javascript
    // assets/app.js

    // Importation de la librairie externe
    import Sortable from 'sortablejs'; 

    console.log('Sortable.js est chargé !');

    // Exemple d'utilisation (à vous de créer l'élément #my-list dans votre Twig)
    document.addEventListener('DOMContentLoaded', () => {
        const list = document.getElementById('my-sortable-list');
        if (list) {
            new Sortable(list, {
                animation: 150
            });
        }
    });
    ```

4.  **Liez le fichier JS principal dans Twig** :
    Dans `templates/base.html.twig`, assurez-vous que votre fichier `app.js` est chargé.

    ```twig
    {# templates/base.html.twig #}

    {# Ajoutez la carte d'importation nécessaire pour que les imports JS fonctionnent #}
    {{ importmap('app') }}

    <script src="{{ asset('app.js') }}" defer></script>
    ```

      * **`{{ importmap('app') }}`** : Cette fonction génère une carte (un ensemble de balises `<script type="importmap">`) qui indique au navigateur où trouver toutes les dépendances importées dans `app.js`.

-----

## **Partie 3 : Interactivité avec Symfony UX et Stimulus**

**Symfony UX** est une collection de packages Symfony qui vous permet d'ajouter des fonctionnalités frontend modernes (graphiques, drag-and-drop, etc.) en utilisant la philosophie **Stimulus**.

### **C'est quoi Stimulus ?** 💡

**Stimulus** est un *framework* JavaScript léger qui relie le HTML (la *Vue* de MVC) au JavaScript (le *Contrôleur* du frontend). Il permet de rendre les éléments HTML interactifs en ajoutant simplement des **attributs `data-controller`** aux balises HTML.

#### **Tâches à réaliser :**

1.  **Installez Stimulus** :
    Le package `symfony/stimulus-bundle` installe le moteur Stimulus dans votre projet.

    ```bash
    composer require symfony/stimulus-bundle
    ```

2.  **Générez un Contrôleur Stimulus** :
    Créons un contrôleur qui affichera un message lorsqu'on clique sur un bouton (un simple *toggle*).

    ```bash
    php bin/console make:stimulus toggle
    ```

      * Cela crée `assets/controllers/toggle_controller.js`.

3.  **Implémentez la Logique du Contrôleur (JavaScript)** :
    Ouvrez `assets/controllers/toggle_controller.js`.

    ```javascript
    // assets/controllers/toggle_controller.js
    import { Controller } from '@hotwired/stimulus';

    export default class extends Controller {
        // Définition des "targets" : les éléments que ce contrôleur gère
        static targets = ['content']; 

        // Définition de l'action à exécuter (méthode de classe)
        connect() {
            console.log('Contrôleur Toggle chargé !');
        }

        toggleVisibility() {
            this.contentTarget.classList.toggle('d-none'); // d-none est une classe Bootstrap pour cacher
        }
    }
    ```

4.  **Appliquez le Contrôleur Stimulus dans Twig (HTML)** :
    Ouvrez, par exemple, `templates/product/show.html.twig` et appliquez les attributs Stimulus au HTML.

    ```twig
    {# templates/product/show.html.twig #}

    <div 
        data-controller="toggle"               {# 1. Attache le contrôleur JS 'toggle' à cette DIV #}
        class="my-4 p-3 border"
    >
        <button 
            data-action="toggle#toggleVisibility" {# 2. Définit l'action (clic -> toggle_controller.js -> toggleVisibility) #}
            class="btn btn-info"
        >
            Afficher/Cacher la description
        </button>

        <p 
            data-toggle-target="content"         {# 3. Définit cet élément comme le 'contentTarget' du contrôleur #}
            class="mt-3"
        >
            {{ product_item.description }}
        </p>
    </div>
    ```

5.  **Testez l'interactivité** :

      * Accédez à la page de détail d'un produit.
      * Cliquez sur le bouton "Afficher/Cacher la description".
      * **Résultat** : La description apparaît et disparaît sans aucune écriture de code JavaScript dans le template lui-même. C'est Stimulus qui fait le lien \!

-----

### **Synthèse des Outils Frontend**

| Outil | Rôle Principal | Approche |
| :--- | :--- | :--- |
| **AssetMapper** | Gestion des fichiers statiques (CSS, JS, Images). | Mapping natif sans Node.js. |
| **Importmap** | Permet d'importer des librairies JS externes de manière simple dans votre JS local. | Standard moderne du navigateur. |
| **Symfony UX** | Collection de paquets pour l'interactivité frontend. | Utilise Stimulus. |
| **Stimulus** | Petit framework pour lier le JS aux éléments HTML via des attributs. | *HTML-first,* logique simple. |

  * **Documentation AssetMapper** : [https://symfony.com/doc/current/frontend/asset\_mapper.html](https://symfony.com/doc/current/frontend/asset_mapper.html)
  * **Documentation Symfony UX (Stimulus)** : [https://symfony.com/doc/current/frontend/stimulus.html](https://www.google.com/search?q=https://symfony.com/doc/current/frontend/stimulus.html)
  * **Documentation Hotwired Stimulus** : [https://stimulus.hotwired.dev/](https://stimulus.hotwired.dev/)
