
SpringOne 2020 Session Notes
 ===================

- [Kubernetes + YAML + Science = Sanity](#kubernetes---yaml---science---sanity)
- [From Java Monoliths to K8s](#from-java-monoliths-to-k8s)
- [Center of the JVM Universe](#center-of-the-jvm-universe)
- [Spring Boot Loves K8s](#spring-boot-loves-k8s)
- [Introducing Spring Cloud Gateway and API Hub for VMware Tanzu](#introducing-spring-cloud-gateway-and-api-hub-for-vmware-tanzu)

---


### Kubernetes + YAML + Science = Sanity ###
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

### From Java Monoliths to K8s ###
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

#### Center of the JVM Universe ####

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
### Spring Boot Loves K8s ###

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
- Expose `Configmap` and `Secret` Kubernetes resources as Spring Boot `PropertySource`
- Layered Jars by default

---

### Introducing Spring Cloud Gateway and API Hub for VMware Tanzu ###
https://youtu.be/KPoCba3F4nY

Tanzu Application Service - VMware platform for modern application deployment build on top of CloudFoundry. Use `cf push` to quickly deploy applications at scale. 

Use ` cf create-service` to use services available from marketplace.

Use `cf bind-service` to bound the service to an app which exposes the URL and credentials required to connect to the service instance.

Spring Cloud Gateway for Tanzu is a commercial VMware offering of Spring Cloud Gateway Open Source. Uses a lot of open source Spring projects such as Spring Cloud, Spring boot, Spring Security and Project reactor. 

There is a difference between Spring Cloud Gateway Open source and Spring Cloud Gateway for VMware Tanzu. SCG Open source is a tool-kit used to build an API gateway from scratch. SCG for VMware Tanzu is a commercial offering and is already built inside the Tanzu platform. You can just use `cf create-service ...` to get started with VMware Tanzu SGW. In addition there is networking, security, platform integrations with this offering.

Function of an API Gateway
 - With many backend microservices you can expose all of them through a single endpoint in the API gateway
 - Easily change the target endpoint for a service by routing it in the API gateway
 - Can build on security, rate limits, authorization

 Route defines how to Gateway will process incoming requests
 - Predicates - test/conditions whether a route matches any given request
 - Filters - apply behaviour to matching requests or their responses
 - Destination URI - where we need to send the request to

To create a simple SCG instance:
 `cf create-service p.gateway standard my-gateway`

p.gateway - name of the service offering in marketplace
standard - only available plan in the marketplace
my-gateway is the name of the gateway

There will be no routes however at the start but you can provide a JSON configuration with -c configuration along using

```
 cf create-service p.gateway standard my-gateway -c '{
	 "routes":[{
		"predicates": [...]
		"filters": [...]
		"uri": <string>

	 }
	 ]

 }'
````

`stripPrefix=1` can be used in the filter section - Drop the first component of the URI. For e.g. if the url is 'a/service/path' then it will strip it down to '/service/path'

This command is handled by the service broker which goes and creates the route in the gateway instance.

To remove routes you can use `cf unbind-service my-app my-gateway`

Within TAS each app can have external URI( connect from outside) and internal URI(connect from inside provided network policies are set up).  When a `bind-service` is made for the SCG and the app, automatically container network policy is configured to allow internal communication between the SCG and app containers. 

Rate-limiting and Single sign on can be applied in SCG

you can use the `cf update-service` to update routes create at Gateway creation time.

You can also use Gateway bound-apps actuator API to update the routes by making a PUT request

Look at the SGW predicates that are also available in VMware SGW for Tanzu like checking headers, host , methods etc

**Filters in SCG**

Filters allows you to do things with request/request like add headers, remove header, redirect, rewrite path etc. 

- Single-Sign-On Filter

	A 'Single-sign-on' service or an authentication proxy is also available in TAS that configures multiple Identity providers with SCG using SAML or OpenID

	Plan - configure Single Sing Identity providers and mapping parameters
	Configure SCG to integrate wit a plan

	Use `sso-enabled=true` in your per-route definition to have SSO integration. For routes which do not have these configuration will not have SSO enabled.

	Gateway will first check if a session is valid i.e. a valid token is available and then it follows the Authorization code flow. SCG redirects to a login page with the Identity provider. After entering the credentials the Identity provider will return a SAML assertion for e.g. which will be converted into a JSON token. 

	Use `token-relay: true` to pass authorization information from Spring Cloud Gateway to backend service in case you need that information in backend.  SCG will also perform the authorization url which can be used to retrieve the public keys to get the authorization information.

	You can also perform authorization at the SCG by providing some additional configuration in routes. 

 - Mutual TLS (mTLS) 

   TAS Gorouter is like an ingress controller that handles all traffic from external world.
   All external and internal communication requires mutual TLS where both the client and server have to provide valid certificates to establish the connection.

   When the goRouter forwards the requests to th SCG it sends it to side-car proxy implemented by envoy proxy. These two establish the mTLS connection by using certificates that are automatically managed by the platform.  Similarly SCG sends the request to backend application via an envoy proxy running as a side-car alongside the backend certification.

   You can use filters to do additional validations on the certifications like check the CommonName value

- Rate Limiting

	Prevents API endpoints being bombarded with many requests.

	Open Source SCG also provides rate-liming but requires an external source to maintain the request numbers. In VMware Tanzu SCG it is provided by an in-memory grid within the platform that stores all details related to rate limiting.

	```
	cf create-service p.gateway standard my-gateway -c '{
	"rate-limit": "100,1s"
	}'
	```
	uses leaky Bucket algorithm and greedy replenishment approach. In above example it does not allow 100 requests in 1 second but 1 request every 10 ms approximately. 

- you can specify the number of instances of SCG for high availability by passing `count` parameter during the service creation
	```
	cf create-service p.gateway standard my-gateway -c '{
	"count": 5
	}
	```
	When gateway instance is more than one, it uses a in-memory data grid. This is used to store shared state such as session and rate limiter data

- By default the domain of SCG is `<gateway-service-name>.<platform apps domain>`

	This can however be configured by using: 
	```
	cf create-service p.gateway standard my-gateway -c '{
		"host": "api"
		"domain": "my-domain.com"
	}
	```

- Gateway can be configured to handle Global CORS support.
	
	```
	cf create-service p.gateway standard my-gateway -c '{
		"cors":{
			"allowed-origins"" []
			...
		}
	}
	```
	You dont need to specify all options, you can only do so for some which you require

**API Hub**

- A place where we can see all our APIs together. Work in progress..
- Can aggregate information from multiple SCG in the environment in  single catalog
- Search for an API using the web console
- Gateways organized as tiles which can be clicked to get further details. You can see version, security configuration, apps configured, rate limiting, and 'Try it out' button.
- 'Try it out' to send an actual request to the backend service and validate the response
- API hub is another Spring boot which exposes OpenAPI spec for all routes configured. You can generate automatically a client based on the OpenAPI specification, Generate your own documentation as well
- Manage API keys in Gateway - Revoke, issue new tokens. Manage access to your applications.  Work in Progress

**Demo**

- Animal Rescue: React + Spring Boot   
- A dashboard is a feature of SCG which provides a high level summary of the SCG instance.  You need to be authorized to access the dashboard
- The manifest file for `cf push` specifies routes which are both internal. They are then configured to be accessible via SCG
- To make the endpoints available in the SCG you use the `cf bind-service` with a configuration of routes

---
