# Observability

## 1. [Pod Phases](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)

The phase of a Pod is a simple, high-level summary of where the Pod is in its lifecycle.

### Pending

The Pod has been accepted by the Kubernetes system, but one or more of the Container images has not been created. This includes time before being scheduled as well as time spent downloading images over the network, which could take a while.

### ContainerCreating

The images required for the application are being pulled. 

> Somehow, this phase isn't described in the official documentation.

### Running

The Pod has been bound to a node, and all of the Containers have been created. At least one Container is still running, or is in the process of starting or restarting.

### Succeeded

All Containers in the Pod have terminated in success, and will not be restarted.

### Failed

All Containers in the Pod have terminated, and at least one Container has terminated in failure. That is, the Container either exited with non-zero status or was terminated by the system.

### Unknown

For some reason the state of the Pod could not be obtained, typically due to an error in communicating with the host of the Pod.

## 2. [Pod Condition Types](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-conditions)

Found using `kubectl describe po <pod-name>` under the Conditions section.

### PodScheduled

The Pod has been scheduled to a node.

### Ready

The Pod is able to serve requests and should be added to the load balancing pools of all matching Services.

### Initialized

All init containers have started successfully.

### Unschedulable

The scheduler cannot schedule the Pod right now, for example due to lack of resources or other constraints.

### ContainersReady

All containers in the Pod are ready.


## 3. Readiness Probes

### Sample YAML definition

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
```

#### HTTP Test 

```
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
```

#### TCP Test

```
    readinessProbe:
      tcpSocket:
        port: 8080
```

#### Exec Command Test

```
    readinessProbe:
      exec:
        command:
        - cat
        - /app/is_ready
```

#### Additional Options

```
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
      initialDelaySeconds: 10   # start probing after 10 seconds
      periodSeconds: 5          # probe every 5 seconds
      failureThreshold: 8       # probe until 8 failures 
```

## 4. Liveness Probes

Almost exactly the same as Readiness Probe.  
Just change from `readinessProbe` to `livelinessProbe`.

### Sample YAML definition

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /api/healthy
        port: 8080
```

#### HTTP Test 

```
    livenessProbe:
      httpGet:
        path: /api/ready
        port: 8080
```

#### TCP Test

```
    livenessProbe:
      tcpSocket:
        port: 8080
```

#### Exec Command Test

```
    livenessProbe:
      exec:
        command:
        - cat
        - /app/is_ready
```

#### Additional Options

```
    livenessProbe:
      httpGet:
        path: /api/healthy
        port: 8080
      initialDelaySeconds: 10   # start probing after 10 seconds
      periodSeconds: 5          # probe every 5 seconds
      failureThreshold: 8       # probe until 8 failures 
```

## 5. Container Logging

Indicate the container name if the pod has multiple containers.

```
kubectl logs -f <pod-name> <container-name>
kubectl logs -f event-simulator-pod event-simulator
```

## 6. Monitor and Debug Applications

Kubernetes does not come with a built-in monitoring solution.

### Open-source monitoring solutions:

- Metrics Server (CKAD relevant)
- Prometheus
- Elastic Stack

### Proprietary solutions:

- Datadog
- dynatrace

### Metrics Server

- Slimmed down version of the deprecated Heapster.  
- Only one metric server per kubernetes cluster.  
- In-memory monitoring solution only -- does store the metrics on the disk. As a result, there is no historical data.

#### How metrics are generated

Metrics on the pods on the nodes are taken from kubelet (cAdvisor).

cAdvisor is responsible for retrieving performance metrics from pods and exposing them through the kubelet API to meet the metrics available for the metrics server.

> Source: Certified Application Developer (CKAD) with Tests (section 65)

#### Minikube

```
minikube addons enable metrics-server
```

#### Others

```
git clone https://github.com/kubernetes-incubator/metrics-server.git
```

```
> kubectl create -f deploy/1.8+/
clusterrolebinding "metrics-server:system:auth-delegator" created
rolebinding "metrics-server-auth-reader" created
apiservice "v1beta1.metrics.k8s.io" created
serviceaccount "metrics-server" created
deployment "metrics-server" created
serviceaccount "metrics-server" created
clusterrole "system:metrics-server" created
custerrolebinding "system:metrics-server" created
```

#### Viewing metrics

```
> kubectl top node
NAME         CPU(cores)   CPU%      MEMORY(bytes)   MEMORY%
kubemaster   166m         8%        1337Mi          70%
kubenode1    36m          1%        1046Mi          55%
kubenode2    39m          1%        1048Mi          55%
```

```
> kubectl top pod
NAME         CPU(cores)   CPU%      MEMORY(bytes)   MEMORY%
nginx        166m         8%        1337Mi          70%
redis        36m          1%        1046Mi          55%
```
