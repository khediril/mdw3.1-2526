# Atelier 6: Sécuriser votre application Symfony (v7.3)

## Introduction aux concepts de Sécurité

La sécurité dans Symfony repose principalement sur deux notions fondamentales :

  * **Authentification**: **Qui êtes-vous?** C'est le processus de vérification de l'identité de l'utilisateur (via un formulaire de login, une clé API, OAuth, etc.).
  * **Autorisation (Authorization)**: **Avez-vous accès à cette ressource?** Elle détermine si un utilisateur (authentifié ou anonyme) est autorisé à effectuer une action ou à accéder à une page spécifique.

Le système de sécurité de Symfony s'articule autour de plusieurs éléments clés :

  * **Le Firewall**: Le point d'entrée qui définit les règles d'authentification pour des portions spécifiques de l'application (basées sur des patterns d'URL).
  * **Le Provider**: Interroge votre source de données (base de données, fichier, etc.) pour trouver et charger les utilisateurs (leurs identifiants et mots de passe hachés).
  * **Le Password Hashers** : Remplace l'ancien concept d'« encoder » pour générer et vérifier le hachage des mots de passe. Le hacheur par défaut est configuré sur `auto` pour utiliser l'algorithme le plus sûr supporté (Argon2i ou bcrypt).
  * **L'Authenticator** : Gère la logique d'authentification (soumission de formulaire, vérification des identifiants) et crée un **Passport** en cas de succès.
  * **Les Rôles**: Définissent le niveau d'accès d'un utilisateur (ex: `ROLE_USER`, `ROLE_ADMIN`).

## Prérequis

Assurez-vous que le composant `security-bundle` est installé. Si votre projet a été créé avec le *Symfony webapp skeleton*, il est déjà présent.

```bash
composer require symfony/security-bundle
# Optionnel: Nécessaire pour les commandes 'make:*'
composer require symfony/maker-bundle --dev
```

-----

## Étape 1 : Créer l'Entité Utilisateur (User)

La classe utilisateur est la base de la sécurité. Elle doit implémenter les interfaces `UserInterface` et `PasswordAuthenticatedUserInterface`.

La commande **MakerBundle** génère automatiquement l'entité et configure le **User Provider** dans `config/packages/security.yaml`.

```bash
symfony console make:user
```

**Réponses aux questions typiques :**

1.  **Le nom de la classe d'entité** (ex: `User`).
2.  **Le champ utilisé comme identifiant** (ex: `email`).
3.  **Voulez-vous hacher le mot de passe?** (`yes`).

Cette commande génère :

  * `src/Entity/User.php` (avec les interfaces implémentées).
  * `src/Repository/UserRepository.php`.
  * La configuration du `provider` dans `config/packages/security.yaml`.

### Mise à jour de la Base de Données

Après la création de l'entité, mettez à jour la structure de votre base de données :

```bash
symfony console make:migration
symfony console doctrine:migrations:migrate
```

-----

## Étape 2 : Créer le Formulaire d'Inscription

Le **MakerBundle** simplifie grandement la création du formulaire, du contrôleur et de la logique d'enregistrement des utilisateurs avec hachage de mot de passe.

```bash
symfony console make:registration-form
```

**Réponses aux questions typiques :**

1.  **L'entité à utiliser** (ex: `App\Entity\User`).
2.  **Ajouter une contrainte `UniqueEntity`** (recommandé: `yes`).
3.  **Voulez-vous un e-mail de vérification?** (dans notre cas No ,optionnel, nécessite `symfonycasts/verify-email-bundle`).

Cette commande génère notamment :

  * `src/Controller/RegistrationController.php`.
  * `src/Form/RegistrationFormType.php`.
  * `templates/registration/register.html.twig`.

-----

## Étape 3 : Créer le Formulaire de Connexion et l'Authenticator

Créez le système de connexion qui utilisera l'entité `User` et le `provider` configurés à l'étape 1.

Utilisez la commande `make:security:form-login` qui remplace l'ancienne commande `make:auth` pour générer un authenticator basé sur un formulaire.

```bash
symfony console make:security:form-login
```

**Réponses aux questions typiques :**


1.  **Nom de la classe du Contrôleur** (ex: `SecurityController`).
2.  **Générer l'URL de déconnexion (`/logout`)** (recommandé: `yes`).
3.  **Do you want to generate PHPUnit tests?** (No).

Cette commande génère et met à jour :

  * `src/Security/LoginFormAuthenticator.php`
  * `src/Controller/SecurityController.php`
  * `templates/security/login.html.twig`
  * `config/packages/security.yaml` (mise à jour de la section `firewalls`).

### Configuration du Firewall

Le fichier `config/packages/security.yaml` est mis à jour. L'authenticator généré est lié au `firewall` principal (`main`).

```yaml
# config/packages/security.yaml

security:
    # ... autres configurations
    firewalls:
        dev:
            # ... pour les outils de debug
        main:
            lazy: true
            # L'authenticator généré est utilisé ici
            # Il gère les identifiants, le hachage et le succès/échec de la connexion
            provider: app_user_provider # Le provider généré par make:user
            form_login:
                login_path: app_login
                check_path: app_login
                # ... autres options de form_login
            logout:
                path: app_logout
                target: app_homepage # Redirection après déconnexion (à définir)
            
            # NOUVEAU en Symfony 7.3 : Gérer l'énumération des utilisateurs
            # 'none' (par défaut) cache toutes les erreurs liées aux utilisateurs non trouvés.
            # L'option 'hide_user_not_found' est dépréciée.
            expose_security_errors: 'none'
```

-----

## Autorisation et Contrôle d'Accès

Utilisez la section `access_control` dans `security.yaml` pour définir les rôles requis pour accéder à certaines URL.

| Rôle | Description |
| :--- | :--- |
| `IS_AUTHENTICATED_ANONYMOUSLY` | Accessible à tous (visiteur ou utilisateur connecté). |
| `IS_AUTHENTICATED_FULLY` | Accessible uniquement aux utilisateurs pleinement authentifiés (ne s'applique pas aux sessions "Remember Me"). |
| `ROLE_USER` | Accessible uniquement aux utilisateurs ayant le rôle `ROLE_USER` (ou un rôle supérieur, si la hiérarchie est configurée). |
| `ROLE_ADMIN` | Accessible uniquement aux utilisateurs avec le rôle `ROLE_ADMIN`. |

```yaml
# config/packages/security.yaml

security:
    # ...
    access_control:
        # Ex: L'accès à /admin nécessite le rôle ROLE_ADMIN
        - { path: ^/admin, roles: ROLE_ADMIN }
        # Ex: L'accès à /profile nécessite d'être connecté
        - { path: ^/profile, roles: IS_AUTHENTICATED_FULLY }
        # Ex: L'accès à la page de connexion est ouvert
        - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
```

-----

## Gérer la Déconnexion (Logout)

La déconnexion est gérée par la configuration du `firewall` et ne nécessite qu'une méthode vide dans un contrôleur.

Le **MakerBundle** l'a déjà ajoutée pour vous dans `SecurityController.php`.

```php
// src/Controller/SecurityController.php

use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Security\Http\Attribute\IsGranted;

class SecurityController extends AbstractController
{
    // ...
    #[Route(path: '/logout', name: 'app_logout')]
    public function logout(): void
    {
        // Cette méthode peut rester vide. 
        // Elle est interceptée par le 'logout' configuré dans security.yaml.
        throw new \LogicException('This method can be blank - it will be intercepted by the logout key on your firewall.');
    }
}
```

-----

## Outils et Bonnes Pratiques

### 1\. Récupérer l'utilisateur connecté

Dans n'importe quel contrôleur, vous pouvez accéder à l'objet utilisateur connecté.

```php
// Dans un contrôleur qui étend AbstractController
$user = $this->getUser();

// Pour les controllers qui n'étendent pas AbstractController (injection de dépendances):
// Injectez UserInterface ou utilisez l'attribut #[CurrentUser]
```

### 2\. Générer un mot de passe haché manuellement

Pour insérer manuellement un utilisateur dans la base de données (ex: un premier administrateur), vous pouvez hacher le mot de passe via la console.

```bash
# La commande va vous demander le mot de passe et utiliser le hacheur configuré dans security.yaml
symfony console security:hash-password
```