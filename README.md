# sociodocker
The Docker Part of the Socio Microservices Project

1) General Info About the Socio-Micro-Service-Demo

2) Specific Info Concerning Docker



## General Info ====

The Socio Micro Services Project will consist of about 10 small (backend) Springboot applications, deployed in a Docker Container/ Linux Oracle Virtual Box. SocioRegister is the principal part of a series of four applications called: starter, mock, jpa, socioregister. Together they show a stepwise buildup to a Springboot REST application, which contains use-cases for registering and adding Socios (similar to Facebook). This line of applications goes from an almost empty Springboot shell (starter: one controller method only) to a small but full-fledged REST application: SocioRegister which will be used as a component of our micro-services.

Next you`ll find four other serving applications. The simple SocioWeather, provides a weather-report by city by consulting an external REST-service called Open Weather. SocioBank, permits money transaction between Socios, also consulting an external service for exchange rates. The SocioSecurity, a Cookie/ Token based SpringSecurity (OAUTH2), still has to be written. Finally the SocioDbBatch application is interesting because it will update, on a daily bases, the databases of SocioRegister (socio_db) and SocioBank (soicio_bank_db). The DBs run on MySQL or Postgres.

From SocioRegister-jpa one finds backend-Validation (javax) and REST-Exception Handeling of Spring (RestControllerAdvice).

Testing, in general, will have an important focus and since we are dealing with Spring(boot) there will systematically testing based on five mayor strategies:

	-@ExtendWith(MockitoExtension.class)

	-@ExtendWith(SpringExtension.class) standalone setup (two ways)

	-@ExtendWith(SpringExtension.class) server tests (@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)

	-@DataJpaTest wich is database testing on H2

	-Spring Batch testing

Testing is still "work in progress"



## Specific Info Docker ====

Docker is a natural and obvious part of deploying Micro Services (DEV and PROD?). Docker containerizes a micro-services setup. A Docker container is like a "real-life" container, but in a virtual manner. There are many ways to build Docker Containers. For this tutorial I choose the following approach:

  1) Dockerfile -> is a script, of a specific format, to generate a Docker Image (docker build);
  2) Docker Image -> is the basis, the blue print, of a Docker Container;
  3) Docker Container -> usually performs one single task (eg beeing a postgres dB, or a Springboot application), so you would have a number of containers;
  3) Docker Compose -> it creates a set of docker containers based on the pre exsisting images and it defines the communication among the containers.

### The Several Components of the Socio Micro Service

Docker is a big subject and I do not intent to cover it here. What I will do is show you, by the hand of several Dockerfiles and Docker Compose YML-files, how the four (five) previous created Springboot applications will eventually work together in a Docker-Way. As a first step realizing this goal, we need to analyse the dependencies of the applications, based on there mutual DBs etc.

	1) SocioWeather REST-service -> stand alone
	2) SocioRegster REST-Service: socio_db -> stand alone
	3) SocioBank SpringBatch/ REST-Service: socio_bank_db -> stand alone
	4) SocioDbBatch SpringBatch -> socio_db, socio_bank_db -> depends on the DBs of 1) and 2)
	5) SocioSecurity has to be written still (all will depend on this one)
	
### 1) SocioWeather Report Docker and Compose files
This one is the simplest among the applications. It is a service with a single REST-controller to obtain the weather of a given City. There is no DB etc, it just consults an external service called OpenWeater Report. In order to get docker going for this service do the following:

	1) clone the socioweather project form this site (git clone https://github.com/artsgard/socioweather);
	2) copy the SocioWeather Docker and Compose file to the root of of the cloned project;
	3) Open a command line at that same root directory and invoke:
	4) mvn clean install -Dmaven.test.skip=true (which will generate the jar snapshot of the Springboot weather application at the target folder);
	5) sudo docker build ./ -t socioweather (will generate the socioweather image);
	6) sudo docker-compose up (will instanciate the container based on one single image, called: socioweather:latest);
	7) http://localhost:8083/Barcelona (will show you the current weather json of Barcelona);

### 2) SocioRegister Docker and Compose files

The steps are about the same, but this time there will be two images and two containers, one for the Springboot app and one for the socio_db postgres DB. Before touching the docker-stuff you have to do one other important thing! You have to change the url of each DB-connection (DataSource) and make it NOT point to localhost. I will list this action under 0) Of the preceding list:

	0) at DBConfig.java (at the root-folder) change the url into: dataSourceBuilder.url("jdbc:postgresql://sociodb:5432/socio_db");
	1) clone the socioregister application;
	2) copy the SocioRegister Docker and Compose file to the root;
	3) Open a command-line;
	4) mvn clean install -Dmaven.test.skip=true
	5) sudo docker build ./ -t socioregister
	6) sudo docker-compose up
	7) http://localhost:8081/socio


### 3) SocioBank Docker and Compose files

This application is simular to the previous one. But now we are dealing with three images because of the fact that there are two postgres DBs, one for the bank data and one for the integrated SpringBatch (dealing with the internal springbatch meta data). Here you also have to touch the two DataSource urls at src.main.resources:

	-app.datasource.batch.url=jdbc:postgresql://sociobankmetadb:5432/socio_bank_batch_meta_data_db
	-app.datasource.db.url=jdbc:postgresql://sociobankdb:5432/socio_bank_db

	For the moment I am having an issue with the auto-generate (initialize) of the Meta data tables. Spring should generate them (spring.batch.initialize-schema=always) but something is going wrong since MySQL (I am trying it out in MySQL) is hanging on creating the tables..... I hope to resolve this soon!


### 4) SocioDbBatch Docker and Compose files

This time there are four images! One is the new Batch-application-image and the other three are DB-images. Interestingly two DB-images are refering to previous declared images, or in other words, they are pointing to the previous declared socio_db and Socio_bank_db of 2) and 3). The third DB-images is the Batch-Meta_dat-DB. The urls to be changed at src.main.resources:

	-app.datasource.batch.url=jdbc:postgresql://sociobatchmetadb:5432/socio_batch_meta_data_db
	-app.datasource.db.url=jdbc:postgresql://sociodb:5432/socio_db
	-app.datasource.db.url=jdbc:postgresql://sociobankdb:5432/socio_bank_db

	But first I have to resolve the problem of 3)
