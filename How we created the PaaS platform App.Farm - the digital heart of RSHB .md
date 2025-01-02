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
