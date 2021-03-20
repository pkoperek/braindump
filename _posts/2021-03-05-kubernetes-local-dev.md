---
layout: post
title: Kubernetes testing environments
---

Testing deployment pipelines, services setup etc in kubernetes is quite a complex task. The first instinct to start playing around the configuration in a safe way is to setup a new cluster in your cloud-vendor-of-choice environment. However, that turns out to be a pricey (and actually quite sluggish) solution usually. As a second thought, you will probably realize that testing out locally might be a better option (its cheaper, faster, probably nicely isolated etc). Below are my few notes on tools which should enable testing locally of kubernetes stuff.

* [k3s](https://k3s.io/)
  * this isn't a test environment per se, it is just a pretty compact k3s distribution which is supposed to even work at the smaller devices
  * [installation](https://rancher.com/docs/k3s/latest/en/installation/install-options/)
    * you can choose the use the script: `curl -sfL https://get.k3s.io | sh -` however this is an option if you want to run k8s as a service on the node where you run the command
        * the script is dedicated for systemd / openrc systems
    * you can also just download the binary (from [here](https://github.com/k3s-io/k3s/releases)) and run it from commandline
  * batteries includes: has traefik, helm controller, flannel etc embedded

* k3d
  * [Wrapper/tool build around k3s](https://en.sokube.ch/post/k3s-k3d-k8s-a-new-perfect-match-for-dev-and-test)
  * Tying it all together with Argo CD for a GitOps pipeline: [link](https://en.sokube.ch/post/gitops-on-a-laptop-with-k3d-and-argocd)

* kind
  * ```kind is a tool for running local Kubernetes clusters using Docker container “nodes”. kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.```
    * [source](https://kind.sigs.k8s.io/)
  * has interesting features which allow it to be easily used in k8s development (e.g. you can pass your own k8s image when creating the cluster)

* minikube
  * sets up a docker cluster (or in a VM)
  * works on Windows/Mac/Linux (because the cluster can live within the VM)
  * adding typically included pieces of setup is done through add-ons
  * ~~seems to be a bit heavier than kind~~ not really - the docker version is quite snappy
  * [great tutorial](https://kubernetes.io/docs/tutorials/hello-minikube/)
  * supports multi-node setups: `minikube start --nodes 2`

* microk8s
  * made by canonical (yup, the ubuntu folks)
  ```
  MicroK8s is the smallest, fastest, fully-conformant Kubernetes that tracks upstream releases and makes clustering trivial. MicroK8s is great for offline development, prototyping, and testing. Use it on a VM as a small, cheap, reliable k8s for CI/CD. It’s also the best production grade Kubernetes for appliances. Develop IoT apps for k8s and deploy them to MicroK8s on your Linux boxes.
  ```
  * seems to be using a different deamon as a controller for containers (snapd instead of containerd) - but can't find a good confirmation for that information (apart from the presentation below)
  * https://microk8s.io/docs
  * seems to position itself similarly to k3s as a production-ready plaform and a potential dev env in the same time
  * more closely tracks the main kubernetes development
  * nice write-up about minikube vs microk8s: [blog link](https://kubernetes.io/blog/2019/11/26/running-kubernetes-locally-on-linux-with-microk8s/)
  * seems to be limited to Ubuntu (and those few platforms which support snap format)

Finally, as a summary:
* [a good presentation on the topic](https://www.cncf.io/wp-content/uploads/2020/08/CNCF-Webinar-Navigating-the-Sea-of-Local-Clusters-.pdf)
* [comparison by tilt](https://docs.tilt.dev/choosing_clusters.html)
