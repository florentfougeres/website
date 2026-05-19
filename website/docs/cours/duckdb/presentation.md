---
marp: true
theme: default 
# theme: uncover, default, nord, gaia, wave, dracula
# class: invert
paginate: true
style: |
  a {
    color: #003d99;
  }
  a:hover {
    color: #002266;
  }
  h2 {
    color: #0066cc;
  }
---

# Traitement de données géospatiales
# avec Parquet et DuckDB

---

# 👋 Florent Fougères

## Développeur SIG chez [Oslandia](https://oslandia.com)

### <img src="https://www.svgrepo.com/show/512317/github-142.svg" height="40"> [@florentfougeres](https://github.com/florentfougeres)
### <img src="https://www.svgrepo.com/show/448226/gitlab.svg" height="40"> [@florentfougeres](https://gitlab.com/florentfougeres)
### <img src="https://www.svgrepo.com/show/349354/email.svg" height="40"> [florent.fougeres@gmail.com](mailto:florent.fougeres@gmail.com)
### <img src="https://www.svgrepo.com/show/452047/linkedin-1.svg" height="40"> [Florent Fougères](https://www.linkedin.com/in/florent-fougeres/)

---

## Plan du cours

1. Introduction au format Parquet
2. Le SGBD DuckDB : présentation et caractéristiques
3. Performances et benchmarks
4. Cas d'usage en SIG
5. La complémentarité Parquet + DuckDB
6. Exercices pratiques

---

## 1. Introduction au format Parquet

![bg right:50% fit](https://cdn.dribbble.com/userupload/19991823/file/original-efc9605e60a43bab4ac37e1fed1c5ebc.png)

---

## Qu'est-ce que Parquet ?

- **Format de fichier open source**
- Développé par la **Fondation Apache**
- Conçu pour stocker et accéder à de **grandes quantités de données**
- **Architecture orientée colonnes**
- **Compression efficace**
- Extension **Geoparquet** pour les géométries spatiales

---

## Architecture orientée colonnes

**Format traditionnel (orienté lignes) :**
```
Ligne 1: nom, prénom, ville, code_postal
Ligne 2: nom, prénom, ville, code_postal
Ligne 3: nom, prénom, ville, code_postal
```

**Parquet (orienté colonnes) :**
```
Colonne nom: [valeur1, valeur2, valeur3, ...]
Colonne prénom: [valeur1, valeur2, valeur3, ...]
Colonne ville: [valeur1, valeur2, valeur3, ...]
```

---

## Avantages de l'architecture colonnes

✅ **Meilleure compression**
- Les données d'une même colonne sont souvent similaires

✅ **Lecture optimisée**
- Ne lit que les colonnes nécessaires

✅ **Performance analytique**
- Idéal pour les agrégations et statistiques

---

## Organisation en dataset

Un dataset Parquet peut être un **dossier de fichiers** :

```
/mon_dataset/
  ├── partie_1.parquet
  ├── partie_2.parquet
  ├── partie_3.parquet
  └── partie_4.parquet
```

**Avantages :**
- Traitement en parallèle (chunks)
- Gestion efficace de gros volumes
- Meilleure performance globale

---

## Geoparquet

> https://geoparquet.org

Extension de Parquet pour les **données géospatiales**

- Stockage des géométries spatiales
- Compatible avec tous les types géométriques
- Métadonnées spatiales (CRS, bbox, etc.)
- Tous les avantages de Parquet préservés

---

## Benchmark : Comparaison des formats

**Données :** Localités mondiales > 1 000 habitants

| Format | Taille du fichier |
|--------|------------------|
| GeoJSON | ~100 MB |
| CSV | ~80 MB |
| **Parquet** | **~20 MB** |

**→ Gain de taille : 5x plus compact que GeoJSON**

---

## Limite importante de Parquet

⚠️ **Parquet = format d'échange**

- Pas un format de travail
- Pas un format de traitement
- Nécessite un système optimisé pour l'exploiter

**Solution : DuckDB**

---

# 02
## Le SGBD DuckDB : présentation et caractéristiques

<center>
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/40/DuckDB_logo.svg/960px-DuckDB_logo.svg.png" height="300">
</center>

---

## Qu'est-ce que DuckDB ?

**Système de gestion de base de données relationnelle (SGBDR)**

- Écrit en **C++**
- Licence **MIT** (open source)
- Base de données en **fichier portable**
- Version stable : **1.4.4**
- **Sans serveur** (serverless)

---

## Pourquoi choisir DuckDB ?

✅ **Sans serveur (Serverless)**
- Pas de processus serveur à gérer
- Configuration minimale

✅ **Installation rapide**
- Mise en place en quelques minutes

✅ **Fichier portable**
- Facilite le partage et la sauvegarde

---

## Pourquoi choisir DuckDB ? (suite)

✅ **Langage SQL standard**
- Facile si vous connaissez SQL
- Nombreuses fonctions analytiques

✅ **Lecture directe de fichiers**
- Requêtes directes sur Parquet, CSV, etc.
- Pas d'import nécessaire

✅ **Extension spatiale**
- Fonctions GIS basées sur GEOS
- Compatible avec PostGIS

---

## Extension spatiale de DuckDB

**Fonctionnalités avancées pour le SIG :**

- Grande variété de fonctions spatiales (GEOS)
- **Noms identiques à PostGIS** (migration facilitée)
- **Indexation spatiale**
- Lecture/écriture de nombreux formats
- Support des **drivers GDAL**

---

## Installation de l'extension spatial

```duckdb
INSTALL spatial; 
```

> A faire qu'une seule fois sur le poste

## Chargement de l'extension spatial

```duckdb
LOAD spatial; 
```

> Il s'agit de l'équivalent du `CREATE EXTENSION postgis` sur une base dans PostgreSQL

---

## OLAP vs OLTP

**DuckDB est optimisé pour l'OLAP**

https://fr.wikipedia.org/wiki/Traitement_analytique_en_ligne

> En informatique, et plus particulièrement dans le domaine des bases de données, le traitement analytique en ligne (anglais online analytical processing, OLAP) est un type d'application informatique orienté vers l'analyse sur-le-champ d'informations.

- OLAP: Traitement analytique en ligne (*online analytical processing*)
- OLTP: Traitement transactionnel en ligne (*online transaction processing*)

---

## OLAP vs OLTP

| OLAP (DuckDB) | OLTP (PostgreSQL) |
|---------------|-------------------|
| Requêtes analytiques complexes | Transactions rapides |
| Lecture de gros volumes | Nombreuses écritures |
| Peu de mises à jour | Mises à jour fréquentes |
| Agrégations et analyses | Intégrité transactionnelle |

---

## Performance : Architecture colonnes

**DuckDB excelle pour :**

- Traitement de **grands volumes** de données
- Données de **plusieurs gigaoctets**
- Tables de **plusieurs millions de lignes**
- **Requêtes analytiques** et agrégations

---

## Benchmark : DuckDB vs PostGIS

**Données :** 13,1 millions de bâtiments (GeoJSON 6,3 GB)

| Opération | DuckDB | PostGIS |
|-----------|---------|---------|
| Intégration | 4 min | 5 min 30 s |
| Calcul de buffer | **38 s** | 1 min 54 s |
| Ajout colonne + surface | **2 s** | 2 min 58 s |

⚠️ **DuckDB ne remplace pas PostGIS**
Ils ont des cas d'usage différents et complémentaires.

---

## Limitations de DuckDB (1/2)

❌ **Pas de support des projections**

- Pas de transformation de systèmes de coordonnées
- Données doivent être dans la même projection

❌ **Mono-utilisateur et local**

- Usage local uniquement
- Pas de multi-utilisateurs simultanés

❌ **Verrouillage en lecture**

- Base verrouillée lors des lectures

---

## Limitations de DuckDB (2/2)

❌ **Pas de gestion des droits**

- Pas de système de permissions
- Pas de gestion de rôles

❌ **Projet jeune**

- Évolution rapide
- API peut changer

❌ **OLAP uniquement**

- Pas pour : écritures fréquentes, transactions concurrentes, haute disponibilité

---

## Utilisation de DuckDB

### Interface en ligne de commande (CLI)
- [Linux](https://duckdb.org/install/?platform=linux&environment=cli), [macOS](https://duckdb.org/install/?platform=macos&environment=cli), [Windows](https://duckdb.org/install/?platform=windows&environment=cli)

### Langages de programmation
- [Python](https://duckdb.org/install/?platform=linux&environment=python), [R](https://duckdb.org/install/?platform=linux&environment=r), Java, Node.js, Rust...

### Dans QGIS

- [QDuckDB](https://plugins.qgis.org/plugins/qduckdb/)
  
---

## Utilisation de DuckDB

### IDE
- DBeaver
- DataGrip (freeware)
- [Beekeeper Studio](https://www.beekeeperstudio.io/fr/)

### Web
Permet grace à [DuckDB WASM](https://duckdb.org/docs/stable/clients/wasm/overview)
- [DuckDB Shell](https://shell.duckdb.org)
- [DuckUI](https://demo.duckui.com/)
- Avec la [CLI](https://duckdb.org/docs/stable/core_extensions/ui) via ` duckdb -ui`


---

# 03
## Cas d'usage en SIG

---

## Plugin QDuckDB pour QGIS

**Intégration de DuckDB dans QGIS**

- Développement financé par l'**IFREMER**
- Mode **lecture seule** actuellement
- Chargement de couches depuis DuckDB
- Disponible dans le gestionnaire d'extensions QGIS

📺 [Vidéo de démonstration](https://video.osgeo.org/w/1YGiDEgqK29RAo9TmtCjc1)

---

## Exemple 1 : Créer une table géographique depuis un CSV

**Créer une couche spatiale depuis un CSV avec X,Y :**

```sql
SELECT 
    *,
    ST_Point(x, y) as geometry
FROM read_csv('fichier.csv')
WHERE x IS NOT NULL 
  AND y IS NOT NULL;
```

---

## Exemple 2 : Conversion Shapefile → GPKG

**Convertir un Shapefile en GeoPackage :**

```sql
COPY (
    SELECT * FROM ST_Read('input.shp')
) TO 'output.gpkg' 
WITH (FORMAT GDAL, DRIVER 'GPKG');
```

---

## Exemple 3 : Parquet → Shapefile avec filtre

**Créer un Shapefile depuis Parquet avec filtre spatial :**

```sql
COPY (
    SELECT * 
    FROM read_parquet('data.parquet')
    WHERE ST_Within(
        geometry,
        ST_MakeEnvelope(xmin, ymin, xmax, ymax)
    )
) TO 'filtered.shp' 
WITH (FORMAT GDAL, DRIVER 'ESRI Shapefile');
```

---

## Génération de tuiles vectorielles

**Nouveauté depuis DuckDB 1.4**

✅ **Avantages :**
- Performance élevée pour la génération de tuiles
- Intégration facile dans des pipelines
- Alternative légère aux solutions traditionnelles

📂 [Exemple par Max Gabrielsson](https://gist.github.com/Maxxen/37e4a9f8595ea5e6a20c0c8fbbefe955)

---

# 04
## Complémentarité Parquet + DuckDB

---

## Le duo gagnant

**Format ouvert (Parquet)**
- Stockage compact
- Compression efficace
- Portabilité

**+**

**Base de données in-process (DuckDB)**
- Requêtes SQL puissantes
- Performance élevée
- Facilité d'utilisation

**= Efficacité maximale**

---

## Workflow recommandé

**1. Stockage en Parquet**
- Compression et archivage
- Partage et échange de données

**2. Traitement avec DuckDB**
- Analyses et requêtes complexes
- Transformations et conversions

**3. Export vers format métier**
- GeoPackage pour QGIS
- Shapefile pour compatibilité
- GeoJSON pour le web
- PostgreSQL/PostGIS pour production

---

## Cas d'usage idéaux ✅

**DuckDB + Parquet est parfait pour :**

- Analyse exploratoire de données (EDA)
- Prototypage rapide d'analyses spatiales
- Traitement batch de grandes données
- Conversions de formats en masse
- Génération de rapports analytiques
- Scripts ETL
- Travail en local sur données volumineuses
- Recherche et développement

---

## Cas d'usage à éviter ❌

**Ne PAS utiliser pour :**

- Applications web avec accès concurrent
- Systèmes nécessitant haute disponibilité
- Applications transactionnelles (CRUD intensif)
- Cas nécessitant gestion fine des droits utilisateurs

---

# 05
## Conclusion

---

## Points clés à retenir

1. **Parquet** : format de stockage efficace pour grandes données
2. **DuckDB** : SGBD performant pour l'analyse volumineuse
3. **Extension spatiale** : capacités GIS avancées
4. **Parquet + DuckDB** : duo idéal pour l'analyse géospatiale
5. **Complémentarité** : ne remplace pas PostgreSQL/PostGIS 

---

## Perspectives

**DuckDB est un projet jeune et dynamique**

- Évolution rapide
- Écosystème grandissant
- Nouvelles fonctionnalités régulières
- Intégrations multiples

**→ Un outil à surveiller de près dans le domaine géospatial**

---
## Site & Documentation officielle

| Ressource | Lien |
|-----------|------|
| 🏠 Site officiel | [duckdb.org](https://duckdb.org) |
| 📖 Documentation | [duckdb.org/docs](https://duckdb.org/docs) |
| 🗺️ Extension spatiale | [docs/extensions/spatial](https://duckdb.org/docs/extensions/spatial) |
| 💻 GitHub | [duckdb/duckdb](https://github.com/duckdb/duckdb) |

---

## Outils & Plugins

- **Awesome DuckDB** — Une liste sélectionnée de bibliothèques, d'outils et de ressources DuckDB  
  → [github.com/davidgasquez/awesome-duckdb](https://github.com/davidgasquez/awesome-duckdb)

- **Plugin QGIS QDuckDB** — Intégration native de DuckDB dans QGIS  
  → [plugins.qgis.org/plugins/qduckdb](https://plugins.qgis.org/plugins/qduckdb/)

- **GeoParquet.io** — Outils de transformation rapides pour les fichiers GeoParquet utilisant PyArrow et DuckDB  
  → [geoparquet.io](https://geoparquet.io/)

---

## 📚 Articles & Tutoriels

### Géotribu
- **DuckDB et données spatiales**  
  → [geotribu.fr — DuckDB données spatiales](https://geotribu.fr/articles/2023/2023-12-19_duckdb-donnees-spatiales/)

### icem7 — Série DuckDB

- **Parquet devrait remplacer le format CSV**  
  → [icem7.fr/cartographie/parquet-devrait-remplacer-le-format-csv](https://www.icem7.fr/cartographie/parquet-devrait-remplacer-le-format-csv/)

---

- **3 explorations bluffantes avec DuckDB** *(série en 3 parties)*

  | Épisode | Thème | Lien |
  |---------|-------|------|
  | 1/3 | Interroger des fichiers distants | [lien](https://www.icem7.fr/pedagogie/3-explorations-bluffantes-avec-duckdb-1-interroger-des-fichiers-distants/) |
  | 2/3 | Butiner des API JSON | [lien](https://www.icem7.fr/pedagogie/3-explorations-bluffantes-avec-duckdb-butiner-des-api-json-2-3/) |
  | 3/3 | Croiser les requêtes spatiales | [lien](https://www.icem7.fr/cartographie/3-explorations-bluffantes-avec-duckdb-croiser-les-requetes-spatiales-3-3/) |

- **Comment bien préparer son Parquet**  
  → [icem7.fr/outils/comment-bien-preparer-son-parquet](https://www.icem7.fr/outils/comment-bien-preparer-son-parquet/)