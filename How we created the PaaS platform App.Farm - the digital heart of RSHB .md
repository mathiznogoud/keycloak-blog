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

    Container startup and management, orchestration;

    Application isolation and compute resource management;

    Service discovery;

    Traffic balancing;

    Application scaling;

    Various upgrade and rollback strategies.

And also provide features such as:

    Traffic introspection and monitoring of network interactions;

    Two-way graph encryption;

    Service Grid - the ability to get information about all applications and their interactions;

    Integration with other infrastructure components: tracing, logging, monitoring;

    Authentication and authorization of services.

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
