---
dg-publish: true
---
# Jenkins Pipeline Script for Building and Deploying Dockerized Application

```groovy
pipeline {
    agent any // This pipeline can be executed on any available agent

    stages {
        stage('Clean') { // Stage for cleaning the project
            steps {
                sh 'mvn clean' // Execute 'mvn clean' command
            }
        }

        stage('Install') { // Stage for installing the project dependencies
            steps {
                sh 'mvn install' // Execute 'mvn install' command
            }
        }

        stage('Stop Existing Docker Container') { // Stage for stopping any existing Docker container
            steps {
                script {
                    // Stop any running Docker containers from the 'btd-app:latest' image
                    sh '''
                       docker ps -q -f ancestor=btd-ui:latest | xargs -r docker stop
                       docker ps -a -q -f ancestor=btd-ui:latest | xargs -r docker rm
                    '''
                }
            }
        }

        stage('Build Docker Image') { // Stage for building the Docker image
            steps {
                script {
                    docker.build('btd-ui:latest', './web-app') // Build Docker image from the Dockerfile in the 'btd-app-ear' directory
                    sh 'docker image prune -f' // Remove all dangling Docker images
                }
            }
        }

        stage('Replace Image in Container') { // Stage for replacing the running Docker container with a new one
            steps {
                script {
                    // Run the Docker container from the 'btd-app:latest' image
                    sh 'docker run --restart unless-stopped -d -p 9081:9081 btd-ui:latest'
                }
            }
        }
    }
}
```

This Jenkins pipeline script defines a multi-stage build process for a Dockerized application. Each stage represents a specific task in the build and deployment process.

## Stages Explanation:

1. **Clean**: This stage executes the `mvn clean` command to clean the project directory before building.
    
2. **Install**: Installs the project dependencies using the `mvn install` command.
    
3. **Stop Existing Docker Container**: Stops and removes any existing Docker containers running the `btd-ui:latest` image. This ensures that the environment is clean before building a new image.
    
4. **Build Docker Image**: Builds the Docker image named `btd-ui:latest` from the Dockerfile located in the `./web-app` directory. After building the image, it prunes any dangling Docker images to free up disk space.
    
5. **Replace Image in Container**: Runs a new Docker container from the `btd-ui:latest` image with port mapping (`-p 9081:9081`) and restart policy (`--restart unless-stopped`).
    

This pipeline automates the process of building, deploying, and updating the Dockerized application, ensuring consistency and reliability in the deployment process.


