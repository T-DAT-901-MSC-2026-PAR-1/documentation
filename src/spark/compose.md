# Déploiement d'un cluster Apache Spark avec Docker Compose 

Ressources:

- [Setting a Spark standalone cluster on Docker in layman terms](https://medium.com/@MarinAgli1/setting-up-a-spark-standalone-cluster-on-docker-in-layman-terms-8cbdc9fdd14b)
- [spark-standalone-cluster - github.com](https://github.com/mrn-aglic/spark-standalone-cluster)
- [DeadSimple: PySpark + Docker Spark Cluster on your Laptop](https://medium.com/programmers-journey/deadsimple-pyspark-docker-spark-cluster-on-your-laptop-9f12e915ecf4)
- [apache/spark - hub.docker.com](https://hub.docker.com/r/apache/spark)
- [Spark Standalone Mode](https://spark.apache.org/docs/latest/spark-standalone.html)

## Le mode standalone de Spark

En plus de s'exécuter sur le gestionnaire de cluster [YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html), Spark fournit un moyen de déploiement simple en mode "autonome" (standalone).

Vous pouvez lancer un cluster autonome manuellement en démarrant les *masters* et *workers* à la main ou en utilisant les [scripts de lancement](https://spark.apache.org/docs/latest/spark-standalone.html#cluster-launch-scripts).

Il est possible d'exécuter ces [*daemons*](https://en.wikipedia.org/wiki/Daemon_(computing)) sur une seule machine pour tester.

### Sécurité

Les fonctionnalités de sécurité comme l'authentification ne sont pas activées par défaut. Quand vous déployez un cluster qui sera accessible depuis l'internet ou un réseau non-fiable, il est important de sécuriser l'accès au cluster pour empécher des applications non autorisées de s'exécuter sur le cluster.

Veuillez consulter [Spark Security](https://spark.apache.org/docs/latest/security.html) et les sections spécifiques à la sécurité dans ce document avant d'exécuter Spark.

### Installer Spark Standalone sur un cluster

Pour installer Spark en mode autonome, placez simplement un version compilée de Spark sur chaque noeud du cluster. Vous pouvez obtenir une version pré-build de Spark pour chaque release ou [faire le build vous même](https://spark.apache.org/docs/latest/building-spark.html).

### Démarrer le cluster manuellement

Vous pouvez lancer un server *master* autonome en exécutant:

```
./sbin/start-master.sh
```

Une fois démarré, le *master* afficher une URL `spark://HOST:PORT` pour lui même, qui peut être utilisée pour y connecter des *workers* ou être passée comme argument "`master`" à un objet `SparkContext`.


