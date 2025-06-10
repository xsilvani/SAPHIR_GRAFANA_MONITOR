# SAPHIR_GRAFANA_MONITOR
+Documentation
+Etat au 10/06/2025
+Tests fonctiionnels

# 🌐 Système de consultation des données météo - SAPHIR

Ce dépôt documente le système de **visualisation des données météorologiques multi-source** via **Grafana**, développé dans le cadre du projet **SAPHIR**. Il permet la consultation interactive des mesures issues de différentes stations (MeteoHelix, MeteoWind, etc.) géolocalisées sur carte.

---

## 📌 Objectifs

* Centraliser les données issues de plusieurs types de stations météo.
* Afficher ces stations sur une carte interactive dans Grafana (GeoMap).
* Permettre un accès direct aux données détaillées de chaque station via des dashboards spécifiques.
* Maintenir la configuration sous contrôle de version (Git).

---

## 🧭 Fonctionnalités principales

### ✅ Visualisation GeoMap

* Chaque station est représentée par un marqueur géographique.
* Une **Vcard** (popup) affiche le nom de la station et un **lien dynamique** vers un dashboard.

### ✅ Lien dynamique par station

* Lien généré automatiquement via un **Field Override** ciblant le champ du nom de station.
* URL avec insertion de la variable Grafana `${__value.text}` pour sélectionner la station dans le dashboard cible.

### ✅ Dashboards spécialisés

* Un dashboard par type de station :

  * `barani-helix-ground-station3a-data-plots`
  * `barani-wind-ground-station3a-data-plots`
* Données filtrées dynamiquement grâce aux variables Grafana.

---

## 🛠️ Structure du dépôt

```
.
├── dashboards/
│   ├── barani-helix-dashboard.json
│   ├── barani-wind-dashboard.json
├── datasources/
│   └── postgres-datasource.yaml (optionnel)
├── README.md
```

---

## 🚀 Déploiement local (manuel)

1. Installer Grafana localement ou via Docker.
2. Importer les dashboards JSON via l'interface Grafana (Settings > Import).
3. S’assurer que la base de données PostgreSQL et les tables nécessaires sont accessibles.
4. Configurer les sources de données si nécessaire via le fichier YAML ou directement dans l’UI.

---

## 📋 Recommandations

* **Un override par champ nommé** utilisé dans une Vcard.
* Regrouper les dashboards dans des sous-dossiers par type de station.
* Versionner tous les JSON Grafana pour permettre un suivi des évolutions.

---

## 🔗 Liens utiles

* [Grafana Documentation](https://grafana.com/docs/)
* [Expressions régulières Grafana](https://grafana.com/docs/grafana/latest/variables/advanced-variable-format-options/#regex)

---

## 👤 Auteur

* Xavier Silvani
* Projet SAPHIR
* Mise à jour : Juin 2025
# ✅ Solution validée : Lien dynamique depuis GeoMap vers dashboard Grafana

## 🌍 Contexte général : Monitoring d’un réseau météo multi-source

Le projet SAPHIR vise à surveiller en temps réel un réseau de stations météorologiques hétérogènes (multi-fabricants, multi-technologies), réparties sur plusieurs sites. Ce réseau inclut notamment des stations de type **MeteoHelix** et **MeteoWind**, chacune fournissant des données spécifiques (température, vent, humidité, etc.).

Pour une visualisation centralisée et intuitive, les données sont intégrées dans des tableaux de bord **Grafana**, avec une carte interactive (**GeoMap**) qui géolocalise chaque station et permet d’accéder dynamiquement à ses données détaillées.

## 🔢 Objectif spécifique

Chaque station affichée sur la carte GeoMap doit permettre, via un lien cliquable dans sa **Vcard** (popup), d’ouvrir un dashboard dédié avec la station automatiquement sélectionnée.

* **Champs identifiant stations :**

  ```sql
  name AS MeteoHelix_Station_Name
  name AS MeteoWind_Station_Name
  ```

* **Variables des dashboards cibles :**

  ```
  Barani_Helix_station_name
  Barani_Wind_station_name
  ```

## 🔄 Problème initial

* L’utilisation naïve des liens dans les panneaux GeoMap produit plusieurs liens par station (lat, lon, nom, etc.).
* Le placeholder `${__value.text}` appliqué sans filtrage renvoie parfois des valeurs incorrectes.

## ✅ Solution détaillée (propre et dynamique)

### 1. Nettoyage de la requête SQL

Sélectionner les champs requis dans chaque couche GeoMap :

```sql
SELECT
  lat,
  lon,
  name AS MeteoHelix_Station_Name
FROM station_metadata
WHERE name LIKE '%MeteoHelix%';
```

```sql
SELECT
  lat,
  lon,
  name AS MeteoWind_Station_Name
FROM station_metadata
WHERE name LIKE '%MeteoWind%';
```

### 2. Création du lien dynamique via **Field Override**

* Dans Grafana > panneau GeoMap > onglet **Field overrides** :

  * Créer **un override distinct pour chaque champ affiché dans la Vcard** (ex : `MeteoHelix_Station_Name`, `MeteoWind_Station_Name`).
  * Ajouter une propriété **Data links** dans chaque override.

#### Exemple d’URL dynamique :

Pour les stations **Helix** :

```text
http://saphirdataservice.ovh:3000/d/aee7ypolf11c0a/barani-helix-ground-station3a-data-plots?orgId=1&var-Barani_Helix_station_name=${__value.text}
```

Pour les stations **Wind** :

```text
http://saphirdataservice.ovh:3000/d/beebp9cewzif4c/barani-wind-ground-station3a-data-plots?orgId=1&var-Barani_Wind_station_name=${__value.text}
```

> ✅ Cocher l’option **Open in new tab** si souhaité.

## 🔸 Résultat attendu

* Un seul lien cliquable par station (dans sa Vcard).
* Lien redirigeant vers un dashboard dédié avec la variable de station automatiquement renseignée.
* Données affichées dynamiquement selon la station ciblée.

## 🌟 Notes supplémentaires

* Compatible avec Grafana v9 et v10+
* Pas besoin de champs calculés ou de manipulations HTML
* Facilement extensible à d’autres types de stations (ex : pluviomètres, capteurs solaires)
* Versionner les dashboards et la configuration via Git :

### 🔗 Dépôt GitHub recommandé

Utiliser un dépôt Git pour stocker les fichiers `.json` exportés de Grafana et les configurations associées :

[GitHub - Saphir Grafana Config](https://github.com/ton-compte/saphir-grafana-config)

===============================================



=============================================

````markdown name=NOTE_TESTS_FONCTIONNELS_PREPROD.md
# 🧪 Note de Guidage pour les Tests Fonctionnels avant Mise en Production

## 📋 Résumé de la fonctionnalité livrée

- **Lien dynamique depuis chaque station sur la GeoMap Grafana** ouvrant le dashboard correspondant, avec la station automatiquement sélectionnée via variable d’URL.
- Adapté aux stations **MeteoHelix** et **MeteoWind** (extensible).
- Intégration propre : 1 seul lien par station, pas de doublons, pas de manipulation HTML.
- Historique et configuration versionnés dans un dépôt GitHub dédié.

---

## 🔎 Points à vérifier lors des tests fonctionnels

### 1. **Affichage sur la carte GeoMap**
- [ ] Chaque station du réseau (Helix, Wind) apparaît bien sur la carte.
- [ ] La Vcard de chaque station affiche le nom attendu (MeteoHelix_Station_Name ou MeteoWind_Station_Name).

### 2. **Lien cliquable dans la Vcard**
- [ ] Il y a **un seul lien cliquable** par station dans la Vcard.
- [ ] Le lien pointe vers le bon dashboard dédié (Helix ou Wind selon le type de station).
- [ ] Le lien transmet la variable de nom de la station dans l’URL (`var-Barani_Helix_station_name` ou `var-Barani_Wind_station_name`).
- [ ] Option "Open in new tab" activée si besoin.

### 3. **Comportement après clic**
- [ ] Le dashboard s’ouvre sans erreur, dans un nouvel onglet si configuré.
- [ ] La variable de la station est pré-sélectionnée dans le dashboard.
- [ ] Les données affichées correspondent à la station choisie.
- [ ] Les autres fonctionnalités du dashboard restent opérationnelles.

### 4. **Cas particuliers à tester**
- [ ] Stations dont le nom comporte des caractères spéciaux.
- [ ] Stations sans données récentes.
- [ ] Ajout d’un nouveau type de station (test d’extensibilité).

### 5. **Versionning et documentation**
- [ ] Les fichiers `.json` des dashboards exportés sont bien présents et à jour dans le dépôt GitHub de configuration.
- [ ] La documentation utilisateur et technique a été mise à jour pour inclure la nouvelle procédure de création de lien dynamique.

---

## ⚠️ Précautions avant mise en production

- Privilégier un environnement de préproduction avec jeu de données représentatif.
- Vérifier le comportement pour plusieurs utilisateurs simultanés.
- S’assurer de la compatibilité Grafana (v9, v10+).
- Prévoir un rollback et sauvegarder la configuration actuelle.

---

## 📂 Liens utiles

- **Dépôt de configuration Grafana** : [https://github.com/ton-compte/saphir-grafana-config](https://github.com/ton-compte/saphir-grafana-config)

---

## ✅ Checklist finale

- [ ] Recette fonctionnelle réalisée et validée
- [ ] Documentation mise à jour
- [ ] Versionning Git effectué
- [ ] Go/nogo pour déploiement production

---

*Rédigé pour accompagner la livraison de la fonctionnalité « lien dynamique GeoMap → dashboard Grafana » dans le projet SAPHIR.*
````
