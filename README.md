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

### 1. Créer et remplir le fichier `.env`
Le fichier doit contenir toutes les variables suivantes :

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

### 2. Placer les fichiers SQL dans `sql/`
- (Créer le dossier `sql/` si inexistant)
- Les fichiers sont exécutés dans l’ordre **alphabétique**.  
- Exemples : `01_init.sql`, `02_schema.sql`, `03_data.sql`.

### 3. Placer les fichiers CSV dans `csv/`
- (Créer le dossier `csv/` si inexistant)
- L’ordre n’a aucune importance.  
- Le **nom du fichier** (sans `.csv`) devient le **nom de la table**. Exemple : `users.csv` → table `users`.

### 4. Lancer le script principal
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
  3. Exécute tous les fichiers SQL du dossier `sql/`.
  4. Importe tous les fichiers CSV du dossier `csv/` dans le schéma `pgSchemaImportsCsv`.

- Les logs affichent :
  - la création de la base si nécessaire ;
  - l’exécution des requêtes SQL ;
  - l’import des fichiers CSV ;
  - les erreurs éventuelles (selon les paramètres `failOnFirstSqlError` et `failOnFirstCsvError`, le script peut s'arrêter immédiatement).

---


## Exemples de logs qu'il est possible d'obtenir

```
2025-11-26 20:22:03.672 | INFO     | Chargement des variables | Début
2025-11-26 20:22:03.674 | INFO     | Chargement des variables | Success
2025-11-26 20:22:03.711 | INFO     | Postgres-global | Engine créé
2025-11-26 20:22:03.778 | INFO     | Postgres-global | Vérification de l'existence de la base test
2025-11-26 20:22:03.779 | INFO     | Postgres-global | La base test n'existe pas, création en cours
2025-11-26 20:22:03.779 | INFO     | Postgres-global | Requête SQL à exécuter : CREATE DATABASE test
2025-11-26 20:22:07.367 | INFO     | Postgres-global | Base test créée avec succès.
2025-11-26 20:22:07.367 | INFO     | Postgres-workingInstance | Engine créé
2025-11-26 20:22:07.444 | INFO     | Postgres-workingInstance | Vérification de l'existence de la base test
2025-11-26 20:22:07.445 | INFO     | Postgres-workingInstance | La base test existe déjà.
2025-11-26 20:22:07.446 | INFO     | SQLDataLoader |

2025-11-26 20:22:07.446 | INFO     | SQLDataLoader | Exécution du fichier SQL : 01.sql
2025-11-26 20:22:07.450 | INFO     | SQLDataLoader | Requête SQL à exécuter : create schema if not exists test;
2025-11-26 20:22:07.452 | INFO     | SQLDataLoader | Success : Requête exécutée correctement
2025-11-26 20:22:07.452 | INFO     | SQLDataLoader | Requête SQL à exécuter : create table if not exists test.testtable(id SERIAL PRIMARY KEY, value TEXT);
2025-11-26 20:22:07.476 | INFO     | SQLDataLoader | Success : Requête exécutée correctement
2025-11-26 20:22:07.476 | INFO     | SQLDataLoader | Requête SQL à exécuter : insert into test.testtable (id, value) values (1, 'a'), (2, 'b') on conflict do nothing;
2025-11-26 20:22:07.487 | INFO     | SQLDataLoader | Success : Requête exécutée correctement
2025-11-26 20:22:07.487 | INFO     | SQLDataLoader |

2025-11-26 20:22:07.487 | INFO     | SQLDataLoader | Exécution du fichier SQL : 02.sql
2025-11-26 20:22:07.488 | INFO     | SQLDataLoader | Requête SQL à exécuter : insert into test.testtable (id, value) values (1,'c'), (2,'d;') on conflict (id) do update set value = excluded.value;
2025-11-26 20:22:07.489 | INFO     | SQLDataLoader | Success : Requête exécutée correctement
2025-11-26 20:22:07.489 | INFO     | SQLDataLoader | Requête SQL à exécuter : insert into test.testtable (id, value) values (5, 'a'), (6, 'b') on conflict do nothing;
2025-11-26 20:22:07.490 | INFO     | SQLDataLoader | Success : Requête exécutée correctement
2025-11-26 20:22:07.490 | INFO     | SQLDataLoader |

2025-11-26 20:22:07.490 | INFO     | SQLDataLoader | Exécution du fichier SQL : 03.sql
2025-11-26 20:22:07.491 | INFO     | SQLDataLoader | Requête SQL à exécuter : commit;
2025-11-26 20:22:07.504 | INFO     | SQLDataLoader | Success : Requête exécutée correctement
2025-11-26 20:22:07.506 | INFO     | CSVDataLoader |

2025-11-26 20:22:07.506 | INFO     | CSVDataLoader | Import du fichier CSV : testcsv.csv
2025-11-26 20:22:07.507 | INFO     | CSVDataLoader | Propriétés CSV détectées : delimiter=,, quotechar=", escapechar=None, doublequote=False, skipinitialspace=False
2025-11-26 20:22:07.511 | INFO     | CSVDataLoader | Début du transfert vers la base de données et la table test
2025-11-26 20:22:07.538 | INFO     | CSVDataLoader | Success : testcsv.csv

```
