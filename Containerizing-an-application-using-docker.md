## CONTAINERIZING JAVA STACK APPLICATION WITH DOCKER 

This documentation provides step-by-step instructions on how to containerize a Java application using Docker. Containerization allows you to package your application along with its dependencies into a self-contained unit called a Docker container, which can be easily deployed and run on any system with Docker installed.

I will be containerizing the [AWS-Lift-and-Shift-project](https://github.com/dybran/AWS-Lift-and-Shift-Project/blob/main/AWS-Lift-and-Shift-Project.md).

__SCENARIO:__

We already have a multi tier application stack running on EC2 instances in the cloud environment which has many services to be managed by the operations or DevOps team. We need to run continous changes and regular deployments on the application.

__PROBLEMS:__

- __High Operational Expenditure:__ In the previous set up in the cloud, We will spend more in procuring the resources as the resources are over provisioned. This will increase the regular operational cost. In the application we deployed in AWS, we provisioned 10GB of RAM on the EC2 instance. The application does not use that much RAM and the remaining RAM is not used and they accumulate bills.
- __Human Errors in Deployment:__ There is a chance of human error in deployment of the resources. Automation can reduce the chance of human error.
- __Microservice:__ The application setup is not compatible with microservice architecture.
- __Not Portable:__ The application setup is not portable and the environment are not in sync. There are chances that it will work on dev envoronment and not work in production environment.

__SOLUTION:__
- __Containers:__ We can solve the above problems using containers. Containers consume very low resources and suites very well for microservices design.  The image is packaged with all the dependencies and libraries so it is portable, reuseable and repeatable and can work across any environment.

__Services to be used in containerizing this project:__

- Docker - container run time environment.
- Nginx
- Tomcat
- Memcache
- Rabbitmq
- MySQL


__STEPS:__
- We should already be aware of the steps to setup the stack from the [AWS-Lift-and-Shift-project](https://github.com/dybran/AWS-Lift-and-Shift-Project/blob/main/AWS-Lift-and-Shift-Project.md).
- Fork the [vprofile-project](https://github.com/devopshydclub/vprofile-project) to my github.
- Find the right base images from dockerhub for all the services that we will be using for this setup
- Write Dockerfiles to customize images (mysql, tomcat and nginx) and  build the images.
- Write a docker compose.yml file to run multiple containers.
- Test the image and push to dockerhub.
  
__ARCHITECTURAL DESIGN__

![](./images/zzz.PNG)

First, fetch the source code from github, then write Dockerfiles for the services that needs customization. The base images mentioned in the Dockerfile will be pulled from dockerhub. Then build the images. Once the images are ready, we will mention all the containers with the image name in the docker compose file and test it. If this works well we will then push the images to dockerhub.

__Finding the right base images from Dockerhub__

First, I will check the requirements we need to set up these services (If there are any customizations to be made) and see how it relates to the images we have on dockerhub.


For our application we need five images for our services:

__Tomcat__ - Customized image will be created from base image.

__MySQL__ - Customized image will be created from base image.

__Memcached__ - Use base image and override settings.

__RabbitMQ__ - Use base image and override settings.

__Nginx__ - Customized image will be created from base image.

__N/B:__
I will be using official images as they have good documentation that will help in decision making on the version given to me by the developers.

__Setting up the Docker Engine__

We can use Ec2 instance or Vagrant to set this up. If i use EC2 instance i need to use atleast a __t2.small__ to make this setup work efficiently but i will be incuring cost. I will be using vagrant to save cost. 

Create a folder 

`$ mkdir Docker-Engine`

`$ cd Docker-Engine`

Pick an ubuntu box from the [Vagrant cloud](https://app.vagrantup.com/boxes/search?utf8=%E2%9C%93&sort=downloads&provider=&q=) and run 

`$ vagrant init <vagrant-box>`

![](./images/weq.PNG)

Open the Vagrantfile and assign a unique IP address and increase the RAM size for the setup to run faster.

![](./images/qwqw.PNG)

Clone the repository

`$ git clone https://github.com/dybran/Containerizing-a-JAVA-Stack-Application.git`

Then `$ vagrant ssh` to login

Install Docker Engine using the [Documentation](https://docs.docker.com/engine/install/ubuntu/).

Add the vagrant user to the docker group

`$ usermod -aG docker vagrant`

![](./images/vfd.PNG)

Log out and login then run 

`$ id vagrant`

![](./images/ids.PNG)


__N/B:__
Make sure it is in the same folder as the __vagrantfile__.

![](./images/cln.PNG)

__Create the repositories in Dockerhub__

Create repositories in dockerhub for the application, the database and the nginx images.

![](./images/qws.PNG)


__Write the Dockerfiles for Tomcat, Mysql and Nginx__

__For the Tomcat:__ This is where the application will reside. We need to build the artifact before building the image. When building the artifact, so many dependencies are downloaded which will increase the size of the image and we need to make the image as small as possible for portability.

We will introduce a multi stage Dockerfile to help us build the artifacts in another image then copy over just the artifacts to another image and built.

__Create a Dockerfile for the app - tomcat__

```
FROM openjdk:11 AS BUILD_ARTIFACT_IMAGE
RUN apt update && apt install maven -y
RUN git clone https://github.com/dybran/vprofile-project.git
RUN cd vprofile-project && git checkout docker && mvn install

FROM tomcat:9-jre11
LABEL "Project"="Vprofile"
LABEL "Author"="Solomon Onwuasoanya"
LABEL "Description"="Building VprofileApp Image"
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=BUILD_ARTIFACT_IMAGE vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
```
![](./images/appp.PNG)

__Create a Dockerfile for DB - MySQL__

```
FROM mysql:8.0.33
LABEL "Project"="Vprofile"
LABEL "Author"="Solomon Onwuasoanya"
LABEL "Description"="Building Vprofiledb image"

ENV MYSQL_ROOT_PASSWORD="sa4la2xa"
ENV MYSQL_DATABASE="accounts"


ADD db_backup.sql docker-entrypoint-initdb.d/db_backup.sql
```
![](./images/vdbb.PNG)

__Create a Dockerfile for Web - Nginx__

```
FROM nginx
LABEL "Project"="Vprofile"
LABEL "Author"="Solomon Onwuasoanya"
LABEL "Description"="Building Vprofileweb image"

RUN rm -rf /etc/nginx/conf.d/default.conf
COPY nginvproapp.conf /etc/nginx/conf.d/vproapp.conf
```
![](./images/nginx.PNG)

Write a __docker-compose.yml__ file to build and test it together.

__Create docker-compose.yml file__

Use information from [__vprofile-project\src\main\resources\application.properties__](https://github.com/dybran/Containerizing-a-JAVA-Stack-Application/blob/main/vprofile-project/src/main/resources/application.properties) to write the __docker-compose.yml__ file.

```
version: '3.8'
services:
  vprodb:
    build:
      context: ./Docker-files/web
    image: dybran/vprofiledb
    ports:
      - "3306:3306"
    volumes:
      - volume-vprodb:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=sa4la2xa

  vprocache01:
    image: memcached
    ports:
      - "11211:11211"

  vpromq01:
    image: rabbitmq
    ports:
      - "15672:15672"
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest

  vproapp:
    build:
      context: ./Docker-files/app
    image: dybran/vprofileapp
    ports:
      - "8080:8080"
    volumes: 
      - volume-vproapp:/usr/local/tomcat/webapps

  vproweb:
    build:
      context: ./Docker-files/web
    image: dybran/vprofileweb
    ports:
      - "80:80"
volumes:
  volume-vprodb: {}
  volume-vproapp: {}
```
__Build and Run__

Switch to root user

`$ sudo -i`

Change into the __/vagrant/__

`$ cd /vagrant/`

Change into the __vprofile-project__

`$ cd vprofile-project`

![](./images/bui.PNG)

Then run 

`$ docker compose up -d` 

![](./images/compose-d.PNG)

Check images

`$ docker images`

Check for running conainers

`$ docker ps`

![](./images/ps-a.PNG)

Then we can access the app from the webpage using the IP address

![](./images/brower.PNG)

Login using the log in details below

Login: __admin_vp__

Password: __admin_vp__

![](./images/er.PNG)

Verify our entire setup

Succesful login shows that the database server is connected.

To check if the rabbitMQ is functioning, we click on rabitMQ on the website.

![](./images/po.PNG)

To check the memcached, click on all users and select a user.

![](./images/xz.PNG)

Then go back and select the same user

![](./images/xz2.PNG)

We will find out that the first time we selected the user the data was from the Database and it took longer time to open but the second time it was faster to access and was from the cache.

__Push to dockerhub__

To push to dockerhub, we login

`$ docker login`

Provide dockerhub __username__ and __password__.

Then push images

`$ docker push <image-name>`

![](./images/dlo.PNG)

![](./images/dlo2.PNG)

The DevOps Team works with the developement Team to understand the development/setup steps for the application and also understand the different services that is needed for the setup.
