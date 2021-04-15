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
- minicube - More difficult than Docker Desktop but I think It's better to struggle a bit but get more accurate, command line experience

If you want to use **Docker-Desktop** because you're on Mac or Windows, then to install it head over to [HERE](https://www.docker.com/products/docker-desktop) and download .exe file

**minicube** - Head over [HERE](https://minikube.sigs.k8s.io/docs/start/) and proceed from their documentation
