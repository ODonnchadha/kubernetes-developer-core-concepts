## Kubernetes for Developers: Core Concepts by Dan Wahlin

- COURSE OVERVIEW:
    - A developer-focused look at key Kubernetes resources, benefits they can provide, and how to get started using them.
    - [Samples Via GitHub](https://github.com/DanWahlin/DockerAndKubernetesCourseCode)

- KUBERNETES FROM A DEVELOPER PERSPECTIVE:
    - Overview: 
        - The context of a developer. Command-line tools and VMs. Software development. Docker containers.
    - Introduction:
    - Kubernetes Overview:
        - Kubernetes (K8s) is an open‑source system for automating deployment, scaling, and management of containerized applications.
        - How are you managing containers? Locally? Docker compose. Package up an app and let someone else manage it.
        - Kubernetes is a conductor of a container orchestra.
            - Discover different services. Load balancing. Storage. Rollouts/rollbacks. Self-healing. Secret & configuration management. Horizontal scaling.
        - The Big Picture:
            - Kubernetes. Container & cluster management. Open source. Used internally by Google. Supported by all major cloud platforms.
        - Benefits and Use Cases:
            - Container benefits:
                - Accelerate developer onboarding. Eliminate app cpfl;icts. Environment consistency. Ship software faster.
            - Kuberenetes Benefits:
                - Orchestrate containers. Zero-downtime deployments. Self-healing powers. And we can scale.
                - Developer use cases:
                    - Emulate production locally. Move from Docker compose to Kubernetes. Create an end-to-end testing environment.
                    - Ensure applcation scales properly. Ensure secrets/configuration. Performance testing. 
                    - Workload scenarios (CI/CD.) Leverage deployment options. Help DevOps create resources & solve problems.
            - Running Kubernetes Locally:
                - Minikube.
                - Docker Desktop. (One master node & one worker node.)
                - KIND. Kubernetes in Docker.
                - kubeadm.
            - Getting Started with kubectl:
                - $kubectl MASTER -> Node(Pod), Node(Pod), Node(Pod).
                ```javascript
                    kubectl version
                    kubectl cluster-info
                    kubectl get all
                    kubectl run [container-name] --image=[image-name]
                    kubectl port-forward [pod] [ports]
                    kubectl expose
                    kubectl create [resource]
                    kubectl apply [resource]
                    alias k="kubectl"
                ```
                ```powershell
                    Set-Alias -Name k -Value kubectl
                ```
            - Web UI Dashboard:
                - URL. Token. & proxy. [Official Guide](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
                ```javascript
                    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml
                    kubectl describe secret -n kube-system
                    kubectl proxy
                ```
                - Look for 'kuberenetes.io/service-account-token.
            - Summary:

- CREATING PODS:
    - Introduction:
        - Pod core concepts: Creating a Pod. kubectl & Pods. YAML fundamentals. Defining a Pod with YAML. Pod health.
    - Pod Core Concepts:
        - A Pod is the basic execution unit of a Kubernetes application, the smallest and simplest unit in the Kubernetes object model that you create or deploy.
        - Pods run containers. The smallest object of the Kubernetes object model. Organize application(s) into Pods: e.g.: Server. Caching. APIs. Database.
        - A Pod has an IP address, memory, volumes shared across containers. Sclae horizontally by apping Pod replicas. Pods live/die buy never come back to life.
        - Master-Node-Pod-Container. Cluster IP address. Container processes need to bind to different ports within a Pod. 
        - Ports can be reused be containers in seperate Pods. Pods never span nodes.
    - Creating a Pod:
        - Imperative
        - $kubectl create/apply command with a YAML file.
        ```javascript
            kubectl run [podname] --image=nginx:alpine
        ```
        - Cluster IP address. Exposed to nodes within. NOTE: 8080 external port. Expose the port via the node.
        ```javascript
            kubectl port-forward [podname] 8080:80
            kubectl delete pod [podname]
        ```
        - kubectl and Pods: Note: ClusterIP are internal only.
        ```javascript
            kubectl get all
            kubectl run my-nginx --image=nginx:alpine
            kubectl get pods
            kubectl port-forward my-nginx 8080:80
        ```
        [localhost](http://localhost:8080/)
        ```javascript
            kubectl delete pod my-nginx
            kubectl get pods
        ```
    - YAML Fundamentals:
        - Declarative.
        - Maps & lists. Indentation matters. Always use spaces. 
        - Maps: Name/value pairs. Contain other maps.
        - Lists: Sequence of items. Multiple maps can be defined within a list.
    - Defining a Pod with YAML:
        - YAML + kubectl = Pod(container)
        ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
                name: my-nginx
            spec:
                containers:
                - name: my-nginx
                  image: mginx:alpine
        ```
    - Perform a dry run with the YAML.
        ```javascript
            kubectl create -f my-nginx.yml --dry-run=client --validate=true
            kubectl create -f my-nginx.yml --save-config
            kubectl apply -f my-nginx.yml
        ```
    - kubectl and YAML:
        ```javascript
            kubectl create -f my-nginx.yml --save-config
            kubectl describe pod my-nginx
            kubectl get pod my-nginx -o yaml
            kubectl exec my-nginx -it sh
        ```
    - Pod Health:
        - Probes determine the health of the container. It's a periodical diagnostics. (1) Liveness probe. (2) Readiness probe.
        - Probe types: ExecAction: Executes an action inside the cointainer.
        - TCPSocketAction: TCP check against the container's IP address via a specific port.
        - HTTPGetAction. All results: Success. Failure. Unknown.
        - Success: As long as the HTTP GET is successful. Allow for one (1) failure. 
        - livenessProbe: When should a container restart? readinessProbe: When should a container start receiving traffic?
        ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
                name: my-nginx
            spec:
                containers:
                - name: my-nginx
                  image: mginx:alpine
                  livenessProbe:
                    httpGet:
                        path: /index.html
                        port: 80
                    initialDelaySeconds: 15
                    timeoutSeconds: 2
                    periodSeconds: 5
                    failureThreshold: 1
        ```
    - Pod Health in Action:
    - Summary:
        - Pods are the smallest unit of Kubernetes.
        - Containers run within Pods and share a Pod's memory, IP, volumes, and more.
        - Pods can be restarted using different commands.
        - YAML can be used to create a Pod.
        - Health checks provide a manner in which to notify Kubernetes when a Pod has an issue.

- CREATING DEPLOYMENTS:
    - Introduction:
    - Deployments Core Concepts:
        - ReplicaSet is a declarative way to manage Pods.
        - A deployment is a declarative way to manage Pods using a ReplicaSet. With Kubernetes, ReplicaSets came first.
        - ReplicaSets act as a Pod controller. Self-healing mechanism. Ensure the required number of Pods are available.
        - Provide fault tolerance. Can be used to scale Pods. Relies on a Pod template. No need to create Pods directly.
        - A deployment manages Pods. Supports zero-downtime updates by creating and destroying ReplicaSets.
        - Provides rollback functionality. Creates an unique label that is assigned to the ReplicaSet & generated Pods.
        - YAML is very similiar to a ReplicaSet.
    - Creating a Deployment:
        - YAML + kubectl = (Deployment => ReplicaSet => Pod => Container)
        - Defining a deployment: Note the overall deployment 'spec.' Labels can be used to tie things together.
        ```yaml
            apiVersion: apps/v1
            kind: Deployment
            metadata:
            spec:
                replicas: 3
                selector:
                template:
                    spec:
                        containers:
                        - name: X
                        image: nginx:alpine
                        ports:
                        - containerPort: 80
                        resources:
                            limits:
                            memory: "128Mi"
                            cpu: "200m"
        ```
    - kubectl and Deployments:
    ```javascript
        kubectl create -f file.deployment.yml
    ```
    - kubectl Deployments in Action:
        - kubectl "apply" to apply changes. Recommended to use always rather than "create."
    ```javascript
        kubectl get deployment --show-labels
        kubectl get deployment -l app=nginx
        kubectl delete deployment [deployment-name]
        kubectl scale deployment [deployment-name] --replicas=5
        kubectl create -f nginx.deployment.yml --save-config
        kubectl descript [pod | deployment] [pod name | deployment name]
        kubectl apply -f nginx.deployment.yml
    ```
    - Deployment Options:
        - Update exising Pods? Zero downtime deployments allow software updates to be deployed without impact.
        - Default: Rolling update. Blue/green deployments. Canary deployments. Rollbacks.
    - Zero Downtime Deployments in Action:
        - Load balancing does occur.
    - Summary:
        - Pods are deployed, managed, and scaled using deployments and ReplicaSets.
        - Deployments are a higher-level resource that define  one or more Pod templates.
        - The kubectl create or apply commands can be used to run a deployment.

- CREATING SERVICES:
    - Introduction:
        - Service types.
    - Services Core Concepts:
        - A service provides a single point of entry for accessing one or more Pods. Can you rely on a Pod's IP address? No. That's why we need services.
        - Also, a Pod receives its IP address only after it has been scheduled. No way to know prior.
        - Services abstract Pod IP addresses from consumers. Load balances between Pods.
        - Relies on labels to associated a service with a Pod. Node's kube-proxy creates a virtual IP for services.
        - Layer 4. Services are not ephemeral. Creates endpoints which sit between the service & the running Pod.
        - NOTE: Browsers use the same connection over & over.
    - Service Types:
        - Four main service types:
            - Default: ClusterIP: Expose the service on a cluster-internal IP. Only Pods within the cluster can talk to the service.
            - NodePort: Expose the service on each Node's IP at a static port. Allocate a port from range. Each Node proxies the allocated port.
            - Load balancer: Provision an external IP to act as a load balancer for the service. NodePort & ClusterIP Services are created. Each Node proxies the allocated port.
            - ExternalName: Maps a service to a DNS name. Service is simply an alias. A proxy for external, details are hidden from cluster.
    - Creating a Service with kubectl:
        - Access a Pod outside of Kubernetes? Port forwarding.
        ```javascript
            kubectl port-forward pod/[pod-name] 8080:80
            kubectl port-forward deployment/[deployment-name] 8080:80
            kubectl port-forward service/[service-name] 8080:80
        ```
    - Creating a Service with YAML:
        - 'type' defaults to ClusterIP. 
        - Name of Service within metadata ensures a DNS entry. e.g.: Frontend Pod can access backend Pod using name: nginx:port.
        ```yaml
            apiVersion: v1
            kind: Service
            metadata:
                name: nginx
                labels:
                    app: nginx
            spec:
                type:
                selector:
                    app: nginx
                ports:
                - name: http
                port: 80
                targetPort: 80
        ```
    - kubectl and Services:
        - Generate a cluster ip as well. Services exist to give a Pod an IP address. Labels and a selector join to two, Pods & Services, together.
        ```javascript
            kubectl create -f file.service.yml
            kubectl apply -f file.service.yml
            kubectl delete -f file.service.yml
        ```
        - Test a service & a Pod with CURL. use `kubectl exec` to shell into a Pod/Container.
        ```javascript
            kubectl exec [pod-name] --curl -s http://podIp
            kubectl exec [pod-name] -it sh
            apk add curl
            curl -s http://podIp
        ```
    - kubectl Services in Action:
        - `describe` or `get` will provide the Pod IP (podIP) address.
        ```javascript
            kubectl get pod [pod-name] -o yaml
            kubectl exec [pod-name] -it sh
            apk add curl
            curl -s http://podIp
        ```
        - Use the ClusterIP. Obtain the IP from kubectl. You can also use the DNS name. CURL will load balance.
        ```javascript
            kubectl get services
        ```
    - Summary:
        - Pods live and die, so their IP addresses change. Services provide that abstraction that allows us to get to the pods without actually having to know the pod IP addresses.
        - Labels associate a Service to a Pod.
        - Four Types of services:
            - Cluster IPs: Those are only accessible within the cluster, and that's the default.
            - Node ports: Open up a port on the node so we can call into a deployment or a service. 
            - Load balancer: Allow something to be accessed externally.
            - ExterName: Alias, if you will, an external name, which can proxy to an external service. 

- UNDERSTANDING STORAGE OPTIONS:
    - Introduction: 
        - Storing data or state somewhere.
    Storage Core Concepts:
        - How do you store application, state, or data and then even exchange it between pods when you have a Kubernetes cluster? And the answer could be volumes. Or a database.
        - A volume can be used to hold data and state for Pods and containers.
        - Kubernetes supports: Volumes. PersistentVolumes. PersistentVolumeClaims. StorageClasses.
    - Volumes: viewing A Pod's Volumes:
        ```javascript
            kubectl describe pod [pod-name]
            kubectl get pod [pod-name] -o yaml
        ```
    - Volumes in Action:
    - PersistentVolumes and PersistentVolumeClaims:
        - A Persistent Volume (PV) is a cluster-wide storage unit provisioned by an administrator with a lifecycle independent from a Pod.
        - Persistent Volume Claim (PVC) is a request for a storage unit (PV.) Used to "talk to" that storage.
        - Persistent Volume does rely upon a storage provider like NFS, cloud storage, or other options.
    - PersistentVolume and PersistentVolumeClaim YAML:
        - Tough to get the YAML right. [YAML Examples](https://github.com/kubernetes/examples)
        - Create PersistenVolume kind. Define storage capacity. Define access modes. Retain ever after claim is deleted. Reference storage to use.
        ```yaml
            apiVersion: v1
            kind: PersistentVolume
            metadata:
                name: my-pv
            spec:
                capacity: 10Gi
                accessModes:
                    - ReadWriteOnce
                    - ReadOnlyMany
                persistentVolumeReclaimPolicy: Retain
                azureFile:
                    secretName: <azure-secret>
                    shareName:  <name-from-azure>
                    readOnly: false
        ```
        - PersistentVolumeClaim:
            - Defaine a PVC. Define an access mode. Request storage amount.
            ```yaml
                kind: PersistentVolumeClaim
                apiVersion: v1
                metadata:
                    name: pv-dd-account-hdd-5g
                    annotations:
                        volume.beta.kubernetes.io/storage-class: accounthdd
                spec:
                    accessModes:
                        - ReadWriteOnce
                    resources:
                        requests: 5Gi
            ```
            - Reference Claim inside as Pod:
            - Create a Volume that bonds to PersistentVolumeClaim
            ```yaml
                kind: Pod
                apiVersion: v1
                metadata:
                    name: pod-uses-account-hdd-5g
                    labels:
                    name: storage
                spec:
                    containers:
                    - image: nginx
                    bane: az-c-01
                    command:
                    - /bin/sh
                    - -c
                    - while true; do echo $(date)>>
                    /mnt/blobdisk/outfile; sleep 1; done
                
                volumes:
                - name: blobdisk01
                  persistentVolumeClaim:
                    claimName: pv-dd-account-hdd-5g
            ```
    - StorageClasses:
        - A StorageClass (SC) is a type of storage template that can be used to dynamically provision storage. Used with different "classes" of storage.
        - Administrators don't have to set these up in advance. Dynamically creating the persistent volume. Retain storage or delete (default) after PVC is released.
            ```yaml
                apiVersion: storage.k8s.io/v1
                kind: StorageClass
                metadata:
                    name: local-storage
                reclaimPolicy: Retain
                provisioner: kubernetes.io/no-provisioner
                volumeBindingMode: WaitForFirstConsumer
            ```
    - PersistentVolumes in Action:
        - Kind of StatefulState: Manages the deployment & scaling of a set of Pods. A database.
    - Summary:
        - Kubernetes supports several different types of storage:
            - Ephemeral storage (emptyDir). Persistent storage (many options.) PersistentVolumes, PersistentVolumeClaims, & StroageClasses. ConfigMaps (kvp.) Secrets.

- CREATING CONFIGMAPS & SECRETS:
    - Introduction:
    - ConfigMaps Core Concepts:
        - Storing configuration information & providing it to containers. Injection into container. Either entire files or KVPs. Provide on the command line. Or COnfigMap manifest.
        - So, environment variables or a ConfigMap volume.
    - Creating a ConfigMap: Name is very important.
        ```yaml
            apiVersion: v1
            kind: ConfigMap
            metadata:
                name: app-settings
                labels: 
                    app: app-settings
            data:
                enemies: aliens
                lives: "3"
        ```
        - Config file. Environment-variable file.
        ```javascript
            kubectl create configmap [configmap-name] --from-file=[path-to-file]
            kubectl create configmap [configmap-name] --from-env-file=[path-to-file]
            kubectl create configmap [configmap-name] --from-literal=key1=A --from-literal=key2=B
            kubectl get cm [configmap-name] -o yaml
        ```
    - Using a ConfigMap:
    - ConfigMaps in Action:
    - Secrets Core Concepts:
        - Sensitive data. An object that contains a small amount of sensitive data. Kubernetes provides a secret store.
        - Mount secrets in Pods as files or as environment variables. Secrets are stored in tmpfs and not stored on disc.
    - Creating a Secret:
        ```javascript
            kubectl create secret generic my-secret --from-literal=pwd=my-password
            kubectl create secret generic my-secret --from-file=ssh-privatekey=~/.ssh/id_rsa --from-file=ssh-publickey=~/.ssh/id_rsa .pub
            kubectl create secret tls tls-secret --cert=path/to/tls.cert --key=path/to/tls.key
            kubectl get secrets
            kubectl get secrets db-passwords -o yaml
        ```
        ```yaml
            apiVersion: v1
            kind: Secret
            metadata:
                name: db-passwords
            type: Opaque 
            data:
                app-password: cGFzc3dvcmQ=
                admin-password: dmVyeV9zZWNyZQXX=
        ```
    - Using a Secret:
    - Secrets in Action:
    - Summary:

- PUTTING IT ALL TOGETHER:
    - Introduction:
    - Application Overview:
    - YAML Manifests:
    - Running the Application:
    - Troubleshooting Techniques:
    - Troubleshooting Techniques in Action:
    - Summary:

- COURSE SUMMARY: