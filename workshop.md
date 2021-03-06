# Cloud Native in Kubernetes Workshop

In this workshop we will be looking at an scenario that illustrate the journey that you will need to experience if you are planning to build using a cloud native architecture which you want to run on Kubernetes. 

Slides: [Slides in SlideShare](https://www.slideshare.net/salaboy/cloud-native-java-in-kubernetes)

This workshop relies on the following technology stack: 
- Spring Boot 2
- Spring Cloud Finchley
- Docker
- Kubernetes

We will be using [Jenkins X](http://jenkinsx.io) to accelerate the entire journey but it is not required to get all the services working. 

This example was built as an Open Source Project to highlight the capabilities of Activiti OSS which is a Cloud Native Business Automation Suite. All the software is currently in Alpha stage and we are planning to release a Beta1 release at the end of June, early July. We would love to get your feedback once that release is out. It is important to understand that even the example was built to highlight some of the Activiti OSS project capabilities, it uses standard practices for Cloud Native Applications that can be reused in any project. We aim to help to reduce the time that Software Developers spend on this journey by sharing examples that you can use as baselines for your projects. 

This example is not perfect in any way, but it is an evolving examples that we keep improving for each of our Activiti OSS releases. We use this example in a continuous deployment approach to test, validate and migration paths between releases. 

Notice that with some work, you should be able to run this example in your local environment, but if you are going Cloud Native we totally recommend a real cluster. 


- [Jenkins X](#jenkins-X)
- [Repositories](#repositories)
- [Front End](#front-end)
- [Single Entrypoint](#single-entrypoint)
- [Our First Service](#our-first-service)
  - [Configuring Nexus](#configuring-nexus)
- [Adding Environment Dependencies](#adding-environment-dependencies)
- [Building a Marketing Campaign](#building-a-marketing-campaign)
- [Consuming Data](#consuming-data)
- [System to System Integration](#system-to-system-integrations)
- [Next Steps](#next-steps)

For more details about the scenario you can look at the [Scenario document](scenario.md)

![Scenario](/scenario.png)

You need to have Jenkins X installed in order to accelerate the deployment of your services into a Kubernetes Cluster. You can definitely achieve the same results without Jenkins X, but the HELM Charts and Kubernetes Descriptors for the Trending Topic Campaign Project were created using Jenkins X.

# Jenkins X
Follow the instructions to install Jenkins X (JX CLI) from their [getting started guide](https://jenkins-x.io/getting-started/install/).

If you don't have a Kubernetes Cluster with [Jenkins X](http://jenkinsx.io) installed on it you can create one with:

> jx create cluster gke --kubernetes-version=1.9 --verbose

  - Select **europe-west1-c** [19]
  - Select **n1-standard-2**

Note: Look at [Jenkins X](http://jenkinsx.io) website for more information about how to install and create K8s clusters in other Cloud Providers
Note (1): In case you have errors about gcloud, please install google cloud sdk from:
https://cloud.google.com/sdk/docs/
Note (2): If you want to acess to your google cloud console go here https://console.cloud.google.com/home/dashboard
Note (3): In case you have errors about helm, install it from the helm 2.8.2 repository : https://github.com/kubernetes/helm/releases/tag/v2.8.2

This will create a new Kubernetes Cluster inside GKE. Notice that you  need to have the Kubernetes Engine enabled into your account in order to use it.
- Make sure that you sign in with the google account where you have your kubernetes cluster.
- Select your availability zone, Then accept all the defaults.
- **When the process ends, make sure that you record the password for Jenkins X admin user.**

When Jenkins X is installed you can use the following command to see all the Jenkins X installed services

> jx get urls

- Copy & Access Jenkins URL and login (user admin/{provided jx password})
- Copy & Access Nexus URL and login with the same credentials used for Jenkins

By default, Jenkins X create 3 environments
- *dev* is where you do the work
- *staging* is where your services will live (run) as soon as you register them in Jenkins X. Services will be automatically promoted to this environment.
- *production* is where your services will serve real clients. You need to manually promote services to this environment.
Use the following command to check and select in which environment you want to be:
> jx env (and select on of the options)

For importing new projects we will need to be in the **dev** environment.

# Repositories

Repositories [https://github.com/activiti/?q=ttc-](https://github.com/activiti/?q=ttc-)


The main idea to fork the projects is to be able to change them by sending PRs or just pushing to these repositories that will be monitored by Jenkins X.

We will start by forking and cloning the following two projects:
- ttc-dashboard-ui -> Front End
- ttc-infra-gateway -> Gateway

Before cloning anything, we recommend to create a **workshop/** directory somewhere in your laptop/pc.

# Front End

As any application we will end up having a Front End that is going to interact with a bunch of services. This Application will need to know where the services are and we are going to start simply by forking the Github repository, and running the Front End application so we can track when our services are being deployed.

- Fork [http://github.com/activiti/ttc-dashboard-ui](http://github.com/activiti/ttc-dashboard-ui)
- Clone to your local environment inside the **workshop/** directory

## (Optional): You can run the Front End Application locally

- go into the ttc-dashboard-ui directory and execute
  - npm install
  - npm start
  - Edit proxy.conf.json 
``` 
    "target": "YOUR GATEWAY URL GOES HERE",
```
Note: If you don't have node installed, please install it from here https://nodejs.org/en/

Once 'npm start' finishes you can access the application by pointing to [http://localhost:8080](http://localhost:8080)

## Alternative You can import the project to Jenkins X
You can import the project to run it inside the Kubernetes Cluster along your services. 

Just import it into Jenkins X by executing:

 > jx import --branches "develop|PR-.*|feature.*"
 
 
 In order to access to the app you can do 
 > jx get apps 
 
 Then copy the URL from **ttc-dashboard-ui**  . 
 
 You can monitor the pipeline which will build and deploy your project to Kubernetes by opening the Jenkins UI by typing:
 > jx console
 

# Single Entrypoint

As mentioned before our Gateway component will be the single entrypoint for our services' clients. We will start by importing the Gateway project into Jenkins X and then we will add our services which implement our scenario. We want to provide a single entry point and service discovery for all our services in our infrastructure. This Service is a simple Spring Boot 2 application built using the [Spring Cloud Gateway Starter](https://cloud.spring.io/spring-cloud-gateway/) and it is using the [Spring Cloud Kubernetes Discovery](https://github.com/spring-cloud-incubator/spring-cloud-kubernetes) project to figure out which services are being deployed into the Kubernetes namespace where our application live.

- Switch to the Dev environment in Jenkins X by
  > jx env dev

- Fork [http://github.com/activiti/ttc-infra-gateway](http://github.com/activiti/ttc-infra-gateway)
- Clone your fork inside the **workshop/** directory
  > git clone http://github.com/{your user}/ttc-infra-gateway

- Import project into Jenkins X:
  > jx import --branches "develop|PR-.*|feature.*"

- Verify in Jenkins UI that the project was imported and the pipeline is running (jx console)



Once the Pipeline is finished and the service promoted to staging, you should be able to access the following URL (which you can obtain by doing jx get apps)

> curl http://{Gateway App URL}/campaigns

or

> curl "$(jx get apps | grep ttc-infra-gateway | awk '{print $4"/campaigns"}')" | json_pp

Also, you can access to:

> curl http://{Gateway App URL}/actuator/gateway/routes

or

> curl "$(jx get apps | grep ttc-infra-gateway | awk '{print $4"/actuator/gateway/routes"}')" | json_pp

Which shows the available registered services inside the gateway. At this point there shouldn't be any service registered.

You can easily tail the logs of your service by executing:
> jx logs

You can also use **kubectl** to check that your Pods, Deployments and Services are up:
> kubectl get pods

> kubectl get services

> kubectl get deployments

All the Deployments are configured to have a single replica for each service so you should see 1/1 pod started.


# Our First Service

This service emulates an internal service that will connect with an external Social Media feed such as Twitter. We wanted to make sure that we have a Dummy Service to be able to control the feed and the content for testing purposes. This service, consume data from the external stream and push it to RabbitMQ (you can use Kafka, ActiveMQ or other binders) via Spring Cloud Streams for other service to consume each tweet in an Async fashion.


- Fork [http://github.com/activiti/ttc-connectors-dummytwitter](http://github.com/activiti/ttc-connectors-dummytwitter)
- Clone it inside the **workshop/** directory
  > git clone http://github.com/{your user}/ttc-connectors-dummytwitter

- Import into Jenkins X:
  > jx import --branches "develop|PR-.*|feature.*"

- Go to Jenkins X and check that the new project is registered and being built (jx get urls -> to retrieve the URLs again)


The build should fail at this point, and this is because we need to configure Nexus to pick up other Maven repositories to download some dependencies 3rd Party dependencies which are not hosted in Maven Central or Spring Repositories (which are included by default).


## Configuring Nexus

- Get Nexus URL (in **dev** environment)
  > jx get urls

- Login with admin/{jx pass}
- Click in the Server Administration and Configuration menu (Gear)
- Click on Maven2 (Proxy)
- Enter the following information:
  - Name: activiti
  - Version Policy: Mixed
  - URL: https://artifacts.alfresco.com/nexus/content/repositories/activiti-snapshots/
- Scroll Down and Create Repository
- With the repository created you need to add it to the Maven Group (From the list of repositories
- Click on Maven Group
- Select the activiti repository from the Available list and move it to the Members list
- Scroll down and save

Now we should be able to build our service.
- Go back to Jenkins -> {Your User} -> ttc-connectors-dummytwitter -> develop
- Hit Build Now

**Note: While you wait for the pipeline to finish (it takes around 7 minutes ) you can fork and clone the other projects**

From the CLI you can execute:
> jx get activities -w (-> to watch what Jenkins X is doing)

> jx env -> select staging

> kubectl get pods -> you should see your pod and the number of replicas

```
NAME                                                      READY     STATUS    RESTARTS   AGE
jx-staging-ttc-connectors-dummytwitter-{hash}   0/1       Running   3          8m
```

> jx logs -> will tail the logs of our only service and it will reveal that the service is looking for RabbitMQ but rabbitMQ is not available

# Adding Environment Dependencies

A very common scenario is when your service depends on a service provided by the infrastructure, such as a Database or a Message Broker.
For such cases, you will need to setup inside your environment these infrastructural (environment) services. You can do that by using HELM charts.
For RabbitMQ you can search for the HELM chart and use it. In Jenkins X you do this by following a GitOps approach, meaning that changing the environment configuration is done by adding commits to a Git repository.

When you installed Jenkins X, two environments and two repositories where created inside your Github accounts and cloned into your laptop/pc.  
You can find these repositories under:
**~/.jx/environments/{github user}/environment-{env name}-staging**
Let's add RabbitMQ to the staging environment:

> cd ~/.jx/environments/{github user}/environment-{env name}-staging
> git pull -> to retrieve the latest changes from the staging environment

Edit env/requirements.yaml and add append:
```
- name: rabbitmq
  repository: https://kubernetes-charts.storage.googleapis.com
  version: 0.8.0
```

Edit env/values.yaml and append:
```
rabbitmq:
  rabbitmq:
    username: "guest"
    password: "guest"
  ingress:
    enabled: true
```

**Be careful with the spaces (it is a yaml file) and caps**
Now let's apply the changes:

- Check that the files were modified
> git status -> you should see two files changed

- Do:
> git add .

- Then:
> git commit -m “adding rabbitmq to staging env”

- Finally:
> git push

You can check now inside the Jenkins UI that the staging environment pipeline has been triggered


After the Staging Environment pipeline finish, you should be able to check that RabbitMQ is up and running and that the Service is connecting to it:

> jx logs ->  inside the staging environment (> jx env staging)

You should see now something like:
```
2018-06-11 21:29:03.831  INFO [-,,,] 1 --- [           main] o.s.a.r.c.CachingConnectionFactory       : Attempting to connect to: [jx-staging-rabbitmq:5672]
2018-06-11 21:29:04.248  INFO [-,,,] 1 --- [           main] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory#2c0950f2:0/SimpleConnection@7c447c76 [delegate=amqp://guest@10.59.240.82:5672/, localPort= 54806]
2018-06-11 21:29:04.539  INFO [-,,,] 1 --- [           main] o.s.integration.channel.DirectChannel    : Channel 'application-1.runtimeCmdProducer' has 1 subscriber(s).
2018-06-11 21:29:04.552  INFO [-,,,] 1 --- [           main] o.s.integration.channel.DirectChannel    : Channel 'application-1.campaignProducer' has 1 subscriber(s).
```


# Building a Marketing Campaign

This service is implementing the Business Logic which defines how the campaign should behave. This is a pretty simple and generic campaign which is designed to track Social Media content related to the **Activiti** project. You can of course adapt this to any other topic and build more complex campaigns.

This service was designed to be the Command aspect of the CQRS (Command / Query Responsibility Segregation) pattern. This service is in charge of executing business logic for every tweet that we receive.

We are using the Activiti Runtime Process Engine to automate the execution of these business processes. 

![Campaign Business Process](/campaign-bpmn-process.png)


- Fork [http://github.com/activiti/ttc-rb-english-campaign](http://github.com/activiti/ttc-rb-english-campaign)
- Clone it inside the **workshop/** directory
  > git clone http://github.com/{your user}/ttc-rb-english-campaign

- Import it to Jenkins X:
  > jx import --branches "develop|PR-.*|feature.*"

- Check in Jenkins UI that the project was imported and the initial build is triggered

Now that you more than one service
> jx logs

will ask you select which project do you want to tail. Remember always to be in **jx env staging** to see your services.

You can also check that your pod is running:
> kubectl get pods

Check that you have 1/1 replicas running.

By doing
> jx get apps

You can also get the service URL which is, by default, automatically exposed by Jenkins X - expose controller.

You should be able to access both URLs via your Browser.

This service is going to use RabbitMQ as well, but because we already set it up, there is nothing more required for this service to run.


## Time for some tests
> jx get apps

- Copy the URL for the **ttc-connectors-dummytwitter** feed app:
> curl {url}/feed

- Should return false -> meaning that the feed is turned off
> curl -X POST {url}/feed/start

- This will start the feed. If you do
> jx logs -> select -> ttc-connectors-dummytwitter

You will see the twitter feed being consumed and sent via Spring Cloud Streams. Every available Campaign (who matches some filters, for example the language filter) will be able to pick up the tweet for processing.

You can stop the logs with CTRL+C or open another terminal
> jx logs -> select -> ttc-rb-english-campaign

You will see tweets being consumed. Then you can stop the feed while we import more projects:

> curl -X POST {url}/feed/stop


# Consuming Data

Until this point we have a Service that produce data, in this case simulates a social media feed (ttc-connectors-dummytwitter). A service that process this data and defines the business logic on how to perform a marketing campaign (ttc-rb-english-campaign). Now we need a service that aggregates the data generated by multiple campaigns and allows us to consume that data without affecting the performance of each of the individual campaigns.

This service is called Query Service and in order to get it up and running we will follow the same approach as before:
- Fork [http://github.com/activiti/ttc-query-campaign](http://github.com/activiti/ttc-query-campaign)
- Clone it inside the **workshop/** directory
  > git clone http://github.com/{your user}/ttc-query-campaign

- Import it to Jenkins X:
  > jx import --branches "develop|PR-.*|feature.*"

- Check in Jenkins UI that the project was imported and the initial build is triggered

At this point, when the pipeline finishes you should be able to query generic information about how many tweets are being processed (in-flight), how many were discarded and how many matched with the campaign and were ranked.


In-flight processes analyzing tweets
> curl http://{Gateway App URL}/ttc-rb-english-campaign/v1/process-instances/

Domain specific data for the campaign, only the tweets that have matched with the campaign.

> curl http://{Gateway App URL}/ttc-rb-english-campaign/processed/activiti


# System to System Integrations

As in every large scenario you will required some kind of Service Orchestration and Choreography. 
Right now all the Service connectors to external system are not present, so we need to plug them in. 
These microservices represent System to System integrations that will be orchestrated by the Marketing Campaigns buiness needs.
As you will see, when you try to consume data from the query service none of the processes are finishing, because they need other services to complete all the operations:

- Fork Rewards Connector [http://github.com/activiti/ttc-connectors-reward](http://github.com/activiti/ttc-connectors-reward)
- Fork Ranking Connector [http://github.com/activiti/ttc-connectors-ranking](http://github.com/activiti/ttc-connectors-ranking)
- Fork Processing Connector [http://github.com/activiti/ttc-connectors-processing](http://github.com/activiti/ttc-connectors-processing)

Clone all of those projects inside the **workshop/** directory

Import them all to Jenkins X
> jx import --branches "develop|PR-.*|feature.*"

Wait for the pipelines to finish and now you are ready to check the UI, it should show the campaign deployed in the main screen and all the Services should be green. You can also turn on the Dummy Twitter Feed and see how the campaign match, process, rank and reward engaged users.


# Next Steps

- You can add PostgreSQL and bind it to Query Service and Campaigns by uncommenting properties inside those projects
- All Services are already built in with Zipkin for Tracing. So configuring Zipking should be matter of adding the Zipkin Helm Chart
- Single Sign On: maybe using Keycloak Helm Chart
- Monitoring with the ELK Stack, all the services are already including Spring Cloud Sleuth
- Istio Service Mesh and rolling upgrades for campaigns
- Spring Application Monitoring with Spring Cloud Kubernetes integration
- Reactive UI with Webflux in the Query Service (half implemented already)
- Adding Activiti Query GraphQL integration



