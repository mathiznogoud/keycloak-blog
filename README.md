
Keycloak for us is the central system for authorization and authentication of all external and internal users. We have “made friends” with other systems and services, raced it with different loads, and we want to share our overall impression.

Here is a story about how everything is organized under the hood, as well as the pros and cons of working with Keycloak, which we have noticed over the past two years.

My name is Konstantin Belkin, and I am a Teamlead SRE at RSHB-Intech. This post is a reworked version of my report from the RSHB Identity Management Meetup we held in February. If you want details on that event or a video, you can find it all at [link](https://rshbdigital.ru/meetup-identity-management).

## Authorization challenges to solve in the bank

Let's run through the list of problems and challenges associated with authorization and authentication in large banks. I would highlight six main ones.

**1. Decentralization problem**. There are many software products in a bank, and each has its own layer of authorization and authentication. Some store data side by side in the database, others log in via LDAP to Active Dirtectory, others somehow else. This all needs to be unified.

**2. User data storage**. The main question is how to store data correctly and how to transfer it.

**3. Encryption**. Somewhere LDAPS is used, somewhere unprotected LDAP is used. This problem should be solved too.

**4. Cross-service integration**. In order to “make friends” with two different systems, it is necessary to write an integration adapter, which requires financial expenses. This problem will arise again and again as new systems are added to the asset.

**5. user lockout**. If one of the systems does not exchange data with central systems (we use IDM, Active Directory in our bank), it will not be able to get information that a certain user should be disabled when he went on vacation or quit. Previously, such systems had separate administrators who had to be contacted personally.

**6. Integration Costs**. In the absence of a single point of exchange, costs would go up because each system had to integrate with other related systems. And for each case we had to write integration adapters and place them somewhere.

Translated with DeepL.com (free version)

## How can Keycloak help?

With Keycloak, there is a single point of authorization and authentication: for bank users, bank customers, bank systems, and bank services across multiple systems. 

There are unified security and analytics standards, as well as a single dashboard for security officers to connect to their systems to retrieve data and track emergencies, such as multiple logins to the same account.

With Keycloak, security policies are standardized for data validation and analysis. Plus, there's a tool to grant permissions to the entire layer of systems for bank administrators. With this, an administrator of one system can quickly replace the administrator of another system. He or she will have the same interface and data set, but will be served by a different role model.

Now a few words about our App.Farm banking platform and how the authentication and authorization center is connected to it.

Translated with DeepL.com (free version)

## How our banking platform App.Farm works

App.Farm is a banking environment that follows PaaS concepts. It has a continuous deployment pipeline, integration systems, access control and a public app storefront.

App.Farm's platform architecture overlaps well with the bank's organizational structure and how we approach banking product development. 

The basis of RSHB-Intech's organizational structure is Blocks. Blocks are large divisions within a company that have departments called Centers of Competence (CoCs). The CCs, in turn, manage specific banking systems.

We are recreating a similar structure within our platform. In addition to it, there are subsystems that store resources and role models for each system. Systems outside of App.Farm are also allowed to authorize with us and identify their users. As a result, we end up with a single authorization layer for a user's walkthrough across all systems.

For example, an employee comes to work and logs into system A, then he can walk through all systems. He does not need to reauthenticate in the browser to log in to system B or C, provided that such behavior is provided by his role.

The platform architecture is built on Kubernetes because the technology provides the best experience of fault tolerance and scalability. If we are short of capacity, we can scale our Keycloak by adding replicas or processors for those replicas.

We started building the platform with Keycloak about two years ago. It initially combined three projects. The first was an integration platform, where systems come together and share data. The second was CI/CD, because we needed to deliver platform adapters and products we develop there, and also let other products in our development cycle deliver software to this platform. We ended up with the CI/CD project as a service.

Then came the third project, OpenAPI, where we open up our APIs to external components. For example, Banki.ru wants to know where our ATMs are. They need to regularly connect via API and take this information. Another example is connecting to the Federal Tax Service and exchanging data.

And then the SSO project came to us, and we had this Voltron based on entities that we combined with each other.


![[Pasted image 20241231154516.png]]

The foundation of the platform is declarative and open. The platform was built on the Infrastructure as Code model. We describe everything in GitLab, which is the base for CI/CD. We always describe everything declaratively, and based on the location of the repository (Block-CC-System) a special project is created, which is called “information system” - abbreviated isys.

```
isys:  
	name: "Электронные площадки"  
	description: "Информационная система для доступа к Электронным площадкам"  
	owner: "ЦК Электронных площадок"  
	cpuLimits: "52"  
	cpuRequests: "32"  
	ramLimits: "70656Mi"  
	ramRequests: "39936Mi"  
	roles:    
	. - code: "cbr-open-api"      
		description: "Роль для использования сервисов ЦБР OpenAPI"    
	  - code: "esia-user"      
		description: "Роль для использования сервиса авторизации через ЕСИА"
```

The isys project has a name, a brief description, consumed or allowed resources. And here the role model is described declaratively, and then it is picked up by a self-described entity - a Kubernetes operator.

Our entire platform is built on Kubernetes operators, of which there are now four. One of them is called auth-operator, it creates and manages Keycloak entities. The platform-operator thinks of a role manifest (in the illustration above), will go to Keycloak and create a client of that information system. Then assign it the necessary roles so that those roles can then be managed, and the system administrator can assign them to some users or other systems for login.
## Keycloak Client Features Overview

I'll start with public clients. They authenticate, authorize, define a role, and provide an interface defined for that role. For example, if a user is assigned the admin role, the interface will be admin. All of this can be described in a program based on the Keycloak library, which comes for almost every language. 

The only disadvantage of a public client is the lack of the ability to access other services or systems, because a public account does not have a service account, which we will talk about separately.

The other type is confidential clients. They have the same set of data that we have seen before. Their roles and tasks include creating service accounts. This is an optional feature, but we always use it.

Let's take a look at the diagram.

![[Pasted image 20241231154555.png]]

There's a data source here, Active Directory, where Keycloak gets its information from. You can manage different types and mappers and get a really rich data set. An internal user can authenticate to both internal systems that are inside the platform and external systems. On top of that, we have a DMZ layer where we have a second Keycloak for external users, our customers. 

Now for more details on all of this.

## User patterns: internal user (bank employee)

![[Pasted image 20241231154647.png]]

In this case, we get information about the user strictly from Active Directory, which is our information center. At one time, the security service restricted us from adding new users “by hand”, so we pull all the information from AD, and new users are created only there.  
  
We all realize that we all need technical users for interactions, we solve this problem through Platform-Operator. It creates a client and enables the service-account feature, which you then use for your system.

For each service and each system, we have a service account. In addition, we have the ability to integrate users with our internal bank Identity Manager. This is necessary so that the manager can manage the roles of his subordinate. Based on this data, we can assign the roles created in Keycloak to a user who is a member of a particular group. Accordingly, to get status information, we constantly synchronize with Active Directory. Keycloak has several synchronization patterns that you can use. One of them is the time pattern, which we use for synchronization every three hours. And we do a full synchronization once a week.

## Custom Patterns: Bank Client

![[Pasted image 20241231154813.png]]

In this case, Keycloak will be a unifying center for storing client accounts. We can safely store quite large amounts of anonymized data: phone numbers, emails. Information about users is contained in the internal CDI banking system, where each user is assigned a unique identifier. Based on the phone number and e-mail, we can always pull up the user's full name from the CDI. In this way, the law on personal data is not violated, because the phone number and e-mail do not fall under it, because there is no comparison with the full name.

The personal identifier is enough to tell another system that it is this user. As a result, we provide a single point of entry for all customers through a single realm layer. If a bank customer has entered one of our platforms, for example, “[Svoje Selo](https://svoe-selo.ru/)” or “[Svoje Rodnoe](https://svoe-rodnoe.ru/)”, wants to buy products there, then afterwards he can safely go through the whole layer of all Svoje ecosystems.

Now I'll tell you about the platform and Keycloak and how things work there.

## Client Lifecycle: Internal System Client (ISYS)

These are the clients that carry role information. They will always be public clients if the system has an entry point, and can be private if it's a layer of some API endpoints that it provides to other systems. 

A client is able to define a set of user data that will be provided when a user authorizes with a given system. The client also carries customized token lifetime settings. We can set even 15 minutes and force the user to reauthenticate after the next session.

## Client Lifecycle: Internal Service Client (PSVC)

The next type of client is the Platform Service Client (PSVC). It is used for authorization in other systems, i.e. it has a service account. Roles are given to the service-account, so it can address some other service to pass authorization and authentication and get some data from it. 

External system client (EXSYS) are systems that are outside of our platform. They have the same set of roles as ISYS.

![[Pasted image 20241231155007.png]]

The platform service that resides in the system can seamlessly connect to other information systems or communicate with external information systems to exchange data. All of this allows you to have Keycloak in a single realm layer.

## Service Accounts, and how we prepare them

Operator manages the creation of role entities in Keycloak. Based on this, we have platform service clients created and used for authorization. The client has the name of the account itself and its secret. It works like this: get the token, go, get the necessary data. We assign roles to the service account to introduce ourselves to the services and get data from them. 

To external off-platform services, we provide the same set of functions and also declare under the IAC model. We have consumers in the bank that use this model. Some of them are internal systems that have an external layer. They go as public - they are “Own Farming”, “Own Business”.

In addition, we at k8s use ISTIO technology, which is a favorite of developers. During development, ISTIO allows you to avoid connecting to an authentication center and not waste time embedding security components like spring-security. And also not to configure it to work with the authorization center, OIDC-client. You just debug the application and write a manifest like the example below, that you have protected the endpoint with so-and-so role, method so-and-so.

```
psvc:
  name: “SAR Screening: bff”
  description: “SAR Screening: Backend for frontend”
  roleMapping:
    - endpoint: /api/baseDictionaries/search/getByBaseDictionaryTypeCodeAndCode
      roles: admin,developer,auditor-sva,user-dkk2,user-dkk1,user-controller-dkk
      methods: GET
      zones: external
    - endpoint: /api/baseDictionaries/search/getByBaseDictionaryTypeCodeAndCode/*
      roles: admin,developer,auditor-sva,user-dkk2,user-dkk1,user-controller-dkk
      methods: GET
      zones: external
```

The arriving user gets to ISTIO Sidecar. If he is not authenticated and authorized, he cannot go anywhere. Thus a developer can make systems without debugging the security subsystem internally, shifting this function to ISTIO, which reduces the time to market.

## External SSO and why we needed it

The SSO module for external users is used for end-to-end transitions, so that the user does not authenticate in ten different systems, but logs in once and goes through the whole layer, as well as for using two-factor authorization.

Within external SSO, we have a project that has emerged called RSHB ID. It is similar to Sber ID or Tinkoff ID, which allow a bank client to authenticate on some other platforms. For example, on mos.ru at the expense of the data that our SSO provider will pass on to him.

The general scheme with external SSO looks like this:

![[Pasted image 20241231155221.png]]

It includes a number of external client services that have appeared and a large ecosystem called “Svoje”. There is a rather complex algorithm of work here, based on YAML manifests that arrive in Kubernetes and are processed by the operator. Based on them, entities are created both internally and externally.

Identical to the internal SSO we have an external entities operator that manages the creation of Keycloak clients based on the declarative model. The operator also manages roles in the external Keycloak and in the case of public system clients we have another entity called PUBSYS. This entity carries a role model that can be imposed on users, so that the user authorizes and authenticates under some specific credentials.

There's also a separate ISTIO-based network accessibility management mechanism for external clients here. It stands up front as a gateway and authenticates clients if needed to pass through to the bank's internal network.

Keycloak imposes many constraints on operators. The system is not flexible and additional modules have to be written as part of the enterprise. We have written two plug-ins that are used inside Keycloak. This is an integration module for pulling user IDs from the CDI system and combining them with phone numbers to get matches. And a two-factor authentication module with our separate SMS center located in the processing center and serving bank customers.

## Pros of Keycloak

- It is opensource with a large community. Keycloak is the industry standard at the moment. If there is a task to make an authorization center for your application, Keycloak is a great choice. It has an advanced API, which allows you to refine and customize it to your needs. 
    
- With Keycloak you get a single authorization and authentication center for different systems, where there is support for commonly accepted models to choose from: OIDC, SAML, OAUTH.
    
- It also features federation. This means you can “make an agreement” to exchange users with a large number of systems, ranging from Active Directory to LDAP.
    
- Keycloak can mediate the transfer of data. This is especially true for the task of migrating from one LDAP directory to another because of the requirement to abandon Active Directory and migrate to some opensource like FreeIPA. Keycloak will take the necessary data and transfer it to another system.
    
- Another plus is the speed of processing. It can handle millions of users simultaneously. Any problems with load may start after a million, but up to 100 or 500 thousand simultaneous users - no problems at all. 
    

To test the speed, we created a script that logged in 20 times per second on the first run and 40 times per second on the second run. In the end, the time lag between these two runs was 8 seconds after a four-hour experiment. This is an excellent performance.

The largest real load we have is 50 logins per second. The system runs on three instances, each with two CPUs and 4GB of memory. Plus, the database is on Postgres. At the same time, the load on Postgres is very low, which is due to the memory cache that Keycloak has.

## Minuses of Keycloak

- The first disadvantage is that it is opensource. 
    
- It will require improvements if you have been sitting on LDAP and plan to switch to some other authorization model.
    
- Enterprise always requires improvements and API extension, as there are tasks of integration into existing systems.
    
- Another disadvantage in terms of enhancements is the lack of full-fledged API documentation. You will have to read a lot of code to refine something. You will have to go through similar plugins, study their code and create your own based on it.
    
- Hence the next disadvantage - you will always have to get into the code.
    
- There are places in Keycloak where you can press something and everything will break. We had such a situation, we had to restore the database. You can accidentally delete some unifying role, and you will have to build everything from scratch.
    
- As a consequence, you always need a test zone to conduct experiments. This takes additional resources.
    

## FAQ

Before I summarize, I'll post here some answers to some questions from the mitap that may be helpful to colleagues looking at Keycloak.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/085/1b0/7bd/0851b07bde4d4f65bec545779a09fb3a.jpeg)

- **Is it reasonable to use Keycloak for small systems?

It doesn't make sense for a system with one user, but when you have a dozen users, you can try it. You can spend more time with your own authorization than with Keycloak. Thanks to the excellent user documentation, Keycloak can be set up in a day and will work fine.

- How secure is it to use an external product for authentication? How fast can vulnerabilities be fixed?

In opensource, you can close vulnerabilities yourself. In addition Keycloak is supported by Red Hat and they fix things quickly.

- Do we encrypt jwt-tokens that are walking around on the system?

Yes. Out of the box, Keycloak provides several encryption algorithms that we use for internal and external SSO.

- **Suppose a user logs in to one application with a username and password, a second with a phone number and SMS, and a third with a password and two-factor authorization, will these be different users?

One. It's just that each client will have a different set of client credentials configured for login.

## Brief conclusions

There are a lot of positives here, so I'll just run through the highlights. Keycloak can be a reliable entry system for all types of users, whether they are bank customers or internal employees. It can solve integration problems and become a reliable center for providing access and data exchange between different systems. Keycloak can become a single point of entry for all your software products. But, this is all subject to the caveats I mentioned in the cons section.