**Crédits : Maxime CHALLON // Adaptations : Lauryne LEMOSQUET**

**Tout se passe (configuration, commandes dans le terminal) au sein de ce dossier dans lequel vous lisez ce fichier**

**Le projet doit s’exécuter avec `python run.py` sans aucune modification du code fourni, seuls le fichier .env doit être modifié.**

## Préparation de l'environnement de travail pour le script

### 1. Créer un environnement virtuel

A créer dans ce dossier, à côté du `README.md` et du `run.py`

Pour rappel: `virtualenv env -p python3` ou `python -m venv env`

### 2. Activer cet environnement

`source env/bin/activate` (ou `source env/Scripts/activate` pour Windows)

### 3. Importer les bonnes dépendances dans l'environnement

`pip install -r requirements.txt`

---

## Étapes à suivre pour remplir la base

### 1. Modifier le fichier `.env`
Le fichier doit contenir toutes les variables suivantes :
Le fichier est déjà pré remplie avec les informations de base mais vous devez y ajouter les informations d'utilisateur et de mot de passe de votre base de données PostgreSQL.

```env
pgDatabase=str
pgUser=str
pgPassword=str
pgPort=int
pgHost=str
pgSchemaImportsCsv=str
failOnFirstSqlError=bool
failOnFirstCsvError=bool
```

- `pgDatabase` : nom de la base à créer/utiliser  pour importer les données et jouer les scripts. Cette base est unique pour l'ensemble du projet.
- `pgUser` : utilisateur PostgreSQL avec lequel se connecter
- `pgPassword` : mot de passe PostgreSQL correspondant à l'utilisateur
- `pgHost` : adresse du serveur PostgreSQL
- `pgPort` : port du serveur PostgreSQL  
- `pgSchemaImportsCsv` : schéma où seront importés les CSV  sous forme de table (1 CSV = 1 table du nom du ficheir CSV)
- `failOnFirstSqlError` : si `True`, le script s’arrête dès qu’une requête SQL échoue  
- `failOnFirstCsvError` : si `True`, le script s’arrête dès qu’un import CSV dans la base de données échoue  

### 2. Créer la base de données et le schéma
Dans DBeaver, exécuter deux requêtes qui permettront de créer une base de données et un schéma dédié. Les informations que vous transmettez à SQL doivent correspondre aux éléments `pgDatabase` et `pgSchemaImportsCsv` du fichier .env.

Créer une nouvelle base de données 
```sql
CREATE DATABASE {database_name} ;
```

Créer un nouveau schéma
```sql
CREATE SCHEMA {schema_name} ;
```

### 3. Lancer le script principal
```bash
python run.py
```
ou selon la configuration :
```bash
python3 run.py
```

---

## Fonctionnement
- Au lancement, le script :
  1. Charge les variables du fichier `.env`.
  2. Vérifie si la base `pgDatabase` existe, sinon la crée.
  3. Importe tous les fichiers CSV du dossier `csv/` dans le schéma `pgSchemaImportsCsv`.
  4. Exécute tous les fichiers SQL du dossier `sql/`.

- Les logs affichent :
  - la création de la base si nécessaire ;
  - l’exécution des requêtes SQL ;
  - l’import des fichiers CSV ;
  - les erreurs éventuelles (selon les paramètres `failOnFirstSqlError` et `failOnFirstCsvError`, le script peut s'arrêter immédiatement).
