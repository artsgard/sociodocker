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
	2) copy the Docker and Compose file to the root of of the cloned project;
	3) Open a commanf line at that same root directory and invoke:
	4) 
