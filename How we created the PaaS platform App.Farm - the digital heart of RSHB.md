![image](https://github.com/user-attachments/assets/716b0ea6-c962-4937-aa1f-2b415a024a99)# How we created the PaaS platform App.Farm - the digital heart of RSHB

### Intro
Hi! My name is Konstantin Belkin, I am Teamlead SRE at RSHB-Intech. Today I will tell you what CI/CD on the App.Farm platform is, what methodologies we use in our work, how the platform works, what tools we provide to developers and how we organized CI/CD in RSHB for our favorite developers.
This article is based on a talk I gave at the RSHB Meetup: Think DevOps in a Big Company, held on August 29 at RSHB-Intech.


### Main
Now all companies are building platforms, small or large, that are filled with their products. Platforms make life easier for developers. The goal of our App.Farm is similar - to simplify life and stimulate internal development. At the time of the project's inception, RSHB was mired in vendors, and this needed to be rectified. One of the right ways out of the situation is to give developers simple and clear tools to write business logic and deploy it somewhere in the form of a product.

The basic principles on which CI/CD is built are GitOps, IaC, and CI/CD.

**GitOps** is a methodology that assumes everything is code. If everything is code, then we can describe it all in code and upload it to a toolkit that will process it all and bring it to a productive loop or development zone.

This is what an example typical route looks like.

![c6b25e42cb692dbab8f7b6e639f71fc7](https://github.com/user-attachments/assets/0c4c5a7f-26ff-4008-bafb-204920f14bd0)

A developer writes code, throws it into Git, something is built there, an image is built, there are some infra-configs that are also built and get into Git. We also have GitOps roundtrips that eventually dump it all onto one or more clusters. We have one cluster used for application application.

If we talk about the IaC approach, it asks you to fully describe your infrastructure as code, specify server addresses, what tools to put, what roles to cast, and so on. Accordingly, the code for infrastructure management can be written in different styles: declarative or imperative.

We use the declarative approach. It can also be used when developing an application or deploying infrastructure. 

The third approach we use is CI/CD Flow. We don't supply developers with giant YAMLs or have a team in every department outlining their CI/CD process. We thought it would be easier to center all the CI/CD competence around a platform team and deliver 3 lines. If a developer wants to write in Java or a vendor wants to drive into the platform, they can prescribe three lines, everything will drive in and work.

Our typical pipeline looks like this. 

![image](https://github.com/user-attachments/assets/0d932791-16a9-4c27-9739-ca08a5139635)

At the beginning, we do verification. Then, accordingly, it goes through building, tests, checks, checks, DevSecOps, publishing and, finally, everything goes into deploy. All of this is owed to a lot of additional tooling. 

In the AppFarm platform, we deliver the following tools as code. 

![image](https://github.com/user-attachments/assets/9ba5bee9-a682-46af-80aa-f1f5418eaa61)

Platform documentation is a tool for knowledge sharing. This is probably the most important thing in CI/CD approaches in general. You have to tell developers or other devops how to use your toolkit in general. Flow is also created with code. 

There are basic data that we use in the platform. This is the source code of the platform itself and the codebase, the configuration for launching this application, additional files that are needed to make the application run at all. And there are additional pieces that we describe as code as well: kafka cluster, links, connections, interconnections between each other. Role model and monitoring dashboards for the service - if needed and the database. We store the secret separately in Vault.

So, here you can see the table, which flows we support at the moment.

![image](https://github.com/user-attachments/assets/584c26f6-f153-4418-9e4f-dfb3848c236a)

We have java as the most popular language in the bank. Almost 30% of applications are written in it. That's why we implemented its support in the first place. And, of course, we did the same for frontend applications for JS + TS. The last feature we implemented is support for direct deployment of docker containers from vendors. Many now ship their applications in docker format, and we can safely accept them and work with them. We have made a specialized pipelines for this purpose.

As I have already said, we use a declarative approach and describe the end result with it. And the further our underlying toolkit will lead it to the actual result that we will get in some form.

What a declarative service description looks like. 

![image](https://github.com/user-attachments/assets/b7759e27-22e4-47bb-9ca4-cc799bba9a4e)

We came up with our own manifestation, meaning we have everything end up in kubernetes. But there is a special handler that handles short manifests. That is, the developer does not need to know the kubernetes specification, how to write deployment, the description of the service entity, how to compose some virtualService, write NetworkSet and NetworkPolicy, and so on. He can write 16 lines of code according to the documentation, parameterize his service, pass some variables that are needed to run it, and work with it.

In addition, we generate various additional entities. Previously, in order to formalize, for example, a kafka cluster, you had to go to the system administration department, order a server, coordinate accesses, get access to the server itself, and deploy Kafka there. With the declarative approach, things have gotten easier. You just write a couple of lines and say that I need a Kafka cluster for an information system. After the operator goes to the resource pool, takes these resources from your resource pool and deploys you a ready Kafka cluster for your work. That's it, you can plug and play with tops from there.

Here is an example of how declarativity looks like in our system.

![image](https://github.com/user-attachments/assets/dcb8c960-2492-4320-a2d4-404bd1a54567)

We achieve the declarative approach with a special platform-operator that we wrote. It serves special K8S entities. If you create a small description that's eight lines long, bring it into a kuber cluster using the deploy tool, the platform operator will come along, subtract that value from the custom resource, and spread it across two entities. It will create a namespace and create a resource quota on the values that are described here.

There's another example.

![image](https://github.com/user-attachments/assets/aadb73ed-648f-4f17-b769-70be4564d83f)

The developer declares the link, says I will go from the `frontend_for_asp` service to the `asp_net_service`, which is also located here, sheds the pipelines, platform entities are hit accordingly, the platform service entity, the deployment entity and the custom link entity appear. 

Platform operator, says:

- Aha a link has appeared.
- What do you need to do?
- We need to make Virtual-service.

So I went and based on the short essence of these six lines I created a large manifest and rolled it out. Thus providing access from the frontend to the backend.

We do not have any dynamic creation of entities in our platform. That is, you can't hand-create something yourself in kubernetes. You always have to describe everything declaratively using the language model that we offer you, and based on that, you get network accesses, roles, permissions, and so on. Everything should be described, it gives us a sense of security. You always know where your service should go and where it shouldn't go. There is no such thing as not understanding at all what the service is interacting with.

## Toolkit

Our toolkit has several components: where we store the code, how we deploy all the tools, where we store our artifacts. When we started the project, there was a choice of five solutions on the market - GitHub, GitLab, Gitea, Bitbucket, Gogs. Now most of them have left the market, but open-source ones remain. Next, we wanted to choose some kind of CI system. We had quite a big choice here, something we knew, something we didn't. In the end, our choice fell on GitLab, GitLab CI, and Nexus as a repository.

![image](https://github.com/user-attachments/assets/8c37b37f-63c5-4db0-b570-95ab9a462520)

Why did we choose them? Because we have worked with them before, and many of you have probably worked with them. There is a large base of experience, and they are generally good quality products that have been on the market for many years.

The final composition is as follows: GitLab, which has Crunchy Postgres under the hood, Minio, which is the object storage for GitLab, GitLab CI as a CI tool, and Nexus.

The benefits of Open Source are clear and clear for everyone. We can work with it, we can change it, we know that it is safe, it is easy to check, and we are not vendor-locked to the software vendor. We can always switch to some other open-source products. It's free, but of course that's a moot point. You always pay for it with human resource. The trend now is to use open-source software in Russia, so this is a good thing.

In order to recreate the whole CI/CD system, apart from some global tools that we use, we need to somehow make life easier for developers and improve the security of our CI/CD pipelines. We've written a few tools that we need to get the job done. Let's talk about each of them.

So, the Verifier verifier. It is needed to guide the developer to the right path. It may happen that a developer decides to recreate something of his own in the project outside the documentation. The Verifier will statically check if he has filled in everything correctly to build his project. That is, at the first stage we launch the project into the verifier, check it and can stop it in advance before errors appear, before it is built there. 

After that it is the turn of DockerfileGen. Many developers can write docker files, but we decided that we shouldn't let them do that. So we choose for the developer which docker file to use. We have DockerfileGen for this purpose. This is very useful, also from a security point of view. You can simply take and, in case of some failures in the common pipelines, or rather in the common images, rebuild the image to get rid of some other problems. You don't have to run around all projects like DevSecOps. 

We also have a product called Signer. It deals with re-layering into the product, creates a key on one side, puts it into the repo and when re-laying it checks that the image is really the one that goes to the product.

**Buildjit-Journey** is our buildjit product. It came to replace Kaniko.

Kube-deploy-apps is our toolkit that allows us to deploy something into the platform, that is, it describes YAML manifestos, templating happens, and it also generates entities, deployments, services, and everything else. 

BASE, BRICKS, JOBS

Let's talk about how we build CI/CD pipelines. We have chosen a rather non-trivial approach. Large manifests and multi-page books are not very good, they are hard to work with. It turned out that there is such a principle - BRICKS. It underlies the division by functional features. We can divide a YAML manifest into short manifests that will each carry some sort of job. This way we can build different pipelines for different languages from different pieces, reusing the same pieces in different pipelines. If you want, you can read on GitLab about this and how you can optimize your YAML manifests in general. Here's a link to the [GitLab doc](https://docs.gitlab.com/ee/ci/yaml/yaml_optimization.html)

So what the structure of our project looks like, which is used to make one small include on three lines.

![image](https://github.com/user-attachments/assets/ff1c59da-3721-48f4-8377-bced8ed6d9ea)

At the beginning are the bricks. These are just the pieces that we can reuse. They are some deploy-functions, build-functions, test-functions and so on. Verification is also included there. Next, base is the foundation, something without which the Pipeline will definitely not work. It already includes some basic components, images, their versions, variables for the work of the Pipeline, some deploy-functions that are definitely needed for the work, build-components, test-components and everything else.

From these first two pieces, bricks and base, we create flow. Within flow there are several types: CDL (delivery flow), CDP (publishing flow), and when we just need CDP without everything else. This is typically used for vendors. We just need to deploy a service from some jarka, and run the jarka in a container and that's it. It's actually a rare case, but it's used. At the end we get compiled langs for languages, i.e. for Java, for Go, Dotnet and so on. If you only want to deliver to a product, then use CDL. If you want to publish to a platform, use CDP.

What a pluggable YAML manifest looks like in the end. 

![image](https://github.com/user-attachments/assets/e8345f1c-7033-4203-abcf-4e51eb4fe623)

Behind the three lines there is such a set of stages and features, which in the end turn into such a wall of jobs. 

![image](https://github.com/user-attachments/assets/de6b7b1f-7d9b-4bb4-b8a3-6c29e6fb2e2e)

Here you can see that some checks failed, but in general the project came together and even went into production.

The connection of the final manifest, as I showed before, looks exactly like this, in the form of three lines, where we specify just flow, that this is publishing, that this is a service written in Java.

![image](https://github.com/user-attachments/assets/f3079eaf-5183-418a-8ad4-cbefabfdd6d4)

What advantages there are here in general. Of course, it is user-friendliness, i.e. in the documentation part we conditionally describe - plug in three lines, and everything will be cool. You don't need to write a bunch of pipelines, some additional manifestos, or anything else. This is, respectively, minimization of fields. We always know that there should be three lines in a project and nothing else. That is, if someone adds some more logic, and we also have such figures, it is bad. We say, take it away, we don't need it.

And in the end, it is not code, it is a reference to code, so there is a giant YAML of 5000 lines behind it.

## Problems

There were a lot of problems, but I'll take out two fundamental ones that caused the “communal” pipelines to stand up indefinitely.

The first problem we encountered was the problem of cache entanglement. Remember when I talked about Kaniko? It spawned this problem for us, and that's how Buildjit-Journey came about. We used Kaniko to speed up builds because it caches the cache images, puts them in Nexus, and based on the caches and some other things, it takes the caches and reuses them in future builds, roughly speaking.

Eventually, what started happening. Kaniko went crazy and started to put pieces of Angular and more into JVM-projects. It all started to turn into a mess. The product could have a built container with pieces from Java, pieces from Angular, and it's not clear how it all worked. 

We thought, let's do something about it and think about how to move on. It was 2021, and we were looking at what was trending. We saw such a ready-made solution as BuildKit from Moby, the creator of Docker. We looked at it, and in principle, it works quite well. We found a very good article by a Japanese researcher and read it. It turned out that BuildKit works much faster than Kaniko - by 15-30%. We decided to take a stab at this story, but we had to flexibly throw out Kaniko and bring in BuildKit.

Kaniko has entrypoint logic for launching. But BuildKit does not. Accordingly, to build BuildKit and have it start working, we had to write a wrapper. This is a wrapper over BuildKit and BuildKitD, so that it can appear as both a client and a daemon at the same time. Why it's needed. We use Istio, we have MTLS everywhere, and it was necessary to get up with certificates in the section for all builds.

Accordingly, plugged in eventually as a Kaniko replacement and got pretty fast pipelines. We have an average pipeline, if just for a developer, is built in 3-4 minutes if it is the first build, further builds are 2 minutes each. (Not during rush hours, where the build stretches from 6 to 10 minutes).

There was a second problem. We claim to write code correctly, so we introduced SonarQube. We even have this topic checked at the top, how many checks were made. That's why we use SonarQube and static checks in it. The average checkout time in SonarQube takes 7 seconds (in an average statistical microservice). We check 25 different technologies that are implemented there, they are JSON, SQL, Java projects and so on.

At one point we started to have terrible build slowdowns because of SonarQube. The average time per checkout increased to three minutes. Since we use Community LTS-version, we had only one developer. Imagine, a bunch of developers developing, and they all get in line. Literally within 20 minutes a queue was being created 16 hours ahead of time.

Looking for the problem, we rolled back the latest release of SonarQube and decided to throw resources at it. And this part was our fatal mistake too. As it turned out, SonarQube has a very interesting feature - the more CPU you give it, the slower it will run. It's like it's getting fat, screaming that it can't move anymore and begging for help. Eventually our pipelines started to slow down because of this.

It was a problem that we hadn't fully investigated. It might have been at the linux kernel level, because we were using an old version of Linux 3.12. But the moral of the story is that you should roll everything consistently, resources roll separately, upgrades roll separately. And always pay attention to all factors and watch for changes in diff.

## Business benefit

The primary means of achieving a positive outcome, when building “utility” pipelines, is documentation. We have to make it very clear to the developer how they should run their project, how the platform works, what manifestos can be applied to the work. That's why we have implemented a quality product as documentation called Docsify.

This is what our platform guide looks like. This is the title page, there's actually a whole bunch of text. You could publish a book of 500 pages.

![image](https://github.com/user-attachments/assets/44ff75db-0060-4e3d-b920-f69c0876b647)

What Docsify represents. It is a technology stack based on Docsify. A good technology to structure information with Markdown. Techpis to write documentation on Docsify, it is enough to learn Markdown. We connected Commento engine to it to allow to leave comments to improve the documentation. Such a doc can be improved by users via Merge-request. [Here is a link to the Docsify project](https://docsify.js.org/#/), you can take it.

### What are the benefits of implementing a unified CI/CD approach.

Now you don't need devops specialists in every department. Usually there's always some devops person sold to departments. We have one support department for the whole platform. The whole bank works in the platform. Accordingly, we have unified everything down to one service, one development and single standards. We know everything, we see everything, we monitor everything.

A developer's transition from one project to another takes a minimum of time. He used to use CI/CD technology in one department, he moved to another - it's the same here, everything is good and familiar.

We have reduced the technology zoo. We have fixed databases we're ready to connect, brokers we're ready to serve. There is a fixed message exchange service across the platform. Accordingly, the involvement of new developers also takes a minimum of time. You can work with the documentation very quickly and see what you need, there is a cool search. 

**Developers focus on business logic, not infrastructure.** They don't write docker files and think about how to deploy Kafka, plug in Zookeeper, deploy IBM MQ or whatever. It's all deployed based on manifests that you just write three lines into and you're all set.

Lastly, let's talk about the numbers. We have **2,300** developers working on the platform, a total of **3,620 users**, if we add management, system admins, and so on. There are **5,518** projects in development right now, **1.123** million Pipelines have been launched over the entire time and **23,689** support requests have been made. **273,697** MRs and **1.598 million** commits have been made.

There's a little bit of graphs here. You can see from them that we are constantly in growth, literally every day.

![image](https://github.com/user-attachments/assets/f7002a05-0e24-4b88-8e15-1960855d9bed)
![image](https://github.com/user-attachments/assets/ce606a0a-ccc5-4ff1-b815-dbb645219e26)
![image](https://github.com/user-attachments/assets/d1c1d9e0-b12b-42e2-8778-72d1e20c43af)
![image](https://github.com/user-attachments/assets/2e88ddf6-0a4d-415e-9415-ede73279a9d6)

This is in general terms. You can read a more detailed basic description of the technical component of App.Farm in the previous article.
