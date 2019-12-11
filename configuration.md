# Configuration

## 1. ConfigMaps

Shorthand for ConfigMaps is `cm`.

### 1.1. Sample YAML definition

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

### 1.2. Using `kubectl` CLI

#### Configuration injection using `--from-literal`

```
kubectl create configmap \
  <config-name> --from-literal=<key>=<value>
  
kubectl create configmap \
  app-config --from-literal=APP_COLOR=blue \
             --from-literal=APP_MODE=prod
```

#### Configuration injection using `--from-file`

```
> cat app_config.properties
APP_COLOR: blue
APP_MODE: prod
```

```
kubectl create configmap \
  app-config --from-file=app_config.properties
```

### 1.3. Configuration injection in YAML definition files

#### Listing under `spec.containers.env`
```
    env:
    - name: APP_COLOR
      value: pink
    - name: APP_MODE
      value: dev
```

#### Injecting from a single configuration in an existing ConfigMap under `spec.containers.env`
```
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_COLOR
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_MODE
```

#### Injecting from existing ConfigMaps under `spec.containers.envFrom`
```
    envFrom:
    - configMapRef:
        name: app-config
    - configMapRef:
        name: app-config-2
```

### 1.4. Viewing ConfigMaps

```
kubectl get cm
kubectl describe cm <config-name>
```

## 2. Secrets

### 2.1. Sample YAML definition

```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql
  DB_User: root
  DB_Password: paswrd
```

### 2.2. Using `kubectl` CLI

#### Creating secret using `--from-literal`
```
kubectl create secret generic \
  <secret-name> --from-literal=<key>=<value>


kubectl create secret generic \
  app-secret --from-literal=DB_Host=mysql \
             --from-literal=DB_User=root \
             --from-literal=DB_Passport=paswrd
```

### 2.3. Secret injection in YAML definition files

#### Injecting from existing Secret under `spec.containers.envFrom`

```
    envFrom:
    - secretRef:
        name: app-secret
```

#### Injecting from a single configuration in an existing Secret under `spec.containers.env`

```
    env:
    - name: DB_Password
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_Password
```

#### Injecting from a secret as files in a volume

If you were to mount the secret as a volume in the pod, each attribute in the secret is created as a file with the value of the secret as its content. In this case, since we have three attributes in our secret, three files are created.

```
    volumes:
    - name: app-secret-volume
      secret:
        secretName: app-secret
```

```
> ls /opt/app-secret-volumes
DB_Host     DB_Password  DB_User

> cat /opt/app-secret-volumes/DB_Password
paswrd
```

#### Creating secret using `--from-file`


```
> cat app_secret.properties
DB_Host: mysql
DB_User: root
DB_Password: paswrd
```

```
kubectl create secret generic \
  <secret-name> --from-file=<path-to-file>

kubectl create secret generic \
  app-secret --from-file=app_secret.properties
```

### 2.4. Encoding Secrets

```
╔═══════════════════════╗   ╔══════════════════════════╗
║  DB_Host: mysql      ─╫───╫─► DB_Host: bXlzcWw=      ║
║  DB_User: root       ─╫───╫─► DB_User: cm9vdA==      ║
║  DB_Password: paswrd ─╫───╫─► DB_Password: cGFzd3Jk  ║
╚═══════════════════════╝   ╚══════════════════════════╝

> echo -n 'mysql' | base64
bXlzcWw=

> echo -n 'root' | base64
cm9vdA==

> echo -n 'paswrd' | base64
cGFzd3Jk
```

### 2.5. Decoding Secrets

```
╔════════════════════════╗   ╔═════════════════════════╗
║  DB_Host: mysql      ◄─╫───╫─ DB_Host: bXlzcWw=      ║
║  DB_User: root       ◄─╫───╫─ DB_User: cm9vdA==      ║
║  DB_Password: paswrd ◄─╫───╫─ DB_Password: cGFzd3Jk  ║
╚════════════════════════╝   ╚═════════════════════════╝

> echo -n 'bXlzcWw=' | base64 --decode
mysql

> echo -n 'cm9vdA==' | base64 --decode
root

> echo -n 'cGFzd3Jk' | base64 --decode
paswrd
```

### 2.6. Viewing Secrets

```
kubectl get secrets
kubectl describe secret <config-name>
```

## 3. Security Contexts

### Setting security context in pod or container
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
  securityContext:           # pod level
    runAsUser: 1001          # 
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:           # container level
      runAsUser: 1002          # 
      capabilities:            # capabilities are only supported at the
        add: ["SYS_TIME"]      #   container level and not at the pod level
```
```
> kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
Thu Apr 19 11:14:00 UTC 2012
```

## 4. Service Account

Shorthand for `serviceaccount` is `sa`.

### 4.1. Creating Service Account

```
kubectl create sa dashboard-sa
```

### 4.2. Default serviceaccount in pod

When serviceAccount is not defined in the pod definition file, kubernetes automatically mounts a default serviceAccount.

```
> kubectl get serviceAccount
NAME                   SECRETS   AGE
default                1         218d

> kubectl describe pod my-kubernetes-dashboard
Name:               my-kubernetes-dashboard
Namespace:          default
Annotations:        <none>
Status:             Running
IP:                 10.244.0.15
Containers:
  nginx:
    Image:          my-kubernetes-dashboard
Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-j4hkv (ro)
Conditions:
  Type           Status
Volumes:
  default-token-j4hkv:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-j4hkv
    Optional:    false

> kubectl exec -it my-kubernetes-dashboard ls /var/run/secrets/kubernetes.io/serviceaccount
ca.crt  namespace  

> kubectl exec -it my-kubernetes-dashboard cat /var/run/secrets/kubernetes.io/serviceaccount.token
eynbGclOiJSUzIlNiIsImtpZCI6Ii79.eylpc3Mi0i3rdWJ1cm51dGVzL3N1cnZ02VhY2NvdW50Iiwia3ViZX3uZXR1cy5pby9zZX32aWNlYWNjb3V
udC9uVW11c3BhY2UiOiRZWZhdWx0Iiwia3ViZXJuZXR1cy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImR1ZmFlbHQtdG9rZW4tajRoa3
iLOrdWl1cm51dGVAmlvL3N1cnZ02VhY2NvdW5OL3N1cnZOQUtYWNjb3VudC5uVW1lIjoiZGVmYXVsdCIsImtlYmVybmVOZXMuaW8vc2VydmljZWE2a
jY291bnQvc2VydmljZS1hY2NydW5OLnVpZCI6IjcxZGM4YWEA_TU2MGMtMTFl0004YmIeLTA4MDAyNzkzMTA3MiIsInNlYiI6InN5c3R1bTpzZXJWN 
```

### 4.3. Setting custom serviceaccount

```
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
  - name: my-kubernetes-dashboard
    image: my-kubernetes-dashboard
  serviceAccount: dashboard-sa
```

```
> kubectl get serviceAccount
NAME                   SECRETS   AGE
default                1         218d
dashboard-sa           1         4d

> kubectl describe pod my-kubernetes-dashboard
Name:               my-kubernetes-dashboard
Namespace:          default
Annotations:        <none>
Status:             Running
IP:                 10.244.0.15
Containers:
  nginx:
    Image:          my-kubernetes-dashboard
Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from dashboard-token-kbbdm (ro)
Conditions:
  Type           Status
Volumes:
  dashboard-token-kbbdm:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  dashboard-token-kbbdm
    Optional:    false
```

### 4.4. Disabling service account auto-mount using `automountServiceAccountToken`

```
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
  - name: my-kubernetes-dashboard
    image: my-kubernetes-dashboard
  automountServiceAccountToken: false
```

### Changing default serviceaccount in deployment definition file

```
spec:
  template:
    spec:
      serviceAccountName: default
```

## 5. Resource Requirements

### 5.1. Setting resource in pod definition file

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
```

### 5.2. Setting resource using `kubectl run` command
 
```
kubectl run nginx --image=nginx --restart=Never \
  --requests=cpu=100m,memory=256Mi \
  --limits=cpu=200m,memory=512Mi \
  --dry-run -o yaml
```

### 5.3. CPU values

```
0.1 CPU = 100m (milli)
minimum = 1m (milli)
1 CPU   = 1 AWS vCPU
          1 GCP Core
          1 Azure Core
          1 Hyperthread
```

### 5.4. Memory values

```
1G (Gigabyte) = 1,000,000,000 bytes
1M (Megabyte) = 1,000,000 bytes
1K (Kilobyte) = 1,000 bytes

1Gi (Gibibyte) = 1,073,741,824 bytes
1Mi (Mebibyte) = 1,048,576 bytes
1Ki (Kibibyte) = 1,024
```

### 5.5. Resource Limits

Pods can request for resource as required.  
Set limits to throttle the resource usage of the pod.

```
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
```

> Note: Resource limits is set "per container"

Kubernetes limits can throttle the CPU usage.  
But for memory usage, the container can use more memory than the limit.  
When this happens constantly, the pod will be terminated.

## 6. Taints and Tolerations

Taints and Tolerations does not tell the pod to go to a particular node.  
Instead, it tells the node to only accept pods with certain tolerations.

Restricting pods to certain nodes can be achieved through Node Affinity.

### 6.1. Tainting nodes
```
kubectl taint nodes <node-name> <key=value>:<taint-effect>
kubectl taint nodes node01 app=blue:NoSchedule
```

#### Removing taint in node

```
kubectl taint nodes <node-name> <key>:<taint-effect>-'
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-'
```

#### Taint effects
`NoSchedule`: pod will not be scheduled on the node  
`PreferNoSchedule`: system will try to avoid placing a pod on the node but it is not guaranteed 
`NoExecute`: new pods will not schedule on the node and existing pods, if any, will be evicted if they do not tolerate the taint. Evicted pods are killed.

### 6.2. Setting tolerations in pod definition file

```
       ╔═══════════════════════════════════════════════╗
       ║ kubectl taint nodes node1 app=blue:NoSchedule ║
       ╚════════════════════════════╪═══╪══╪════╪══════╝
                                    │   │  │    │
╔════════════════════════════╗      │   │  │    │
║ apiVersion: v1             ║      │   │  │    │
║ kind: Pod                  ║      │   │  │    │
║ metadata:                  ║      │   │  │    │
║   name: myapp-pod          ║      │   │  │    │
║ spec:                      ║      │   │  │    │
║   containers:              ║      │   │  │    │
║   - name: nginx-container  ║      │   │  │    │
║     image: nginx           ║      │   │  │    │
║   tolerations:             ║      │   │  │    │
║   - key: "app" ◄───────────╫──────┘   │  │    │
║     value: "blue" ◄────────╫──────────┘  │    │
║     operator: "Equal" ◄────╫─────────────┘    │
║     effect: "NoSchedule" ◄─╫──────────────────┘
╚════════════════════════════╝
```

### 6.3. Viewing taints in nodes

```
> kubectl describe node kubemaster | grep Taints
Taints:            node-role.kubernetes.io/master:NoSchedule
```

## 7. Node Selectors

### 7.1. Setting node selectors in pod definition file

```
apiVersion: v1           
kind: Pod                
metadata:                
  name: myapp-pod        
spec:                    
  containers:            
  - name: nginx-container
    image: nginx         
  nodeSelector:
    size: Large
```

> Note: Key-values under nodeSelector are labels assigned to the nodes.  
> The scheduler uses these labels to match and identify the right node to place the pods on. So it must be set on the node beforehand.

### 7.2. Setting labels to nodes

```
kubectl label node <node-name> <key>=<value>
kubectl label nodes node-1 size=Large
```
 
### 7.3. Limitations

`nodeSelector` is unable to cater for advanced expressions like:
- Large OR Medium
- NOT Small

These can be done using Node Affinity and Anti-Affinity.

## 8. Node Affinity

### 8.1. Sample node affinity setting in pod definition file

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
```

### 8.2. Node Affinity and Anti Affinity operators

#### Node Affinity

`In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`

#### Anti Affinity

`NotIn`, `DoesNotExist`  
...or use node taints to repel pods from specific nodes.

### 8.3. Node Affinity types

`requiredDuringSchedulingignoredDuringExecution`  
if no match is found, pod is not scheduled

`preferredDuringSchedulingIgnoredDuringExecution`  
if no match is found, node affinity rules is ignored and pod is scheduled

> (planned) `requiredDuringSchedulingrequiredDuringExecution`  
> if no match is found, pod is not scheduled  
> if label in the node is changed or deleted, the running pod will be evicted 