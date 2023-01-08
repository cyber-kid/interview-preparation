# k8s API Objects

* [Basic Objects: Pods, ReplicaSets, and Services](#basic-objects-pods-replicasets-and-services)
* [Storage: Persistent Volumes, ConfigMaps, and Secrets](#storage-persistent-volumes-configmaps-and-secrets)
* [Organizing Your Cluster with Namespaces, Labels, and Annotations](#organizing-your-cluster-with-namespaces-labels-and-annotations)
* [Advanced Concepts: Deployments, Ingress, and StatefulSets](#advanced-concepts-deployments-ingress-and-statefulsets)
* [Batch Workloads: Job and ScheduledJob](#batch-workloads-job-and-scheduledjob)
* [Cluster Agents and Utilities: DaemonSets](#cluster-agents-and-utilities-daemonsets)

### Basic Objects: Pods, ReplicaSets, and Services
#### Pods
A Pod is the atomic unit of scheduling in a Kubernetes cluster. A Pod is made up of a collection of one or more running containers. (A Pod is a collection of whales, derived from Docker’s whale logo.) When we say that a Pod is atomic, what we mean is that all of the containers in a Pod are guaranteed to land on the same machine in the cluster. Pods also share many resources between the containers. For example, they all share the same network namespace, which means that each container in a Pod can see the other containers in the Pod on localhost. Pods also share the process and interprocess communication namespaces so that different containers can use tools, like shared memory and signaling, to coordinate between the different processes in the Pod.

Pods also do things to keep your application running. If the process in a container crashes, Kubernetes automatically restarts it. Pods can also define application-level health checks that can provide a richer, application-specific way of determining whether the Pod should be automatically restarted.

#### ReplicaSets
Of course, if you are deploying a container orchestrator just to run individual containers, you are probably overcomplicating your life. In general, one of the main reasons for container orchestration is to make it easier to build replicated, reliable systems. Although individual containers may fail or may be incapable of serving the load of a system, replicating an application out to a number of different running containers dramatically reduces the probability that your service will completely fail at a particular moment in time. Plus, horizontal scaling enables you to grow your application in response to load. In the Kubernetes API, this sort of stateless replication is handled by a ReplicaSet object. A ReplicaSet ensures that, for a given Pod definition, a number of replicas exists within the system. The actual replication is handled by the Kubernetes controller manager, which creates Pod objects that are scheduled by the Kubernetes scheduler.

#### Services
After you can replicate your application out using a replica set, the next logical goal is to create a load balancer to spread traffic to these different replicas. To accomplish this, Kubernetes has a Service object. A Service represents a TCP or UDP loadbalanced service. Every Service that is created, whether TCP or UDP, gets three things:
* Its own IP address
* A DNS entry in the Kubernetes cluster DNS
* Load-balancing rules that proxy traffic to the Pods that implement the Service

When a Service is created, it is assigned a fixed IP address. This IP address is virtual — it does not correspond to any interface present on the network. Instead, it is programmed into the network fabric as a load-balanced IP address. When packets are sent to that IP, they are load balanced out to a set of Pods that implements the Ser vice. The load balancing that is performed can either be round robin or deterministic, based on source and destination IP address tuples. 

Given this fixed IP address, a DNS name is programmed into the Kubernetes cluster’s DNS server. This DNS address provides a semantic name (e.g., “frontend”), which is the same as the name of the Kubernetes Service object and which enables other containers in the cluster to discover the IP address of the Service load balancer.

Finally, the Service load balancing is programmed into the network fabric of the Kubernetes cluster so that any container that tries to talk to the Service IP address is correctly load balanced to the corresponding Pods. This programming of the network fabric is dynamic, so as Pods come and go due to failures or scaling of a ReplicaSet, the load balancer is constantly reprogrammed to match the current state of the cluster. This means that clients can rely on connections to the Service IP address always resolving to a Pod that implements the Service.

### Storage: Persistent Volumes, ConfigMaps, and Secrets
#### Volumes
The first storage concept introduced in Kubernetes was Volume, which is actually a part of the Pod API. Within a Pod, you can define a set of Volumes. Each Volume can be one of a large number of different types. At present, there are more than 10 different types of Volumes you can create, including NFS, iSCSI, gitRepo, cloud storage–based Volumes, and more. Volume types are developed outside of the Kubernetes code and use the Container Storage Interface (CSI), an interface for storage that is independent of Kubernetes.

Over time, deploying applications with Volumes revealed that the tight binding of Volumes to Pods was actually problematic. For example, when creating a replicated container (via a ReplicaSet) the same exact volume must be used by all replicas. In many situations, this is acceptable, but in some cases, you migth want a different Volume for each replica. Additionally, specifying a precise volume type (e.g., an Azure disk-persistent Volume) binds your Pod definition to a specific environment (in this case, the Microsoft Azure cloud), but it is often desirable to have a Pod definition that requests a generic type of storage (e.g., 10 gigabytes of network storage) without specifying a provider. To accomplish this, Kubernetes introduced the notion of Persis tentVolumes and PersistentVolumeClaims. Instead of binding a Volume directly into a Pod, a PersistentVolume is created as a separate object. This object is then claimed to a specific Pod by a PersistentVolumeClaim and finally mounted into the Pod via this claim. At first, this seems overly complicated, but the abstraction of Volume and Pod enables both the portability and automatic volume creation required by the two previous use cases.

#### ConfigMaps
In addition to basic files, there are several types of Kubernetes objects that can themselves be mounted into your Pod as a Volume. The first of these is the ConfigMap object. A ConfigMap represents a collection of configuration files. In Kubernetes, you want to have different configurations for the same container image. When you add a ConfigMap-based Volume to your Pod, the files in the ConfigMap show up in the specified directory in your running container.

#### Secrets
Kubernetes uses the Secret configuration type for secure data, such as database passwords and certificates. In the context of Volumes, a Secret works identically to a Con figMap. It can be attached to a Pod via a Volume and mounted into a running container for use.

### Organizing Your Cluster with Namespaces, Labels, and Annotations
#### Namespaces
The first object for organizing your cluster is Namespace. You can think of a Name space as something like a folder for your Kubernetes API objects. Namespaces provide directories for containing most of the other objects in the cluster. Namespaces can also provide a scope for role-based access control (RBAC) rules. Like a folder, when you delete a Namespace, all of the objects within it are also destroyed, so be careful! Every Kubernetes cluster has a single built-in Namespace named default, and most installations of Kubernetes also include a Namespace named kube-system, where cluster administration containers are created.

Kubernetes objects are divided into namespaced and nonnamespaced objects, depending on whether they can be placed in a Namespace. Most common Kubernetes API objects are namespaced objects. But some objects that apply to an entire cluster (e.g., Name space objects themselves, or cluster-level RBAC), are not namespaced.
#### Labels and label queries
Every object in the Kubernetes API can have an arbitrary set of labels associated with it. Labels are string key-value pairs that help identify the object. For example, a label might be "role": "frontend", which indicates that the object is a frontend. These labels can be used to query and filter objects in the API. For example, you can request that the API server provide you with a list of all Pods where the label role is backend. These requests are called label queries or label selectors. Many objects within the Kubernetes API use label selectors as a way to identify sets of objects that they apply to. For example, a Pod can have a node selector, which identifies the set of nodes on which the Pod is elegible to run (nodes with GPUs, for example). Likewise, a Service has a Pod selector, which identifies the set of Pods that the Service should load balance traffic to. Labels and label selectors are the fundamental manner in which Kubernetes loosely couples its objects together.

#### Annotations
Not every metadata value that you want to assign to an API object is identifying information. Some of the information is simply an annotation about the object itself. Thus every Kubernetes API object can also have arbitrary annotations. These might include something like the icon to display next to the object or a modifier that changes the way that the object is interpreted by the system.

### Advanced Concepts: Deployments, Ingress, and StatefulSets
#### Deployments
Although ReplicaSets are the primitive for running many different copies of the same container image, applications are not static entities. They evolve as developers add new features and fix bugs. This means that the act of rolling out new code to a Service is as important a feature as replicating it to reliably handle load. 

The Deployment object was added to the Kubernetes API to represent this sort of safe rollout from one version to another. A Deployment can hold pointers to multiple ReplicaSets, (e.g., v1 and v2), and it can control the slow and safe migration from one ReplicaSet to another.

To understand how a Deployment works, imagine that you have an application that is deployed to three replicas in a ReplicaSet named rs-v1. When you ask a Deploy ment to roll out a new image (v2), the Deployment creates a new ReplicaSet (rs-v2)with a single replica. The Deployment waits for this replica to becomes healthy, and when it is, the Deployment reduces the number of replicas in rs-v1 to two. It then increases the number of replicas in rs-v2 to two also, and waits for the second replica of v2 to become healthy. This process continues until there are no more replicas of v1 and there are three healthy replicas of v2.

#### HTTP load balancing with Ingress
Although Service objects provide a great way to do simple TCP-level load balancing, they don’t provide an application-level way to do load balancing and routing. The truth is that most of the applications that users deploy using containers and Kubernetes are HTTP web-based applications. These are better served by a load balancer that understands HTTP. To address these needs, the Ingress API was added to Kubernetes. Ingress represents a path and host-based HTTP load balancer and router. When you create an Ingress object, it receives a virtual IP address just like a Service, but instead of the one-to-one relationship between a Service IP address and a set of Pods, an Ingress can use the content of an HTTP request to route requests to different Services.

#### StatefulSets
Most applications operate correctly when replicated horizontally and treated as identical clones. Each replica has no unique identity independent of any other. For representing such applications, a Kubernetes ReplicaSet is the perfect object. However, some applications, especially stateful storage workloads or sharded applications, require more differentiation between the replicas in the application. Although it is possible to add this differentiation at the application level on top of a ReplicaSet, doing so is complicated, error prone, and repetitive for end users.

To resolve this, Kubernetes has recently introduced StatefulSets as a complement to ReplicaSets, but for more stateful workloads. Like ReplicaSets, StatefulSets create multiple instances of the same container image running in a Kubernetes cluster, but the manner in which containers are created and destroyed is more deterministic, as are the names of each container.

In a ReplicaSet, each replicated Pod receives a name that involves a random hash(e.g., frontend-14a2), and there is no notion of ordering in a ReplicaSet. In contrast, with StatefulSets, each replica receives a monotonically increasing index (e.g., backed-0, backend-1, and so on).

Further, StatefulSets guarantee that replica zero will be created and become healthy before replica one is created and so forth. When combined, this means that applications can easily bootstrap themselves using the initial replica (e.g., backend-0) as a bootstrap master. All subsequent replicas can rely on the fact that backend-0 has to exist. Likewise, when replicas are removed from a StatefulSet, they are removed at the highest index. If a StatefulSet is scaled down from five to four replicas, it is guaranteed that the fifth replica is the one that will be removed.

Additionally, StatefulSets receive DNS names so that each replica can be accessed directly, in addition to the complete StatefulSet. This allows clients to easily target specific shards in a sharded service.

### Batch Workloads: Job and ScheduledJob
#### Job
In Kubernetes, a Job represents a set of tasks that needs to be run. Like ReplicaSets and StatefulSets, Jobs operate by creating Pods to execute work by running container images. However, unlike ReplicaSets and StatefulSets, the Pods created by a Job only run until they complete and exit. A Job contains the definition of the Pods it creates, the number of times the Job should be run, and the maximum number of Pods to create in parallel. For example, a Job with 100 repetitions and a maximum parallelism of 10 will run 10 Pods simultaneously, creating new Pods as old ones complete, until there have been 100 successful executions of the container image.

#### ScheduledJob
ScheduledJobs build on top of the Job object by adding a schedule to a Job. A Sched uledJob contains the definition of the Job object that you want to create, as well as the schedule on which that Job should be created.

### Cluster Agents and Utilities: DaemonSets
One of the most common questions that comes up when people are moving to Kubernetes is, “How do I run my machine agents?” Examples of agents’ tasks include intrusion detection, logging and monitoring, and others. Many people attempt non- Kubernetes approaches to enable these agents, such as adding new systemd unit files or initialization scripts. Although these approaches can work, they have several downsides. The first is that Kubernetes does not include agents’ activity in its accounting of resources in use on the cluster. The second is that container images and Kubernetes APIs for health checking, monitoring, and more cannot be applied to these agents. Fortunately, Kubernetes makes the DaemonSet API available to users to install such agents on their clusters. A DaemonSet provides a template for a Pod that should be run on every machine. When a DaemonSet is created, Kubernetes ensures that this Pod is running on each node in the cluster. If, at some later point, a new node is added, Kubernetes creates a Pod on that node, as well. Although by default Kubernetes places a Pod on every node in the cluster, a DaemonSet can also provide a node selector label query, and Kubernetes will only place that DaemonSet’s Pods onto nodes that match that label query.