---
dg-publish: true
---
# Dockerfile Explanation

This Dockerfile is used to build a Docker image for a Jenkins environment with additional tools and dependencies installed. Below is a breakdown of what each step in the Dockerfile does:

1. **Base Image Selection**: 
   - `FROM jenkins/jenkins:lts`: This line specifies the base image to be used, which is `jenkins/jenkins` with the LTS (Long Term Support) tag.

2. **Switch to Root User**: 
   - `USER root`: Switches the user to root to perform administrative tasks.

3. **Package Installation**:
   - Updates the package lists and installs necessary packages using `apt-get`.
   - Installs `apt-transport-https`, `ca-certificates`, `curl`, `gnupg2`, and `software-properties-common` needed for adding repositories and installing packages securely.
   - Adds Docker's official GPG key and adds the Docker repository.
   - Installs `docker-ce-cli`, `maven`, `zip`, `unzip` for Docker, Maven, zip operations, and Java respectively.

4. **Node.js Installation**:
   - Adds the Node.js repository using a script fetched with `curl`.
   - Installs Node.js and npm using `apt-get`.

5. **Verify Node.js and npm Installation**:
   - Checks the installed versions of Node.js and npm using `node -v` and `npm -v` respectively.

6. **Angular CLI Installation**:
   - Installs Angular CLI globally using npm.

7. **Java Installation**:
   - Downloads the latest version of Java from Adoptium using `curl`.
   - Extracts the downloaded tarball and moves it to `/opt/jdk21`.
   - Sets up alternatives for `java` and `javac` commands to point to the installed Java version.
   - Removes the downloaded tarball.

8. **Setting JAVA_HOME**:
   - Sets the `JAVA_HOME` environment variable to point to the installed Java version.
   - Updates the `PATH` environment variable to include the Java binary directory.

9. **Switch to Jenkins User**:
   - Switches the user back to `jenkins` user for running Jenkins server.

This Dockerfile sets up an environment suitable for running Jenkins with Docker, Maven, Node.js, npm, Angular CLI, and Java installed.


```
FROM jenkins/jenkins:lts

USER root

# Install required packages for Docker, Maven, zip, and Java
RUN apt-get update && \
    apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common && \
    curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - && \
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" && \
    apt-get update && \
    apt-get install -y docker-ce-cli maven zip unzip

# Install Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs

# Verify Node.js and npm installation
RUN node -v && npm -v

# Install Angular CLI
RUN npm install -g @angular/cli@17.3.4

# Install Java from Adoptium
RUN curl -Lo temurin.tar.gz https://api.adoptium.net/v3/binary/latest/21/ga/linux/x64/jdk/hotspot/normal/eclipse?project=jdk && \
    tar -xzf temurin.tar.gz && \
    mv jdk-21* /opt/jdk21 && \
    update-alternatives --install /usr/bin/java java /opt/jdk21/bin/java 1 && \
    update-alternatives --install /usr/bin/javac javac /opt/jdk21/bin/javac 1 && \
    rm temurin.tar.gz

# Set JAVA_HOME
ENV JAVA_HOME /opt/jdk21
ENV PATH $JAVA_HOME/bin:$PATH

USER jenkins

```

## Building the Jenkins image
To build: ` docker build -t my-custom-jenkins .` in the command line from the directory containing the docker file. 

## Running the Jenkins image
To run: `docker run -d --name jenkins -p 8082:8080 -p 50000:50000 -e DOCKER_HOST=tcp://host.docker.internal:2375 -v C:/jenkins_home:/var/jenkins_home my-custom-jenkins`
This command maps the docker runtime to the Jenkins instance to allow docker in docker control from within Jenkins. 