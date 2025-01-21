+++
date = '2022-08-03T22:15:49+01:00'
title = 'Facilitez vos tests d’intégrations et/ou les améliorer en utilisant les Testcontainers.'
description = "Testcontainers facilite l’uniformisation de l’environnement de dev, s’assurer que les applications en cours de développement peuvent être testées indépendamment de l’environnement local d’un développeur spécifique."
tags = ['java', 'spring-boot', 'testcontainers', 'junit', 'mysql', 'lombok']

+++

## Introduction
Testcontainers a été créé par Richard North et Sergei Egorov founder en 2015.

C'est un ensemble de librairies construites au-dessus de docker, que vous pouvez utiliser à chaque fois que vous avez besoin de tester votre code ayant des dépendances externes, comme une base de données, une queue etc ...
<!--more-->
Pourquoi utiliser Testcontainers

Testcontainers facilite l’uniformisation de l’environnement de dev, s’assurer que les applications en cours de développement peuvent être testées indépendamment de l’environnement local d’un développeur spécifique.
Il permet aussi de faire des tests d’intégrations en utilisant une vraie base de donnée sans quelle ne soit physiquement installée sur la machine  , ce qui rend les tests un peu plus fiables.

Introduction au domaine

Pour notre article, nous allons utiliser une application springboot avec laquelle je vous montrerai comment écrire un test d’intégration pour une application comportant une base de donnée en utilisant Testcontainers.

Notre test d’intégration.
Vous avez accès au code source de cet article ici (lien gitlab).

## 1. Dépendances

À part les dépendances spring boot, nous devons rajouter les dépendances suivantes avant de commencer  :

- Testcontainers qui nous permettra de lancer la base nécessaire et l'éxecution des tests.

- Mysql qui est la base de donnée qui sera chargée par testcontainers, puis utilsée afin de valider les tests.


```xml
<dependencies>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
<dependency>
	<groupId>org.testcontainers</groupId>
	<artifactId>mysql</artifactId>
	<scope>compile</scope>
</dependency>
<dependency>
	<groupId>org.testcontainers</groupId>
	<artifactId>testcontainers</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.testcontainers</groupId>
	<artifactId>junit-jupiter</artifactId>
	<scope>test</scope>
</dependency>
    ...
</dependencies>
```

## 2. Model

Je vous présente notre model qui sera utilisé pour nos différents tests.

```java
@Table(name = "company")
public class Company implements Serializable{
/**
*
*/
private static final long serialVersionUID = 1L;

	@Id
	@GeneratedValue(strategy = GenerationType.AUTO)
	@Column(name = "id", unique = true, nullable = false)
	private int id;

	private String name;

	private String country;

	private String city;

}
```
Notre model `Company` (entreprise ou société) a quatre attributs  :

- `id` (identifiant unique)
- `name` pour nom,
- `country ` pour pays
- `city` pour ville

Structure du projet  complet  :

Je vous présente les services que nous voulons tester :

```java
public interface ICompanyService {

    void save(Company company);

    List<Company> getAllCompanies();
}

``` 
* `save (Company company)` : nous permet de sauvegarder une entreprise
* `getAllCompanies()`     : nous permet de récupérer toutes les entreprises

## 3. Nos tests

Pour les tests, nous avons créé une classe _**abstraite**_ qui sera notre classe de base qui sera héritée par les classes qui auront besoin de ses éléments, ici la configuration nécessaire à l'utilisation de Tesrtcontainers.

Dans cette configuration, nous précisons : 
 * le type de base de donnée que nous souhaitons utiliser -> `mysql:8.0`
 * le nom de la base de donnée -> `app_db`
 * les identifiants de connection  -> `Username, Password`
 * les propriétés pour la source de données du conténaire utilisé 

Cette approche fonctionne pour la plupart des bases de données.
```java
@SpringBootTest(classes = Application.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("it")
public abstract class BaseIT {

    @Autowired
    public TestRestTemplate restTemplate;

    private static final MySQLContainer mySQLContainer;

    static {
        mySQLContainer = (MySQLContainer) (new MySQLContainer("mysql:8.0")
                .withUsername("root")
                .withPassword("")
                .withDatabaseName("app_db")
                .withReuse(true));
        mySQLContainer.start();
    }

    @DynamicPropertySource
    public static void setDatasourceProperties(final DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mySQLContainer::getJdbcUrl);
        registry.add("spring.datasource.password", mySQLContainer::getPassword);
        registry.add("spring.datasource.username", mySQLContainer::getUsername);
    }
}
```
Après la configuration nous pouvons écrire notre premier test d'intégration.
Pour ce faire, nous avons créé la classe suivante :

```java
class CompanyServiceIT extends BaseIT {

    @Autowired
    ICompanyService companyService;

    @Test
    @Sql({"/database/dump_app_db.sql"})
    void whenCompanyIsSaveThenCompanyIsinDB() {

        Company company = new Company();
        company.setName("One company inc");
        company.setCity("paris");
        company.setCountry("France");
        companyService.save(company);

        List<Company> companies = companyService.getAllCompanies();
        assert (companies != null);
        assert (companies.size() > 0);
        assert ("France".equals(companies.get(0).getCountry()));
        assert (companies.get(0).getId() != 0);
        System.out.printf(String.valueOf(companies.get(0)));
    }
}
```

Pour pouvoir tester nos enregistrements en base, même si nous avons déjà 
créé la connection à une base de données, notre base est vide.
Nous nous attendons à avoir une certaine structure concernant la base de donnée, donc il nous faut des tables pour faire nos opérations.
Pour cela nous avons l'annotation spring `@Sql` qui prend en paramètre des bumps de base de données ou des scripts sql qui 
nous permettrons d'avoir la structure de base de donnée attendue.

Voici les parties les plus intéressantes de notre "dump" sql `dump_app_db.sql` :

```sql
CREATE DATABASE  IF NOT EXISTS `app_db` ;
USE `app_db`;



--
-- Table structure for table `company`
--

DROP TABLE IF EXISTS `company`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `company` (
        `id` int auto_increment,
        `name` varchar(45) COLLATE utf8_general_mysql500_ci DEFAULT NULL,
         `country` varchar(45) COLLATE utf8_general_mysql500_ci DEFAULT NULL,
         `city` varchar(45) COLLATE utf8_general_mysql500_ci DEFAULT NULL,
         PRIMARY KEY (`id`)
)
```

Au lancement, le script sera chargé, puis les tests serons exécutés.

Nos tests devraient s'exécuter sans erreurs.

# Conclusion

Nous avons mis en place une configuration qui nous permet de lancer un test qui contrôle un comportement de notre code
concernant l'accès à la base de données, de sauvegarde de société puis de la recherche et récupération de toutes les
sociétés en base. 