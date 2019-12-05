# Configuration

## 1. ConfigMaps

Shorthand for ConfigMaps is `cm`.

### Sample YAML definition

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

### Using `kubectl` CLI

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

### Configuration injection in YAML definition files

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

### Viewing ConfigMaps

```
kubectl get cm
kubectl describe cm <config-name>
```

## 2. Secrets

### Sample YAML definition

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

### Using `kubectl` CLI

#### Creating secret using `--from-literal`
```
kubectl create secret generic \
  <secret-name> --from-literal=<key>=<value>


kubectl create secret generic \
  app-secret --from-literal=DB_Host=mysql \
             --from-literal=DB_User=root \
             --from-literal=DB_Passport=paswrd
```

### Secret injection in YAML definition files

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

If you were to mount the secret as a volume in the pod, each attribute in the secret is created as a file with thte value of the secret as its content. In this case, since we have three attributes in our secret, three files are created.

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

### Encoding Secrets

```
╔═══════════════════════╗   ╔══════════════════════════╗
║  DB_Host: mysql      ─╫───╫─► DB_Host: bXlzcWw=      ║
║  DB_User: root       ─╫───╫─► DB_User: cm9vdA==      ║
║  DB_Password: paswrd ─╫───╫─► DB_Password: cGFzd3Jk  ║
╚═══════════════════════╝   ╚══════════════════════════╝

> echo -n `mysql` | base64
bXlzcWw=

> echo -n `root` | base64
cm9vdA==

> echo -n `paswrd` | base64
cGFzd3Jk
```

### Decoding Secrets

```
╔════════════════════════╗   ╔═════════════════════════╗
║  DB_Host: mysql      ◄─╫───╫─ DB_Host: bXlzcWw=      ║
║  DB_User: root       ◄─╫───╫─ DB_User: cm9vdA==      ║
║  DB_Password: paswrd ◄─╫───╫─ DB_Password: cGFzd3Jk  ║
╚════════════════════════╝   ╚═════════════════════════╝

> echo -n `bXlzcWw=` | base64 --decode
mysql

> echo -n `cm9vdA==` | base64 --decode
root

> echo -n `cGFzd3Jk` | base64 --decode
paswrd
```

### Viewing Secrets

```
kubectl get secrets
kubectl describe secret <config-name>
```

## 3. Security Contexts

## 4. Service Account

## 5. Resource Requirements

## 6. Taints and Tolerations

## 7. Node Selectors

## 8. Node Affinity
