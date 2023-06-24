## CONTAINERIZING AN APPLICATION USING DOCKER 

I will be containerizing the [AWS-Lift-and-Shift-project](https://github.com/dybran/AWS-Lift-and-Shift-Project/blob/main/AWS-Lift-and-Shift-Project.md).

__SCENARIO:__

We already have a multi tier application stack which has many services to be managed as an operations or DevOps team and these services are running on EC2 instances in the cloud environment. We also need to run continous changes and regular deployments.

__PROBLEMS:__

- __High Operational Expenditure:__ In the previous set up in the cloud, We will spend more in procuring the resources as the resources are over provisioned. This will increase the regular operational cost. In the application we deployed in AWS, we provisioned 10GB of RAM on the EC2 instance. The application does not use that much RAM and the remaining RAM is not used and they accumulate bills.
- __Human Errors in Deployment:__ There is a chance of human error in deployment of the resources. Automation can reduce the chance of human error.
- __Microservice:__ The application setup is not compatible with microservice architecture.
- __Not Portable:__ The application setup is not portable and the environment are not in sync. There are chances that it will work on dev envoronment and not work in production environment.

__SOLUTION:__
- __Containers:__ We can solve the above problems using containers. Containers consume very low resources and suites very well for microservices design.  The image is packaged with all the dependencies and libraries so it is portable, reuseable and repeatable and can work across any environment.

__Tools to be used in containerizing this project:__

- __Docker:__ which is a container run time environment.
- Nginx
- Tomcat
- Memcache
- Rabbitmq
- 