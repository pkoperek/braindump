---
layout: post
title: extending kubernetes with use of the Operator pattern
comments: true
---

Sometimes even the best pieces of software are not good enough and require
customizing. Or you are looking for introducing some automation to frequently
executed actions. The *Operator* pattern is kubernetes's answer to that.
It shows how to introduce your own management logic into a nicely functioning
cluster and not lose your mind in the process :) Below are my notes from
creating a sample controller (code available here:
https://github.com/pkoperek/kubernetes-custom-resource).

## Initial notes

* "Control plane" is a set of services which manage the cluster: [source](https://kubernetes.io/docs/concepts/overview/components/#control-plane-components)
* Kubernetes can be extended by adding CustomResourceDefinitions or Aggregated APIS
* A CRD can be created simply by creating another object in k8s (you don't have to programm anything to get it setup) but then it doesn't do anything apart from storing some structured data in k8s API
* "On their own, custom resources let you store and retrieve structured data. When you combine a custom resource with a custom controller, custom resources provide a true declarative API."
  * [source](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#custom-controllers)
* Relationship between CDR and custom controllers is described by the "Operator pattern"
  * [source](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)
* What is the operator pattern?
  * "Operator" code defines what a human operator would do when they are managing a particular app
  * operators are  tightly coupled to the specific application
  * they should maximally utilize existing k8s resources for improved stability
  * "Operator" code is actually written in form of a custom controller
  * [original article about the Operator pattern](https://web.archive.org/web/20170129131616/https://coreos.com/blog/introducing-operators.html)
  * as per [this](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) Operator typically is:

>  What might an Operator look like in more detail? Here's an example in more detail:
>
>  1. A custom resource named SampleDB, that you can configure into the cluster.
>  2. A Deployment that makes sure a Pod is running that contains the controller part of the operator.
>  3. A container image of the operator code.
>  4. Controller code that queries the control plane to find out what SampleDB resources are configured.
>  5. The core of the Operator is code to tell the API server how to make reality match the configured resources.
>    * If you add a new SampleDB, the operator sets up PersistentVolumeClaims to provide durable database storage, a StatefulSet to run SampleDB and a Job to handle initial configuration.
>    * If you delete it, the Operator takes a snapshot, then makes sure that the StatefulSet and Volumes are also removed.
>  6. The operator also manages regular database backups. For each SampleDB resource, the operator determines when to create a Pod that can connect to the database and take backups. These Pods would rely on a ConfigMap and / or a Secret that has database connection details and credentials.
>  7. Because the Operator aims to provide robust automation for the resource it manages, there would be additional supporting code. For this example, code checks to see if the database is running an old version and, if so, creates Job objects that upgrade it for you.

### So how does all of that look in practice?

In general the operator pattern involves creating two components: a Custom Resource Definition and a controller which is going to handle its management.

#### How to create a CRD?

* Creating CRD is relatively simple: you just need to specify another kubernetes resource - but this time it is a bit "meta" as the resource defines other resources
  * [source](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)
  * create a file with the definition, e.g.
  ```
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
      # name must match the spec fields below, and be in the form: <plural>.<group>
      name: databases.stable.imaginedata.com
    spec:
      # group name to use for REST API: /apis/<group>/<version>
      group: stable.imaginedata.com
      # list of versions supported by this CustomResourceDefinition
      versions:
        - name: v1
          # Each version can be enabled/disabled by Served flag.
          served: true
          # One and only one version must be marked as the storage version.
          storage: true
          schema:
            openAPIV3Schema:
              type: object
              properties:
                spec:
                  type: object
                  properties:
                    name:
                      type: string
                    port:
                      type: integer
                    storage-size:
                      type: string
                    image:
                      type: string
                    replicas:
                      type: integer
      # either Namespaced or Cluster
      scope: Namespaced
      names:
        # plural name to be used in the URL: /apis/<group>/<version>/<plural>
        plural: databases
        # singular name to be used as an alias on the CLI and for display
        singular: database
        # kind is normally the CamelCased singular type. Your resource manifests use this.
        kind: Database
        # shortNames allow shorter string to match your resource on the CLI
        shortNames:
        - db
  ```
* You can install it in your cluster by simply running: `kubectl apply -f crd.yml`
* Initially there will be no resources of that kind:
  ```
  $ kubectl get databases
  No resources found in default namespace.
  ```
* You can create one by simply applying a resource definition:
  ```
  $ cat sample-object.yml
  apiVersion: "stable.imaginedata.co/v1"
  kind: Database
  metadata:
    name: my-first-crd-database
    namespace: default
  spec:
    name: name-of-database
    port: 5432
    storage-size: 50G
    image: super-great-docker-image
    replicas: 3
  $ kubectl apply -f sample-object.yml
  database.stable.imaginedata.co/my-first-crd-database created
  $ kubectl get db
  NAME                    AGE
  my-first-crd-database   5s
  ```

One additional topic which I have omitted was to [create a structured schema with validation definitions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/). If you are developing a production system, you probably want to have a look at it.

#### How to implement the custom controller?

Finally we can get into implementing a custom controller. First, some general info about controllers.

```A controller is a client of Kubernetes. When Kubernetes is the client and calls out to a remote service, it is called a Webhook. The remote service is called a Webhook Backend. Like Controllers, Webhooks do add a point of failure.```
[source](https://kubernetes.io/docs/concepts/extend-kubernetes/extend-cluster/#extension-patterns)

```There is a specific pattern for writing client programs that work well with Kubernetes called the Controller pattern. Controllers typically read an object's .spec, possibly do things, and then update the object's .status.```.
[source](https://kubernetes.io/docs/concepts/extend-kubernetes/extend-cluster/#extension-patterns)

How is a controller associated with CRD? Controller watches constantly on the queue of resources of a given kind and reconciles the state of cluster to fulfill the requirements of the resource. It is up to the controller to schedule any required changes (e.g. introducing a new set of pods if we are scaling something).

Building a controller involves the following steps:

* Generating code stubs (if you are working with a staticly typed language). For Java I have followed
[these steps](https://github.com/kubernetes-client/java/blob/94c1e042aa423e45aeb1c489babececa18632c1e/docs/generate-model-from-third-party-resources.md)
* Implementing the logic (a good starting point is [this](https://kubernetes.io/blog/2019/11/26/develop-a-kubernetes-controller-in-java/))
* Deploying the controller to kubernetes as a pod. The controller can run e.g. as a Deployment.

To write a controller you need to use one of the client libraries: [list](https://kubernetes.io/docs/reference/using-api/client-libraries/) ... or use one of the frameworks which are suppoed to make it easier :)

* [https://operatorframework.io/](https://operatorframework.io/)
* [https://kudo.dev/](https://kudo.dev/)
* [https://metacontroller.github.io/metacontroller/guide/create.html](https://metacontroller.github.io/metacontroller/guide/create.html)
* [https://book.kubebuilder.io/](https://book.kubebuilder.io/)

### Links

Below are links to some additional resources and examples I have been using.

* [https://kubebuilder.io](https://kubebuilder.io)
* [https://github.com/kubernetes/sample-controller](https://kubebuilder.io)
* [https://kubebuilder.io/introduction.html](https://kubebuilder.io/introduction.html)
* [https://kubernetes.io/docs/concepts/extend-kubernetes/](https://kubernetes.io/docs/concepts/extend-kubernetes/)
* [https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
* [https://github.com/kubernetes-client/java/blob/master/examples/examples-release-12/src/main/java/io/kubernetes/client/examples/KubeConfigFileClientExample.java](https://github.com/kubernetes-client/java/blob/master/examples/examples-release-12/src/main/java/io/kubernetes/client/examples/KubeConfigFileClientExample.java)
* [https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/)
