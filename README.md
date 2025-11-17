# OpenClassrooms Projet 11 - AWS Big Data

## Description

Projet de mise en place d'une infrastructure Big Data scalable sur AWS pour la startup **Fruits!**, développant une application mobile de classification de fruits.

## Contexte

La startup Fruits! souhaite développer une application mobile permettant aux utilisateurs de prendre en photo un fruit et d'obtenir des informations sur celui-ci. Pour construire cette fonctionnalité de classification, l'entreprise a besoin d'une infrastructure Big Data capable de traiter des millions d'images de manière scalable et économique.
## Lien vers le jeux de données 

[Jeu de données fruits](https://www.kaggle.com/moltean/fruits)

## Objectifs

- Compléter un notebook PySpark pour l'extraction de features et la réduction de dimensionnalité
- Déployer la solution sur AWS EMR (Elastic MapReduce)
- Respecter les contraintes RGPD (serveurs en UE)
- Maintenir un budget < 10 EUR

## Technologies utilisées

- **AWS EMR** : Cluster Spark managé
- **PySpark** : Traitement distribué des données
- **TensorFlow / MobileNetV2** : Extraction de features via deep learning
- **S3** : Stockage des données et résultats
- **Région AWS** : eu-west-3 (Paris) pour conformité RGPD

## Architecture

```
Images (S3) → PySpark → MobileNetV2 (Feature Extraction) → PCA → Résultats (S3)
```

## Fonctionnalités principales

### 1. Broadcast des poids du modèle
- Chargement du modèle TensorFlow une seule fois sur le driver
- Broadcast des poids vers tous les workers
- **Résultat** : Accélération ~6x du traitement (30 min → 5 min)

### 2. Réduction de dimensionnalité avec PCA
- Réduction des features de 1280 à 50 dimensions
- **Résultat** : Compression ~25x du stockage (5 GB → 200 MB pour 1M d'images)
- Conservation de 85-90% de la variance

## Fichiers du projet

- **bootstrap-emr.sh** : Script de bootstrap pour installer les librairies Python sur le cluster EMR (TensorFlow, Pillow, pandas, etc.)
- **P8_Notebook_Linux_EMR_PySpark_V1.0.ipynb** : Notebook principal contenant le code PySpark pour l'extraction de features et la PCA
- **Presentation_vidéo_aws.mp4** : Vidéo de démonstration de l'implémentation AWS
- **config.json** : Configuration Spark et Jupyter pour le cluster EMR
- **jupyter-s3-conf.json** : Configuration de persistance Jupyter sur S3

## Installation et déploiement

### Prérequis
- Compte AWS configuré
- Bucket S3 créé (ex: `fruit-projet-11-feniou`)
- Rôles IAM configurés (EMR_DefaultRole, EMR_EC2_DefaultRole)

### Étapes de déploiement

1. **Créer un cluster EMR**
   ```bash
   # Région: eu-west-3 (Paris)
   # Configuration: 1 master + 2-4 core nodes (m5.xlarge)
   # Applications: Spark, Hadoop, JupyterHub
   ```

2. **Uploader les fichiers sur S3**
   ```bash
   aws s3 cp bootstrap-emr.sh s3://votre-bucket/scripts/
   aws s3 cp P8_Notebook_Linux_EMR_PySpark_V1.0.ipynb s3://votre-bucket/notebooks/
   ```

3. **Configurer le bootstrap lors de la création du cluster**
   - Ajouter `bootstrap-emr.sh` comme action de bootstrap

4. **Exécuter le notebook**
   - Se connecter à JupyterHub via l'interface EMR
   - Ouvrir et exécuter le notebook

## Résultats

- **Performance** : Traitement de 10 000 images en ~5 minutes avec broadcast
- **Coût** : < 10 EUR pour l'ensemble du projet (optimisation via arrêt du cluster)
- **Stockage** : Réduction de 96% de l'espace de stockage grâce à la PCA
- **RGPD** : Conformité assurée avec la région eu-west-3

## Gestion des coûts

**Important** : Pour respecter le budget de 10 EUR :
- Arrêter le cluster EMR immédiatement après utilisation
- Utiliser des instances Spot si possible
- Monitorer les coûts via AWS Cost Explorer
- Supprimer les données temporaires sur S3


