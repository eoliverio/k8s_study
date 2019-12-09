# Services and Networking

## 1. Service

Shorthand for service is `svc`.

### 1.1. [Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

- NodePort

Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default `ServiceType`.

- ClusterIP

Exposes the Service on each Node's IP at a static port (the `NodePort`). A `ClusterIP` Service, to which the `NodePort` Service routes, is automatically created. You’ll be able to contact the `NodePort` Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.

- LoadBalancer

Exposes the Service externally using a cloud provider’s load balancer. `NodePort` and `ClusterIP` Services, to which the external load balancer routes, are automatically created.

### 1.2. NodePort

> Source: Certified Application Developer (CKAD) with Tests (section 74)

#### 1.2.1. Accessing the application from outside the Node

```
       ┌──┬───────────────────┬─────────────────────────┐
       │  │    192.168.1.2    │                         │
       │  └───────────────────┘       ╭──┬──┬──┬──╮     │
       │                            ╭─┴           ┴─╮   │
       │                           ╭┴  10.244.0.0  ┬╯◄─ ┼ ─ internal
       │                           ╰─┬           ┬─╯    │      pod
       │                             ╰──┴──┴──┴──╯      │    network
       │                                    ╵           │
       │                           ┌─┬──────┴──────┬─┐  │
       ├───────┐   ┌───────────┐   │ │ 10.244.0.2  │ │  │
USER ─ ┤ 30008 ├ ─ ┤  SERVICE  ├ ─ ┤ └─────────────┘ │  │
       ├───────┘   └───────────┘   │  ┌───────────┐  │  │
       │                           │  │ Container │  │  │
       │                           │  └───────────┘  │  │
       │                           │       POD       │  │
       │   NODE                    └─────────────────┘  │
       │                                                │
       └────────────────────────────────────────────────┘
```

#### 1.2.2. Terminologies

```
                                    ┌ ─ (2) Port       
                ClusterIP ─ ┐       ╷       ┌ ─ (1) TargetPort
       ┌────────────────────┴───────┴───────┴─────────────────┐
       │           ┌────────┴───────┴─┐   ┌─┴───────────────┐ │
       ├───────┐   │ ┌──────┴──────┬┴─┤   ├─┴┬────────────┐ │ │
USER ─ ┤ 30008 ├ ─ ┤ │ 10.106.1.12 │80├ ─ ┤80│ 10.244.0.2 │ │ │
       ├──┬────┘   │ └─────────────┴──┤   ├──┴────────────┘ │ │
       │  ╷        │     SERVICE      │   │  ┌───────────┐  │ │
       │  ╷        └──────────────────┘   │  │ Container │  │ │
       │  ╷                               │  └───────────┘  │ │
       │  ╷             NODE              │       POD       │ │
       │  ╷                               └─────────────────┘ │
       └──┬───────────────────────────────────────────────────┘
          └ ─ (3) NodePort
```

TargetPort: where the service forwards the request to 
Port: the port on the service itself
NodePort: the port on the node (Range: 30000-32767)

> Note: these terms are based from the view point of the service

### 1.3. Creating service using YAML definition file

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports
  - targetPort: 80    # targetPort is the same as port if not defined
    port: 80          # port is mandatory
    nodePort: 30008
  selector:
    app: myapp        # labels as defined 
    type: front-end   #   from the pod
```

```
> kubectl create -f service-definition.yaml
service "myapp-service" created
```

### 1.4. Creating service using `kubectl create`

```
kubectl create svc <service-type> <service-name> \
  --node-port=<node-port> \
  --tcp=<port>:<targetPort>
  
kubectl create service nodeport webapp-service \
  --node-port=30008 \
  --tcp=8080:8080 \
  --dry-run -o yaml > svc-definition.yaml
```

### 1.5. Creating service using `--expose` flag in `kubectl run`

If expose is true, a public, external service is created for the container(s) which are run.

```
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
```

> Source: [kubectl run](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run)

### 1.6. Viewing/Testing service

```
> kubectl get svc
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP        16d
myapp-service   NodePort    10.106.127.123   <none>        80:30008/TCP   5m
```

```
> curl http:// 192.168.1.2:30008
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
.
.
```

### 1.7. Sample Use Cases

#### Case 01: (NodePort) multiple similar pods in a single node

In this architecture, where all pods have the labels as defined in the service selector, the service automatically sets all the pods as the endpoints to forward the external requests coming from the user.

Algorithm: Random  
SessionAffinity: Yes

```
                       ┌───────────┐
                       │   USER    │
                       └─────┬─────┘
                             ╷
┌───────────────┬────────┬───┴───┬────────────────────────┐
│  192.168.1.2  │        │ 30008 │                        │
├───────────────┘        └───────┘                        │
│                            ╵                            │
│                      ┌─────┴─────┐                      │
│                      │  SERVICE  │                      │
│                      └─────┬─────┘                      │
│                            ╵                            │
│          ┌ ─ ─ ─ ─ ─ ─ ─ ─ ┼ ─ ─ ─ ─ ─ ─ ─ ─ ┐          │
│          ╷                 ╷                 ╷          │
│  ┌┬──────┴──────┬┐ ┌┬──────┴──────┬┐ ┌┬──────┴──────┬┐  │
│  ││ 10.244.0.2  ││ ││ 10.244.0.2  ││ ││ 10.244.0.2  ││  │
│  │└─────────────┘│ │└─────────────┘│ │└─────────────┘│  │
│  │ ┌───────────┐ │ │ ┌───────────┐ │ │ ┌───────────┐ │  │
│  │ │ Container │ │ │ │ Container │ │ │ │ Container │ │  │
│  │ └───────────┘ │ │ └───────────┘ │ │ └───────────┘ │  │
│  │      POD      │ │      POD      │ │      POD      │  │
│  └───────────────┘ └───────────────┘ └───────────────┘  │
│                           NODE                          │
└─────────────────────────────────────────────────────────┘
```

> Note: No addition configuration required to make this happen

#### Case 02: (NodePort) distributed pods across multiple nodes

In this case, kubernetes automatically creates a service across all nodes in the cluster and maps the TargetPort to the same NodePort on all the nodes in the cluster.

This way the application can be accessed thtough the IP of any node in the cluster and using the same port number (30008).

```
                          ┌───────────┐
                          │   USER    │
                          └─────┬─────┘
                                ╷           
                 ┌─ ─ ─ ─ ─ ─ ─ ┴ ─ ─ ─┬─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┐
                 ╷                     ╷                     ╷
┌─────────────┬──┴──┐ ┌─────────────┬──┴──┐ ┌─────────────┬──┴──┐
│ 192.168.1.2 │30008│ │ 192.168.1.3 │30008│ │ 192.168.1.4 │30008│
├─────────────┴──┬──┤ ├─────────────┴──┬──┤ ├─────────────┴──┬──┤
│                ╵  │ │                ╵  │ │                ╵  │
│                └ ─│─│─ ─ ─ ─ ─ ─ ─ ─ ┴ ─│─│─ ─ ─ ─ ─ ─ ─ ─ ┘  │
│                   │ │         ┌─ ─ ─ ┘  │ │                   │
│ ┌─────────────────┴─┴─────────┴─────────┴─┴─────────────────┐ │
│ │                          SERVICE                          │ │
│ └─────────────────┬─┬─────────┬─────────┬─┬─────────────────┘ │
│                   │ │         ╷         │ │                   │
│         ┌ ─ ─ ─ ─ │─│ ─ ─ ─ ─ ┬ ─ ─ ─ ─ │─│ ─ ─ ─ ─ ┐         │
│         ╷         │ │         ╷         │ │         ╷         │
│ ┌┬──────┴──────┬┐ │ │ ┌┬──────┴──────┬┐ │ │ ┌┬──────┴──────┬┐ │
│ ││ 10.244.0.2  ││ │ │ ││ 10.244.0.3  ││ │ │ ││ 10.244.0.4  ││ │
│ │└─────────────┘│ │ │ │└─────────────┘│ │ │ │└─────────────┘│ │
│ │ ┌───────────┐ │ │ │ │ ┌───────────┐ │ │ │ │ ┌───────────┐ │ │
│ │ │ Container │ │ │ │ │ │ Container │ │ │ │ │ │ Container │ │ │
│ │ └───────────┘ │ │ │ │ └───────────┘ │ │ │ │ └───────────┘ │ │
│ │      POD      │ │ │ │      POD      │ │ │ │      POD      │ │
│ └───────────────┘ │ │ └───────────────┘ │ │ └───────────────┘ │
│        NODE       │ │        NODE       │ │        NODE       │
└───────────────────┘ └───────────────────┘ └───────────────────┘
```

## 2. ClusterIP

## 3. LoadBalancer
