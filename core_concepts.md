# CORE CONCEPTS

#### Using YAML definition files (applicable for any k8s resource)
```
kubectl create  -f resource-definition.yaml
kubectl apply   -f resource-definition.yaml
kubectl replace -f resource-definition.yaml
kubectl delete  -f resource-definition.yaml
```

#### Navigating k8s resources
```
kubectl describe <resource> <name>
kubectl get      <resource>
kubectl get      <resource> <name> [-o wide]
kubectl delete   <resource> <name>

kubectl get all
```
Kubectl resources and shorthand names
- `pod` or `po`
- `replicationcontroller` or `rc`
- `replicationset` or `rs`
- `deployment` or `deploy`
- `namespace` or `ns`

## Pods

### Sample YAML definition
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

### PODS command line reference sheet

#### Using `kubectl run` command
```
kubectl run \
  --dry-run \
  --generator=run-pod/v1 \
  --image=nginx \
  --port=8080 \
  --env="DNS_DOMAN=cluster" \
  --labels="tier=db" \
  --command echo hi \
  nginx \
  --output yaml > pod-definition.yaml
```
> Note: All generators except `run-pod/v1` are deprecated.

#### Using `kubectl set` command to change pod image
```
kubectl set image pod/nginx nginx=nginx:1.7.1 redis=redis:4.0.14
```

#### Using `kubectl exec` command to get as shell to a running container in the pod
```
kubectl exec -it <pod-name> bash
kubectl exec -it <pod-name> /bin/sh
```

#### For one-time use pods (e.g. delete the pod after executing an operation)
```
kubectl run \
  --image=<image-name> \
  --command <command> <args>
  --restart=Never
  <pod name>  
```

## Replication Controller

### Sample YAML definition
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
replicas: 3
```

### Replication Controller command line reference sheet

#### Using `kubectl run` command
```
kubectl run \
  --dry-run \
  --generator=run/v1 \
  --image=redis \
  --replicas=3 \
  rc-redis \
  --output yaml > rc-definition.yaml
```


## Replication Set

### Sample YAML definition
```
apiVersion: apps/v1
kind: ReplicationSet
metadata:
  name: myapp-rs
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
replicas: 3
selector:                      # "selector" is REQUIRED in rs but not in rc. 
  matchLabels:                 # This is the only difference from rc and rs 
    type: front-end            #   definition files.
```

### Replication Set command line reference sheet

#### Using `kubectl run` command
There is no specific way to use `kubectl run` for replication sets.  
Just `--dry-run` a deployment and replace the kind in the YAML file to ReplicationSet.

#### Using the `kubectl scale` command
```
kubectl scale --replicas=6 -f rs-definition.yaml
```
```
kubectl scale --replicas=6  rs rs-redis
```


## Deployment

### Sample YAML definition
```
apiVersion: apps/v1
kind: Deployment               # The only difference from replication set
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
replicas: 3
selector:
  matchLabels:
    type: front-end
```

### Deployment command line reference sheet

#### Using `kubectl run` command
```
kubectl run \
  --dry-run \
  --image=nginx \
  --port=8080 \
  --env="DNS_DOMAN=cluster" \
  --env="POD_NAMESPACE=default" \
  --labels="tier=db,env=prod" \
  --replicas=5
  --command -- echo hi \
  nginx \
  --output yaml > deployment-definition.yaml
```

#### Using `kubectl set` command to change pod image
```
kubectl set image deploy <deployment-name> nginx=nginx:1.7.1
```

#### Using `kubectl autoscale` command
```
kubectl autoscale deploy <deployment-name> \
  --dry-run \
  --min=5 --max=10 \
  --cpu-percent=80 \
  -o yaml
```


## Namespace

### Creating a namespace through YAML definition file
```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

### Creating a namespace through command line
```
kubectl create ns <namespace-name>
```

### Other commands relating to namespaces
```
kubectl get all --all-namespaces
kubectl run nginx --image=nginx -n <namespace-name>
```
