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
            app: nginx

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
            app: nginx
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
