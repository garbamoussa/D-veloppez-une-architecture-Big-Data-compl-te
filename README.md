# Développez-une-architecture-Big-Data-compl-te
Développez une architecture Big Data complète
Au bon vieux temps on utilisait des lettres et des timbres. On écrivait avec beaucoup d’application des missives qui devaient traverser le temps et l’espace afin de joindre leur destinataire. A cette époque, on avait IRC et ICQ (mais ça c’était il y a vraiment très longtemps). Aujourd’hui ça se passe différemment : on envoie des phrases de 140 caractères à des milliers de personnes d’un coup et on fait ça sur Twitter. Alors d’accord, ça n’a peut-être pas la poésie des vieux jours, mais tout est public et il y a une API !
Dans ce projet vous allez sonder les pensées les plus secrètes de l'humanité… enfin, celles qui sont sur Twitter. Vous allez recréer le tableau de bord des tendances les plus virales. Par exemple, le 6 janvier 2017 nous fêtions les rois et les 185 ans de Gustave Doré et les sujets les plus en vogue étaient les suivants :

Votre mission

Vous êtes un data architect chez Twitter et votre rôle est de déployer une solution complète d’analyse de données pour créer un top 10 des sujets les plus tendance. Vous allez devoir mettre en place :
la collecte des données,
leur stockage dans des structures adaptées,
leur traitement au coup par coup et en temps réel,
des solutions pour améliorer la robustesse de l’architecture globale et de chacun de ses composants.
Il s’agit donc d’un projet au croisement de toutes les connaissances et compétences que vous avez pu acquérir jusque là dans ce parcours. A vous de réaliser les bons choix techniques qui vous permettront de créer une architecture robuste et capable de passer à l’échelle.
Vous devrez créer un outil qui permet de lister les dix hashtags les plus fréquents pour chaque heure. Vous devez par exemple pouvoir répondre aux questions suivantes :
Quels sont les mots-clés les plus tendance depuis une heure ?
Quels étaient les mots-clés les plus tendance lundi dernier entre 7h et 8h ? (on ne vous demandera pas une précision par minute)
J’ai fait une erreur dans mon algorithme d’identification des hashtags ; peut-on recalculer les hashtags les plus fréquents utilisés depuis lundi dernier à 7h ?
Pour ce faire, vous devrez mettre en place à la fois un outil de calcul de statistiques en batch, à partir de données stockées sur HDFS, et en temps réel, à l’aide d’un pipeline de traitement de données.

La collecte des informations se fera à partir de l’API de streaming de Twitter. Les statistiques que vous récolterez ne concernent que les “hashtags” présents dans les tweets. A titre d’exemple, voici un script écrit en Python permettant de collecter les hashtags présents dans tous les tweets en anglais :

https://developer.twitter.com/en/docs/tutorials/consuming-streaming-data

#! /usr/bin/env python
import twitter # pip install twitter

def main():
    # Put here your Twitter API credentials obtained at https://apps.twitter.com/
    # Note: you need a Twitter account to create an app.
    oauth = twitter.OAuth("token", "token_secret", "consumer_key", "consumer_secret")
    t = twitter.TwitterStream(auth=oauth)

    sample_tweets_in_english = t.statuses.sample(language="en")
    for tweet in sample_tweets_in_english:
       if "delete" in tweet:
           # Deleted tweet events do not have any associated text
           continue

       print("===================================")

       # Tweet text
       print(tweet["text"])

       # Collect hashtags
       hashtags = [h['text'] for h in tweet["entities"]["hashtags"]]
       if len(hashtags) > 0:
           print(hashtags)

if __name__ == "__main__":
    main()
    
    
 Notez que vous devrez créer une application Twitter pour accéder à l’API, et que vous devrez donc avoir un compte Twitter.
Vous disposez de beaucoup de liberté pour réaliser les choix techniques de ce projet. Les instructions suivantes ne sont donc données qu’à titre indicatif, et vous pouvez tout à fait faire des choix différents, du moment que vous êtes en mesure de les justifier.
Les données temps-réel seront transmises à un cluster Kafka pour à la fois être stockées dans un système de fichiers distribués (HDFS) et transférées à un pipeline Storm.
Batch layer : Les données présentes dans HDFS seront analysées périodiquement et par lot (“batch”) par des jobs MapReduce (Hadoop ou Spark).
View layer : Le résultat de ces jobs MapReduce sera stocké dans MongoDb.
Speed layer : Les données traitées par le pipeline Storm/Kafka seront stockées dans MongoDb. Les données rendues inutiles par l’exécution des tâches de la batch layer devront être effacées au moment opportun.
Un simple script en ligne de commande suffira pour visualiser les tendances Twitter à une heure donnée.
Notez qu'on ne s'intéresse dans ce projet qu'aux hashtags utilisés dans les tweets. En bonus, vous pouvez décider d’inclure les statistiques relatives aux bi-grammes ou aux tri-grammes des tweets, mais ils s’agit là d’un sujet beaucoup plus exigeant…


Livrables
Visualisation des hashtags Twitter les plus populaires des dernières 24h, heure par heure
Schéma d’architecture des différents composants et leurs interactions


GUIDE MENTOR

Ce projet va réunir d’un seul tenant :
Les concepts de création de datalake et de traitement de données batch du projet 4
Et un cluster temps réel Storm identique à celui du projet 3
Dans le but d’archiver les conversations Twitter et de permettre d’en dégager les tendances sur une période choisie ou bien en temps réel.
Rappel :
Twitter a changé sa politique de création de compte développeur nécessaire à l’exploitation de l’API (probablement pour répondre aux exigences de la GDPR).
La création du compte nécessite de motiver le besoin de création et quelques validations de la part de Twitter. Il faut donc anticiper ce besoin car cela peut être long.
Comme pour le projet 3, un peu d’organisation pour commencer.
La donnée étant précieuse, nous allons faire comme dans la vrai vie, la sauvegarder à tout prix, et en commençant par cela.
L'étudiant va commencer par capter la donnée, et la router vers un datalake. Cette étape clé réalisée, nous auront tout le temps (et de la donnée) pour concevoir le reste du pipeline.

Acquisition :
La première tâche de l’étudiant va donc être de monter un cluster kafka (cf. projet 3) et d’écrire un producer utilisant l’API Twitter mentionnée plus haut pour capter les tweets.
Seules les informations de timestamp et le hashtag sont utiles au projet.
Quelques astuces :
Créer un message distinct pour chaque hashtag rencontré pour pouvoir les router facilement dans Storm ;
Il est possible de créer un topic kafka pour la batch layer et un autre pour la speed layer.  Mais cela double les volumes stockés dans Kafka. Il est préférable de gérer plusieurs groupes de consumers sur le même topic, chaque groupe gérant son propre offset ;
Réfléchir très tôt à la manière de grouper les hashtag par heure. Une solution peut être de créer un champ timestamp “réduit” à l”heure zéro minute zéro seconde dans le message envoyé vers Kafka.
Le stockage des données sur HDFS, tout comme le projet 4, devra se faire au format Avro, l’étudiant peut suivre la même logique.
Le programme d’interrogation ayant pour objectif de cibler des tranches horaires, il peut être intéressant de stocker les fichiers Avro dans des répertoires timestampés.
Si l’étudiant est un peu en avance sur son planning, vous pouvez lui proposer de mettre en place un serveur Kafka connect et un plugin pour dumper le contenu des topics sur HDFS, en adoptant un partitionnement horaire.
Quelques exemples simples à mettre en oeuvre sont disponibles sur le net, comme ici.
S’il utilise Kafka seul, il devra écrire cette fonctionnalité par lui même dans un consumer, et formater les fichiers avro comme dans le projet 4.
 
Batch layer :
Une fois Kafka en route et le vidage sur HDFS en fonction, l’étudiant va pouvoir attaquer la Batch-Layer.
Un premier job servira de mise en bouche, en injectant l’historique complet de tous les tweets stockés sur HDFS vers mongoDB. Il ne sera à éxécuter qu’une seule fois (ou bien si on RAZ la base mongoDB).
L’opération peut très facilement se paralléliser en utilisant une fonction map sur la liste des fichiers présents sur le datalake.

Exemple :
RDD_TWEETS = sc.binaryFiles("hdfs://...*.avro")
RDD_TWEETS_FLAT = RDD_TWEETS.flatMap(lambda args: fastavro.reader(BytesIO(args[1]), reader_schema=votre_schema))
L’idée est de profiter de la puissance de Spark pour précalculer les tendances phares de chaque heure, et ne stocker que les 10 meilleurs dans MongoDB.
Les habitués du SQL crieront “OVER PARTITION BY” et ils auront raison, calculer un ranking sur un regroupement de données passant par cette syntaxe peu connue mais très puissante. On l’utilise aussi pour calculer le dernier état d’un enregistrement parmi des dizaines de mises à jour dans les scénarios de réconciliation de base de données.

Dans le cas présent, Spark calculera ce ranking d’une façon qui pourrait ressembler à cela :
DF_TENDANCES = DF_TWEETS.groupBy("heure", "hashtag").count().sort(desc("heure"), desc("count"))

window = Window.partitionBy(DF_TENDANCES['heure']).orderBy(DF_TENDANCES['count'].desc())

·······DF_BESTOF = DF_TENDANCES.select('*', rank().over(window).alias('rank')).filter(col('rank') <=10
 
Il suffit ensuite d’insérer dans MongoDB les 10 hashtags remontés par la requête spark avec le nombre d’occurence de ce hashtag dans l’heure concernée.
Le second Job constituera le coeur de la Batch Layer, puisqu’il sera programmé pour s’exécuter toutes les heures (ou bien tournera en permanence mais avec un timer interne.
Chaque fin d’heure il opérera le même ranking que le programme d’historique, mais sur les fichiers avro stockés sur le datalake pendant la dernière heure uniquement (pour des raisons de performance).
L’étudiant pourra s’aider du partitionnement par timestamping proposé dans le paragraphe sur Kafka pour isoler les fichiers avro de la dernière heure et limiter le volume traité pour ce job schédulé.
 
Speed layer :
Le rôle de la speed-layer va être de suppléer à la Batch Layer entre deux traitements horaires.
Comme la speed-layer n’est pas en mesure de déterminer les meilleures tendances, elle va se contenter d’injecter dans mongoDB chaque hashtag rencontré en temps réel, et d’en incrémenter le comptage.
Pour cela l’étudiant devra recréer le cluster Storm utilisé dans le projet 3, et venir consommer le topic Kafka.
Attention : Comme évoqué en préambule, un seul topic est utilisé pour la batch-layer et pour la speed-layer, il faut veiller à mettre les deux consumers dans des groupes séparés, pour qu’un offset différent soit géré pour chacun.
Comme dans le projet 3 également, afin de pré calculer les tendances de chaque heure pour un hashtag donné, il faudra programmer le pipeline Storm pour opérer un Field grouping sur l’heure du tweet.
Chaque fin d’heure, le Bolt d’un hashtag se réinitialisera et commencera à insérer le hashtag pour une nouvelle tranche horaire
Attention !
L’étudiant devra mettre en place un moyen de différencier les informations injectées dans MongoDB par sa Batch-layer et sa Speed-layer, car chaque heure, la batch-layer va injecter des données qui ont été déjà injectées par la speed-layer pendant l’heure écoulée.
Un champ supplémentaire dans  l’enregistrement du tweet, indiquant son origine, fera l’affaire, et servira au job schédulé de la batch-layer à supprimer les enregistrements en double à chaque passage.
Restitution (view-layer) :
L’étudiant a plusieurs choix possibles pour interroger sa base mongoDB.
Il peut utiliser des requêtes mongo directement dans l’interface de la base de données, ou écrire un programme python (ou autre) d’interrogation.
La syntaxe d’extraction pourrait ressembler à celle-ci :
collection.aggregate([{'$match' : {"heure":{"$gte":date_debut,"$lte":date_fin}}}, {'$project' : {"heure":1, "_id":0, "hashtag":1, "count":1}}, {'$group' : {"_id" : "$hashtag", "total" : {'$sum' : "$count"}}}, {'$sort' : {"total":-1}}, {'$limit':10} ])
Résilience :
Si le projet ne présente pas de difficulté particulière, car reprenant en grande partie des éléments déjà maîtrisés dans les projets précédents, l’étudiant devra tout de même accorder un soin tout particulier à la résilience de son architecture.
La création d’un job historique complet dans la section batch-layer, permettra par exemple de réinitialiser complètement la base mongoDB en cas d’erreur, et grâce à tous les tweets sauvegardé sur le datalake et constituant son Master Dataset.
Les mécanismes de redondance proposés par Kafka (plusieurs brokers), par Storm (reprise des spouts, ack/nack), de mongoDB (replicaSets) devront être expliqués en théorie au moins.
Ainsi que la capacité à supporter une montée en charge (multiplication des spouts, augmentation du nombre de workers spark si la durée du job horaire commence à atteindre une durée limite) etc.
 
