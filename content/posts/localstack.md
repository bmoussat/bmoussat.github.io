+++
date = '2019-10-04T01:54:26+01:00'
title = 'Utilisation de localStack S3 avec une application Spring Boot'
tags = ['java', 'Spring Boot', 'aws', 'localstack', 's3', 'docker', 'thymeleaf']
description = "This is bold text, and this is emphasized text."

+++

Dans cet article, nous allons vous montrer comment tester votre application Spring Boot qui utilise
aws s3 en local avec localstack.
<!--more-->
Dans un premier temps, nous allons créer un bucket s3, puis utiliser ce bucket pour stocker des
fichiers, en simulant aws en local avec localstack.

Pour cela, nous devrions :
- lancer localstack dans un conteneur docker
- créer notre bucket
- lancer notre application, pour lire le bucket s3
local.

Nous aurons besoin des outils suivants :

| coté back-end | coté Front-end |
|---------------|----------------|
| AWS-cli       | thymeleaf      |
| Docker        | bootstrap      |
| Localstack    |                |
| Spring Boot   |                |
| Lombok        |                |


Dans cet article, nous avons utilisé le système d’exploitation Ubuntu.

Vous pouvez trouver les sources de ce projet ici :
https://github.com/bmoussat/s3demo

## Pourquoi Localstack ?

Localstack est un outil qui nous permet de simuler la plupart des services aws en local, donc pas
besoin de se connecter sur la console aws, ce qui nous permet de :

- travailler hors connexion
- tester nos applications sans coût,
- s’isoler par rapport aux autres développeurs concernant les tests et l’utilisation des ressources,
chacun a ses propres services en local,
- tout effacer et tout refaire sans difficulté si rien ne vous convient sans coût supplémentaire :-).

## 1- Installation initiales

### 1.1-  aws-cli, installation et configuration
aws-cli ou _Amazon Webservices commande line interface_, permet d’utiliser les services aws en
ligne de commande.

Pour l’installation d'aws-cli, il faut entrer la commande suivante :
```bash
      sudo snap install aws-cli --classic
```
Pour tester si la aws command line interface est bien installée, taper `aws --version ` dans votre
console, normalement, vous verrez la version d'aws-cli que vous avez installé s’afficher.

### 1.2- configuration de aws-cli

LocalStack comme AWS a besoin des clés d’accès pour fonctionner, même s’il ne vérifie pas si ces
clés sont bonnes ou pas, c'est-à-dire que nous pouvons mettre de faux identifiants pour nos tests.

Pour configurer aws-cli, entrez la commande suivante dans votre console `aws configure`, puis
entrez les informations demandées.
<br/><br/>
![aws configure](/images/localstack/aws-configure.png)

> Pour information `Aws Access key ID`
> et `Aws secret Access key` sont les identifiants que vous récupérez lorsque vous créez un compte
> utilisateur appelé ‘IAM user’ dans le jargon aws.

Pour plus d’information sur les IAM user voici un lien qui serait utile.

C’est tout pour l’installation et la configuration de AWS-CLI.

### 1.3- Docker, Installation

Docker est un conteneur qui permet d’exécuter vos applications correctement et avec le même
comportement sur n’importe quel environnement, il permet aussi de déployer et de tester nos
applications très rapidement.

#### Pourquoi utiliser Docker?

LocalStack peut être lancé dans un conteneur docker ou directement sur la machine courante.
Nous avons choisi de le lancer avec docker pour découvrir cette techno, donc pour voir un cas
pratique d’utilisation de ce conteneur, car il est très utile et populaire.

**Quelques avantages de docker :**
 - pas besoin d’installer localstack sur la machine directement,
 - simplifie la configuration
 - isole les applications,
 - reploiement rapide et simple des applications

Pour installer Docker, entrez la commande suivante
```bash
snap install docker 
```
ou
```bash
sudo apt-get install docker.io
```
C’est tout pour l’installation de docker, nous verrons par la suite comment le lancer pour localstack.

Prochaine étape, installation et utilisation et configuration de localstack

### 1.4- LocalStack, Installation et configuration

> Nous avons besoin de python pip pour l’installation de localstack, nous vous laissons le soin de faire l'installation selon la version de votre système.

Pour installer localstack entré la commande suivante dans votre terminal :
```bash
pip install localstack
```
Pour tester si localstack se lance bien, entrer la commande :
```bash
 localstack start --docker
```
Cette commande nous permet de lancer localstack dans un conteneur docker.
## 2- Création des scripts de lancement

> Rapel : Notre objectif est de créer un bucket s3, puis utiliser ce bucket pour
stocker des fichiers, en simulant aws en local utilisant localstack.

Le script que nous allons créer, nous permettra d’automatiser le lancement de
localstack et la création de notre _bucket s3_.

Pour cela, nous allons utiliser `docker-compose.yml`, qui est un fichier que docker
recherche dans le répertoire courant lorsque l’on utilise la commande `docker-
compose`. Nous verrons un peu plus loin à quoi sert cette commande.

Ce fichier nous permet de donner des instructions de configuration docker pour
le lancement d’image docker, ici l’image que nous voulons lancer est celle de
Localstack.

Voici à quoi ressemble notre fichier `docker-compose.yml`
```bash {linenos=inline}
version: '3.7'

services:
  localstack:
    image: localstack/localstack:latest
    ports:
      - "4567-4585:4567-4585"
    environment:
      - DEFAULT_REGION=eu-central-1
      - SERVICES=s3:4572
    network_mode: "host"
    environment:
      - AWS_REGION=eu-central-1
    volumes:
      - './.localstack:/tmp/localstack'
      -
'/var/run/docker.sock:/var/run/docker.sock'
``` 
**Explication :**

La ligne 5, 
nous permet d’utiliser la dernière version de l'image localstack venant de DockerHub.

La ligne 6,7,
`Ports :
"4567-4585:4567-4585"` correspondes aux ports exposés et la
correspondance des ports côté docker et côté machine courante.

La ligne 10, `SERVICES=s3:4572`
permet de lancer le service s3 de localstack sur le port `4572`. 
Ici, nous définissons les services de localstack que nous voulons lancer `S3`. Si par
exemple, nous voulons lancer les services sqs et/ou sns en plus, nous écrivons :

 `SERVICES=s3,sqs,sns.`

### 2.1 Le script `start_localaws.sh`

Nous allons créer notre fichier de lancement de l’image localstack dans le
conteneur docker ainsi que la création de notre bucket s3.

Voici à quoi ressemble notre fichier `start_localaws.sh`
```bash
#!/bin/bash

docker-compose -p localaws up -d

# waits  a few seconds for localstack docker container to start
sleep 5

# s3
aws --endpoint-url=http://localhost:4572 s3 mb s3://local-s3-bucket
``` 
**Un peu d’explication :**

`docker-compose -p localaws up -d` : nous permet de lancer localstack dans un
container docker en utilisant le fichier docker-compose.yml que nous avons créé
plus haut.
```bash
`aws --endpoint-url=http://localhost:4572 s3 mb s3://local-s3-bucket`
``` 
Est la commande aws-cli qui nous permet de créer un bucket s3 au nom de `local-
s3-bucket`

Aprés avoir créé le fichier et donné les droits nécessaires pour l'exécution,
vous pouvez exécuter votre script pour lancer localstack et créer votre bucket
s3 :
```bash
./start_localaws.sh
``` 
<br/><br/>
![start_localstack](/images/localstack/start_localstack.png)

<br/><br/>
Super! Vous venez de lancer localstack, et vous avez créé un bucket s3 nommé
`local-s3-bucket`.

 Pour vérifier que votre bucket s3 est bien créé, entrez la commande suivante :
```bash
aws --endpoint-url=http://localhost:4572 s3 ls
``` 
Normalement, vous verrez la liste de vos bucket s3 local, ici en l'occurrence :
`local-s3-bucket`.
<br><br>
![ls_aws_s3_bucket](/images/localstack/ls_aws_s3_bucket.png)

### 2.2 Le script `stop_localaws.sh`

Pour arrêter docker et donc nos services localstack, nous utilisons la commande
suivante :
```bash
docker-compose -p localaws down
```
Nous allons mettre cette commande dans un fichier nommé `stop_localaws.sh` pour simplifier l'arrêt
de docker.

Voici à quoi ressemblera notre fichier `stop_localasw.sh` :
```bash
#!/bin/bash

docker-compose -p localaws down
``` 
Faisons un petit recap :

Jusque-là, nous avons créé les fichiers suivants :

![script_folder](/images/localstack/script_folder.png)

C'est tout pour les scripts, passons à notre application Spring Boot utilisatrice de notre bucket s3.

## 3- L’application Spring Boot

Maintenant que nous avons créé notre bucket s3 local, utilisons ce stockage
avec une application Spring Boot.

> Pour cet exemple, nous ne pouvons charger que des fichiers de moins de 125ko.

Voici la structure de notre application Spring Boot

![Spring Boot project structure](/images/localstack/spring_boot_project_structure.png)

 Dans un premier temps, nous allons créer notre souscripteur au Endpoint
localstack :

`EndpointRegistrar.java` :
```java 
 @UtilityClass
public class EndpointRegistrar {

  @Getter
  @AllArgsConstructor
  private enum Service {

    S3(ServiceName.S3, "http://localhost:4572");

    private String name;
    private String defaultEndpoint;

  }

  public static String getS3Endpoint() {
    return getServiceEndpoint(Service.S3);
  }

  private static String getServiceEndpoint(final Service service) {
      try {
          return LocalstackDocker.INSTANCE.endpointForService(service.getName());
      } catch (final Exception e) {
          return service.getDefaultEndpoint();
      }
  }
}
``` 

Ensuite, nous allons créer notre `S3Objectrepository.java` qui nous permet de
sauvegarder nos fichiers dans le s3 local :

```java
@RequiredArgsConstructor
@Repository
public class S3ObjectRepository {

@Value("${localaws.region}")
private String region;

  @Bean
  @ConditionalOnProperty(name = "localaws.enabled", havingValue = "false")
  public AmazonS3 localstackS3Client() {
    return AmazonS3ClientBuilder.standard()
      .withEndpointConfiguration(new
AwsClientBuilder.EndpointConfiguration(EndpointRegistrar.getS3Endpoint(), region))
      .withClientConfiguration(new ClientConfiguration().withRequestTimeout(50000))
      .withPathStyleAccessEnabled(true)
      .build();
  }

  @SneakyThrows
  public void save(final String bucketName, final String objectKey, final byte[]
objectContent, final String objectContentType) {
    try (final InputStream byteArrayInputStream = new ByteArrayInputStream(objectContent))
{
      final ObjectMetadata objectMetadata = new ObjectMetadata();
      objectMetadata.setContentLength(objectContent.length);
      objectMetadata.setContentType(objectContentType);
      localstackS3Client().putObject(new PutObjectRequest(bucketName, objectKey,
byteArrayInputStream, objectMetadata));
    }
  }

  @SneakyThrows
  public List<String> getFilesNames(final String bucketName) {

  ListObjectsV2Result result = localstackS3Client().listObjectsV2(bucketName);
  List<S3ObjectSummary> objects = result.getObjectSummaries();
  return

objects.stream().map(S3ObjectSummary::getKey).collect(Collectors.toList());
  }
}
``` 

Par la suite, nous créons notre contrôleur qui nous permet de charger des fichiers :

`FileUploadController.java`
```java
@Controller
@Slf4j
public class FileUploadController {

@Autowired
S3ObjectRepository s3ObjectRepository;

@Value("${localaws.s3-bucket.name}")
String bucketName;

@GetMapping("/bucket/home")
public String homePage(Model model) {
retrieveAllFiles(model);
return "home";
}

@RequestMapping(value = "/bucket/upload", method = RequestMethod.POST)
public String uploadFile(Model model, @RequestParam("file") MultipartFile file)

throws IOException {

log.info("uploading the file : "+file.getName());

if (!file.isEmpty()) {

String fileName = file.getOriginalFilename();
String fileType = file.getContentType();
InputStream is = file.getInputStream();
s3ObjectRepository.save(bucketName, fileName, is.readAllBytes(),

fileType);

}

return "redirect:/bucket/home";
}

private void retrieveAllFiles(Model model) {
List<String> files = s3ObjectRepository.getFilesNames(bucketName);
model.addAttribute("files", files);
}

}
``` 
Après, nous créons notre front qui nous permet de charger les fichiers et de les
envoyer vers le contrôleur.

Voir le fichier [home.html](https://github.com/bmoussat/s3demo/blob/master/src/main/resources/templates/home.html)
dans le dossier `resources/templates` de notre projet.

## Démo

Lancer localstack en exécutant le script `start_localasw.sh`.

Lancer le projet Spring Boot avec votre IDE préféré.

 - Avec eclipse : clique droit sur la racine du projet -> _executer comme -> une
application java_

 - ou en ligne de commande : `mvn spring-boot:run`

Puis aller dans le navigateur en utilisant le lien suivant :

http://localhost:8080/bucket/home

Chargez des fichiers (de moins de 125ko) en utilisant la page web explicite.

![home_page_upload_1](/images/localstack/upload_1.png)

![home_page_upload_2](/images/localstack/upload_2.png)

![home_page_upload_2_1](/images/localstack/upload_2_1.png)

![home_page_upload_3](/images/localstack/upload_3.png)

Après avoir chargé quelques fichiers, vous pouvez regarder dans votre s3 local
que les fichiers chargés sont bien dans le bucket, ici notre bucket `local-
s3-bucket`.

Entrer la commande suivante :

```bash
aws --endpoint-url=http://localhost:4572 s3 ls s3://local-s3-bucket
```
Vous verrez la liste des fichiers chargés dans votre bucket s3 local.

![s3_bucket_result](/images/localstack/result.png)

Conclusion :

Dans cet article, nous avons démarré un service s3 local en utilisant localstack
lancé dans un conteneur docker, créé un bucket s3 puis chargé les fichiers à
partir de notre projet Spring Boot.




