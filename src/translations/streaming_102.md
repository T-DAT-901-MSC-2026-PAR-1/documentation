# Streaming 102 : Le monde au-delà du batch

> This blog post is a translation of the one from [Tyler Akidau](https://www.oreilly.com/people/tyler-akidau/) on the [Oreilly's website](https://www.oreilly.com/radar/the-world-beyond-batch-streaming-102/)

!["Bateliers sur le Missouri" par George Caleb Bingham (source: Wikimedia Commons)](./images/streaming_102/boatmen_missouri.png)

> Note de l'éditeur: Ceci est le second post d'une série en deux parites sur l'évolution du traitement de données avec un focus sur les sytèmes de streaming, le jeux de données non-bornés et le futur du Big Data. [Voir la partie une.](./streaming_101.md) Aussi jettez un oeil à ["Streaming Systems"](https://www.oreilly.com/library/view/streaming-systems/9781491983867/) par Tyler Akdiau, Slava Chernyak et Reuven Lax.

## Introduction

Bon retour ! Si vous avez manqué l'article précédent, ["The world beyond batch: Streaming 101"](./streaming_101.md), je vous recommande vivement de prendre le temps de le lire en premier. Il pose les fondations nécessaires pour les sujets que je vais couvrir dans cet article, et je vais supposer que vous êtes déjà familier avec la terminologie et les notions introduites là-bas.

Notez également que cet article contient un certain nombre d'animations, donc ceux d'entre vous qui essayeront de l'imprimer vont manquer certaines des meilleures parties.

Les avertissments passés, nous pouvons commencer cette partie. Pour récapituler brièvement, la dernière fois, je me suis concentré sur trois domaines principaux : la **terminologie**, la comparaison **batch versus streaming**, et les **patterns de traitement de données**. 

Dans cet article, je veux me concentrer davantage sur les patterns de traitement de données de la dernière fois, mais de manière plus détaillée, et dans le contexte d'exemples concrets. L'arc de cet article traversera deux sections majeures :

- **Streaming 101 Redux** : Une brève promenade à travers les concepts introduits dans Streaming 101, avec l'ajout d'un exemple tout du long pour mettre en avant les points soulevés.
- **Streaming 102** : La pièce complémentaire de Streaming 101, détaillant les concepts additionnels qui sont importants lors du traitement de données non bornées (*unbounded data*), en continuant d'utiliser notre exemple concret comme moyen d'explication.

D'ici la fin de cet article, nous aurons couvert ce que je considère être le coeur des principes et concepts requis pour un traitement robuste de données désordonnées (*out-of-order data processing).* Ce sont les outils pour raisonner sur le temps qui vous permettent vraiment d'aller au-delà du traitement batch classique.

Pour vous donner un sens de ce à quoi ils ressemblent en action, j'utiliserais des extraits de code du [SDK Dataflow](https://github.com/GoogleCloudPlatform/DataflowJavaSDK), couplés ave des animations pour fournir une représenation visuelle des conepcts. La raison pour laquelle j'utilises le SDK Dataflow, et pas quelque chose auquels les gens sont plus familiers comme Spark Streaming ou Storm, est qu'il n'y a littéralement pas d'autre système à ce point qui fournit l'expréssivité nécessaire pour tous les examples que je veux couvrir. La bonne nouvelle est qu'il y'a d'autres projets qui commencent à aller dans cette direction. Une encore plus bonne nouvelle est que nous (Google) avons à ce jour [soumis une proposition](https://cloud.google.com/blog/products/gcp/dataflow-and-open-source-proposal-to-join-the-apache-incubator/?hl=en) à la Apache Software Foundation pour créer un projet d'incubation Apache Dataflow (en conjonction avec data Artisans, Cloudera, Talend, et quelques autres entreprises), dans l'espoir de construire une communauté ouverte et un écosystème autour des des robustes sémantiques pour le traitement désordonné permises par le [Modèle Dataflow](https://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf). Cela devrait rendre 2016 trés intéressant. Mais je digrésses.

## Récapitulatif et feuille de route

Dans Streaming 101, j'ai d'abord clarifié certains termes. J'ai commencé par distinguer les données bornées (bounded) versus non bornées (unbounded). Les sources de données bornées ont une taille finie, et sont souvent appelées données "batch". Les sources non bornées peuvent avoir une taille infinie, et sont souvent appelées données "streaming". J'essaie d'éviter d'utiliser les termes batch et streaming pour désigner les sources de données car ces noms portent avec eux certaines implications qui sont trompeuses et souvent limitantes.

J'ai ensuite défini les différences entre les moteurs batch et streaming : les moteurs batch sont ceux qui sont conçus uniquement avec les données bornées à l'esprit, tandis que les moteurs streaming sont conçus avec les données non bornées à l'esprit.

Après la terminologie, j'ai couvert deux concepts de base importants pertinents pour traiter les données non bornées. J'ai d'abord établi la distinction critique entre **event time** (le moment où les événements se produisent) et **processing time** (le moment où ils sont observés pendant le traitement). 

J'ai ensuite introduit le concept de **windowing** (fenêtrage), qui est une approche courante utilisée pour faire face au fait que les sources de données non bornées peuvent techniquement ne jamais se terminer.

En plus de ces deux concepts, nous allons maintenant examiner de près trois autres :

- **Watermarks** (filigranes) : Une watermark est une notion de complétude de l'entrée par rapport aux event times. Une watermark avec une valeur de temps X fait l'affirmation : "toutes les données d'entrée avec des event times inférieurs à X ont été observées". Les watermarks agissent ainsi comme une métrique de progression lors de l'observation d'une source de données non bornée sans fin connue.

- **Triggers** (déclencheurs) : Un trigger est un mécanisme pour déclarer quand la sortie pour une window doit être matérialisée par rapport à un signal externe. Les triggers offrent une flexibilité dans le choix du moment où les sorties doivent être émises.

- **Accumulation** : Un mode d'accumulation spécifie la relation entre les multiples résultats observés pour la même window. Ces résultats peuvent être complètement disjoints, représentant des deltas indépendants au fil du temps, ou il peut y avoir un chevauchement entre eux.

Pour faciliter la compréhension des relations entre tous ces concepts, nous allons revisiter l'ancien et explorer le nouveau dans la structure de réponse à quatre questions, qui sont toutes critiques pour chaque problème de traitement de données non bornées :

1. **What** : Quels résultats sont calculés ? Cette question trouve sa réponse dans les types de transformations au sein du pipeline.

2. **Where** : Où dans l'event time les résultats sont-ils calculés ? Cette question trouve sa réponse dans l'utilisation du windowing en event-time au sein du pipeline.

3. **When** : Quand dans le processing time les résultats sont-ils matérialisés ? Cette question trouve sa réponse dans l'utilisation des watermarks et des triggers.

4. **How** : Comment les raffinements de résultats sont-ils liés ? Cette question trouve sa réponse dans le type d'accumulation utilisé : discarding (où les résultats sont tous indépendants et distincts), accumulating (où les résultats ultérieurs s'appuient sur les précédents), ou accumulating and retracting (où à la fois la valeur accumulée plus une rétraction pour la/les valeur(s) déclenchée(s) précédemment sont émises).

## Streaming 101 Redux

### What : transformations

Les transformations appliquées dans le traitement batch classique répondent à la question : "Quels résultats sont calculés ?"

Pour cette section, nous allons examiner un seul exemple : calculer des sommes d'entiers avec clé sur un ensemble de données simple composé de 10 valeurs. Si vous voulez une vision plus pragmatique, vous pouvez imaginer calculer un score global pour une équipe d'individus jouant à un jeu mobile en combinant leurs scores indépendants.

Pour chaque exemple, j'inclurai un court extrait de pseudo-code Java du Dataflow SDK pour rendre la définition du pipeline plus concrète. Il y a deux primitives de base dans Dataflow :

- **PCollections** : qui représentent des ensembles de données (possiblement massifs), sur lesquels des transformations parallèles peuvent être effectuées.
- **PTransforms** : qui sont appliquées aux PCollections pour créer de nouvelles PCollections.

Pour un pipeline qui lit simplement les données d'une source I/O, analyse les paires équipe/score, et calcule les sommes par équipe des scores, nous aurions quelque chose comme ceci :

```java
PCollection<String> raw = IO.read(...);
PCollection<KV<String, Integer>> input = raw.apply(ParDo.of(new ParseFn());
PCollection<KV<String, Integer>> scores = input
    .apply(Sum.integersPerKey());
```

Puisque c'est un pipeline batch, il accumule l'état jusqu'à ce qu'il ait vu toutes les entrées, moment auquel il produit sa sortie unique de 51. Dans cet exemple, nous calculons une somme sur tout l'event time puisque nous n'avons appliqué aucune transformation de windowing spécifique.

### Where : windowing

Comme discuté la dernière fois, le windowing est le processus de découpage d'une source de données le long de frontières temporelles. Les stratégies de windowing courantes incluent les **fixed windows** (fenêtres fixes), les **sliding windows** (fenêtres glissantes), et les **sessions windows** (fenêtres de session).

Pour mieux comprendre à quoi ressemble le windowing en pratique, prenons notre pipeline de sommation d'entiers et mettons-le en fenêtres fixes de deux minutes. Avec le Dataflow SDK, le changement est un simple ajout d'une transformation `Window.into` :

```java
PCollection<KV<String, Integer>> scores = input
    .apply(Window.into(FixedWindows.of(Duration.standardMinutes(2))))
    .apply(Sum.integersPerKey());
```

Comme auparavant, les entrées sont accumulées dans l'état jusqu'à ce qu'elles soient entièrement consommées, après quoi la sortie est produite. Dans ce cas, cependant, au lieu d'une sortie, nous obtenons quatre : une sortie unique pour chacune des quatre fenêtres d'event-time de deux minutes pertinentes.

## Streaming 102

Nous venons d'observer l'exécution d'un pipeline avec fenêtres sur un moteur batch. Mais idéalement, nous aimerions avoir une latence plus faible pour nos résultats, et nous aimerions également gérer nativement les sources de données non bornées. Passer à un moteur streaming est un pas dans la bonne direction, mais alors que le moteur batch avait un point connu auquel l'entrée pour chaque fenêtre était complète, nous manquons actuellement d'un moyen pratique de déterminer la complétude avec une source de données non bornée. C'est là qu'interviennent les watermarks.

### When : watermarks

Les watermarks sont la première moitié de la réponse à la question : "Quand dans le processing time les résultats sont-ils matérialisés ?" Les watermarks sont des notions temporelles de complétude de l'entrée dans le domaine de l'event-time.

Conceptuellement, vous pouvez penser à la watermark comme une fonction, F(P) -> E, qui prend un point dans le processing time et retourne un point dans l'event time. Ce point dans l'event time, E, est le point jusqu'auquel le système croit que toutes les entrées avec des event times inférieurs à E ont été observées. En d'autres termes, c'est une assertion qu'aucune autre donnée avec des event times inférieurs à E ne sera jamais vue à nouveau.

Selon le type de watermark, parfaite ou heuristique, cette assertion peut être une garantie stricte ou une supposition éclairée, respectivement :

- **Perfect watermarks** (watermarks parfaites) : Dans le cas où nous avons une connaissance parfaite de toutes les données d'entrée, il est possible de construire une watermark parfaite ; dans un tel cas, il n'y a pas de données tardives ; toutes les données sont précoces ou à temps.

- **Heuristic watermarks** (watermarks heuristiques) : Pour de nombreuses sources d'entrée distribuées, une connaissance parfaite des données d'entrée n'est pas pratique, auquel cas la meilleure option suivante est de fournir une watermark heuristique. Les watermarks heuristiques utilisent toutes les informations disponibles sur les entrées pour fournir une estimation de la progression aussi précise que possible.

Les watermarks peuvent être :

- **Trop lentes** : Lorsqu'une watermark de n'importe quel type est correctement retardée en raison de données non traitées connues, cela se traduit directement par des retards dans la sortie si l'avancement de la watermark est la seule chose sur laquelle vous comptez pour stimuler les résultats.

- **Trop rapides** : Lorsqu'une watermark heuristique est incorrectement avancée plus tôt qu'elle ne devrait l'être, il est possible que des données avec des event times avant la watermark arrivent quelque temps plus tard, créant des données tardives (late data).

### When : triggers

Les triggers sont la seconde moitié de la réponse à la question : "Quand dans le processing time les résultats sont-ils matérialisés ?" Les triggers déclarent quand la sortie pour une window doit se produire dans le processing time. Chaque sortie spécifique pour une window est appelée un **pane** (panneau) de la window.

Exemples de signaux utilisés pour le triggering :

- **Watermark progress** (progression de la watermark) : progrès dans l'event time
- **Processing time progress** (progression du processing time) : utile pour fournir des mises à jour régulières et périodiques
- **Element counts** (comptes d'éléments) : utile pour déclencher après qu'un nombre fini d'éléments a été observé dans une window
- **Punctuations** ou autres triggers dépendants des données

En plus des triggers simples qui se déclenchent en fonction de signaux concrets, il existe également des **composite triggers** (triggers composites) qui permettent la création d'une logique de déclenchement plus sophistiquée :

- **Repetitions** (répétitions)
- **Conjunctions** (conjonctions - AND logique)
- **Disjunctions** (disjonctions - OR logique)
- **Sequences** (séquences)

Voici un exemple de code avec des triggers early et late explicites :

```java
PCollection<KV<String, Integer>> scores = input
    .apply(Window.into(FixedWindows.of(Duration.standardMinutes(2)))
        .triggering(
            AtWatermark()
                .withEarlyFirings(AtPeriod(Duration.standardMinutes(1)))
                .withLateFirings(AtCount(1))))
    .apply(Sum.integersPerKey());
```

Cette version présente deux améliorations claires :

1. Pour le cas "watermarks trop lentes" : nous fournissons maintenant des mises à jour early périodiques une fois par minute.
2. Pour le cas "watermarks heuristiques trop rapides" : lorsque des données tardives arrivent, nous les incorporons immédiatement dans un nouveau pane corrigé.

### When : allowed lateness (garbage collection)

Avant de passer à notre dernière question, j'aimerais aborder une nécessité pratique dans les systèmes de traitement de flux à longue durée de vie : le garbage collection (collecte des déchets). Dans un système réel de traitement désordonné, tout système doit fournir un moyen de limiter les durées de vie des windows qu'il traite. Une façon propre et concise de faire cela est de définir un horizon sur la **allowed lateness** (latence autorisée) dans le système.

Une fois que vous avez limité le retard des données individuelles, vous avez également établi précisément combien de temps l'état pour les windows doit être conservé : jusqu'à ce que la watermark dépasse l'horizon de latence pour la fin de la window.

```java
PCollection<KV<String, Integer>> scores = input
    .apply(Window.into(FixedWindows.of(Duration.standardMinutes(2)))
        .triggering(
            AtWatermark()
                .withEarlyFirings(AtPeriod(Duration.standardMinutes(1)))
                .withLateFirings(AtCount(1)))
        .withAllowedLateness(Duration.standardMinutes(1)))
    .apply(Sum.integersPerKey());
```

### How : accumulation

Lorsque les triggers sont utilisés pour produire plusieurs panes pour une seule window au fil du temps, nous nous retrouvons confrontés à la dernière question : "Comment les raffinements de résultats sont-ils liés ?" Il existe en fait trois modes d'accumulation différents :

- **Discarding** (rejet) : Chaque fois qu'un pane est matérialisé, tout état stocké est rejeté. Cela signifie que chaque pane successif est indépendant de ceux qui l'ont précédé. Le mode discarding est utile lorsque le consommateur en aval effectue lui-même une sorte d'accumulation.

- **Accumulating** (accumulation) : Chaque fois qu'un pane est matérialisé, tout état stocké est conservé, et les entrées futures sont accumulées dans l'état existant. Cela signifie que chaque pane successif s'appuie sur les panes précédents. Le mode accumulating est utile lorsque les résultats ultérieurs peuvent simplement écraser les résultats précédents.

- **Accumulating & retracting** (accumulation et rétraction) : Comme le mode accumulating, mais lors de la production d'un nouveau pane, produit également des rétractions indépendantes pour le(s) pane(s) précédent(s). Les rétractions sont particulièrement utiles lorsque les consommateurs en aval regroupent les données selon une dimension différente, ou lorsque des dynamic windows (fenêtres dynamiques) comme les sessions sont utilisées.

Exemple de code avec mode discarding :

```java
PCollection<KV<String, Integer>> scores = input
    .apply(Window.into(FixedWindows.of(Duration.standardMinutes(2)))
        .triggering(
            AtWatermark()
                .withEarlyFirings(AtPeriod(Duration.standardMinutes(1)))
                .withLateFirings(AtCount(1)))
        .discardingFiredPanes())
    .apply(Sum.integersPerKey());
```

Exemple de code avec mode accumulating & retracting :

```java
PCollection<KV<String, Integer>> scores = input
    .apply(Window.into(FixedWindows.of(Duration.standardMinutes(2)))
        .triggering(
            AtWatermark()
                .withEarlyFirings(AtPeriod(Duration.standardMinutes(1)))
                .withLateFirings(AtCount(1)))
        .accumulatingAndRetractingFiredPanes())
    .apply(Sum.integersPerKey());
```

Comme vous pouvez l'imaginer, les modes dans l'ordre présenté (discarding, accumulating, accumulating & retracting) sont chacun successivement plus coûteux en termes de coûts de stockage et de calcul. À cette fin, le choix du mode d'accumulation fournit encore une autre dimension pour faire des compromis le long des axes de correction, latence et coût.

## When/Where : Processing-time windows

Le windowing en processing-time est important pour deux raisons :

1. Pour certains cas d'usage, comme la surveillance d'utilisation (par exemple, le QPS du trafic de service Web), où vous voulez analyser un flux de données entrant tel qu'il est observé, le windowing en processing-time est absolument l'approche appropriée à prendre.

2. Pour les cas d'usage où le moment où les événements se sont produits est important, le windowing en processing-time est absolument la mauvaise approche à prendre, et être capable de reconnaître ces cas est critique.

Il existe deux méthodes pour obtenir le windowing en processing-time :

- **Triggers** : Ignorer l'event time (utiliser une window globale couvrant tout l'event time) et utiliser des triggers pour fournir des instantanés de cette window dans l'axe processing-time.

- **Ingress time** : Assigner les temps d'entrée comme event times pour les données lorsqu'elles arrivent, et utiliser le windowing en event time normal à partir de là.

Le grand inconvénient du windowing en processing-time est que le contenu des windows change lorsque l'ordre d'observation des entrées change. Si vous vous souciez des moments auxquels vos événements se sont réellement produits, vous devez utiliser le windowing en event-time ou vos résultats seront dénués de sens.

## Where : session windows

Les sessions sont un type spécial de window qui capture une période d'activité dans les données qui est terminée par un écart d'inactivité. Elles sont particulièrement utiles dans l'analyse de données car elles peuvent fournir une vue des activités pour un utilisateur spécifique sur une période spécifique où il était engagé dans une activité.

Du point de vue du windowing, les sessions sont particulièrement intéressantes de deux manières :

1. Ce sont un exemple de **data-driven window** (fenêtre pilotée par les données) : l'emplacement et les tailles des windows sont une conséquence directe des données d'entrée elles-mêmes.

2. Ce sont également un exemple d'**unaligned window** (fenêtre non alignée), c'est-à-dire une window qui ne s'applique pas uniformément à travers les données, mais seulement à un sous-ensemble spécifique des données (par exemple, par utilisateur).

L'insight clé pour fournir un support général des sessions est qu'une session window complète est, par définition, une composition d'un ensemble de windows plus petites et se chevauchant, chacune contenant un seul enregistrement, avec chaque enregistrement dans la séquence séparé du suivant par un écart d'inactivité ne dépassant pas un timeout prédéfini.

```java
PCollection<KV<String, Integer>> scores = input
    .apply(Window.into(Sessions.withGapDuration(Duration.standardMinutes(1)))
        .triggering(
            AtWatermark()
                .withEarlyFirings(AtPeriod(Duration.standardMinutes(1)))
                .withLateFirings(AtCount(1)))
        .accumulatingAndRetractingFiredPanes())
    .apply(Sum.integersPerKey());
```

C'est du matériel assez puissant. Et ce qui est vraiment génial, c'est la facilité avec laquelle il est possible de décrire quelque chose comme ça dans un modèle qui décompose les dimensions du traitement de flux en pièces distinctes et composables.

## Conclusion

Nous avons maintenant couvert les concepts majeurs :

- **Event-time versus processing-time** : La distinction cruciale entre le moment où les événements se sont produits et le moment où ils sont observés par votre système de traitement de données.

- **Windowing** : L'approche couramment utilisée pour gérer les données non bornées en les découpant le long de frontières temporelles.

- **Watermarks** : La notion puissante de progression dans l'event-time qui fournit un moyen de raisonner sur la complétude dans un système de traitement désordonné opérant sur des données non bornées.

- **Triggers** : Le mécanisme déclaratif pour spécifier précisément quand la matérialisation de la sortie a du sens pour votre cas d'usage particulier.

- **Accumulation** : La relation entre les raffinements de résultats pour une seule window dans les cas où elle est matérialisée plusieurs fois au fur et à mesure qu'elle évolue.

Et les quatre questions utilisées pour cadrer notre exploration :

1. **What** : Quels résultats sont calculés ? = transformations
2. **Where** : Où dans l'event-time les résultats sont-ils calculés ? = windowing
3. **When** : Quand dans le processing-time les résultats sont-ils matérialisés ? = watermarks + triggers
4. **How** : Comment les raffinements de résultats sont-ils liés ? = accumulation

Cette approche offre la flexibilité nécessaire pour équilibrer les tensions concurrentes comme la correction, la latence et le coût dans le traitement de flux moderne.