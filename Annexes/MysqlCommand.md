
# Voici quelques commandes MySQL (SQL) essentielles et courantes que vous utiliserez souvent pour gérer vos bases de données. Ces commandes sont à exécuter dans l'invite `mysql>` après vous être connecté au serveur.

---

## Commandes de Gestion du Serveur et des Bases de Données

| Commande | Description | Exemple d'utilisation |
| :--- | :--- | :--- |
| **`STATUS;`** | Affiche des informations détaillées sur l'état du serveur MySQL (version, temps de fonctionnement, etc.). | `STATUS;` |
| **`SHOW DATABASES;`** | Liste toutes les bases de données disponibles sur le serveur. | `SHOW DATABASES;` |
| **`CREATE DATABASE nom_db;`** | Crée une nouvelle base de données. | `CREATE DATABASE mon_projet;` |
| **`DROP DATABASE nom_db;`** | **Supprime** définitivement une base de données. **À utiliser avec prudence !** | `DROP DATABASE mon_projet;` |
| **`USE nom_db;`** | Sélectionne la base de données avec laquelle vous voulez travailler. Toutes les commandes suivantes (comme `SHOW TABLES;`) s'appliqueront à cette base. | `USE mon_projet;` |

---

## Commandes de Gestion des Tables (après `USE nom_db;`)

| Commande | Description | Exemple d'utilisation |
| :--- | :--- | :--- |
| **`SHOW TABLES;`** | Liste toutes les tables dans la base de données actuellement sélectionnée. | `SHOW TABLES;` |
| **`DESCRIBE nom_table;`** | ou **`DESC nom_table;`** | Affiche la structure (colonnes, types de données, clés) d'une table spécifique. | `DESCRIBE utilisateurs;` |
| **`SHOW CREATE TABLE nom_table;`** | Affiche la commande SQL exacte utilisée pour créer la table, incluant les options d'encodage et le moteur de stockage. | `SHOW CREATE TABLE produits;` |
| **`DROP TABLE nom_table;`** | Supprime définitivement une table. | `DROP TABLE anciens_logs;` |
| **`ALTER TABLE nom_table ...;`** | Modifie la structure d'une table (ajouter, supprimer ou modifier des colonnes, etc.). | `ALTER TABLE clients ADD COLUMN email VARCHAR(255);` |

---

## Commandes de Gestion des Utilisateurs et Privilèges

| Commande | Description | Exemple d'utilisation |
| :--- | :--- | :--- |
| **`SELECT user, host FROM mysql.user;`** | Affiche la liste des utilisateurs MySQL et les hôtes à partir desquels ils peuvent se connecter. | `SELECT user, host FROM mysql.user;` |
| **`CREATE USER 'user'@'host' IDENTIFIED BY 'mdp';`** | Crée un nouvel utilisateur. L'`host` est souvent `'localhost'` pour les connexions locales ou `'%'` pour toutes les connexions distantes. | `CREATE USER 'dev'@'localhost' IDENTIFIED BY 'devpass';` |
| **`GRANT privilèges ON db.table TO 'user'@'host';`** | Accorde des privilèges à un utilisateur. | `GRANT ALL PRIVILEGES ON *.* TO 'mon_admin'@'localhost';` |
| **`FLUSH PRIVILEGES;`** | Recharge les tables de privilèges pour que les changements (comme les `GRANT`) prennent effet immédiatement. **Essentiel après une modification de privilèges !** | `FLUSH PRIVILEGES;` |
| **`SHOW GRANTS FOR 'user'@'host';`** | Affiche les privilèges accordés à un utilisateur spécifique. | `SHOW GRANTS FOR 'dev'@'localhost';` |

---

## Commandes de Requête (DML)

| Commande | Description | Exemple d'utilisation |
| :--- | :--- | :--- |
| **`SELECT * FROM nom_table;`** | Récupère toutes les colonnes (`*`) et toutes les lignes d'une table. | `SELECT * FROM articles;` |
| **`SELECT col1, col2 FROM nom_table WHERE condition;`** | Récupère des colonnes spécifiques en fonction d'une condition. | `SELECT nom, prix FROM produits WHERE stock < 10;` |
| **`INSERT INTO nom_table (col1) VALUES ('valeur');`** | Insère une nouvelle ligne dans une table. | `INSERT INTO commandes (client_id) VALUES (15);` |
| **`UPDATE nom_table SET col1='valeur' WHERE condition;`** | Modifie les données existantes. | `UPDATE produits SET prix = 25.50 WHERE id = 4;` |
| **`DELETE FROM nom_table WHERE condition;`** | Supprime des lignes d'une table. | `DELETE FROM logs WHERE date < '2024-01-01';` |
| **`EXIT;`** | ou **`\q`** | Quitte la console MySQL. | `EXIT;` |
