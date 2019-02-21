# Kubernetes Cheatsheet
brought to you by [Sematext](https://sematext.com/kubernetes) ![](https://sematext.com/wp-content/uploads/2017/01/octi-footer-circle.png) 
# Client Configuration

- Setup autocomplete in bash; bash-completion package should be installed first
  
  `source <(kubectl completion bash)`

- View Kubernetes config
  
  `kubectl config view`

- View specific config items by json path

  `kubectl config view -o jsonpath='{.users[?(@.name == "k8s")].user.password}'`
 
# Manage Resources
- Get documentation for pod or service

  `kubectl explain pods,svc`

- Create resource(s) like pods, services or daemonsets

  `kubectl create -f ./my-manifest.yaml`

- Apply a configuration to a resource

  `kubectl apply -f ./my-manifest.yaml`

- Start a single instance of Nginx

  `kubectl run nginx --image=nginx`

- Create a secret with several keys

	```
	cat <<EOF | kubectl create -f -
	apiVersion: v1
	kind: Secret
	metadata:
	  name: mysecret
	type: Opaque
	data:
	  password: $(echo -n "s33msi4" | base64)
	  username: $(echo -n "jane" | base64)
	EOF
	```

- Delete a resource
  
   `kubectl delete -f ./my-manifest.yaml`

# Viewing, Finding Resources

- List all services in the namespace

  `kubectl get services`

- List all pods in all namespaces in wide format

  `kubectl get pods -o wide --all-namespaces`

- List all pods in json (or yaml) format

  `kubectl get pods -o json`

- Describe resource details (node, pod, svc)

  `kubectl describe nodes my-node`

- List services sorted by name

  `kubectl get services --sort-by=.metadata.name`

- List pods sorted by restart count

  `kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'`

- Rolling update pods for frontend-v1

  `kubectl rolling-update frontend-v1 -f frontend-v2.json`

- Scale a replicaset named 'foo' to 3

  `kubectl scale --replicas=3 rs/foo`

- Scale a resource specified in "foo.yaml" to 3

  `kubectl scale --replicas=3 -f foo.yaml`

- Execute a command in every pod / replica 

  `for i in 0 1; do kubectl exec foo-$i -- sh -c 'echo $(hostname) > /usr/share/nginx/html/index.html'; done`

# Monitoring & Logging

- Deploy Heapster from Github repository 
https://github.com/kubernetes/heapster

  `kubectl create -f deploy/kube-config/standalone/`

- Show metrics for nodes

  `kubectl top node`
  
  `kubectl top node my-node-1`

- Show metrics for pods
 
  `kubectl top pod`
  
  `kubectl top pod my-pod-1`

- Show metrics for a given pod and its containers

  `kubectl top pod pod_name --containers`

- Dump pod logs (stdout)

  `kubectl logs pod_name`

- Stream pod container logs 
(stdout, multi-container case)

  `kubectl logs -f pod_name -c my-container`

- Create a daemonset from stdin. The example deploys [Sematext Docker Agent](https://sematext.com/kuberntes) to all nodes for the cluster-wide collection of metrics, logs and events. There is NO need to deploy cAdvisor, Heapster, Prometheus, Elasticsearch, Grafana, InfluxDb on your local nodes. Please replace YOUR_SPM_DOCKER_TOKEN and YOUR_LOGSENE_TOKEN with your tokens created in [Sematext Cloud UI](https://apps.sematext.com/ui/integrations/create/docker) before you run the command. 

```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: sematext-agent
spec:
  template:
    metadata:
      labels:
        app: sematext-agent
    spec:
      nodeSelector: {}
      hostNetwork: true
      dnsPolicy: "ClusterFirst"
      restartPolicy: "Always"
      containers:
      - name: sematext-agent
        image: sematext/sematext-agent-docker:latest
        imagePullPolicy: "Always"
        env:
        - name: SPM_TOKEN
          value: "YOUR_SPM_DOCKER_TOKEN"
        - name: LOGSENE_TOKEN
          value: "YOUR_LOGSENE_TOKEN"
        volumeMounts:
          - mountPath: /var/run/docker.sock
            name: docker-sock
          - mountPath: /etc/localtime
            name: localtime
        securityContext:
          privileged: true
      volumes:
        - name: docker-sock
          hostPath:
            path: /var/run/docker.sock
        - name: localtime
          hostPath:
            path: /etc/localtime
EOF
```


============================================================================

**** http://www.openkb.info/2018/11/kubernetes-tutorial-cheat-sheet.html ****

Kubernates ChetSheet from OPENKB Website

Kubernetes Tutorial cheat sheet
This article records the commands for Kubernetes Tutorial online sessions:https://kubernetes.io/docs/tutorials/kubernetes-basics/

1. Create a Cluster
https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/

1.1 Show minikube version

$ minikube version
minikube version: v0.28.2

1.2 Start minikube cluster

$ minikube start

Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.

1.3 Show client and server side version

$ kubectl version

Client Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.0", GitCommit:"91e7b4fd31fcd3d5f436da26c980becec37ceefe", GitTreeState:"clean", BuildDate:"2018-06-27T20:17:28Z", 
GoVersion:"go1.10.2", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", BuildDate:"2018-04-10T12:46:31Z", 
GoVersion:"go1.9.4", Compiler:"gc", Platform:"linux/amd64"}

1.4 View cluster information

$ kubectl cluster-info

Kubernetes master is running at https://172.17.0.109:8443
 
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

1.5 List all Nodes in the cluster

$ kubectl get nodes

NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    2m        v1.10.0
(Currently only 1 node named "minikube" is in the cluster, and its STATUS is ready to accept applications for deployment.)

2. Deploy an App
https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-interactive/ 

2.1 Create a deployment for application

$ kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1--port=8080

deployment.apps/kubernetes-bootcamp created

This command creates a deployment named "kubernetes-bootcamp" based on Docker image from "gcr.io/google-samples/kubernetes-bootcamp" with tag "v1". And this application will run on 
port 8080.

2.2 List deployment

$ kubectl get deployments

NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           2m
Above shows there is 1 deployment running a single instance of the application.

2.3 Create a proxy

$ kubectl proxy

Starting to serve on 127.0.0.1:8001
This proxy creates the connection between our host (the online terminal) and the Kubernetes cluster.

2.4 Query the API through proxy

$ curl http://localhost:8001/version
{
  "major": "1",
  "minor": "10",
  "gitVersion": "v1.10.0",
  "gitCommit": "fc32d2f3698e36b93322a3465f63a14e9f0eaead",
  "gitTreeState": "clean",
  "buildDate": "2018-04-10T12:46:31Z",
  "goVersion": "go1.9.4",
  "compiler": "gc",
  "platform": "linux/amd64"
}

2.5 Save the POD name into environment variable $POD_NAME

$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metdata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME

Name of the Pod: kubernetes-bootcamp-5c69669756-4qdmk

2.6 Query the API through proxy to specific POD

$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/

Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-4qdmk | v=1

3. Explorer App

https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-interactive/

3.1 List PODs

$ kubectl get pods

NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5c69669756-5mxvh   0/1       Pending   0          5s

Currently only 1 POD named "kubernetes-bootcamp-5c69669756-5mxvh" is running.

If you want to list all the containers inside each pod:

kubectl get pods --all-namespaces -o=custom-columns=NameSpace:.metadata.namespace,NAME:.metadata.name,CONTAINERS:.spec.containers[*].name

3.2 View Container information inside the POD

$ kubectl describe pods

Name:           kubernetes-bootcamp-5c69669756-5mxvh
Namespace:      default
Node:           minikube/172.17.0.17
Start Time:     Thu, 01 Nov 2018 21:20:21 +0000
Labels:         pod-template-hash=1725225312
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-5c69669756
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://f9d6331b1c7dcc24e6c18a920fa65f2344c2421f5f394665668fd406c5641e22
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 01 Nov 2018 21:20:22 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-b8t9g (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-b8t9g:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-b8t9g
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                 Age              From               Message
  ----     ------                 ----             ----               -------
  Warning  FailedScheduling       1m (x4 over 1m)  default-scheduler  0/1 nodes are available: 1 node(s) were not ready.
  Normal   Scheduled              1m               default-scheduler  Successfully assigned kubernetes-bootcamp-5c69669756-5mxvh to minikube
  Normal   SuccessfulMountVolume  1m               kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-b8t9g"
  Normal   Pulled                 1m               kubelet, minikube  Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal   Created                1m               kubelet, minikube  Created container
  Normal   Started                1m               kubelet, minikube  Started container

3.3 View Container logs

$ kubectl logs $POD_NAME

Kubernetes Bootcamp App Started At: 2018-11-01T21:20:22.891Z | Running On:  kubernetes-bootcamp-5c69669756-5mxvh
 
Running On: kubernetes-bootcamp-5c69669756-5mxvh | Total Requests: 1 | App Uptime: 309.865 seconds | Log Time: 2018-11-01T21:25:32.756Z
Here we only have 1 Container inside the POD, so no need to specify the Container name.

The usage for "kubectl logs" is:

kubectl logs [-f] [-p] (POD | TYPE/NAME) [-c CONTAINER] [options]
So we can specify both POD and Container name as well:

kubectl logs $POD_NAME -c kubernetes-bootcamp

3.4 Execute command inside the Container

$ kubectl exec $POD_NAME env

PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubernetes-bootcamp-5c69669756-5mxvh
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
NPM_CONFIG_LOGLEVEL=info
NODE_VERSION=6.3.1
HOME=/root

3.5 Start an open console inside the Container

$ kubectl exec -ti $POD_NAME bash

root@kubernetes-bootcamp-5c69669756-5mxvh:/#
If you want to open a console to specific container inside the POD:

kubectl exec -ti $POD_NAME -c <Container_Name> bash

4. Expose App

https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-interactive/

4.1 List Services

$ kubectl get services

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   17m
The default service(created by minikube) named "kubernetes" is based on TYPE "ClusterIP".

4.2 Expose a new Service

$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
service/kubernetes-bootcamp exposed
Above command creates a new service named "kubernetes-bootcamp" based on TYPE "NodePort" on internal port 8080. The external port is 30336. 

$ kubectl get services

NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1      <none>        443/TCP          15s
kubernetes-bootcamp   NodePort    10.100.77.17   <none>        8080:30336/TCP   1s

4.3 View Service information

$ kubectl describe services/kubernetes-bootcamp

Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.100.77.17
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30336/TCP
Endpoints:                172.18.0.2:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

4.4 Fetch the external port of the Service

$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index.spec.ports 0).nodePort}}')
$ echo NODE_PORT=$NODE_PORT

NODE_PORT=30336
Test from IP of the Node and the external port:

$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-f7d5c | v=1

4.5 Fetch the label from the deployment

$ kubectl describe deployment|grep -i label

Labels:                 run=kubernetes-bootcamp
  Labels:  run=kubernetes-bootcamp

4.6 List PODs and Services based on label

$ kubectl get pods -l run=kubernetes-bootcamp

NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5c69669756-f7d5c   1/1       Running   0          17m
$ kubectl get services -l run=kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.100.77.17   <none>        8080:30336/TCP   17m

4.7 Apply a new label to a POD

$ kubectl label pod $POD_NAME app=v1

pod/kubernetes-bootcamp-5c69669756-f7d5c labeled
Check the new label of the POD:

$ kubectl describe pods $POD_NAME

Name:           kubernetes-bootcamp-5c69669756-f7d5c
Namespace:      default
Node:           minikube/172.17.0.18
Start Time:     Thu, 01 Nov 2018 22:22:22 +0000
Labels:         app=v1
                pod-template-hash=1725225312
                run=kubernetes-bootcamp
...

4.8 Delete a Service based on label

$ kubectl delete service -l run=kubernetes-bootcamp
service "kubernetes-bootcamp" deleted

4.9 Confirm a Service is removed

$ kubectl get services

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   29m

$ curl $(minikube ip):$NODE_PORT

curl: (7) Failed to connect to 172.17.0.18 port 30336: Connection refused
Confirm the App is still running inside POD:

$ kubectl exec -ti $POD_NAME curl localhost:8080

Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-f7d5c | v=1

5. Scale App

https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-interactive/

5.1 Scale up the deployment to 4 replicas

$ kubectl scale deployments/kubernetes-bootcamp --replicas=4

deployment.extensions/kubernetes-bootcamp scaled

$ kubectl get deployments

NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4         4         4            4           1m
Confirm that more PODs are running now:

$ kubectl get pods -o wide

NAME                                   READY     STATUS    RESTARTS   AGE       IP      NODE
kubernetes-bootcamp-5c69669756-5dcc6   1/1       Running   0          1m        172.18.0.7   minikube
kubernetes-bootcamp-5c69669756-8bbrg   1/1       Running   0          2m        172.18.0.2   minikube
kubernetes-bootcamp-5c69669756-9jcgl   1/1       Running   0          1m        172.18.0.5   minikube
kubernetes-bootcamp-5c69669756-s7vlm   1/1       Running   0          1m        172.18.0.6   minikube

5.2 Check event log

$ kubectl describe deployments/kubernetes-bootcamp
...
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  3m    deployment-controller  Scaled up replica set kubernetes-bootcamp-5c69669756 to 1
  Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set kubernetes-bootcamp-5c69669756 to 4

5.3 Find the external port for the Service

$ kubectl describe services/kubernetes-bootcamp

Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.107.230.53
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30798/TCP
Endpoints:                172.18.0.2:8080,172.18.0.5:8080,172.18.0.6:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index.spec.ports 0).nodePort}}')

$ echo NODE_PORT=$NODE_PORT

NODE_PORT=30798

$ curl $(minikube ip):$NODE_PORT

Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5c69669756-9jcgl | v=1

5.4 Scale down the deployment to 2 replicas

$ kubectl scale deployments/kubernetes-bootcamp --replicas=2

deployment.extensions/kubernetes-bootcamp scaled

$ kubectl get deployments

NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   2         2         2            2           8m

$ kubectl get pods -o wide

NAME                                   READY     STATUS        RESTARTS   AGE       IP          NODE
kubernetes-bootcamp-5c69669756-5dcc6   1/1       Terminating   0          7m        172.18.0.7   minikube
kubernetes-bootcamp-5c69669756-8bbrg   1/1       Running       0          8m        172.18.0.2   minikube
kubernetes-bootcamp-5c69669756-9jcgl   1/1       Terminating   0          7m        172.18.0.5   minikube
kubernetes-bootcamp-5c69669756-s7vlm   1/1       Running       0          7m        172.18.0.6   minikube
Eventually the 2 "Terminating" PODs will be gone:

$ kubectl get pods -o wide

NAME                                   READY     STATUS    RESTARTS   AGE       IP      NODE
kubernetes-bootcamp-5c69669756-8bbrg   1/1       Running   0          12m       172.18.0.2   minikube
kubernetes-bootcamp-5c69669756-s7vlm   1/1       Running   0          11m       172.18.0.6   minikube

6. Update App

https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-interactive/

6.1 List current Deployment and PODs

$ kubectl get deployments

NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4         4         4            4           15s

$ kubectl get pods

NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5c69669756-4gvjd   1/1       Running   0          18s
kubernetes-bootcamp-5c69669756-j8wf8   1/1       Running   0          18s
kubernetes-bootcamp-5c69669756-m5fnv   1/1       Running   0          18s
kubernetes-bootcamp-5c69669756-qlqsj   1/1       Running   0          18s

6.2 Update the image for the Deployment

$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
deployment.extensions/kubernetes-bootcamp image updated

6.3 Monitor the PODs status

$ kubectl get pods

NAME                                   READY     STATUS        RESTARTS   AGE
kubernetes-bootcamp-5c69669756-4gvjd   1/1       Running       0          3m
kubernetes-bootcamp-5c69669756-j8wf8   1/1       Terminating   0          3m
kubernetes-bootcamp-5c69669756-m5fnv   1/1       Terminating   0          3m
kubernetes-bootcamp-5c69669756-qlqsj   1/1       Running       0          3m
kubernetes-bootcamp-7799cbcb86-7c6h7   1/1       Running       0          2s
kubernetes-bootcamp-7799cbcb86-xwglv   1/1       Running       0          2s

$ kubectl get pods

NAME                                   READY     STATUS        RESTARTS   AGE
kubernetes-bootcamp-5c69669756-4gvjd   1/1       Terminating   0          3m
kubernetes-bootcamp-5c69669756-j8wf8   1/1       Terminating   0          3m
kubernetes-bootcamp-5c69669756-m5fnv   1/1       Terminating   0          3m
kubernetes-bootcamp-5c69669756-qlqsj   1/1       Terminating   0          3m
kubernetes-bootcamp-7799cbcb86-6rk6l   1/1       Running       0          11s
kubernetes-bootcamp-7799cbcb86-7c6h7   1/1       Running       0          13s
kubernetes-bootcamp-7799cbcb86-x4cgg   1/1       Running       0          11s
kubernetes-bootcamp-7799cbcb86-xwglv   1/1       Running       0          13s

$ kubectl get pods

NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-7799cbcb86-6rk6l   1/1       Running   0          2m
kubernetes-bootcamp-7799cbcb86-7c6h7   1/1       Running   0          2m
kubernetes-bootcamp-7799cbcb86-x4cgg   1/1       Running   0          2m
kubernetes-bootcamp-7799cbcb86-xwglv   1/1       Running   0          2m

6.4 Check the rollout status

$ kubectl rollout status deployments/kubernetes-bootcamp

deployment "kubernetes-bootcamp" successfully rolled out
And check the image for each container:

$ kubectl describe pods|grep Image:
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image:          jocatalin/kubernetes-bootcamp:v2

6.5 Another WRONG image update

$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10

deployment.extensions/kubernetes-bootcamp image updated

$ kubectl get deployments

NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4         5         2            3           12m

$ kubectl get pods

NAME                                   READY     STATUS         RESTARTS   AGE
kubernetes-bootcamp-5f76cd7b94-6smkb   0/1       ErrImagePull   0          34s
kubernetes-bootcamp-5f76cd7b94-sbk7t   0/1       ErrImagePull   0          34s
kubernetes-bootcamp-7799cbcb86-7c6h7   1/1       Running        0          9m
kubernetes-bootcamp-7799cbcb86-x4cgg   1/1       Running        0          9m
kubernetes-bootcamp-7799cbcb86-xwglv   1/1       Running        0          9m

6.6 Find out the root cause

$ kubectl describe pods|grep Failed
  Warning  Failed                 5m (x4 over 6m)   kubelet, minikube  Failed to pull image "gcr.io/google-samples/kubernetes-bootcamp:v10": rpc error: code = Unknown desc 
=unauthorized: authentication required
  Warning  Failed                 5m (x4 over 6m)   kubelet, minikube  Error: ErrImagePull
  Warning  Failed                 4m (x6 over 6m)   kubelet, minikube  Error: ImagePullBackOff
  Warning  Failed                 5m (x4 over 6m)   kubelet, minikube  Failed to pull image "gcr.io/google-samples/kubernetes-bootcamp:v10": rpc error: code = Unknown desc 
=unauthorized: authentication required
  Warning  Failed                 5m (x4 over 6m)   kubelet, minikube  Error: ErrImagePull
  Warning  Failed                 4m (x6 over 6m)   kubelet, minikube  Error: ImagePullBackOff
There is no image called v10 in the repository.

6.7 Rollback to previous working version

$ kubectl rollout undo deployments/kubernetes-bootcamp

deployment.extensions/kubernetes-bootcamp
Confirm the rollback is done:

$ kubectl get pods

NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-7799cbcb86-7c6h7   1/1       Running   0          19m
kubernetes-bootcamp-7799cbcb86-8d5h6   1/1       Running   0          1m
kubernetes-bootcamp-7799cbcb86-x4cgg   1/1       Running   0          19m
kubernetes-bootcamp-7799cbcb86-xwglv   1/1       Running   0          19m
 
$ kubectl describe pods |grep Image:
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image:          jocatalin/kubernetes-bootcamp:v2
    Image:          jocatalin/kubernetes-bootcamp:v2

