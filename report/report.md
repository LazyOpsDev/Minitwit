# DevOps, Software Evolution & Software Maintenance
 
#### Course code: KSDSESM1KU
 
### May 2020
 
**EXAM ASSIGNMENT BY**

 
|   Student    |    Email    |
| :----------: | :---------: |
| Marek Kisiel | maki@itu.dk |
| Alexander Banks | alsb@itu.dk |
| Philip Korsholm | phko@itu.dk |
| Krzysztof Abram | krza@itu.dk |
| Arian Sajina | arca@itu.dk |

 
<div style="page-break-after: always;"></div>

## Minitwit
### Group E

## Introduction

Minitwit is a twitter-like application which has been developed and maintained over the duration of the course “DevOps, Software Evolution and Software Maintenance”, spring 2020. This report will elaborate on the design and architecture of the system we built, as well the processes and DevOps practices we have applied, to support the project. 

Our Minitwit application is containerized using Docker Swarm, hosted on DigitalOcean. The the tech stack running in our swarm is the Flutter for Web, ASP.NET C#, and MySQL. We have used build and deployment pipelines running on Travis-CI, Azure DevOps and CircleCI to support continuous integration and continuous deployment. Git has been used for version control of our code base. GitHub has been used as our software development platform, and to support our workflow with its integrated KanBan board, which allowed us to delegate issues among ourselves. 


## Architecture & Design

This section seeks to provide an overview of the architecture and design of the Minitwit application. The architecture of our Minitwit implementation is based on typical client-server architecture with our backend and frontend running on different physical machines. We will describe the design and architecture of the application using the 3+1 approach. We also outline the dependencies of the system’s functional core, as well as the tech stack the application is running on.


### 3+1 Viewpoints

This section outlines the architecture of our Minitwit system using the 3+1 approach


#### Module View Point

![alt_text](images/module_vp.png "image_tooltip")


In the above figure, we have created a package diagram of our backend. In the Data Access Layer package, we have a CustomDbContext module which we have modelled as a class. We decided to add CustomDBContexts fields to this package diagram, as its class name is rather ambiguous on its own. Beneath is a list with brief descriptions of each backend package’s purpose and responsibility. The arrows indicate dependencies between modules.

**Repository**

We are using the repository pattern to encapsulate logic for interacting with our data sources. Furthermore, since we are using Entity Framework as our object relational mapper (ORM). We can use C#’s LINQ library to implement data queries. 

**Infrastructure**

This package contains DTO’s used for communication between the modules of the backend API, as well as public interfaces for low coupling between modules.

**Data Access Layer**

We have a single class named “CustomDbContext” which contains all of our data sources. A representation of a database table is stored in a DbSet, which is a type provided by Entity Framework that supports LINQ querying.

**API**

The API module contains the primary business logic of the application, exposing endpoints (controllers) that can be interacted with using standard HTTP requests.

**Models**

Models contain POCOs (Plain Old CLI Objects) that only contain properties of the corresponding conceptual business entity.


#### Connector & Component View Point

We consider a component to be an implementation-agnostic logical unit of software.This means that a concrete implementation of any component can be replaced, as long as the new implementation provides the interfaces that are required from it by other components of the system. The following diagram shows what we identified as components of our system and the interfaces through which the components interact and depend on each other.


![alt_text](images/C_and_C.png "image_tooltip")


In our application, the backend is the central component. The backend exposes two interfaces: a frontend-facing interface and a logging interface, whose functionalities are implemented by the backend’s subcomponents. The API subcomponent implements the frontend-facing interface, provides a logger interface to the logging subcomponent, and requires a database manipulation interface from the repository subcomponent. The logging subcomponent implements the logging interface.

The frontend-facing interface is required by the simulator, frontend and the monitoring dashboard component. Remark that the requirements for the frontend-facing interface were determined by the simulator, as described in OpenAPI format in [session 7](https://github.com/itu-devops/2020-spring/blob/master/sessions/session_07/Session%2007.ipynb); this is signified in the figure by the dashed lines for the frontend and monitoring dashboard components. The logging interface is required by the log storage and visualization component, which allows for the logs to be inspected.

The colors code for component development responsibility: the green component was provided as course material, the blue components are the ones we implemented ourselves, and the orange components are the ones where we mostly relied on third-party technology and only used the component’s provided interface.

We deliberately choose to keep this description very high-level, and describe the interfaces in terms of the function they should fulfill, rather than a set of concrete functions, as these depend on the concrete implementation of green and blue components, and the choice of third-party technologies for orange components. Rather we provide another diagram showing only the central component, meaning the backend, in greater detail.

![alt_text](images/C_and_C_detail.png "image_tooltip")


Here all interfaces are made explicit, except for the logging interface. The orange subcomponent is again a third-party technology, whereas all other subcomponents were developed by ourselves. The repositories communicate with the database using MySql protocol. All incoming communication to the API is via http.


#### Deployment View Point

This subsection will describe how our system is structured in terms of deployment with regards to hosting, nodes and orchestration.



<p id="gdcalert4" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/Report3.png). Store image on your image server and adjust path/filename if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert5">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/deployment.png "image_tooltip")


**<&lt;cloud>>** 

Cloud describes which cloud provider is used to deploy the entire application. In this case the application is being deployed to DigitalOcean, as the team had a voucher from the student pack provided by GitHub and the service was strongly encouraged by the lecturers. In a different setting the application could be deployed to a more robust provider such as Google, AWS or Azure.

**<&lt;orchestration>>** 

Specifies which orchestration strategy is being used for the different images. In the minitwit architecture two orchestration strategies are used: Docker Swarm and docker compose. Docker Swarm is used to host the backend and monitoring tools, while docker compose is being used for Logging.

**<&lt;image>>**

Image is a docker image which contains the environment of the

desired application. In Minitwit project there are following images:



*   API \
Contains the Minitwit backend API, which was used by the frontend and the simulator
*   MySQL \
MySQL image is used to host the database
*   Prometheus

    Collected the different metrics exposed by the API image

*   PromSwarmConnect

    Allowed Prometheus to collect data from multiple instances of the API image 

*   Grafana

	Grafana was used to visualize metrics gathered by Prometheus



*   Kibana

    Kibana was used to visualize the logs exposed by ElasticSearch

*   ElasticSearch

    ElasticSearch has an indexed version of logs sent by the API image.


**<&lt;volume>>**

Volume is a docker volume, which is stored on the server

instance instead of the memory in a docker container, so it can be reused whenever a container fails or is shutdown during a deploy. In the minitwit application there are two volumes:



*   DB Storage 

    Used to store database files

*   ElasticSearch Storage

    Is used to store logs from all instances of the API image.


**<&lt;service>> **

Service tag identifies the load balancer that is built-in to docker swarm, however it is represented as part of the architecture, so that the architecture can easily be reused when switching away to an orchestration service such as Kubernetes. 

**<&lt;device>> **

Device represents the device the user uses to access the Minitwit UI.


### Tech Stack

When initially rewriting the provided Minitwit application in a different language we chose to separate the frontend and the backend API. Since the simulator would run against a backend API, this approach gives us the ability to scale and load balance the backend independently of the frontend. During the rewrite we also wanted to upgrade the database from SQlite to something more performant.

We rewrote the API in C#. We chose C# because it’s a language used widely in the industry and has many addons which may prove useful. Furthermore some group members had previous experience with the language. 

We chose to use Flutter for the frontend. When deciding what technology to use we looked at React and Flutter. In deciding which one to use we looked at Flutter, since it’s a relatively new cross platform framework that can be used in both desktop, mobile and web applications. React is a very popular framework and more stable, but to try something different and exciting we chose Flutter.

When choosing a database technology we considered MySql and MSSql, since both are viable options for our use case. MySql is an open source DBMS while MSSql is a Microsoft product. They both offer similar performance, functionality, and both integrate with Entity Framework using a Nuget package meaning switching from one to the other at a later point would be very simple. We chose MySql as both choices seemed equal and swapping later wouldn’t issue. 

To communicate between C# and MySql, we introduced an abstraction layer for the database. This would allow the system to interact with the database without directly writing SQL queries. We considered both Dapper and Entity framework core. We chose Entity framework core instead of Dapper due to the very simple integration to the existing stack we have chosen which reduces the build time, and due to Entity Framework being richer in features. Dapper has better performance than Entity Framework and in a different use case than a small scale twitter clone, it might present a more appropriate choice. Though Dapper’s performance is better, there is no significant performance gap between the two options. 


### Dependencies

The following graph visualizes the high-level software dependencies for the backend, excluding infrastructure code, artifacts, services, etc. The arrows show the direction of the dependency and the gray boxes indicate the weeks in which dependencies were added.

![alt_text](images/deps_simple.png "image_tooltip")


For a more detailed visualization, showing the concrete package dependencies, we used NDepend; a static code analysis tool, to generate a dependency matrix for the backend. This matrix shows also the external .NET dependencies. From the matrix we can read that the Infrastructure package (row) is used (blue color) by five code elements in Minitwit.API and Repository (column headers) and is using (green color) one code element from Minitwit.Models (docs: [https://www.ndepend.com/docs/dependency-structure-matrix-dsm](https://www.ndepend.com/docs/dependency-structure-matrix-dsm)).


![alt_text](images/deps_matrix.png "image_tooltip")


The frontend is written using Flutter with Dart, we can extract the dependencies from the pubspec.yaml file. From this file, we can identify the following external dependencies with the versions:



*   [responsive_builder: ^0.1.2](https://pub.dev/packages/responsive_builder)
*   [http: ^0.12.0+4](https://pub.dev/packages/http)
*   [toast: ^0.1.5](https://pub.dev/packages/toast)

Our frontend also depends on the Flutter SDK. We will omit dependencies among views in the frontend, as they are not relevant for the system.


![alt_text](images/deps_full.png "image_tooltip")


The above diagram displays the dependencies of the backend API on multiple levels of abstractions; including infrastructure code, artifacts and services, and is color-coded to show how dependencies developed throughout the project. The bold arrows display a dependency cycle, which is typically bad practice, however this is okay because it is part of the continuous delivery pipeline. Also, not that the docker images are pushed and pulled from DockerHub, but this is omitted.


## Application Infrastructure

This section will outline the infrastructure in place in our Minitwit application. We will outline how it’s orchestrated, monitored, secured and how we are logging events in the system.


### Orchestration 

When choosing virtualization we considered traditional VM deployment, Vagrant and Docker. We quickly chose not to go for traditional VM deployment because of the large overhead of managing the machine. Instead we chose Docker for virtualization.

We chose Docker instead of Vagrant because it’s a lightweight option, and it supports quickly spinning up new instances of containers and stopping running containers. Also when using Docker we can delegate keeping the environment updated to a host provider. 

When deploying our system, instead of deploying individual docker containers using `_docker-compose_` and `_docker-compose up_`, we opted to use an orchestrator for deploying the backend. We chose to release using an orchestrator to run a full scale orchestrated system that would allow us to easily scale horizontally if we needed to during the simulation. Looking at options we looked at Docker swarm and Kubernetes. Kubernetes is a more feature rich orchestration tool than Docker swarm is. It requires more configuration and requires the developers to set up an HA proxy for load balancing. This in turn gives developers more control of the system, but at an added complexity. 

We chose to orchestrate our containers using Docker swarm. We chose Docker swarm instead of Kubernetes because it’s simpler to get up and running, and when it’s running it provides automatic load balancing across containers. These features would allow us to set up orchestration with very minimal configuration and get it running fast. Given the time constraints and requirements, Docker swarm provided the needed functionality.

We configured Docker swarm with a service that runs 4 replicas of our api, one replica of our database and one replica of our monitoring system. 

Docker swarm has an added benefit in terms of updating services. When updating a service with replicas, Docker swarm allows us to configure how the service is updated, such as rolling, blue-green and canary updates. We chose a rolling deploy, where replicas would be taken down and updated one at a time with a waiting period in between. When using the built in rolling update, if a service fails to start by entering not the `RUNNING` state, the update stops, so that the entire service is not brought down. We chose a rolling deploy to have a maximal amount of replicas active at all times, and using this strategy we had at least 3 replicas running at the same time. 


### Monitoring Strategy 

To add monitoring to our system we decided to use prometheus for collecting logs from our containers, and grafana for visualizing the logs. We looked at alternatives such as DataDog and netdata, but ultimately chose Prometheus and Grafana because they are open source, and were presented during lectures. We used pull logging and decided to let Prometheus collect the logs of each container, and let Grafana pull from Prometheus at regular intervals (15 seconds). We added both a Grafana and a Prometheus container to our existing stack.

We ran into an issue with prometheus and pull based monitoring, where the prometheus service discovery doesn’t work in Docker Swarm. To remedy the issue we used a third party connecting service to allow Prometheus to discover the API containers and obtain their logs. The service we used is called PromSwarmConnect ([https://github.com/function61/promswarmconnect](https://github.com/function61/promswarmconnect)) and presents a triton service discovery endpoint which prometheus can find, from this endpoint it obtains the ip’s of the API containers and can obtain the logs. We set up the Grafana to pull from Prometheus and using PromQL we configured a monitoring dashboard.

When deciding which parameters to measure we opted for standard endpoint monitoring such as number of requests, response time and size of requests. Along with endpoint monitoring we measured CPU usage and RAM usage for each of our containers as well. 

We chose to monitor response time to see if any endpoints were slower than the allowed response time allowed by the simulator. We looked at container metrics such as ram and cpu usage to gain an idea if we should scale horizontally to gain more resources. Number of requests and size of requests is in this case not very interesting since the simulator is running a fixed number of requests. However in a real scenario the number of requests is important for business.

Below are two screenshots from our Grafana monitoring dashboard. There are duplicate entries due to a bug with our Prometheus and PromSwarmConnect solution. Specifically, Grafana is still attached to the old containers after a deployment. 


![alt_text](images/grafana1.png "image_tooltip")

![alt_text](images/grafana2.png "image_tooltip")



### Security Assessment 

When conducting a security and risk assessment of our system we discovered we violated at least two of the OWASP top 10 security risks to web applications:



*   Authorization (OWASP nr. 2)
*   Insufficient logging (OWASP nr. 10)

To remedy this, we started by adding additional logging to our controllers (see[ PR #59](https://github.com/LazyOpsDev/Minitwit.Backend/pull/59) on Minitwit.Backend). We added logging where it seemed appropriate, i.e. at the controllers where the business logic resides. Specifically, we log events such as failed and successful login attempts as well as general errors and exceptions.

Initially the API used authentication on the appropriate endpoints. However due to differences in authentication in between the simulator and the provided API implementation, we disabled authentication. The simulator uses an authentication header with a fixed value and the given API used cookie based authentication. While we initially implemented cookie based authentication we disabled it to allow traffic from the simulator. 

This meant we had authentication in place, however it was not enabled. To enable authentication we had to introduce both the initial authentication and an option to use the simulator authentication. To do this we included an additional statement to the authentication that validates the request, using either the authentication header or cookies, and if either is valid the request is accepted.


![alt_text](images/auth_variable.png "image_tooltip")


We have hardcoded the authorization header value, which still poses a security risk to the system, but given the environment it’s acceptable.

The full security assessment can be found [here](https://github.com/LazyOpsDev/Minitwit/blob/develop/SLA_vulnerability%20report.pdf ), including the security assessment of another group’s application.


### Logging

When introducing logging we chose the _ELK _stack for logging without _Logstash_. We didn’t use Logstash because there was no perceived gain in adding Logstash and also we didn’t see a need for transforming the log entries initially. Adding Logstash at a later point is possible, which is why we decided not to use Logstash at first.

Server-side we use SeriLog, a .NET logging framework, for logging events and parse those directly to Elasticsearch. SeriLog is configured with the IP of our ElasticSearch service and pushes logs directly to that, which Kibana then uses to visualize logs. We only do logging in the backend api since this is where all the business logic is encapsulated.


![alt_text](images/logging.png "image_tooltip")


As alternatives to SeriLog, we considered using NLog and Log4Net, both of which are popular logging solutions in the .NET world. We decided to go with SeriLog due to previous experience and that Serilog is also widely used in the industry.

We decided to go with the elastic stack as it is also industry standard. Microsoft provides the Elastic Stack as a logging example for their cloud solution: [https://docs.microsoft.com/en-us/dotnet/architecture/cloud-native/logging-with-elastic-stack](https://docs.microsoft.com/en-us/dotnet/architecture/cloud-native/logging-with-elastic-stack) 

We also took a look at Splunk, an alternative to the ELK stack. It is very powerful, but is geared more towards enterprise architecture, and is expensive. Hence, why we ended up using ElasticSearch and Kibana instead.


## Team Collaboration and Work Distribution

This section will outline the process the group has undertaken during the Minitwit project.


### Distributed Workflow

After consideration of many workflows, the team decided to stay with _Centralized Workflow _with two repositories described in the _Repository Model_ section.

The project has a small size code base and closed team of contributors, also it is supposed to be a private project. For this reason, there will never be a need to fork any repositories and use neither _Integration-Manager_ nor _Dictator and Lieutenants Workflow_. For sake of this course repositories are public, this way we could use many free tools which support open-source.


### Repository Model

The project consists of two packages:



*   [Minitwit.Backend](https://github.com/LazyOpsDev/Minitwit.Backend)
*   [Minitwit.Frontend](https://github.com/LazyOpsDev/Minitwit.Frontend)

Linked as git submodules in the main application repository [Minitwit](https://github.com/LazyOpsDev/Minitwit).

We decided to adopt this strategy because we can develop backend, frontend separately and keep the commits for each of that code separate. Then, for everything backend- and frontend- related, we can cherry-pick the commits from their master branches to commit to the master branch of the top-level repository, and make a release this way. Read more about git submodules here: [https://git-scm.com/book/en/v2/Git-Tools-Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

We use the same branching model for each repository.


### Branching Model

The project tries to follow git-flow branching strategy, it doesn’t take all rules into account but uses subset of the idea:



*   _master_ - serves as the latest release branch and only CI user can merge directly to it (no other user has permissions to push there).
*   _develop_ - is the main branch, all other branches are merged to it by PR.

Other branches, e.g. feature, document and fix, were named freely by a developer, without specific git-flow naming convention (characteristic prefixes). All these branches were short-lived, some of them introduced a new feature in code and other provided relevant fixes.

We also provided branch protection rules for develop and master branch:

![alt_text](images/branch_rules.png "image_tooltip")


The master branch is protected and no one can push, merge or commit to it, except from one github user - _lazyopsdev-ci_. The user was created for these purposes. The develop branch follows different rules, no one can directly commit to it, but it allows merging by creating pull requests. Before the code gets merged, at least one collaborator needs to approve changes and all checks, like Travis CI and SonarCloud, need to pass.

This makes sense since we are developing a web application where a unit of work is usually a feature. For instance, something like a trunk/no-branches model would be more appropriate for some computational library where a unit of work is to add a function that performs some calculation, but in our case it would force us to make larger, stable commits, since otherwise going back in history to fix something or developing more than one feature in parallel would be a pain. By using feature branches we avoid this.


### Commits

Stable release or branch means that it can build, run and pass all tests.



*   Commit to _master_ is a release and must be stable.
*   Commit to _master_ is either a merge from _develop_ or _hotfix_ branch.
*   Commit to _develop_ is either merge from a _feature_, _document_ or _hotfix_ branch.
*   Commit to _develop_ must be stable (don't commit unfinished feature or hotfix work to develop, this does not apply to documentation).
*   Commit to _feature_, _hotfix_ or _document_ branch has no requirements.


### Pull Requests

When making a pull request to merge a branch into develop it requires the approval of one other developer. Merging to master is totally suppressed and allowed only by a special CI user who merges from develop to master branch when the release pipeline succeeds. This way we organize integration of code among ourselves.


### Releases

The project utilizes the git releases to track each development stage. For backend the format of release is **Major.Minor.Patch **whereas for frontend it was sufficient to have format **vMajor.Minor**. An example for [Minitwit.Backend releases](https://github.com/LazyOpsDev/Minitwit.Backend/releases).

We have established that the initial release was 1.0.0 for backend and v0.1 for frontend. The reason for that was the Minitwit project didn’t have any management, product roadmap or milestones, thus there was no need to think in terms of pre- or beta-releases.

Releases are triggered by the developers by creating a tag on the develop branch. Later the CI tools are performing actual artifact building and deployment - detailed description in section _Continuous integration, delivery and deployment_.


### Task Planning and Assignment

We used GitHub’s _Projects_ for task identification, planning and specification. Each week we received many tasks as an assignment for the course, so first we listed them, then we described acceptance criterias (what condition the task has to meet in order to be accepted and closed as done), next we converted tasks into github issues and assigned people.

After each task was concluded as a pull request, collaborators had enough time to review it and approve the change. After that, the PR was merged into develop.


![alt_text](images/github_board.png "image_tooltip")


As seen in the picture above We decided to use _GitHub Projects_ because this tool was available off-the-shelf without extra configuration, deployment and setup. There were other possibilities, such as [jira.com](https://www.atlassian.com/software/jira), [teamgantt.com](https://www.teamgantt.com) or [monday.com](https://monday.com), however after discussion we concluded that the scope of this course doesn’t involve project management or similar and mainly for that reason we decided not to use it. 


## Current State of the Application 

This section will briefly describe the current state of the Minitwit application based on its technical debt and its compliance in regards to its service level agreements.


### Technical Debt

Looking at the results of our static code analysis from _SonarCloud_, we found that a large portion of our code has “code smells” and that a lot of technical debt was due to unused “using” statements in our code, specifically 49/105 code smells. Each of which is estimated to take 2 minutes to remedy, according to _SonarCloud_. This equates to over 1½ hours of maintenance on using statements which obviously creates a bloated technical debt. To remedy this, we ran an integrated code-clean up tool from Visual Studio, which automatically removes all unnecessary using statements instantly.

![alt_text](images/techdebt.png "image_tooltip")


The result of running this code-clean can clearly be seen in the decline in code smells and thereby technical debt. The reason we had the initial spike in code smells was due to misconfiguration of SonarCloud. Specifically, not all projects were included and analysed initially, which resulted in the spike when we fixed the misconfiguration.


### SLA Compliance

Through monitoring, we can see that we comply with all of our Service Level Agreements (SLAs), except two, namely the response time of the /public endpoint, exceeding its SLA limit by 0.6 seconds and our uptime.

We believe the response time could be remedied by applying indices to the database, as we heard other groups experienced significant performance increases by doing so. Our SLA document can be found in the appendix.

Furthermore, we computed our uptime via the latest table and error overview presented at [http://142.93.104.18/status.html](http://142.93.104.18/status.html) (2020-05-10). By computing the total amount of errors and adding it to our latest count, we computed the total amount of requests received. We then divided this by our latest tag to compute our uptime. This resulted in an uptime of 99.85%, which is sadly 0.1% shy of our SLA goal. 

Below is a table with the calculation of the uptime as specified in the SLA and can be calculated with following formula: 

(Total Transaction Attempts - Failed Transactions) / Total Transaction Attempts


<table>
  <tr>
   <td>read timeout
   </td>
   <td><p style="text-align: right">
8962</p>

   </td>
  </tr>
  <tr>
   <td>connection error
   </td>
   <td><p style="text-align: right">
3</p>

   </td>
  </tr>
  <tr>
   <td>follow
   </td>
   <td><p style="text-align: right">
2563</p>

   </td>
  </tr>
  <tr>
   <td>unfollow
   </td>
   <td><p style="text-align: right">
1439</p>

   </td>
  </tr>
  <tr>
   <td>tweets
   </td>
   <td><p style="text-align: right">
1596</p>

   </td>
  </tr>
  <tr>
   <td>Total failed transactions
   </td>
   <td><p style="text-align: right">
14563</p>

   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Successful transactions
   </td>
   <td><p style="text-align: right">
9404399</p>

   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Total transactions
   </td>
   <td><p style="text-align: right">
9418962</p>

   </td>
  </tr>
  <tr>
   <td><strong>Uptime</strong>:
   </td>
   <td><p style="text-align: right">
99.84538636</p>

   </td>
  </tr>
</table>



## Continuous integration, delivery and deployment

The project uses concepts such as automated pipeline for code integration, part of github checks and automated delivery and deployment. In the coming sections we describe the setup and infrastructure, and in the _discussion_ part we elaborate on reasons why we’ve chosen these and not others technologies and options.


### Continuous Integration

For the CI checks we decided to use [TravisCI](https://travis-ci.com) (bootstrap plan, education subscription). Travis is a popular tool for software delivery and integration, widely used for open-source and commercial projects. It offers capabilities for constructing and maintaining pipelines, all out of the box, ready as SaaS. The product offers a limited free plan, which still works for testing and exploring.

Travis doesn’t require any infrastructure, setup or maintenance, it is deployed and offered as a service. Because the platform is tightly integrated with github and other services, there is no overhead to communicate pipelines, configure triggers and test its integration.

The pipeline can be specified in a conventional format, such as YAML. Let’s zoom in on an example from our _Minitwit.Backend_ repository, part by part. The first section of the pipeline configures the programming language, framework version (dotnet 3.1.100) and declares what branches are included in the pipeline. In our case any branch except master is a target for jobs, this is because we treat master as a snapshot of develop in the moment we make a release, hence no need to rerun builds and tests.


```
language: csharp
mono: none
dotnet: 3.1.100
services:
  - docker
install:
  - dotnet restore API/Solution1.sln
branches:
  except:
    - master
```


The next part of the pipeline describes the tools that are used for static code analysis, as we can see below, we used [SonarCloud](https://sonarcloud.io). The service runs in the background and delivers extra information about technical debts found in the code.


```
addons:
  sonarcloud:
    organization: "lazyopsdev"
    token:
      secure: "secure-token-here"
```


The section below defines many stages which are inherent components of every pipeline. Each stage consists of multiple jobs, in our case however, there is only one job per stage, for simplicity. The first stage builds the source code, and if it is successful it allows the pipeline to continue to the second stage. The second stage runs unit tests which are included in the source code. The final stage is conditional, and is executed  if a tag is set while triggering a run. In most cases the pipeline is triggered by a pull request or commits, so this part is usually omitted. At the moment, when the developers decide to release the software, they only tag the develop branch and this job merges it automatically to master branch. However Travis doesn’t make it easy to merge the branch from the pipeline. We used a special GitHub user for doing these kinds of operations - _lazyopsdev-ci_.


```
jobs:
  include:
    # Build and run tests
    - stage: build
      name: "Build Solution"
      script: 
        - dotnet build --configuration Release API/Solution1.sln

    - stage: test
      name: "Run Tests"
      script: 
        - dotnet test API/Minitwit.Tests/Minitwit.Tests.csproj

    # GitHub auto merge to master:
    - stage: auto-merge
      if: tag IS present
      name: Merge to master
      env:
        - GITHUB_USER_EMAIL="lazyopsdev@gmail.com"
        - GITHUB_USER_NAME="lazyopsdev-ci"
        - GITHUB_REPO_URI="LazyOpsDev/Minitwit.Backend"
      script:
        - git config --global user.email "$GITHUB_USER_EMAIL"
        - git config --global user.name "$GITHUB_USER_NAME"
        - git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* || exit
        - git fetch --unshallow || exit
        - git checkout master || exit
        - git merge --no-ff "$TRAVIS_COMMIT" || exit
        - git remote add travis-origin https://${GITHUB_USER_NAME}:${GITHUB_TOKEN}@github.com/${GITHUB_REPO_URI}.git > /dev/null 2>&1
        - git push --quiet --set-upstream travis-origin master
```


The last section of pipeline is the notification part, each time the pipeline finishes it sends a message with appropriate information to listed emails about success or failure.


```
notifications:
  email:
    - krza@itu.dk
    - ...
    - arca@itu.dk
```


Below the screenshot from TravisCI, _Minitwit.Backend_ pipeline.

![alt_text](images/travis_results.png "image_tooltip")


It is worth to notice the pipeline finishes in 3.5 mins, which is our deployment lead time. Below also a view of the build logs.


![alt_text](images/travis_output.png "image_tooltip")



### Continuous Delivery

We are using [CircleCI](https://circleci.com) for continuous delivery. The platform has as much to offer and as TravisCI it is also a SaaS. We first talk about configuration of workflows. We have defined one workflow with two jobs: _build_and_deliver_ and _deploy_artifact_; delivery focusing only on the first job. Workflow defines conditions i.e. on what event the pipeline should be triggered, in CircleCI its called filters. We declared the delivery pipeline to be triggered only by a github tag, during the release process.


```
 workflows:
  version: 2
  deliver_and_deploy:
    jobs:
      - build_and_deliver:
          filters:
            tags:
              only: /.*/
            branches:
              ignore: /.*/
      - deploy_artifact:
          requires:
            - build_and_deliver
          filters:
            tags:
              only: /.*/
            branches:
              ignore: /.*/
```


Next part of the pipeline is jobs specification. The _build_and_deliver_ runs docker build to obtain docker image and then pushes it to DockerHub, the artifactory for docker images.


```
 jobs:
  build_and_deliver:
    docker:
      - image: circleci/buildpack-deps:bionic
    steps:
      - checkout
      - setup_remote_docker
      - run:
          name: Build Docker Image
          command: |
            cd API && docker build -t $IMAGE_NAME .
      - run:
          name: Push Docker Image
          command: |
            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
            docker tag $IMAGE_NAME "$ORG_NAME/$IMAGE_NAME:$CIRCLE_TAG"
            docker tag $IMAGE_NAME "$ORG_NAME/$IMAGE_NAME:$DEFAULT_TAG"
            docker push "$ORG_NAME/$IMAGE_NAME:$CIRCLE_TAG"
            docker push "$ORG_NAME/$IMAGE_NAME:$DEFAULT_TAG"
```


Caching is a powerful approach to speed up the pipeline, as it allows reuse of previous builds in future runs. We did some experimentation with it, however it was not included in the final version of the pipeline as in our case there was no significant improvement, it only added complexity.


```
      ## Caching example:
      # - run:
      #     name: Save Docker layer cache
      #     command: |
      #       mkdir -p /home/circleci/docker-caches
      #       docker save -o /home/circleci/docker-caches/app.tar $IMAGE_NAME
      # - save_cache:
      #     paths:
      #       - /home/circleci/docker-caches
      #     key: v-{{ .Environment.CIRCLE_TAG }}
```



### Release Artifactory

The deployable software artifacts in our project were defined at an early stage of application development. We decided to build docker images, because these kinds of artifacts are easy to ship and have a very concise way to download or run it. The images are stored in DockerHub. The team also discussed other possibilities, such as building NuGet packages or docker images and later storing them in [JFrog Artifactory](https://jfrog.com/artifactory/), however this turns out to be too heavy for our application. NuGet packages are not strictly deployable. This kind of release would be a good choice if we had a considerable amount of custom packages written in C# for reusability and modularity, this project doesn’t have that.

The picture below shows backend releases, each release on github has its own corresponding image, the tag _latest_ always points to the very last release, in this case v1.3.1. This way, in case of emergency, we can very quickly revert the latest image to the previous one, without rebuilding anything.

![alt_text](images/dockerhub.png "image_tooltip")



### Continuous Deployment

Provided that other jobs in the deployment pipeline succeed, the final job can take over and start deploying to a remote server. Before any action can take place on a remote host, an ssh connection has to be established. For that purpose the CircleCI service needs a private ssh key configured, so it can be used while running the pipeline. In the configuration file, any credentials are referenced by an ID or, in case of ssh keys, fingerprint. The deployment command is fairly easy, it copies docker-compose file and deployment script on remote server and executes it there.


```
  deploy_artifact:
    docker:
      - image: circleci/buildpack-deps:bionic
    steps:
      - checkout
      - add_ssh_keys:
          fingerprints:
            - "af:ba:8c:dc:ed:b1:80:2e:6b:c7:72:74:2f:3a:1c:40"
      - run:
          name: Deploy Application
          command: |
            # Update docker compose file:
            scp -o "StrictHostKeyChecking no" API/docker-compose.yml ${SERVER_USER}@${SERVER_IP}:~/docker-compose.yml
            scp -o "StrictHostKeyChecking no" deploy.bash ${SERVER_USER}@${SERVER_IP}:~/deploy.bash
            ssh -o "StrictHostKeyChecking no" ${SERVER_USER}@${SERVER_IP} \
              "source /root/.bashrc && \
              ./deploy.bash"
```


The deployment script first checks whether services are running, and if they are, then it applies a rolling update on all backend replicas (at the moment 4 are configured). We used the _docker service update_ command to apply the docker image update. Otherwise if the backend is down the pipeline brings it online with docker stack deploy command. The content of [deploy.bash](https://github.com/LazyOpsDev/Minitwit.Backend/blob/develop/deploy.bash) script:


```
#!/bin/bash

# Check if services are up and running:
# - fails when docker swarm isn't configured,
# - fails when docker services are not running
docker service ls | grep minitwit
if [ $? -eq 0 ]; then
    echo "Apply rolling updates"
    docker service update \
        --image lazyopsdev/api:latest \
        --update-parallelism 1 \
        --update-delay 10s \
        minitwit_minitwit

# If services are not yet running, bring them up:
else
    echo "Deploy all services if not running"
    docker stack deploy -c docker-compose.yml minitwit
fi
```


The view of release pipeline summary from CircleCI.


![alt_text](images/circleCI.png "image_tooltip")



### Static Code Analysis

The static code analysis would allow the development team to see software quality and the technical debt. SonarCloud was identified as the desired tool to use for the static analysis, as it was open source and easily available. SonarCloud is easily modifiable and allows the development team to see the quality of the code and assign tasks to each other.

Whilst implementing SonarCloud in the Travis pipeline the team realized that Travis CI and SonarCloud with a .NET solution was not supported. A more suitable option for C# solutions is Azure DevOps. 

We created a pipeline on Azure DevOps that will perform our static analysis. The pipeline consists of the following steps (see picture below).


![alt_text](images/ci_devops.png "image_tooltip")


The pipeline was automatically run by activating the continuous integration and pull request validation in Azure DevOps. The CI pipeline would activate whenever a change was made on the ‘develop’ branch in folder “API”. The pull request validation will be activated whenever a PR is made to the ‘develop’ branch. Below the view of continuous integration and pull request configuration.

![alt_text](images/ci_trigger_2.png "image_tooltip")

![alt_text](images/ci_trigger_1.png "image_tooltip")


The result of static code analysis captured as a screenshot from SonarCloud, the summary presents reliability, security and maintainability measures.

![alt_text](images/sonarcloud_results.png "image_tooltip")


The picture below presents the result of static code analysis included automatically as a github pull request comment to make it visible for developers and reviewers.


![alt_text](images/sonarcloud_github.png "image_tooltip")



### Discussion About CI & CD

We have tried many CI services, _TravisCI_, _CircleCI_, _Jenkins _and _GitHub Actions_. All of them deliver features we need, and each service would work for us. We tried four of them to practice and learn significant differences between them, to see how hard it is to get it working. Apart from that we also wanted to know what other capabilities and use cases they can provide, for the future applications.

Our first integration platform was _GitHub Actions_, it provides very suitable and intuitive configuration for pipelines. It is still included in the backend repository in _.github/workflows_ directory. It is fairly easy to build simple checks for source code, however actions have limited applications and therefore if one thinks to build advanced and highly configurable automated pipelines, then this is not a good choice.

The second platform we tried was _TravisCI_. We wanted to try this one because it is a very popular CI service, widely used in business and open-source. Initially TravisCI was the best fit for our needs, however with time we realized that deployment of the backend couldn’t be done with TravisCI. The reason was that our project is also a _GitHub Organization_ and to add deployment ssh keys, the TravisCI requires _GitHub Team_ or _Enterprise_ (a paid subscription) to enable this option. This is not a case when using a normal github repository (not under organization). Also, using TravisCI for open-source gave us similar problems with organization and also other limitations, therefore we’ve chosen _travis-ci.com_ over_ travis-ci.org_.

The third option was _CircleCI_, which has been recently gaining popularity among software developers. We also tried this for the same reason, to compare it to TravisCI and others. The CircleCI platform was very easy to use and configure. Moreover, we haven’t experienced any problems with ssh deployment keys, which also worked correctly with GitHub Free Organization. The only inconvenience with CircleCI was pull request checks, it turned out that the service didn’t support them initially and this feature could be enabled in a non intuitive way - by enabling option _Only build pull request_. We used CircleCI for continuous delivery and deployment.

We have also tested _Jenkins _pipelines. Jenkins isn’t provided as a service, it needs to be deployed and configured for custom usage. We’ve run it on _DigitalOcean Droplet_, applied simple configurations, such as, installation of necessary plugins, and creation of user’s accounts etc. There were many security issues during the setup, a GitHub pull request plugin couldn’t work due to missing java dependencies, so we’ve rebuilt jenkins’ docker image. In order to ensure higher security of Jenkins we would need to run it under https instead of http, maybe it is even wiser to use Jenkins over VPN, thus avoiding exposing it directly (many companies use this kind of setup in production). We spent a lot of time setting up Jenkins before we realized that Jenkins isn’t the best fit for small projects, it is much more advanced and allows us to make any conceivable pipeline in a declarative or even scripting style (Groovy Scripts).

Typical process time in our project was around one week, from the moment when we received assignment tasks to actual delivery. The diagram below depicts developing phase (1 week) and release time (only 3 minutes). More strictly, the process time includes release time, diagram only focuses on presenting duration proportions.

![alt_text](images/lead_time.png "image_tooltip")


All CI services offer build status as a badge icon, depending on pipeline status it tells whether build is passing or failing. The picture below presents conventional use of build badges included in the repository’s _README.md_ file.


![alt_text](images/badges.png "image_tooltip")



## Software quality

This section will describe our views of software quality, as described in the “Software Quality: An Elusive Target” paper by Kitchenham & Pfleeger.

**Manufacturing view **

During development of the Minitwit application we used code reviews and code analysis tools to ensure that the software we merged to our development was stable. 

These measures were in place to focus on the reliability and stability of the final product. By testing and reviewing code we aimed to deliver a stable product.

A way to increase reliability and stability is to completely commit to test driven development. We did not go for this option, but did during the project introduce tests to ensure that new code wouldn’t make the product unstable or unreliable.

**User view **

From the point of view of the user our system focuses on maintaining a high availability and good performance. By maintaining fast response times we can keep the user from leaving the site if it feels sluggish. According to nngroup([https://www.nngroup.com/articles/website-response-times/](https://www.nngroup.com/articles/website-response-times/)), a response time less than a second gives the user a seamless experience of the page. With less than half a second response times on almost all our endpoints, we aim to give the user a seamless experience.

	

**Product view**

We obtain the maintainability and reliability from our static analysis via SonarCloud reports, which describes maintainability as our technical debt, and reliability as our technical debt vs test coverage. 

**Value based view**

In general with a Minitwit system with the functionality we have in our system monetization will be a challenge. Similar applications social media platforms have monetize their platform via ads, i.e. Twitter, Facebook, Instagram etc. but not by having users pay for the platform. 

This would be a possible monetization strategy for the Minitwit platform as well.

In general we focussed on maintaining good performance and availability of the API. And in ranking these we prioritized performance over availability. This was due to the nature of the simulation, where performance was directly impacting the visual representation of the scoreboard. Availability, while very important as well if servers went down, was not always directly correlated with results as performance.


## 


## Reflections

From the start of the project, we chose to use docker swarm, which proved to be a good idea, since we aimed to minimize downtime. In particular, docker swarm allowed us to have more instances running of the same application and with that also enabling us to make use of rolling deployment. 

The way we implemented the database in the swarm was not ideal. We added a MySQL container and mounted a volume so data in the database would be stored on the virtual machine and not in the container. This in itself is recommended, however since we were using droplets in DigitalOcean we didn’t have full control of the VM. It would have been preferable to run a managed database as a service instead. We had an issue of only having one database container fixed on a specific node, and we had to introduce placement constraints if adding additional nodes to the swarm to avoid that the database would be redistributed. Having the database on a single server with no backups is a single point of failure for the system, regular timed backups of the database would minimize data loss in case of the server dying.

Though the monitoring solution to make pull logging possible was not elegant, it was implemented. Later in the project we looked into changing from pull logging to push logging since this would solve the problem with service discovery entirely. This would mean we could have set up the replicated services to push logging to prometheus. To achieve this we could have used a push gateway, which is an intermediary service provided by Prometheus to allow monitoring of services which can’t be scraped. A push gateway however comes with issues as well, since while it is made to be used for push logging, it’s not meant as a permanent solution. A push gateway is meant for storing monitoring logs of scripts and short lived programs that are not continually running. While the Push Gateway is a solution presented by Prometheus, it’s not a recommended solution for our needs. In the end we didn’t change the logging, since neither way was recommended or elegant we kept the existing solution.

Though picking Flutter for Web as our frontend technology was exciting to work with, it is still currently a beta technology. This caused us to spend a lot of time with issues related to the framework being beta technology. An example of this was a routing bug, which was an open issue at their GitHub repository at the time we were developing the frontend. In retrospective React or Blazor pages would have been a more robust and solid choice. 

At the end of the project our _/public_ endpoint resulted in breaking our SLA by exceeding the average response time with 0.6 seconds. This problem went unnoticed as the monitoring dashboard was not regularly checked. This problem would have been avoided with continuous monitoring of the dashboard, and solved by adding indexes to the database. Specifically indexing the creation date of a post would improve the speed in the endpoint of concern because when querying for all posts we would first sort them.

The team has gone from implementing different services for CI and CD. We used TravisCI.com to build pipelines, then we tried the same settings with TravisCI.org, which we chose not to go with. We also evaluated our deployment pipeline on CircleCI. In retrospect, it would have been nice if we made a fork of our repor with which to test out different CI tools and configuration, since now there is a lot of noise in the commit history related to this.

At the end of the project, we needed to have a static analysis tool in the build pipeline. We went with SonarQube, however after having spent several days trying to get our code to be analyzed we figured out that Travis CI and SonarQube with C# codebase is not a recommended setup. Thus, we needed to shift once again to a newer platform, which was Azure DevOps, as that was the recommended way of setting up the pipeline according to SonarQube. 

Looking back at the process it would have been better if the development team only used Azure DevOps, as it allows the development team to manage all resources in one place. 




## Appendix


### Service Level Agreement

Document details & change history


<table>
  <tr>
   <td><strong>Version</strong>
   </td>
   <td><strong>Date</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>1.0
   </td>
   <td>2020/03/26
   </td>
   <td>Initial SLA Agreement
   </td>
  </tr>
</table>


Document approvals


<table>
  <tr>
   <td><strong>Name</strong>
   </td>
   <td><strong>Signature</strong>
   </td>
   <td><strong>Date</strong>
   </td>
  </tr>
  <tr>
   <td>Krzysztof Abram
   </td>
   <td>Krzysztof Abram
   </td>
   <td>2020/03/26
   </td>
  </tr>
  <tr>
   <td>Alexander Banks
   </td>
   <td>Alexander Banks
   </td>
   <td>2020/03/26
   </td>
  </tr>
  <tr>
   <td>Philip Korsholm
   </td>
   <td>Philip Korsholm
   </td>
   <td>2020/03/26
   </td>
  </tr>
  <tr>
   <td>Arian Sajina
   </td>
   <td>Arian Sajina
   </td>
   <td>2020/03/26
   </td>
  </tr>
  <tr>
   <td>Marek Kisiel
   </td>
   <td>Marek Kisiel
   </td>
   <td>2020/03/26
   </td>
  </tr>
</table>


Last Review: 2020/03/26


## 


#### SLA Introduction

_This is a Service Level Agreement (SLA) between Minitwit Users and Minitwit. This document identifies the services required and the expected level of services between 2020/03/26 to 2020/04/30, of the Minitwit API located at: [http://207.154.252.199](http://207.154.252.199/)._


#### Definitions

This section explains all keywords used in the SLA contract.


<table>
  <tr>
   <td><strong>Term</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>SLA
   </td>
   <td>Service Level Agreement
   </td>
  </tr>
  <tr>
   <td>API
   </td>
   <td>Application Programming Interface
   </td>
  </tr>
  <tr>
   <td>Failure frequency
   </td>
   <td> Failed Transactions per week
   </td>
  </tr>
  <tr>
   <td>Total Transaction Attempts
   </td>
   <td>Total amount of attempted transactions
   </td>
  </tr>
  <tr>
   <td>Failed Transactions
   </td>
   <td>Transactions resulting in either a 5xx status code or with a duration longer than 0.5  second.
   </td>
  </tr>
  <tr>
   <td>Uptime Percentage
   </td>
   <td>(Total Transaction Attempts - Failed Transactions) / Total Transaction Attempts
   </td>
  </tr>
</table>



#### Purpose

This section defines the goals of this agreement.


<table>
  <tr>
   <td><strong>Purpose</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>Agreed service target
   </td>
   <td>Minitwit API
   </td>
  </tr>
  <tr>
   <td>Duration of this SLA contract
   </td>
   <td>35 days
   </td>
  </tr>
</table>



#### Service Agreement Details

The minitwit team aim to uphold the following metrics: 


<table>
  <tr>
   <td><strong>Service</strong>
   </td>
   <td><strong>Metric</strong>
   </td>
   <td><strong>SLA Target</strong>
   </td>
  </tr>
  <tr>
   <td>Minitwit API
   </td>
   <td>Availability / Uptime percentage
   </td>
   <td>99.95%
   </td>
  </tr>
  <tr>
   <td>Minitwit API
   </td>
   <td>Max response time on individual endpoints of 
   </td>
   <td>1 second
   </td>
  </tr>
  <tr>
   <td>Minitwit API
   </td>
   <td>Failure frequency 
   </td>
   <td>&lt; 30 failed requests pr. week
   </td>
  </tr>
</table>

