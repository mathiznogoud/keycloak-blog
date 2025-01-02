# How we created the PaaS platform App.Farm - the digital heart of RSHB

Hi, Hubr, my name is Konstantin Belkin, I am Teamlead SRE at RSHB-Intech. Today I will tell you about App.Farm - a PaaS platform that we have been independently developing and supporting since September 2020.

The main purpose of implementing this product is to create conditions for import substitution of RSHB's high-critical information systems and stimulate the development of RSHB's own development. We aim to: minimize the bank's dependence on vendor solutions and outsourcing support for these solutions, develop internal competencies for software maintenance and support, and create all necessary conditions for the bank's internal developers.

PaaS platform App.Farm also pursues a regulatory goal - transition to import-substituted solutions and open source software and rejection of proprietary solutions that were implemented in the bank in the past decades.

App.Farm is built on the principles and methodologies of GitOps and IaC. These, in turn, pursue the principle of Everything as Code (EaC). Currently, more than 60 bank systems/services are deployed in App.Farm, which generate a daily flow of at least 50 million messages and requests per day. For centralized work with our solution, 21 vendor products have been adapted and 290 external integrations have been performed for 30 external customers.

You may have heard about App.Farm at various conferences, seen mentions in articles on Hubre or in the media, but we haven't made a solid presentation as such. Now it's time to tell you what App.Farm is and share our plans for the future. The solution is very large-scale, so in this article I will present the general points, features and components of the platform. In the future, we'll break down each of them in more detail.

So here we go.

![image](https://github.com/user-attachments/assets/7bd7d9fe-ab83-47ae-9dc1-bcddcf18507d)

## History, goals and objectives

Work on the solution began in 2020. The goals were global but realizable, although there was a lot of work to be done. It was necessary to centralize the development of information systems for the RSHB from the so-called “zoo” of vendors and disparate systems and improve internal processes.

* Reducing the time-to-market of version releases through a gradual transition to the PaaS (Platform as a Service) model.
	* In this model, all activities related to organizing product development are provided “as a service”: tools for working with source code and releases, code building and verification processes, software deployment processes, collection and review of telemetry, authentication / authorization in applications, integration of different communication protocols, databases, queues, caches, etc.

* Increased throughput and scalability with increasing load on the Integration subsystem. 

* Getting rid of vendor lock-in and reducing costs by reducing the cost of ownership of the solution.
	* On the old integration solution, in addition to the high cost of licenses directly, there were also annual payments for support of the solution to the vendor. 

* Standardization of development processes across the bank and consolidation of development in a single environment.
	* Elaboration and implementation of rules for standardization: internal development processes, stack-dependent source code writing conventions, approaches for working with business application telemetry, operation and maintenance of business applications, creation and maintenance of integration interactions.

At the end of the day, we needed to present a ready-made toolkit that allows us to:
* Store and share source code;
* Check source code for quality and vulnerabilities;
* Collect, publish and use artifacts;
* Deploy off-the-shelf business applications in various environments;
* Establish integration interactions over various protocols.

Automation was and is an important aspect of the work. One of the main goals is to minimize the human factor and bureaucracy in the processes, which is exactly due to automation. And this means that the following things were to be done:
* Ready coordinated network diagrams;
* Automatic opening of network accesses;
* Work with information security in an automated mode;
* Automated integration flow reconciliation process;
* Automated build and delivery of business applications to the productive;
* Automated application deployment.

We took on the job from scratch in September 2020. And the initial version of the product had support for the following set of tools:
* Gitlab and basic CI/CD processes for major popular development languages (Java, Kotlin, JS) built on Gitlab CI;
* CPU and RAM resource allocations;
* Description of Systems and Services as Code;
* Support for Deployment Deployment;
* Storage of logs and metrics in a unified storage system;
* Support for typical HTTP-to-HTTP and MQ integrations.

In June 2021, App.Farm had already been put into production. The second half of 2021 was marked by global updates and refinements to the platform. We managed to implement and launch a bunch of useful and necessary things: 

Support for: .NET Core, projects using Lerna, deployment of projects built outside the platform, integration (e2e) tests built into the CI/CD pipeline, complex (with transformation) and composite adapters in the platform, support for delivery (CDL) of arbitrary types of standalone artifacts appeared.

* Going CI/CD into production.
* Announcement of overlay subnets via BGP.
* Control of subdivision resource quota in the platform.
* Custom dashboards in the monitoring subsystem.
* Kafka as a Service
* Autoscailing

In 2022, the following were added to this list: editing platform entities on the portal, feedback from the CI/CD pipeline on the deployment of platform entities, and the introduction of distributed tracing functionality. A key innovation this year was work on a proprietary DisasterRecovery solution to improve the bank's disaster resilience.

## Platform components and architecture

App.Farm is a platform consisting of various infrastructure components working together. These components are encapsulated in the platform and are integral parts of the platform. That is, App.Farm does not provide Kubernetes or other components to the platform as a service. Instead, there is an API of the platform itself, which can hide various components behind it. We can swap them out for any other without having to notify, negotiate and migrate systems that have come to depend on it directly.

App.Farm is built on the following open source solution stack:
* Kubernetes;
* Gitlab;
* OpenSearch;
* Grafana;
* Jaeger;
* Istio;
* VictoriaMetrics;
* and other open source products.

As well as on proprietary tools:

* A placer of Kubernetes platform operators;
* Dockerfile generator;
* Integration adapters;
* Platform Deployment Toolkit.

For the convenience of working with applications and services hosted in App.Farm, an information portal for managing platform entities has been developed. It is used by developers, architects, testers, IT managers and engineers of the bank's systems to obtain up-to-date information on the full status of their services and applications hosted on the platform in the context of the bank's various development circuits.

The application execution environment of the App.Farm platform, which uses Kubernetes under the hood, enables the deployment of services and information systems, as well as the creation of integration links between services. The solution is based on a set of more than 30 open source components and technologies: Java, Go, Python, .net (C#), Kotlin, JavaScript, TypeScript, NodeJS, React, Angular, OCI-Image.

The CI/CD development pipeline used in the platform regulates unified code standards and development architecture for all developed and implemented solutions, which allows to significantly reduce the delivery time to the product, improve the quality and security of the bank's information systems. Due to the unified approach and standards of development and architecture, the App.Farm platform supports a single SLA for all hosted information systems. Systems can have their own SLA, but we offer a quality SLA of a single layer in the amount of 99.85%. In addition, the platform sets a unified approach and standards for development and architecture for the entire bank by providing a single service desk (SRE).

To support the platform, we have extensive capability documentation that describes requirements and guidelines for developers in great detail. In addition, we hold regular training and mitups to make it easier for colleagues to work in App.Farm, and have opened a chat room with tech support, announcements, and a separate channel for publishing platform updates.

## Main components
### Runtime Environment: Kubernetes + Istio 

Kubernetes is an internal infrastructure component of the platform used as an application execution environment. Users have no direct access to Kubernetes, only the platform can host and manage applications. The user may not know anything about the existence of Kubernetes, they interact with it through the platform tools (e.g. “Portal”).

Istio is a Service Mesh for Kubernetes, implemented on the basis of the Sidecar pattern, i.e. before each service in Kubernetes a small proxy service is started, which passes through all incoming and outgoing traffic. This allows us to perform any manipulations with it - inspect, control, encrypt, supplement or cut, authorize, journal, and so on.

Service Mesh manages traffic between services, diagnoses errors. It is designed to solve problems by controlling the interaction of services with each other. In particular: dynamic service discovery, routing and traffic management, encryption, authentication and authorization, metrics collection and monitoring, http tracing, outbound traffic management (istio-egressgateway). Service Mesh is very useful when there are a large number of services and cascading communications between them. Also, istio-egressgateway controllers simplify outbound traffic management by applying network access policies to external resources.

Kubernetes and Istio together provide a huge number of features that applications don't need to implement on their own, such as: 

* Container startup and management, orchestration;

* Application isolation and compute resource management;

* Service discovery;

* Traffic balancing;

* Application scaling;

* Various upgrade and rollback strategies.

And also provide features such as:

* Traffic introspection and monitoring of network interactions;

* Two-way graph encryption;

* Service Grid - the ability to get information about all applications and their interactions;

* Integration with other infrastructure components: tracing, logging, monitoring;

* Authentication and authorization of services.

## Monitoring: VictoriaMetrics + Grafana

In our platform, the monitoring stack is:
* Victoria Metrics (Cluster) as the database;
* As a collector - VMAgent;
* As a display tool: Grafana.

Users have direct access to Grafana only, allowing them to view their application metrics and create dashboards. At the same time, dashboards are deployed as code from git repositories of projects.
Initially Prometheus was used in the platform for monitoring. But due to the development of the platform, a number of problems with high resource consumption, low performance and lack of multitenancy appeared. We managed to make a seamless transition to another monitoring stack for users, replacing Prometheus with VictoriaMetrics.

## Journaling: Vector + OpenSearch + Kibana

The following components are used in the journaling platform: 
* OpenSearch - as a database;
* vector - as a log collector;
* Kibana as a data display tool.

At the moment there are a number of problems with performance and resource consumption in the logging stack. Some of them were solved by changing the collector: fluentd to vector. For license reasons we also had to replace ElasticSearch with OpenSearch. For users, these changes were unnoticeable because users only have direct access to Kibana to query and view logs of their applications and integrations.

## Distributed tracing: Istio + Jaeger + Clickhouse

App.Farm uses the following components for tracing:

* Jaeger - as a distributed tracing system, including: jaeger-collector - for collecting traces and placing them in the database; jaeger-query - provides API for working with traces; jaeger-ui - WEB-interface for analyzing traces, which we modified to allow users to log in with their credentials and see traces of only their applications (to which they have access);

* Clickhouse - as a database for storing traces, allowing to execute SQL queries and build analytical queries to analyze traces. Grafana is used as WEB UI for viewing;

* Istio - as a Service Mesh, thanks to which spans are generated and sent to the collector automatically from sidecar, without the need to create spans in custom applications.

Initially, the platform used OpenSearch as a trace repository. But due to the development of the platform, a number of problems with high resource consumption, low performance and lack of ability to make flexible analytical queries over data appeared. We managed to make a seamless transition to another trace repository for users, replacing OpenSearch with Clickhouse.

## Authorization: KeyCloak + Istio + PostgreSQL 

Typically, implementing authentication and authorization requires a system that allows you to manage credentials and accesses. In addition, such a system must support popular authentication and authorization standards. There are several systems that provide such functionality, such as: Keycloak, Ory, Okta and ZITADEL.

But in the conditions of the bank requires additional features: integration with a single directory on LDAP, open source solution, Self-Hosted installation and a large and active community. Given these conditions, we chose Keycloak as the most suitable option for us, which is already, in fact, a standard for solving such problems and has a wide application in the community. 

The platform uses Keycloak as an authentication and authorization system, which has the following features among others:

* Single-Sign On support;
* Support OpenID Connect, OAuth 2.0, SAML 2.0;
* LDAP integration with Directory;
* Flexible administration console and powerful REST-API.

Keycloak uses Postgres as the database, which is an infrastructure component and deployed inside App.Farm. Istio is used as a Service Mesh so that authentication and RBAC authorization can be set up at the sidecar level using configuration. Without the need to implement JWT token validation and access control in custom application code.

## Software development: Gitlab + Nexus + PostgreSQL + Minio

The following components are used in the software development platform:

* Gitlab - as a system for software development including code repository, access management, task handling, code review tool, CI integration, etc. For its operation it uses DB: PostgreSQL - for data storage; Minio - for object storage; Redis - for caching and task queue storage.

* Gitlab CI - as a CI tool;

* Nexus - as an artifact repository.

## Integration: Istio + Calico + Kafka + Active MQ Artemis

Typically in large organizations, there are many business systems that need to be integrated with each other. One popular way to accomplish this is through an integration bus where everyone communicates. In the App.Farm platform, we supported backward compatibility with the integration bus that existed prior to the platform. Additional integration methods such as HTTP(S) 1.1 / 2, gRPC, Kafka, and ActiveMQ Artemis were also supported.

For observability and control of interactions, App.Farm utilizes:

* Calico - A tool for monitoring network interactions;

* Istio - Service Mesh, which thanks to sidecar, can encrypt HTTP traffic via mTLS, so that user applications communicate over HTTP, although in reality it will be HTTPS; and also thanks to Envoy-filters allows to inspect and log HTTP traffic.

## Platform Features
### Open Source technologies 

The platform is completely built on open source technologies and a free license that allows the use of such technologies in commercial software. Advantages: free, public code, it is clear what is going on, safe (open source code can be checked), there is no lock-in on the software vendor (vendor), we can make changes to the code, and the trend is towards open source software.

Since the platform is built on open source technologies, the products running in the platform should predominantly be opensource. This applies both to the business applications themselves and the infrastructure components used to make it work. Therefore, when onboarding new products in App.Farm, we give preference to open source systems and opensource infrastructure.

### Multitenancy

Multitenancy is an element of software architecture in which a single instance of an application serves multiple organizations, departments, or teams. In App.Farm, much of the infrastructure that serves the platform and is provided to users is multitenant.

Predominantly we choose multitenant applications for encapsulation in the platform, this leads us to:

* We save computational resources: in the realities of a bank, it is quite difficult to rapidly scale computational resources, you have to provide strong justifications, model sizing, and wait a long time for approval and delivery to obtain them; 

* It is easier to maintain and support one common application instance rather than multiple instances for each division or team;

* Re-use of common configuration and data;

* Ability to integrate tenants with each other.

Below is a list of App.Farm multi-tenant infrastructure serving all divisions of RSHB-Intech. These are applications that allow customization of isolation of individual consumer groups at the single instance level, and provide a flexible role model for access customization:

* Keycloak (rhylms/clients);

* Gitlab (groups/projects);

* Nexus (repositories/directories);

* Sonarqube (projects);

* ActiveMQ Artemis (addresses/ queues);

* Kubernetes (neumspaces);

* Logging stack (indexes);

* Monitoring Stack (indexes).

### GitOps + IaC + Flow

GitOps is a software development methodology based on the rule that “everything is code”. This means that no manual customization or configuration activities are used when deploying applications, everything must be described declaratively as code under the control of a version control system and applied using CI/CD tools. 

We follow this approach not only for custom applications, but also for the development of the platform itself, including its infrastructure components (IaC - infrastructure as code).

The App.Farm platform delivers as code:

* Application source code;

* Application configuration (environment variables, computational resources, metadata, scaling information, upgrade method, and more). The exception is secrets (logins/passwords/keys, etc.) - they are stored in a special Vault secrets repository and must be referenced in the source code;

* Documentation;

* Additional configuration in the form of files required by applications (e.g. XML schemas);

* Monitoring dashboards;

* Dependencies on other services (runtime bindings);

* Database usage information;

* Roles, endpoint access rules;

* Kafka brokers and topics;

* CI/CD Flow used by the application.

The platform provides ready-made CI/CD sets. We call them CI/CD Flow. They are plugged by users into their repository with a couple of lines like a plugin.

A complete Flow typically consists of the following steps:

* Validating the project against the platform requirements;

* Building the artifact and container from source code;

* Checking the source code against community accepted standards: looking for bugs or “bad” code;

* Running tests and reporting test results and the level of code coverage by tests;

* Security checks of source code, dependencies and container;

* Publishing the built container and/or artifact to the artifact registry;

* Deploying the application in one of the platform environments.

![image](https://github.com/user-attachments/assets/2725bdbf-5512-4fce-9cb6-7f9d70092634)

Since recently we have started to support specialized OCI-Flow, which allows us to run ready-made application containers on the platform. That is, we are ready to support any application packaged in an image and provided to us by a vendor or developer, formed according to our rules and standards.

When onboarding new products to the App.Farm platform, we give preference to those developments that utilize technologies supported by App.Farm. If a potential product uses a technology we don't support, we do a preliminary evaluation and planning of that technology, and only then can the developer continue onboarding the product.

We offer to move to App.Farm CI/CD and ready flow without the need to modify them to the specifics of the contractor, division or team that has developed there over time. That is, it is desirable to put applications into universal replicable flows, the rules for which have been formed by the global community, not by a single team.
Checking the source code against community accepted standards: looking for bugs or “bad” code;

Running tests and reporting test results and the level of code coverage by tests; Security checks of source code, dependencies and container;

![image](https://github.com/user-attachments/assets/79a208ac-4ff2-4323-be4f-d72db748b09b)

In the platform there are specifications for declarative description of various entities: Divisions and their Resource Pools, System, Service, Communication, Database Access, Kafka Clusters and Kafka Topics, Artemis Queues, etc. - are the APIs of the platform. All these entities are declared in git repositories, have history and access control - you can find out the state at any point in time. CI/CD is used to bring the state in the cluster to the declared state in the specification.

Thus, modification of platform entities, either from low-level components, by means of custom applications is not available in the platform. For example, services / pods, network accesses, roles and permissions. In addition, arbitrary code execution is not available, as the code must be pre-built and tested in the development loop and then reused in the product. Onboarding should pay attention to the compatibility of potential systems with this declarative development approach.

## Integration mechanisms

Integration on the platform is one of the core functionalities of App.Farm. 

Integration, in simple words, is the way applications or systems interact with each other. Often, systems that are deployed in App.Farm require interaction with: 

* Other business systems in the platform and in the bank's loop;

* Platform component infrastructures;

* Bank component infrastructures (e.g. databases);

* Services located on the Internet.

The following integration methods are supported in App.Farm:

* Cross-service integration within one system hosted in App.Farm: HTTPS (HTTP/1.1, HTTP/2, gRPC, WebSocket) and Kafka;

* Cross-system integration, if both systems are hosted in App.Farm: HTTPS (HTTP/1.1, HTTP/2, gRPC, WebSocket) and Kafka;

* Integration with services on the Internet: HTTPS (HTTP/1.1, HTTP/2, gRPC, WebSocket).

In terms of integration methods, these are: 

* Integration with enterprise infrastructure services outside of App.Farm: HTTPS (HTTP/1.1, HTTP/2, gRPC, WebSocket); SMTP; SMB;

* Integration with business systems outside of App.Farm: HTTPS (HTTP/1.1, HTTP/2, gRPC, WebSocket), Kafka and Integration Bus (Based on ActiveMQ Artemis).

A feature of integration interaction in App.Farm is a default ban on all network interactions, even within the same system. Simply put, you will not be able to access anything from your service. In order to open network access, a developer needs to declare an integration link in his project, in which he must specify where he wants to access and via which protocol.

Thanks to integration links, the platform knows the complete dependency graph of systems and services. This information is public and is displayed on the Platform Management Portal. In addition, the linking mechanism allows system owners to know who depends on their services and who uses them. If desired, system owners can prohibit linking to their systems by granting such permission after approval.

## Authentication and Authorization

The App.Farm platform uses a single SSO provider built on Keycloak for authentication and authorization of users and applications, I've talked about it before. Predominantly, OIDC is used as the authentication standard. Platform-based authentication system is used for authorization of developers, their applications, as well as users of their applications. Among other things, the RSHB ID system is built on the basis of the platform authorization system.

The App.Farm authorization system is integrated with the corporate directory via LDAP protocol, which stores information about users and groups. App.Farm is constantly synchronized with the directory, thanks to which it has up-to-date domain information.

When onboarding new products, you should pay attention to several things related to authentication and authorization. Prioritize products

* Using OIDC authorization that can reuse our platform authorization;

* using infrastructure that can be integrated with our OIDC platform authorization;

* willing to reuse platform mechanisms for role management and endpoint access. It is worth avoiding systems that use authorization methods and protocols that are unsupported in the platform.

## Application delivery format

Many of the products for RSHBs are still developed by contractors. However, the products they develop may be licensed and delivered to the client in different ways. For example, we have encountered the following delivery formats: product source code, built binary artifact, Docker image.

Depending on the delivery format, the App.Farm platform may have some leeway in dealing with such a product. For example, when a contractor delivers source code, the platform can:

* Fully analyze it for vulnerabilities;

* Analyze the quality of the code;

* Apply replicated flow to build and run;

* Guarantee the quality of the built application as it is built by the platform;

* Ability to improve the product independently (if the license allows), for example, to fix an emergency bug before the contractor does it.

When onboarding new products at App.Farm, we give preference to products that are delivered as source code and licenses that allow us to work with and modify such source code. In the event that source code delivery is not possible, the next highest priority is delivery in the form of a binary artifact. For example, for Java applications it can be a set of jar files. If the binary artifact is not supplied by the contractor, then the option remains in the form of OCI-image delivery, but in compliance with our requirements on formation and rules of image/application preparation and operation.

## Bottom line, opportunities and future plans

With the introduction of App.Farm, the development and support stack has become more rigorous, there is no longer a zoo of technologies. Automation and the transition of the entire bank to unified CI/CD approaches has made it possible to make cross-functional teams that know that when you move from one team to another you can just start writing code.

The software development cost has been reduced by lowering the threshold for entry into the development technology stack. Currently in the system: 2,300 users (CI/CD), 197 systems, 3,300 integration flows. PaaS platform App.Farm provides 6.5 million events per hour or more, and supports more than 12 integration types. No Vendor Lock-in. External contractors do not participate in development.

Now, thanks to the versatility and convenience of the platform, 612 microservices, 50 internal and 147 external systems are served in App.Farm:

* 612 microservices, 50 internal and 147 external systems are served;

* 2,296 connections and 2,800 integration requests are processed per second.

In addition, 21 vendor products have been adapted to App.Farm for centralized work with the solution and 290 external integrations have been performed through OpenAPI functionality for 30 customers.

We have implementation plans in the near future:

* S3 as a Service;

* Redis as a Service;

* PostgreSQL as a Service;

* Deployment capabilities development on the platform: support for Stateful applications, App Review, Canary / Blue-Green Deployment.
