# Introduction to Docker and Kubernetes Hands-On Workshop

## Working Environment

### Cloud9
We have provisioned a Cloud9 instance and its browser-based IDE and terminal for each of you. This is a web interface to a Linux machine with Docker installed and ready to go. Being Linux, containers run right on these Cloud9 instances when you use the docker commands.

TODO - Add instructions/screenshots to launch the Cloud9

### Kubernetes
We have provisioned a kubernetes cluster based on AWS' EKS with a namespace for each of you and your Cloud9 is set up with the right config and credentials where it is ready to go. Using the `kubectl` command will 'just work' in the examples below. We'll be covering how to set up Kubernetes and how things work under-the-hood in another session soon.

## The Workshop

### Docker Basics
The first part of this workshop will focus on the fundamentals of Docker and how to use it locally within one machine.

1. Type `docker version` to confirm that both the client and server are there and working.
1. Type `docker pull nginx:latest` to pull down the latest nginx trusted image from Docker Hub.
1. Type `docker images` to verify that the image is now on your local machine's Docker cache. If we start it then it won't have to pull it down from Docker Hub first.
1. Type `docker run –d –p 8080:80 --name nginx nginx:latest` to instantiate the nginx image as a background daemon with port 8080 on the host forwarding through to port 80 within the container
1. Type `docker ps` to see that our nginx container is running.
1. Type `curl http://localhost:8080` to use the nginx container and verify it is working with its default `index.html`.
1. Type `docker logs nginx` to see the logs produced by nginx and the container from our loading a page from it.
1. Type `docker exec -it nginx /bin/bash` for an interactive shell into the container's filesystem and constraints
1. Type `cd /usr/share/nginx/html` and `cat index.html` to see the content the nginx is serving which is part of the container.
1. Type `exit` to exit our shell within the container.
1. Type `docker stop nginx` to stop the container.
1. Type `docker ps -a` to see that our container is still there but stopped. At this point it could be restarted with a `docker start nginx` if we wanted.
1. Type `docker rm nginx` to remove the container from our machine
1. Type `docker rmi nginx:latest` to remove the nginx image from our machine's local cache
1. Type `cd docker-kube-intro-workshop` to change into that project.
1. Type `docker build -t nginx:1.0 .` to build nginx from our Dockerfile
1. Type `docker history nginx:1.0`. to see all the steps and base containers that our nginx:1.0 is built on. Note that our change amounts to one new tiny layer on top.
1. Type `docker run -p 8080:80 --name nginx nginx:1.0` to run our new container. Note that we didn't specify the `-d` to make it a daemon which means it holds control of our terminal and outputs the containers logs to there which can be handy in debugging.
1. Type `curl http://localhost:8080` in another terminal tab a few times and see our new content as well as the log lines in our origional terminal. 
1. At this point we could push it to Docker Hub or a private Registry like AWS' ECR for others to pull and run. We won't worry about that yet and will cover it in the next Kubernetes session.
1. Type Ctrl-C to exit the log output. Note that the container is still running though if you do a `docker ps`.
1. Type `sudo docker inspect nginx` to see lots of info about our running container.
1. Type `docker stop nginx` to shut our container down.

### Kubernetes Basics
This part of the workshop will focus on how to deploy and machine our containers on a pool of machines managed by Kuberenetes.

1. Type `kubectl version` to confirm that both the client and server are there and working.
1. Type `kubectl create deployment nginx --image=nginx` to create a single-Pod deployment of nginx.
1. Type `kubectl describe deployments` to see the details of our new deployment.
1. Type `kubectl describe pods` to see the pod that our deployment has created for us. Note that there is no port exposed for this contianer.
1. Type `curl http://<IP from describe pods>` and see that it times out loading the page.
1. Type `kubectl edit deployment nginx` to edit the YAML definition of our deployment.
    1. Add the containerPort mapping under spec/containers/ports as follows and save the file.
    ````yaml
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
    ````
1. Type `curl http://<IP from describe pods>` and see that it now loads the page.
    1. In AWS' Kuberenetes EKS the Pods all get 'real' VPC IP addresses and your Cloud9 is in that VPC so can reach the pods directly. In other Kubernetes implementations the IPs might not be reachable directly and you might need to go through a port mapped to the host or other such method.
1. Type `kubectl scale --replicas=3 deployment/nginx` to launch two more nginx Pods taking us to three.
1. Type `kubectl describe deployments` and `kubectl describe pods` to see our change has taken effect.
1. Type `kubectl expose deployment nginx --port=80 --target-port=80 --name nginx --type=LoadBalancer` to create a service backed by an AWS Elastic Load Balancer that not only balances the load between all the Pods but exposes it to the Internet.
1. Type `kubectl describe services` and copy the `LoadBalancer Ingress` address. Open a web browser tab and go to `http://<that address>` and see it load. Note that it will take a minute or so for the ELB to be provisioned before this will work.
1. Type `kubectl logs -lapp=nginx` to get the aggregated logs of all the nginx Pods to see the details of our recent requests.
1. Type `kubectl get service/nginx deployment/nginx --export=true -o yaml > nginx.yml` to back up our work.
    1. Edit the file and have a look. We could have written our requirements for what we need Kubernetes to do into YAML files like this in the first place instead of using kubectl to create them - and often you would do that and put it in a CI/CD pipeline etc.
1. Type `kubectl delete service/nginx deployment/nginx` to clean up what we've done.
1. Type `kubectl apply -f nginx.yml` to put it back again. You can put the definitions for multiple types of things (in this example a service and a deployment) in one YAML file.