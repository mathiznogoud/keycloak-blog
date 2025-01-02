# How we created the PaaS platform App.Farm - the digital heart of RSHB

Hi! My name is Konstantin Belkin, I am Teamlead SRE at RSHB-Intech. Today I will tell you what CI/CD on the App.Farm platform is, what methodologies we use in our work, how the platform works, what tools we provide to developers and how we organized CI/CD in RSHB for our favorite developers.
This article is based on a talk I gave at the RSHB Meetup: Think DevOps in a Big Company, held on August 29 at RSHB-Intech.

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

