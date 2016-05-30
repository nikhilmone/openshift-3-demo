##OpenShift 3 demonstration

# Usecase 1 : There is an existing application code in your github/gitlab repo and you want to deploy it on your choice of application server(JBoss EAP) running on OpenShift.

Source-To-Image (S2I) is a standalone tool which is very useful when creating builder images. As the name suggests we will be taking the source code from our git repository and will create a docker image using it. Once image is created we can assign this image to a container, which would be running the version of the code which we fed to S2I.

OpenShift Allows to create applications wither using `oc` command line utility or directly through the management console. First it requires a project to be created, which contains the applications, services and routes. 

Here is how you can create a project and an application using `oc` utility.

```
  oc login <master server url> -u <username> -p <password>
  oc new-project demo
  oc new-app eap64-basic-s2i \
        --param=APPLICATION_NAME=ticket-monster \
        --param=SOURCE_REPOSITORY_URL=https://github.com/nikhilmone/ticket-monster.git \
        --param=SOURCE_REPOSITORY_REF=2.7.0.Final \
        --param=CONTEXT_DIR=demo
```

You can also create the project and application from thre management console, as shown below :

Login to management console with your credentials and create a project.

<img width="1440" alt="screen shot 2016-05-28 at 10 33 52 am" src="https://cloud.githubusercontent.com/assets/1744307/15631217/f66629ce-257a-11e6-98b1-73b9515a56d6.png">

Now add an application to your newly created project. You see below templates which you can choose from as your base image:

<img width="1440" alt="screen shot 2016-05-28 at 10 36 26 am" src="https://cloud.githubusercontent.com/assets/1744307/15631236/0196ab4c-257c-11e6-9a7f-bf2f8cc32a65.png">

I chose `jboss-eap64-openshift:1.1`, since I want to deploy a J2EE application on JBoss EAP 6.4, which I have aleady created.
[](https://github.com/nikhilmone/jboss-eap-quickstarts.git). Then I choose branch name as `6.4.x` and the sub-directory as `kitchensink`. These two things are completely optional and you can skip this if your git repository is not too complex.

<img width="1440" alt="screen shot 2016-05-28 at 5 41 03 pm" src="https://cloud.githubusercontent.com/assets/1744307/15629659/79f11860-253c-11e6-8578-17d819ab4a7b.png">

Click on advanced and configure your routing, deployment and build config. I have kept all the options default as I want to expose my application on public internet and want to trigger build and deployment after a code push.

<img width="1440" alt="screen shot 2016-05-28 at 5 42 09 pm" src="https://cloud.githubusercontent.com/assets/1744307/15629661/81ae6bfc-253c-11e6-9361-8f159daf4c33.png">

Submit the information above and a new application is created. In this page you can see a `git webhook` that you can set in your github repo. So that as soon as you push the new code to git repo, a rest call will be made to your master server and a new build will e triggered, which eventually triggers a new deployment as well.

<img width="1440" alt="screen shot 2016-05-28 at 5 43 42 pm" src="https://cloud.githubusercontent.com/assets/1744307/15629666/9682b6dc-253c-11e6-96bb-29aad0b4976e.png">

Let's click on `Go to overview` link and it takes you to this page. Here you can see a build is triggered.

<img width="1440" alt="screen shot 2016-05-28 at 5 48 34 pm" src="https://cloud.githubusercontent.com/assets/1744307/15629667/a6602080-253c-11e6-9d50-779ef563779e.png">

Click on `view logs`

<img width="1440" alt="screen shot 2016-05-28 at 5 49 40 pm" src="https://cloud.githubusercontent.com/assets/1744307/15629669/c1028dce-253c-11e6-81d8-6af71410c565.png">

You can also tail the logs from `oc` command line utility. Simply do :

```
oc get pods
oc logs <pod name>
```
<img width="1440" alt="screen shot 2016-05-28 at 6 13 58 pm" src="https://cloud.githubusercontent.com/assets/1744307/15629672/d8dfe55e-253c-11e6-9955-83a35db1a95a.png">

Now the build is successful and finished. Let's run `oc get pods` again and we can see OpenShift has triggered a deployment. You can tail the logs again if you wish.

<img width="526" alt="screen shot 2016-05-28 at 6 15 34 pm" src="https://cloud.githubusercontent.com/assets/1744307/15629678/f456ca14-253c-11e6-9602-43e13de49cd2.png">

Let's go to overview page again. We can see the application is deployed and has been exposed as service via a route.

<img width="1440" alt="screen shot 2016-05-28 at 6 17 38 pm" src="https://cloud.githubusercontent.com/assets/1744307/15629683/059d4bc2-253d-11e6-9bfe-0e5905d7b3c1.png">

Let's click on the service url and it takes us to the application landing page.

<img width="1440" alt="screen shot 2016-05-28 at 6 18 14 pm" src="https://cloud.githubusercontent.com/assets/1744307/15629685/11bb6268-253d-11e6-890e-a0ef4167b7f8.png">

Done. You have a application running in cloud !! :)



#Usecase 2 : You already have an application running on OpenShift and want to perform incremental builds via Git webhooks.

1) Developers tend to immidiately test their latest codebase by quickly deploying developer branch to the server
2) OR once the changes are verified and merged to the master, you immidiately want to push the code to the server

This is easy with OpenShift 3. As soon as you create an application a git webhook is generated (unless you have disabled it explicitly) and you can add it to your github repository.

Navigate to your application's builds and copy the git web-hook url :

<img width="1440" alt="screen shot 2016-05-29 at 10 39 37 am" src="https://cloud.githubusercontent.com/assets/1744307/15631467/b20de172-2589-11e6-83d1-259930ef171d.png">

Login to your github/gitlab repo and navigate to settings:

<img width="1440" alt="screen shot 2016-05-28 at 5 43 55 pm" src="https://cloud.githubusercontent.com/assets/1744307/15631431/39a29c56-2588-11e6-9756-0c7f35c2f6f7.png">

Click on `Add Webhook` and paste the url, which we figured from our application's build settings, somrthing like below : 

https://master.openshift.com:8443/oapi/v1/namespaces/development/buildconfigs/kitchensink/webhooks/d4aeaa1fd59beab2/github

<img width="1440" alt="screen shot 2016-05-28 at 5 45 07 pm" src="https://cloud.githubusercontent.com/assets/1744307/15631501/efb00504-258a-11e6-8ebe-c9c631f07a7f.png">

Keep remaining options as default. I have `Disabled SSL verification` for my application. Save the settings and that's it.

Here is my current index page :

<img width="1440" alt="screen shot 2016-05-28 at 6 18 14 pm" src="https://cloud.githubusercontent.com/assets/1744307/15631523/d95027d4-258b-11e6-9d6c-ef8b8f7b028f.png">


Edit your code and push to your git repository. Immidiately GitHub sends a REST call to OpenShift Master Server notifying about the event and a new build is triggered.

You can verify the new build triggered from management console :

<img width="1440" alt="screen shot 2016-05-28 at 6 26 22 pm" src="https://cloud.githubusercontent.com/assets/1744307/15631505/2819ed38-258b-11e6-9d88-7c378704bed1.png">

Click on `view logs` if you wish to.

Once build and deployment is over, you can see the changes :

<img width="1440" alt="screen shot 2016-05-28 at 6 30 33 pm" src="https://cloud.githubusercontent.com/assets/1744307/15631525/1075a946-258c-11e6-91e8-cbb80af7fd9f.png">

Remember, I have configured my app to use `master` branch as Source for S2I build. You can have multiple branches like dev/qa etc. Once they are merged with master the build gets triggered automatically and then deploys the latest version.



#Usecase 3 : Use Jenkins for Continuous Integration and Continuos Deployment. 

Use Jenkins image for CI/CD flow. Follow below steps :

```
 oc new-project ci
 oc new-app library/jenkins:2.0
 oc expose svc jenkins
```

Login to OpenShift Management Console and you see the Jenkins app created and the route for thr same.

<img width="1440" alt="screen shot 2016-05-28 at 6 41 33 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655407/8c00e966-26ba-11e6-9a7b-e293cc4f1512.png">

Tail the logs using `oc get pods` and `oc logs <pod-name>`.

<img width="1440" alt="screen shot 2016-05-28 at 6 42 46 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655413/98da8304-26ba-11e6-8611-c22b12a159cd.png">

Save the seceret as shown in the image for login purpose. Then install the necessary plugins for creating CD/CI flow.


<img width="1440" alt="screen shot 2016-05-28 at 6 46 23 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655419/ad9810ae-26ba-11e6-9e53-1abac6c5aa75.png">

OpenShift Pipeline Plugin is an important one here :

<img width="1440" alt="screen shot 2016-05-28 at 6 52 05 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655434/c667c1d8-26ba-11e6-8e75-b7d5c7979a85.png">


Installing those plugin are also the builds running of jenkins. Fascinating isn't it :)

<img width="1440" alt="screen shot 2016-05-28 at 6 46 32 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655444/e551e0ec-26ba-11e6-889b-6256978d92ef.png">

Once done quickly create an admin user

<img width="1440" alt="screen shot 2016-05-28 at 6 50 35 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655498/5eae07a4-26bb-11e6-80a8-99b38d67c829.png">


And then login to Jenkins UI

<img width="1440" alt="screen shot 2016-05-28 at 6 51 10 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655515/7f40678c-26bb-11e6-99c8-464b6524d987.png">

Create a Pipeline with below groovy script :

<img width="1440" alt="screen shot 2016-05-28 at 7 13 34 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655533/9af130e2-26bb-11e6-884f-39b3661ddded.png">

Start the build and tail the logs :

<img width="1440" alt="screen shot 2016-05-28 at 7 11 03 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655550/b7fd9072-26bb-11e6-8934-5b601bef301e.png">

Check the pipeline building on Jenkins server :

<img width="1074" alt="screen shot 2016-05-30 at 10 49 10 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655563/e339d714-26bb-11e6-92eb-9ab814476430.png">

The application is up and running :

<img width="1440" alt="screen shot 2016-05-30 at 11 21 58 pm" src="https://cloud.githubusercontent.com/assets/1744307/15655745/6128bfc2-26bd-11e6-8ec7-6b75d5d415d4.png">

Jenkins can also monitor the git repository branches and trigger the builds as needed and execute the test cases if any.

