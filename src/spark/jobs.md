# Jobs Spark

## Vue d'ensemble

Dans Apache Spark, les **jobs** sont les unités fondamentales d'exécution du travail. Comprendre la relation entre les actions et les jobs est essentiel pour optimiser les applications Spark et interpréter les métriques d'exécution.

## Concept de base

**Un job Spark est créé à chaque fois qu'une action est invoquée dans le code de votre application.** Il ne s'agit pas simplement d'une abstraction conceptuelle—cela représente des frontières d'exécution réelles dans votre programme.

## La relation Action-Job

### Ce qui déclenche un job

Les actions sont des opérations qui retournent des résultats au programme driver ou écrivent des données vers un stockage externe. Lorsque Spark rencontre une action, il doit matérialiser le calcul défini par les transformations précédentes, créant et exécutant ainsi un job.

Les actions courantes incluent le comptage d'enregistrements, la collecte de données vers le driver, l'écriture sur disque, l'affichage de résultats, et l'exécution d'agrégations qui retournent des valeurs.

### Ce qui ne déclenche pas un job

Les transformations définissent des opérations sur les données mais ne déclenchent pas d'exécution en raison du modèle d'évaluation paresseuse de Spark. Les opérations comme le filtrage, le mapping, les jointures et les groupements s'accumulent dans un plan d'exécution logique sans créer de jobs. Le calcul reste différé jusqu'à ce qu'une action force l'évaluation.

## Composition d'un job

Chaque job encapsule :
- Toutes les transformations accumulées depuis la dernière action (ou depuis le démarrage du programme)
- Un graphe acyclique dirigé (DAG) représentant les dépendances entre opérations
- Une division en stages, séparés par des frontières de shuffle
- Une distribution des tâches à travers le cluster

## Flux d'exécution

Lorsqu'une action est appelée, Spark suit cette séquence :

1. **Création du job** : L'action déclenche la création d'un nouveau job avec un identifiant unique
2. **Analyse du DAG** : Spark analyse la chaîne de transformations pour construire le plan logique
3. **Planification physique** : Le plan logique est converti en plan d'exécution physique
4. **Division en stages** : Le plan est divisé en stages aux points où les données doivent être redistribuées (shuffles)
5. **Ordonnancement des tâches** : Chaque stage est divisé en tâches qui s'exécutent en parallèle à travers les partitions
6. **Exécution** : Les tâches s'exécutent sur les executors, et les résultats remontent selon le type d'action

## Plusieurs actions = Plusieurs jobs

Chaque action crée un job indépendant, même lorsqu'elle opère sur le même DataFrame ou RDD. Cela signifie que plusieurs actions en séquence génèreront plusieurs jobs, chacun avec sa propre surcharge d'exécution.

Si votre application appelle cinq actions différentes, vous observerez cinq jobs distincts dans l'interface Spark, chacun avec ses propres métriques de temps et d'utilisation des ressources.

## Implications pour la performance

### Considérations sur le cache

Lorsque plusieurs actions opèrent sur le même jeu de données, envisagez de mettre en cache le résultat intermédiaire. Sans cache, chaque action déclenche un recalcul depuis les données source, créant du travail redondant à travers les jobs.

### Stratégie de placement des actions

Minimisez les actions inutiles pendant le développement et le débogage. Chaque action entraîne une surcharge pour l'ordonnancement, la planification et la coordination du job. Consolidez les opérations lorsque possible pour réduire le nombre total de jobs.

### Surveillance et débogage

L'interface Spark affiche les jobs dans l'ordre d'exécution, facilitant la corrélation entre les actions du code et les jobs observés. Chaque entrée de job montre la durée, les stages, les tâches et les métriques de données, facilitant l'analyse de performance.

## Relation avec le plan d'exécution

Le plan d'exécution est la représentation interne de Spark de **comment** un job sera exécuté, et non **quand** il sera créé. Le plan est généré après la création du job, traduisant les opérations de haut niveau en étapes d'exécution de bas niveau.

Vous pouvez examiner le plan d'exécution sans déclencher de job en utilisant les opérations explain, qui montrent la stratégie d'exécution planifiée sans réellement exécuter le calcul.

## Bonnes pratiques

- **Soyez intentionnel sur le placement des actions** : Chaque action a un coût
- **Utilisez l'interface Spark** : Surveillez le ratio jobs/actions dans votre application
- **Exploitez l'évaluation paresseuse** : Enchaînez les transformations efficacement avant de déclencher l'exécution
- **Mettez en cache stratégiquement** : Persistez les jeux de données qui alimentent plusieurs actions
- **Considérez les alternatives d'actions** : Certaines actions sont plus coûteuses que d'autres pour atteindre le même objectif

## Résumé

Les jobs sont des unités d'exécution concrètes créées explicitement par les actions dans votre code Spark. Ce ne sont pas des concepts abstraits mais de réelles soumissions de travail au planificateur Spark. Comprendre cette relation permet une meilleure conception d'application, un débogage plus efficace et une optimisation améliorée de la performance.
