# POD DESIGN

## 1. Labels and Selectors

### 1.1. Listing and filtering pods

#### `--show-labels` displays all labels under a LABELS column
```
kubectl get po --show-labels
```

#### `--selector, -l` filter results by labels in key:value format
```
kubectl get po --selector app=App1
kubectl get po -l app=App1
```

#### `--label-columns, -L` displays labels and its values in a separate column
```
kubectl get po --label-columns=app,release
kubectl get po -L app,release
```

#### `--field-selector` filters results by fields using comparison operators
```
kubectl get po --field-selector=status.phase!=Running,spec.restartPolicy=Always
```

### 1.2. Labeling Pods

#### `--overwrite` overwrites if label already exists on the resource
```
kubectl label po nginx2 app=App2 --overwrite
```

#### `-` appended after the label key removes the label from the resource
```
kubectl label po <pod1-name> <pod2-name> <pod3-name> app-
```
> Note: `--overwrite` is not required

#### `--resource-version` Update pod 'foo' only if the resource is unchanged from version 1
```
kubectl label po foo status=unhealthy --resource-version=1
```

## 2. Annotations

### 2.1. Annotating Pods

#### `--overwrite` overwrites if annotation already exists on the resource
```
kubectl annotate po <pod1-name> <pod2-name> description='my description' --overwrite
```

#### `-` appended after the annotation key removes the annotation from the resource
```
kubectl annotate po <pod1-name> <pod2-name> description-
```

#### `--resource-version` Update pod 'foo' only if the resource is unchanged from version 1
```
kubectl annotate po foo description='my frontend running nginx' --resource-version=1
```

## 3. Rolling Updates and Deployment Rollbacks

### 3.1. Strategies

#### Recreate Strategy

Recreate deletes the resources of the current deployment before deploying the updated resources resulting to service downtime.

#### Rolling Updates Strategy

Rolling Updates deletes and updates the current deployment's resources one by one so there is no service downtime.

### 3.2. Handling Rollouts

#### creating and updating deployments
```
kubectl create -f deployment-definition.yaml
kubectl apply -f deployment-definition.yml
kubectl set image deploy/myapp-deployment nginx=nginx:1.9.1
```
> Note: refer to [Deployment Command Line Reference](./core_concepts.md#deployment-command-line-reference-sheet) for details.

#### `status` to check the rollout status
```
kubectl rollout status deployment/myapp-deployment
```

#### `history` to check the rollout history
```
kubectl rollout history deployment/myapp-deployment
```

#### `history --revision` to view history details, including podTemplate of the revision specified of a specific revision
```
kubectl rollout history deployment/myapp-deployment --revision=3
```

#### `pause` and `resume` ongoing rollouts
```
kubectl rollout pause deploy nginx
kubectl rollout resume deploy nginx
```

#### `undo` to rollback (`--to-revision` to specify to a specific revision)
```
kubectl rollout undo deployment/myapp-deployment --to-revision=2
```

## 4. Jobs and Cronjobs

### 4.1. Restart Policies

- Always, a deployment is created (default, not supported for CronJobs) 
- OnFailure, a job is created
- Never, a regular pod is created (default for CronJobs) 

### 4.2. Sample YAML definition

#### Jobs

```
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  backoffLimit: 25   # no. of times pods can be recreated
  completions: 3     # target no. of completed jobs
  parallelism: 3     # no. of pods that runs concurrently to run the job
  template:
    spec:
      containers:                          # 
      - name: math-add                     # 
        image: ubuntu                      # spec of pod definition
        command: ['expr', '3', '+', '2']   # 
      restartPolicy: Never                 #
```

#### CronJobs

```
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:                                                                 # 
      backoffLimit: 25   # no. of times pods can be recreated             # 
      completions: 3     # target no. of completed jobs                   # 
      parallelism: 3     # no. of concurrent pods to run the job          # 
      template:                                                           # 
        spec:                                                             # spec of job template
          containers:                          #                          # 
          - name: math-add                     #                          # 
            image: ubuntu                      # spec of pod definition   # 
            command: ['expr', '3', '+', '2']   #                          # 
          restartPolicy: Never                 #                          # 
```

### 4.3. Using `kubectl run` command (deprecated)

#### Jobs

```
kubectl run \
  --generator=job/v1 \
  --image=ubuntu myjob \
  --restart=OnFailure \
  -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

#### CronJobs

```
kubectl run \
  --generator=cronjob/v1beta1 \
  --image=ubuntu \
  --restart=Never \
  --schedule="30 21 * * *" \
  cron-job 
```