# Security

## References
- [Kafka Security - kafka.apache.org](https://kafka.apache.org/documentation/#security)
- [Simple Authentication and Security Layer (SASL) - fr.wikipedia.org ](https://fr.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer)
- [Salted Challenge Response Authentication Mechanism (SCRAM) - ietf.org](https://datatracker.ietf.org/doc/html/rfc5802)

## Vue d'ensemble

Les mesures de sécurité suivantes sont actuellement supportées:

**1. Authentification**

Kafka proposes actuellement l'authentification pour la connection inter-broker ou broker/client. Il est possible d'utiliser soit SSL soit SASL.

Les mécanismes SASL suivants sont disponibles:
- SASL/GSSAPI (Kerberos)
- SASL/PLAIN
- SASL/SCRAM-SHA-256 et SASL/SCRAM-SHA-512
- SASL/OAUTHBEARER

**2. Chiffrement des données en transit**

Les données en transit inter-broker ou client/broker peuvent être chiffrées avec SSL.

Le chiffrement des données dégrades légérement les performances quand il est activé.

**3. Gestion des authorisations pour les lectures/écritures par les clients**

**4.  Branchement a des services d'authorisation externes**

## Configuration des *Listeners*

Pour sécuriser les communications avec le cluster, il est nécessaire de sécuriser les canaux de communications qu'il utilises.

Chaque serveur doit définir les un ensemble de *listeners* (écouteurs) utilisés pour recevoir les requêtes des clients et des autres serveurs.

Chaque *listener* peut être configuré en utilisant divers mécanismes d'authentification de chiffrement de données.

Les serveurs Kafka peuvent écouter des connections sur plusieurs ports différents.
Ceci est configuré à travers la propriété `listeners` du serveur. 

> Dans le cas du conteneur docker kafka fourni par Apache on définit les propriétés avec des variables d'environnement ou l'on prefixe les noms des propriétés par `KAFKA` et on passe la propriété en majuscule en remplacant les points par des underscores. e.g: `KAFKA_LISTENERS`

Le format pour définir un *listener* est le suivant:

```
{LISTENER_NAME}://{hostname}:{port}
```

Le `LISTENER_NAME` est habituellement un nom descriptif qui définit l'usage du *listener*

*e.g:* De nombreuses configurations utilisent un *listener* séparé pour les clients, dans ce cas là ils pourraient faire référence à ce *listener* en le nommant `CLIENT` dans la configuration.

Le protocole de sécurité de chaque *listener* est ensuite défini dans la propriété `listener.security.protocol.map`. Sa valeur est une liste séparée par des virgules des noms des listeners mappés à leur protocole.

```
listener.security.protocol.map=CLIENT:SSL,BROKER:PLAINTEXT
```

Les options possible pour le protocole de sécurité sont:
- PLAINTEXT (ne fournit pas de sécurité et ne requiert aucune configuration additionnelle)
- SSL
- SASL\_PLAINTEXT
- SASL\_SSL

Le nom de l'option est insensible à la casse.

> Si chaque *listener* utilises un protocole différent il est possible d'utiliser le nom du protocole comme nom du *listener*.
>
> Il est cependant recommandé d'utiliser un nom explicite pour les *listeners*
>
> *e.g*: `listeners=SSL://localhost:9092,PLAINTEXT://localhost:9093`

### Désigner des *listeners* pour la communication inter-broker

La propriété `inter.broker.listener.name` fournit la liste des noms de *listeners* à utiliser. Leur objectif premier est la réplication des partitions.

Si cette propriété n'est pas définie le *listener* choisi est celui utilisant le protocole indiqué dans `security.inter.broker.protocol` (`PLAINTEXT` par défaut).

### Spécificités de KRaft

Dans un cluster KRaft un *broker* est n'importe quel server qui a le rôle `broker` activé dans `process.roles`. Un controller est n'importe quel serveur qui a le rôle `controller` activé.

La configuration des *listeners* dépends du rôle du serveur.

Les *listeners* définis dans `inter.broker.listener.name` sont utilisés exclusivement pour la communication inter-broker.

Les *controllers* doivent utiliser des *listeners* séparés définis par la propriété `controller.listener.names`. Ils ne peuvent pas avoir la même valeur que ceux utilisés pour la communication des brokers.

Les *controllers* recoivent des requêtes à la fois des autres *controllers* et des *brokers*. Pour cette raison même si un serveur n'a pas le rôle *controller* il doit tout même définir `controller.listener.names` dans ses propriétés de sécurité.

Il est requis que l'hôte et le port défini dans `controller.quorum.bootstrap.servers` soit routé vers le *controller listener* exposé par le seveur.

```
process.roles=broker,controller
controller.quorum.bootstrap.servers=localhost:9093
listeners=BROKER://localhost:9092,CONTROLLER://localhost:9093
inter.broker.listener.name=BROKER
controller.listener.names=CONTROLLER
listener.security.protocol.map=BROKER:SASL_SSL,CONTROLLER:SASL_SSL
```

*e.g:* Le *listener* `CONTROLLER` est lié à `localhost:9093`

**Controller listeners multiples**

Les *controllers* acceptent les requêtes provenant de tous les *listeners* définis dans `controller.listener.names`.
Typiquement il y'aura seulement un *controller listener*. Mais il est possible d'en avoir plus, par example pour avoir un moyen de changer le *listener* actif vers un port ou protocole différent à travers un roulement sur le serveur. (un roulement pour créer le nouveau *listener* et un roulement pour supprimer l'ancien).
Quand plusieurs *controller listeners* sont actifs le premier de la liste sera utilisé pour les requêtes sortantes.

**Isolation réseau des _listeners_ clients et inter-brokers**

C'est une convention d'utiliser des *listeners* séparés pour les clients. Cela permet aux *listeners* inter-cluster d'être isolés au niveau réseau.

Dans le cas des *controller listeners* de KRaft le *listener* devrait être isolé puisqu'il ne fonctionnera pas avec les clients dans tous les cas.

Il est attendu que les clients se connectent à n'importe lequel des *listeners* configurés sur un *broker*.

Toute requête liée au *controllers* sera transmise comme décrit [ici](https://kafka.apache.org/documentation/#kraft_principal_forwarding)

## Authentification avec SASL

### 1. Configuration JAAS

Kafka utilises le Java Authentication and Authorization Service ([JAAS](https://docs.oracle.com/en/java/javase/25/security/java-authentication-and-authorization-service-jaas1.html)) pour la configuration SASL.

#### 1.1 Configuration JAAS pour les *brokers* Kafka

Les serveurs/brokers Kafka utilisent la section `KafkaServer` du fichier JAAS.

Elle fournit les options de configuration pour les *brokers* et inclut toute connection SASL utilisée pour la communication inter-broker.

Si plusieurs *listeners* utilisent SASL le nom de la section doit etre préfixé avec le nom du *listener* en minuscules suivi d'un point.

*e.g:* `my_listener.KafkaServer`.

Les *brokers* peuvent aussi configurer JAAS en utilisant le propriété `sasl.jaas.config`. Le nom de la propriété doit être préfixé avec le préfixe du *listener* et inclure le mécanisme SASL à utiliser.

*e.g:* `listener.name.{listenerName].{saslMechanism].sasl.jaas.config`

Un seul module de login peut être spécifié par clé de configuration. Si plusieurs mécanismes sont utilisés pour le même *listener* il faudra utiliser plusieurs clés de configuration en changeant le préfixe du mécanisme.

```
listener.name.sasl_ssl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="admin" \
    password="admin-secret";
listener.name.sasl_ssl.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="admin" \
    password="admin-secret" \
    user_admin="admin-secret" \
    user_alice="alice-secret";
```
