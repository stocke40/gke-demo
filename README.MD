# Demo Spring Boot App

In chapter 2 of the book [Pro Spring Boot 2](https://www.amazon.com/Pro-Spring-Boot-Authoritative-Microservices/dp/1484236750/ref=asc_df_1484236750/), the author takes the reader through the creation of a demo application, called TODO, to demonstrate how easy it is to create an application using Spring Boot.  This README.MD file in this git repo contains my notes on going through that process.  Where I can, I will try to have the notes here in the git repo, where I cannot, those notes will appear in my accompanying [DemoSpringBootApp](https://docs.google.com/document/d/1fJfHCTvL4EDmIVlIbAvb6n04_C8p-0cf_VtRm3LIGEY/edit#) Google Docs file.  That is the thought, anyway.

My first goal was to create an application and get it to successfully execute with bootRun.

My second goal (Squirrel!) was to containerize that application and run it locally on my laptop.

My third goal is to deploy that image to GKE and run it in the cloud.

And, you're thinking it, and YES, this is similar to [turning on a light](https://youtu.be/BW9FbjjkKo4) in The Big Bang Theory.

## Create an application and get it to run locally

### Created starter application using Sprint Initializr

I used the [Spring Initializr](https://start.spring.io/) to create the initial starter application 18-Dec 2022.

### Installed a JDK

I installed a JDK on my MSI Laptop on 3-Dec 2022.  It was raining and we couldn't yet go outside to do our Christmas lights.
  
* [Oracle JDK 19 Documentation](https://docs.oracle.com/en/java/javase/19/index.html)
* [Oracle JDK 19 API Documentation](https://docs.oracle.com/en/java/javase/19/docs/api/index.html)
* [Oracle JDK 19 Windows MSI Installer](https://www.oracle.com/java/technologies/downloads/#jdk19-windows)

#### Problems

***

##### ERROR: JAVA_HOME is not set and no 'java' command could be found in your PATH

In settings; Environment Variables, User variables for grego
C:\Users\grego\AppData\Local\Programs\Python\Python38-32\Scripts\
C:\Users\grego\AppData\Local\Programs\Python\Python38-32\
%USERPROFILE%\AppData\Local\Microsoft\WindowsApps

Moved all of these to the end of the path in system variables,  DELETED the path variable for grego, and moved the following to the END of the path,
which the JDK installer placed at the beginning of the path:

C:\Program Files\Common Files\Oracle\Java\javapath

Also added a JAVA_HOME variable in the system variables and assigned it to the same above path

***

##### ERROR: JAVA_HOME is set to an invalid directory: C:\Program Files\Common Files\Oracle\Java\javapath

Interesting, I chose that location since it was where the Oracle JDK 19 installer set in the system path -- even though that is not where it is actually installed.  

See the below [markdown](https://www.markdownguide.org/basic-syntax/) snippet of a successful java --version command output:

```bash
grego@MSI MINGW64 ~/gitrepos/TastyAPI
$ java --version
java 19.0.1 2022-10-18
Java(TM) SE Runtime Environment (build 19.0.1+10-21)
Java HotSpot(TM) 64-Bit Server VM (build 19.0.1+10-21, mixed mode, sharing)

grego@MSI MINGW64 ~/gitrepos/TastyAPI
$
```

I am not going to try to untangle this -- I am simply going to set JAVA_HOME to where Gradle and/or VSCode wants it, and leave the path as the JDK19 installer set it.

***

##### Can't use Java 19.0.1 and Gradle 7.5.1 to import Gradle project TastyAPI

This looks like a VSCODE generated error.  And it may have something to do with my [mismatched releases of stuff](https://docs.gradle.org/current/userguide/compatibility.html).  I am done for today.  Will pick this up later...

##### Downloaded Oracle JDK 11 after creating Oracle account

27-Dec 2022.

##### Installed JDK 11

2-Jan 2023.  Installed JDK 11.  Updated JAVA_HOME and Path system environment variables and restarted MSI laptop.  Used Spring Initializer to create *another* demo repo specifying the right versions ['n nat](https://pittsburghspeech.pitt.edu/PittsburghSpeech_PgheseOverview.html)).  Spring still modified the Java Version from 11 to 17 -- see [HELP.MD](.doc/HELP.md) and the [JDK Version Range](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-Versions#jdk-version-range) on the wiki for more details.

See [Spring Framework Support](https://spring.io/projects/spring-framework#support).

Spring Boot 2.7.6 (24-Nov 2022; commercial support through 2025)
[Spring Framework 5.3.24](https://docs.spring.io/spring-framework/docs/5.3.24/reference/html/) (commercial support through 2027)

Manually modified build.gradle to specify spring boot version and java source compatibility to 11.

Even though my VSCODE gitbash terminal escape sequences are weird, gradlew bootRun successfully builds and executes the initialized application.  Yay!

To test that we opened up a browser on the (MSI Laptop) and hit the [http://localhost:8080/](http://localhost:8080/) URI.  Doing so returns the return string from [WebController.java](./src/main/java/com/example/demo/WebController.java), "Hello Spring Boot!", to the browser.

Yay!  Goal 1 is complete.  Success!

## Containerize application and execute it locally

Now that we have successfully compiled and executed our application, I wanted to containerize the application and execute it locally.

### Installed Docker Desktop

Installed Docker Desktop 4.15.0 (93002) to get a docker daemon running on MSI Laptop, since this is required in order for ./gradlew bootBuildImage to build an OCI container.  Used WSL2 instead of Hyper-V (because it was recommended in the installer).

### Build a container image

Now that Docker Desktop is installed, you can [build a container image](https://ashoksubburaj.medium.com/build-docker-image-using-spring-boot-buildimage-gradle-ac5bc1f71303).

You will need to ensure that Docker Desktop (and a docker daemon) is running on the build machine.  *In my case, I need to manually start Docker Desktop after each MSI Laptop restart.*

Now that a docker daemon running on the build machine (MSI Laptop), you can create a container image with the following command:

```bash
grego@MSI MINGW64 ~/gitrepos/demo
$ ./gradlew bootBuildImage
```

You can then see the result with the docker image ls command:

```bash
grego@MSI MINGW64 ~/gitrepos/demo
$ docker image ls
REPOSITORY                 TAG              IMAGE ID       CREATED        SIZE
paketobuildpacks/run       base-cnb         9aa882ca71d8   47 hours ago   87MB
demo                       0.0.1-SNAPSHOT   b3f1735ae16e   43 years ago   261MB
paketobuildpacks/builder   base             2a27a16f2a71   43 years ago   1.2GB
```

But where did this image go?  Where is the image stored locally on my computer?  How do we find out more information about the resulting image?  We'll save these questions for a bit later.  For now, we'll push on and assume that magic will just take care of it.  

### Run a container image locally

After we have built a container image, we can run the image with a [docker run](https://docs.docker.com/engine/reference/run/) command:

```bash
grego@MSI MINGW64 ~/gitrepos/demo
$ docker run -d --name greg_demo_app -p 8080:8080 b3f1735ae16e
ca45eaed002995ceae938bc9fca6335d941f6004efa8436a418f3fd65c017348
```

We can verify that the docker run has created a running container instance from the image by issuing a docker ps command:

```bash
grego@MSI MINGW64 ~/gitrepos/demo
$ docker ps
CONTAINER ID   IMAGE          COMMAND              CREATED         STATUS         PORTS
      NAMES
ca45eaed0029   b3f1735ae16e   "/cnb/process/web"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->8080/tcp   greg_demo_app
```

Since we have exposed the 8080 port inside the container to the outside world (outside the docker network), we should be able to open up a browser on the host machine (MSI Laptop) and hit the [http://localhost:8080/](http://localhost:8080/) URI.  Doing so returns the return string from [WebController.java](./src/main/java/com/example/demo/WebController.java), "Hello Spring Boot!", to the browser.  Success!

### Accessing log data from a running container

Since we ran the container in detached mode (with the docker run -d option), we will not get our console log output at the terminal by default.  But we can access the logs via the docker logs command:

```bash
grego@MSI MINGW64 ~/gitrepos/demo
$ docker logs greg_demo_app
Setting Active Processor Count to 16
Calculating JVM memory based on 25013188K available memory
For more information on this calculation, see https://paketo.io/docs/reference/java-reference/#memory-calculator
Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -Xmx24418442K -XX:MaxMetaspaceSize=82745K -XX:ReservedCodeCacheSize=240M -Xss1M (Total Memory: 25013188K, Thread Count: 250, Loaded Class Count: 12195, Headroom: 0%)
Enabling Java Native Memory Tracking
Adding 124 container CA certificates to JVM truststore
Spring Cloud Bindings Enabled
Picked up JAVA_TOOL_OPTIONS: -Djava.security.properties=/layers/paketo-buildpacks_bellsoft-liberica/java-security-properties/java-security.properties -XX:+ExitOnOutOfMemoryError -XX:ActiveProcessorCount=16 -XX:MaxDirectMemorySize=10M -Xmx24418442K -XX:MaxMetaspaceSize=82745K -XX:ReservedCodeCacheSize=240M -Xss1M -XX:+UnlockDiagnosticVMOptions -XX:NativeMemoryTracking=summary -XX:+PrintNMTStatistics -Dorg.springframework.cloud.bindings.boot.enable=true

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.7.6)

2023-01-07 19:34:56.695  INFO 1 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication using Java 11.0.17 on ca45eaed0029 with PID 1 (/workspace/BOOT-INF/classes started by 
cnb in /workspace)
2023-01-07 19:34:56.698  INFO 1 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to 1 default profile: "default"
2023-01-07 19:34:57.346  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-01-07 19:34:57.354  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-01-07 19:34:57.354  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.69]
2023-01-07 19:34:57.407  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-01-07 19:34:57.407  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 672 ms
2023-01-07 19:34:57.648  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-01-07 19:34:57.656  INFO 1 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 1.277 seconds (JVM running for 1.54)
2023-01-07 19:39:08.336  INFO 1 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2023-01-07 19:39:08.337  INFO 1 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2023-01-07 19:39:08.338  INFO 1 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms

grego@MSI MINGW64 ~/gitrepos/demo
$
```

Getting log output (and other Telemetry) from containerized applications, while simple in concept, is actually a pretty big and complex topic.  So we'll save that for another time.

### Summary so far

So far, we have built and executed a sample REST Controller application on our laptop.  We have also packaged that application into an [OCI image](https://github.com/opencontainers/image-spec), and we have executed that image as a container under a container runtime environment, Docker Desktop, locally on our laptop.

Next up, we want to take that image and deploy it to a public cloud service provider like Google Cloud Platform (GCP).

## Deploy and execute the image to the cloud

Our next and final goal will be to deploy an image to Google Cloud Platform and execute it under Google Kubernetes Engine (GKE).

### Push the image to a registry

It appears that the first thing that we need to do is get the OCI image to the cloud.  It appears that GKE expects the image to be in some sort of container registry, and GKE will "pull" that image from that container registry to create a running instance of that code.  *A running instance of an image is called a container.*  Here is an article that explains what a container registry is, how to choose a registry, and the popular registries available: [How to Choose a Container Registry: The Top 9 Picks](https://bluelight.co/blog/how-to-choose-a-container-registry).

*It appears that was happening locally in our last step as well, but the details were abstracted from us -- it was magic.*

For this project, I will likely be choosing between Docker Hub and Google Artifact Repository (GAR).

[Docker Hub](https://docs.docker.com/docker-hub/) is in the running simply because Docker Desktop defaults to Docker Hub.  

[Google Artifact Repository](https://cloud.google.com/artifact-registry) is in the running because Google Cloud Platform is our current deployment target.  The above article contains a general observation that the recommended artifact registry is often the one associated with your cloud provider.

*In the case of a software vendor, they may have a much more complicated CI/CD pipeline for a few reasons.  First, they may be providing both container images, helm charts and possibly other artifacts.  Second, they may be packaging and bundling artifacts by products and SKUs into tar files and distributing them via a support site.  Third, they may need to support any deployment target of a customer's choosing.  Finally, they may need to support the container registry preferences of their customers.*

Since I am targeting GKE, I *could* just start with [CI/CD on Google Cloud](https://cloud.google.com/docs/ci-cd), but that would make too much sense, and would expand the list of things that I "know I do not know" as aggressively.

#### Docker Hub

Docker Hub Personal (free) only allow one private repository, and only allows manual image pushes.  Docker Hub Pro allows unlimited private repositories, [automatic image pushing](https://docs.docker.com/docker-hub/builds/) from GitHub, and costs $5 per month.  See [Docker Hub Plans](https://hub.docker.com/billing/plan/update) for more information about the options available.

I must have created an account with Docker Hub a while ago under the user id gshrivepitsteel.  See the [Docker Hub Repositories documentation](https://docs.docker.com/docker-hub/repos/) for help on pushing an image to a Docker Hub Repo.  The local docker build (from the previous section) did not name and tag the image in a way that allowed for direct pushing to Docker Hub.  The documentation referenced above provided several alternatives for dealing with this.  I chose to simply re-tag the existing image and then push the image to my newly created Docker Hub private repository, [gshrivepitsteel/demo](https://hub.docker.com/repository/docker/gshrivepitsteel/demo/general).

```bash
grego@MSI MINGW64 ~/gitrepos/demo
$ docker image ls
REPOSITORY                 TAG              IMAGE ID       CREATED        SIZE 
paketobuildpacks/run       base-cnb         bc9712e00864   10 days ago    87MB 
paketobuildpacks/run       <none>           9aa882ca71d8   2 weeks ago    87MB 
paketobuildpacks/builder   base             b35ddaec9008   43 years ago   1.2GB
<none>                     <none>           b3f1735ae16e   43 years ago   261MB
demo                       0.0.1-SNAPSHOT   e54fa2842f73   43 years ago   261MB
paketobuildpacks/builder   <none>           2a27a16f2a71   43 years ago   1.2GB

grego@MSI MINGW64 ~/gitrepos/demo
$ docker tag demo:0.0.1-SNAPSHOT gshrivepitsteel/demo:0.0.1-SNAPSHOT

$ docker image ls
REPOSITORY                 TAG              IMAGE ID       CREATED        SIZE
paketobuildpacks/run       base-cnb         bc9712e00864   10 days ago    87MB
paketobuildpacks/run       <none>           9aa882ca71d8   2 weeks ago    87MB
gshrivepitsteel/demo       0.0.1-SNAPSHOT   e54fa2842f73   43 years ago   261MB
demo                       0.0.1-SNAPSHOT   e54fa2842f73   43 years ago   261MB
paketobuildpacks/builder   <none>           2a27a16f2a71   43 years ago   1.2GB
paketobuildpacks/builder   base             b35ddaec9008   43 years ago   1.2GB
<none>                     <none>           b3f1735ae16e   43 years ago   261MB

grego@MSI MINGW64 ~/gitrepos/demo
$ docker push gshrivepitsteel/demo:0.0.1-SNAPSHOT
The push refers to repository [docker.io/gshrivepitsteel/demo]
1dc94a70dbaa: Pushed
66139dae5e26: Pushed
b0ab7e0bbf67: Pushed
5f70bf18a086: Pushed
0a423f241a26: Pushed
a258d3332031: Pushed
1fcf7948c0aa: Pushed
6484a15f3b9f: Pushed
6763b99e3792: Pushed
30a03cb7ff3f: Pushed
7039cb19c7da: Pushed
7fbc97c38fad: Pushed
672c633068cb: Pushed
ec0381c8f321: Pushed
e6aa48c81965: Pushed
9651e85005c8: Pushed
308b4b02da9a: Pushed
be948c2f3324: Pushed
d88b70389417: Pushed
0e974cc1d198: Pushed
dc622bd7e11b: Pushed
16dcc2eadf63: Pushed
101b05ef38e1: Pushed
0.0.1-SNAPSHOT: digest: sha256:4a54b035f953a510d4758592b5c5565964f9c8da95b2d8f2da6bfb11a36862e4 size: 5325

grego@MSI MINGW64 ~/gitrepos/demo
$ 
```
