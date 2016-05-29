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




