---
title: Basics of Kubernetes (Hands-On Experience)
date: 2021-05-05 06:46:00 +0100
categories: [DevSecOps, Kubernetes]
tags: [devsecops, kubernetes, containers]
lang: en
---

## kubectl
A command-line tool to access the K8s API Server

### Installation

[Here're the docs to guide you with installation](https://kubernetes.io/docs/tasks/tools/)

On my host machine, I only run
  ```sudo pacman -Syu kubectl```

### Installing Kubernetes itself
We have 2 choices here
- Docker Desktop - Easier one, however, **only Windows and Mac are suppoerted** :/
- minicube - More difficult than Docker Desktop, but I think It's better to struggle a bit but get more accurate, command line experience

If you want to use **Docker-Desktop** because you're on Mac or Windows, then to install it head over to [HERE](https://www.docker.com/products/docker-desktop) and download .exe file

**minicube** - Head over [HERE](https://minikube.sigs.k8s.io/docs/start/) and proceed from their documentation

## Managing Pods
### Process Overview:
1. Build container image out of the app,
2. Store it in the container registry (e.g. Docker Hub)
3. Create Manifest file in ```.yml``` format that would define the Pod's structure
4. Send it to the K8s API Server

We'll focus on the 3 and 4 step, because we already have containerized apps - by it, I mean even the simplest ones, like ```mongodb``` container — that is also a container app!

This → [repo](https://github.com/kubernetes/examples) contains a lot of more examples to work on with Kubernetes

I'll be choosing ```Guestbook```. And because I'm also a learner just like you, I'll do everything as described in Kubernetes own [Tutorial/Guide](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/) from now on.


## Guestbook

### Starting minikube cluster
With ```minikube start```

Then, if we run ```kubectl get po -A```
```
NAMESPACE              NAME                                        READY   STATUS    RESTARTS   AGE
kube-system            coredns-74ff55c5b-87864                     1/1     Running   1          23d
kube-system            etcd-minikube                               1/1     Running   1          23d
kube-system            kube-apiserver-minikube                     1/1     Running   1          23d
kube-system            kube-controller-manager-minikube            1/1     Running   1          23d
kube-system            kube-proxy-bzqzx                            1/1     Running   1          23d
kube-system            kube-scheduler-minikube                     1/1     Running   1          23d
kube-system            storage-provisioner                         1/1     Running   2          23d
kubernetes-dashboard   dashboard-metrics-scraper-f6647bd8c-6tz59   1/1     Running   0          6m18s
kubernetes-dashboard   kubernetes-dashboard-968bcb79-2226c         1/1     Running   0          6m18s

```

We'll see that there are various Kubernetes objects running like the ```etcd```, ```kube-apiserver-minikube```, ```storage-provisioner```.

Many of them we discussed in previous blog-post related to Kubernetes.

So as discussed in the kubernetes tutorial We need to create a Manifest of the **MongoDB Deployment Controller**

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  labels:
    app.kubernetes.io/name: mongo
    app.kubernetes.io/component: backend
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: mongo
      app.kubernetes.io/component: backend
  replicas: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mongo
        app.kubernetes.io/component: backend
    spec:
      containers:
      - name: mongo
        image: mongo:4.2
        args:
          - --bind_ip
          - 0.0.0.0
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 27017

```
And write it to the ```deployment.yml```,
Then execute:

```kubectl apply -f mongo-deployment.yaml```

We'll get the output:

```
deployment.apps/mongo created
```
Then to get the Pod We've created run
```kubectl get pods```
```
NAME                     READY   STATUS    RESTARTS   AGE
mongo-75f59d57f4-8kjp2   1/1     Running   0          93s
```
So our Deployment Controller is running!

Now Lets create the **MongoDB Service**, with very similar steps:
1. Copying this yml content:
```yml
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    app.kubernetes.io/name: mongo
    app.kubernetes.io/component: backend
spec:
  ports:
  - port: 27017
    targetPort: 27017
  selector:
    app.kubernetes.io/name: mongo
    app.kubernetes.io/component: backend

```
into mongo-service.yml file inside your App directory

2. Apply it with
```kubectl apply -f mongo-service-yml```

And We'll **do the same** with **Guestbook Frontend Deployment** and **Guestbook Frontend Service** just like the [tutorial](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/) says

And verify if replicas were made with:

```kubectl get pods -l app.kubernetes.io/name=guestbook -l app.kubernetes.io/component=frontend```

And If you've done everything the docs specified, your WebApp will be running

![guesbook-frontend](https://imgur.com/LZqCIkl.png)

>I know that there were little to no work from my part, looks like these docs have everything explained, which is amazing!

### Conclusions with Guestbook
- Writing Manifests in ```.yml``` format is essential, that's basically the main part of configuring the services in K8s

