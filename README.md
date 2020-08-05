# Kubernetes IBM Cloud guide

At this point, you should have all of the appropiate CLI tools installed.
[Reference this guide to double check](https://www.ibm.com/cloud/architecture/tutorials/microservices-app-on-kubernetes?task=3).

This guide is an altered version of [IBM's official guide](https://www.ibm.com/cloud/architecture/tutorials/microservices-app-on-kubernetes), tailored for Tech Talent South. 

Here are the steps you need to take in order to utilize your cloud cluster.

## 1. Install Docker ## 

Make sure you have Docker installed on your system. [Go here for the install.](https://docs.docker.com/get-docker/)

## 2. Login and Setup ##

Make sure you're logged in before you continue and have used your api key. The login command will look something like this (note that your values will be different):

```

ibmcloud login -a cloud.ibm.com -r us-east --apikey d7c8a8762ad22ac5161f3a74170c2017

```

Run the following command using the id for your cluster (again, your value will be different):

```

ibmcloud ks cluster config --cluster bsladm9d055ugh4ehitg

```


You will need to make sure you target a resource group. This could be either `Default` or `default`:

```

ibmcloud target -g Default

```

or 

```

ibmcloud target -g default

```

## 3. Create a Starter App ##


Enter the following into the console and follow the prompts:

```

ibmcloud dev create

```

You may select whatever app you like but lets try a Node app. Perform the following steps:

1. Select `Backend Service / Web App > Node> Node.js Express App`.

2. Enter a unique name for your application such as `<your-initials>kubeapp`.

3. Select the resource group where your cluster has been created. This might happen automatically.

4. Do not add additional services.
Do not add a DevOps toolchain, select manual deployment.

5. Choose "Deploy to Helm-based Kubernetes containers"


## 4. Build your App ##

Let's go ahead and build our app using Docker.

Define an environment variable named MYPROJECT set with the name of the application you generated in the previous section:

```

export MYPROJECT=<your-initials>kubeapp

```

Go into your generated project's directory: 

```

cd $MYPROJECT && ls

```

Now, build your application using the following command: 

```

ibmcloud dev build --use-root-user-tools

```

Note that this command failed for me, so I had to use the Docker commands in order to build. Refer to Docker's documentation for more info! 

You can now run your application by doing the following:

```

ibmcloud dev run

```

Your app should now be running and can be reached by going to `localhost:3000` if you created a Node application.

## 5. Create a Docker account ##

Docker allows you to push Docker images to a public registry known as DockerHub. Go to [hub.docker.com](https://hub.docker.com) and create an account.

Once your account is created run the following to set your namespace:

```

export MYNAMESPACE=<REGISTRY_NAMESPACE>

```

Your namespace will be in reference to your username on DockerHub.

Using Docker, you may now build and tag your application like so: 

```

docker build . -t ${MYNAMESPACE}/${MYPROJECT}:v1.0.0

```

Now, login to your Docker account:

```

docker login

```

You can then push your image to the public registry:

```

docker push ${MYNAMESPACE}/${MYPROJECT}:v1.0.0

```

## 6. Deployment ##

We can finally deploy our application using Helm. Recall that helm utilizes "charts." These can be thought of as packages like on npm.

First, we need to change directories:

```

cd chart/$MYPROJECT

```

install the chart: 

```

helm install ${MYPROJECT} . --set image.repository=${MYNAMESPACE}/${MYPROJECT}

```

## 7. View your App ##

You can list the Kubernetes services in the namespace like so:

```

kubectl get services

```

You will now need to locate the service linked to your app. Keep in mind that hypens may have been removed in during the previous process. 

Locate your port number tied to your service under `PORT(S)`. 

You can now identify a public IP of a worker node like so:

```

ibmcloud ks workers --cluster cldlabs-iks-iuhawl

```

Finally, go the url formatted like so to access your application: `Access the application at http://worker-ip-address:portnumber/`