# Sécuriser l'accés à un cluster Kafka

Sécurisez les connexions en configurant Kafka et des utilisateurs Kafka. À travers la configuration vous pouvez implémenter des mécanismes de chiffrement, d'authentification, et de d'autorisation.

## Configuration de Kafka

Pour établir un accés securisé à Kafka, configurez la ressource `Kafka` avec les configurations suivantes en fonctions de vos besoins:

- Des *listeners* spécifiant un type d'authentification pour définir comment les clients s'authentifient 
- Les autorisations pour l'entièreté du cluster
- Les politiques réseau pour restreindre les accés
- Les super-utilisateurs pour un accés sans contraintes aux brokers

L'authentification est configurée indépendamment pour chaque listener, tandis que l'autorisation est mise en place pour l'ensemble du cluster.

Pour plus d'information sur la configuration des accès de Kafka, consultez la [référence du schema de `Kafka`](https://strimzi.io/docs/operators/latest/configuring#type-Kafka-reference) et la [référence du schema du `GenericKafkaListener`](https://strimzi.io/docs/operators/latest/configuring#type-GenericKafkaListener-reference)