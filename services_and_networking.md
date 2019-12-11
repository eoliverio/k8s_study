# SERVICES AND NETWORKING

## 1. Service

Shorthand for service is `svc`.

### 1.1. Handing Services using `kubectl`

#### 1.1.1. Creating `NodePort` service using `kubectl create`

```
kubectl create svc <service-type> <service-name> \
  --node-port=<node-port> \
  --tcp=<port>:<targetPort>
  
kubectl create service nodeport webapp-service \
  --node-port=30008 \
  --tcp=8080:8080 \
  --dry-run -o yaml > svc-definition.yaml
```

#### 1.1.2. Creating service using `--expose` flag in `kubectl run`

If expose is true, a public, external service is created for the container(s) which are run.

```
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
```

> Source: [kubectl run](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run)

#### 1.1.3. Viewing/Testing service

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

### 1.2. [Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

#### 1.2.1. NodePort

Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default `ServiceType`.

##### Accessing the application from outside the Node

> Source: Certified Application Developer (CKAD) with Tests (section 74)
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

##### Terminologies
> Source: Certified Application Developer (CKAD) with Tests (section 74)

```
                                    ┌ ─ (2) Port       
                ClusterIP ─ ┐       ╷       ┌ ─ (1) TargetPort
       ┌────────────────────┴───────┴───────┴─────────────────┐
       │           ┌────────┴───────┴─┐   ┌─┴───────────────┐ │
       ├───────┐   │ ┌──────┴──────┬┴─┤   ├─┴┬────────────┐ │ │
USER ─ ┤ 30008 ├ ─ ┤ │ 10.106.1.12 │80├ ─ ┤80│ 10.244.0.2 │ │ │
       ├───┬───┘   │ └─────────────┴──┤   ├──┴────────────┘ │ │
       │   ╷       │     SERVICE      │   │  ┌───────────┐  │ │
       │   ╷       └──────────────────┘   │  │ Container │  │ │
       │   ╷                              │  └───────────┘  │ │
       │   ╷            NODE              │       POD       │ │
       │   ╷                              └─────────────────┘ │
       └───┬──────────────────────────────────────────────────┘
           └ ─ (3) NodePort
```

`TargetPort`: where the service forwards the request to  
`Port`: the port on the service itself  
`NodePort`: the port on the node (Range: 30000-32767)

> Note: these terms are based from the view point of the service

##### Creating `NodePort` service using YAML definition file

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
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

##### Sample Use Cases

- Case 01: (NodePort) multiple similar pods in a single node

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

- Case 02: (NodePort) distributed pods across multiple nodes

In this case, kubernetes automatically creates a service across all nodes in the cluster and maps the TargetPort to the same NodePort on all the nodes in the cluster.

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

This way the application can be accessed through the IP of any node in the cluster and using the same port number (30008). For example:

```
> curl http://192.168.1.2:30008

> curl http://192.168.1.3:30008

> curl http://192.168.1.4:30008
```

#### 1.2.2. ClusterIP

Exposes the Service on each Node's IP at a static port (the `NodePort`). A `ClusterIP` Service, to which the `NodePort` Service routes, is automatically created. You’ll be able to contact the `NodePort` Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.

##### ClusterIP Diagram
```
┌┬─────────────┬┐ ┌┬─────────────┬┐ ┌┬─────────────┬┐
││ 10.244.0.3  ││ ││ 10.244.0.4  ││ ││ 10.244.0.5  ││
│└─────────────┘│ │└─────────────┘│ │└─────────────┘│
│ ┌───────────┐ │ │ ┌───────────┐ │ │ ┌───────────┐ │
│ │ Container │ │ │ │ Container │ │ │ │ Container │ │
│ └───────────┘ │ │ └───────────┘ │ │ └───────────┘ │
│      POD      │ │      POD      │ │      POD      │
└───────┬───────┘ └───────┬───────┘ └───────┬───────┘
        ╰─────────────────┼─────────────────╯
            ┌─────────────┴─────────────┐
            │        back-end svc       │
            └─────────────┬─────────────┘
        ╭─────────────────┼─────────────────╮
┌┬──────┴──────┬┐ ┌┬──────┴──────┬┐ ┌┬──────┴──────┬┐
││ 10.244.0.6  ││ ││ 10.244.0.7  ││ ││ 10.244.0.8  ││
│└─────────────┘│ │└─────────────┘│ │└─────────────┘│
│ ┌───────────┐ │ │ ┌───────────┐ │ │ ┌───────────┐ │
│ │ Container │ │ │ │ Container │ │ │ │ Container │ │
│ └───────────┘ │ │ └───────────┘ │ │ └───────────┘ │
│      POD      │ │      POD      │ │      POD      │
└───────┬───────┘ └───────┬───────┘ └───────┬───────┘
        ╰─────────────────┼─────────────────╯
            ┌─────────────┴─────────────┐
            │       front-end svc       │
            └─────────────┬─────────────┘
        ╭─────────────────┼─────────────────╮
┌┬──────┴──────┬┐ ┌┬──────┴──────┬┐ ┌┬──────┴──────┬┐
││ 10.244.0.9  ││ ││ 10.244.0.10 ││ ││ 10.244.0.11 ││
│└─────────────┘│ │└─────────────┘│ │└─────────────┘│
│ ┌───────────┐ │ │ ┌───────────┐ │ │ ┌───────────┐ │
│ │ Container │ │ │ │ Container │ │ │ │ Container │ │
│ └───────────┘ │ │ └───────────┘ │ │ └───────────┘ │
│      POD      │ │      POD      │ │      POD      │
└───────────────┘ └───────────────┘ └───────────────┘
```

##### Creating `ClusterIP` service using YAML definition file

```
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
  - targetPort: 80
    port: 80
  selector:
    app: myapp
    type: backend
```
> Note: `ClusterIP` is the default `spec.type` value.

#### 1.2.3. LoadBalancer

Exposes the Service externally using a cloud provider’s load balancer. `NodePort` and `ClusterIP` Services, to which the external load balancer routes, are automatically created.

# 2. Ingress

## 2.1. Ingress Controller

This is a special build of NGINX built specifically to be used as an ingress-controller in Kubernetes.

```
 Deployment: to run a nginx-ingress image
╔══════════════════════════════════════════════════════════╗  
║ apiVersion: extensions/v1beta1                           ║  Service: to expose the nginx-ingress
║ kind: Deployment                                         ║  ╔══════════════════════════════════════╗
║ metadata:                                                ║  ║ apiVersion: v1                       ║
║   name: nginx-ingress-controller                         ║  ║ kind: ConfigMap                      ║
║ spec:                                                    ║  ║ metadata:                            ║
║   replicas: 1                                            ║  ║   name: nginx-configuration          ║
║   selector:                                              ║  ╚══════════════════════════════════════╝
║     matchLabels:                                         ║  
║       name: nginx-ingress                                ║  ConfigMap to feed nginx config data
║   template:                                              ║  ╔══════════════════════════════════════╗
║     metadata:                                            ║  ║ apiVersion: v1                       ║
║       labels:                                            ║  ║ ind: Service                         ║
║         name: nginx-ingress                              ║  ║ etadata:                             ║
║     spec:                                                ║  ║  name: nginx-ingress                 ║
║       containers:                                        ║  ║ pec:                                 ║
║       - name: nginx-ingress-controller                   ║  ║  type: NodePort                      ║
║         image: >                                         ║  ║  ports:                              ║
║           quay.io/kubernetes-ingress-controller/         ║  ║  - port: 80                          ║
║           nginx-ingress-controller:0.2.0 ║               ║  ║    targetPort: 80                    ║
║       args:                                              ║  ║    protocol: TCP                     ║
║       - /nginx-ingress-controller                        ║  ║    name: http                        ║
║       - --configmap=$(POD_NAMESPACE)/nginx-configuration ║  ║  - port: 443                         ║
║       env:                                               ║  ║    targetPort: 443                   ║
║       - name: POD_NAME                                   ║  ║    protocol: TCP                     ║
║         valueFrom:                                       ║  ║    name: https                       ║
║           fieldRef:                                      ║  ║  selector:                           ║
║              fieldPath: metadata.name                    ║  ║    name: nginx-ingress               ║
║       - name: POD_NAMESPACE                              ║  ╚══════════════════════════════════════╝
║         valueFrom:                                       ║  
║           fieldRef:                                      ║  ServiceAccount:
║              fieldPath: metadata.namespace               ║  permission access all of these objects
║       ports:                                             ║  ╔══════════════════════════════════════╗
║       - name: http                                       ║  ║ apiVersion: v1                       ║
║         containerPort: 80                                ║  ║ kind: ServiceAccount                 ║
║       - name: https:                                     ║  ║ metadata:                            ║
║         containerPort: 443                               ║  ║   name: nginx-ingress-serviceaccount ║
╚══════════════════════════════════════════════════════════╝  ╚══════════════════════════════════════╝
```

## 2.2. Ingress Resources

### Case 01: single backend
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
    servicePort: 80
```

### Case 02: multiple paths

- `www.my-online-store.com/wear`
- `www.my-online-store.com/watch`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 80
      - path: /watch
        backend:
          serviceName: watch-service
          servicePort: 80
```

### Case 03: multiple domains

- `wear.my-online-store.com`
- `watch.my-online-store.com`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 80
  - host: watch.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: watch-service
          servicePort: 80
```

# 3. Network Policies

## 3.1. Ingress and Egress
```
┌─────────────────────┬─────────────────────┬─────────────────────┐
│ Web Server POV      │ API Server POV      │ DB Server POV       │
├─────────────────────┼─────────────────────┼─────────────────────┤
│        ┌───────┐    │        ┌───────┐    │        ┌───────┐    │
│        │ User  │    │        │ User  │    │        │ User  │    │
│        └──┬─▲──┘    │        └──┬─▲──┘    │        └──┬─▲──┘    │
│ ┌─────────┤ ╵       │           │ ╵       │           │ ╵       │
│ │ Ingress │ ╵       │     ┌─────▼─┴─────┐ │     ┌─────▼─┴─────┐ │
│ └─────────┤ ╵       │     │ Web Server  │ │     │ Web Server  │ │
│     ╔═════▼═╧═════╗ │     └─────┬─▲─────┘ │     └─────┬─▲─────┘ │
│     ║ Web Server  ║ │ ┌─────────┤ ╵       │           │ ╵       │
│     ╚═════╤═▲═════╝ │ │ Ingress │ ╵       │     ┌─────▼─┴─────┐ │
│  ┌────────┤ ╵       │ └─────────┤ ╵       │     │ API Server  │ │
│  │ Egress │ ╵       │     ╔═════▼═╧═════╗ │     └─────┬─▲─────┘ │ 
│  └────────┤ ╵       │     ║ API Server  ║ │ ┌─────────┤ ╵       │
│     ┌─────▼─┴─────┐ │     ╚═════╤═▲═════╝ │ │ Ingress │ ╵       │
│     │ API Server  │ │  ┌────────┤ ╵       │ └─────────┤ ╵       │
│     └─────┬─▲─────┘ │  │ Egress │ ╵       │     ╔═════▼═╧═════╗ │
│           │ ╵       │  └────────┤ ╵       │     ║  DB Server  ║ │
│     ┌─────▼─┴─────┐ │     ┌─────▼─┴─────┐ │     ╚═════════════╝ │
│     │  DB Server  │ │     │  DB Server  │ │                     │
│     └─────────────┘ │     └─────────────┘ │                     │
└─────────────────────┴─────────────────────┴─────────────────────┘
```   

## 3.2. Network Policy

Kubernetes is configured by default with an `All Allow` rule that allows traffic from any pod to any other pod or services within the cluster.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: db-policy      DB pod
spec:                ┌────────────┐
  podSelector:       │ labels:    │   # apply policy to pods 
    matchLabels:  ┌──┼── role: db │   #   whose role label is db
      role: db ◄──┘  └────────────┘
  policyTypes:              
  - Ingress
  ingress:
  - from:                 # only accept traffic
    - podSelector:        #   from pods whose
        matchLabels:      #   name label is
          name: api-pod   #   api-pod
    ports:
    - protocol: TCP
      port: 3306
```

Network Policies are enforced by the network solution implemented on kubernetes cluster.  
Not all Network Solutions support network policies.

#### Solutions that support Network Policies:

- Kube-router
- Calico
- Romana
- Weave-net

#### Solutions that DO NOT support Network Policies:
- Flannel