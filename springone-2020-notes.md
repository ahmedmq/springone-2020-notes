#### Kubernetes + YAML + Science = Sanity
[https://youtu.be/cdfEQlTmDq8](https://youtu.be/cdfEQlTmDq8)

[`Octant`](https://octant.dev/) - real time graphical user interface of what is going on inside the Kubernetes cluster
 
**Pro Tip1**: Generate YAML configuration using `kubectl -o yaml`

**Pro Tip2**: Once the YAML files are generated for a specific resource ('pristine source'), do not edit the source files directly for any changes. Instead apply changes on top of these source files. 

Tools to help you apply changes on top of the 'pristine' source:

1. [`Kustomize`](https://kustomize.io/) - Customize resource configuration

2. [`kapp`](https://get-kapp.io/) : part of carvel - To validate the changes and also see the differences when applying the files. Also keep track of the changes
				-	Shows a changes summary of the deployment to confirm before they are applied
				-	`--diff-changes` shows details difference against versions of resources in the cluster

**Pro Tip3**: Use tools that can update the config in ways that are more approachable to humans. Let people know what they can change in the config

1. [`kpt`](https://opensource.google/projects/kpt) - Annotate lines in YAML and then you can specify description, define constraints

2. [`ytt`](https://get-ytt.io/): part of carvel - templating language for YAML
				- Use 'macros' to extend the functionality within the YAML syntax
				- Patch YAML using the `overlay` package

3. [`ytt-gen-ng`](https://github.com/bryanl/ytt-gen-ng) - Still under development, tool by Bryan. Web based browser for YAML editing

---

#### From Java Monoliths to K8s
https://youtu.be/E8TfLWMD-4U

Repo: https://github.com/salaboy/from-monolith-to-k8s/
This repo contains all the information related to this talk:  The repo includes code for 4 microservices :  C4p (Call for Proposal), Agenda, and Email Service and an API gateway service which acts as a front-door for these microservices. This is conference scheduling application which accepts proposals for talks from users and once approved by back office slots them into a schedule in conference day.

Traditionally we had a monolith with a single artefact bundled as JAR/EAR and deployed on Apache Tomcat, JBoss or IBM Websphere.  With microservices we have the single artefact broken down into multiple applications each using its own set of technologies like Spring Boot for development, docker for images and help for packaging and deployment. There are two main consideration in moving from monoliths to microservices
1. Technical Considerations - Use of tooling, the technical infrastructure to build, package and deploy microservices
2. Business Value - Benefit of going to microservices, ease of changing application logic, and do they understand what is happening in the microservices

CI/CD is a must to automate as much as you can to deploy microservices .
Jenkins X - CI/CD for Kubernetes uses Tekton as underlying engine
-  From Source to Running in K8s - An opinionated way of deploying source to K8s
- Trunk Based development - Directly merge into master and do builds for every merge from master
- GitOps approach for environment using Helm. A Git repository that specifies the desired state of the cluster and another application that watches for changes in the Git repository and syncs with the cluster.
- Preview environments - Preview changes before they move ahead- Once a pull request is submitted Jenkins X create a preview environment of the pull request which can be viewed and shared for early feedback.
Full traceability of what is running on the cluster by using the same version number for JAR artefact, Docker tag, Helm chart version. The application also shows the same version to link everything together

With microservices you have multiple services and there is a risk of ensuring services are integrated together to work as a single application. Helm can be used to create a single application package by using a chart. Think Helm as maven tooling for kubernetes.  

The Helm charts are stored in a separate Git repository and all the dependencies for each of the microservices are listed as requirements. Once any of the service application is built and a new version is released , Jenkins X can do a pull request on the Helm chart Git repository to indicate a new version of the service. Then a manual merge can be done to release a new version of the application chart.


Tools & Applications
**Spring Cloud Gateway**
- Single entry point for all microservices. Configured via YAML where you can specify routes to different microservices 
- Use circuit breaker pattern to handle failures of API request 
 
**Spring Boot 2.3.3**
-  includes Kuberenetes Livenesss and readiness Probes in Actuators

**Spring Cloud Open API specification** 
- documentation for API that are exposed by the microservices

**Spring Cloud Contracts** 
- Consumer driven contract Testing. Specify a set of contract that define the requests and response for a particular API. 
- A stub is automatically created for consume to verify the integration. Use `autoConfigureStubRunner` in your consumer tests to create a new instance of the stub by fetching it from the artefact repository and running it.

**Zeebe** - Distributed Workflow engine

---

****Center of the JVM Universe****

https://youtu.be/_P22DQ8glac?t=307

New Unit of Deployment 

Traditionally for a Spring boot application we would do a `mvn package` to generate a  FAT jar which is a self contained jar with all its dependencies and run as `java-jar application.jar`

With Kubernetes in picture now we need to build a docker image out of the Spring boot application and deploy it into Kubernetes. You could do this by create a `dockerfile` and then do docker build
Typically you have 3 layers of a docker image: OS, JDK and my-application.jar. This has two problems:
1. Single layer of the whole application JAR containing all code/dependencies
2. FAR Jar overhead resuling in slow startup

We could tackle 1 by separating into multiple layers like Application, Snapshot dependencies, Spring boot loader, Dependencies, JDK and OS. Now only when application code changes we build only that specific layer and not others resulting in faster builds
 
With Spring boot 2.3.3 you can create a multi-layered docker image by using `jarMode`. When a jar is launched with `jarMode` with the `layertools` and `extract` options the jar can self extract into folders which can then be used in a multi stage docker file. For `layertools` to know which folders to extract to it uses a `layer.idx` file

`RUN java -Djarmode-layertools -jar application.jar extract`

Another way to build an image from a Spring Boot application is to use buildpacks. With Spring boot 2.3.3 you could do 

`mvn spring-boot:build-image` to create an image using buildpacks. This does not require a dockerfile to be created

With Spring boot 2.3.3 you can use `/actuator/liveness` and `/actuator/readiness` to signal kubernetes about the health status of the application. You could also use the Spring boot graceful shutdown feature to handle in-flight transactions

With Spring boot 2.4 `configMap` and `secrets` will be made available as Spring property sources.

Spring boot Initializer uses 2.3.3 to deploy to Kubernetes

---
****Spring Boot Loves K8s ****

https://youtu.be/nPACI6-J9Jc
https://github.com/snicoll/spring-boot-loves-k8s


How Spring boot is adopting the best practices of Kubernetes and can be applied to other deployment platforms.

Scribe - Markdown editor with spell check. Has a preview tab to show the actual result of the text. Application does a spell check and then calls a external Markdown converter endpoint to convert markdown to HTML

Efficient container Images

Simple docker file is to build an image with the Fat jar. This is very in-efficient as it builds the whole application in a single layer.

Use `dive` to verify the contents of the images
You could expand the jar using `RUN jar-xf ./application.jar` and then use the layers in the next stage to create a layered docker image. This option however requires you to modify the ENTRYPOINT to correctly specify the classpath of the layer folders and is also verbose

An improvement to above is to extract the layers in a format which is similar to a Spring Boot application and then have the ENTRYPOINT as the `org.springframework.boot.loader.JarLauncher`

Layered JARs are also faster as the JAR is already exploded on disk.

Opt-in to the layered JAR mode by updating the POM file (`spring-boot-maven-plugin.configuration.layers.enabled = true`) and repacking the app. The different layers spit out by default are:
dependencies, spring-boot-loader, snapshot-dependencies,application

if you need to create a separate layer then in the case of maven you can use `layers.xml` in the resources folder of the application. Once you update the layers in `layers.xml` you still need to go update in the dockerfile the layers to be included

Use the build-image goal of Spring boot maven plugin to build an image using buildpacks. Refer to Center of JVM universe talk for more details.

**Liveness and Readiness**

Liveness 
CORRECT - app working fine
BROKEN - app is broken, like a cache is broken or app hung in infinite loop

Readiness
ACCEPTING-TRAFFIC - app ready to accept traffic
REFUSING-TRAFFIC - app load is high or still not up

To opt-in enable using `management.endpoint.health.probes.enabled=true`. If Spring Boot detects the app is running on K8s then this is enabled by default

By default there is no binding of all the groups into the health/liveness and health/readiness . This means that these two endpoints may still show status=UP but the /health endpoint my show a failure since there was a failure but was handled by a circuit breaker.

You could also set the liveness state manually by emitting an `AvailabilityChangeEvent`

**Graceful shutdown**

In case of rolling update or scaling or application shutdown you may want to complete the in-flight requests

`server.shutdown=graceful` . The default grace period is 30 sec. By default it is immediate.

The application will not process new requests but will complete on going requests.

**Spring Boot 2.4**
- Graceful shutdown improvements
- Configmap and secret kubernetes resources as Spring Boot propertysource
- Layered Jars by default