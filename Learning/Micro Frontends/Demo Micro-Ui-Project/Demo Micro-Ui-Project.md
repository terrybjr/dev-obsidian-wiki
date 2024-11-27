---
dg-publish: true
---

![[ezgif.com-video-to-gif-converter.gif|  750]]


The demo-micro-ui project consists of 6 individually deployable applications. We have 3 front ends, 3 backends.  The front ends consist of natively federated angular applications that share a common root context with a split context for the "children" of the root project. 

# demo-micro-ui-root
This application is a micro-frontend web application built using a combination of Java, Maven, TypeScript, JavaScript, and Angular. It is designed to be a modular and scalable solution for modern web development. The root application (`demo-micro-ui-root`) serves as the main entry point and integrates various federated modules to provide a cohesive user experience. Additionally, the application is integrated with Open Liberty, a lightweight Java EE runtime, to ensure robust and efficient server-side processing.

The setup for this root app follows the [[Hello World#Step 3 Execute Automated Setup for Host]] 

## Project Structure  
  
### `demo-micro-ui-root`  
  
This is the parent Maven project that includes the `web-app` module and manages the federated modules.  
  
### `web-app`  
  
This module contains the Angular application and is packaged as a WAR file for deployment.

## Key Files
### `ibm-web-ext.xml` File

The `ibm-web-ext.xml` file is a configuration file used in IBM WebSphere and Open Liberty environments to define additional settings for web applications. This file is located in the `WEB-INF` directory of the `web-app` module.

### web-app/WebContent/WEB-INF/ibm-web-ext.xml
````xml nums
<?xml version="1.0" encoding="UTF-8"?>  
<web-ext  
        xmlns="http://websphere.ibm.com/xml/ns/javaee"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xsi:schemaLocation="http://websphere.ibm.com/xml/ns/javaee http://websphere.ibm.com/xml/ns/javaee/ibm-web-ext_1_0.xsd"  
        version="1.0">  
  
    <context-root uri="/micro-ui"/>  
  
</web-ext>
````
The `<context-root>` element specifies the URI path


### web-app/pom.xml
The `pom.xml` file for the `web-app` module defines the necessary configurations and dependencies to build and package the Angular application as a WAR file. It includes plugins for compiling Java code, packaging the WAR file, and executing Node.js commands to build the Angular application.

For the purposes of the micro-frontend, a couple particular lines within this pom.xml are important. The execution: copy-prod-manifest, ensures that we have a separate manifest for production ('build'), and development ('serve'). We replace the manifest within the target directory, once the application has built. 

We also set the --base-href to /micro-ui/ this is to match the endpoint of the web-context. 
``` XML nums
                <executions>
                    <execution>
                        <id>copy-prod-manifest</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>exec</goal>
                        </goals>
                        <configuration>
                            <executable>cp</executable>
                            <workingDirectory>${project.basedir}/angular/public</workingDirectory>
                            <arguments>
                                <argument>federation.manifest.prod.json</argument>
                                <argument>${project.build.directory}/federation.manifest.json</argument>
                            </arguments>
                        </configuration>
                    </execution>
                    <execution>
                        <id>npm-install</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>exec</goal>
                        </goals>
                        <configuration>
                            <executable>npm</executable>
                            <workingDirectory>${project.basedir}/angular</workingDirectory>
                            <arguments>
                                <argument>install</argument>
                            </arguments>
                        </configuration>
                    </execution>
                    <execution>
                        <id>ng build</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>exec</goal>
                        </goals>
                        <configuration>
                            <executable>ng</executable>
                            <workingDirectory>${project.basedir}/angular</workingDirectory>
                            <arguments>
                                <argument>build</argument>
                                <argument>--base-href</argument>
                                <argument>/micro-ui/</argument>
                            </arguments>
                        </configuration>
                    </execution>
                </executions>
```
### web-app/angular/src/index.html
This otherwise 'vanilla' angular html file has had the /micro-ui/ value added to it's href. 

``` html nums
<!doctype html>  
<html lang="en">  
<head>  
  <meta charset="utf-8">  
  <title>Angular</title>  
  <base href="/micro-ui/">  
  <meta name="viewport" content="width=device-width, initial-scale=1">  
  <link rel="icon" type="image/x-icon" href="favicon.ico">  
</head>  
<body>  
  <app-root></app-root></body>  
</html>
```

### web-app/angular/angular.json
When building with "Federation" micro services, the initialization substitutes the default angular builder with a federation builder. This happens "Automagically" when you execute the initialization scripts. What does not happen automatically however, is the addition of the href context values. These will need to be added to avoid file not found errors as well as configure ng serves context. 
``` json nums
"build": {  
  "builder": "@angular-architects/native-federation:build",  
  "options": {},  
  "configurations": {  
   "production": {  
    "target": "angular:esbuild:production"  
   },  
   "development": {  
    "target": "angular:esbuild:development",  
    "dev": true  
   }  
  },  
  "defaultConfiguration": "production"  
},  
"serve": {  
  "builder": "@angular-architects/native-federation:build",  
  "options": {  
   "target": "angular:serve-original:development",  
   "rebuildDelay": 0,  
   "dev": true,  
   "port": 0  
  }  
},  
"extract-i18n": {  
  "builder": "@angular-devkit/build-angular:extract-i18n"  
},  
"test": {  
  "builder": "@angular-devkit/build-angular:karma",  
  "options": {  
   "polyfills": [  
    "zone.js",  
    "zone.js/testing"  
   ],  
   "tsConfig": "tsconfig.spec.json",  
   "assets": [  
    {  
     "glob": "**/*",  
     "input": "public"  
    }  
   ],  
   "styles": [  
    "src/styles.css"  
   ],  
   "scripts": [],  
        "baseHref": "/micro-ui/",  
        "deployUrl": "/micro-ui/"  
  }  
},
```

### web-app/angular/public/federation.manifest.json 
``` json nums
{  
  "app1": " http://localhost:4201/micro-ui/app1/remoteEntry.json",  
  "app2": "http://localhost:4202/micro-ui/app2/remoteEntry.json"  
}
```

The file web-app/angular/public/federation.manifest.json is a configuration file used in a micro-frontend architecture. It defines the remote entry points for different micro-frontend applications, allowing them to be dynamically loaded into the main application. In this case, it specifies the URLs for app1 and app2, which are hosted on different ports (4201 and 4202 respectively) and have their remote entry points located at /micro-ui/app1/remoteEntry.json and /micro-ui/app2/remoteEntry.json. This setup enables the main application to integrate and communicate with these micro-frontend applications seamlessly.

### web-app/angular/public/federation.manifest.prod.json

``` json nums
{  
  "app1": "/micro-ui/app1/remoteEntry.json",  
  "app2": "/micro-ui/app2/remoteEntry.json"  
}
```
The file above is what the production build manifest is replaced with, so that the routes to the dependencies can be dynamically loaded. The reason why this doesn't work for live reload, is that we cannot live reload every single project on the same port number. Maybe we can, but this is the route I took for now.  
# demo-micro-ui-app(n)
App(n) is a demo child app where n is the number of child apps. In this example we have 2 child apps. The general structure of the child apps is identical except for the setup in [[Hello World#Step 4 Execute Setup for Remotes]]

For now I'm not going to publish the differences in the remotes from the "hosts" except to say that the remotes do not have or need manifests. These applications also have a different application context that follows the same rules as the context defined for the root application. 

pay special attention to line 6, where we have the context defined. This matches the context definitions in the other files. 

``` html nums
<!doctype html>  
<html lang="en">  
<head>  
  <meta charset="utf-8">  
  <title>Angular</title>  
  <base href="/micro-ui/app1/">  
  <meta name="viewport" content="width=device-width, initial-scale=1">  
  <link rel="icon" type="image/x-icon" href="favicon.ico">  
</head>  
<body>  
  <app-root></app-root>  <script src="https://unpkg.com/flowbite@1.5.3/dist/flowbite.js"></script>  
</body>  
</html>
```