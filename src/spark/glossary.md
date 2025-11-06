# Apache Spark - Glossaire 

## A

**Action**  
Opération qui déclenche l'exécution d'un job et retourne un résultat au driver ou écrit des données vers un système de stockage externe. Contrairement aux transformations, les actions forcent l'évaluation immédiate du DAG accumulé. Exemples : count(), collect(), save(), show().

**Agrégation**  
Opération qui combine plusieurs valeurs pour produire un résultat unique ou réduit. Dans Spark, les agrégations peuvent être des transformations (groupBy()) ou des actions (reduce(), aggregate()) selon qu'elles retournent un résultat final ou un dataset intermédiaire.

## C

**Cache / Caching**  
Mécanisme de persistance en mémoire qui stocke les résultats intermédiaires d'un RDD ou DataFrame pour éviter les recalculs lors d'actions multiples sur les mêmes données. Améliore significativement les performances lorsque des données sont réutilisées.

**Cluster**  
Ensemble de machines (nœuds) travaillant ensemble pour exécuter des applications Spark de manière distribuée. Composé d'un nœud driver et de plusieurs nœuds executors.

## D

**DAG (Directed Acyclic Graph / Graphe Acyclique Dirigé)**  
Structure de données représentant les dépendances entre les opérations Spark. Chaque nœud représente un RDD ou DataFrame, et chaque arête représente une transformation. "Acyclique" signifie qu'il n'y a pas de boucles, garantissant un flux de données unidirectionnel.

**DataFrame**  
Structure de données distribuée organisée en colonnes nommées, similaire à une table de base de données ou un dataframe pandas. Construit au-dessus des RDD avec des optimisations supplémentaires via Catalyst optimizer.

**Driver**  
Programme principal qui exécute la fonction main de l'application Spark. Coordonne l'exécution des jobs, maintient les informations sur l'application et planifie les tâches sur les executors.

## E

**Évaluation paresseuse (Lazy Evaluation)**  
Stratégie d'exécution où les transformations ne sont pas calculées immédiatement mais sont enregistrées dans un plan d'exécution. Le calcul réel est différé jusqu'à ce qu'une action soit appelée, permettant des optimisations globales.

**Executor**  
Processus lancé sur un nœud worker qui exécute les tâches et stocke les données pour l'application Spark. Chaque application a ses propres executors qui persistent pendant toute la durée de l'application.

## J

**Job**  
Unité d'exécution la plus large dans Spark, créée en réponse à une action. Un job est composé d'un ou plusieurs stages et représente l'ensemble du travail nécessaire pour calculer le résultat d'une action.

**Jointure (Join)**  
Transformation qui combine deux datasets basée sur une ou plusieurs clés communes. Opération coûteuse car elle nécessite souvent un shuffle des données à travers le cluster.

## M

**Mapping**  
Transformation qui applique une fonction à chaque élément d'un dataset pour produire un nouveau dataset. Opération parallélisable qui ne nécessite pas de shuffle.

## P

**Partition**  
Unité logique de données distribuées dans Spark. Un RDD ou DataFrame est divisé en plusieurs partitions qui peuvent être traitées en parallèle sur différents nœuds du cluster. Le nombre de partitions influence directement le parallélisme.

**Persistance**  
Action de sauvegarder un RDD ou DataFrame en mémoire, sur disque, ou une combinaison des deux pour réutilisation ultérieure. Inclut le caching (mémoire uniquement) et d'autres niveaux de stockage.

**Plan d'exécution (Execution Plan)**  
Représentation détaillée de comment Spark va physiquement exécuter une requête. Comprend le plan logique (opérations de haut niveau) et le plan physique (implémentation concrète avec détails sur les stages et shuffles).

**Plan logique (Logical Plan)**  
Représentation abstraite des opérations à effectuer, indépendante de l'implémentation physique. Décrit ce qui doit être fait sans spécifier comment.

**Plan physique (Physical Plan)**  
Traduction du plan logique en opérations concrètes exécutables, incluant les décisions sur les algorithmes de jointure, l'ordre des opérations et les points de shuffle.

## R

**RDD (Resilient Distributed Dataset)**  
Structure de données fondamentale de Spark représentant une collection immuable d'objets distribuée sur le cluster. Tolérante aux pannes grâce au lignage qui permet la reconstruction en cas de perte de partition.

## S

**Shuffle**  
Redistribution coûteuse des données à travers les partitions du cluster. Se produit lors d'opérations comme groupBy, join, ou repartition. Implique l'écriture sur disque, le transfert réseau et la lecture, ce qui en fait l'opération la plus lente dans Spark.

**Stage**  
Division d'un job en unités d'exécution séquentielles. Les frontières entre stages sont déterminées par les opérations de shuffle. Toutes les tâches d'un stage peuvent s'exécuter en parallèle car elles n'ont pas de dépendances de shuffle entre elles.

**Stockage externe (External Storage)**  
Système de persistance hors de Spark comme HDFS, S3, bases de données, ou systèmes de fichiers locaux où les données peuvent être lues ou écrites.

## T

**Tâche (Task)**  
Plus petite unité de travail envoyée à un executor. Représente le calcul sur une seule partition de données. Un stage est composé de plusieurs tâches exécutées en parallèle.

**Transformation**  
Opération qui crée un nouveau RDD ou DataFrame à partir d'un existant sans déclencher d'exécution immédiate. Les transformations sont paresseuses et incluent map, filter, groupBy, join, etc.

## W

**Worker**  
Nœud du cluster qui exécute les executors. Fournit les ressources CPU et mémoire pour l'exécution des tâches Spark.

---

## Relations entre concepts clés

- **Job** → composé de **Stages** → composés de **Tâches**
- **Action** → déclenche un **Job** → génère un **Plan d'exécution**
- **Transformation** → construit le **DAG** → exécuté lors d'une **Action**
- **Shuffle** → délimite les **Stages** → coûteux en performance
- **Partition** → unité de **Parallélisme** → traité par une **Tâche**