## **Atelier : Le Conteneur de Services et l'Injection de Dépendances avec Symfony**

### **Public Cible**
* Étudiants de 3$^{\text{e}}$ Licence Informatique (ayant des bases en PHP orienté objet et MVC).

### **Prérequis Techniques**
* **Environnement Symfony** (version récente) fonctionnel.
* Connaissances de base de la **ligne de commande**.
* Éditeur de code (VS Code, PhpStorm, etc.).

### **Durée et Structure**
* **Durée Totale :** 3 heures.
* **Structure :** Théorie (30 min) + Pratique Guidée (120 min) + Synthèse/Questions (30 min).

---

## **Phase 1 : Fondamentaux (30 minutes)** 💡

Cette partie est principalement théorique, avec des exemples de code pour illustrer les concepts.

### **1. Introduction (10 min)**
* **Problématique du Code "Rigide" :** Expliquer le couplage fort (ex. : `new Mailer()`) et pourquoi il est problématique (testabilité, réutilisation).
* **Qu'est-ce qu'un Service ?** Définition : un objet qui effectue une tâche globale (ex. : envoyer un email, enregistrer en base de données). 
* **Qu'est-ce que le Conteneur de Services ?** Un "annuaire" ou une "usine" d'objets (services) qui gère leur cycle de vie et leurs dépendances.
* **Qu'est-ce que l'Injection de Dépendances (ID) ?** Le principe de fournir les dépendances (objets requis) à un objet de l'extérieur (via le constructeur, une méthode ou une propriété), au lieu de le laisser les créer lui-même.

### **2. Le Conteneur Symfony (20 min)**
* **Le Fichier `services.yaml` :** Rôle des configurations par défaut (`_defaults`).
* **Auto-wiring (Câblage Automatique) :**
    * Le concept : le conteneur devine et injecte les dépendances automatiquement en se basant sur le **Type Hinting**.
    * Démonstration dans un contrôleur : injection d'un service Symfony existant (ex. : `LoggerInterface`, `MailerInterface`).
    * Commande utile : `php bin/console debug:autowiring`

---

## **Phase 2 : Pratique Guidée - Création de Services (120 minutes)** 🛠️

Les étudiants travaillent sur un projet Symfony pour appliquer les concepts.

### **3. Exercice 1 : Créer un Service Personnalisé (40 min)**
* **Objectif :** Créer un service simple, le déclarer (implicitement) et l'utiliser.
    * **Tâche 1 :** Créer l'interface `App\Service\TextProcessorInterface` (ex. : `process(string $text): string`).
    * **Tâche 2 :** Créer la classe d'implémentation `App\Service\UpperCaseProcessor` qui implémente l'interface et met le texte en majuscule.
    * **Tâche 3 :** Créer un contrôleur `ProcessorController` et **injecter** l'`UpperCaseProcessor` dans la méthode du contrôleur.
    * **Vérification :** Exécuter une route pour vérifier que le service fonctionne.

### **4. Exercice 2 : Injection de Dépendances Manuelle (40 min)**
* **Objectif :** Maîtriser l'injection dans les constructeurs et la configuration explicite si nécessaire.
    * **Tâche 1 :** Créer une nouvelle implémentation `App\Service\PrefixProcessor` qui ajoute un préfixe configurable au texte (ex. : "\[PREFIKSÉ\] ").
    * **Tâche 2 :** Cette classe doit prendre le préfixe en argument de son **constructeur** (`__construct(string $prefix)`).
    * **Tâche 3 :** **Désactiver temporairement l'auto-wiring** pour ce service dans `services.yaml` (ou le marquer `autowire: false`) et le **configurer manuellement** pour passer une valeur string au constructeur (dans la section `arguments`).
    * **Tâche 4 :** Injecter ce service dans un nouveau contrôleur pour le tester.
    * **Démonstration (Optionnel) :** Rendre le paramètre configurable globalement via `config/packages/parameters.yaml` (ou une section de `services.yaml`).

### **5. Exercice 3 : Utiliser les Interfaces pour le Conteneur (40 min)**
* **Objectif :** Comprendre l'avantage d'injecter une interface plutôt qu'une implémentation concrète.
    * **Tâche 1 :** Revenir à l'interface `TextProcessorInterface`.
    * **Tâche 2 :** Dans `services.yaml`, utiliser l'**Alias (ou la configuration de l'interface)** pour indiquer au conteneur que lorsqu'on demande `TextProcessorInterface`, il doit injecter l'`UpperCaseProcessor` (ou une autre implémentation).
    * **Tâche 3 :** Modifier le contrôleur pour n'injecter que l'**Interface** (`TextProcessorInterface $processor`).
    * **Vérification :** Changer rapidement l'alias dans `services.yaml` pour injecter le `PrefixProcessor` sans modifier le code du contrôleur. **C'est la puissance de l'ID/ID !**

---

## **Phase 3 : Conclusion et Q/R (30 minutes)** 🎓

### **6. Synthèse et Débogage (15 min)**
* **Rappel des Concepts Clés :**
    * Couplage faible (Loose Coupling).
    * Testabilité (facile de *mock* un service injecté).
    * **Auto-wiring vs. Configuration Manuelle.**
* **Outils de Débogage :**
    * `php bin/console debug:container` (lister tous les services).
    * `php bin/console debug:container <id_du_service>` (détails d'un service).

### **7. Questions/Réponses (15 min)**
* Session ouverte pour répondre aux questions des étudiants et éclaircir les points complexes (ex. : la notion de services *privés* ou *publics*, les *tags*).
