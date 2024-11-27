---
dg-publish: true
---
# Docker Desktop instructions
1. Pull the code into the server.
2. Run `mvn clean` to clean the Maven project.
3. Run `mvn install` to build the project and install dependencies.
4. Build a Docker image using the command:
    
    `docker build -t btd-ui:1.0`
    
    Replace `btd-ui:1.0` with any new name and version you prefer. This command will place the deployable file into a new image.
5. In Docker Desktop (for now), navigate to Images and then click "run" on the newly created image.
6. When prompted, map the container ports to the server ports. For instance, use `8081:8081` to expose the port to Nginx.

# Docker Command Line Instructions 
1. Pull the code into the server.
2. Run `mvn clean` to clean the Maven project.
3. Run `mvn install` to build the project and install dependencies.
4. Build a Docker image using the command:
    
    
    `docker build -t btd-ui:1.0 .`
    
    Replace `btd-ui:1.0` with any new name and version you prefer. This command will build the Docker image using the Dockerfile in the current directory (`.`).
5. Run the Docker container using the built image:
    
    `docker run -p 8081:8081 btd-ui:1.0`
    
    Replace `8081:8081` with the desired host:container port mapping. This will expose port 8081 of the container to port 8081 on the host.