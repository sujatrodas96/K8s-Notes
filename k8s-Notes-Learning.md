Kubernetes Arch
------------------

It has 2 parts

1. Control-Plane 
2. Nodes

Control Plane has 5 things which is very Important
-----------------------------------------------------

    The control plane's components make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events (for example, starting up a new pod when a Deployment's replicas field is unsatisfied).
    Control plane components can be run on any machine in the cluster. However, for simplicity, setup scripts typically start all control plane components on the same machine, and do not run user containers on this machine.

1. kube-apiserver
-------------------
    The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.
    The main implementation of a Kubernetes API server is kube-apiserver. kube-apiserver is designed to scale horizontally—that is, it scales by deploying more instances. You can run several instances of kube-apiserver and balance traffic between those instances.

2. etcd
---------
    Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.
    If your Kubernetes cluster uses etcd as its backing store, make sure you have a back up plan for the data.

3. kube-scheduler
------------------

    Control plane component that watches for newly created Pods with no assigned node, and selects a node for them to run on.
    Factors taken into account for scheduling decisions include: individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference, and deadlines.


4. kube-controller-manager
----------------------------

    Control plane component that runs controller processes.
    Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.
    There are many different types of controllers. Some examples of them are:

    Node controller: Responsible for noticing and responding when nodes go down.
    Job controller: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
    EndpointSlice controller: Populates EndpointSlice objects (to provide a link between Services and Pods).
    ServiceAccount controller: Create default ServiceAccounts for new namespaces.

5. cloud-controller-manager
-----------------------------

    A Kubernetes control plane component that embeds cloud-specific control logic. The cloud controller manager lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster.
    The cloud-controller-manager only runs controllers that are specific to your cloud provider. If you are running Kubernetes on your own premises, or in a learning environment inside your own PC, the cluster does not have a cloud controller manager.

    As with the kube-controller-manager, the cloud-controller-manager combines several logically independent control loops into a single binary that you run as a single process. You can scale horizontally (run more than one copy) to improve performance or to help tolerate failures.

    The following controllers can have cloud provider dependencies:

    Node controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
    Route controller: For setting up routes in the underlying cloud infrastructure
    Service controller: For creating, updating and deleting cloud provider load balancers



Node components
------------------

    Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.



1. kubelet
------------

    An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.

    The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn't manage containers which were not created by Kubernetes.

2. kube-proxy
---------------
    kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.

    kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

    kube-proxy uses the operating system packet filtering layer if there is one and it's available. Otherwise, kube-proxy forwards the traffic itself.

    If you use a network plugin that implements packet forwarding for Services by itself, and providing equivalent behavior to kube-proxy, then you do not need to run kube-proxy on the nodes in your cluster.

3. Container runtime 
----------------------

    A fundamental component that empowers Kubernetes to run containers effectively. It is responsible for managing the execution and lifecycle of containers within the Kubernetes environment.

Addons
-------

    Addons use Kubernetes resources (DaemonSet, Deployment, etc) to implement cluster features. Because these are providing cluster-level features, namespaced resources for addons belong within the kube-system namespace.

    Selected addons are described below; for an extended list of available addons, please see Addons.

DNS
-----

    While the other addons are not strictly required, all Kubernetes clusters should have cluster DNS, as many examples rely on it.

    Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services.

    Containers started by Kubernetes automatically include this DNS server in their DNS searches.

Web UI (Dashboard)
--------------------

    Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.

Container resource monitoring
-------------------------------

    Container Resource Monitoring records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.

Cluster-level Logging
----------------------
    A cluster-level logging mechanism is responsible for saving container logs to a central log store with a search/browsing interface.

Network plugins
-------------------

    Network plugins are software components that implement the container network interface (CNI) specification. They are responsible for allocating IP addresses to pods and enabling them to communicate with each other within the cluster.

    By Default: Calico, Weave Net

Architecture variations
-------------------------

    While the core components of Kubernetes remain consistent, the way they are deployed and managed can vary. Understanding these variations is crucial for designing and maintaining Kubernetes clusters that meet specific operational needs.

Control plane deployment options
-----------------------------------

    The control plane components can be deployed in several ways:

Traditional deployment
-----------------------

    Control plane components run directly on dedicated machines or VMs, often managed as systemd services.

Static Pods
-------------

    Control plane components are deployed as static Pods, managed by the kubelet on specific nodes. This is a common approach used by tools like kubeadm.

Self-hosted
---------------

    The control plane runs as Pods within the Kubernetes cluster itself, managed by Deployments and StatefulSets or other Kubernetes primitives.

Managed Kubernetes services
-----------------------------

    Cloud providers often abstract away the control plane, managing its components as part of their service offering.

Workload placement considerations
-----------------------------------

    The placement of workloads, including the control plane components, can vary based on cluster size, performance requirements, and operational policies:

    In smaller or development clusters, control plane components and user workloads might run on the same nodes.
    Larger production clusters often dedicate specific nodes to control plane components, separating them from user workloads.
    Some organizations run critical add-ons or monitoring tools on control plane nodes.
    Cluster management tools
    Tools like kubeadm, kops, and Kubespray offer different approaches to deploying and managing clusters, each with its own method of component layout and management.

Customization and extensibility
-----------------------------------

    Kubernetes architecture allows for significant customization:

    Custom schedulers can be deployed to work alongside the default Kubernetes scheduler or to replace it entirely.
    API servers can be extended with CustomResourceDefinitions and API Aggregation.
    Cloud providers can integrate deeply with Kubernetes using the cloud-controller-manager.
    The flexibility of Kubernetes architecture allows organizations to tailor their clusters to specific needs, balancing factors such as operational complexity, performance, and management overhead.



To Create Cluster
-------------------

    vi config.yaml

    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4

    nodes:
    - role: control-plane
        image: kindest/node:v1.29.0

    - role: worker
        image: kindest/node:v1.29.0

    - role: worker
        image: kindest/node:v1.29.0

this has one control-plane and 2 nodes

To create cluster
I am using KIND so the cmd will be

kind create cluster --name=my-cluster --config=config.yaml

To see the Cluster info you have to write cmd
----------------------------------------------

    kubectl cluster-info

Output:
--------

    Kubernetes control plane is running at https://127.0.0.1:41735
    CoreDNS is running at https://127.0.0.1:41735/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

To check the nodes and control-plane are working properly or not
-------------------------------------------------------------------

    kubectl get nodes

Output:
--------

    NAME                           STATUS   ROLES           AGE   VERSION
    devops-cluster-control-plane   Ready    control-plane   96d   v1.29.0
    devops-cluster-worker          Ready    <none>          96d   v1.29.0
    devops-cluster-worker2         Ready    <none>          96d   v1.29.0



NameSpaces
--------------
    In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc.) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc.).

When to Use Multiple Namespaces
---------------------------------
    Namespaces are intended for use in environments with many users spread across multiple teams, or projects. For clusters with a few to tens of users, you should not need to create or think about namespaces at all. Start using namespaces when you need the features they provide.

    Namespaces provide a scope for names. Names of resources need to be unique within a namespace, but not across namespaces. Namespaces cannot be nested inside one another and each Kubernetes resource can only be in one namespace.

    Namespaces are a way to divide cluster resources between multiple users

this way manage kubernetes resource in a structured way
------------------------------------------------------------

    Namespace -> [docker-container(NGINX) -> pods -> deployment -> service -> User]
    Namespace -> [docker-container(MYSQL) -> pods -> deployment -> service -> User]

[In this structure, we have two namespaces: one for NGINX and another for MySQL. 
Each namespace contains a docker container, which is then organized into pods, deployments, services, and users. 
This allows for better management and isolation of resources within the Kubernetes cluster.]

To see Namespaces in Kubernetes, you can use the following command:
---------------------------------------------------------------------

    cmd: kubectl get namespaces

by default, Kubernetes has a few namespaces like 
    'default', 'kube-node-lease', 'kube-public', 'kube-system', and 'local-path-storage'.

to see pods in kubernetes by default namespaces you have to run cmd
---------------------------------------------------------------------

    cmd: kubectl get pods -n kube-public
    cmd: kubectl get pods -n kube-system --> run pods like

        1.etcd-devops-cluster-control-plane                                  
        2.kube-apiserver-devops-cluster-control-plane                        
        3.kube-controller-manager-devops-cluster-control-plane


    cmd: kubectl get pods -n local-path-storage -> run pod in local-path-storage
        local-path-provisioner-6f8956fb48-rh9jn 

to create namespace
----------------------

    cmd: kubectl create ns nginx

    output: namespace/nginx created

next to run nginx
----------------------

    cmd: kubectl run nginx --image=nginx

    output: pod/nginx created but it created in default namespace not nginx namespace

if this happenes then delete your pod
-----------------------------------------
    cmd: kubectl delete pod nginx

    output: pod "nginx" deleted

so if pod have to define nginx namespace type cmd
---------------------------------------------------

    cmd: kubectl run nginx --image=nginx -n nginx  -n == nginx

    output: pod/nginx created now its created in namespace nginx

now for checking the pod is in the correct namespace or not
-------------------------------------------------------------

    cmd: kubectl get pods -n nginx

    output: nginx will show

delete pod in nginx
------------------------

    cmd: kubectl delete pods nginx -n nginx

    output: pod "nginx" deleted 

but all things happen through cmd line but we want to do all of the things through YAML/YML file

First we will create a namespace which can be called also groups
--------------------------------------------------------------------

    vim namespace.yaml


    kind: Namespace
    apiVersion: v1
    metadata:
      name: nginx

    cmd: kubectl apply -f namespace.yaml

    output: namespace/nginx created


Now we will create a pod
--------------------------------
    vim pod.yaml

    kind: Pod
    apiVersion: v1
    metadata:
      name: nginx-pod
      namespace: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports: 
         - containerPort: 80


Another Yaml for Pod
-------------------------
    vim pod1.yaml
    
    apiVersion: v1
    kind: Pod
    metadata:
    name: label-demo
    labels:
        environment: production
        app: nginx
    spec:
    containers:
    - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80


    Note: in spec you can define one or more containers also

    cmd: kubectl apply -f pod.yaml

To see the pods is running in nginx namespace after apply cmd
---------------------------------------------------------------

    cmd: kubectl get pods -n nginx

To enter in the pod section like docker container
-----------------------------------------------------

    cmd: kubectl exec -it pod/ngnix-pod -n nginx -- bash

to describee your pods
-------------------------

    cmd: kubectl describe pod/nginx-pod -n nginx

Most Important Part to know
----------------------------------

Events:

Output:
--------------

    Type    Reason     Age   From               Message
    ----    ------     ----  ----               -------
    Normal  Scheduled  9s    default-scheduler  Successfully assigned nginx/nginx-pod to devops-cluster-worker2
    Normal  Pulling    9s    kubelet            spec.containers{nginx}: Pulling image "nginx:latest"
    Normal  Pulled     7s    kubelet            spec.containers{nginx}: Successfully pulled image "nginx:latest" in 1.501s (1.501s including waiting)
    Normal  Created    7s    kubelet            spec.containers{nginx}: Created container nginx
    Normal  Started    7s    kubelet            spec.containers{nginx}: Started container nginx


Labels and Selectors
---------------------

    Labels are key/value pairs that are attached to objects such as Pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system. Labels can be used to organize and to select subsets of objects. Labels can be attached to objects at creation time and subsequently added and modified at any time. Each object can have a set of key/value labels defined. Each Key must be unique for a given object.


You can find Pod in this specif way Also
-----------------------------------------

    kubectl get pods -n nginx -l environment=production,tier=frontend

Pod has 5 phase Those Phase are
--------------------------------------

Pending:
---------
	The Pod has been accepted by the Kubernetes cluster, but one or more of the containers has not been set up and made ready to run. This includes time a Pod spends waiting to be scheduled as well as the time spent downloading container images over the network.
Running:
---------
	The Pod has been bound to a node, and all of the containers have been created. At least one container is still running, or is in the process of starting or restarting.
Succeeded:
-----------
	All containers in the Pod have terminated in success, and will not be restarted.
Failed:
-----------
	All containers in the Pod have terminated, and at least one container has terminated in failure. That is, the container either exited with non-zero status or was terminated by the system, and is not set for automatic restarting.
Unknown:
-----------
	For some reason the state of the Pod could not be obtained. This phase typically occurs due to an error in communicating with the node where the Pod should be running.

    When a pod is failing to start repeatedly, CrashLoopBackOff may appear in the Status field of some kubectl commands. Similarly,
    when a pod is being deleted, Terminating may appear in the Status field of some kubectl commands.

    Make sure not to confuse Status, a kubectl display field for user intuition, with the pod's phase. Pod phase is an explicit part of the Kubernetes data model and of the Pod API.

  NAMESPACE               NAME               READY   STATUS             RESTARTS   AGE
  alessandras-namespace   alessandras-pod    0/1     CrashLoopBackOff   200        2d9h

    A Pod is granted a term to terminate gracefully, which defaults to 30 seconds. You can use the flag --force to terminate a Pod by force.

Container Lifecycle
--------------------

    Once the scheduler assigns a Pod to a Node, the kubelet starts creating containers for that Pod using a container runtime. 
    There are three possible container states:

    Waiting 
    Running
    Terminated.

For creating Pod is easy but need the Pod making scalable so thats way Deployment cames into picture
------------------------------------------------------------------------------------------------------------

    A Deployment manages a set of Pods to run an application workload, usually one that doesn't maintain state.
    A Deployment provides declarative updates for Pods and ReplicaSets.

    You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

    Note:
    Do not manage ReplicaSets owned by a Deployment. Consider opening an issue in the main Kubernetes repository if your use case is not covered below.

Use Case
----------
    The following are typical use cases for Deployments:

    Create a Deployment to rollout a ReplicaSet. The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.

    Declare the new state of the Pods by updating the PodTemplateSpec of the Deployment. A new ReplicaSet is created, and the Deployment gradually scales it up while scaling down the old ReplicaSet, ensuring Pods are replaced at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.

    Rollback to an earlier Deployment revision if the current state of the Deployment is not stable. Each rollback updates the revision of the Deployment.
    Scale up the Deployment to facilitate more load.

    Pause the rollout of a Deployment to apply multiple fixes to its PodTemplateSpec and then resume it to start a new rollout.

    Use the status of the Deployment as an indicator that a rollout has stuck.

    Clean up older ReplicaSets that you don't need anymore.


So, Creating deployment 

    vim deployment.yaml
------------------------------

    kind: Deployment
    apiVersion: apps/v1
    metadata:
    name: nginx-deployment
    namespace: nginx
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: nginx

    template:
        metadata:
        name: nginx-dep-pod
        labels:
            app: nginx-deploy

        spec:
        containers:
        - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80

To see the deployment
--------------------------

    cmd: kubectl get deployment -n nginx

Output:
----------

    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   2/2     2            2           90d

To scale up your deployment
------------------------------

    cmd: kubectl scale deployment/nginx-deployment -n nginx --replicas=5
    
    Output: deployment.apps/nginx-deployment scaled

    cmd: kubectl get pods -n nginx

Output: 
-----------

    NAME                                READY   STATUS    RESTARTS        AGE
    nginx-deployment-6cb68cf79d-565cx   1/1     Running   154 (75m ago)   90d
    nginx-deployment-6cb68cf79d-8bvgb   1/1     Running   0               7s
    nginx-deployment-6cb68cf79d-kg68q   1/1     Running   0               7s
    nginx-deployment-6cb68cf79d-s2drh   1/1     Running   0               7s
    nginx-deployment-6cb68cf79d-wqjmq   1/1     Running   154 (75m ago)   90d

To describe the deployment status
-------------------------------------

    cmd: kubectl describe deployment -n nginx

    Output:

    Name:                   nginx-deployment
    Namespace:              nginx
    CreationTimestamp:      Thu, 22 Jan 2026 20:27:47 +0530
    Labels:                 <none>
    Annotations:            deployment.kubernetes.io/revision: 2
    Selector:               app=nginx
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
    Labels:  app=nginx
    Containers:
    nginx:
        Image:         nginx:1.29.4
        Port:          80/TCP
        Host Port:     0/TCP
        Environment:   <none>
        Mounts:        <none>
    Volumes:         <none>
    Node-Selectors:  <none>
    Tolerations:     <none>
    Conditions:
    Type           Status  Reason
    ----           ------  ------
    Progressing    True    NewReplicaSetAvailable
    Available      True    MinimumReplicasAvailable
    OldReplicaSets:  nginx-deployment-6dbd7f7b8c (0/0 replicas created)
    NewReplicaSet:   nginx-deployment-6cb68cf79d (3/3 replicas created)
    Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  39m   deployment-controller  Scaled up replica set nginx-deployment-6cb68cf79d to 5 from 2
    Normal  ScalingReplicaSet  3s    deployment-controller  Scaled down replica set nginx-deployment-6cb68cf79d to 3 from 5


If i want more information about Pods then
------------------------------------------------

        cmd: kubectl get pods -n nginx -o wide

        Output:

    NAME                                READY   STATUS    RESTARTS         AGE    IP           NODE                     NOMINATED NODE   READINESS GATES
    nginx-deployment-6cb68cf79d-565cx   1/1     Running   154 (133m ago)   90d    10.244.1.2   devops-cluster-worker2   <none>           <none>
    nginx-deployment-6cb68cf79d-8bvgb   1/1     Running   0                58m    10.244.2.5   devops-cluster-worker    <none>           <none>
    nginx-deployment-6cb68cf79d-s2drh   1/1     Running   0                58m    10.244.2.4   devops-cluster-worker    <none>           <none>
    nginx-pod                           1/1     Running   1 (133m ago)     139m   10.244.1.4   devops-cluster-worker2   <none>           <none>
    nginx-replicaset-7vqwh              1/1     Running   144 (133m ago)   87d    10.244.2.3   devops-cluster-worker    <none>           <none>
    nginx-replicaset-pnw2k              1/1     Running   143 (133m ago)   87d    10.244.2.2   devops-cluster-worker    <none>           <none>

Rolling updates:
-----------------------
    to update the pods we use rolling updates

    cmd: kubectl set image deployments/nginx-deployment -n nginx nginx=nginx:1.27.2
    
    Output:
    deployment.apps/nginx-deployment image updated

    cmd: kubectl get pods -n nginx -o wide

    Output:
    NAME                                READY   STATUS              RESTARTS         AGE    IP            NODE                     NOMINATED NODE   READINESS GATES
    nginx-deployment-774f469f5c-lxwv4   1/1     Running             0                98s    10.244.2.6    devops-cluster-worker    <none>           <none>
    nginx-deployment-774f469f5c-w8fzt   1/1     Running             0                73s    10.244.1.11   devops-cluster-worker2   <none>           <none>
    nginx-deployment-774f469f5c-zcx9p   1/1     Running             0                2m4s   10.244.1.10   devops-cluster-worker2   <none>           <none>
    nginx-deployment-cbd8747bb-292zv    0/1     ContainerCreating   0                2s     <none>        devops-cluster-worker2   <none>           <none>
    nginx-pod                           1/1     Running             1 (169m ago)     175m   10.244.1.4    devops-cluster-worker2   <none>           <none>
    nginx-replicaset-7vqwh              1/1     Running             144 (169m ago)   87d    10.244.2.3    devops-cluster-worker    <none>           <none>
    nginx-replicaset-pnw2k              1/1     Running             143 (169m ago)   87d    10.244.2.2    devops-cluster-worker    <none>           <none>

To check all rollout history
------------------------------------

    cmd: kubectl rollout history deployment/nginx-deployment -n nginx

    Output:deployment.apps/nginx-deployment 

    REVISION  CHANGE-CAUSE
    2         <none>
    3         <none>
    5         <none>
    6         <none>
    7         <none>
    8         <none>
    9         image updated to 1.16.1
    10        <none>

To see details of each version
---------------------------------

    cmd: kubectl rollout history deployment/nginx-deployment -n nginx --revision=2


Output:
---------

    deployment.apps/nginx-deployment with revision #2
    Pod Template:
    Labels:	app=nginx
        pod-template-hash=6cb68cf79d
    Containers:
    nginx:
        Image:	nginx:1.29.4
        Port:	80/TCP
        Host Port:	0/TCP
        Environment:	<none>
        Mounts:	<none>
    Volumes:	<none>
    Node-Selectors:	<none>
    Tolerations:	<none>


Now you've decided to undo the current rollout and rollback to the previous revision:
----------------------------------------------------------------------------------------
    cmd: kubectl rollout undo deployment/nginx-deployment -n nginx

Alternatively, you can rollback to a specific revision by specifying it with --to-revision:
----------------------------------------------------------------------------------------------
    cmd: kubectl rollout undo deployment/nginx-deployment -n nginx --to-revision=2

Output:
---------
    deployment.apps/nginx-deployment rolled back

Assuming Horizontal Pod AutoScaling
---------------------------------------

    cmd: kubectl autoscale deployment/nginx-deployment -n nginx --min=10 --max=15 --cpu-percent=80%

ReplicaSet
-------------

    A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. Usually, you define a Deployment and let that Deployment manage ReplicaSets automatically.
    A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

How a ReplicaSet works
--------------------------

    A ReplicaSet is defined with fields, including a selector that specifies how to identify Pods it can acquire, a number of replicas indicating how many Pods it should be maintaining, and a pod template specifying the data of new Pods it should create to meet the number of replicas criteria. A ReplicaSet then fulfills its purpose by creating and deleting Pods as needed to reach the desired number. When a ReplicaSet needs to create new Pods, it uses its Pod template.

    A ReplicaSet is linked to its Pods via the Pods' metadata.ownerReferences field, which specifies what resource the current object is owned by. All Pods acquired by a ReplicaSet have their owning ReplicaSet's identifying information within their ownerReferences field. It's through this link that the ReplicaSet knows of the state of the Pods it is maintaining and plans accordingly.

    A ReplicaSet identifies new Pods to acquire by using its selector. If there is a Pod that has no OwnerReference or the OwnerReference is not a Controller and it matches a ReplicaSet's selector, it will be immediately acquired by said ReplicaSet.

When to use a ReplicaSet
----------------------------

    A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

    This actually means that you may never need to manipulate ReplicaSet objects: use a Deployment instead, and define your application in the spec section.

To Create a ReplicaSet through Yaml File
------------------------------------------

    vi replicaset.yaml

    kind: ReplicaSet
    apiVersion: apps/v1
    metadata:
    name: nginx-replicaset
    namespace: nginx
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: nginx
    template:
        metadata:
        name: nginx-rep-pod
        labels:
            app: nginx-rs
        spec:
        containers:
        - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80

    kubectl apply -f replicaset.yaml

To see ReplicaSet through cmd
-------------------------------

    kubectl get rs -n nginx

To describe the ReplicaSet
----------------------------

    kubectl describe rs/nginx-replicaset -n nginx


DaemonSet
-----------

    A DaemonSet defines Pods that provide node-local facilities. These might be fundamental to the operation of your cluster, such as a networking helper tool, or be part of an add-on.

    A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

To create DaemonSet through Yaml File
--------------------------------------

    vi daemonset.yaml

    kind: DaemonSet
    apiVersion: apps/v1
    metadata:
    name: nginx-daemonSet
    namespace: nginx
    spec:
    selector:
        matchLabels:
        app: nginx
    template:
        metadata:
        name: nginx-dmn-pod
        labels:
            app: nginx
        spec:
        containers:
        - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80

    kubectl apply -f daemonset.yaml
