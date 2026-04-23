# Bloc1 - Data Collection & Management

# Projet KAYAK

## 1. Objectif du projet

Construire une base de donnees exploitable par KAYAK pour recommander:
- les meilleures destinations de vacances en France
- les meilleurs hotels dans ces destinations

Les recommandations s'appuient sur des donnees reelles:
- geolocalisation des villes
- meteo previsionnelle
- donnees d'hotels issues de Booking

## 2. Livrables attendus

- Un fichier CSV enrichi dans S3 contenant les donnees villes + meteo + hotels.
- Une base SQL sur AWS RDS contenant les memes donnees nettoyees que S3.
- Deux cartes:
  - Top 5 destinations (score meteo)
  - Top 20 hotels

## 3. Perimetre fonctionnel (features)

- Selection d'un scope de villes francaises.
- Geocodage (latitude/longitude) des villes.
- Extraction meteo sur 7 jours et calcul d'un score meteo.
- Classement des destinations par attractivite meteo.
- Scraping des hotels Booking pour les meilleures destinations.
- Enrichissement et nettoyage des donnees hotels.
- Generation de cartes PNG pour la visualisation.
- Upload des CSV vers AWS S3.
- ETL depuis S3 vers une base PostgreSQL AWS RDS.

## 4. Architecture du pipeline

1. **Notebook 01**: Scope + geocodage des villes  
2. **Notebook 02**: Meteo 7 jours + scoring + carte Top 5  
3. **Notebook 03**: Scraping Booking + enrichissement + carte Top 20  
4. **Notebook 04**: Envoi des CSV vers AWS S3  
5. **Notebook 05**: ETL S3 -> PostgreSQL AWS RDS

## 5. Stack technique et choix

### Python & notebooks
- **Jupyter Notebook**: ideal pour un workflow data iteratif, pedagogique et facilement demonstrable.
- **Pandas / NumPy**: manipulation tabulaire, nettoyage, calculs de score.

### Collecte de donnees
- **Requests**: appels API (OpenWeatherMap, geocodage selon notebook).
- **Playwright**: scraping Booking avec rendu JavaScript et meilleure robustesse qu'un simple parser HTML.

### Visualisation
- **Plotly**: cartes interactives et export PNG via Kaleido.
- **Kaleido**: rendu statique des figures Plotly.

### Cloud & stockage
- **AWS S3**: stockage central des artefacts CSV.
- **AWS RDS PostgreSQL**: entrepot SQL cible pour l'equipe d'analyse.

### Connexion / ETL
- **boto3**: acces AWS (S3).
- **SQLAlchemy + psycopg2-binary**: chargement des DataFrames vers PostgreSQL.
- **python-dotenv**: gestion des secrets via variables d'environnement.

## 6. Structure du projet

- `notebooks/01_scope_geocoding_nominatim.ipynb`
- `notebooks/02_weather_scoring_top5_map.ipynb`
- `notebooks/03_booking_scraping_playwright.ipynb`
- `notebooks/04_send_csv_to_aws_s3.ipynb`
- `notebooks/05_etl_s3_to_rds_postgres.ipynb`
- `outputs/data/` : CSV intermediaires et enrichis
- `outputs/maps/` : cartes exportees en PNG
- `outputs/logs/` : journaux d'upload et d'ETL
- `.env.dist` : template des variables d'environnement

## 7. Prerequis

- Python 3.12
- Pipenv
- Acces internet
- Compte AWS avec:
  - un bucket S3
  - une instance RDS PostgreSQL
- Cle API OpenWeatherMap

## 8. Configuration

### 8.1 Installation des dependances

```bash
pipenv install
pipenv run playwright install chromium
```

### 8.2 Variables d'environnement

Copier `.env.dist` vers `.env`, puis renseigner:

```env
OPEN_WEATHER_MAP_API_KEY=your_api_key

AWS_BUCKET_NAME=your_bucket_name
AWS_REGION_NAME=your_region_name
AWS_ACCESS_KEY=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_access_key

AWS_RDS_HOST=your_rds_host
AWS_RDS_PORT=5432
AWS_RDS_USER=your_rds_user
AWS_RDS_PASSWORD=your_rds_password
AWS_RDS_DATABASE=your_database
```

## 9. Execution pas a pas

Executer les notebooks dans cet ordre avec le kernel Pipenv:

1. `notebooks/01_scope_geocoding_nominatim.ipynb`
2. `notebooks/02_weather_scoring_top5_map.ipynb`
3. `notebooks/03_booking_scraping_playwright.ipynb`
4. `notebooks/04_send_csv_to_aws_s3.ipynb`
5. `notebooks/05_etl_s3_to_rds_postgres.ipynb`

## 10. Sorties generees

### Donnees
- `outputs/data/cities_geocoded.csv`
- `outputs/data/cities_weather_7d_raw.csv`
- `outputs/data/cities_weather_scored.csv`
- `outputs/data/top20_booking_hotels_enriched.csv`

### Cartes
- `outputs/maps/top5_destinations_weather_map.png`
- `outputs/maps/top20_booking_hotels_map.png`

### Logs
- `outputs/logs/s3_uploads.csv`
- `outputs/logs/etl_extract_s3.csv`
- `outputs/logs/etl_load_rds.csv`

## 11. Qualite et bonnes pratiques

- Chargement idempotent dans RDS (`replace`) dans le notebook ETL pour faciliter les reruns.
- Verification explicite des variables d'environnement critiques.
- Verification des connexions S3/RDS avant ecriture.
- Journalisation des extractions et chargements pour audit.

## 12. Limites connues

- Le scraping Booking peut etre sensible aux mecanismes anti-bot.
- L'export PNG des cartes Plotly avec tuiles web peut parfois echouer selon le provider.
- Les donnees hotels evoluent dans le temps (disponibilite, score, URL).
- Le scraping peut prendre quelques temps (en general 12mn avec 6 top hotels par pays)

## 13. Pistes d'amelioration

- Ajouter des contraintes SQL (PK, index, unicite).
- Remplacer `replace` par des strategies d'upsert incremental.
- Convertir les notebooks en script `.py`.  
- Orchestrer le pipeline (Airflow).
- Ajouter des tests de qualite de donnees (schema, null checks, doublons).
