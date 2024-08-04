# Installing Kubernetes using Ansible Playbooks
Kubernetes is a frustrating topic to learn. It is the Linux of distributed
computing. That is to say it provides you with a kernel of a distributed
Operating System but leaves everything else up to you to decide. Everything from
basic PodNetworking substrate to LoadBalancer can be different from one vendor
to another.

On top of this, managed Kubernetes solution like GKE, AKS, etc just abstract
away a lot of details. And add their own special sauce like VPCs for
Podnetworking, Block storage for persistent data from stateful apps, and
LoadBalancers for exposing services.

This is good for production, bad for learning...and worse for debugging.

So, I decided to create my own "vanilla" Kubernetes cluster using Kubernetes'
Kubeadm. This cluster is made of a bunch of VMs in the cloud with just a
single public IP on each of them. `kubeadm init` is used to initialize the
control plane and `kubeadm join` is used to connect the worker nodes to the
cluster. For Pod Networking I will be using Flannel, but other alternatives
should work equally well.

## The Goal
The Goal of this post is to show how you can automate

## Spinning Up Cloud VMs
