# Docker Discovery
## Practical Work 1 
The primary goal of this work was to discover Docker, images and containers, while creating a 3-tiers web application (Database, Backend API, Http server) with docker-compose.

The final images can be found on dockerhub : https://hub.docker.com/u/azkssd

We used for the HTTP server the image :  
https://hub.docker.com/_/httpd  
for the backend API :  
https://hub.docker.com/_/openjdk  
and for the Database :  
https://hub.docker.com/_/postgres

### 1-1 Document your database container essentials: commands and Dockerfile.
We launched a postgres database, using first the adminer interface to access it. 
To built the database container image:
`docker build -t azkssd/database .`  
To create the network:  
`docker network create app-network`  
To deploy the adminer interface:  
`docker run -p 8090:8080 --network app-network --name adminer -d adminer`  
To deploy the database, we added a volume with the -v option to save the data after stoping the container, and used environment variables with the -e option do not display our important information directly in the code:  
`docker run -p 5432:5432 --name my-postgres-container -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -v ./data:/var/lib/postgresql/data --network app-network -d azkssd/database`

Our first dockerfile, which is present at the project's root, specifies the version of the postgress application, the database name, and copies all initialisation files from a local source to the container's directory.

### 1-2 Why do we need a multistage build? And explain each step of this dockerfile.
Multistage builds in Docker help keep things clean by separating the build process from the runtime environment. This means we end up with smaller, safer Docker images because we only include what's absolutely necessary for running the app. It's all about efficiency and security.

The dockerfile at the path api/simple-api-main-students/ has two parts: Building and Running  
The building part uses the Maven image with Amazon Corretto JDK 17 as the base image and labels it as "myapp-build". This image includes all the necessary tools to build a Java application using Maven, sets the working directory inside the container to /opt/myapp, copies the pom.xml file and the src directory to the working directory in the container, and executes the Maven package command with the argument -DskipTests to compile the Java code and package it into a JAR file, skipping the tests.  
The running part switches to using the Amazon Corretto 17 image as the base image for the runtime environment, (sets again the same environment variable MYAPP_HOME to /opt/myapp and sets again the working directory inside the container to /opt/myapp), copies the built JAR file (from the previous build stage) located in $MYAPP_HOME/target/*.jar to /opt/myapp/myapp.jar in the current stage, and specifies the entry point for the container, instructing it to run the Java application by executing java -jar myapp.jar when the container starts.

### 1-3 Document docker-compose most important commands. 
`docker-compose up`
This command builds, (re)creates, starts, and attaches to containers for a service. It also starts any linked services.  
`docker-compose down`
Stops and removes containers, networks, volumes, and images created by up.  
`docker-compose build`
Builds or rebuilds services defined in the docker-compose.yml file.  
`docker-compose start`
Starts existing containers for a service.  
`docker-compose stop`
Stops running containers without removing them.  
`docker-compose restart`
Restarts services.  
`docker-compose pause`
Pauses services.  
`docker-compose unpause`
Unpauses services.  
`docker-compose logs`
Displays log output from services.  
`docker-compose ps`
Lists containers.  
`docker-compose exec`
Executes a command in a running container.  
`docker-compose down --volumes`
Stops and removes containers, networks, and volumes created by up.

### 1-4 Document your docker-compose file.
This Docker Compose configuration defines three services: backend, database, and httpd, along with a custom network called app-network.

Version  
`version: '3.7'` Specifies the version of the Docker Compose file syntax being used  
Backend Service  
`backend` Specifies the name of the service.  
`build` Defines how to build the Docker image for this service.  
`context` Specifies the path to the directory containing the Dockerfile for building the image.  
`container_name` Sets the name for the container.  
`networks` Attaches the service to the app-network.  
`depends_on` Specifies that this service depends on the database service.  
Database Service  
`database` Specifies the name of the service.  
`build` Defines how to build the Docker image for this service.  
`context` Specifies the path to the directory containing the Dockerfile for building the image.  
`container_name` Sets the name for the container.  
`networks` Attaches the service to the app-network.  
Httpd Service  
`httpd` Specifies the name of the service.  
`build` Defines how to build the Docker image for this service.  
`context` Specifies the path to the directory containing the Dockerfile for building the image.  
`ports` Maps port 8086 of the host machine to port 80 of the container.  
`networks` Attaches the service to the app-network.  
`depends_on` Specifies that this service depends on both the backend and database services.  
Network  
`networks` Defines a custom network named app-network that will be used to connect the services.

### 1-5 Document your publication commands and published images in dockerhub.
To publish our images onto our DockerHub account we have first to login  
`doxker login`  
Then we should `docker tag` our images to version them
And finally we can push them onto DockerHub with `docker push` 

## Practical Work 2
This practical work focused on setting up a CI/CD pipeline using GitHub Actions. We created workflows to automatically build, test, and deploy a Java application. SonarCloud was integrated to monitor code quality. The goal was to ensure continuous integration and delivery.

### 2-1 What are testcontainers?
Testcontainers is an open-source library that provides lightweight, disposable instances of common services like databases or web servers, all encapsulated in Docker containers. It's used in testing to ensure that tests run in an isolated and consistent environment, similar to the real world. For example, you can use Testcontainers to spin up a PostgreSQL database container for testing the insertion and retrieval of data, making sure your application behaves as expected in production.

### 2-2 Document your Github Actions configurations.
The Github actions configurations can be found at the path .github/workflows/  
It is designed to automatically run tests on the backend whenever code is pushed to or a pull request is made to the main branch. It checks out the code, sets up JDK 17, and then builds and tests the project using Maven.

Workflow Name   
`name: "CI devops 2024"` This sets the name of the workflow to "CI devops 2024".   
Trigger Events  
`on:` Specifies the events that will trigger the workflow.  
`push:` Triggers the workflow when code is pushed to the main branch.  
`pull_request:`Triggers the workflow when a pull request is made to the main branch.  
Jobs  
`jobs:` Defines a job named test-backend that will be executed.  
Job Configuration   
`runs-on: ubuntu-22.04` Specifies that the job will run on an Ubuntu 22.04 runner provided by GitHub.  
`steps:` Defines the individual steps that will be executed as part of the job.  
Checkout Code:  
`name: Checkout code
uses: actions/checkout@v2.5.0` This step uses the actions/checkout@v2.5.0 action to clone the repository's code into the runner.   
Set Up JDK 17:
`name: Set up JDK 17
uses: actions/setup-java@v3
with:
java-version: '17'` This step sets up Java Development Kit (JDK) version 17 using the actions/setup-java@v3 action.  
Specifies the version of Java to be set up.  
`distribution: 'adopt'` Specifies the distribution of JDK to use (AdoptOpenJDK).  
Build and Test with Maven:  
`name: Build and test with Maven
run:
mvn clean verify --file api/simple-api-student-main/pom.xml` This step runs Maven commands to build and test the project.  
Executes the mvn clean verify command using the pom.xml file located in the api/simple-api-student-main directory. This command cleans the project and runs all tests.

### 2-3 Document your quality gate configuration.
Additionally, we use SonarCloud to analyze our code quality. SonarCloud provides detailed insights into various aspects of our program:

Security: There are two security vulnerabilities and three security hotspots identified.  
Code Coverage: The code coverage is at 53.6%, indicating the percentage of the code executed during tests.  
Reliability and Maintainability: No issues related to reliability and maintainability were found.  
Code Quality: There are no duplications or code smells detected.  
Overall, while the code quality of our application is good, there is still room for improvement, particularly in addressing the security vulnerabilities and increasing code coverage.

## Practical Work 3
This practical work focused on using Ansible for automated application deployment. Key steps included:

Inventory Setup: Define servers in an inventory file.
Playbooks: Write and execute playbooks for server management and software installation.
Roles: Organize tasks into roles for better structure and reuse.
Deployment: Use the docker_container module to deploy applications.
CI/CD Integration: Set up GitHub Actions for automated deployment upon code release.
Tools used: Ansible, GitHub Actions, and SonarCloud.

### 3-1 Document your inventory and base commands
The inventory file can be found at the following path: my-project/ansible/inventories/

The configuration sets up a basic Ansible inventory for our host anne-zoe.kassidonis.takima.cloud in the prod group. It specifies that Ansible should connect to this host using the centos user and the SSH private key located at ~/.ssh/id_rsa locally (not in repo)

We used the `ansible all -i inventories/setup.yml -m ping` command to check the connectivity to all hosts in the inventory
And the `ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become` command to remove Apache (httpd) from all hosts using yum with superuser privileges.

### 3-2 Document your playbook
Our playbook can be found at the path my-project/ansible/ and executes tasks on all hosts defined in the inventory, disables gathering of facts about the hosts, and executes tasks with superuser privileges.  
It also launches roles, which are organized tasks installing Docker, creating a network, launching the database, application, and proxy services on all hosts.

### 3-3 Document your docker_container tasks configuration.
Finally, our docker_container tasks configuration are found at the path my-project/ansible/roles/install-docker/tasks/  
This Ansible playbook snippet installs Docker and ensures it is running on the target system:

Tasks

Install device-mapper-persistent-data:  
Uses the `yum` module to ensure that `device-mapper-persistent-data package` is installed and up to date.

Install lvm2:  
Uses the `yum` module to ensure that `lvm2` package is installed and up to date.

Add Docker repository:  
Uses the `command` module to add the Docker repository URL to the system's repository list using `yum-config-manager`.

Install Docker CE:  
Uses the `yum` module to ensure that the `docker-ce` package is installed and available.

Install python3:  
Uses the `yum` module to ensure that `python3` package is installed and up to date.

Install docker Python package:  
Uses the `pip` module to install the `docker` Python package using `pip3`.  
Sets the Python interpreter to `/usr/bin/python3`.

Ensure Docker service is running:  
Uses the `service` module to ensure that the `docker` service is started on the target system.
