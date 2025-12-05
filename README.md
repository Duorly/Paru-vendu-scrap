# Paru-vendu-scrap

Ce projet est un outil de scraping automatisé conçu pour extraire des annonces immobilières du site ParuVendu et les stocker dans une base de données relationnelle.

Il est structuré en plusieurs étapes distinctes : la récupération des liens des annonces, l'extraction des détails de chaque annonce, et enfin l'insertion des données nettoyées dans une base de données MySQL.

## Structure du Projet

Le projet est organisé comme suit :

```
Paru-vendu-scrap/
├── README.md           # Ce fichier
├── db/                 # Dossier pour les fichiers liés à la base de données (si applicable)
└── src/                # Code source principal
    ├── data/           # Dossier contenant les fichiers CSV intermédiaires
    │   ├── annonces_link.csv  # Liens des annonces récupérés (étape 1)
    │   └── results.csv        # Détails complets des annonces (étape 2)
    ├── scrap_step_1.ipynb     # Notebook pour récupérer les liens des annonces
    ├── scrap_step_2.ipynb     # Notebook pour scraper les détails de chaque annonce
    └── db-insertion.ipynb     # Notebook pour insérer les données dans la base MySQL
```

## Prérequis

Avant de lancer le projet, assurez-vous d'avoir installé les éléments suivants :

*   **Python 3.x**
*   **MySQL Server** (local ou distant)
*   **Google Chrome** (pour Selenium)

### Bibliothèques Python

Installez les dépendances nécessaires via pip :

```bash
pip install selenium beautifulsoup4 sqlalchemy pymysql pandas webdriver-manager requests
```

## Utilisation

Le processus se déroule en trois étapes séquentielles, chacune correspondant à un notebook Jupyter dans le dossier `src/`.

### Étape 1 : Récupération des Liens (`src/scrap_step_1.ipynb`)

Ce script parcourt les pages de résultats de recherche sur ParuVendu (immobilier vente en Ile-de-France par défaut) et extrait les URL de toutes les annonces trouvées.

*   **Entrée** : Aucune (paramètres de recherche définis dans le code).
*   **Sortie** : `src/data/annonces_link.csv` contenant la liste des URLs.
*   **Fonctionnement** : Il itère sur les pages (pagination) et utilise des requêtes HTTP pour récupérer les liens.

### Étape 2 : Scraping des Détails (`src/scrap_step_2.ipynb`)

Ce script lit le fichier CSV généré à l'étape précédente et visite chaque lien pour extraire les informations détaillées de l'annonce.

*   **Entrée** : `src/data/annonces_link.csv`
*   **Sortie** : `src/data/results.csv` contenant les données structurées de chaque annonce.
*   **Fonctionnement** :
    *   Utilise **Selenium** pour gérer le contenu dynamique (cliquer sur "Lire plus", accepter les cookies).
    *   Utilise **BeautifulSoup** pour parser le HTML statique.
    *   Extrait : Titre, prix, description, localisation, surface, nombre de pièces, informations sur l'agence, caractéristiques (balcon, parking, etc.).

### Étape 3 : Insertion en Base de Données (`src/db-insertion.ipynb`)

Ce script prend le fichier CSV final, nettoie les données et les insère dans une base de données MySQL.

*   **Entrée** : `src/data/results.csv`
*   **Sortie** : Données insérées dans la base de données `paruvendu`.
*   **Configuration** : Modifiez la variable `DATABASE_URL` pour correspondre à vos identifiants MySQL (par défaut : `mysql+pymysql://root:password@127.0.0.1:3306/paruvendu`).
*   **Modèle de Données** :
    *   **announcer** (Agency) : Informations sur l'agence ou le vendeur.
    *   **announcements** (Announcement) : Informations principales de l'annonce (prix, titre, ref, etc.).
    *   **estate** (Caracteristic) : Détails spécifiques du bien (pièces, surface, DPE, etc.).

## Base de Données

Le projet utilise **SQLAlchemy** comme ORM. Les tables sont créées automatiquement si elles n'existent pas.

*   **Agency (`announcer`)** : Stocke les agences immobilières pour éviter les duplications.
*   **Announcement (`announcements`)** : Liée à une agence. Contient les infos clés.
*   **Caracteristic (`estate`)** : Liée à une annonce. Contient les détails techniques.
