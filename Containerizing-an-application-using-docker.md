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

__Tools to be used in containerizing this project:__

- Docker - container run time environment.
- Nginx
- Tomcat
- Memcache
- Rabbitmq
- MySQL


__STEPS:__
- We should already be aware of the steps to setup the stack from the [AWS-Lift-and-Shift-project](https://github.com/dybran/AWS-Lift-and-Shift-Project/blob/main/AWS-Lift-and-Shift-Project.md).
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

Then run `$ vagrant up` to bring up the VM

![](./images/vup.PNG)

Install Docker Engine using the [Documentation](https://docs.docker.com/engine/install/ubuntu/).

Then `$ vagrant ssh` to login

Add the vagrant user to the docker group

`$ usermod -aG docker vagrant`

![](./images/vfd.PNG)

Log out and login then run 

`$ id vagrant`

![](./images/ids.PNG)

