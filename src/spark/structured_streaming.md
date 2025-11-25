# Guide de programmation - Structured Streaming API

[Source: Structured Streaming Programming Guide - spark.apache.org](https://spark.apache.org/docs/latest/streaming/index.html)

## Vue d'ensemble

"Structured Streaming" est un moteur de traitement de *datasets* non-bornées (streams) *scalable* et *fault-tolerant* construit par dessus le moteur Spark SQL.

Vous pouvez exprimer vos traitements *streaming* de la même manière que vous exprimeriez un traitement *batch* (en lot) sur des données bornées.
Le moteur Spark SQL prendra soin de l'exécuter incrémentalement, en continu, en mettant à jour le résultat final alors que les données continuent d'arriver.
Vous pouvez utiliser l'API Dataset/DataFrame en Scala, Java, Python ou R pour exprimer des aggrégations en streaming, des *event-time windows*, des jointures *stream-to-batch* etc... Les calculs sont effectués sur le même moteur Spark SQL optimisé. Enfin, le système garantit le *exactly-once* de bout en bout même en cas d'erreur grâce au *checkpointing* et des logs *write-ahead*.

Par défaut, les requêtes *Structured Streaming* sont traitées en interne en utilisant un moteur de traitement ***micro-batch***, qui traites le flux de données comme un série de petits traitements *batchs* atteignant ainsi des latences aussi faibles que 100ms. Cependant depuis Spark 2.3, nous avons introduit un nouveau mode de traitment à faible latence appelé **Continuous Processing** qui peut atteindre des latences jusqu'à 1ms. Sans changer les opérations Dataset/Dataframe dans vos requêtes, vous aurez la possibilité de choisir le mode traitement en fonction des besoins de votre application.

Dans ce guide, nous parcourerons [le modèle de programmation](https://en.wikipedia.org/wiki/Programming_model) et les APIs. Nous expliquerons les concepts principalement à travers le modèle de traitement par défaut qu'est le *micro-batch* puis nous discuterons du modèle de traitement en continu.

Commençons par un simple example de requête *Structured Streaming* - un "word count" en streaming.

## Un exemple rapide

Mettons que vous souhaitiez maintenir le nombre de mots dans des données textuelles reçues depuis un serveur de données écoutant sur un socket TCP.
Voyons voir comment on peut exprimer cela en utilisant Spark Structured Streaming.
tout d'abord nous devons importer les classes nécessaires et créer une `SparkSession` (le point d'accés à toutes fonctionnalités de Spark).

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import explode
from pyspark.sql.functions import split

spark = SparkSession \
    .builder \
    .appName("StructuredNetworkWordCount") \
    .getOrCreate()
```

Ensuite, créeons un DataFrame non-borné représentant les données reçues depuis un serveur écoutant sur `localhost:9999`, et transformons le DataFrame pour calculer le nombre de mots.

```python
# Créer le DataFrame représentent le flux de données en entrée venant de la connexion à localhost:9999
lines = spark \
    .readStream \
    .format("socket")
    .option("host", "localhost") \
    .load()

# Séparer les lignes de texte en mots
words = lines.select(
    explode(
        split(lines.value, " ")
    ).alias("word")

# Générer le nombre de mots actuel
wordCoutns = words.groupBy("word").count()
```

Le DataFrame `lines` représente une table non-bornée contenant le flux de données textuelles. 
Cette table contient un colonnes de *strings* nommée `value`, et chaque ligne est dans le flux de donnée devient une rangée de cette table.

> Notez qu'il ne reçoit pas de données actuellement, nous sommes seulement entrain de mettre en place la transformation et ne l'avons pas encore démarré.

Ensuite nous avons utilisé deux fonctions SQL intégrées - `split` et `explode`, pour séparer chaque ligne en de nouvelles lignes contenant chacunes un seul mot.
On a également utilisé la fonction `alias` pour nommer la nouvelle colonne `word`.

Enfin, nous avons défini le DataFrame `wordCounts` qui représent le nombre actuel de mots dans le stream (*dataset* non-borné).

On a désormais la requête à exécuter sur le flux de données. 
Il ne nous reste plus qu'a commencer à recevoir ces données et calculer le compte. 
Pour cela nous configurons l'impression du compte (spécifié par `outputMode("complete")`) vers la console à chaque fois qu'il est mis à jour et démarrons le traitement avec `start()`.

```python
# Démarrer l'exécution de la requête qui va imprimer le compte actuel dans la console
query = wordCounts \
    .writeStream \
    .outputMode("complete") \
    .format("console") \
    .start()

query.awaitTermination()
```

Après que ce code ait été exécuté, le traitement streaming aura démarré en arrière plan. L'objet `query` est une attache à requête streaming en cours,
et nous avons décidé d'attendre la fin de la requête en utilisant `awaitTermination()` pour éviter de sortir du processus alors que la requête est en cours.


