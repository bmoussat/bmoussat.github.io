+++
date = '2020-01-23T08:59:36+01:00'
title = 'Comment Utiliser plusieurs profils d’application avec Spring Boot'
tags = ['java', 'spring boot', 'properties', 'profils', 'configuration', 'mysql']
description = "This is bold text, and this is emphasized text."

+++

Dans ce post, nous allons voir comment utiliser plusieurs profils d’applications dans une même application spring boot.
<!--more-->
## **Pourquoi utiliser des profils ?**

Selon ***larousse.fr***, « un profil est un ensemble de caractéristiques qui définissent fondamentalement un type de chose ; configuration de quelque chose à un moment donné ».

Lorsque l’on crée une application, il faut la développer, la tester, faire la recette de cette application, la mettre en pré-production puis en production.

Pour ces différentes étapes, il faudra plusieurs environnements, donc par conséquent plusieurs configurations, une pour chaque environnement, car les urls des serveurs, les comptes d’accès aux bases de données peuvent être différentes selon que l’on soit en développement ou en production.

Dans ce cadre, les profils nous permettent de faciliter et simplifier le changement de configuration selon l’environnement ou on veut déployer l’application.

## **Création et utilisation des profils**

L’une des manières la plus simple de l’utilisation des profils, est de créer plusieurs fichiers ***properties*** avec les valeurs nécessaires au bon fonctionnement de l’application dans l ‘environnement cible.

Le nom de ces fichiers ***properties***, doit être préfixé du mot « ***application*** » suivi du nom de l’environnement cible, séparé par un tiret et avec l’extension « ***.properties***»

# **Rien de mieux que la pratique**

Pour l’environnement de développement, nous allons créer le fichier ***application-dev.properties*** et pour la production, un fichier au nom de : ***application-prod.properties.***

## **Rajoutons nos nouveaux profils dans notre projet.**

Vous pouvez télécharger les sources de cet article [ici](https://github.com/bmoussat/multi_profile_app>). Ou bien, vous pouvez rajouter les profils à votre application si vous en avez déjà une.

Voici la structure de notre projet après ajout des nouveaux fichiers « ***properties*** » :

![Emplacement des fichiers properties](/images/projectStructure_.png)

## **Maintenant mettons du contenu dans nos fichiers *properties*:**

*- application-dev.properties :*
```properties
1. spring.datasource.url=jdbc:mysql://localhost:3306/dev?serverTimezone=UTC

2. spring.datasource.username= dev_db_user_login

3. spring.datasource.password=dev_db_user_mdp

4. logging.level.org.springframework.web= DEBUG

5. logging.level.com.concretepage= DEBUG

6. spring.jpa.hibernate.ddl-auto=none

7. spring.jpa.show-sql=true
```
*- application-prod.properties :*
```properties
1. spring.datasource.url=jdbc:mysql://localhost:3306/prod?serverTimezone=UTC

2. spring.datasource.username=prod_db_user_login

3. spring.datasource.password=prod_db_user_mdp

4. logging.level.org.springframework.web=info

5. logging.level.com.concretepage=info

6. spring.jpa.hibernate.ddl-auto=none

7. spring.jpa.show-sql=false
```

### **Un peu d’explication**

1 : correspondent aux liens d’accès à la base de données, dont `jdbc:mysql://localhost:3306/` qui correspond à l’accès à la base et `dev/prod` qui correspondent aux tables de la base de données.

2 : correspondent respectivement aux utilisateurs (login) d’accès à la base de données de développement et de production.

3 : correspondent respectivement aux mots de passe des utilisateurs d’accès à la base de données de développement et de production.

Par conséquent, il faut remplacer `dev_db_user_login, prod_db_user_login` par respectivement les noms d’utilisateur de la base de développement et de production puis `dev_db_user_mdp, prod_db_user_mdp` par respectivement les mots de passe de développement et production.

4 et 5 : concernent le niveau de logs souhaité.

Les lignes 6 et 7 : concernent la base de données, plus précisément (juste pour infos car ce n’est pas notre sujet) la ligne 6 permet ou non (selon le mode choisi) de générer la structure des tables à partir des modèles ayant des annotations appropriées contenues dans le projet.

Le fichier ***application.properties*** contiendra la propriété qui nous permettra de préciser quelle configuration choisir, soit :

`spring.profiles.active=dev`

Nous attribuons la valeur « *dev* » lorsque nous voulons choisir la configuration pour l’environnement de développement, et la valeur « *prod* » lorsque l’on veut choisir la configuration pour l’environnement de production.

Grâce à la propriété « ***spring.profiles.active*** », Spring Boot sait quelle configuration charger.

## **Conclusion**

Dans cet article, nous avons vu, comment créer de nouveaux profils dans un même projet Spring Boot afin d’avoir plusieurs configurations pour la même application, ce qui facilite la génération de paquets applicatifs pour plusieurs environnements différents.


